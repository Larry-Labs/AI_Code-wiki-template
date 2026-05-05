# 10-human-guide 填写指南

> 本指南展示各项目类型的「人类阅读指南」填写示例。
> 对应模板：`template/10-human-guide.md`

---

## Web 后端

### 项目概述

这是一个电商后台 API 服务，为前端应用和移动端提供商品、订单、支付等核心业务接口。
解决商品管理、订单流转、支付对接等后台业务逻辑问题。
给前端开发者、移动端开发者、运营人员使用的后端服务。

### 技术路线

用 Express 搭建 HTTP 服务，用 Prisma 操作 PostgreSQL，因为 TypeScript 全栈一致性好、Prisma 类型安全。
用 Redis 做缓存和 Session，因为需要高并发下快速响应。
用 JWT + Refresh Token 做认证，因为无状态、适合分布式部署。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  前端/移动端   │
└──────┬──────┘
       │ HTTPS
       ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  API 服务    │────▶│   Stripe    │     │   Redis     │
│  (Express)  │────▶│   (支付)     │     │   (缓存)     │
└──────┬──────┘     └─────────────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│ PostgreSQL  │
│  (业务数据)  │
└─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  API 路由层  │────▶│  服务/业务层  │────▶│  数据访问层  │
│  (Express)  │     │ (Services)  │     │ (Prisma)   │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
             ┌──────────┐  ┌──────────┐
             │ JWT/加密  │  │ 邮件服务  │
             └──────────┘  └──────────┘
```

### 核心数据流

#### 场景 1：用户下单

```
用户提交订单
  → API 路由层    (接收请求、参数校验)
  → 订单服务      (库存检查、价格计算)
  → 支付模块      (调用 Stripe 创建 PaymentIntent)
  → 数据访问层    (写入订单表，扣减库存)
  → 返回订单确认
```

#### 场景 2：用户登录

```
用户提交邮箱密码
  → API 路由层    (接收请求)
  → 认证服务      (验证密码、生成 JWT)
  → 返回 token + refreshToken
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件（10-human-guide.md）— 建立全貌
2. `00-manifest.yaml` — 技术栈和入口点
3. `02-architecture.md` — 架构和模块
4. `05-logic.md` — 核心业务流程（下单、登录）

#### 深入理解（2 小时）

5. `03-interfaces.md` — API 接口定义
6. `04-data.md` — 数据库表结构
7. `08-file-specs/` — 关键文件的实现细节

#### 开发时查阅

8. `06-config.md` — 环境变量和部署
9. `07-conventions.md` — 代码风格
10. `01-constraints.md` — 性能和安全约束

### 快速上手

#### 环境准备

- Node.js 20+
- PostgreSQL 16+
- Stripe 测试密钥（在 Stripe Dashboard 获取）

#### 启动项目

```bash
git clone <仓库地址>
cd my-api
npm install
cp .env.example .env   # 编辑填入数据库连接和 Stripe 密钥
npx prisma migrate dev  # 初始化数据库
npm run dev             # 启动开发服务器
```

#### 验证

访问 `http://localhost:3000/health`，返回 `{"status":"ok"}` 表示成功。

#### 第一个改动

1. 打开 `src/api/health/routes.ts`
2. 找到 health check 端点
3. 在返回的 JSON 中加上 `"version": "1.0.0"`
4. 访问 `http://localhost:3000/health`，看到 version 字段

### 常见场景

#### 场景：添加一个新接口

1. 在 `src/api/` 下创建新路由文件
2. 在 `src/services/` 中实现业务逻辑
3. 在 `src/api/index.ts` 中注册路由
4. 用 curl 或 Postman 测试

#### 场景：修改数据库表

1. 修改 `prisma/schema.prisma`
2. 运行 `npx prisma migrate dev --name <迁移名>`
3. 更新对应的 Service 和 Repository 文件
4. 更新受影响的 API 接口

#### 场景：排查线上问题

1. 查看 Sentry 错误面板
2. 检查 `logs/` 目录下的日志
3. 常见错误：401（Token 过期）、409（数据冲突）、500（服务端异常）

### 设计决策解读

#### 为什么选 Prisma 而不是 TypeORM

当时需要一个类型安全的 ORM，候选有 Prisma、TypeORM、Drizzle。

Prisma 的 Schema 语言独立于 TypeScript，能自动生成类型，迁移体验好。
TypeORM 装饰器模式在 TypeScript 编译时偶尔有坑。

代价是 Prisma 的 Schema 语言有学习成本，复杂查询不如原生 SQL 灵活。

### FAQ

#### Q: 为什么用 Refresh Token 而不是延长 JWT 有效期？

短生命周期 JWT（15 分钟）+ Refresh Token（7 天）平衡了安全和体验。JWT 泄露后最多 15 分钟有效，Refresh Token 可以吊销。

#### Q: 数据库连接数不够怎么办？

检查 Prisma 连接池配置（`connection_limit`），默认 10。高并发场景需要加大或用 PgBouncer 做连接池代理。

---

