# 04-data.md 填写指南

> 本指南展示各项目类型的数据结构和协议示例。
> 对应模板：`template/04-data.md`

---

## Web 后端 — TypeScript 数据结构

```typescript
interface User {
  id: string;           // UUID v4
  email: string;        // 唯一，小写
  name: string;         // 1-50 字符
  passwordHash: string; // bcrypt, 10 rounds
  role: 'user' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}

interface Order {
  id: string;
  userId: string;       // 外键 → User.id
  status: 'pending' | 'paid' | 'shipped' | 'cancelled';
  items: OrderItem[];
  totalCents: number;   // 以分为单位，避免浮点精度问题
  paidAt: Date | null;
  createdAt: Date;
}

interface OrderItem {
  productId: string;
  quantity: number;
  priceCents: number;   // 下单时快照价格
}
```

---

## Go 数据结构

```go
type User struct {
    ID        string    `json:"id" db:"id"`
    Email     string    `json:"email" db:"email"`
    Name      string    `json:"name" db:"name"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
}
```

---

## Python 数据结构

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: str
    email: str
    name: str
    created_at: datetime
```

---

## Rust 数据结构

```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct User {
    pub id: String,
    pub email: String,
    pub name: String,
    pub created_at: DateTime<Utc>,
}
```

---

## Swift 数据结构

```swift
struct User: Codable, Identifiable {
    let id: String
    let email: String
    let name: String
    let createdAt: Date
}
```

---

## Kotlin 数据结构

```kotlin
data class User(
    val id: String,
    val email: String,
    val name: String,
    val createdAt: Instant,
)
```

---

## 嵌入式 — C 数据结构

```c
// 传感器数据包
typedef struct {
    float temperature;    // °C, 精度 ±0.3
    float humidity;       // %RH, 精度 ±2
    uint32_t timestamp;   // Unix 时间戳
    uint8_t battery_pct;  // 电池百分比 0-100
} sensor_data_t;

// LoRa 发送帧
typedef struct {
    uint8_t header;       // 帧头 0xAA
    uint8_t node_id;      // 节点 ID
    sensor_data_t data;   // 传感器数据
    uint16_t crc;         // CRC-16 校验
} lora_frame_t;
```

---

## ML/AI — 张量格式

```python
@dataclass
class ModelInput:
    input_ids: torch.Tensor      # shape: [batch_size, seq_len]
    attention_mask: torch.Tensor  # shape: [batch_size, seq_len]

@dataclass
class ModelOutput:
    logits: torch.Tensor          # shape: [batch_size, num_classes]
    probabilities: torch.Tensor   # shape: [batch_size, num_classes]
```

---

## Web 后端 — 数据库 Schema

```sql
-- 用户表
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(50) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(10) NOT NULL DEFAULT 'user',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);

-- 订单表
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    total_cents INTEGER NOT NULL,
    paid_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
```

---

## 嵌入式 — Flash 存储布局

| 地址范围 | 大小 | 内容 | 说明 |
|----------|------|------|------|
| 0x08000000 - 0x0801FFFF | 128KB | 固件 | 只读 |
| 0x08020000 - 0x08020FFF | 4KB | 配置区 | 可写，掉电保存 |
| 0x08021000 - 0x08023FFF | 12KB | 数据缓冲区 | 环形写入 |
| 0x08024000 - 0x08027FFF | 16KB | OTA 备份区 | 固件升级用 |

---

## Web 后端 — 数据流

```
HTTP POST /api/auth/register
  → src/api/auth/register.ts    (参数校验)
  → src/services/auth.ts        (业务逻辑)
  → src/repositories/user.ts    (数据库写入)
  → PostgreSQL: users 表
  → src/services/email.ts       (发送欢迎邮件)
  → Resend API
```

---

## 嵌入式 — 数据流

```
TIM2 中断触发
  → src/tasks/sensor_task.c     (任务调度)
  → drivers/sht31.c             (I2C 读取)
  → src/filters/median.c        (中值滤波)
  → src/buffer/ring.c           (写入环形缓冲)
  → src/tasks/lora_task.c       (读取缓冲、组装帧)
  → drivers/sx1276.c            (SPI 发送)
  → LoRa 网关
```

---

## iOS — 数据流

```
View.onAppear
  → ViewModel.fetchData()        (调用 Repository)
  → NetworkService.request()     (URLSession)
  → API 服务器
  → JSON 解码 → Model
  → ViewModel 更新 @Published
  → View 自动刷新
```

---

## 数据管道 — 数据流

```
Airflow DAG 触发
  → Extractor.extract()          (读取数据源 API/DB)
  → Transformer.clean()          (去重、类型转换)
  → Transformer.aggregate()      (按维度聚合)
  → Loader.load()                (写入数据仓库)
  → 通知下游
```

---

## ML/AI — 数据流

```
Dataset.__getitem__()
  → DataLoader (batch)
  → Model.forward()              (前向传播)
  → Loss function                (计算损失)
  → Optimizer.step()             (反向传播、更新参数)
  → Logger.log()                 (记录指标)
  → Checkpoint.save()            (保存模型)
```

---

## 嵌入式 — LoRa 帧格式

```
┌────────┬──────────┬─────────────┬──────────┬─────┐
│ Header │ Node ID  │ Sensor Data │ Reserved │ CRC │
│ 1 byte │ 1 byte   │ 17 bytes    │ 3 bytes  │ 2B  │
└────────┴──────────┴─────────────┴──────────┴─────┘

Header: 0xAA (固定)
CRC: CRC-16/CCITT, 多项式 0x1021
字节序: 大端
```

---

## Web 后端 — WebSocket 消息格式

```json
{
  "type": "string (消息类型)",
  "payload": "object (消息体)",
  "timestamp": "number (Unix ms)"
}
```

| type | payload | 说明 |
|------|---------|------|
| `chat.message` | `{ roomId, content }` | 发送消息 |
| `chat.typing` | `{ roomId, userId }` | 正在输入 |
| `notification` | `{ title, body }` | 系统通知 |

---

## Web 后端 — Protobuf 消息格式

```protobuf
message GetUserRequest {
  string id = 1;
}

message GetUserResponse {
  User user = 1;
}
```

---

## CLI 工具 — 输出格式

```json
// JSON 输出模式 (--output json)
{
  "status": "success",
  "data": { ... },
  "meta": { "duration_ms": 42 }
}
```

```
# 表格输出模式 (--output table)
NAME    EMAIL              ROLE
Alice   alice@example.com  admin
Bob     bob@example.com    user
```

---

## 游戏 — 网络协议

```
┌────────┬──────────┬─────────────┬─────┐
│ Type   │ Sequence │ Payload     │ CRC │
│ 1 byte │ 2 bytes  │ N bytes     │ 2B  │
└────────┴──────────┴─────────────┴─────┘

Type: 0x01=Input, 0x02=State, 0x03=Event
Sequence: 递增序号，用于丢包检测
CRC: CRC-16 校验
```

---

## Web 后端 — 缓存策略

| 数据 | 缓存位置 | TTL | 失效策略 |
|------|---------|-----|---------|
| 用户信息 | Redis | 30min | 写入时更新 |
| 产品列表 | Redis | 5min | 写入时删除 |
| 配置数据 | 内存 | 永久 | 重启时重新加载 |

---

## 嵌入式 — 缓存策略

| 数据 | 缓存位置 | TTL | 失效策略 |
|------|---------|-----|---------|
| 传感器数据 | Ring Buffer | 不缓存 | 先进先出 |
