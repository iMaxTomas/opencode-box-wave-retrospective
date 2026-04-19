# Governed Session Template

本文件定义所有受管治理面在开工前的统一 session header，以及默认隔离模型。  
目标不是增加仪式，而是让每次改动都先回答：

`这次只做什么 -> 谁来写 -> 写到哪算停 -> 哪些 sidecar 不许阻塞 -> 需不需要 branch/worktree/subagent`

如果这次工作不是一次性小修，而是要在一个窗口里连续跑几次小闭环，
那就不要只写 header。要把下面的 `Strategy Card Layer` 一起写出来。

## 1. Standard Session Header

每次进入受管治理面修改前，先写这组字段：

```md
## Governed Session Header

- Change: <change-name>
- Slice: <short slice name>
- Primary milestone: <the one thing this session must finish>
- Stop line: <where new primary work stops>
- Write owner: <main / worker / host / vm103 owner>
- Promotion target: <repo baseline / host ~/.codex target / vm103 target>
- Governed surfaces: <files, dirs, or classes touched>
- Sidecars: <read-heavy side tasks that must not block the milestone>
- Validation intent: <what will be checked before this slice is accepted>
- Evidence path: <where the proof or notes will be saved>
```

## 2. Field Meaning

- `Change`
  - 当前工作属于哪个 OpenSpec change
- `Slice`
  - 当前这次会话只承接的最小交付块
- `Primary milestone`
  - 这次如果只完成一件事，必须是哪件
- `Stop line`
  - 到哪一步就收口，不再继续开新的主线工作
- `Write owner`
  - 谁拥有共享写面最终修改权
- `Promotion target`
  - 这次改动最后是要进入 repo 基线、host `~/.codex`、还是其他执行目标
- `Governed surfaces`
  - 这次会触达哪些受管面
- `Sidecars`
  - 可以并行，但不能阻塞主里程碑的读重任务
- `Validation intent`
  - 这次至少要跑哪些检查
- `Evidence path`
  - 验证结果和结论最终落在哪个文档或输出路径

## 3. Example

```md
## Governed Session Header

- Change: govern-openspec-and-codex-devops-workflow
- Slice: define-session-and-gate-contract
- Primary milestone: land the reusable session template and validation gate doc
- Stop line: after docs are written and task checkboxes are updated
- Write owner: main
- Promotion target: repo baseline
- Governed surfaces: tasks.md, governed-session-template.md, validation-and-adoption-gates.md
- Sidecars: read-only lookup of related change specs
- Validation intent: docs are internally consistent and task progress matches
- Evidence path: change docs + updated tasks.md
```

## 3.5 Strategy Card Layer

当这次治理需要在一个窗口里连续推进几轮小闭环时，给 session header
再叠一张 strategy card。

共享方法现在有一个长期位置：

- [openspec/methods/README.md](/home/imax/codex-workspaces/document-test/openspec/methods/README.md:1)
- [openspec/methods/experience-notes.md](/home/imax/codex-workspaces/document-test/openspec/methods/experience-notes.md:1)
- [openspec/methods/heavy-task-template.md](/home/imax/codex-workspaces/document-test/openspec/methods/heavy-task-template.md:1)

也就是说：

- 共享经验和模板留在 `openspec/methods/`
- 当前任务的具体战略卡继续留在对应 change 目录里

它回答的是：

`我当前只服务哪个 active plan -> 这一个窗口只修哪一个小面 -> 这一轮做完后什么时候继续 -> 什么时候正常停 -> 什么时候被迫停`

推荐模板：

```md
## Governed Strategy Card

- Active plan: <the only main plan for this window>
- Current phase: <which phase or governed slice is active>
- Window objective: <the smallest useful outcome for this window>
- Current wave: <the one bounded loop now being opened>
- Execution hook: <shell / opencode local / opencode VM103 / other explicit hook>
- Governed surfaces: <the exact files or targets this loop may touch>
- Not in this wave: <what this loop must not widen into>
- Preflight: <permissions, paths, live targets, secrets/overlay assumptions that must already be true>
- Output artifact: <the one note, diff, proof, or rendered file this loop must leave>
- Evidence path: <where the result is written back>
```

### Meaning

- `Active plan`
  - 当前窗口只服务的唯一主线
- `Current phase`
  - 当前属于哪个治理阶段或问题面
- `Window objective`
  - 这个窗口里最小且有价值的完成点
- `Current wave`
  - 当前准备打开的那一个小闭环
- `Execution hook`
  - 这一轮真正用什么执行：
    - shell
    - `opencode` 本地入口
    - `opencode` + `VM103`
    - 或其他明确入口
- `Governed surfaces`
  - 这轮可以改的确切受管面