## 嵌入式

### 项目概述

这是一个 LoRa 温湿度采集节点，部署在农业大棚中，定时采集环境数据并通过 LoRa 无线上传。
解决大棚环境监测的远程数据采集问题。
给农业物联网平台提供底层传感器数据。

### 技术路线

用 STM32L071 做主控，因为超低功耗适合电池供电场景。
用 FreeRTOS 做任务调度，因为需要同时处理传感器采集和 LoRa 发送。
用 SHT31 传感器采集温湿度，I2C 接口简单可靠。
用 SX1276 芯片做 LoRa 通信，覆盖距离远、功耗低。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  SHT31 传感器 │
│  (I2C)      │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  STM32L071  │────▶│  SX1276     │
│  (主控 MCU)  │     │  (LoRa 模块) │
└──────┬──────┘     └──────┬──────┘
       │                   │
       ▼                   ▼
┌─────────────┐     ┌─────────────┐
│  Flash 存储  │     │  LoRa 网关   │
│  (本地缓存)  │     │  (数据上传)  │
└─────────────┘     └─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  传感器任务  │────▶│  环形缓冲区  │────▶│  LoRa 任务   │
│  (采集数据)  │     │  (暂存数据)  │     │  (发送数据)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心数据流

#### 场景 1：定时采集并上报

```
FreeRTOS 定时器触发（30s）
  → 传感器任务    (I2C 读取 SHT31，CRC 校验)
  → 环形缓冲区   (暂存采集数据)
  → LoRa 任务    (从缓冲区取出，组装帧，SPI 发送)
  → 进入低功耗休眠
```

#### 场景 2：采集失败重试

```
传感器任务
  → I2C 读取失败
  → 等待 100ms 重试（最多 3 次）
  → 3 次都失败 → 记录错误到 Flash，跳过本轮采集
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `00-manifest.yaml` — 硬件和软件栈
3. `01-constraints.md` — 硬件约束和功耗预算
4. `05-logic.md` — 采集和发送流程

#### 深入理解（2 小时）

5. `08-file-specs/drivers/sht31.c` — 传感器驱动细节
6. `08-file-specs/drivers/sx1276.c` — LoRa 驱动细节
7. `04-data.md` — 数据帧格式和协议

#### 开发时查阅

8. `02-architecture.md` — 模块划分
9. `03-interfaces.md` — I2C/SPI 接口时序
10. `07-conventions.md` — MISRA C 编码规范

### 快速上手

#### 环境准备

- arm-none-eabi-gcc 13+
- STM32CubeMX（引脚配置）
- ST-Link 调试器
- 目标开发板 + SHT31 模块 + SX1276 模块

#### 启动项目

```bash
git clone <仓库地址>
cd my-sensor-node
# 用 STM32CubeMX 打开 .ioc 文件生成代码（如需要）
make -j4
# 烧录
st-flash write build/firmware.bin 0x08000000
```

#### 验证

连接串口（115200 波特率），看到每 30 秒输出一次温湿度数据。

#### 第一个改动

1. 打开 `src/tasks/sensor_task.c`
2. 找到采集周期常量 `#define采集间隔 30`
3. 改成 `60`（60 秒采集一次）
4. 重新编译烧录，观察串口输出间隔变化

### 常见场景

#### 场景：更换传感器型号

1. 在 `drivers/` 下参考 `sht31.c` 编写新驱动
2. 实现 Init、Measure 函数
3. 在 `sensor_task.c` 中替换驱动调用
4. 验证数据正确性

#### 场景：修改 LoRa 发送频率

1. 修改 `src/tasks/sensor_task.c` 中的采集周期
2. 修改 `src/tasks/lora_task.c` 中的发送逻辑
3. 注意功耗预算是否允许

#### 场景：排查采集数据异常

1. 用逻辑分析仪抓 I2C 波形
2. 检查 CRC 校验是否通过
3. 查看 `drivers/sht31.c` 中的错误处理日志
4. 常见问题：I2C 地址错误、上电时序不对

### 设计决策解读

#### 为什么用环形缓冲区而不是队列

传感器和 LoRa 任务的速率不一致：采集 30 秒一次，发送可能需要几百毫秒。

环形缓冲区无锁、固定内存、适合单生产者单消费者场景。
FreeRTOS 队列更通用但有额外的内存拷贝和锁开销。

代价是缓冲区满时新数据会覆盖旧数据，但对温湿度场景可以接受。

### FAQ

#### Q: 电池能用多久？

30 秒采集一次，平均功耗约 50μA，2000mAh 电池理论可用 4.5 年。实际受 LoRa 发送频率和环境温度影响。

#### Q: LoRa 信号穿不透金属大棚怎么办？

换用 433MHz 频段（比 868MHz 穿透力更强），或在大棚外加装天线延长线。

---

## CLI 工具 (Go)

### 项目概述

这是一个项目脚手架生成工具，通过交互式问答快速创建项目模板。
解决重复创建项目结构、配置文件的效率问题。
给后端开发者、全栈开发者使用的命令行工具。

