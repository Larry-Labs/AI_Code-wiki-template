# AI Code Wiki

一套面向 AI Agent 的代码文档规范。目标：**任何 AI 拿到这套文档，不看源码，就能重建出功能等价的代码。**

## 核心理念

传统文档给人看，省略"显而易见"的东西。
AI Code Wiki 给 AI 看，**必须无歧义、无遗漏、自包含。**

## 两层分离

本模板采用 **template + guides** 两层结构：

| 层 | 目录 | 用途 | 特点 |
|----|------|------|------|
| 模板层 | `template/` | 纯结构，零领域示例 | 40-80 行/文件，直接复制填写 |
| 指南层 | `guides/` | 各项目类型的填写示例 | 按需查阅，不复制到项目中 |

**使用方式**：复制 `template/` 到你的项目，改名为 `ai-code-wiki/`，参照 `guides/` 中的示例填写。

## 目录结构

```
ai-code-wiki-template/
├── README.md               # 本文件
├── CLAUDE.md               # Agent 指令
├── template/               # 纯结构模板（复制到项目中）
│   ├── 00-manifest.yaml    # 项目身份
│   ├── 01-constraints.md   # 约束条件
│   ├── 02-architecture.md  # 架构设计
│   ├── 03-interfaces.md    # 接口定义
│   ├── 04-data.md          # 数据结构
│   ├── 05-logic.md         # 业务逻辑
│   ├── 06-config.md        # 配置部署
│   ├── 07-conventions.md   # 代码约定
│   ├── 08-file-specs/      # 文件规格
│   │   └── _example.md     # 规格模板
│   └── 09-rebuild.md       # 重建指南
├── guides/                 # 填写指南（按需查阅）
│   ├── 00-manifest-guide.md
│   ├── 01-constraints-guide.md
│   ├── 02-architecture-guide.md
│   ├── 03-interfaces-guide.md
│   ├── 04-data-guide.md
│   ├── 05-logic-guide.md
│   ├── 06-config-guide.md
│   ├── 07-conventions-guide.md
│   ├── 08-file-specs-guide.md
│   └── 09-rebuild-guide.md
└── .gitignore
```

## 六层抽象

任何软件系统都在回答六个问题：

| 层 | 问题 | 文件 |
|----|------|------|
| 约束 | 什么不能做？ | `01-constraints.md` |
| 模块 | 系统怎么切分？ | `02-architecture.md` |
| 接口 | 组件怎么通信？ | `03-interfaces.md` |
| 数据 | 信息怎么表示？ | `04-data.md` |
| 逻辑 | 行为怎么执行？ | `05-logic.md` |
| 构建 | 怎么组装运行？ | `06-config.md` + `09-rebuild.md` |

## 适用领域

同一套规范，不同领域填充不同内容。`guides/` 覆盖以下项目类型：

- **Web 后端**：HTTP API、JWT、数据库、Docker
- **Web 前端**：路由、状态管理、组件、构建优化
- **CLI 工具**：参数解析、输出格式化、交叉编译
- **嵌入式**：I2C/SPI、RTOS、功耗优化、Flash 布局
- **iOS**：SwiftUI、URLSession、CoreData、App Store
- **Android**：Compose、Room、Retrofit、Play Store
- **桌面应用**：Electron/Tauri、本地存储
- **数据管道**：ETL、Airflow、Spark
- **ML/AI**：Dataset、Model、训练循环、推理服务
- **游戏**：游戏循环、物理、ECS、网络同步
- **编译器**：前端、IR、后端
- **库/SDK**：公共 API、内部实现
- **基础设施**：Terraform、K8s、Helm
- **微服务**：服务拆分、API 网关、消息队列

## 使用方式

### 快速开始

```bash
# 1. 复制模板到你的项目
cp -r template/ /path/to/your/project/ai-code-wiki/

# 2. 按推荐顺序填写（见下方）

# 3. 参照 guides/ 中的示例
#    例如填写 constraints 时，参考 guides/01-constraints-guide.md
```

### 推荐填写顺序

| 顺序 | 文件 | 原因 | 参考指南 |
|------|------|------|----------|
| 1 | `00-manifest.yaml` | 30 秒搞定，AI 第一眼就知道项目是什么 | `guides/00-manifest-guide.md` |
| 2 | `01-constraints.md` | 约束决定设计空间，必须先写 | `guides/01-constraints-guide.md` |
| 3 | `02-architecture.md` | 理解全局才能写好局部 | `guides/02-architecture-guide.md` |
| 4 | `08-file-specs/` | 核心工作量，逐文件写规格 | `guides/08-file-specs-guide.md` |
| 5 | `03-interfaces.md` | 模块间接口往往在写 file-specs 时才能确定 | `guides/03-interfaces-guide.md` |
| 6 | `04-data.md` | 数据结构同理 | `guides/04-data-guide.md` |
| 7 | `05-logic.md` | 业务流程依赖接口和数据的定义 | `guides/05-logic-guide.md` |
| 8 | `09-rebuild.md` | 所有细节确定后，才能写出正确的重建顺序 | `guides/09-rebuild-guide.md` |
| 9 | `06-config.md` | 构建和部署细节，优先级最低 | `guides/06-config-guide.md` |
| 10 | `07-conventions.md` | 约定往往在代码写完后才明确 | `guides/07-conventions-guide.md` |

**核心投入在 `08-file-specs/`**，这是工作量最大、价值最高的部分。其他文件可以从 file-specs 中提炼。

## Agent 指南

`CLAUDE.md` 是给 AI Agent 的指令文件。如果你用 Claude Code / Cursor / Codex 等工具，把 `CLAUDE.md` 放在项目根目录，Agent 会自动读取并按照规范工作。

## 灵感来源

本规范受 [Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 启发——用结构化的 Markdown 文件作为 AI 的知识载体。区别在于：Karpathy 的 Wiki 面向通用知识，本规范面向**代码重建**。

## 设计原则

1. **约束优先**：先知道什么不能做，再设计怎么做
2. **接口精确**：函数签名、协议格式必须精确到可直接翻译成代码
3. **流程自包含**：每个逻辑流程不引用隐含上下文
4. **关键细节不省**：不抄代码，但写"地址要左移"这种 AI 会踩坑的点
5. **重建有顺序**：底层先于上层，定义先于实现
