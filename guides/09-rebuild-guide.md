# 09-rebuild.md 填写指南

> 本指南展示各项目类型的重建顺序示例。
> 对应模板：`template/09-rebuild.md`

---

## Web 后端 — 重建顺序

### 前置条件

- [ ] 安装 Node.js 20+
- [ ] 安装 PostgreSQL 16+
- [ ] 获取 Stripe 测试密钥
- [ ] 获取 Resend API 密钥

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
mkdir my-app && cd my-app
npm init -y
npm install typescript @types/node tsx
npx tsc --init
```

**验证**：`npx tsx -e "console.log('ok')"` 输出 ok

#### Step 2：安装核心依赖

```bash
npm install express prisma bcrypt jsonwebtoken ioredis
npm install -D @types/express @types/bcrypt @types/jsonwebtoken vitest eslint prettier
```

**验证**：`npm install` 无报错

#### Step 3：初始化数据库

```bash
npx prisma init
# 编辑 prisma/schema.prisma（见 04-data.md 的 Schema 定义）
npx prisma migrate dev --name init
```

**验证**：`npx prisma studio` 能看到空的 users 表

#### Step 4：创建目录结构

```
src/
├── api/
├── services/
├── repositories/
├── models/
├── middleware/
├── utils/
└── config/
```

**验证**：目录结构符合 `07-conventions.md` 的文件组织约定

### 第二阶段：基础层

#### Step 5：实现配置模块

- 文件：`src/config/index.ts`
- 内容：读取环境变量，导出类型安全的配置对象
- 参考：`06-config.md` 的环境变量表

**验证**：`import { config } from '@/config'` 能正确读取环境变量

#### Step 6：实现数据模型

- 文件：`src/models/user.ts`, `src/models/order.ts`
- 内容：TypeScript 接口定义
- 参考：`04-data.md` 的核心数据结构

**验证**：TypeScript 编译无错误

#### Step 7：实现 Repository 层

- 文件：`src/repositories/user.ts`, `src/repositories/order.ts`
- 内容：CRUD 操作封装
- 参考：`08-file-specs/` 中对应文件的规格

**验证**：编写单元测试，mock Prisma Client，CRUD 操作通过

#### Step 8：实现工具函数

- 文件：`src/utils/crypto.ts`, `src/utils/jwt.ts`, `src/utils/errors.ts`
- 内容：密码哈希、JWT 操作、自定义错误类
- 参考：`05-logic.md` 的 JWT 签发算法、`07-conventions.md` 的错误处理

**验证**：编写单元测试，密码哈希/验证、JWT 签发/验证通过

### 第三阶段：业务层

#### Step 9：实现认证服务

- 文件：`src/services/auth.ts`
- 内容：注册、登录、Token 刷新
- 参考：`05-logic.md` 的用户注册流程、`08-file-specs/` 中对应文件的规格

**验证**：
- [ ] 注册 → 返回 user + token
- [ ] 重复注册 → 返回 409
- [ ] 登录 → 返回 token + refreshToken
- [ ] 刷新 token → 返回新 token

#### Step 10：实现 API 路由

- 文件：`src/api/auth/routes.ts`
- 内容：注册、登录、刷新的 HTTP 端点
- 参考：`03-interfaces.md` 的 HTTP API 定义

**验证**：用 curl 或 Postman 测试所有端点

#### Step 11：实现中间件

- 文件：`src/middleware/auth.ts`, `src/middleware/error-handler.ts`
- 内容：JWT 验证中间件、全局错误处理

**验证**：
- [ ] 无 token 访问受保护路由 → 返回 401
- [ ] 无效 token → 返回 401
- [ ] 业务错误 → 返回正确状态码和错误信息

### 第四阶段：集成测试

#### Step 12：端到端测试

**验证**：
- [ ] 完整注册 → 登录 → 访问受保护资源流程
- [ ] 并发注册同一邮箱 → 只有一个成功
- [ ] Token 过期 → 刷新 → 继续访问

#### Step 13：性能验证

**验证**：
- [ ] API P99 < 200ms（基准测试）
- [ ] 100 并发无报错

---

## 嵌入式 — 重建顺序

### 前置条件

- [ ] 安装 arm-none-eabi-gcc 13+
- [ ] 安装 STM32CubeMX（可选，用于引脚配置）
- [ ] 准备 ST-Link 调试器
- [ ] 准备目标开发板

### Phase 1：硬件抽象

1. 初始化 STM32CubeMX 工程，配置时钟、引脚
2. 实现 `drivers/sht31.c` — I2C 传感器驱动
3. 实现 `drivers/sx1276.c` — SPI LoRa 驱动
4. 验证：单独读取传感器数据成功

### Phase 2：系统服务

5. 配置 FreeRTOS，创建任务
6. 实现 `src/buffer/ring.c` — 环形缓冲区
7. 实现 `src/utils/crc.c` — CRC 校验
8. 验证：任务调度正常，缓冲区读写正确

### Phase 3：业务逻辑

9. 实现 `src/tasks/sensor_task.c` — 传感器采集任务
10. 实现 `src/tasks/lora_task.c` — LoRa 发送任务
11. 验证：采集 → 缓冲 → 发送 完整流程跑通

### Phase 4：系统集成

12. 集成所有任务，测试长时间运行
13. 测量功耗，优化休眠策略
14. 验证：连续运行 24h 无异常

---

## 嵌入式 — AUTOSAR 重建顺序

### 前置条件

- [ ] 安装 Vector DaVinci Configurator 或 EB tresos Studio
- [ ] 获取 BSW 栈源码（供应商授权）
- [ ] 获取 MCAL 驱动（芯片厂商提供）
- [ ] 获取 AUTOSAR ARXML Schema 文件
- [ ] 准备调试器（Lauterbach / iSYSTEM）

### Phase 1：MCAL 配置与生成

1. 使用 EB tresos 配置 MCAL 模块（CAN、SPI、ADC、DIO、PWM）
2. 生成 MCAL 初始化代码到 `mcal/generated/`
3. 验证：MCAL 模块初始化成功，CAN 控制器进入 Normal 模式

### Phase 2：BSW 集成

4. 使用 DaVinci Configurator 配置 BSW 模块（COM、DCM、DEM、NVM、EcuM）
5. 生成 RTE 和 BSW 配置代码到 `generated/`
6. 集成 BSW 源码到 `bsw/` 目录
7. 编译链接，确保零错误
8. 验证：EcuM 启动流程跑通，COM 模块发送/接收 CAN 报文成功

### Phase 3：SWC 开发

9. 实现 `SensorAcq` SWC — 传感器数据采集
10. 实现 `LightControl` SWC — 灯光控制逻辑
11. 实现 `DiagManager` SWC — 诊断请求处理
12. 验证：SWC 间通过 RTE 通信正常，CAN 信号收发正确

### Phase 4：诊断与标定集成

13. 配置 DCM 诊断服务（0x22/0x2E/0x27/0x31）
14. 配置 DEM 事件和 DTC 定义
15. 配置 NVM 块（DTC 快照、标定数据）
16. 验证：UDS 诊断仪可以读 DID、写 DID、清 DTC

### Phase 5：系统集成与验证

17. 全功能集成测试（CAN 矩阵验证、诊断全流程）
18. 功能安全测试（ASIL-B/D 故障注入测试）
19. 耐久测试（连续运行 1000h 无异常）
20. 验证：通过 OEM 认证测试规范

---

## CLI 工具（Go）— 重建顺序

### 前置条件

- [ ] 安装 Go 1.22+
- [ ] 安装 golangci-lint（可选）

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
mkdir mycli && cd mycli
go mod init github.com/user/mycli
mkdir -p cmd/app internal/{handler,service,config} pkg/utils
```