### 技术路线

用 Go + Cobra 实现 CLI 框架，因为 Go 编译快、单二进制分发方便。
用 survey 库做交互式问答，因为比自己写 stdin 读取更健壮。
用 embed 嵌入模板文件，因为不依赖外部文件、分发简单。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  开发者终端   │
└──────┬──────┘
       │ 命令行输入
       ▼
┌─────────────┐     ┌─────────────┐
│  CLI 工具    │────▶│  模板文件    │
│  (Cobra)    │     │  (embed)    │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│  文件系统    │
│  (生成项目)  │
└─────────────┘
```

### 核心数据流

#### 场景 1：初始化项目

```
用户运行 mycli init
  → 交互式问答    (项目名、语言、框架、数据库)
  → 配置生成      (写入 mycli.yaml)
  → 模板渲染      (根据选择渲染模板文件)
  → 文件写入      (输出到目标目录)
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `cmd/app/main.go` — 命令定义入口
3. `internal/cmd/init.go` — 核心 init 命令

#### 深入理解（1 小时）

4. `internal/config/config.go` — 配置读写
5. `internal/template/template.go` — 模板渲染逻辑

### 快速上手

#### 环境准备

- Go 1.22+

#### 启动项目

```bash
git clone <仓库地址>
cd mycli
go build -o mycli ./cmd/app
./mycli --help
```

#### 验证

运行 `./mycli --help` 看到命令列表。

#### 第一个改动

1. 打开 `internal/cmd/init.go`
2. 找到问候语 `"Welcome to MyCLI!"`
3. 改成 `"Hello from MyCLI!"`
4. `go build -o mycli ./cmd/app && ./mycli init`，看到新问候语

### 常见场景

#### 场景：添加新子命令

1. 在 `internal/cmd/` 下创建新文件
2. 定义 `cobra.Command`
3. 在 `cmd/app/main.go` 中注册
4. 运行 `./mycli <新命令> --help` 验证

### 设计决策解读

#### 为什么用 embed 而不是外部模板目录

embed 把模板编译进二进制，用户只需下载一个文件就能用。
外部模板需要额外安装步骤或联网下载。

代价是更新模板需要重新编译发布，不能热更新。

### FAQ

#### Q: 支持哪些语言的项目模板？

目前支持 Go、TypeScript、Python。通过 `--template` 参数选择。

---

## iOS 应用

### 项目概述

这是一个社交分享应用，用户可以发布图文动态、关注好友、点赞评论。
解决用户分享生活瞬间的社交需求。
给 iOS 用户使用的原生应用。

### 技术路线

用 SwiftUI 做界面，因为声明式语法开发效率高、苹果主推方向。
用 MVVM + Combine 做数据绑定，因为 SwiftUI 天然支持。
用 CoreData 做本地缓存，因为离线体验好、和 SwiftUI 集成好。
用 URLSession 做网络请求，因为系统原生、无额外依赖。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  iOS 用户    │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  本 App     │────▶│  后端 API   │
│  (SwiftUI)  │     │  (REST)    │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│  CoreData   │
│  (本地缓存)  │
└─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Features   │────▶│  Core 层    │────▶│  系统框架   │
│  (页面/UI)  │     │ (网络/缓存)  │     │ (URLSession)│
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心数据流

#### 场景 1：发布动态

```
用户点击发布
  → HomeViewModel    (组装数据)
  → APIClient        (POST /api/posts)
  → 服务端返回       (创建成功)
  → CoreData         (缓存到本地)
  → UI 刷新          (列表顶部显示新动态)
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `Sources/App/MyApp.swift` — App 入口
3. `Sources/Features/Home/HomeView.swift` — 主页面

#### 深入理解（2 小时）

4. `Sources/Core/Network/APIClient.swift` — 网络层
5. `Sources/Features/Home/HomeViewModel.swift` — 业务逻辑
6. `04-data.md` — 数据模型

### 快速上手

#### 环境准备

- macOS + Xcode 15+
- Apple Developer 账号（真机调试需要）

#### 启动项目

```bash
git clone <仓库地址>
open MyApp.xcodeproj
# 选择模拟器，Cmd+R 运行
```

#### 验证

模拟器显示首页动态列表（可能为空数据）。

#### 第一个改动

1. 打开 `Sources/Features/Home/HomeView.swift`
2. 找到导航栏标题 `"首页"`
3. 改成 `"动态"`
4. Cmd+R 运行，看到新标题

### 常见场景

#### 场景：新增一个页面

1. 在 `Sources/Features/` 下创建新文件夹
2. 创建 `XxxView.swift` + `XxxViewModel.swift`
3. 在 `NavGraph` 或上级页面中添加 NavigationLink
4. 编译运行

### 设计决策解读

#### 为什么用 CoreData 而不是直接 JSON 缓存

CoreData 支持增量更新、关系查询、和 SwiftUI 的 @FetchRequest 深度集成。
JSON 缓存简单但每次都要全量读写，大列表性能差。

代价是 CoreData 学习曲线陡，模型迁移需要额外处理。

### FAQ

#### Q: 最低支持到哪个 iOS 版本？

iOS 16+。因为用了 SwiftUI 的 NavigationStack 等新 API。

---

## Android 应用

### 项目概述

这是一个任务管理应用，支持创建任务、设置截止日期、分类管理。
解决个人任务跟踪和时间管理问题。
给 Android 用户使用的原生应用。

### 技术路线

用 Jetpack Compose 做界面，因为声明式 UI 是 Android 主推方向。
用 Hilt 做依赖注入，因为官方推荐、和 Compose 集成好。
用 Room 做本地数据库，因为类型安全、编译期校验 SQL。
用 Paging 3 做分页，因为大列表性能好、和 Compose 集成好。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  Android 用户│
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  本 App     │────▶│  后端 API   │
│  (Compose)  │     │  (REST)    │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│  Room DB    │
│  (本地数据)  │
└─────────────┘
```

