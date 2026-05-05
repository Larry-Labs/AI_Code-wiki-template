# 05-logic.md 填写指南

> 本指南展示各项目类型的逻辑流程示例。
> 对应模板：`template/05-logic.md`

---

## Web 后端 — 用户注册流程

### 触发条件

用户提交 `POST /api/auth/register` 请求

### 执行步骤

1. 校验请求参数（email 格式、密码长度）
2. 查询数据库：邮箱是否已注册
3. 若已注册 → 返回 409
4. bcrypt 哈希密码（rounds=10）
5. 写入 users 表
6. 生成 JWT（有效期 7 天）
7. 异步发送欢迎邮件
8. 返回 `{ user, token }`

### 输入/输出

- **输入**：`{ email: string, password: string, name: string }`
- **输出**：`{ user: User, token: string }`

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| 邮箱已存在 | 返回 409，不泄露是否已注册（安全考虑） |
| 邮件发送失败 | 不阻塞注册，后台重试 3 次 |
| 数据库写入失败 | 返回 500，不发送邮件 |
| 并发注册同一邮箱 | 数据库唯一约束兜底，返回 409 |

### 涉及文件

- `src/api/auth/register.ts` — 路由入口、参数校验
- `src/services/auth.ts` — 业务逻辑
- `src/repositories/user.ts` — 数据库操作
- `src/utils/crypto.ts` — 密码哈希
- `src/utils/jwt.ts` — Token 生成
- `src/services/email.ts` — 邮件发送

---

## 嵌入式 — 传感器数据采集流程

### 触发条件

TIM2 定时器中断，周期 30 秒

### 执行步骤

1. 获取 I2C 总线互斥锁
2. 发送 SHT31 单次测量命令 `{0x24, 0x00}`
3. 等待 15ms（datasheet 指定）
4. 读取 6 字节原始数据
5. CRC-8 校验（多项式 0x31）
6. 转换为浮点值：
   - `temp = -45 + 175 * raw_temp / 65535`
   - `hum = 100 * raw_hum / 65535`
7. 释放 I2C 锁
8. 写入 ring buffer

### 输入/输出

- **输入**：无（定时触发）
- **输出**：`{ temp: float, hum: float, timestamp: uint32 }` 写入 ring buffer

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| I2C NACK | 重试 3 次，间隔 10ms，仍失败则写入错误码 0x01 |
| CRC 校验失败 | 丢弃本次数据，不重试 |
| I2C 总线超时 | HAL_I2C_State 返回 HAL_TIMEOUT，复位 I2C 外设 |
| Ring buffer 满 | 覆盖最旧数据，递增丢弃计数器 |

### 涉及文件

- `src/tasks/sensor_task.c` — 任务入口
- `drivers/sht31.c` — SHT31 驱动
- `src/buffer/ring.c` — 环形缓冲区
- `src/utils/crc.c` — CRC 计算

---

## 嵌入式 — AUTOSAR 诊断会话状态机

### 状态定义

```
┌─────────────┐   0x10 01    ┌─────────────┐
│   Default   │─────────────▶│  Extended   │
│   Session   │◀─────────────│  Session    │
│  (默认会话)  │   0x10 01    │ (扩展会话)   │
└──────┬──────┘              └──────┬──────┘
       │                            │
       │ 0x10 02                    │ 0x10 02
       ▼                            ▼
┌─────────────┐              ┌─────────────┐
│ Programming │              │ Programming │
│  Session    │◀─────────────│  Session    │
│ (编程会话)   │   0x10 02    │ (编程会话)   │
└─────────────┘              └─────────────┘
```

### 会话切换规则

| 当前会话 | 目标会话 | 条件 | 说明 |
|----------|----------|------|------|
| Default → Extended | 0x10 01 | 无 | 无需安全解锁 |
| Default → Programming | 0x10 02 | 需先 0x27 安全解锁 | S3 超时保护 |
| Extended → Default | 0x10 01 或 S3 超时 | S3=5000ms | 自动回退 |
| Programming → Default | 0x10 01 或 P2 超时 | P2=5000ms | 超时复位 |

