## Why

当前 `103` 上的 `Codex`、`Claude Code` 和本机 `Codex` 已经分别具备可用运行面，但它们还没有共享同一份长期记忆层。现在最短路径不是先引入更大的 agent 通信协议，而是把 `RecallNest` 部署到 `VM103`，让 `103` 本机工具和本机电脑上的 `Codex` 都能通过同一份 `RecallNest` 数据库与 continuity 规则联动。

这件事现在就值得单独做，因为它正好卡在“实验室底座已可用”和“多 agent 团队开始变复杂”之间。再往后拖，跨窗口、跨机器、跨工具的上下文丢失会越来越频繁，而 ACP 之类的协议化工作反而会把第一条 shared-memory 主线继续推迟。

## What Changes

- 在 `VM103` 上部署 `RecallNest` 作为当前实验室的 authoritative shared memory host。
- 固定 `RecallNest` 在这套系统里的职责：shared memory / continuity / workflow observation，不替代 artifact、trace 或 provider routing。
- 为 `103` 上的 `Codex` 与 `Claude Code` 接入同一份 `RecallNest` MCP continuity 规则。
- 为本机 `Codex` 定义“通过 SSH 远程运行 `103` 上的 RecallNest MCP server”的消费路径，从而让本机和 `103` 共用同一份记忆库，而不是各自建立本地孤岛。
- 为 `RecallNest` 的 API / UI / doctor / continuity seed / ingest / backup 定义最小运维面，保证它可启动、可检查、可恢复。
- 把 `ACP` 明确留在 follow-up，不让它阻塞第一条 shared-memory 落地主线。

## Capabilities

### New Capabilities
- `vm103-recallnest-memory-host`: Deploy RecallNest on `VM103` as the shared memory host, including runtime layout, data location, health checks, and service lifecycle.
- `remote-recallnest-mcp-consumers`: Define how `VM103` local tools and the local machine's `Codex` consume the same RecallNest memory host through MCP, including the SSH-remote MCP path for the local machine.
- `recallnest-continuity-operations`: Define the minimum operational contract for seeding continuity, running doctor checks, ingesting conversation history, opening the UI or API safely, and backing up the RecallNest state.

### Modified Capabilities
- None.

## Impact

- Affected systems:
  - `VM103` runtime layout, package/runtime dependencies, and service surface
  - local machine `Codex` MCP configuration
  - `VM103` `Codex` / `Claude Code` MCP and continuity setup
  - `Bitwarden` as the likely source for `RecallNest` embedding or model-related secrets if needed
- Affected artifacts:
  - new OpenSpec planning artifacts under `openspec/changes/deploy-recallnest-shared-memory-on-vm103/`
  - future runtime/evidence files on `103:/opt/ai-lab/docs/` and the RecallNest data/runtime directories
- External dependencies:
  - local RecallNest repo at `/home/imax/document/test/recallnest`
  - Bun runtime and RecallNest MCP/API processes
  - SSH connectivity from the local machine to `VM103`