### 核心数据流

#### 场景 1：创建任务

```
用户填写任务表单
  → HomeViewModel    (验证输入)
  → CreateTaskUseCase (业务逻辑)
  → TaskRepository   (写入 Room + 同步 API)
  → UI 刷新          (列表显示新任务)
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `app/build.gradle.kts` — 依赖配置
3. `presentation/home/HomeScreen.kt` — 主页面

#### 深入理解（2 小时）

4. `domain/usecase/` — 业务用例
5. `data/remote/ApiService.kt` — 网络层
6. `data/local/AppDatabase.kt` — 本地数据库

### 快速上手

#### 环境准备

- Android Studio Hedgehog+
- Android SDK 34+
- JDK 17+

#### 启动项目

```bash
git clone <仓库地址>
# 用 Android Studio 打开项目
# 选择模拟器，点击 Run
```

#### 验证

模拟器显示任务列表页面。

#### 第一个改动

1. 打开 `presentation/home/HomeScreen.kt`
2. 找到顶部标题 `"任务管理"`
3. 改成 `"我的待办"`
4. 运行，看到新标题

### 常见场景

#### 场景：添加新页面

1. 在 `presentation/` 下创建新文件夹
2. 创建 `XxxScreen.kt` + `XxxViewModel.kt`
3. 在 `NavGraph.kt` 中注册路由
4. 编译运行

### 设计决策解读

#### 为什么用 Room 而不是直接 SQLite

Room 提供编译期 SQL 校验、类型安全的 DAO、和 Paging/Flow 集成。
手写 SQLite 容易出错，SQL 字符串拼接风险高。

代价是 Room 对复杂联表查询的支持不如原生 SQL 灵活。

### FAQ

#### Q: 支持离线使用吗？

支持。数据存在 Room 本地数据库，网络恢复后自动同步。

---

## 数据管道

### 项目概述

这是一个用户行为数据 ETL 管道，每天凌晨从 API 拉取用户行为数据，清洗后写入分析数据库。
解决用户行为分析的数据供给问题。
给数据分析师、产品经理使用的分析数据源。

### 技术路线

用 Airflow 做任务调度，因为 DAG 定义清晰、重试机制完善。
用 Pandas 做数据转换，因为 DataFrame API 适合批量数据处理。
用 PostgreSQL 做目标数据库，因为分析查询性能好。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  数据源 API  │
│  (用户行为)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  Airflow    │────▶│  Pandas     │
│  (调度)     │     │  (转换)     │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  PostgreSQL │────▶│  BI 工具    │
│  (分析库)   │     │  (报表)     │
└─────────────┘     └─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Extractor  │────▶│  Cleaner    │────▶│  Loader     │
│  (数据抽取)  │     │  (清洗转换)  │     │  (数据写入)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心数据流

#### 场景 1：每日 ETL

```
Airflow 凌晨 2:00 触发 DAG
  → Extractor    (调用 API 拉取昨日数据)
  → Cleaner      (去重、类型转换、异常值处理)
  → Loader       (Upsert 写入 PostgreSQL)
  → 发送完成通知
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `dags/daily_etl.py` — DAG 定义
3. `src/transform/cleaner.py` — 核心清洗逻辑

### 快速上手

#### 环境准备

- Python 3.10+
- Airflow 2.8+
- PostgreSQL 16+

#### 启动项目

```bash
pip install -r requirements.txt
airflow db init
airflow webserver --port 8080
airflow scheduler
```

#### 验证

访问 `http://localhost:8080`，看到 DAG 列表。

### 常见场景

#### 场景：添加新的数据源

1. 在 `src/extract/` 下创建新 Extractor 类
2. 在 DAG 中添加对应的 Task
3. 在 `cleaner.py` 中添加清洗规则
4. 更新 `db_loader.py` 中的目标表映射

#### 场景：修改清洗规则

1. 打开 `src/transform/cleaner.py`
2. 修改对应的 `clean_*` 方法
3. 运行 `pytest tests/test_cleaner.py` 验证
4. 注意：修改历史数据需要重跑管道

