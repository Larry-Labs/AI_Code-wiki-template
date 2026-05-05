# 00-manifest.yaml 填写指南

> 本指南展示各项目类型如何填写 manifest 文件。
> 对应模板：`template/00-manifest.yaml`

---

## Web 后端

```yaml
name: my-api
description: "用户认证和订单管理的 REST API 服务"

tech_stack:
  language: typescript
  runtime: node@20
  framework: express@4
  database: postgresql@16
  orm: prisma
  style: none

dependencies:
  critical:
    - name: stripe
      version: "^14.0"
      purpose: "支付处理"
    - name: resend
      version: "^3.0"
      purpose: "邮件发送"
  optional:
    - name: sentry
      version: "^8.0"
      purpose: "错误监控"

entry_points:
  main: src/main.ts
  config: src/config/index.ts
  api: src/api/
  db: prisma/schema.prisma

directories:
  src/: "源代码"
  tests/: "测试"
  prisma/: "数据库 Schema"
  scripts/: "脚本工具"

meta:
  author: ""
  license: MIT
  repo: ""
  created: ""
```

---

## Web 前端

```yaml
name: my-webapp
description: "电商平台前端，支持商品浏览和下单"

tech_stack:
  language: typescript
  runtime: browser
  framework: next@14
  database: none
  orm: none
  style: tailwindcss

dependencies:
  critical:
    - name: react
      version: "^18"
      purpose: "UI 框架"
    - name: zustand
      version: "^4"
      purpose: "状态管理"
  optional:
    - name: framer-motion
      version: "^11"
      purpose: "动画"

entry_points:
  main: app/layout.tsx
  config: next.config.js
  api: app/api/

directories:
  app/: "页面路由"
  components/: "UI 组件"
  lib/: "工具函数"
  hooks/: "自定义 Hook"
```

---

## CLI 工具 (Go)

```yaml
name: mycli
description: "项目脚手架生成工具"

tech_stack:
  language: go
  runtime: go@1.22
  framework: cobra
  database: none
  orm: none
  style: terminal

dependencies:
  critical:
    - name: github.com/spf13/cobra
      version: "^1.8"
      purpose: "CLI 框架"
    - name: gopkg.in/yaml.v3
      version: "^3.0"
      purpose: "配置文件解析"
  optional:
    - name: github.com/fatih/color
      version: "^1.16"
      purpose: "终端彩色输出"

entry_points:
  main: cmd/app/main.go
  config: internal/config/config.go

directories:
  cmd/: "命令入口"
  internal/: "内部实现"
  pkg/: "可复用公共库"
```

---

## 嵌入式

```yaml
name: my-sensor-node
description: "LoRa 温湿度采集节点，电池供电"

tech_stack:
  language: c
  runtime: stm32
  framework: freertos
  database: flash
  orm: none
  style: none

dependencies:
  critical:
    - name: STM32 HAL
      version: "v1.14"
      purpose: "硬件抽象层"
    - name: FreeRTOS
      version: "v10.5"
      purpose: "实时操作系统"
  optional:
    - name: SX1276 Driver
      version: "v2.1"
      purpose: "LoRa 通信驱动"

entry_points:
  main: src/main.c
  config: config/FreeRTOSConfig.h

directories:
  src/: "应用代码"
  drivers/: "硬件驱动"
  config/: "板级配置"
  Middlewares/: "RTOS 和协议栈"
```

### 嵌入式 — AUTOSAR Classic Platform

```yaml
name: my-autosar-ecu
description: "车身控制器 ECU，基于 AUTOSAR Classic Platform"

tech_stack:
  language: c
  runtime: autosar-classic
  framework: autosar-bsw
  database: nvm
  orm: none
  style: misra-c-2012

dependencies:
  critical:
    - name: AUTOSAR BSW
      version: "4.4"
      purpose: "基础软件栈（COM, NM, DCM, DEM, NVM 等）"
    - name: MCAL
      version: "v3.2"
      purpose: "微控制器抽象层（CAN, SPI, ADC, DIO, PWM 驱动）"
  optional:
    - name: Vector DaVinci Configurator
      version: "v4.0"
      purpose: "BSW 模块配置和代码生成"
    - name: EB tresos Studio
      version: "v2024"
      purpose: "MCAL 配置和代码生成（Elektrobit 方案）"

entry_points:
  main: src/EcuM/EcuM_Main.c
  config: config/ECU_Configuration.arxml

directories:
  src/: "应用层 SWC 代码（手写）"
  generated/: "配置工具生成的代码（不要手动修改）"
  config/: "ARXML 配置文件"
  bsw/: "BSW 模块源码（供应商提供）"
  mcal/: "MCAL 驱动（芯片厂商提供）"
```

