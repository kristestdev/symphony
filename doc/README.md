# Symphony 中文学习文档

本目录是 [Symphony](../README.md) 项目的**结构化中文导读**，与仓库根目录的 [SPEC.md](../SPEC.md)（语言无关协议）和 [elixir/README.md](../elixir/README.md)（Elixir 实现说明）配套阅读。

## 文档列表

| 序号 | 文档 | 内容概要 |
|------|------|----------|
| 01 | [01-overview.md](01-overview.md) | 定位、问题域、目标与非目标、信任边界 |
| 02 | [02-architecture.md](02-architecture.md) | 六层抽象、Elixir OTP 监督树、数据流 |
| 03 | [03-core-concepts.md](03-core-concepts.md) | Issue、Workspace、Run、Session、Workflow 等概念词典 |
| 04 | [04-end-to-end-flow.md](04-end-to-end-flow.md) | 从轮询到 Agent 退出的端到端时序 |
| 05 | [05-workflow-md.md](05-workflow-md.md) | `WORKFLOW.md` 格式、YAML 字段、热重载 |
| 06 | [06-orchestrator-state.md](06-orchestrator-state.md) | 编排状态机、调度、重试、对账 |
| 07 | [07-codex-integration.md](07-codex-integration.md) | Codex App-Server、多 Turn、Token、`linear_graphql` |
| 08 | [08-linear-tracker.md](08-linear-tracker.md) | Linear GraphQL、适配器、分页与错误 |
| 09 | [09-workspace-safety.md](09-workspace-safety.md) | 工作区布局、Hooks、路径安全、SSH |
| 10 | [10-elixir-modules.md](10-elixir-modules.md) | `elixir/lib` 模块地图与职责 |
| 11 | [11-getting-started.md](11-getting-started.md) | 环境、mise、运行、最小配置 |
| 12 | [12-operations-and-debug.md](12-operations-and-debug.md) | 仪表盘、HTTP API、日志、排障 |

## 推荐阅读路径

### 路径 A：约 30 分钟快速建立心智模型

适合想先知道「这东西干什么、怎么跑起来」的读者：

1. [01-overview.md](01-overview.md)
2. [02-architecture.md](02-architecture.md)（只看「六层」与「监督树」两节）
3. [11-getting-started.md](11-getting-started.md)
4. （可选）[05-workflow-md.md](05-workflow-md.md) 前半「文件格式与最小示例」

### 路径 B：约 2 小时系统深入

适合要参与开发、改编排逻辑或对接自己仓库的读者：

按编号 **01 → 12** 顺序阅读；其中 **04、06、07、09** 与 SPEC 行为强相关，建议精读。

**穿插阅读仓库原文：**

- 协议细节：[SPEC.md](../SPEC.md)
- 运行与 Codex 默认值：[elixir/README.md](../elixir/README.md)
- 本仓库示例工作流：[elixir/WORKFLOW.md](../elixir/WORKFLOW.md)
- 贡献约定：[elixir/AGENTS.md](../elixir/AGENTS.md)
- 日志与 Token：[elixir/docs/logging.md](../elixir/docs/logging.md)、[elixir/docs/token_accounting.md](../elixir/docs/token_accounting.md)

## 约定

- 文中「见 [某文件](../path)」均相对于本仓库根目录。
- Mermaid 图在支持 Markdown 预览的编辑器中渲染；若渲染失败，可直接阅读图下文字说明。

## 版权声明

Symphony 项目以 [Apache 2.0](../LICENSE) 授权；本 `doc/` 为学习导读，行为以源码与 SPEC 为准。