### 设计决策解读

#### 为什么用 Pandas 而不是 Spark

数据量在百万级以内，单机 Pandas 处理足够，部署简单。
Spark 需要集群，运维成本高，小数据量下反而更慢。

代价是数据量增长到千万级时需要迁移，但目前够用。

#### 为什么 ETL 而不是 ELT

API 数据源格式不规范，需要先清洗再写入（ETL）。
如果数据源格式规范，ELT（先写入再用 SQL 清洗）更高效。

代价是清洗逻辑写在 Python 中，调试不如 SQL 直观。

### FAQ

#### Q: 失败了怎么办？

Airflow 自动重试 3 次，间隔 5 分钟。全部失败后发送告警邮件。

#### Q: 怎么重跑历史数据？

用 Airflow 的 Backfill 功能：`airflow dags backfill -s 2024-01-01 -e 2024-01-31 daily_etl`。

---

## ML/AI

### 项目概述

这是一个文本分类模型，将用户评论分为正面/负面/中性三类。
解决评论情感分析的自动化问题。
给内容审核团队、推荐系统使用。

### 技术路线

用 PyTorch 做训练框架，因为灵活、社区大。
用 Transformers 加载预训练 BERT，因为迁移学习效果好。
用 WandB 做实验追踪，因为参数管理方便。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  数据标注员  │
└──────┬──────┘
       │ 标注数据
       ▼
┌─────────────┐     ┌─────────────┐
│  训练脚本   │────▶│  WandB      │
│  (PyTorch)  │     │  (实验追踪)  │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  模型文件   │────▶│  推理服务   │
│  (Checkpoint)│     │  (FastAPI)  │
└─────────────┘     └─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Dataset    │────▶│  Model      │────▶│  Trainer    │
│  (数据加载)  │     │  (BERT)     │     │  (训练循环)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心数据流

#### 场景 1：模型训练

```
加载预处理数据集
  → DataLoader     (Batch 加载、Tokenize)
  → BERT 模型      (前向传播)
  → Loss 计算      (交叉熵)
  → 反向传播       (梯度更新)
  → 保存 Checkpoint
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `src/train.py` — 训练入口
3. `src/models/model.py` — 模型定义

### 快速上手

#### 环境准备

- Python 3.10+
- CUDA（GPU 训练需要）
- WandB 账号

#### 启动项目

```bash
pip install -r requirements.txt
python src/train.py --config configs/base.yaml
```

#### 验证

WandB 面板看到 Loss 曲线下降。

### 常见场景

#### 场景：调整超参数

1. 复制 `configs/base.yaml` 为新配置文件
2. 修改 learning_rate、batch_size 等参数
3. 运行训练，对比 WandB 面板中的指标
4. 选择最优配置

#### 场景：新增数据类别

1. 在 `data/raw/` 下添加新类别的标注数据
2. 修改 `src/data/dataset.py` 中的标签映射
3. 重新运行预处理脚本
4. 重新训练模型

#### 场景：部署推理服务

1. 导出模型：`python src/export.py --checkpoint best.pt`
2. 启动服务：`uvicorn src.serve:app --port 8000`
3. 测试：`curl -X POST http://localhost:8000/predict -d '{"text":"..."}` 

### 设计决策解读

#### 为什么用 BERT 而不是自训练 Embedding

BERT 在大规模语料上预训练过，迁移学习效果好，小数据集也能收敛。
自训练 Embedding 需要海量标注数据，效果不如预训练模型。

代价是模型体积大（~400MB），推理延迟较高，不适合边缘部署。

#### 为什么用 WandB 而不是 TensorBoard

WandB 云端存储实验记录，团队协作方便，支持超参数搜索。
TensorBoard 是本地的，多人协作需要共享日志目录。

代价是 WandB 有免费额度限制，大团队需要付费。

### FAQ

#### Q: 没有 GPU 能训练吗？

可以但很慢。建议用 Google Colab 或云 GPU。

#### Q: 怎么提升模型精度？

增加标注数据、调整学习率、尝试更大的预训练模型（如 RoBERTa）。

---

## 游戏

### 项目概述

这是一个 2D 平台跳跃游戏，玩家控制角色在不同关卡中跳跃、躲避障碍、收集道具。
解决休闲娱乐需求。
给 PC 和主机玩家使用。

### 技术路线

用 Unity 引擎，因为 2D 工具链成熟、跨平台部署方便。
用 C# 做脚本语言，因为 Unity 原生支持。
用 Tilemap 做地图编辑，因为关卡设计师可以直接在编辑器中画地图。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  玩家       │
│  (手柄/键鼠) │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐
│  Unity 引擎 │────▶│  物理系统   │
│  (游戏循环)  │     │  (碰撞/重力) │
└──────┬──────┘     └─────────────┘
       │
       ▼
┌─────────────┐
│  渲染管线   │
│  (2D Sprite)│
└─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Input      │────▶│  Controller │────▶│  Physics    │
│  (输入检测)  │     │  (角色控制)  │     │  (物理计算)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心数据流