---

## iOS 应用

```yaml
name: MyApp
description: "社交分享应用"

tech_stack:
  language: swift
  runtime: ios@17
  framework: swiftui
  database: coredata
  orm: none
  style: swiftui

dependencies:
  critical:
    - name: Alamofire
      version: "^5.8"
      purpose: "网络请求"
    - name: Kingfisher
      version: "^7.0"
      purpose: "图片加载缓存"
  optional:
    - name: SwiftLint
      version: "^0.54"
      purpose: "代码规范"

entry_points:
  main: Sources/App/MyApp.swift
  config: Sources/Core/Config/AppConfig.swift

directories:
  Sources/: "源代码"
  Resources/: "资源文件"
  Tests/: "测试"
```

---

## Android 应用

```yaml
name: myapp
description: "任务管理应用"

tech_stack:
  language: kotlin
  runtime: android@34
  framework: jetpack-compose
  database: room
  orm: none
  style: material3

dependencies:
  critical:
    - name: com.squareup.retrofit2
      version: "^2.9"
      purpose: "网络请求"
    - name: androidx.room
      version: "^2.6"
      purpose: "本地数据库"
  optional:
    - name: com.google.dagger
      version: "^2.50"
      purpose: "依赖注入"

entry_points:
  main: app/src/main/java/com/example/app/MainActivity.kt
  config: app/build.gradle.kts

directories:
  app/: "应用模块"
  data/: "数据层"
  domain/: "领域层"
  presentation/: "表现层"
```

---

## 数据管道

```yaml
name: my-pipeline
description: "用户行为数据 ETL 管道"

tech_stack:
  language: python
  runtime: python@3.12
  framework: airflow
  database: postgresql@16
  orm: sqlalchemy
  style: none

dependencies:
  critical:
    - name: apache-airflow
      version: "^2.8"
      purpose: "任务调度"
    - name: dbt-core
      version: "^1.7"
      purpose: "数据转换"
  optional:
    - name: great_expectations
      version: "^0.18"
      purpose: "数据质量检查"

entry_points:
  main: dags/daily_etl.py
  config: configs/pipeline.yaml

directories:
  dags/: "Airflow DAG 定义"
  src/: "ETL 逻辑"
  tests/: "测试"
  configs/: "配置文件"
```

---

## ML/AI

```yaml
name: my-model
description: "文本分类模型训练和推理"

tech_stack:
  language: python
  runtime: python@3.12
  framework: pytorch
  database: none
  orm: none
  style: none

dependencies:
  critical:
    - name: torch
      version: "^2.2"
      purpose: "深度学习框架"
    - name: transformers
      version: "^4.40"
      purpose: "预训练模型"
  optional:
    - name: wandb
      version: "^0.17"
      purpose: "实验追踪"

entry_points:
  main: src/train.py
  config: configs/base.yaml

directories:
  src/: "源代码"
  configs/: "训练配置"
  data/: "数据集"
  models/: "模型定义"
  notebooks/: "实验笔记本"
```

---

## 游戏

```yaml
name: my-game
description: "2D 平台跳跃游戏"

tech_stack:
  language: csharp
  runtime: unity@2022
  framework: unity
  database: none
  orm: none
  style: unity

dependencies:
  critical:
    - name: com.unity.inputsystem
      version: "^1.7"
      purpose: "输入系统"
    - name: com.unity.2d.tilemap
      version: "^1.0"
      purpose: "2D 地图编辑"
  optional:
    - name: com.unity.cinemachine
      version: "^2.9"
      purpose: "摄像机管理"

entry_points:
  main: Assets/Scripts/GameManager.cs
  config: ProjectSettings/ProjectSettings.asset

directories:
  Assets/: "游戏资源"
  Assets/Scripts/: "脚本代码"
  Assets/Scenes/: "场景文件"
  ProjectSettings/: "项目设置"
```

