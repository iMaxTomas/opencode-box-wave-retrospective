# Heavy Task Strategy Card Template

## Purpose

这是一张重任务战略卡模板。

它适合这种情况：

- 当前任务已经不是一个小修
- 需要先搭证据地图
- 需要拆成多个 repair waves
- 需要明确 continue / stop gate

## Template

```md
# <Task Name>

## Purpose

This note is the heavy-task strategy card for <task>.

Use it like this:

`I open this card -> I see the design intent -> I see the strongest rebuttals -> I see the current factual state -> I choose one bounded repair wave`

## Current Position

- 当前任务现在的总状态是什么
- 现在真正的问题已经从“能不能跑”变成了什么

## Strategy Card

- Active plan:
- Current phase:
- Window objective:
- Current wave:
- Not in this wave:
- Output artifact:

## Evidence Chain

### 1. Design intent

- 旧 proposal / design / spec 里本来想达到什么

### 2. Rebuttal / risk

- 这个设计为什么可能不成立
- 已知的运行风险是什么
- 已知的治理风险是什么

### 3. Real-state evidence

- 现场探针证明了什么
- 当前 runtime 是什么状态
- 当前 source / worktree / data / config 是什么状态

## The Real Repair Question

`这轮真正要回答的核心问题是什么`

## Design Claim vs Rebuttal vs Current Fact

### Claim A

<设计主张>

### Rebuttal A

<为什么可能不成立>

### Current fact

<现场证据站在哪边>

### Decision

<当前怎么处理这条主张>

## Wave Menu

### Wave H1

Goal:

Questions:

Output:

Stop if:

### Wave H2

Goal:

Questions:

Output:

Stop if:

## Go / No-Go Gates

### Go

- 什么时候继续

### No-Go

- 什么时候停下来重切

## Current Recommendation

- 当前最该先开哪一波
- 为什么

## Next Entry

1. <next step 1>
2. <next step 2>
3. <next step 3>
```

## 使用规则

### 1. 先搭地图，再开波次

不要上来就填 Wave H1。

先把：

- design intent
- rebuttal / risk
- real-state evidence

搭出来。

### 2. 一次只开一个波次

模板可以列多个波次。

但当前窗口只能开一个。

### 3. 证据必须能落回文件

不要写：

- “我感觉”
- “大概”
- “应该是”

要写：

- 哪个 proposal / design / spec 支持这条 claim
- 哪个现场探针支持这条 current fact

### 4. 这张卡不是战报

这张卡是修复地图。

真正的每轮结果，应该另写：

- `wave-*.md`
- `note`
- `handoff`

一句话：

这张模板是给重任务“先搭地图再开工”用的。