#### 场景 1：玩家跳跃

```
玩家按下空格键
  → Input System    (检测按键)
  → PlayerController (检查地面、施加力)
  → Rigidbody2D     (物理计算)
  → Animator        (播放跳跃动画)
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `Assets/Scripts/GameManager.cs` — 游戏主循环
3. `Assets/Scripts/Player/PlayerController.cs` — 玩家控制

### 快速上手

#### 环境准备

- Unity 2022+

#### 启动项目

```bash
# 用 Unity Hub 打开项目
# 点击 Play
```

#### 验证

游戏窗口启动，角色可以移动和跳跃。

### 常见场景

#### 场景：添加新敌人

1. 在 `Assets/Prefabs/Enemies/` 下创建新预制体
2. 创建 `NewEnemy.cs` 脚本，继承 `EnemyBase`
3. 实现行为逻辑（巡逻、攻击）
4. 在场景中放置预制体

#### 场景：添加新关卡

1. 在 `Assets/Scenes/` 下复制现有场景
2. 用 Tilemap 编辑地图布局
3. 设置出生点和终点
4. 在 `GameManager` 中注册新关卡

#### 场景：调整游戏手感

1. 修改 `PlayerController.cs` 中的 `moveSpeed`、`jumpForce`
2. 调整 `Rigidbody2D` 的 gravity scale
3. 修改 `Animator` 中的动画过渡时间
4. 反复测试直到手感舒适

### 设计决策解读

#### 为什么用 Rigidbody2D 而不是手动移动

Rigidbody2D 提供物理模拟（重力、碰撞、摩擦），手感自然。
手动 Transform.Translate 需要自己处理所有物理，容易穿墙。

代价是物理参数调优需要反复试验，不如手动移动精确可控。

#### 为什么用 Tilemap 而不是手绘背景

Tilemap 复用瓦片资源，内存占用小，关卡编辑效率高。
手绘背景每个关卡都是独立图片，美术工作量大。

代价是视觉效果有重复感，需要精心设计瓦片来避免。

### FAQ

#### Q: 怎么加新关卡？

在 `Assets/Scenes/` 下复制现有场景，用 Tilemap 编辑地图。

#### Q: 怎么添加音效？

将音频文件放入 `Assets/Audio/`，在 `AudioManager.cs` 中注册，通过 `AudioManager.Play("soundName")` 播放。

---

## 桌面应用 (Electron)

### 项目概述

这是一个 Markdown 编辑器，支持实时预览、语法高亮、插件扩展。
解决写作和笔记管理需求。
给开发者和写作者使用的桌面应用。

### 技术路线

用 Electron 做桌面框架，因为 Web 技术栈开发效率高、跨平台。
用 SQLite 做本地存储，因为文件数据库无需安装。
用 React + Tailwind 做界面，因为组件化开发效率高。

### 核心数据流

#### 场景 1：编辑并保存文档

```
用户输入 Markdown
  → 渲染进程      (实时预览)
  → IPC 通信      (发送保存请求)
  → 主进程        (写入 SQLite)
  → 保存成功提示
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `src/main/index.ts` — 主进程入口
3. `src/renderer/App.tsx` — 界面入口

### 快速上手

#### 环境准备

- Node.js 18+

#### 启动项目

```bash
npm install
npm run dev
```

#### 验证

应用窗口启动，可以输入 Markdown 并看到预览。

### 常见场景

#### 场景：添加新功能面板

1. 在 `src/renderer/components/` 下创建新组件
2. 在主布局中注册面板
3. 如需持久化，通过 IPC 调用主进程写入 SQLite
4. 测试窗口缩放和面板响应

#### 场景：修改快捷键

1. 打开 `src/main/menu.ts`
2. 修改对应菜单项的 `accelerator` 字段
3. 如需全局快捷键，在 `src/main/shortcuts.ts` 中注册
4. 重启应用验证

### 设计决策解读

#### 为什么用 SQLite 而不是文件系统直接存储

SQLite 支持全文搜索（FTS5）、事务、并发读，适合文档管理。
文件系统存储需要自己建索引，搜索性能差。

代价是数据库文件可能损坏（断电场景），需要定期备份。

### FAQ

#### Q: 插件怎么开发？

参考 `docs/plugin-api.md`，插件是独立的 npm 包，通过 IPC 和主进程通信。

#### Q: 支持多窗口吗？

支持。每个窗口是独立的渲染进程，共享同一个主进程和 SQLite 数据库。

---

## 编译器

### 项目概述

这是一个玩具语言编译器，支持函数定义、模式匹配、类型推断。
解决学习编译器原理的教学需求。
给编程语言爱好者使用。

### 技术路线

用 Rust 实现，因为内存安全、模式匹配语法适合写编译器。
用 LLVM 做后端代码生成，因为优化成熟、支持多平台。

### 核心数据流

#### 场景 1：编译源码

