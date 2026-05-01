# 05 WORKFLOW.md：仓库契约与热重载

`WORKFLOW.md` 是 Symphony 的**单一事实来源**：前半（可选）YAML 配置运行时，后半 Markdown 是发给 Codex 的 **Prompt 模板**。规范见 [SPEC.md](../SPEC.md) 第 5–6 节；本仓库示例见 [elixir/WORKFLOW.md](../elixir/WORKFLOW.md)。

## 文件发现与路径

1. **显式路径**：CLI 第一个参数传入的 `WORKFLOW.md` 绝对路径（经 [CLI.run/2](../elixir/lib/symphony_elixir/cli.ex) `Path.expand` 后写入 `Application` 环境）。
2. **默认**：当前工作目录下的 `./WORKFLOW.md`（与 SPEC 一致）。

实现：[Workflow.workflow_file_path/0](../elixir/lib/symphony_elixir/workflow.ex)。

## 文件格式

- 若以 `---` 开头：第一个 `---` 与第二个 `---` 之间为 **YAML front matter**；其后全部为 **Prompt 正文**。
- 若无 front matter：整文件视为 Prompt，`config` 视为空 map。
- YAML 根必须是 **map**；否则报错 `workflow_front_matter_not_a_map`。

解析：[Workflow.load/1](../elixir/lib/symphony_elixir/workflow.ex) `parse/1`。

## 顶层配置键（SPEC 5.3 + Elixir 扩展）

SPEC 规定的核心键：`tracker`、`polling`、`workspace`、`hooks`、`agent`、`codex`。

本实现额外嵌入（见 [Config.Schema](../elixir/lib/symphony_elixir/config/schema.ex)）：

| 键 | 用途 |
|----|------|
| `worker` | `ssh_hosts`、`max_concurrent_agents_per_host`（SSH 扩展，SPEC 附录 A） |
| `observability` | 终端仪表盘 `dashboard_enabled`、`refresh_ms`、`render_interval_ms` |
| `server` | HTTP 服务 `port`、`host`（与 CLI `--port` 覆盖关系见 [11-getting-started.md](11-getting-started.md)） |

未知键：SPEC 建议忽略以向前兼容；具体以 Schema `cast` 行为为准。

## 各段字段摘要

### `tracker`

- `kind`：`linear`（规范）；实现另有 `memory` 供测试。
- `endpoint`：默认 `https://api.linear.app/graphql`。
- `api_key`：字面量或 `$VAR`；Linear 规范下常用 `LINEAR_API_KEY`。
- `project_slug`：**Linear 下必填**（对应项目 `slugId`）。
- `active_states` / `terminal_states`：字符串列表，与 Linear 工作流状态名**大小写需一致**（对账时多会 normalize，见代码 `Schema.normalize_issue_state`）。

### `polling`

- `interval_ms`：默认 `30000`；热重载后影响后续 tick 间隔（SPEC 6.2）。

### `workspace`

- `root`：默认 `<系统临时目录>/symphony_workspaces`；支持 `~` 与 `$VAR`；相对路径相对 **WORKFLOW.md 所在目录** 解析（SPEC 6.1）。

### `hooks`

- `after_create` / `before_run` / `after_run` / `before_remove`：多行 shell 字符串。
- `timeout_ms`：默认 `60000`。

失败语义见 [09-workspace-safety.md](09-workspace-safety.md)。

### `agent`

- `max_concurrent_agents`：全局并发上限。
- `max_turns`：单次 worker 内最多连续 Codex Turn 数（默认 20）。
- `max_retry_backoff_ms`：失败重试指数上限（默认 5 分钟）。
- `max_concurrent_agents_by_state`：`状态名(小写) -> 正整数` 覆盖该状态下的槽位。

### `codex`

- `command`：默认 `codex app-server`；经 `bash -lc` 在**工作区目录**执行（SPEC 10.1）。
- `approval_policy`、`thread_sandbox`、`turn_sandbox_policy`：透传给 Codex；**省略时的安全默认值**见 [elixir/README.md](../elixir/README.md)「Configuration」节与 Schema 中 `Codex` embed 默认值。
- `turn_timeout_ms` / `read_timeout_ms` / `stall_timeout_ms`：`stall_timeout_ms <= 0` 关闭 stall 检测（SPEC 8.5）。

## Prompt 模板（Markdown 体）

- 引擎：**Solid**（Liquid 风格），且 **严格变量/过滤器**（[PromptBuilder](../elixir/lib/symphony_elixir/prompt_builder.ex) `strict_variables: true`）。
- 可用变量：`issue`（结构体转字符串键 map）、`attempt`（首跑 `nil`）。
- 正文为空：使用 [Config](../elixir/lib/symphony_elixir/config.ex) 内建默认模板（含 identifier/title/description）。

本仓库示例 [elixir/WORKFLOW.md](../elixir/WORKFLOW.md) 还包含大量**团队流程**（Linear 状态机、workpad 评论、`land` 技能等），复制到自己项目时应按需删减。

## 配置解析管道（SPEC 6.1）

1. 选定 workflow 文件路径。  
2. 解析 YAML → 原始 map。  
3. 缺省字段补默认。  
4. 仅对显式写出 `$VAR` 的值做环境展开（非全局 env 覆盖 YAML）。  
5. 类型强制与校验（Elixir：`Config.Schema.parse/1`）。

## 动态重载（SPEC 6.2）

[WorkflowStore](../elixir/lib/symphony_elixir/workflow_store.ex) 以约 **1s** 间隔检查文件 mtime；变更时重新 `Workflow.load/1`：

- **成功**：新配置应用于后续派发、重试、Hook、新会话；**已在跑的 Codex 会话**不要求自动重启（SPEC 允许）。
- **失败**：记录错误，**保留上次有效配置**，服务不崩溃。

Orchestrator 在 tick 内也会 `refresh_runtime_config`（防御性重读，防漏掉事件）。

## 派发前校验（SPEC 6.3）

包括但不限于：workflow 可读、`tracker.kind` 支持、`api_key` 非空、`project_slug`（Linear）、`codex.command` 非空。失败时：**启动失败** 或 **该 tick 跳过派发**（仍可对账）。

## 最小示例（教学用）

```yaml
---
tracker:
  kind: linear
  project_slug: "your-project-slug"
workspace:
  root: ~/symphony-workspaces
agent:
  max_concurrent_agents: 2
  max_turns: 10
codex:
  command: codex app-server
---
You are working on {{ issue.identifier }}: {{ issue.title }}.
```

将 `project_slug` 换成你的 Linear 项目；并设置环境变量 `LINEAR_API_KEY`。

## 下一篇

- [06-orchestrator-state.md](06-orchestrator-state.md)：编排状态机与重试。
- [09-workspace-safety.md](09-workspace-safety.md)：Hook 与路径安全细节。