### 超时机制

- **S3 Server Timer**：扩展会话/编程会话无请求 5s 后自动回退 Default
- **P2 Server Timer**：单个请求处理超时 5s，超时返回 NRC 0x78（ResponsePending）
- **P2* Server Timer**：NRC 0x78 后的扩展超时 5s

---

## 嵌入式 — AUTOSAR DEM 事件处理流程

### 触发条件

SWC 检测到故障（如传感器断线、通信超时）

### 执行步骤

1. SWC 调用 `Dem_SetEventStatus(EventId, DEM_EVENT_STATUS_FAILED)`
2. DEM 记录事件状态（Pending → Confirmed）
3. 达到 trip 阈值后写入 DTC 到 NVM
4. 同时记录快照数据（环境数据：车速、电压、温度等）
5. 通过 `Dem_GetDTCStatusByte()` 返回 DTC 状态给诊断仪

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| 同一 DTC 重复上报 | 更新 Occurrence Counter，不重复写入 NVM |
| DTC 达到老化阈值 | 自动清除（Aging Counter 递增） |
| NVM 写入失败 | 保持内存中的 DTC 状态，下次写入重试 |
| 快照数据缓冲区满 | 丢弃最早的快照记录 |

### 涉及文件

- `src/swc/DiagManager/` — 诊断管理 SWC
- `generated/Dem/` — DEM 配置生成代码
- `generated/NvM/` — NVM 块配置

---

## CLI 工具 — 命令执行流程

### 触发条件

用户执行 `mycli build --target linux --release`

### 执行步骤

1. 解析命令行参数（cobra/pflag）
2. 读取配置文件 `.mycli/config.yaml`
3. 校验参数合法性（target 是否支持）
4. 执行构建逻辑（调用 go build）
5. 输出进度到 stderr
6. 写入产物到 `./dist/`

### 输入/输出

- **输入**：命令行参数 `--target`, `--release`
- **输出**：构建产物文件，退出码 0/1

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| 配置文件不存在 | 使用默认配置 |
| target 不支持 | 输出支持列表，退出码 1 |
| 构建失败 | 输出错误信息，退出码 1 |
| 磁盘空间不足 | 提前检查，不足则报错 |

---

## iOS — 网络请求流程

### 触发条件

用户下拉刷新或页面加载

### 执行步骤

1. ViewModel 调用 `repository.fetchUsers()`
2. Repository 检查本地缓存（CoreData）
3. 若缓存有效且未过期 → 返回缓存数据
4. 若缓存过期 → 发起 URLSession 请求
5. JSON 解码为 `[User]` 模型
6. 更新本地缓存
7. 返回数据，ViewModel 更新 `@Published` 属性
8. View 自动刷新

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| 网络不可用 | 返回缓存数据，标记为离线 |
| JSON 解码失败 | 返回 `.decodingError`，记录日志 |
| HTTP 401 | 触发 Token 刷新，重试一次 |
| HTTP 429 | 退避重试（指数退避，最多 3 次） |

---

## 数据管道 — ETL 任务流程

### 触发条件

Airflow DAG 调度（每日 UTC 02:00）

### 执行步骤

1. Extractor 连接数据源（API/数据库）
2. 拉取增量数据（基于 `updated_at` 时间戳）
3. Transformer 清洗：去重、类型转换、空值处理
4. Transformer 聚合：按业务维度汇总
5. Loader 写入目标表（upsert 语义）
6. 记录处理行数和耗时

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| 数据源不可达 | 重试 3 次，间隔 5min，仍失败则告警 |
| 数据量超预期 | 分批处理，每批 10000 行 |
| Schema 变更 | 检测字段差异，记录但不中断 |
| 目标表写入冲突 | upsert，以最新 updated_at 为准 |

---

## ML/AI — 模型训练流程

### 触发条件

执行 `python -m src.train --config configs/base.yaml`

### 执行步骤