---

## 微服务

```yaml
name: my-platform
description: "电商平台微服务架构"

tech_stack:
  language: typescript
  runtime: node@20
  framework: express@4
  database: postgresql@16
  orm: prisma
  style: none

dependencies:
  critical:
    - name: kafkajs
      version: "^2.2"
      purpose: "消息队列"
    - name: ioredis
      version: "^5.3"
      purpose: "缓存"
  optional:
    - name: jaeger-client
      version: "^1.0"
      purpose: "分布式追踪"

entry_points:
  main: gateway/src/main.ts
  config: infra/k8s/

directories:
  services/: "各微服务"
  gateway/: "API 网关"
  infra/: "基础设施配置"
```

---

## 桌面应用 (Electron/Tauri)

```yaml
name: my-desktop
description: "Markdown 编辑器，支持实时预览和插件"

tech_stack:
  language: typescript
  runtime: electron@28
  framework: electron
  database: sqlite
  orm: none
  style: tailwindcss

dependencies:
  critical:
    - name: electron
      version: "^28.0"
      purpose: "桌面应用框架"
    - name: better-sqlite3
      version: "^11.0"
      purpose: "本地数据库"
  optional:
    - name: electron-updater
      version: "^6.0"
      purpose: "自动更新"

entry_points:
  main: src/main/index.ts
  config: src/main/config.ts

directories:
  src/main/: "主进程"
  src/renderer/: "渲染进程"
  src/shared/: "共享类型"
```

---

## 编译器/语言工具

```yaml
name: my-lang
description: "玩具语言编译器，支持函数和模式匹配"

tech_stack:
  language: rust
  runtime: 无
  framework: 无
  database: none
  orm: none
  style: none

dependencies:
  critical:
    - name: logos
      version: "^0.14"
      purpose: "词法分析"
    - name: inkwell
      version: "^0.5"
      purpose: "LLVM IR 生成"
  optional:
    - name: clap
      version: "^4.5"
      purpose: "CLI 参数解析"

entry_points:
  main: src/main.rs
  config: Cargo.toml

directories:
  src/lexer/: "词法分析"
  src/parser/: "语法分析"
  src/ast/: "AST 定义"
  src/codegen/: "代码生成"
  tests/: "测试用例"
```

---

## 库/SDK (npm)

```yaml
name: my-sdk
description: "类型安全的 HTTP 客户端 SDK"

tech_stack:
  language: typescript
  runtime: node@18
  framework: none
  database: none
  orm: none
  style: none

dependencies:
  critical:
    - name: axios
      version: "^1.7"
      purpose: "HTTP 请求"
  optional:
    - name: zod
      version: "^3.23"
      purpose: "运行时类型校验"

entry_points:
  main: src/index.ts
  config: tsconfig.json

directories:
  src/: "源代码"
  src/types/: "类型定义"
  src/client/: "客户端实现"
  tests/: "测试"
```

---

## 基础设施 (Terraform + K8s)

```yaml
name: my-infra
description: "AWS EKS 集群 + RDS + Redis 基础设施"

tech_stack:
  language: hcl
  runtime: 无
  framework: terraform
  database: none
  orm: none
  style: none

dependencies:
  critical:
    - name: hashicorp/aws
      version: "~> 5.0"
      purpose: "AWS Provider"
    - name: hashicorp/kubernetes
      version: "~> 2.27"
      purpose: "K8s Provider"
  optional:
    - name: hashicorp/helm
      version: "~> 2.12"
      purpose: "Helm Chart 部署"

entry_points:
  main: main.tf
  config: variables.tf

directories:
  modules/: "自定义模块"
  environments/: "环境配置 (dev/staging/prod)"
  charts/: "Helm Charts"
```
