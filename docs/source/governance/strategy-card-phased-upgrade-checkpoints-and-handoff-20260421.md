# Strategy Card Phased Upgrade Checkpoints And Handoff 2026-04-21

这份文档不是新的战略卡大包。

它只解决一件事：

`如果战略卡2升级不能再整包推进，那么分阶段升级时，每一阶段到哪里算过关，中断后下一窗口怎么接。`

---

## Purpose

Use it like this:

`I open this card -> I confirm the current phase -> I check whether the current checkpoint is already passed -> I only work on the next missing checkpoint -> I leave one handoff packet -> I stop`

This card is not:

- a permission slip to reopen the whole `strategy card 2` bundle at once
- a replacement for the methods layer
- a claim that all four phases must be finished in one cycle

---

## Core Rule

分阶段升级时，不按“概念包”推进，而按“成熟度层”推进：

1. `控制壳子`
2. `执行路由`
3. `证据与裁决`
4. `最小协作包`

只有当前一阶段达到 checkpoint，才允许打开下一阶段。

不是这样：

- 不是四个阶段同时铺开
- 不是因为后面阶段看起来重要，就跳过前面薄弱层
- 不是因为已经有局部经验，就默认它已经进入长期基线

---

## Phase Map

### Phase 1: Control Shell MVP

#### Goal

先让战略卡能稳定跑完一轮闭环，不再主线漂移。

#### Scope

- `Active plan`
- `Current phase`
- `Current wave`
- `Stop line`
- `Output artifact`
- `Evidence path`

#### Checkpoint 1

当前战略卡必须能在一张卡里明确回答：

- 这轮只服务哪个 active plan
- 当前只做哪一个 bounded wave
- 到哪算停
- 这轮会留下哪个唯一 artifact

#### Passed When

- 一个窗口只认一个主线
- 一个 wave 只认一个 bounded objective
- 恢复时不需要额外解释“这轮在干嘛”

#### Handoff Must Leave

- current active plan
- current wave
- stop line
- one output artifact
- next exact re-entry point

---

### Phase 2: Execution Routing MVP

#### Goal

让战略卡知道什么时候用 `shell`，什么时候用 same-runtime side work，什么时候才允许升级到 `VM103`。

#### Scope

- `Execution hook`
- `Preflight`
- `Sidecar Boundary Rule`
- `host-only / host + VM103` choice rule

#### Checkpoint 2

当前卡必须能明确回答：

- 这轮是不是 shell-first
- 这轮是不是 same-runtime child work
- 这轮为什么需要或不需要 `VM103`

#### Passed When

- 机器事实默认走 `shell`
- 读重默认走 same-runtime side work
- 只有真需要 remote facts / remote state / remote evidence 时才升级 `VM103`
- `spawned` 不等于 `integrated`

#### Handoff Must Leave

- chosen execution hook
- why heavier hooks were rejected or accepted
- sidecar ownership boundary
- if remote was used, what packet or proof came back

---

### Phase 3: Evidence And Verdict MVP

#### Goal

让战略卡不只会记录，还会判定：继续、停止、延后、follow-up。

#### Scope

- `Evidence Status`
- `Entry gate`
- `Exit gate`
- `verdict`
- `closeout`

#### Checkpoint 3

当前卡必须能明确回答：

- 当前证据是什么等级
- 什么条件下允许进入下一轮
- 什么条件下必须停止
- 这轮结论究竟是什么

#### Passed When

- 每轮都有 verdict
- 每轮结束后都知道：
  - continue
  - stop
  - delay
  - follow-up
- 不再靠对话临场解释来收口

#### Handoff Must Leave

- evidence status
- verdict
- closeout or next-gate condition
- what remains unresolved

---

### Phase 4: Minimal Collaboration Bundle v1

#### Goal

最后才把 `KPI + Main-Agent + Context-Sharing + Runtime Hygiene` 作为一个最小 bundle 做短周期判生死。

#### Scope

- `KPI`
- `Main-Agent`
- `Context-Sharing`
- `Runtime Hygiene`

#### Checkpoint 4

当前 bundle 必须能明确回答：

- 每一项的最小定义是什么
- 哪些是尺子，哪些是被尺子看的对象
- 哪些样本够用
- 到点后究竟是升入长期基线，还是退回候选

#### Passed When

- bundle 定义不再漂
- 样本数量受控
- 时间窗口受控
- 到点后只允许：
  - promote
  - reject

#### Handoff Must Leave

- current bundle definition
- sample list
- current verdict
- promote / reject recommendation

---

## Checkpoint Ladder

阶段之间采用固定梯子：

`Phase 1 checkpoint passed -> Phase 2 can open`

`Phase 2 checkpoint passed -> Phase 3 can open`

`Phase 3 checkpoint passed -> Phase 4 can open`

如果前一阶段 checkpoint 没过：

- 不许偷开下一阶段
- 不许靠更多文档堆积来假装过关
- 不许用后面阶段的讨论替代前面阶段的收口

---

## Handoff Packet Template

每个阶段结束时，handoff 至少要留这 8 项：

```md
## Phase Upgrade Handoff

- Active plan:
- Current phase:
- Checkpoint status: passed / not passed
- What was fixed in this phase:
- What is still weak:
- Next exact action:
- Not in next window:
- Evidence path:
```

如果当前阶段已经 closeout，还要再补一条：

- `Promotion posture:` baseline / candidate / rejected / follow-up

---

## Stop Rules

满足任一条就停止，不再继续往下一阶段开：

- 当前阶段的 checkpoint 还没过
- 命名开始漂移
- 当前窗口开始同时服务两个 active plans
- 当前阶段开始混入下一阶段的主要问题
- 当前 handoff 已经写不清“下一步只做什么”

---

## Recovery Rule

下次恢复时，不从“大包战略卡2”重新开讲。

只做这条链：

1. identify current phase
2. identify whether current checkpoint is already passed
3. if not passed, only finish this checkpoint
4. if passed, open the next phase

一句话：

`恢复时先找阶段 checkpoint，不要先重开整包讨论。`

---

## One-Sentence Summary

战略卡分段升级的关键不是“列四个阶段”，而是让每个阶段都有明确 checkpoint、明确 handoff、明确停线条件，这样升级才不会重新变成一个傻大包。