1. 加载配置（学习率、batch size、epoch 数）
2. 初始化 Dataset 和 DataLoader
3. 初始化 Model、Optimizer、Scheduler
4. 循环每个 epoch：
   - 遍历 DataLoader 获取 batch
   - 前向传播 → 计算 loss
   - 反向传播 → 更新参数
   - 记录 train loss
5. 每 N 步在验证集评估
6. 保存最优 checkpoint

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| GPU 显存不足 | 自动减小 batch size，重试 |
| Loss 爆炸 (NaN) | 回滚到上一个 checkpoint，降低学习率 |
| 训练中断 | 从最近 checkpoint 恢复 |
| 过拟合 | Early stopping（patience=5） |

---

## 游戏 — 主循环流程

### 触发条件

每帧触发（目标 60fps，16.67ms/帧）

### 执行步骤

1. `Input.poll()` — 读取输入事件
2. `Physics.update(dt)` — 物理模拟
3. `AI.update(dt)` — AI 决策
4. `GameLogic.update(dt)` — 游戏逻辑
5. `Animation.update(dt)` — 动画更新
6. `Renderer.draw()` — 渲染画面
7. `Audio.update()` — 音频处理

### 边界情况

| 场景 | 处理方式 |
|------|---------|
| 帧率低于 30fps | 降低渲染质量（LOD） |
| 物理穿透 | CCD 连续碰撞检测 |
| 网络延迟 > 100ms | 客户端预测 + 服务器校正 |

---

## Web 后端 — 订单状态机

```
                    ┌──────────┐
         创建订单   │          │  取消
    ┌──────────────▶│ pending  │──────────┐
    │               │          │          │
    │               └────┬─────┘          │
    │                    │                ▼
    │               支付成功          ┌──────────┐
    │                    │            │cancelled │
    │               ┌────▼─────┐      └──────────┘
    │               │          │
    │               │   paid   │
    │               │          │
    │               └────┬─────┘
    │                    │
    │               发货 │
    │               ┌────▼─────┐
    │               │          │
    └───────────────│ shipped  │
         退货       │          │
                    └──────────┘
```

### 状态转换规则

| 当前状态 | 事件 | 下一状态 | 条件 | 行为 |
|---------|------|---------|------|------|
| pending | payment.success | paid | 金额匹配 | 记录 paidAt，扣减库存 |
| pending | order.cancel | cancelled | 未超时 | 释放库存 |
| pending | timeout (24h) | cancelled | 自动 | 释放库存 |
| paid | order.ship | shipped | 有物流单号 | 发送通知 |
| shipped | order.return | paid | 7天内 | 退款、恢复库存 |

---

## Web 后端 — 核心算法示例

### 中值滤波

- **用途**：去除传感器数据中的脉冲噪声
- **窗口大小**：5
- **算法**：取最近 5 个采样值，排序后取中间值
- **触发**：每次新数据写入 ring buffer 后
- **文件**：`src/filters/median.c`

### JWT 签发

- **算法**：HS256
- **Payload**：`{ sub: userId, role: userRole, iat: now, exp: now + 7d }`
- **密钥来源**：环境变量 `JWT_SECRET`
- **文件**：`src/utils/jwt.ts`

---

## Web 后端 — 业务规则示例

- 每个用户最多有 3 个活跃订单
- 同一邮箱不能重复注册（数据库唯一约束）
- 订单金额以分为单位存储，避免浮点精度问题
- 密码最少 8 位，必须包含大小写和数字

---

## 嵌入式 — 业务规则示例

- 传感器数据超过阈值（temp > 60°C）立即上报，不等定时周期

### 嵌入式 — AUTOSAR 业务规则示例

- 车速 > 5 km/h 时禁止进入 Programming Session（安全保护）
- 发动机运行时禁止执行 RoutineControl 刷写检查
- DTC 的 trip 阈值为 1 次（Confirmed），aging 阈值为 40 次（自动清除）
- NVM 写入失败时重试 3 次，仍失败则记录 DEM 事件
- 安全访问（0x27）种子-密钥算法：seed = random, key = seed XOR 0xA5A5 + 0x1234
