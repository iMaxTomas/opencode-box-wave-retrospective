## Why

当前已经具备一个很合适的低成本验证起点：`PVE` 宿主机、`VM 103 (debian-openclaw)`、可回滚快照、以及从本机直连客体机的免密 SSH。但这些条件还没有被组织成一个可复用、可迭代、可回滚的 OpenSpec change，导致“实验验证”“正式环境配置”“长期架构设想”容易重新缠在一起，一有新目标就会再次摇摆。

需要一个更具体的 change，把这条链固定下来：先在 `103` 里做分阶段验证，再决定哪些配置值得回灌到正式机器、哪些应该拆成后续 change、哪些只保留在实验室环境。

## What Changes

- 新增一个面向 `PVE -> VM 103 -> 快照 -> SSH -> 基础环境 -> Codex / Claude Code / Gemini / OpenClaw -> 验证 -> 回滚/复用` 的实验室 capability。
- 固化 `snapshot-first` 推进方式，要求在每个高风险阶段前后都有明确的检查点和回滚点。
- 固化从本机直连客体机的访问路径，而不是依赖每次先登录 `PVE` 再手工跳转。
- 固化客体机中的标准目录结构，用于保存 bootstrap 脚本、检查脚本、文档、日志和派生产物。
- 固化 `103` 内的 lane-based 隔离骨架，把不同 `Codex` 账号、不同 API provider、以及后续 `Claude Code` provider 的运行态分开。
- 固化 `Codex` 在该实验室中的官方并行基线：显式 subagents、优先内置角色、读重任务优先并行、写重任务默认串行或进入 worktree 隔离。
- 把 `Bitwarden` secrets 服务端固定为独立实例，而不是放进 `103` 本身；`103` 只作为消费方。
- 固化 `104` 上 `Bitwarden` 的 TLS 基线，要求它提供带正确 `SAN` 的证书，并要求 `103` 在消费前先信任对应 CA。
- 定义分阶段接入顺序和阶段退出条件，避免四套工具同时混装后难以判断问题归因。
- 定义实验室验证完成后的“是否推广到正式环境”评审门槛，避免把实验结论直接等同于正式方案。
- 把待确认事项和阶段性复盘问题纳入 change 跟踪，而不是继续散落在聊天上下文中。

## Capabilities

### New Capabilities
- `pve-vm-agent-lab`: 定义基于 PVE 客体机的 AI agent 实验室基线、阶段推进、验证、回滚与复用规则。

### Modified Capabilities

- None.

## Impact

- `PVE` 宿主机上的 `VM 103` 生命周期管理与快照命名/使用方式
- 本机到 `103` 的 SSH 访问方式与别名管理
- 客体机内的标准目录与后续 bootstrap/check/doc/log/artifact 产物布局
- 独立 `Bitwarden` 实例与 `103` 消费端之间的 secrets 流程，包括 `104` 证书生成、`103` CA 信任链、以及操作员的 HTTPS 访问提示
- `Codex` / `Claude Code` 按 lane 进行的账号、provider 和运行态隔离方式
- `Codex` 在当前实验室中的 subagents / worktree 使用边界，以及它与其它工具阶段的衔接顺序
- 后续 `Codex`、`Claude Code`、`Gemini`、`OpenClaw` 的安装顺序、验证顺序和回滚顺序
- 与更宽的基础环境治理 change 的衔接方式，特别是 [`add-ai-agent-base-environment`](/home/imax/codex-workspaces/document-test/openspec/changes/add-ai-agent-base-environment)
