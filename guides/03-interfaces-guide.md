# 03-interfaces.md 填写指南

> 本指南展示各项目类型的接口定义示例。
> 对应模板：`template/03-interfaces.md`

---

## Web 后端 — HTTP API

### `POST /api/auth/register`

- **用途**：用户注册
- **鉴权**：无
- **请求体**：
  ```json
  {
    "email": "string (required, valid email)",
    "password": "string (required, min 8 chars)",
    "name": "string (required, max 50 chars)"
  }
  ```
- **响应** `201`：
  ```json
  {
    "user": { "id": "string", "email": "string", "name": "string" },
    "token": "string (JWT)"
  }
  ```
- **错误**：
  - `409`：邮箱已注册
  - `422`：参数校验失败
- **实现文件**：`src/api/auth/register.ts`

### `POST /api/auth/login`

- **用途**：用户登录
- **鉴权**：无
- **请求体**：
  ```json
  {
    "email": "string",
    "password": "string"
  }
  ```
- **响应** `200`：
  ```json
  {
    "token": "string (JWT)",
    "refreshToken": "string"
  }
  ```
- **错误**：
  - `401`：邮箱或密码错误
- **实现文件**：`src/api/auth/login.ts`

---

## Web 后端 — gRPC API

```protobuf
syntax = "proto3";
package myapp.v1;

service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message GetUserRequest {
  string id = 1;
}

message User {
  string id = 1;
  string email = 2;
  string name = 3;
}
```

---

## Web 后端 — GraphQL API

```graphql
type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): UserConnection!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}

type User {
  id: ID!
  email: String!
  name: String!
  orders: [Order!]!
}
```

---

## CLI 工具 — 命令定义

### `mycli init`

- **用途**：初始化项目配置
- **参数**：
  - `--name <string>` (required) — 项目名称
  - `--template <string>` (optional) — 模板名称，默认 "default"
- **行为**：在当前目录创建 `.mycli/config.yaml`
- **实现文件**：`src/commands/init.ts`

### `mycli build`

- **用途**：构建项目
- **参数**：
  - `--target <string>` (optional) — 目标平台
  - `--release` (flag) — 发布构建
- **标准输出**：构建进度和结果
- **退出码**：0=成功，1=失败
- **实现文件**：`src/commands/build.go`

---

## 嵌入式 — 硬件接口

### I2C 总线

| 设备 | 地址 | 用途 | 引脚 |
|------|------|------|------|
| SHT31 | 0x44 | 温湿度传感器 | SCL=PB6, SDA=PB7 |
| BMP280 | 0x76 | 气压传感器 | SCL=PB6, SDA=PB7 |

### SPI 总线

| 设备 | CS 引脚 | 时钟 | 用途 |
|------|---------|------|------|
| LoRa SX1276 | PA4 | SPI1 | 无线通信 |

### UART

| 端口 | 波特率 | 用途 |
|------|--------|------|
| USART2 | 115200 | 调试日志 |

---

## 嵌入式 — 模块间接口 (C)

### 传感器任务 → 驱动

```c
// 传感器任务调用 SHT31 驱动
error_code_t SHT31_Measure(I2C_HandleTypeDef *hi2c, float *temp, float *hum);
error_code_t SHT31_Init(I2C_HandleTypeDef *hi2c);
```

### 通信任务 → 驱动

```c
// 通信任务调用 SX1276 驱动
error_code_t SX1276_Send(SPI_HandleTypeDef *hspi, uint8_t *data, uint8_t len);
error_code_t SX1276_Receive(SPI_HandleTypeDef *hspi, uint8_t *buf, uint8_t *len);
```

---

## Web 后端 — 模块间接口 (TypeScript)

### 认证 → 用户

```typescript
interface UserService {
  findByEmail(email: string): Promise<User | null>;
  create(data: CreateUserInput): Promise<User>;
  verifyPassword(user: User, password: string): Promise<boolean>;
}
```

### 支付 → 订单

```typescript
interface OrderService {
  createOrder(userId: string, items: OrderItem[]): Promise<Order>;
  markPaid(orderId: string, paymentId: string): Promise<void>;
  markFailed(orderId: string, reason: string): Promise<void>;
}
```

---

## iOS — 模块间接口 (Swift)

### ViewModel → Repository

```swift
protocol UserRepositoryProtocol {
    func fetchUser(id: String) async throws -> User
    func updateUser(_ user: User) async throws -> User
    func deleteUser(id: String) async throws
}
```

---

## Android — 模块间接口 (Kotlin)

### ViewModel → Repository

```kotlin
interface UserRepository {
    suspend fun getUser(id: String): Result<User>
    suspend fun updateUser(user: User): Result<User>
    suspend fun deleteUser(id: String): Result<Unit>
}
```

---

## Go — 模块间接口

### Service → Repository

```go
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*User, error)
    Create(ctx context.Context, user *User) error
    Update(ctx context.Context, user *User) error
}
```

---

## Python — 模块间接口

### Service → Repository

```python
from abc import ABC, abstractmethod

class UserRepository(ABC):
    @abstractmethod
    async def find_by_id(self, id: str) -> User | None: ...
    @abstractmethod
    async def create(self, user: User) -> User: ...
```

---

## Web 后端 — 事件定义

