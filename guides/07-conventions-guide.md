# 07-conventions.md 填写指南

> 本指南展示各项目类型的命名规范、文件组织、设计模式和错误处理示例。
> 对应模板：`template/07-conventions.md`

---

## TypeScript / JavaScript — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 文件名 | kebab-case | `user-service.ts` |
| 类名 | PascalCase | `UserService` |
| 函数/方法 | camelCase | `findByEmail()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 接口 | PascalCase，无 `I` 前缀 | `UserRepository` |
| 数据库表 | snake_case，复数 | `users`, `order_items` |
| 数据库字段 | snake_case | `created_at`, `user_id` |
| API 路径 | kebab-case | `/api/user-profiles` |
| 环境变量 | UPPER_SNAKE_CASE | `DATABASE_URL` |

---

## C (嵌入式) — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 源文件 | snake_case | `sensor_task.c` |
| 结构体 | snake_case + `_t` 后缀 | `sensor_data_t` |
| 全局变量 | `g_` 前缀 | `g_sensor_buffer` |
| 静态变量 | `s_` 前缀 | `s_i2c_handle` |
| 函数 | 模块前缀 + 动词 | `SHT31_Measure()` |
| 宏 | UPPER_SNAKE_CASE | `SHT31_I2C_ADDR` |
| 枚举 | 模块前缀 | `SENSOR_STATE_IDLE` |

---

## Go — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 文件名 | snake_case | `user_service.go` |
| 结构体 | PascalCase | `UserService` |
| 导出函数 | PascalCase | `FindByEmail()` |
| 非导出函数 | camelCase | `validateEmail()` |
| 接口 | PascalCase + `er` 后缀 | `Reader`, `Writer` |
| 常量 | PascalCase | `MaxRetryCount` |
| 包名 | lowercase, 单词 | `auth`, `httputil` |

---

## Python — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 文件名 | snake_case | `user_service.py` |
| 类名 | PascalCase | `UserService` |
| 函数/方法 | snake_case | `find_by_email()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 私有成员 | `_` 前缀 | `_internal_method()` |
| 类型变量 | PascalCase | `T`, `UserType` |
| 包名 | snake_case | `my_package` |

---

## Rust — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 文件名 | snake_case | `user_service.rs` |
| 结构体/枚举 | PascalCase | `UserService` |
| 函数/方法 | snake_case | `find_by_email()` |
| 常量 | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 模块 | snake_case | `user_service` |
| Trait | PascalCase | `Drawable`, `Clone` |

---

## Swift — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 文件名 | PascalCase（与类型同名） | `UserService.swift` |
| 类/结构体 | PascalCase | `UserService` |
| 方法 | camelCase | `findByEmail()` |
| 常量 | camelCase | `maxRetryCount` |
| 协议 | PascalCase + `able`/`ing` | `Codable`, `Identifiable` |

---

## Kotlin — 命名规范

| 元素 | 规则 | 示例 |
|------|------|------|
| 文件名 | PascalCase（可含多个类） | `UserService.kt` |
| 类/接口 | PascalCase | `UserService` |
| 方法 | camelCase | `findByEmail()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 包名 | lowercase | `com.example.auth` |

---

## Web 项目 — 文件组织

```
src/
├── api/                # 路由层（只做参数校验和调用 service）
│   └── {domain}/
│       ├── routes.ts
│       └── validators.ts
├── services/           # 业务逻辑层
│   └── {domain}.ts
├── repositories/       # 数据访问层
│   └── {domain}.ts
├── models/             # 数据模型/类型定义
│   └── {domain}.ts
├── middleware/          # 中间件
│   └── {name}.ts
├── utils/              # 工具函数（纯函数，无副作用）
│   └── {name}.ts
└── config/             # 配置
    └── index.ts
```

---

## 嵌入式 — 文件组织

```
src/
├── main.c              # 入口
├── tasks/              # FreeRTOS 任务
│   └── {task_name}_task.c
├── filters/            # 数据处理（滤波、校准）
│   └── {name}.c
├── buffer/             # 缓冲区管理
│   └── {name}.c
└── utils/              # 工具函数
    └── {name}.c

