# 12 运维与排障：仪表盘、HTTP API、日志与 SSH

## 终端仪表盘（StatusDashboard）

[StatusDashboard](../elixir/lib/symphony_elixir/status_dashboard.ex) 在启用时于 stderr/控制台绘制**运行中工单、重试队列、token 吞吐、最近事件**等。刷新与节流由 `observability` 配置段控制（见 [Config.Schema Observability](../elixir/lib/symphony_elixir/config/schema.ex)）。

若 HTTP 未开，终端 UI 仍是主要「现场观测」手段之一。

## HTTP 观测栈

[HttpServer](../elixir/lib/symphony_elixir/http_server.ex) 在端口有效时启动 [Endpoint](../elixir/lib/symphony_elixir_web/endpoint.ex)，使用 **Bandit** 承载 Phoenix。

### 路由（见 [router.ex](../elixir/lib/symphony_elixir_web/router.ex)）

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/` | LiveView 仪表盘 |
| GET | `/api/v1/state` | 全局快照 JSON |
| GET | `/api/v1/:issue_identifier` | 单工单调试信息 |
| POST | `/api/v1/refresh` | 触发一次 poll/reconcile（202，可能合并重复请求） |
| GET | `/dashboard.css` 等 | 静态资源 |

未实现的动词返回 **405**；未知路径 **404** JSON envelope（见 `ObservabilityApiController`）。

### 绑定地址

`server.host` 默认 `127.0.0.1`（Schema）；生产若改 `0.0.0.0` 需自行网络 ACL / 认证——当前设计为**本地观测**。

## 日志

规范与字段要求见：

- [elixir/docs/logging.md](../elixir/docs/logging.md)

**排障最小集合**：`issue_id` + `issue_identifier` +（若有）`session_id`。全文检索某工单时三路一起搜。

日志根目录可通过 CLI `--logs-root` 调整（见 [11-getting-started.md](11-getting-started.md)）。

## Token 与限流展示

聚合与线程级语义见：

- [elixir/docs/token_accounting.md](../elixir/docs/token_accounting.md)

若仪表盘上「累计 token」异常跳变，先对照该文档检查是否误把 `usage` delta 当 total。

## SSH Worker 排障清单

1. **本机** `ssh` 在 PATH；`ssh user@host` 免交互或已配 agent。  
2. **远端** `codex` 与仓库依赖齐全；`workspace.root` 在远端存在且可写。  
3. **认证**：远端需能访问 Linear / GitHub 等——常见做法是把 `~/.codex/auth.json` 挂到 worker（README e2e 描述）。  
4. **一致性**：同一工单重试应固定在同一 host（实现注释：`AgentRunner` 由 Orchestrator 侧 host 重试策略约束）。

## 常见故障模式

| 现象 | 可能原因 | 建议 |
|------|----------|------|
| 一直无派发 | `WORKFLOW.md` 校验失败、无 API key、无候选状态 | 看启动日志；临时 `GET /api/v1/state` 看 validation 提示 |
| 工单反复重试 | Codex 启动失败、Hook 失败、stall | 搜 `issue_identifier`；读 `after_create`/`before_run` 输出；调 `stall_timeout_ms` |
| 槽位爆满 | `max_concurrent_agents` 过小或按状态限制 | 调 workflow 或缩短长任务 |
| Linear 429/5xx | 上游限流 | 调大 `polling.interval_ms`；减并发 |
| PR/land 不前进 | 属于 Prompt 与技能层 | 读你 fork 的 `WORKFLOW.md` 与 `.codex/skills/*` |

## 与 Cursor / Codex Skills 的关系

仓库内 [.cursor/skills](../.cursor/skills) 与 [.codex/skills](../.codex/skills) 提供 **commit / push / pull / land / linear** 等技能文本；Symphony 运行时代码在 `elixir/lib`。学习时区分：**编排服务** vs **代理侧技能文档**。

## 回读索引

- 协议全文：[SPEC.md](../SPEC.md)  
- 概念词典：[03-core-concepts.md](03-core-concepts.md)  
- 端到端时序：[04-end-to-end-flow.md](04-end-to-end-flow.md)
