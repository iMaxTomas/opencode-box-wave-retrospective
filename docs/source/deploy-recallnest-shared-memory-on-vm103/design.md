## Context

当前已经成立的基线是：

- `VM103` 已经是当前 AI 实验室的主工作面，`Codex`、`Claude Code`、`Bitwarden consumer`、lane wrapper 和证据目录都已存在。
- 本机和 `VM103` 都能运行 `Codex`，但它们还没有共享同一份长期记忆层。
- 本机已经有完整的 `RecallNest` 仓库副本：`/home/imax/document/test/recallnest`
- `RecallNest` 当前的官方集成方式有两条：
  - `MCP (stdio)` 给 `Codex / Claude Code / Gemini CLI`
  - `HTTP API` 给 custom agents
- `RecallNest` 的默认 API server 绑定 `127.0.0.1:4318`
- `RecallNest` 的 `Codex` 集成脚本默认是在当前机器本地 `~/.codex/config.toml` 中加入：
  - `command = "bun"`
  - `args = ["run", ".../src/mcp-server.ts"]`
- 这意味着如果直接在本机和 `103` 上各自跑本地集成，它们会自然形成两套独立记忆库，而不是自动共享。

因此这条 deployment change 的核心不是“把 RecallNest 随便装上”，而是：

`我把 RecallNest 部署到 103 -> 103 本地的 Codex/Claude 直接消费这台 RecallNest -> 我在本机开 Codex 时，不是连本地孤岛，而是通过 ssh 在 103 上启动 RecallNest MCP server -> 这个 MCP 进程在 103 上读写同一份 data -> 结果存回 103 的 RecallNest data -> 下次不管我是在本机还是 103 上开窗口，都复用同一份 shared memory`

不是这样：
- 不是本机和 `103` 各自跑一套本地 RecallNest，再假装它们已经联动
- 不是先引入 `ACP` 才能完成 shared memory
- 不是让 RecallNest 替代 `artifact`、`trace`、或 provider routing

## Goals / Non-Goals

**Goals:**
- 把 `RecallNest` 部署到 `VM103`，作为当前实验室 authoritative shared memory host
- 让 `103` 本地 `Codex` 与 `Claude Code` 消费这台 RecallNest
- 让本机 `Codex` 通过 SSH-remote MCP 方式消费同一台 RecallNest
- 为 `RecallNest` 定义清晰的代码目录、数据目录、日志目录、运行入口、健康检查和最小运维动作
- 定义 continuity seed、doctor、ingest、UI/API 访问、以及备份边界
- 保持当前 commander / lane / artifact / trace 边界不被 RecallNest 越层覆盖

**Non-Goals:**
- 本 change 不引入 `ACP`
- 本 change 不要求 `Gemini` 同时接入
- 本 change 不把 RecallNest 升级成新的 routing control plane
- 本 change 不替代 `Bitwarden`
- 本 change 不要求现在就迁移 root commander 到完全隔离的 commander lane

## Decisions

### 1. `VM103` 是 authoritative memory host，不做双主
当前固定为：

`我先把 RecallNest 部署到 103 -> 103 上保存 authoritative repo/runtime/data -> 本机和 103 上的客户端都消费这同一份 host`

不是这样：
- 不是本机一份、`103` 一份双主并行
- 不是先把 RecallNest 放到本机，再让 `103` 去消费

原因：
- `103` 已经是当前 AI 实验室工作面
- 用户明确希望本机 `Codex` 和 `103` `Codex` 联动完成主线任务
- 让 shared memory 跟实验室主工作面在一起，备份和快照边界最清楚

替代方案：
- 本机作为 RecallNest host  
  放弃原因：会让 `103` 变成依赖外部桌面机的消费者，不适合当前实验室主线。

### 2. 本机 `Codex` 通过 SSH-remote MCP 消费 RecallNest，不先造协议桥
当前消费链固定为：

`我在本机启动 Codex -> 本机 Codex 的 MCP command 不是本地 bun run，而是 ssh vm103-openclaw 在 103 上启动 RecallNest MCP server -> MCP stdio 通过 SSH 传回本机 -> RecallNest 在 103 上读写同一份 LanceDB 与 continuity state`

不是这样：
- 不是本机先跑一套本地 MCP server
- 不是现在就先做 HTTP bridge 或 ACP bridge

原因：
- RecallNest 的 MCP server 本来就是 stdio 进程
- `ssh` 本身就能把远端 stdio 进程接成本地 MCP command
- 这样可以直接复用 `103` 上的同一份数据，不需要先改 RecallNest 架构

替代方案：
- 让本机 `Codex` 走 HTTP API  
  放弃原因：当前 `Codex` 现成集成路径是 MCP，不是 HTTP custom agent。

### 3. `103` 本地工具走本地 MCP；API/UI 只作为运维面
当前分层固定为：

- `Codex / Claude Code on 103`
  - 走本地 MCP command
- `Codex on local machine`
  - 走 SSH-remote MCP command
- `RecallNest API`
  - 主要给 ops、debug、custom agent、curl smoke
- `RecallNest UI`
  - 主要给 inspection，不作为主运行面

原因：
- RecallNest 官方产品定位就是 `MCP + UI`
- 对当前 CLI agents 来说，MCP 是主接入面
- API/UI 更适合诊断、审查和手动检查

替代方案：
- 所有消费都改成 HTTP API  
  放弃原因：会偏离现有 CLI 集成方式，也会让 Codex/Claude 的 continuity 规则不能直接复用。

### 4. 代码、数据、日志分离，不直接把 repo 当运行态
当前目录边界固定为：

- 代码：
  - `/opt/recallnest`
