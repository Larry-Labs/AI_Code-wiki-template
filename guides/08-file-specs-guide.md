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

#### 调用方

- `src/api/auth/routes.ts`

#### 下游调用

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

#### 调用方

- `src/tasks/sensor_task.c`

---

## 嵌入式 — AUTOSAR 文件规格示例

### `src/swc/DiagManager/DiagManager.c`（手写 SWC）

#### 职责

诊断管理 SWC：处理 UDS 诊断请求，管理诊断会话状态，协调 DEM 事件上报。

#### 导出

```c
// RTE 提供的生命周期接口（由 EcuM 调用）
void DiagManager_Init(void);
void DiagManager_MainFunction(void);  // 周期调用，10ms

// SWC 内部函数（不对外暴露，仅文件内使用）
static void HandleReadDID(uint16 did, uint8 *resp, uint16 *respLen);
static void HandleWriteDID(uint16 did, const uint8 *data, uint16 dataLen);
static void HandleRoutineControl(uint16 routineId, uint8 subFunc, const uint8 *req, uint8 *resp);
```

#### 依赖

- `Rte_Read_RPort_DiagRequest_DiagRequest()` — 读取诊断请求（来自 DCM）
- `Rte_Write_PPort_DiagResponse_DiagResponse()` — 写入诊断响应（到 DCM）
- `Dem_SetEventStatus()` — 上报 DEM 事件
- `NvM_WriteBlock()` — 写入标定数据到 NVM

#### 关键实现细节

- `MainFunction` 每 10ms 被 RTE 调用一次，检查是否有新的诊断请求
- ReadDID 处理：DID 0xF180（Boot SW 版本）、0xF187（零件号）、0xF18A（供应商 ID）从 ROM 常量读取
- WriteDID 处理：写入前校验数据长度和范围，通过后调用 `NvM_WriteBlock` 持久化
- RoutineControl 0xFF01（刷写前检查）：检查电压 > 9V、车速 = 0、发动机熄火
- 所有诊断响应必须在 P2 时间（5000ms）内返回，否则 DCM 自动发 NRC 0x78

#### 调用方

- `generated/Rte/Rte_DiagManager.c`（通过 RTE 周期调度）

#### 下游调用

- `generated/Dcm/` — DCM 模块（路由诊断请求）
- `generated/Dem/` — DEM 模块（事件管理）
- `generated/NvM/` — NVM 模块（数据持久化）

### `generated/Com/Com_Cfg.h`（生成文件 — 仅供参考）

#### 职责

COM 模块配置头文件，定义所有 CAN 信号的 ID、长度、字节序。由 DaVinci Configurator 自动生成。

#### 导出（生成内容示例）

```c
// 信号 ID 定义（自动生成，勿修改）
#define COM_SIG_VehicleSpeed    0u
#define COM_SIG_EngineRPM       1u
#define COM_SIG_LightCmd        2u
#define COM_SIG_LightStatus     3u

// PDU ID 定义
#define COM_PDU_VehicleInfo     0u
#define COM_PDU_LightControl    1u
#define COM_PDU_LightFeedback   2u

// 信号发送/接收函数宏（RTE 内部使用）
Com_SendSignal(Com_SignalIdType SignalId, const void *SignalDataPtr);
Com_ReceiveSignal(Com_SignalIdType SignalId, void *SignalDataPtr);
```

**重要**：此文件每次重新配置后会被覆盖，不要手动编辑。如需修改信号定义，通过 DaVinci Configurator GUI 操作。

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

#### 调用方

- `internal/handler/auth.go`

#### 下游调用

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

#### 调用方

- `src/api/auth.py`

#### 下游调用

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

#### 调用方

- `Sources/Features/Home/HomeViewModel.swift`

---

## Web 前端 — 文件规格示例

### `src/components/ProductList.tsx`

#### 职责

商品列表组件，支持分页、筛选、排序。

#### 导出

```tsx
interface ProductListProps {
  category?: string;
  sortBy?: 'price' | 'rating' | 'newest';
  page?: number;
}

export function ProductList(props: ProductListProps): JSX.Element;
```

#### 依赖

- `useProducts` — 数据获取 Hook
- `useRouter` — 路由参数同步
- `ProductCard` — 单个商品卡片组件

#### 关键实现细节

- 列表使用 `react-window` 虚拟滚动，否则 1000+ 商品会卡顿
- 筛选条件同步到 URL query params，刷新页面不丢失
- 加载更多使用 IntersectionObserver，不是按钮
- 骨架屏和实际内容切换时避免布局抖动

#### 调用方

