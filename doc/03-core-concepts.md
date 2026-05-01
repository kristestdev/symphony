# 03 核心概念词典

本节与 [SPEC.md](../SPEC.md) 第 4 节对齐，并标注 Elixir 中的对应类型或字段。

## Issue（工单）

**含义**：从 Linear 归一化后的记录，供编排、Prompt 渲染、观测使用。

**主要字段**（SPEC 4.1.1；实现见 [Linear.Issue](../elixir/lib/symphony_elixir/linear/issue.ex)）：

| 字段 | 说明 |
|------|------|
| `id` | Linear 内部 UUID，映射键、刷新状态用 |
| `identifier` | 人类可读键，如 `MT-123` |
| `title` / `description` | 标题与描述 |
| `priority` | 数值越小优先级越高（调度排序） |
| `state` | 当前状态名 |
| `branch_name` / `url` | 分支元数据、工单 URL |
| `labels` | 小写字符串列表 |
| `blocked_by` | 阻塞关系列表（含 id/identifier/state） |
| `created_at` / `updated_at` | 时间戳 |

**调度提示**：`Todo` 且存在**非终态**阻塞者时，SPEC 规定不派发（见 [06-orchestrator-state.md](06-orchestrator-state.md)）。

## WorkflowDefinition / 已加载工作流

**含义**：解析 `WORKFLOW.md` 后的结构。

- `config`：YAML front matter 根对象（不是嵌在 `config:` 键下）。
- `prompt_template`：front matter 之后、trim 过的 Markdown 正文。

实现：[Workflow.load/1](../elixir/lib/symphony_elixir/workflow.ex)、`WorkflowStore.current/0`。

## Service Config（类型化运行时配置）

**含义**：从 `config` 经默认值、`$VAR` 展开、路径归一化、Ecto changeset 校验得到的结构体视图。

实现：[SymphonyElixir.Config.Schema](../elixir/lib/symphony_elixir/config/schema.ex)，访问入口 `Config.settings!/0`。

## Workspace（工作区）

**含义**：分配给**单个** `issue.identifier` 的目录。

- **逻辑字段**：绝对路径 `path`、`workspace_key`（由 identifier 净化）、`created_now`（是否本次新建，用于是否跑 `after_create`）。

路径规则见 SPEC 9.1：`workspace.root` + 净化后的目录名。

## Run Attempt（一次执行尝试）

**含义**：某工单的一次 Agent 运行，从准备工作区到进程退出。

阶段名见 SPEC 7.2（如 `PreparingWorkspace`、`StreamingTurn`、`Succeeded` 等）；日志与排障时应对齐这些语义。

## Live Session（Codex 会话元数据）

**含义**：子进程存活期间跟踪的 thread/turn、PID、最后事件、Token 累计等。

SPEC 4.1.6 列有 `session_id`（`<thread_id>-<turn_id>`）、各类 token 计数、`turn_count` 等。实现中由 Orchestrator 的 `running` 表项维护。

## Retry Entry（重试队列项）

**含义**：工单暂时不能跑或失败后，计划在 `due_at_ms` 再次尝试。

- **连续完成**：正常退出后短延迟（实现为约 **1s**）再拉一次，看工单是否仍在活动态。
- **失败**：指数退避，基数与 cap 见 SPEC 与 [06-orchestrator-state.md](06-orchestrator-state.md)。

## Orchestrator 内存状态（权威）

SPEC 4.1.8 概括：

- `running`：`issue_id ->` 运行中条目  
- `claimed`：已占位（运行中或重试中）  
- `retry_attempts`：重试定时器与元数据  
- `completed`： bookkeeping，不单独作为派发门闩  
- `codex_totals` / `codex_rate_limits`：聚合用量与限流快照  

实现：`Orchestrator.State` 结构体。

## Claim（认领）与「编排状态」

SPEC 第 7.1 节的 **Unclaimed / Claimed / Running / RetryQueued / Released** 是服务内部认领状态，**不要**与 Linear 的 `Todo`、`In Progress` 混淆。

## Prompt 模板变量

SPEC 5.4 / 实现 [PromptBuilder](../elixir/lib/symphony_elixir/prompt_builder.ex)：

- `issue`：工单对象（键转为字符串，嵌套 map/list 保留，供 Solid 模板使用）。
- `attempt`：首次为 `null`，重试/续跑时为整数。

模板引擎为 **Solid**，且 `strict_variables` / `strict_filters` 为真：未知变量或过滤器会直接失败该次尝试。

## Tracker 与 Adapter

**Tracker** 行为接口：[SymphonyElixir.Tracker](../elixir/lib/symphony_elixir/tracker.ex)（`fetch_candidate_issues` 等）。

生产路径一般为 **Linear**；测试可使用 `memory` 适配器（`tracker.kind == "memory"` 时）。

## Hook（工作区脚本）

四个生命周期钩子：`after_create`、`before_run`、`after_run`、`before_remove`。语义见 SPEC 9.4 与 [09-workspace-safety.md](09-workspace-safety.md)。

## 下一篇

- [04-end-to-end-flow.md](04-end-to-end-flow.md)：把上述概念串成一次完整运行。
- [05-workflow-md.md](05-workflow-md.md)：`WORKFLOW.md` 字段详解。
