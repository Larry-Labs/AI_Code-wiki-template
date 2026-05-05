# 01 — 约束

> 做任何事之前必须知道的边界。约束决定了设计空间的大小。
> 根据项目类型填写对应章节，不适用的章节直接删除。
> 参考 guides/01-constraints-guide.md 查看各项目类型的填写示例。

## 项目类型

<!-- 勾选你的项目类型，删除不适用的章节 -->

- [ ] Web 前端（React/Vue/Angular/Svelte）
- [ ] Web 后端（Express/Django/Spring/Gin）
- [ ] CLI 工具（Go/Rust/Python/Node）
- [ ] 嵌入式（STM32/ESP32/nRF/RISC-V）
- [ ] iOS（Swift/ObjC，Xcode）
- [ ] Android（Kotlin/Java，Android Studio）
- [ ] 桌面应用（Electron/Tauri/Qt/GTK）
- [ ] 数据管道（Airflow/dbt/Spark/Flink）
- [ ] ML/AI（PyTorch/TF 训练/推理）
- [ ] 游戏（Unity/Unreal/Godot）
- [ ] 编译器/语言工具（LLVM/tree-sitter）
- [ ] 库/SDK（npm/cargo/pip crate）
- [ ] 基础设施（Terraform/K8s operator）
- [ ] 微服务（多服务架构）

---

## 非功能需求

<!-- 性能、安全、可用性等约束。根据项目类型选择填写 -->

### 性能

<!-- 根据项目类型填写对应的性能指标，参考 guides/01-constraints-guide.md -->

- <指标>: <目标值>

### 安全

- 认证方式：<JWT / OAuth2 / 无>
- 数据加密：<传输层协议，存储层算法>
- 合规要求：<GDPR / 等保三级 / HIPAA / 无>

### 可用性

- SLA 目标：<99.9% / 不适用>
- 部署方式：<Docker / K8s / App Store / OTA / npm publish>

---

## 运行环境约束

<!-- Web/后端/CLI/桌面/移动项目填写此节 -->

| 项目 | 要求 |
|------|------|
| 操作系统 | <Linux / macOS / Windows / 裸机> |
| 最低配置 | <2C4G / 无限制> |
| 网络 | <需要公网 / 仅局域网 / 离线> |
| 浏览器 | <Chrome 90+ / 不适用> |
| 移动端 | <iOS 16+ / Android 12+ / 不适用> |
| 分发方式 | <App Store / Play Store / npm / pip / brew / 直接下载> |

---

## 硬件约束

<!-- 嵌入式项目填写此节，Web/后端项目删除 -->

| 项目 | 规格 |
|------|------|
| MCU | <型号> |
| Flash | <大小> |
| RAM | <大小> |
| 时钟 | <频率> |
| 功耗预算 | <平均电流> |
| 工作温度 | <温度范围> |
| 外设 | <I2C/SPI/UART/GPIO 数量> |

---

## 遵循的标准/协议

<!-- 项目遵循的行业标准、内部规范，不适用的删除 -->

- [ ] <标准名称及版本>

---

## 已知限制

<!-- 不能做的事情、技术债务、历史包袱 -->

- <限制描述>