- `src/pages/products.tsx`

#### 下游调用

- `src/hooks/useProducts.ts`
- `src/components/ProductCard.tsx`

---

## CLI 工具 (Go) — 文件规格示例

### `internal/cmd/init.go`

#### 职责

`mycli init` 命令：交互式创建项目配置文件。

#### 导出

```go
func NewInitCmd() *cobra.Command
```

#### 依赖

- `internal/config` — 配置文件读写
- `internal/template` — 模板渲染
- `github.com/AlecAivazis/survey/v2` — 交互式提示

#### 关键实现细节

- 交互式提问顺序：项目名 → 语言 → 框架 → 数据库
- 配置文件写入当前目录 `mycli.yaml`，已存在时询问是否覆盖
- 模板文件嵌入二进制（`//go:embed`），不依赖外部文件
- 输出彩色状态信息：绿色成功、红色失败、黄色警告

#### 调用方

- `cmd/root.go`（注册子命令）

#### 下游调用

- `internal/config/config.go`
- `internal/template/template.go`

---

## Android — 文件规格示例

### `presentation/home/HomeViewModel.kt`

#### 职责

首页 ViewModel，管理商品列表状态和用户交互。

#### 导出

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getProductsUseCase: GetProductsUseCase
) : ViewModel() {

    val uiState: StateFlow<HomeUiState>
    val events: SharedFlow<HomeEvent>

    fun onRefresh()
    fun onLoadMore()
    fun onProductClick(productId: String)
}
```

#### 依赖

- `GetProductsUseCase` — 获取商品列表
- `SavedStateHandle` — 恢复状态

#### 关键实现细节

- 分页使用 Paging 3 库，不是手动管理 offset
- 下拉刷新和加载更多共享同一个 Flow，通过 `LoadType` 区分
- 错误状态包含重试按钮，重试时保留已加载数据
- `onProductClick` 发送 Navigation Event，不在 ViewModel 里直接导航

#### 调用方

- `presentation/home/HomeScreen.kt`

#### 下游调用

- `domain/usecase/GetProductsUseCase.kt`

---

## 桌面应用 (Electron) — 文件规格示例

### `src/main/database.ts`

#### 职责

SQLite 数据库管理，提供文档 CRUD 操作。

#### 导出

```typescript
class Database {
  constructor(dbPath: string);

  createDoc(doc: CreateDocInput): Promise<Document>;
  getDoc(id: string): Promise<Document | null>;
  updateDoc(id: string, updates: UpdateDocInput): Promise<Document>;
  deleteDoc(id: string): Promise<void>;
  listDocs(query: ListDocsQuery): Promise<PaginatedResult<Document>>;
  close(): void;
}
```

#### 依赖

- `better-sqlite3` — SQLite 驱动
- `electron-app` — 获取 userData 路径

#### 关键实现细节

- 数据库文件存放在 `app.getPath('userData')/data.db`，不是项目目录
- 使用 WAL 模式提升并发读性能：`PRAGMA journal_mode=WAL`
- 全文搜索使用 SQLite FTS5 扩展
- 关闭时必须调用 `db.close()`，否则 WAL 文件不会合并

#### 调用方

- `src/main/ipc-handlers.ts`

#### 下游调用

- 无（直接操作数据库）

---

## 数据管道 — 文件规格示例

### `src/transform/cleaner.py`

#### 职责

原始数据清洗：去重、类型转换、异常值处理。

#### 导出

```python
class DataCleaner:
    def __init__(self, config: CleanerConfig): ...

    def clean(self, df: pd.DataFrame) -> pd.DataFrame: ...
    def deduplicate(self, df: pd.DataFrame, keys: list[str]) -> pd.DataFrame: ...
    def validate(self, df: pd.DataFrame) -> ValidationResult: ...
```

#### 依赖

- `pandas` — 数据处理
- `great_expectations` — 数据质量校验

#### 关键实现细节

- 去重使用 `df.drop_duplicates(subset=keys, keep='last')`，保留最新记录
- 时间戳统一转 UTC，时区敏感字段单独标注
- 数值列的 null 填充策略在 config 中配置，不是硬编码
- 校验失败时记录到异常表，不中断管道

#### 调用方

- `dags/daily_etl.py`

#### 下游调用

- `src/load/db_loader.py`

---

## ML/AI — 文件规格示例

### `src/train/trainer.py`

#### 职责

模型训练循环：前向传播、反向传播、checkpoint 管理。

#### 导出

```python
class Trainer:
    def __init__(
        self,
        model: nn.Module,
        optimizer: optim.Optimizer,
        scheduler: _LRScheduler,
        config: TrainConfig,
    ): ...

    def train_epoch(self, dataloader: DataLoader) -> dict[str, float]: ...
    def evaluate(self, dataloader: DataLoader) -> dict[str, float]: ...
    def save_checkpoint(self, path: str, metrics: dict) -> None: ...
    def load_checkpoint(self, path: str) -> dict: ...
