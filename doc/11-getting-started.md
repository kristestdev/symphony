# 11 上手：从环境到第一次运行

## 前置假设

- 已安装 [mise](https://mise.jdx.dev/)（推荐）或可自管 **Elixir ~> 1.19** 与 **OTP 28**（见 [elixir/mix.exs](../elixir/mix.exs) 与 [AGENTS.md](../elixir/AGENTS.md)）。
- 已安装 **Codex CLI**，且 `codex app-server` 在 PATH 中可用。
- 已创建 **Linear Personal API Key**，并导出为环境变量 `LINEAR_API_KEY`。

## 克隆与安装

```bash
git clone https://github.com/openai/symphony
cd symphony/elixir
mise trust
mise install
mise exec -- mix setup
mise exec -- mix build
```

`mix build` 会生成 escript：**[elixir/bin/symphony](../elixir/bin/symphony)**（由 `mix.exs` `escript` 段定义）。

## 运行

在 `elixir/` 目录下（保证相对路径与 `WORKFLOW.md` 默认可找，或你显式传入路径）：

```bash
export LINEAR_API_KEY=...   # Windows 可用 setx / PowerShell $env:
mise exec -- ./bin/symphony --i-understand-that-this-will-be-running-without-the-usual-guardrails ./WORKFLOW.md
```

### 关于守门参数

CLI **必须**带 `--i-understand-that-this-will-be-running-without-the-usual-guardrails`，否则会打印红色横幅并退出（见 [cli.ex](../elixir/lib/symphony_elixir/cli.ex)）。含义：你确认在**低护栏**预览软件上运行 Codex。

### 可选参数

| 参数 | 作用 |
|------|------|
| `--logs-root <path>` | 日志根目录（默认 `./log` 行为见 README / LogFile） |
| `--port <n>` | 覆盖 HTTP 端口，启用 Phoenix 观测（`0` 为临时端口） |

`WORKFLOW.md` 中 `server.port` 也可启用 HTTP；**CLI `--port` 优先**（见 [elixir/README.md](../elixir/README.md)）。

## 最小 WORKFLOW 示例

复制 [elixir/WORKFLOW.md](../elixir/WORKFLOW.md) 为模板，至少修改：

```yaml
tracker:
  kind: linear
  project_slug: "你的项目slug"
workspace:
  root: ~/symphony-workspaces
```

Prompt 段可先用短句引用 `{{ issue.identifier }}` 测试 Solid 渲染。

## 质量门禁（开发者）

```bash
cd elixir
make all
```

包含格式、lint、coverage、dialyzer 等（见 AGENTS.md）。

## 真实 E2E（可选）

需要真实 Linear + Codex + 可能 Docker SSH worker，见 [elixir/README.md](../elixir/README.md) `make e2e` 章节。日常学习可跳过。

## 常见问题

1. **启动即崩溃**：多为 `WORKFLOW.md` YAML 无效或缺 `LINEAR_API_KEY` / `project_slug`。WorkflowStore 在 init 失败会直接 `{:stop, reason}`。  
2. **无 HTTP**：未传 `--port` 且 workflow 未配置 `server.port` 时 `HttpServer` 为 `:ignore`。  
3. **Windows 路径**：尽量使用 WSL 或与实现一致的 bash 环境；Hooks 依赖 shell。

## 下一篇

- [12-operations-and-debug.md](12-operations-and-debug.md)：运行中观测与排障。