- 数据：
  - `/srv/ai-memory/recallnest`
- 日志与证据：
  - `/opt/ai-lab/logs/recallnest`
  - `/opt/ai-lab/docs/recallnest-*`

这意味着：

`我更新代码 -> RecallNest 运行时继续把数据写到 /srv/ai-memory/recallnest -> 健康检查和运维记录进 /opt/ai-lab/... -> 下次回滚或升级时，代码与数据边界是清楚的`

不是这样：
- 不是把数据直接混进 git repo 目录
- 不是把运维日志和 memory store 混成同一层

原因：
- 便于备份、回滚和容量巡检
- 便于和现有 `103` 实验室目录边界保持一致

### 5. `JINA_API_KEY` 等 live secrets 继续由 Bitwarden 提供，不写进 repo
当前 secrets 路径固定为：

`我在 104 的 Bitwarden 里保存 RecallNest 需要的 live key -> 103 在部署或服务启动时读取并注入当前进程 -> repo 里只保留 .env.example / config example，不留 live 值`

原因：
- 保持和现有实验室 secret contract 一致
- 本机 `Codex` 如果只是通过 SSH remote MCP 消费，就不需要单独拿一份 Jina key

替代方案：
- 把 live `.env` 一起放进 `/opt/recallnest`  
  放弃原因：会破坏当前 non-secret / live-secret 的边界。

### 6. API server 默认继续只绑 loopback；跨机器 inspection 用 SSH tunnel
当前 API/UI 暴露方式固定为：

`我在 103 上启动 RecallNest API -> API 默认只绑定 127.0.0.1 -> 如果我要从本机访问 UI 或 API，我用 ssh 隧道转发端口 -> 不是直接把 RecallNest API 暴露到整个局域网`

原因：
- RecallNest 的 `api-server.ts` 当前就是 loopback 默认
- 这比直接对整个 LAN 暴露更稳
- 对本机 inspection 足够用

替代方案：
- 直接开放 0.0.0.0 给局域网  
  放弃原因：当前阶段没必要扩大暴露面。

### 7. continuity seed、doctor、ingest 是第一阶段必备运维面
当前第一阶段最小 ops surface 固定为：

`我先把 RecallNest 启起来 -> 我跑 doctor -> 我跑 seed:continuity -> 我根据需要 ingest 现有会话 -> 我再做本机 Codex 与 103 Codex/Claude 的联动验证`

原因：
- 没有 doctor，就不知道 runtime 和 key 是否健康
- 没有 seed，就没有 continuity baseline
- 没有 ingest，shared memory 初始值太空

### 8. `ACP` 明确延后，不进这条 deployment 主线
当前判断固定为：

`我先把 RecallNest shared memory on VM103 跑通 -> 本机 Codex 与 103 通过同一份 MCP memory 联动 -> 如果后面真的需要 agent-to-agent protocol，再单独开 ACP follow-up`

原因：
- 当前缺的是 shared memory，不是协议编排层
- 先把 shared memory 跑通，回报更直接

## Risks / Trade-offs

- [Risk] 本机 `Codex` 通过 SSH remote MCP 使用远端进程，稳定性取决于 SSH 和远端 Bun 运行状态 -> Mitigation: 固定 SSH alias、保留本地 smoke command、把 doctor 和远端 MCP 检查写进运维手册。
- [Risk] `RecallNest` 需要 embedding key；如果 secret 注入没收敛，会卡在“服务能起但 recall 质量不稳定” -> Mitigation: 把 Bitwarden item、服务启动方式和 doctor 检查写成显式步骤。
- [Risk] 继续使用 root 作为 `103` 上 RecallNest 的运行面，会继承当前实验室的 root 风险 -> Mitigation: 当前接受 root 部署，但把目录、服务和证据边界固定清楚，为后续迁移保留空间。
- [Risk] 如果把 API/UI 直接暴露到 LAN，会扩大攻击面和误用面 -> Mitigation: 默认 loopback + SSH tunnel。
- [Risk] 若本机与 `103` 同时各自运行本地 RecallNest 集成，会重新分叉成两套记忆库 -> Mitigation: 在本机配置里明确使用 SSH-remote MCP command，而不是本地 MCP command。

## Migration Plan

1. 在 `103` 上准备 RecallNest runtime 依赖和目录边界。
2. 将本机已有 RecallNest repo 复制或同步到 `103:/opt/recallnest`，并固定数据目录到 `/srv/ai-memory/recallnest`。
3. 用 Bitwarden 提供 RecallNest 所需 live key，在 `103` 上完成 doctor 与最小 API smoke。
4. 在 `103` 上为 `Codex` / `Claude Code` 接入本地 RecallNest MCP continuity。
5. 在本机为 `Codex` 接入 SSH-remote RecallNest MCP continuity。
6. 跑一次联动验证：
   - 在 `103` 上写入 checkpoint 或 durable memory
   - 在本机 `Codex` 恢复或检索同一条上下文
7. 把运维与验证证据写入 `103:/opt/ai-lab/docs/` 和当前 change。
8. 如果任一步破坏现有工作流，就先回退 RecallNest 集成配置，不碰现有 `Codex` / `Claude` 基线。

## Open Questions

- `RecallNest` 在 `103` 上是直接复用 root 运行，还是这次顺手切到专用服务用户？
- 第一阶段是否只接 `Codex + Claude`，把本机 `Claude` 或其它客户端留到 follow-up？
- 当前是否已经有可用的 `JINA_API_KEY` 可以直接进 Bitwarden，还是这一步还需要新的 live 输入？
- 现有 conversation ingest 的首批源要不要只限于 `103` 当前实验室主线，而不是一次 ingest 所有历史窗口？
