# 09 工作区：布局、Hooks 与安全不变量

## 布局（SPEC 9.1）

- **根目录**：`workspace.root`，解析为绝对路径。  
- **每工单目录**：`<root>/<workspace_key>`，其中 `workspace_key` 由 `issue.identifier` 净化：非 `[A-Za-z0-9._-]` 替换为 `_`。

**持久化**：同一工单多次运行**复用**同一目录；成功运行**不会**自动删除（SPEC 9.2）；终态或对账路径触发清理。

## 创建与复用（SPEC 9.2）

实现 [Workspace](../elixir/lib/symphony_elixir/workspace.ex)：

- **本地**：若路径已是目录 → 复用；若存在**非目录**文件 → 删除后新建目录（实现选择清空障碍）。  
- **远端（SSH）**：通过远程 shell 脚本 `mkdir` / `rm -rf` 等价逻辑，保持与本地语义接近。

## 四个 Hooks（SPEC 9.4）

| Hook | 触发时机 | 失败后果 |
|------|----------|----------|
| `after_create` | 目录**本次**新建后 | **致命**：工作区创建失败，本次 attempt 失败 |
| `before_run` | 每次 Agent 尝试前 | **致命**：跳过 Codex |
| `after_run` | 每次尝试结束（成功/失败/超时/取消） | 记录日志，**忽略**失败 |
| `before_remove` | 删除工作区目录前 | 记录日志，**忽略**失败；仍继续删 |

执行上下文：**cwd = 工作区**；超时 `hooks.timeout_ms`；POSIX 侧等价 `bash -lc`（见 Workspace/SSH 实现）。

**典型用法**：`after_create` 里 `git clone ... .` 拉副本；`before_run` 里 `git fetch` / `mise install`；`before_remove` 里运行 [mix workspace.before_remove](../elixir/lib/mix/tasks/workspace.before_remove.ex) 等清理任务（本仓库示例 WORKFLOW）。

## 三条安全不变量（SPEC 9.5，最高优先级）

1. **Codex cwd 不变量**：启动子进程前验证工作目录等于**该工单工作区**的绝对路径。  
2. **根前缀不变量**：工作区路径必须落在 `workspace.root` 之下（规范化后前缀检查）。  
3. **目录名净化**：`workspace_key` 仅含允许字符集。

实现辅助：[PathSafety](../elixir/lib/symphony_elixir/path_safety.ex)；Codex 启动前在 [AppServer](../elixir/lib/symphony_elixir/codex/app_server.ex) 调 `validate_workspace_cwd/2`。

违反任一条件：**拒绝启动**，防止代理在仓库根或其他敏感路径执行。

## SSH 远端工作区（SPEC 附录 A）

- `worker.ssh_hosts`：候选执行机列表。  
- `workspace.root` 在远端解释；Orchestrator 仍在中心节点。  
- Codex stdio 经 SSH 管道回到本机解析（见 `SSH.start_port/3`）。

**运维注意**：每台 worker 机需自备 Codex、认证、`git`、依赖工具；与中心机时钟/磁盘无关。

## 工作区人口（SPEC 9.3）

规范**不要求**内置 `git clone`；由 Hook 或外部策略完成。失败应返回错误给当前 attempt；是否删除半成品目录由实现策略决定（本仓库 Workspace 对「非目录占用」会 `rm_rf` 后重建）。

## 下一篇

- [04-end-to-end-flow.md](04-end-to-end-flow.md)：Hook 在 AgentRunner 中的调用顺序。  
- [11-getting-started.md](11-getting-started.md)：最小 `after_create` 示例。
