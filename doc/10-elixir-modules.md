# 10 Elixir 模块地图

以下按**职责**分组，路径均相对于 [elixir/lib](../elixir/lib)。`@spec` 要求与 PR 规范见 [elixir/AGENTS.md](../elixir/AGENTS.md)。

## 应用与入口

| 模块 | 类型 | 职责 |
|------|------|------|
| [symphony_elixir.ex](../elixir/lib/symphony_elixir.ex) `SymphonyElixir` | API | `start_link` → 转调 Orchestrator |
| 同上 `SymphonyElixir.Application` | Application | 监督树子项启动顺序、`stop/2` 时仪表盘 offline |
| [symphony_elixir/cli.ex](../elixir/lib/symphony_elixir/cli.ex) | Escript 主模块 | 解析 CLI、守门提示、`ensure_all_started` |

## 编排与状态

| 模块 | 类型 | 职责 |
|------|------|------|
| [orchestrator.ex](../elixir/lib/symphony_elixir/orchestrator.ex) | GenServer | Poll、派发、重试、对账、聚合指标 |
| [agent_runner.ex](../elixir/lib/symphony_elixir/agent_runner.ex) | 纯模块（异步由 Task 调） | 单工单：工作区 → Hook → 多 Turn Codex |
| [status_dashboard.ex](../elixir/lib/symphony_elixir/status_dashboard.ex) | GenServer | 终端 UI 渲染、吞吐采样 |
| [workflow_store.ex](../elixir/lib/symphony_elixir/workflow_store.ex) | GenServer | `WORKFLOW.md` 轮询热加载 |

## 配置与工作流

| 模块 | 职责 |
|------|------|
| [workflow.ex](../elixir/lib/symphony_elixir/workflow.ex) | 读文件、拆 front matter、YAML 解析 |
| [config.ex](../elixir/lib/symphony_elixir/config.ex) | 从 Workflow 取配置、默认 Prompt |
| [config/schema.ex](../elixir/lib/symphony_elixir/config/schema.ex) | Ecto embed 校验、路径/策略 finalize |

## 执行与 Codex

| 模块 | 职责 |
|------|------|
| [workspace.ex](../elixir/lib/symphony_elixir/workspace.ex) | 创建/复用目录、Hooks、远端脚本 |
| [path_safety.ex](../elixir/lib/symphony_elixir/path_safety.ex) | 路径规范化与前缀校验 |
| [ssh.ex](../elixir/lib/symphony_elixir/ssh.ex) | 本地 `ssh` 可执行查找、参数拼装、`Port` |
| [prompt_builder.ex](../elixir/lib/symphony_elixir/prompt_builder.ex) | Solid 严格模板渲染 |
| [codex/app_server.ex](../elixir/lib/symphony_elixir/codex/app_server.ex) | JSON-RPC、会话/turn、事件解析 |
| [codex/dynamic_tool.ex](../elixir/lib/symphony_elixir/codex/dynamic_tool.ex) | `linear_graphql` 工具执行 |

## Linear 与 Tracker

| 模块 | 职责 |
|------|------|
| [tracker.ex](../elixir/lib/symphony_elixir/tracker.ex) | behaviour 门面 + adapter 选择 |
| [linear/adapter.ex](../elixir/lib/symphony_elixir/linear/adapter.ex) | Linear 实现 Tracker 回调 |
| [linear/client.ex](../elixir/lib/symphony_elixir/linear/client.ex) | Req HTTP、GraphQL 文档、分页 |
| [linear/issue.ex](../elixir/lib/symphony_elixir/linear/issue.ex) | 归一化结构体 |
| [tracker/memory.ex](../elixir/lib/symphony_elixir/tracker/memory.ex) | 内存测试适配器 |

## 观测与 HTTP

| 模块 | 职责 |
|------|------|
| [http_server.ex](../elixir/lib/symphony_elixir/http_server.ex) | 条件启动 Phoenix Endpoint（Bandit） |
| [log_file.ex](../elixir/lib/symphony_elixir/log_file.ex) | 日志文件后端配置 |
| [symphony_elixir_web/endpoint.ex](../elixir/lib/symphony_elixir_web/endpoint.ex) | Phoenix 端点 |
| [symphony_elixir_web/router.ex](../elixir/lib/symphony_elixir_web/router.ex) | LiveView `/` 与 `/api/v1/*` |
| [symphony_elixir_web/live/dashboard_live.ex](../elixir/lib/symphony_elixir_web/live/dashboard_live.ex) | 仪表盘 LiveView |
| [symphony_elixir_web/controllers/observability_api_controller.ex](../elixir/lib/symphony_elixir_web/controllers/observability_api_controller.ex) | JSON API |
| [symphony_elixir_web/observability_pubsub.ex](../elixir/lib/symphony_elixir_web/observability_pubsub.ex) | PubSub 辅助 |
| [symphony_elixir_web/presenter.ex](../elixir/lib/symphony_elixir_web/presenter.ex) | 展示层数据整形 |

## Mix 任务与质量

| 路径 | 用途 |
|------|------|
| [mix/tasks/specs.check.ex](../elixir/lib/mix/tasks/specs.check.ex) | 规范/静态检查（`mix specs.check`） |
| [mix/tasks/pr_body.check.ex](../elixir/lib/mix/tasks/pr_body.check.ex) | PR 模板校验 |
| [mix/tasks/workspace.before_remove.ex](../elixir/lib/mix/tasks/workspace.before_remove.ex) | 删除工作区前钩子调用的 Mix 任务 |
| [specs_check.ex](../elixir/lib/symphony_elixir/specs_check.ex) | 与 specs 检查相关逻辑 |

## 测试目录提示

- `elixir/test/symphony_elixir/*_test.exs`：按模块行为测试。  
- `elixir/test/support/*`：测试支撑与 live e2e Docker 配置。

## 下一篇

- [11-getting-started.md](11-getting-started.md)：从安装到第一条命令。  
- [12-operations-and-debug.md](12-operations-and-debug.md)：HTTP 与排障。
