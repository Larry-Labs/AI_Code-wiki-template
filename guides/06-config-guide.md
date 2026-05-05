# 06-config.md 填写指南

> 本指南展示各项目类型的构建、依赖、部署配置示例。
> 对应模板：`template/06-config.md`

---

## Web 后端 — 构建命令

```bash
# 安装依赖
npm install

# 开发模式
npm run dev

# 构建
npm run build

# 运行测试
npm test

# 代码检查
npm run lint
```

---

## Go 项目 — 构建命令

```bash
# 编译
go build -o bin/app ./cmd/app

# 测试
go test ./...

# 交叉编译
GOOS=linux GOARCH=arm64 go build -o bin/app-arm64 ./cmd/app
```

---

## Python 项目 — 构建命令

```bash
# 安装依赖
pip install -e ".[dev]"

# 运行
python -m src.main

# 测试
pytest

# 类型检查
mypy src/

# 打包
python -m build
```

---

## Rust 项目 — 构建命令

```bash
# 编译
cargo build --release

# 测试
cargo test

# 交叉编译
cargo build --target aarch64-unknown-linux-gnu
```

---

## 嵌入式 — 构建命令

```bash
# 编译
make all

# 烧录
make flash

# 调试
make debug

# 清理
make clean
```

---

## iOS — 构建命令

```bash
# 命令行构建
xcodebuild -scheme MyApp -configuration Release \
  -destination 'generic/platform=iOS' \
  archive -archivePath build/MyApp.xcarchive

# 导出 IPA
xcodebuild -exportArchive \
  -archivePath build/MyApp.xcarchive \
  -exportOptionsPlist ExportOptions.plist \
  -exportPath build/

# Swift Package Manager
swift build
swift test
```

---

## Android — 构建命令

```bash
# Debug 构建
./gradlew assembleDebug

# Release 构建
./gradlew assembleRelease

# 运行测试
./gradlew test

# 安装到设备
./gradlew installDebug
```

---

## 桌面应用 — 构建命令

```bash
# Electron
npm run dev
npm run build
npm run make    # 打包为安装程序

# Tauri
cargo tauri dev
cargo tauri build
```

---

## 游戏 — 构建命令

```bash
# Unity（命令行构建，CI 环境）
Unity -batchmode -nographics -projectPath . \
  -buildTarget StandaloneWindows64 \
  -executeMethod BuildScript.Build \
  -quit

# Unreal
UnrealBuildTool MyProject Win64 Development
```

---

## 基础设施 — 构建命令

```bash
# Terraform
terraform init
terraform plan
terraform apply
terraform destroy
```

---

## IDE / 构建环境

| 项目类型 | IDE | 构建工具 | 说明 |
|---------|-----|---------|------|
| Web 前端 | VS Code, WebStorm | npm/pnpm, Vite/Webpack | 无 IDE 依赖 |
| Web 后端 | VS Code, JetBrains | npm/pip/go/mvn, Docker | 无 IDE 依赖 |
| CLI 工具 | VS Code, terminal | go build/cargo/pip | 无 IDE 依赖 |
| 嵌入式 | STM32CubeIDE, IAR, Keil, PlatformIO | make/cmake, cube, platformio | 可能需要厂商 IDE 配置引脚和时钟 |
| iOS | Xcode | SPM, CocoaPods, xcodebuild | 必须 macOS + Xcode |
| Android | Android Studio | Gradle | 需要 Android SDK |
| 桌面应用 | VS, Qt Creator, VS Code | cmake, npm, dotnet | 取决于框架 |
| 数据管道 | VS Code, Jupyter | Airflow, dbt, Spark | 无 IDE 依赖 |
| ML/AI | Jupyter, VS Code | pip, conda, torchrun | 可能需要 GPU 环境 |
| 游戏 | Unity, Unreal, Godot | 引擎内置构建系统 | 必须使用游戏引擎 IDE |
| 编译器 | VS Code, CLion | cmake, make, cargo | 无 IDE 依赖 |
| 库/SDK | VS Code | cargo/npm/pip/maturin | 无 IDE 依赖 |
| 基础设施 | VS Code | terraform/pulumi | 无 IDE 依赖 |
| 微服务 | VS Code | Docker, K8s, Helm | 无 IDE 依赖 |

---

## Web 后端 — 环境变量

| 变量名 | 必填 | 默认值 | 说明 |
|--------|------|--------|------|
| `DATABASE_URL` | 是 | — | PostgreSQL 连接字符串 |
| `JWT_SECRET` | 是 | — | JWT 签名密钥（至少 32 字符） |
| `REDIS_URL` | 否 | `redis://localhost:6379` | Redis 连接字符串 |
| `STRIPE_SECRET_KEY` | 是 | — | Stripe API 密钥 |
| `RESEND_API_KEY` | 是 | — | Resend 邮件 API 密钥 |
| `NODE_ENV` | 否 | `development` | 运行环境 |
| `PORT` | 否 | `3000` | 服务端口 |
| `LOG_LEVEL` | 否 | `info` | 日志级别 |

