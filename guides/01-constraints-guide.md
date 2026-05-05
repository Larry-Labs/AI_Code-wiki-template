# 01-constraints.md 填写指南

> 本指南展示各项目类型的约束指标示例。
> 对应模板：`template/01-constraints.md`

---

## Web 后端

### 性能

- 响应时间：API P99 < 200ms
- 并发量：支持 1000 并发用户
- 数据量：预计 100 万条记录

### 安全

- 认证方式：JWT + refresh token
- 数据加密：传输层 TLS 1.3，存储层 AES-256
- 合规要求：OWASP Top 10

### 运行环境

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux (Docker) |
| 最低配置 | 2C4G |
| 网络 | 需要公网 |
| 分发方式 | Docker |

---

## Web 前端

### 性能

- 首屏加载：< 3s（LCP）
- 交互响应：< 100ms（FID）
- 包大小：< 200KB（gzipped）

### 兼容性

| 项目 | 要求 |
|------|------|
| 浏览器 | Chrome 90+, Safari 16+, Firefox 100+ |
| 移动端 | iOS 16+, Android 12+ |
| 分发方式 | Vercel / Docker |

---

## CLI 工具 (Go)

### 性能

- 启动时间：< 100ms
- 内存占用：< 50MB

### 兼容性

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux, macOS, Windows |
| 分发方式 | brew, direct download |

### 遵循标准

- [ ] POSIX (CLI 兼容性)

---

## 嵌入式

### 硬件约束

| 项目 | 规格 |
|------|------|
| MCU | STM32L071 |
| Flash | 192KB |
| RAM | 20KB |
| 时钟 | 32MHz HSE |
| 功耗预算 | < 100uA 平均（电池供电） |
| 工作温度 | -40°C ~ 85°C |
| 外设 | I2C x1, SPI x1, UART x1, GPIO x4 |

### 性能

- 采集周期：30 秒
- 发送延迟：< 500ms
- 电池寿命：> 1 年

### 遵循标准

- [ ] IEEE 802.15.4 (无线协议)
- [ ] MISRA C:2012 (编码规范)

### 嵌入式 — AUTOSAR Classic Platform

#### 硬件约束

| 项目 | 规格 |
|------|------|
| MCU | Infineon TC397 / NXP S32K344 |
| Flash | 4MB（BSW 栈 ~500KB，应用 ~1MB，OTA 双分区） |
| RAM | 512KB（BSW ~64KB，应用 ~128KB） |
| CAN 通道 | 3x CAN-FD + 1x LIN |
| 功耗预算 | < 5mA 正常运行，< 100μA 休眠 |
| 工作温度 | -40°C ~ 125°C（车规 Grade 0） |

#### 性能

- CAN 报文周期：10ms（底盘域），100ms（车身域）
- 诊断响应时间：< 50ms（UDS 0x22 读 DID）
- NM 网络唤醒时间：< 100ms
- Bootloader 刷写时间：< 5 分钟（完整 ECU）

#### 遵循标准

- [ ] ISO 26262 (功能安全，ASIL-B/D)
- [ ] ISO 14229 (UDS 统一诊断服务)
- [ ] ISO 11898 (CAN 总线协议)
- [ ] AUTOSAR Classic Platform R22-11
- [ ] MISRA C:2012 (编码规范，含 Amendment 2)
- [ ] SAE J1939 (商用车通信协议，如适用)

---

## iOS 应用

### 性能

- 启动时间：冷启动 < 2s
- 帧率：60fps（列表滚动不掉帧）
- 包大小：< 50MB（不含资源）

### 兼容性

| 项目 | 要求 |
|------|------|
| 系统版本 | iOS 16+ |
| 设备 | iPhone XS 及以上 |
| 分发方式 | App Store |

### 遵循标准

- [ ] Apple HIG (设计规范)

---

## Android 应用

### 性能

- 启动时间：冷启动 < 2s
- 帧率：60fps
- 包大小：< 50MB

### 兼容性

| 项目 | 要求 |
|------|------|
| 系统版本 | Android 12+ (API 31) |
| 分发方式 | Play Store |

### 遵循标准

- [ ] Material Design 3

---

## 数据管道

### 性能

- 吞吐量：100万条/分钟
- 端到端延迟：< 5min
- 数据量：日处理 10GB

### 可靠性

- SLA：99.9%
- 失败重试：3 次，间隔 5min

---

## ML/AI

### 性能

- 训练时间：< 8h（单卡 A100）
- 推理延迟：< 100ms（单样本）
- 模型大小：< 500MB

### 硬件

| 项目 | 要求 |
|------|------|
| GPU | NVIDIA A100 / V100 |
| 显存 | >= 16GB |
| 存储 | >= 100GB SSD |

### 遵循标准

- [ ] ONNX (模型格式互操作)

---

## 游戏

### 性能

- 帧率：60fps @ 1080p
- 加载时间：< 5s
- 内存预算：< 2GB

### 平台

| 项目 | 要求 |
|------|------|
| 目标平台 | Windows, macOS |
| 最低配置 | GTX 1060 / 8GB RAM |

### 遵循标准

- [ ] OpenGL 4.5 / Vulkan 1.3

---

## 桌面应用

### 性能

- 启动时间：< 2s
- 内存占用：< 200MB
- 包大小：< 100MB

### 运行环境

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows 10+, macOS 12+, Ubuntu 20+ |
| 最低配置 | 4GB RAM |
| 分发方式 | 直接下载 / App Store / Homebrew |

### 遵循标准

- [ ] 各平台 UI 规范（HIG / Material / Fluent）

---

## 编译器/语言工具

### 性能

- 编译速度：< 1s（1000 行源码）
- 内存占用：< 500MB

### 运行环境

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux, macOS, Windows |
| 分发方式 | npm / pip / cargo / brew |

### 遵循标准

- [ ] 语言规范版本（如 ECMAScript 2024 / C17）

---

## 库/SDK

### 性能

- 包大小：< 50KB（gzipped）
- 无运行时开销（零成本抽象优先）

### 运行环境

| 项目 | 要求 |
|------|------|
| 操作系统 | 跨平台 |
| 分发方式 | npm / pip / cargo / Maven Central |

### 遵循标准

- [ ] SemVer（语义化版本）
- [ ] 向后兼容性承诺

---

## 基础设施

### 性能

- 部署时间：< 10min
- 回滚时间：< 5min

### 运行环境

| 项目 | 要求 |
|------|------|
| 目标平台 | AWS / GCP / Azure |
| 工具版本 | Terraform >= 1.7, K8s >= 1.28 |
| 分发方式 | Terraform Registry / Helm Chart |

### 遵循标准

- [ ] Well-Architected Framework
- [ ] CIS Benchmarks

---

## 微服务

### 性能

- 服务间延迟：< 50ms（P99）
- 可用性：99.95%

### 运行环境

| 项目 | 要求 |
|------|------|
| 编排平台 | Kubernetes |
| 服务网格 | Istio / Linkerd（可选） |
| 消息队列 | Kafka / RabbitMQ |
| 分发方式 | Docker + Helm |

### 遵循标准

- [ ] 12-Factor App
- [ ] OpenTelemetry（可观测性）
