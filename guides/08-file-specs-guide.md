# 08-file-specs 填写指南

> 本指南展示各项目类型的文件规格示例。
> 对应模板：`template/08-file-specs/_example.md`

---

## Web 后端 — 文件规格示例

### `src/services/auth.ts`

#### 职责

用户认证相关的业务逻辑：注册、登录、Token 管理。

#### 导出

```typescript
class AuthService {
  constructor(
    private userRepo: UserRepository,
    private jwtService: JwtService,
    private emailService: EmailService,
  ) {}

  async register(input: RegisterInput): Promise<{ user: User; token: string }>;
  async login(input: LoginInput): Promise<{ token: string; refreshToken: string }>;
  async refreshToken(refreshToken: string): Promise<{ token: string }>;
}
```

#### 依赖

- `UserRepository` — 数据库操作
- `JwtService` — Token 生成/验证
- `EmailService` — 发送欢迎邮件
- `bcrypt` — 密码哈希

#### 关键实现细节

- 注册时先检查邮箱唯一性，再写入数据库（顺序不能反）
- 密码哈希使用 bcrypt，rounds=10
- JWT payload 包含 `{ sub: userId, role }`，有效期 7 天
- 邮件发送是异步的，不阻塞注册返回
- 刷新 token 时验证旧 token 是否在黑名单中

#### 被调用者

- `src/api/auth/routes.ts`

#### 调用

- `src/repositories/user.ts`
- `src/utils/jwt.ts`
- `src/services/email.ts`

---

## 嵌入式 — 文件规格示例

### `drivers/sht31.c`

#### 职责

SHT31 温湿度传感器驱动，通过 I2C 总线通信。

#### 导出函数

```c
// 初始化：软复位 + 检查状态寄存器
HAL_StatusTypeDef SHT31_Init(I2C_HandleTypeDef *hi2c);

// 单次测量：发送命令 → 等待 → 读取 → CRC 校验 → 转换
HAL_StatusTypeDef SHT31_Measure(I2C_HandleTypeDef *hi2c, float *temp, float *hum);

// CRC-8 校验（多项式 0x31，初始值 0xFF）
uint8_t SHT31_CRC8(const uint8_t *data, uint8_t len);
```

#### 内部常量

```c
#define SHT31_ADDR          (0x44 << 1)   // 8-bit 地址（HAL 要求左移）
#define SHT31_CMD_MEASURE_H  0x24         // 高精度测量命令 MSB
#define SHT31_CMD_MEASURE_L  0x00         // 高精度测量命令 LSB
#define SHT31_MEASURE_DELAY  15           // 测量等待时间 (ms)
```

#### 关键实现细节

- I2C 地址必须左移 1 位（STM32 HAL 库要求 8-bit 格式）
- 测量命令 `{0x24, 0x00}` 发送后必须等待 15ms 再读取
- 返回数据格式：`[temp_msb, temp_lsb, temp_crc, hum_msb, hum_lsb, hum_crc]`
- 转换公式：`temp = -45 + 175 * raw / 65535`，`hum = 100 * raw / 65535`
- CRC 校验失败时返回 `HAL_ERROR`，调用方决定是否重试

#### 依赖

- `stm32l0xx_hal_i2c.h` — I2C HAL 驱动

#### 被调用者

- `src/tasks/sensor_task.c`

---

## Go — 文件规格示例

### `internal/service/auth.go`

#### 职责

用户认证业务逻辑：注册、登录、JWT 管理。

#### 导出

```go
type AuthService struct {
    userRepo  UserRepository
    jwtSecret []byte
}

func NewAuthService(userRepo UserRepository, jwtSecret []byte) *AuthService
func (s *AuthService) Register(ctx context.Context, input RegisterInput) (*User, string, error)
func (s *AuthService) Login(ctx context.Context, input LoginInput) (string, string, error)
```

#### 关键实现细节

- 使用 `golang.org/x/crypto/bcrypt` 哈希密码
- JWT 使用 `github.com/golang-jwt/jwt/v5`
- 错误返回 `*AppError` 类型，包含 HTTP 状态码

#### 被调用者

- `internal/handler/auth.go`

#### 调用

- `internal/repository/user.go`

---

## Python — 文件规格示例

### `src/services/auth.py`

#### 职责

用户认证业务逻辑。

#### 导出

```python
class AuthService:
    def __init__(self, user_repo: UserRepository, jwt_service: JwtService): ...

    async def register(self, input: RegisterInput) -> tuple[User, str]: ...
    async def login(self, input: LoginInput) -> tuple[str, str]: ...
```

#### 关键实现细节

- 使用 `bcrypt` 库哈希密码
- JWT 使用 `python-jose`
- 异步操作使用 `async/await`

#### 被调用者

- `src/api/auth.py`

#### 调用

- `src/repositories/user.py`

---

## iOS — 文件规格示例

### `Sources/Core/Network/APIClient.swift`

#### 职责

URLSession 封装，提供类型安全的 API 请求。

#### 导出

```swift
protocol APIClientProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

class APIClient: APIClientProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}
```

#### 关键实现细节

- 使用 `async/await`，不使用 Combine
- JSON 解码使用 `JSONDecoder`，日期格式 `.iso8601`
- 401 响应触发 token 刷新后重试一次

#### 被调用者

- `Sources/Features/Home/HomeViewModel.swift`

---

## 文件规格模板

每个核心文件一个规格文件，包含以下字段：

| 字段 | 说明 |
|------|------|
| 职责 | 一句话说明这个文件做什么 |
| 导出 | 函数/类/接口的完整签名（类型必须精确） |
| 依赖 | 本文件依赖的模块及用途 |
| 关键实现细节 | AI 会踩坑的点、非直觉行为、协议/硬件约束 |
| 被调用者 | 调用本文件的其他文件 |
| 调用 | 本文件调用的其他文件 |

**填写建议**：
- 只写关键文件，不需要覆盖所有文件
- 文件路径对应源码路径，去掉文件扩展名
- 导出签名必须包含完整类型信息
- 关键实现细节重点写"AI 容易出错的地方"