**验证**：`go build ./cmd/app` 无报错

#### Step 2：实现配置模块

- 文件：`internal/config/config.go`
- 内容：读取 `.mycli/config.yaml`，导出配置结构体

**验证**：`go test ./internal/config/` 通过

### 第二阶段：核心逻辑

#### Step 3：实现命令解析

- 文件：`cmd/app/main.go`
- 内容：使用 cobra 定义命令和子命令

**验证**：`go run ./cmd/app --help` 输出帮助信息

#### Step 4：实现业务逻辑

- 文件：`internal/service/*.go`
- 内容：核心功能实现

**验证**：单元测试通过

### 第三阶段：集成

#### Step 5：实现 CLI 输出

- 文件：`internal/handler/*.go`
- 内容：JSON/表格输出格式化

**验证**：
- [ ] `mycli build --output json` 输出合法 JSON
- [ ] `mycli build --output table` 输出对齐表格

#### Step 6：交叉编译和发布

```bash
GOOS=linux GOARCH=amd64 go build -o dist/mycli-linux-amd64 ./cmd/app
GOOS=darwin GOARCH=arm64 go build -o dist/mycli-darwin-arm64 ./cmd/app
GOOS=windows GOARCH=amd64 go build -o dist/mycli-windows-amd64.exe ./cmd/app
```