| 事件名 | 触发时机 | 消费者 | 数据 |
|--------|---------|--------|------|
| `user.registered` | 用户注册成功 | 通知模块 | `{ userId, email }` |
| `order.paid` | 订单支付成功 | 库存模块、通知模块 | `{ orderId, userId }` |

---

## 嵌入式 — 中断定义

| 中断源 | 优先级 | 处理函数 | 行为 |
|--------|--------|---------|------|
| EXTI1 (LORA_DIO0) | 高 | `LORA_IRQHandler()` | 读取接收数据 |
| TIM2 | 中 | `TIM2_IRQHandler()` | 定时采集触发 |
| I2C1_ER | 低 | `I2C1_ErrorHandler()` | 记录错误、重置总线 |

---

## 嵌入式 — AUTOSAR CAN 信号矩阵

| 信号名 | CAN ID | 周期 | 方向 | 长度 | 说明 |
|--------|--------|------|------|------|------|
| `VehicleSpeed` | 0x1A0 | 10ms | 接收 | 16bit | 车速信号，0.01 km/bit |
| `EngineRPM` | 0x1A0 | 10ms | 接收 | 16bit | 发动机转速，0.25 rpm/bit |
| `LightCmd` | 0x2B0 | 100ms | 接收 | 8bit | 灯光控制命令 |
| `DiagRequest` | 0x7E0 | 事件型 | 接收 | 64bit | UDS 诊断请求（CAN-FD） |
| `DiagResponse` | 0x7E8 | 事件型 | 发送 | 64bit | UDS 诊断响应（CAN-FD） |
| `LightStatus` | 0x3C0 | 100ms | 发送 | 8bit | 灯光状态反馈 |
| `DTC_Status` | 0x3C1 | 1000ms | 发送 | 16bit | DTC 状态字节 |

---

## 嵌入式 — AUTOSAR UDS 诊断服务

| 服务 ID | 服务名 | 用途 | 处理模块 |
|---------|--------|------|----------|
| 0x10 | DiagnosticSessionControl | 切换诊断会话（Default/Extended/Programming） | DCM |
| 0x22 | ReadDataByIdentifier | 读取 DID（软件版本、硬件号等） | DCM + SWC |
| 0x2E | WriteDataByIdentifier | 写入 DID（标定参数、配置数据） | DCM + NVM |
| 0x27 | SecurityAccess | 安全解锁（种子-密钥算法） | DCM |
| 0x31 | RoutineControl | 执行例程（刷写检查、自检） | DCM + SWC |
| 0x14 | ClearDiagnosticInformation | 清除 DTC | DEM |
| 0x19 | ReadDTCInformation | 读取 DTC 及快照数据 | DEM |
| 0x28 | CommunicationControl | 控制通信收发 | COM |

---

## 嵌入式 — AUTOSAR RTE Sender-Receiver 接口

```c
/* RTE 提供的 SWC 间通信接口（自动生成，勿手改） */

// SensorAcq SWC → LightControl SWC：车速信号
Std_ReturnType Rte_Read_RPort_VehicleSpeed_VehicleSpeed(uint16 *data);

// LightControl SWC → COM：灯光状态反馈
Std_ReturnType Rte_Write_PPort_LightStatus_LightStatus(uint8 data);

// DiagManager SWC 读取 DCM 提供的诊断请求
Std_ReturnType Rte_Read_RPort_DiagRequest_DiagRequest(DiagRequestType *data);

// DiagManager SWC 写入诊断响应到 DCM
Std_ReturnType Rte_Write_PPort_DiagResponse_DiagResponse(const DiagResponseType *data);

// NVM 块读写接口
Std_ReturnType Rte_Call_RPort_NvmBlock_Read(NvmBlockType *data);
Std_ReturnType Rte_Call_RPort_NvmBlock_Write(const NvmBlockType *data);
```

**RTE 接口关键规则**：
- 所有 RTE 接口由配置工具根据 ARXML 自动生成
- SWC 只能调用 RTE API，不能直接访问 BSW 或 MCAL
- Sender-Receiver 是异步的（读取上次缓存值），Client-Server 是同步的
- 接口命名格式：`Rte_Read/Write_RPort/PPort_{SWC名}_{信号名}`

---

## 游戏 — 事件定义

| 事件名 | 触发时机 | 消费者 | 数据 |
|--------|---------|--------|------|
| `player.died` | 玩家生命值归零 | UI、Audio、GameLogic | `{ playerId, position, killerId }` |
| `item.picked` | 拾取道具 | Inventory、UI | `{ itemId, playerId }` |
| `level.completed` | 通关 | Progression、UI | `{ levelId, score, time }` |

---

## 数据管道 — 事件定义

| 事件名 | 触发时机 | 消费者 | 数据 |
|--------|---------|--------|------|
| `pipeline.started` | DAG 开始执行 | Monitor | `{ dagId, runId }` |
| `task.completed` | 单个任务完成 | Scheduler | `{ taskId, duration, rows }` |
| `pipeline.failed` | DAG 失败 | Alert、Monitor | `{ dagId, error, retryCount }` |

---

## iOS/Android — 导航事件

| 事件名 | 触发时机 | 消费者 | 数据 |
|--------|---------|--------|------|
| `navigation.detail` | 点击列表项 | Router | `{ screen, params }` |
| `auth.required` | Token 过期 | Auth、Router | `{ returnUrl }` |
