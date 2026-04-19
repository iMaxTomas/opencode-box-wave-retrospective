# Review Packet Index

这份索引用来说明，这个仓库里新增的 `docs/source/` 审查包各自是什么、为什么放进来。

目标不是把原仓库完整镜像过来。

目标是给外部审查者一组**足够审查这轮方法演进**的原始材料。

## 1. 方法层

目录：

- `docs/source/openspec-methods/`

包含：

- `README.md`
- `experience-notes.md`
- `heavy-task-template.md`

作用：

- 看这轮之后，方法层是怎么被单独抽出来的
- 看“经验层”是怎么和“配置真相源”分开的
- 看重任务模板到底长什么样

## 2. 当前任务线：stage-pve-vm-agent-lab

目录：

- `docs/source/stage-pve-vm-agent-lab/`

包含：

- `proposal.md`
- `design.md`
- `tasks.md`
- `vm103-login-chain-diagnostic-20260420.md`
- `vm103-login-chain-handoff-20260420.md`
- `vm103-recallnest-relaunch-20260420.md`
- `vm103-recallnest-status-audit-20260420.md`
- `vm103-recallnest-heavy-repair-strategy-20260420.md`

作用：

- 看当前 `RecallNest` / `VM103` 这条线的真实任务情报
- 看这次战略卡不是空谈，而是怎么落到诊断卡、复用卡、状态审计、重任务卡上的

## 3. 旧部署基线：deploy-recallnest-shared-memory-on-vm103

目录：

- `docs/source/deploy-recallnest-shared-memory-on-vm103/`

包含：

- `proposal.md`
- `design.md`
- `tasks.md`
- `specs/vm103-recallnest-memory-host.spec.md`
- `specs/remote-recallnest-mcp-consumers.spec.md`
- `specs/recallnest-continuity-operations.spec.md`

作用：

- 看旧方案当时是怎么定义 `VM103` shared-memory host 的
- 看这次新战略卡到底复用了哪些旧 contract
- 看“设计诉求”这一条证据线来自哪里

## 4. 风险 / 反驳基线：design-six-agent-serena-recallnest-adoption

目录：

- `docs/source/design-six-agent-serena-recallnest-adoption/`

包含：

- `proposal.md`
- `design.md`
- `tasks.md`

作用：

- 看 `RecallNest` 和 `Serena` 的职责边界
- 看“风险 / 反驳”这条证据线来自哪里

## 5. 这包材料怎么读

推荐顺序：

1. 先看根目录：
   - `README.md`
   - `TIMELINE.md`
   - `FRAMEWORK.md`
2. 再看这份：
   - `DOCS-INDEX.md`
3. 然后按这个顺序看原始文档：
   - `openspec-methods`
   - `deploy-recallnest-shared-memory-on-vm103`
   - `design-six-agent-serena-recallnest-adoption`
   - `stage-pve-vm-agent-lab`

一句话：

这个 `docs/source/` 不是随手堆材料，而是给别人审你这轮“战略卡分层”和 `RecallNest` 重任务判断的证据包。