```

#### 依赖

- `torch` — 训练框架
- `wandb` — 实验追踪

#### 关键实现细节

- 混合精度训练使用 `torch.amp.autocast`，不是手动 `.half()`
- Gradient clipping 在 optimizer.step() 之前：`torch.nn.utils.clip_grad_norm_`
- Checkpoint 保存 optimizer state + scheduler state + epoch + best_metric
- wandb.log 每 N step 记录一次，不是每个 step

#### 调用方

- `src/train.py`

#### 下游调用

- `src/models/model.py`
- `src/data/dataset.py`

---

## 游戏 — 文件规格示例

### `Assets/Scripts/Player/PlayerController.cs`

#### 职责

玩家控制器：移动、跳跃、碰撞检测。

#### 导出

```csharp
public class PlayerController : MonoBehaviour {
    public float moveSpeed;
    public float jumpForce;

    public Vector2 Velocity { get; }
    public bool IsGrounded { get; }

    public void Move(float horizontalInput);
    public void Jump();
    public void TakeDamage(int amount);
}
```

#### 依赖

- `Rigidbody2D` — 物理组件
- `BoxCollider2D` — 碰撞体
- `Animator` — 动画控制

#### 关键实现细节

- 移动使用 `Rigidbody2D.velocity`，不是 `Transform.Translate`
- 跳跃前检查 `Physics2D.OverlapCircle` 地面检测，避免空中连跳
- 受伤后有 0.5s 无敌时间，用协程实现
- 动画状态切换用 Animator 参数，不用直接 Play

#### 调用方

- `Assets/Scripts/Game/GameManager.cs`

#### 下游调用

- `Assets/Scripts/Player/PlayerHealth.cs`

---

## 编译器 — 文件规格示例

### `src/lexer/mod.rs`

#### 职责

词法分析器：将源码字符串转换为 Token 流。

#### 导出

```rust
#[derive(Debug, Clone, PartialEq)]
pub enum Token {
    // 字面量
    IntLiteral(i64),
    FloatLiteral(f64),
    StringLiteral(String),
    // 标识符和关键字
    Ident(String),
    Fn,
    Let,
    If,
    Else,
    Return,
    // 运算符
    Plus, Minus, Star, Slash,
    Eq, EqEq, Bang, BangEq,
    Lt, LtEq, Gt, GtEq,
    // 分隔符
    LParen, RParen, LBrace, RBrace, LBracket, RBracket,
    Semicolon, Comma, Colon, Arrow,
    // 特殊
    Eof,
}

pub struct Lexer {
    source: Vec<char>,
    pos: usize,
    line: usize,
    col: usize,
}

impl Lexer {
    pub fn new(source: &str) -> Self;
    pub fn tokenize(&mut self) -> Result<Vec<Token>, LexError>;
    fn next_token(&mut self) -> Result<Token, LexError>;
    fn read_string(&mut self) -> Result<String, LexError>;
    fn read_number(&mut self) -> Result<Token, LexError>;
}
```

#### 依赖

- 无外部依赖

#### 关键实现细节

- 字符串字面量支持转义：`\n`, `\t`, `\\`, `\"`
- 数字字面量：有小数点是 float，否则是 int
- 行号追踪在 `\n` 处递增，col 重置为 0
- 单行注释 `//` 跳过整行，不产出 Token

#### 调用方

- `src/parser/mod.rs`
- `src/main.rs`

#### 下游调用

- 无（词法分析是第一阶段）

---

## 库/SDK (npm) — 文件规格示例

### `src/client/index.ts`

#### 职责

HTTP 客户端核心：请求构造、响应解析、错误处理。

#### 导出

```typescript
interface ClientConfig {
  baseUrl: string;
  apiKey?: string;
  timeout?: number;
  retries?: number;
}

interface RequestOptions {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  path: string;
  body?: unknown;
  query?: Record<string, string>;
  headers?: Record<string, string>;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  headers: Record<string, string>;
}

class ApiClient {
  constructor(config: ClientConfig);

  request<T>(options: RequestOptions): Promise<ApiResponse<T>>;
  get<T>(path: string, query?: Record<string, string>): Promise<ApiResponse<T>>;
  post<T>(path: string, body?: unknown): Promise<ApiResponse<T>>;
  put<T>(path: string, body?: unknown): Promise<ApiResponse<T>>;
  delete<T>(path: string): Promise<ApiResponse<T>>;
}
```