---

## 嵌入式 — 编译宏

| 宏 | 说明 | 取值 |
|----|------|------|
| `STM32L071xx` | 芯片型号 | 定义/不定义 |
| `USE_FREERTOS` | 启用 RTOS | 1 / 0 |
| `LORA_FREQUENCY` | LoRa 频率 | 470000000 (Hz) |
| `SENSOR_INTERVAL_S` | 采集周期 | 30 (秒) |
| `DEBUG_UART` | 调试串口 | USART2 |

---

## 嵌入式 — AUTOSAR 构建命令

```bash
# 使用 Vector DaVinci Configurator 生成 BSW 配置代码
# GUI 操作：打开 .dpa 工程 → Generate → 输出到 generated/ 目录
# 命令行（CI 用）：
DaVinciConfiguratorCLI -project config/MyECU.dpa -generate

# 使用 EB tresos Studio 生成 MCAL 配置代码
# GUI 操作：打开 .epc 工程 → Generate → 输出到 mcal/generated/ 目录
# 命令行（CI 用）：
tresos_cmd -project config/MyECU.epc generate

# 编译（生成代码 + 手写代码 + BSW 库）
make all

# 烧录（通过调试器或 UDS 刷写）
make flash
# 或通过 UDS 刷写（需要 Programming Session）：
# python tools/uds_flash.py --target 192.168.1.100 build/firmware.hex
```

---

## Web 后端 — 运行时依赖

| 依赖 | 版本 | 用途 | 可替代方案 |
|------|------|------|-----------|
| express | ^4.18 | Web 框架 | fastify, koa |
| prisma | ^5.0 | ORM | drizzle, typeorm |
| bcrypt | ^5.1 | 密码哈希 | argon2 |
| jsonwebtoken | ^9.0 | JWT 操作 | jose |
| ioredis | ^5.3 | Redis 客户端 | — |

---

## 嵌入式 — 依赖

| 依赖 | 版本 | 用途 | 来源 |
|------|------|------|------|
| STM32 HAL | v1.14 | 硬件抽象层 | ST 官方 |
| FreeRTOS | v10.5 | 实时操作系统 | 官方 |
| SX1276 Driver | v2.1 | LoRa 驱动 | 自研/社区 |

### 嵌入式 — AUTOSAR 依赖

| 依赖 | 版本 | 用途 | 来源 |
|------|------|------|------|
| AUTOSAR BSW Stack | R22-11 | 基础软件栈 | Vector / EB 提供 |
| MCAL (TC397) | v3.2 | 微控制器抽象层 | Infineon 提供 |
| Vector DaVinci Configurator | v4.0 | BSW 配置工具 | 商业许可 |
| EB tresos Studio | v2024 | MCAL 配置工具 | 商业许可（可选） |
| AUTOSAR ARXML Schema | R22-11 | 配置文件格式 | AUTOSAR 标准 |

---

## iOS — 依赖

| 依赖 | 版本 | 用途 | 管理方式 |
|------|------|------|---------|
| Alamofire | ^5.8 | 网络请求 | SPM |
| Kingfisher | ^7.0 | 图片加载缓存 | SPM |
| SwiftLint | ^0.54 | 代码规范 | Homebrew |

---

## Python ML — 依赖

| 依赖 | 版本 | 用途 | 可替代方案 |
|------|------|------|-----------|
| torch | ^2.2 | 深度学习框架 | tensorflow, jax |
| transformers | ^4.40 | 预训练模型 | — |
| datasets | ^2.18 | 数据集加载 | — |
| wandb | ^0.17 | 实验追踪 | mlflow |

---

## Go — 依赖

| 依赖 | 版本 | 用途 | 可替代方案 |
|------|------|------|-----------|
| gin | ^1.9 | Web 框架 | echo, fiber |
| gorm | ^1.25 | ORM | sqlx, ent |
| cobra | ^1.8 | CLI 框架 | urfave/cli |

---

## Rust — 依赖

| 依赖 | 版本 | 用途 | 可替代方案 |
|------|------|------|-----------|
| tokio | ^1.36 | 异步运行时 | async-std |
| serde | ^1.0 | 序列化 | — |
| axum | ^0.7 | Web 框架 | actix-web |

---

## Web 后端 — 开发依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| typescript | ^5.3 | 类型检查 |
| vitest | ^1.0 | 测试框架 |
| eslint | ^8.56 | 代码检查 |
| prettier | ^3.2 | 代码格式化 |

---

## Web 后端 — Docker 部署

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY dist/ ./dist/
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

```bash
# 构建镜像
docker build -t myapp .

# 运行
docker run -p 3000:3000 --env-file .env myapp
```

---

## 嵌入式 — 烧录

```bash
# 使用 ST-Link 烧录
st-flash write build/firmware.bin 0x08000000

# 使用 OpenOCD
openocd -f interface/stlink.cfg -f target/stm32l0.cfg \
  -c "program build/firmware.elf verify reset exit"
```

---

## Web 后端 — CI/CD

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```