```
源码字符串
  → Lexer         (词法分析 → Token 流)
  → Parser        (语法分析 → AST)
  → TypeChecker   (类型检查)
  → CodeGen       (AST → LLVM IR)
  → 输出可执行文件
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `src/main.rs` — 入口
3. `src/lexer/mod.rs` — 词法分析

### 快速上手

#### 环境准备

- Rust 1.75+
- LLVM 17

#### 启动项目

```bash
cargo build
echo 'fn main() { print("hello") }' > test.lang
cargo run -- test.lang
./output
```

#### 验证

输出 `hello`。

### 常见场景

#### 场景：添加新关键字

1. 在 `src/lexer/mod.rs` 的 `Token` 枚举中添加新变体
2. 在 `next_token()` 中添加识别逻辑
3. 在 `src/parser/mod.rs` 中添加解析规则
4. 在 `src/codegen/` 中添加代码生成逻辑

#### 场景：添加新的类型

1. 在 `src/types/` 中定义新类型
2. 在 `TypeChecker` 中添加类型推断规则
3. 在 `CodeGen` 中添加 LLVM IR 生成
4. 添加测试用例验证

#### 场景：调试编译错误

1. 确认是词法、语法、类型检查哪个阶段报错
2. 查看错误信息中的行号和列号
3. 用 `--verbose` 参数查看详细的 Token 流和 AST
4. 常见问题：括号不匹配、类型不兼容

### 设计决策解读

#### 为什么用 LLVM 而不是自写代码生成

LLVM 有成熟的优化器（常量折叠、循环优化），支持多平台（x86、ARM）。
自写代码生成工作量巨大，且优化效果远不如 LLVM。

代价是 LLVM 依赖庞大（~1GB），编译时间长，学习曲线陡。

### FAQ

#### Q: 支持哪些语言特性？

目前支持：函数、整数/浮点/字符串、if-else、模式匹配。详见 `docs/syntax.md`。

#### Q: 怎么添加新的运算符？

在 Lexer 中添加 Token，在 Parser 中添加优先级规则，在 CodeGen 中添加 LLVM IR 生成。

---

## 库/SDK (npm)

### 项目概述

这是一个类型安全的 HTTP 客户端 SDK，封装 REST API 调用。
解决 API 调用的类型安全和错误处理问题。
给使用该 API 的前端/后端开发者使用。

### 技术路线

用 TypeScript 实现，因为类型推断是核心卖点。
用 Axios 做 HTTP 底层，因为拦截器机制适合统一处理认证和错误。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  开发者应用  │
│  (使用者)   │
└──────┬──────┘
       │ import sdk
       ▼
┌─────────────┐     ┌─────────────┐
│  本 SDK     │────▶│  REST API   │
│  (TypeScript)│     │  (后端服务)  │
└─────────────┘     └─────────────┘
```

#### 服务/模块划分（Container）

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Public API │────▶│  Client     │────▶│  Axios      │
│  (用户接口)  │     │  (请求构造)  │     │  (HTTP 底层) │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心数据流

#### 场景 1：发起 API 请求

```
用户调用 client.get('/users')
  → 请求拦截器    (添加 Authorization header)
  → Axios        (发送 HTTP 请求)
  → 响应拦截器    (处理 401/429/5xx)
  → 返回类型安全的 Response<T>
```

### 推荐阅读顺序

#### 快速理解（15 分钟）

1. 本文件 — 建立全貌
2. `src/index.ts` — 公共 API 导出
3. `src/client/index.ts` — 核心客户端

### 快速上手

#### 环境准备

- Node.js 18+

#### 启动项目

```bash
npm install
npm test
```

#### 验证

测试全部通过。

### 常见场景

#### 场景：添加新的 API 方法

1. 在 `src/types/` 中定义请求/响应类型
2. 在 `src/client/index.ts` 中添加方法
3. 在 `src/index.ts` 中导出
4. 添加单元测试

#### 场景：处理新的错误码

1. 在 `src/errors/` 中定义新的错误类
2. 在响应拦截器中添加判断逻辑
3. 确保错误类型导出给用户使用

### 设计决策解读

#### 为什么用 Axios 而不是原生 fetch

Axios 有拦截器机制，适合统一处理认证、重试、错误。
原生 fetch 需要自己封装这些逻辑，代码量差不多。

代价是多了一个依赖（~13KB gzip），对纯前端项目有体积敏感。

### FAQ

#### Q: 怎么处理 Token 过期？

SDK 自动处理 401：用 refreshToken 换新 token 后重试原请求。

#### Q: 支持 Node.js 和浏览器吗？

支持。浏览器环境用 XMLHttpRequest，Node.js 环境用 http 模块，Axios 自动切换。

---

## 基础设施 (Terraform + K8s)

### 项目概述

这套 Terraform 模块管理 AWS EKS 集群、RDS 数据库、Redis 缓存等基础设施。
解决云资源的版本化管理和自动化部署问题。
给 DevOps 工程师、后端开发者使用。

### 技术路线

用 Terraform 做 IaC，因为声明式、状态管理成熟。
用 EKS 做容器编排，因为 AWS 生态集成好。
用 Helm 做应用部署，因为模板化、版本管理方便。

