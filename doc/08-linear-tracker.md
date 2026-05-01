# 08 Linear 工单系统接入

## 适配器入口

[SymphonyElixir.Tracker](../elixir/lib/symphony_elixir/tracker.ex) 根据 `Config.settings!().tracker.kind` 选择：

- `"memory"` → `SymphonyElixir.Tracker.Memory`（测试）。
- 其他 → `SymphonyElixir.Linear.Adapter`（默认生产路径）。

## 必需操作（SPEC 11.1）

| 操作 | 用途 |
|------|------|
| `fetch_candidate_issues/0` | 轮询：活动状态下某项目的候选工单 |
| `fetch_issues_by_states/1` | 启动：终态工单列表，用于清理工作区 |
| `fetch_issue_states_by_ids/1` | 对账：批量刷新运行中工单的当前状态 |

Elixir `Linear.Adapter` 将上述委托给 [Linear.Client](../elixir/lib/symphony_elixir/linear/client.ex)。

## GraphQL 查询要点（SPEC 11.2）

- **端点**：默认 `https://api.linear.app/graphql`，`Authorization: <api_key>`。
- **项目过滤**：`project: { slugId: { eq: $projectSlug } }`（注意是 **slugId**，与 Linear UI 里复制的 slug 对应；不清楚时在 Linear 项目 URL 中确认）。
- **分页**：候选拉取必须分页；默认 `first: 50`，循环 `pageInfo.hasNextPage` / `endCursor`。
- **超时**：规范建议网络超时 30000 ms（实现以 Req 客户端为准）。

客户端模块内嵌了长查询字符串 `@query`、`@query_by_ids` 等，字段覆盖 Issue 结构体所需列。

## 归一化（SPEC 11.3）

[Linear.Issue](../elixir/lib/symphony_elixir/linear/issue.ex) 结构体承载：

- `labels`：小写字符串列表。
- `blocked_by`：从 `inverseRelations` 中 `type == "blocks"` 的关系推导。
- `priority`：非整数则变 `nil`。
- 时间：ISO-8601 解析为 `DateTime`。

## 错误语义（SPEC 11.4 风格）

实现将 HTTP/GraphQL/解析问题映射为可日志化的 reason，编排器行为：

- **候选拉取失败**：记录错误，**本 tick 不派发**新工单。
- **运行中刷新失败**：**不杀** worker，下 tick 再试。
- **启动终态清理失败**：警告，**继续启动**。

具体 atom / 字符串以 `Linear.Client` 源码为准。

## Tracker 写能力（实现补充）

`Tracker` behaviour 还包含 `create_comment/2`、`update_issue_state/2`，供测试或未来扩展；**规范主路径**仍强调工单写入由 Agent 工具链完成。若你阅读测试代码看到直接写 Linear，请勿与「生产最小权限」假设混淆。

## 配置 checklist

1. `LINEAR_API_KEY` 或个人 API key 已通过 `$LINEAR_API_KEY` 注入。  
2. `project_slug` 与团队工作流状态名与 `active_states` / `terminal_states` **一致**。  
3. 代理侧有权限操作该项目（读必须；写取决于你的 Prompt/工具）。

## 下一篇

- [05-workflow-md.md](05-workflow-md.md)：`tracker` 段配置。  
- [03-core-concepts.md](03-core-concepts.md)：`blocked_by` 与调度关系。