drivers/                # 硬件驱动
├── {device}.c
└── {device}.h

config/                 # 板级配置
├── pin_config.h
└── FreeRTOSConfig.h
```

### 嵌入式 — AUTOSAR 文件组织

```
src/
├── swc/                    # 应用层 SWC（手写）
│   ├── DiagManager/
│   │   ├── DiagManager.c
│   │   └── DiagManager.h
│   ├── LightControl/
│   │   ├── LightControl.c
│   │   └── LightControl.h
│   └── SensorAcq/
│       ├── SensorAcq.c
│       └── SensorAcq.h

generated/                  # 配置工具生成（不要手动修改）
├── Rte/                    # RTE 接口代码
│   ├── Rte.h
│   ├── Rte_<SWC>.h        # 每个 SWC 的专用头文件
│   └── Rte.c
├── Com/                    # COM 信号配置
│   ├── Com_Cfg.h
│   └── Com.c
├── Dcm/                    # DCM 诊断配置
│   ├── Dcm_Cfg.h
│   └── Dcm.c
├── Dem/                    # DEM 事件配置
│   ├── Dem_Cfg.h
│   └── Dem.c
├── NvM/                    # NVM 块配置
│   ├── NvM_Cfg.h
│   └── NvM.c
└── EcuM/                   # ECU 状态管理配置
    ├── EcuM_Cfg.h
    └── EcuM.c

bsw/                        # BSW 模块源码（供应商提供，勿修改）
├── Com/
├── Dcm/
├── Dem/
├── NvM/
├── EcuM/
└── CanSM/

mcal/                       # MCAL 驱动（芯片厂商提供）
├── Can/
├── Spi/
├── Adc/
├── Dio/
└── generated/              # MCAL 配置生成代码
```

**文件组织关键规则**：
- `generated/` 目录下所有文件由工具生成，**绝对不要手动修改**（每次重新生成会覆盖）
- `src/swc/` 下的文件是手写的应用逻辑
- BSW 模块版本升级时，整个 `bsw/` 目录替换
- MCAL 配置通过 EB tresos 或芯片厂商工具生成

---

## Go — 文件组织

```
cmd/
└── app/
    └── main.go         # 入口
internal/
├── handler/            # HTTP 处理器
├── service/            # 业务逻辑
├── repository/         # 数据访问
├── model/              # 数据模型
├── middleware/          # 中间件
└── config/             # 配置
pkg/                    # 可复用的公共库
└── utils/
```

---

## Python — 文件组织

```
src/
├── __init__.py
├── main.py             # 入口
├── api/                # API 路由
├── services/           # 业务逻辑
├── repositories/       # 数据访问
├── models/             # 数据模型
├── utils/              # 工具函数
└── config.py           # 配置
tests/
├── test_api/
├── test_services/
└── conftest.py
```

---

## iOS — 文件组织

```
Sources/
├── App/                # App 入口
├── Features/           # 功能模块（按功能划分）
│   ├── Home/
│   │   ├── HomeView.swift
│   │   ├── HomeViewModel.swift
│   │   └── HomeRouter.swift
│   └── Profile/
├── Core/               # 核心能力
│   ├── Network/
│   ├── Persistence/
│   └── UI/
└── Resources/          # 资源文件
```

---

## Android — 文件组织

```
app/src/main/
├── java/com/example/app/
│   ├── di/             # 依赖注入
│   ├── data/           # 数据层
│   │   ├── local/      # Room DAO
│   │   ├── remote/     # Retrofit API
│   │   └── repository/
│   ├── domain/         # 领域层
│   │   ├── model/
│   │   └── usecase/
│   └── presentation/   # 表现层
│       ├── home/
│       └── profile/
└── res/                # 资源文件
```

---

## 微服务 — 文件组织

```
services/
├── auth/               # 认证服务
│   ├── src/
│   ├── tests/
│   └── Dockerfile
├── order/              # 订单服务
│   ├── src/
│   ├── tests/
│   └── Dockerfile
└── payment/            # 支付服务
    ├── src/
    ├── tests/
    └── Dockerfile