### 核心数据流

#### 场景 1：部署新环境

```
terraform workspace select staging
  → terraform plan   (预览变更)
  → terraform apply  (创建 VPC → EKS → RDS → Helm 部署)
  → kubectl get pods (验证服务运行)
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `main.tf` — Provider 定义
3. `modules/eks/main.tf` — 核心集群模块

### 快速上手

#### 环境准备

- Terraform >= 1.7
- kubectl
- AWS CLI + 凭证配置

#### 启动项目

```bash
cd environments/dev
terraform init
terraform plan
terraform apply
```

#### 验证

`kubectl get nodes` 显示节点就绪。

### 常见场景

#### 场景：添加新的 Terraform 模块

1. 在 `modules/` 下创建新目录
2. 定义 `variables.tf`、`main.tf`、`outputs.tf`
3. 在 `environments/*/main.tf` 中引用模块
4. 运行 `terraform plan` 验证

#### 场景：升级 EKS 版本

1. 修改 `modules/eks/main.tf` 中的 `cluster_version`
2. 先升级控制平面：`terraform apply -target=module.eks`
3. 再升级节点组：`terraform apply`
4. 验证：`kubectl get nodes` 确认版本

#### 场景：排查 Pod 启动失败

1. `kubectl describe pod <pod-name>` 查看事件
2. `kubectl logs <pod-name>` 查看应用日志
3. 检查资源限制：`kubectl top pod`
4. 常见问题：镜像拉取失败、资源不足、探针失败

### 设计决策解读

#### 为什么用 Terraform 而不是 CloudFormation

Terraform 跨云支持（AWS、GCP、Azure），HCL 语法比 CloudFormation 的 YAML/JSON 简洁。
CloudFormation 只支持 AWS，但和 AWS 集成更深。

代价是 Terraform 状态文件需要远程存储（S3 + DynamoDB Lock），有状态管理风险。

### FAQ

#### Q: 状态文件丢失怎么办？

用 `terraform import` 重新导入已有资源。预防措施：S3 启用版本控制 + DynamoDB Lock 防止并发写入。

#### Q: 怎么回滚部署？

Terraform 没有内置回滚。用 Git 回退代码到上一个版本，重新 `terraform apply`。

---

## 微服务

### 项目概述

这是一个电商平台微服务架构，包含用户服务、订单服务、库存服务、支付服务。
解决单体应用扩展性差、部署耦合的问题。
给大型电商团队使用。

### 技术路线

用 Express + TypeScript 做各服务，因为团队技术栈统一。
用 Kafka 做事件驱动，因为服务间解耦、支持高吞吐。
用 Kubernetes 做编排，因为自动扩缩容、服务发现。
用 Istio 做服务网格，因为流量管理、可观测性。

### 架构全景

#### 系统边界（Context）

```
┌─────────────┐
│  前端/移动端  │
└──────┬──────┘
       │
       ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  API 网关    │────▶│  用户服务    │     │  订单服务    │
│  (路由/认证)  │────▶│  (认证/用户) │     │  (订单/支付) │
└─────────────┘     └──────┬──────┘     └──────┬──────┘
                           │                   │
                           └─────────┬─────────┘
                                     ▼
                              ┌─────────────┐
                              │  Kafka      │
                              │  (事件总线)  │
                              └─────────────┘
```

### 核心数据流

#### 场景 1：用户下单

```
用户提交订单
  → API 网关       (认证、路由)
  → 订单服务       (创建订单)
  → Kafka          (发布 OrderCreated 事件)
  → 库存服务       (消费事件、扣减库存)
  → 支付服务       (消费事件、发起支付)
  → 订单服务       (消费支付结果、更新状态)
```

### 推荐阅读顺序

#### 快速理解（30 分钟）

1. 本文件 — 建立全貌
2. `docker-compose.yaml` — 本地开发环境
3. `gateway/src/main.ts` — API 网关

#### 深入理解（2 小时）

4. `services/order-service/` — 订单服务
5. `services/user-service/` — 用户服务
6. `shared/events/` — 事件定义

### 快速上手

#### 环境准备

- Docker + Docker Compose
- kubectl（K8s 部署需要）

#### 启动项目

```bash
docker compose up -d
# 等待所有服务启动
curl http://localhost:3000/health
```

#### 验证

所有服务健康检查通过。

### 常见场景

#### 场景：添加新服务

1. 复制现有服务目录
2. 修改服务名和端口
3. 在 `docker-compose.yaml` 中添加
4. 定义事件接口（如需要）

### 设计决策解读

#### 为什么用 Kafka 而不是 RabbitMQ

Kafka 吞吐量高、支持事件回溯、适合大数据量场景。
RabbitMQ 更适合低延迟的任务队列。

代价是 Kafka 运维复杂度更高，需要 ZooKeeper（或 KRaft）。

### FAQ

#### Q: 本地开发要启动所有服务吗？

不需要。`docker compose up user-service order-service` 只启动需要的服务。