**验证**：各平台二进制可执行

---

## Python 库 — 重建顺序

### 前置条件

- [ ] 安装 Python 3.10+
- [ ] 安装 pip / poetry / uv

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
mkdir mylib && cd mylib
python -m venv .venv
source .venv/bin/activate
pip install build pytest mypy ruff
```

**验证**：`python -c "import mylib"` 无报错

#### Step 2：创建包结构

```
src/mylib/
├── __init__.py
├── core.py
└── utils.py
tests/
├── test_core.py
└── conftest.py
pyproject.toml
```

**验证**：`pytest` 能发现测试文件

### 第二阶段：核心实现

#### Step 3：实现核心功能

- 文件：`src/mylib/core.py`
- 内容：主要 API 和数据结构

**验证**：`pytest tests/test_core.py` 通过

#### Step 4：实现工具函数

- 文件：`src/mylib/utils.py`
- 内容：辅助函数

**验证**：类型检查 `mypy src/` 无错误

### 第三阶段：发布

#### Step 5：打包和发布

```bash
python -m build
twine upload dist/*
```

**验证**：`pip install mylib` 从 PyPI 安装成功

---

## iOS 应用 — 重建顺序

### 前置条件

- [ ] macOS + Xcode 15+
- [ ] Apple Developer 账号（发布需要）

### 第一阶段：项目骨架

#### Step 1：创建 Xcode 项目

- 使用 Xcode 创建 SwiftUI App 项目
- 配置 Bundle Identifier、Team、Signing

**验证**：模拟器运行显示空白页面

#### Step 2：搭建架构

```
Sources/
├── App/
│   └── MyApp.swift
├── Core/
│   ├── Network/
│   └── Persistence/
└── Features/
    └── Home/
```

**验证**：项目编译无错误

### 第二阶段：数据层

#### Step 3：实现网络层

- 文件：`Sources/Core/Network/APIClient.swift`
- 内容：URLSession 封装、JSON 解码

**验证**：Playground 测试 API 调用

#### Step 4：实现数据模型

- 文件：`Sources/Models/User.swift`
- 内容：Codable 结构体

**验证**：JSON 解码测试通过

### 第三阶段：UI 层

#### Step 5：实现主界面

- 文件：`Sources/Features/Home/HomeView.swift`
- 内容：SwiftUI 列表、ViewModel

**验证**：模拟器显示数据列表

#### Step 6：实现详情页和导航

- 文件：`Sources/Features/Detail/DetailView.swift`
- 内容：NavigationStack、数据传递

**验证**：点击列表项跳转详情页

### 第四阶段：发布

#### Step 7：Archive 和上传

```bash
xcodebuild -scheme MyApp -configuration Release archive -archivePath build/MyApp.xcarchive
```

**验证**：App Store Connect 收到构建版本

---

## Android 应用 — 重建顺序

### 前置条件

- [ ] Android Studio Hedgehog+
- [ ] Android SDK 34+
- [ ] JDK 17+

### 第一阶段：项目骨架

#### Step 1：创建项目

- 使用 Android Studio 创建 Empty Compose Activity 项目
- 配置 `build.gradle.kts` 依赖

**验证**：模拟器运行显示 "Hello World"

#### Step 2：搭建架构

```
app/src/main/java/com/example/app/
├── di/
├── data/local/
├── data/remote/
├── domain/model/
├── domain/usecase/
└── presentation/home/
```

**验证**：项目编译无错误

### 第二阶段：数据层

#### Step 3：实现 Retrofit API

- 文件：`data/remote/ApiService.kt`
- 内容：接口定义、DTO

**验证**：单元测试通过

#### Step 4：实现 Room 数据库

- 文件：`data/local/AppDatabase.kt`
- 内容：Entity、DAO、Database

**验证**：Instrumented 测试通过

### 第三阶段：UI 层

#### Step 5：实现主界面

- 文件：`presentation/home/HomeScreen.kt`
- 内容：Compose UI、ViewModel

**验证**：模拟器显示数据列表

#### Step 6：实现导航

- 文件：`presentation/navigation/NavGraph.kt`
- 内容：NavHost、路由定义

**验证**：页面跳转正常

### 第四阶段：发布

#### Step 7：签名和打包

```bash
./gradlew assembleRelease
# 或
./gradlew bundleRelease
```

**验证**：APK/AAB 文件生成，可安装到设备

---

## 数据管道 — 重建顺序

### 前置条件

- [ ] 安装 Python 3.10+
- [ ] 安装 Airflow / dbt / Spark（根据技术栈）
- [ ] 配置数据源连接

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
mkdir my-pipeline && cd my-pipeline
python -m venv .venv
pip install apache-airflow dbt-core
```

**验证**：`airflow version` 输出版本号

#### Step 2：创建 DAG 结构

```
dags/
└── daily_etl.py
src/
├── extract/
├── transform/
└── load/
tests/
```

**验证**：Airflow UI 能看到 DAG

### 第二阶段：ETL 逻辑

#### Step 3：实现 Extractor

- 文件：`src/extract/api_extractor.py`
- 内容：数据源连接、增量拉取

**验证**：能拉取测试数据

#### Step 4：实现 Transformer

- 文件：`src/transform/cleaner.py`
- 内容：去重、类型转换、聚合

**验证**：转换后数据格式正确

#### Step 5：实现 Loader

- 文件：`src/load/db_loader.py`
- 内容：upsert 写入目标表

**验证**：数据写入目标数据库

### 第三阶段：调度和监控

#### Step 6：配置 DAG 调度

- 文件：`dags/daily_etl.py`
- 内容：任务依赖、重试策略、告警

**验证**：
- [ ] DAG 按时触发
- [ ] 失败时发送告警
- [ ] 重试机制正常工作

---

## ML/AI — 重建顺序

### 前置条件

- [ ] 安装 Python 3.10+
- [ ] 安装 CUDA（如需 GPU）
- [ ] 准备训练数据

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
mkdir my-model && cd my-model
python -m venv .venv
pip install torch transformers datasets wandb
```

**验证**：`python -c "import torch; print(torch.cuda.is_available())"` 正常输出

#### Step 2：创建目录结构

```
src/
├── data/               # Dataset、DataLoader
├── models/             # 模型定义
├── train/              # 训练循环
└── inference/          # 推理服务
configs/
└── base.yaml
```

### 第二阶段：数据和模型

#### Step 3：实现 Dataset

- 文件：`src/data/dataset.py`
- 内容：数据加载、预处理

**验证**：DataLoader 能正确产出 batch

#### Step 4：实现模型

- 文件：`src/models/model.py`
- 内容：模型定义

**验证**：前向传播能跑通，输出 shape 正确

### 第三阶段：训练

#### Step 5：实现训练循环

- 文件：`src/train/trainer.py`
- 内容：训练循环、优化器、checkpoint

**验证**：
- [ ] Loss 正常下降
- [ ] Checkpoint 能保存和加载

#### Step 6：训练和评估

**验证**：
- [ ] 验证集指标达标
- [ ] 模型大小在预算内

---

## 游戏 — 重建顺序

### 前置条件

- [ ] 安装 Unity/Unreal/Godot
- [ ] 准备美术资源

### 第一阶段：项目骨架

#### Step 1：创建项目

- 使用引擎创建项目
- 配置基础设置

**验证**：运行显示空白场景

#### Step 2：实现游戏循环

- 内容：Update 循环、输入处理

**验证**：能检测到输入事件

### 第二阶段：核心系统

#### Step 3：实现物理/碰撞

**验证**：物体能正确碰撞

#### Step 4：实现渲染

**验证**：画面正确显示

### 第三阶段：游戏逻辑

#### Step 5：实现游戏机制

**验证**：核心玩法可体验

#### Step 6：实现 UI

**验证**：HUD 和菜单正常工作

### 第四阶段：发布

#### Step 7：打包和测试

**验证**：各平台构建正常，帧率达标

---

## 桌面应用 (Electron/Tauri) — 重建顺序

### 前置条件

- [ ] 安装 Node.js 18+（Electron）或 Rust（Tauri）
- [ ] 安装对应框架 CLI

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
# Electron
npx create-electron-app my-desktop --template typescript

# Tauri
npm create tauri-app@latest my-desktop -- --template react-ts
```

**验证**：应用窗口能启动

#### Step 2：搭建架构

```
src/
├── main/           # 主进程
├── renderer/       # 渲染进程
├── shared/         # 共享类型和工具
└── preload.ts      # 预加载脚本
```

**验证**：主进程和渲染进程通信正常（ipcRenderer/ipcMain）

### 第二阶段：核心功能

#### Step 3：实现数据持久化

- 文件：`src/main/database.ts`
- 内容：SQLite 连接、CRUD 操作

**验证**：数据重启后保留

#### Step 4：实现 UI

- 文件：`src/renderer/App.tsx`
- 内容：主界面、交互逻辑

**验证**：功能可正常使用

### 第三阶段：发布

#### Step 5：打包

```bash
# Electron
npm run make

# Tauri
npm run tauri build
```

**验证**：安装包在目标平台可安装运行

---

## 编译器/语言工具 — 重建顺序

### 前置条件

- [ ] 安装 Rust 或对应语言工具链
- [ ] 理解目标语言规范

### 第一阶段：项目骨架

#### Step 1：初始化项目

```bash
cargo init my-lang
```

**验证**：`cargo run` 输出 "Hello, world!"

#### Step 2：创建模块结构

```
src/
├── lexer/          # 词法分析
├── parser/         # 语法分析
├── ast/            # AST 定义
├── codegen/        # 代码生成
└── main.rs         # 入口
```

### 第二阶段：前端

#### Step 3：实现词法分析器

- 文件：`src/lexer/mod.rs`
- 内容：Token 定义、词法分析逻辑

**验证**：能正确 tokenize 测试用例

#### Step 4：实现语法分析器

- 文件：`src/parser/mod.rs`
- 内容：AST 节点、递归下降解析

**验证**：能正确 parse 测试用例，AST 结构正确

### 第三阶段：后端

#### Step 5：实现代码生成

- 文件：`src/codegen/mod.rs`
- 内容：AST → IR/目标代码

**验证**：生成的代码能正确执行

### 第四阶段：集成

#### Step 6：端到端测试

**验证**：
- [ ] 编译 hello world 程序
- [ ] 错误信息友好且准确
- [ ] 性能指标达标

---

## 基础设施 (Terraform + K8s) — 重建顺序

### 前置条件

- [ ] 安装 Terraform >= 1.7
- [ ] 安装 kubectl
- [ ] 配置云厂商凭证

### 第一阶段：基础设施

#### Step 1：定义 Provider

```hcl
# main.tf
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}
```

**验证**：`terraform init` 成功

#### Step 2：创建网络

- 文件：`modules/vpc/main.tf`
- 内容：VPC、子网、路由表

**验证**：`terraform plan` 无错误

### 第二阶段：核心服务

#### Step 3：创建 K8s 集群

- 文件：`modules/eks/main.tf`
- 内容：EKS 集群、节点组

**验证**：`kubectl get nodes` 显示节点就绪

#### Step 4：创建数据库

- 文件：`modules/rds/main.tf`
- 内容：RDS 实例、安全组

**验证**：数据库可连接

### 第三阶段：应用部署

#### Step 5：部署 Helm Charts

- 文件：`charts/app/`
- 内容：Deployment、Service、Ingress

**验证**：应用可通过 Ingress 访问

### 第四阶段：多环境

#### Step 6：配置环境

- 文件：`environments/dev/`, `environments/prod/`
- 内容：环境变量覆盖

**验证**：
- [ ] `terraform workspace select dev && terraform apply` 成功
- [ ] `terraform workspace select prod && terraform apply` 成功

---

## 微服务 — 重建顺序

### 前置条件

- [ ] 安装 Docker + Docker Compose
- [ ] 安装 Kubernetes（本地用 kind/minikube）
- [ ] 安装 Helm

### 第一阶段：项目骨架

#### Step 1：初始化 monorepo

```
services/
├── user-service/
├── order-service/
├── gateway/
infra/
├── docker-compose.yaml
├── k8s/
```

**验证**：`docker compose up` 所有服务启动

#### Step 2：实现 API 网关

- 文件：`gateway/src/main.ts`
- 内容：路由转发、认证中间件

**验证**：网关能转发请求到后端服务

### 第二阶段：核心服务

#### Step 3：实现用户服务

- 文件：`services/user-service/`
- 内容：CRUD、JWT 认证

**验证**：注册、登录接口正常

#### Step 4：实现订单服务

- 文件：`services/order-service/`
- 内容：订单 CRUD、状态机

**验证**：创建订单、查询订单正常

### 第三阶段：服务间通信

#### Step 5：实现消息队列

- 内容：Kafka/RabbitMQ 事件发布和消费

**验证**：订单创建后用户服务收到事件

#### Step 6：实现分布式追踪

- 内容：OpenTelemetry 集成

**验证**：能在 Jaeger UI 看到完整调用链

### 第四阶段：部署

#### Step 7：K8s 部署

- 文件：`infra/k8s/`
- 内容：Deployment、Service、ConfigMap

**验证**：
- [ ] 所有 Pod Running
- [ ] 服务间通信正常
- [ ] 端到端请求成功

---

## 回顾（通用模板）

重建完成后，记录实际遇到的问题和调整：

### 实际与预期的差异

- ...

### 踩过的坑

- ...

### 对规范的改进建议

- ...