gateway/                # API 网关
├── src/
└── Dockerfile
infra/                  # 基础设施
├── terraform/
├── k8s/
└── helm/
```

---

## Web 后端 — 设计模式

### 依赖注入

```typescript
// 构造函数注入，不使用全局单例
class AuthService {
  constructor(
    private userRepo: UserRepository,
    private jwtService: JwtService,
    private emailService: EmailService,
  ) {}
}
```

### Repository 模式

```typescript
// 每个实体一个 Repository，封装数据访问
interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(data: CreateUserInput): Promise<User>;
  update(id: string, data: UpdateUserInput): Promise<User>;
}
```

---

## 嵌入式 — 设计模式

### HAL 抽象

```c
// 通过 HAL 句柄操作硬件，不直接操作寄存器
I2C_HandleTypeDef hi2c1;
HAL_StatusTypeDef result = HAL_I2C_Master_Transmit(&hi2c1, addr, data, len, timeout);
```

### 状态机模式

```c
// 枚举定义状态，switch-case 处理转换
typedef enum { STATE_IDLE, STATE_MEASURING, STATE_SENDING } sensor_state_t;

void Sensor_Task(void *pvParameters) {
    sensor_state_t state = STATE_IDLE;
    for (;;) {
        switch (state) {
            case STATE_IDLE:    /* ... */ break;
            case STATE_MEASURING: /* ... */ break;
            case STATE_SENDING: /* ... */ break;
        }
    }
}
```

---

## Web 后端 — 错误处理

```typescript
// 自定义错误类
class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string,
  ) {
    super(message);
  }
}

// 使用方式
throw new AppError(409, 'EMAIL_EXISTS', 'Email already registered');

// 全局错误处理中间件统一捕获
```

**规则**：
- 业务逻辑抛 `AppError`，中间件统一转为 HTTP 响应
- 不在 service 层直接操作 `res` 对象
- 数据库错误、外部服务错误统一包装为 `AppError`
- 所有异步操作使用 `try/catch`，不使用 `.catch()` 链式调用

---

## 嵌入式 — 错误处理

```c
// 统一错误码
typedef enum {
    ERR_OK = 0,
    ERR_TIMEOUT = -1,
    ERR_CRC = -2,
    ERR_NACK = -3,
    ERR_BUSY = -4,
} error_code_t;

// 使用方式
error_code_t SHT31_Measure(I2C_HandleTypeDef *hi2c, float *temp, float *hum) {
    if (HAL_I2C_Master_Transmit(...) != HAL_OK) return ERR_NACK;
    if (CRC_Check(...) != 0) return ERR_CRC;
    // ...
    return ERR_OK;
}
```

**规则**：
- 所有驱动函数返回 `error_code_t`
- 不使用 `assert` 处理运行时错误（仅用于开发调试）
- 错误日志通过 UART 输出，格式：`[ERROR] module: message`
- 致命错误触发系统复位前，先保存错误日志到 Flash

---

## 注释规范（通用）

**原则**：代码自解释，只在以下情况写注释：
- **为什么**（而非做了什么）：设计决策、权衡取舍
- **坑**：非直觉行为、硬件限制、workaround
- **约束**：性能要求、协议规范、标准引用

```typescript
// ✅ 好的注释
// bcrypt rounds=10 是经过基准测试的选择：安全性足够，单次哈希 < 100ms
const hash = await bcrypt.hash(password, 10);

// ❌ 坏的注释
// 对密码进行哈希
const hash = await bcrypt.hash(password, 10);
```

```c
// ✅ 好的注释
// Datasheet Section 4.3: 测量命令后至少等待 15ms
HAL_Delay(15);

// ❌ 坏的注释
// 延时 15 毫秒
HAL_Delay(15);
```