#### 依赖

- `axios` — HTTP 请求

#### 关键实现细节

- 请求拦截器自动添加 `Authorization: Bearer ${apiKey}` header
- 响应拦截器统一处理 401（token 过期）、429（限流）、5xx（服务端错误）
- 429 响应解析 `Retry-After` header，自动等待后重试
- 超时默认 30s，可通过 config 覆盖
- 导出所有类型定义，用户需要 `import type { ApiResponse } from 'my-sdk'`

#### 调用方

- `src/index.ts`（作为公共 API 导出）

#### 下游调用

- `src/types/index.ts`（类型定义）

---

## 基础设施 — 文件规格示例

### `modules/eks/main.tf`

#### 职责

EKS 集群模块：控制平面 + 节点组 + IAM 角色。

#### 导出

```hcl
# 输入变量
variable "cluster_name" { type = string }
variable "cluster_version" { type = string, default = "1.29" }
variable "node_instance_types" { type = list(string), default = ["t3.medium"] }
variable "node_desired_size" { type = number, default = 2 }
variable "vpc_id" { type = string }
variable "subnet_ids" { type = list(string) }

# 输出
output "cluster_endpoint" { value = aws_eks_cluster.main.endpoint }
output "cluster_ca_certificate" { value = aws_eks_cluster.main.certificate_authority[0].data }
output "cluster_name" { value = aws_eks_cluster.main.name }
output "node_group_arn" { value = aws_eks_node_group.main.arn }
```

#### 依赖

- `modules/vpc` — VPC 和子网
- `modules/iam` — IAM 角色和策略

#### 关键实现细节

- 节点组的 IAM 角色必须附加 `AmazonEKSWorkerNodePolicy` + `AmazonEKS_CNI_Policy` + `AmazonEC2ContainerRegistryReadOnly`
- 控制平面安全组必须允许 443 端口入站（kubectl 访问）
- 节点组更新策略用 `MAX_UNAVAILABLE` 而不是 `MAX_SURGE`，避免 IP 耗尽
- `cluster_version` 升级前必须先升级节点组

#### 调用方

- `environments/dev/main.tf`
- `environments/prod/main.tf`

#### 下游调用

- `modules/iam/main.tf`

---

## 微服务 — 文件规格示例

### `services/order-service/src/handlers/create_order.rs`

#### 职责

创建订单处理器：验证请求、扣减库存、写入订单、发布事件。

#### 导出

```rust
pub async fn create_order(
    State(state): State<AppState>,
    Json(req): Json<CreateOrderRequest>,
) -> Result<Json<OrderResponse>, AppError>;

#[derive(Deserialize)]
pub struct CreateOrderRequest {
    pub user_id: String,
    pub items: Vec<OrderItem>,
}

#[derive(Serialize)]
pub struct OrderResponse {
    pub order_id: String,
    pub status: OrderStatus,
    pub total: Decimal,
}
```

#### 依赖

- `sqlx` — 数据库操作
- `reqwest` — 调用库存服务
- `kafka-rs` — 发布事件

#### 关键实现细节

- 先调用库存服务扣减，再写订单。顺序不能反（否则有库存但无订单）
- 库存扣减失败时返回 409 Conflict，不创建订单
- 订单写入和事件发布用同一数据库事务，保证一致性
- order_id 使用 UUID v7（时间有序），不用 v4
- 超时设置：库存服务 5s，数据库 3s

#### 调用方

- `services/order-service/src/router.rs`

#### 下游调用

- `services/order-service/src/db/orders.rs`
- `services/inventory-service`（HTTP 调用）
- `shared/events/kafka.rs`

---

## 文件规格模板

每个核心文件一个规格文件，包含以下字段：

| 字段 | 说明 |
|------|------|
| 职责 | 一句话说明这个文件做什么 |
| 导出 | 函数/类/接口的完整签名（类型必须精确） |
| 依赖 | 本文件依赖的模块及用途 |
| 关键实现细节 | AI 会踩坑的点、非直觉行为、协议/硬件约束 |
| 调用方 | 调用本文件的其他文件 |
| 下游调用 | 本文件调用的其他文件 |

**填写建议**：
- 只写关键文件，不需要覆盖所有文件
- 文件路径对应源码路径，去掉文件扩展名
- 导出签名必须包含完整类型信息
- 关键实现细节重点写"AI 容易出错的地方"