- `Not in this wave`
  - 这轮明确不碰的相邻问题
- `Preflight`
  - 这轮开始前必须已满足的前置条件
- `Output artifact`
  - 这轮必须留下的唯一结果
- `Evidence path`
  - 这轮结果最终写回哪里

## 3.6 Window Loop Rule

Strategy card 不是大计划说明书，它只管一个窗口里的小循环：

`我先锁定一个小目标 -> 我做一次小尝试 -> 我记录结果 -> 我判断还能不能继续下一小轮`

默认规则：

1. 一个窗口里最多连续做 `3-5` 个小闭环。
2. 一轮只证明一个事实，或只修一个明确边界。
3. 每轮结束后先回写 note / task / checkpoint，再决定下一轮。
4. 只要下一轮还是同一个 plan 里的小事实验证，就继续。
5. 一旦开始改 contract、改 phase、改治理边界，先停下来收束。

推荐补一段：

```md
## Continue / Stop Gate

- Continue when:
  - the next loop is still one small proof or one small governed repair
  - the next loop stays inside the same active plan
  - one more loop will still produce clearly new information

- Normal stop when:
  - the window objective is reached
  - the current wave leaves one usable artifact and one clear next entry

- Forced stop when:
  - the work starts widening into a second plan
  - the work starts changing contract/phase instead of finishing the current loop
  - the live target becomes risky enough that dry-run or isolation must be reopened first
```

## 4. Default Isolation Model

### A. Shared writes default to one owner

默认链路：

`我先判断这次会不会改共享治理面 -> 如果会，就指定一个 write owner -> 其他线程只能读、总结或在隔离根里改 -> 最后由 write owner 合并`

### B. Read-heavy side work can go parallel

以下任务可以默认并行：

- rule lookup
- spec comparison
- log reading
- evidence整理
- summary / review notes

这些任务的返回形态应当是：

- 摘要
- review note
- evidence note
- isolated diff

而不是共享工作区里的未协调写入。

### C. Parallel writes require explicit isolation

以下情况不允许直接并行写同一 checkout：

- 两个任务会改同一份 `AGENTS.md`
- 两个任务会改同一份 rules 文件
- 两个任务会改同一个 OpenSpec change 下的共享文档
- 两个任务会改同一份 host promotion script

需要并行写时，只能走：

- branch
- worktree
- 或其他显式隔离根

### D. Host targets are never edited casually first

如果目标是 host `~/.codex`：

`我先改 repo source -> 我先在 repo 侧验证 -> 我再 promotion 到 host target -> 我补 host smoke validation -> 然后才把它当新基线`

不是这样：

- 不是直接先手改 host，再回忆写回 repo

### E. Strategy card decides the loop; config only supplies the habit

治理 `Codex` 配置时，不要把当前 wave 的业务细节塞进长期 config。

正确分工是：

- OpenSpec strategy card
  - 决定这轮只做什么
- Codex durable config
  - 只保证默认工作习惯不跑偏
- execution hook
  - 真正执行这轮 proof 或 repair

也就是说：

`我先读 strategy card -> 我按 card 只开这一轮 -> 我用 shell/opencode 执行 -> 我把结果写回 OpenSpec`

## 5. Session Outcome Rule

每个 session 结束时，至少要回答三件事：

1. primary milestone 是否完成
2. stop line 到了哪
3. 证据或下次入口落在哪

如果 primary milestone 未完成，也不能空着离开；必须留下下一次可接手的 entry point。

## 6. Example: Minimal Codex-Config Governance Re-entry

```md
## Governed Session Header

- Change: govern-openspec-and-codex-devops-workflow
- Slice: minimal-codex-config-reentry
- Primary milestone: land one reusable strategy card and one current re-entry card
- Stop line: after the template and current card are written and linked
- Write owner: main
- Promotion target: repo baseline
- Governed surfaces: governed-session-template.md, current-codex-config-reentry-card.md, tasks.md
- Sidecars: read-only lookup of prior config projection evidence
- Validation intent: docs are internally consistent and clearly separate plan/card/execution responsibilities
- Evidence path: change docs + updated tasks.md

## Governed Strategy Card

- Active plan: govern-openspec-and-codex-devops-workflow
- Current phase: minimal Codex config re-entry
- Window objective: define the next safe config-governance slice without recreating a large config program
- Current wave: write the reusable card and freeze the next minimal slice
- Execution hook: shell only
- Governed surfaces: repo docs only
- Not in this wave: no live apply, no large config rewrite, no second governance branch
- Preflight: repo state is readable and the failed previous governance attempt has already been rolled back
- Output artifact: one reusable template plus one current re-entry card
- Evidence path: this change
```
