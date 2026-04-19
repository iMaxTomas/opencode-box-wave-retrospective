# Methods

这个目录是当前仓库里**长期持有**的方法层位置。

它解决的问题不是：

- 某一个 change 当前在干嘛
- 某一轮 wave 具体执行什么

它解决的是：

- 战略卡到底是什么
- 重任务应该怎么拆
- 哪些规则属于共享打法
- 哪些内容应该留在具体 change 里

## 分层

### 1. 这里放什么

这里放长期稳定的共享内容：

- 经验笔记
- 重任务修复打法
- 波次菜单怎么开
- 设计诉求 / 反驳 / 现场事实 怎么并排收证据
- 什么时候继续，什么时候停
- 模板

### 2. 这里不放什么

这里不放具体任务的当前状态：

- 当前 phase
- 当前 wave
- 当前机器的即时健康状态
- 某个 change 的本轮 not-in-scope
- 某个问题的当下 verdict

这些都应该留在对应的 change 目录里。

## 当前使用方式

推荐这样用：

1. 先看这里的方法和经验
2. 再去具体 change 里写：
   - `campaign.md`
   - `execution-waves.md`
   - `phase-*.md`
   - `wave-*.md`
   - `handoff`
3. 每次重任务只开一个 bounded wave
4. 跑完后把结果写回具体 change

## 当前文件

- [experience-notes.md](/home/imax/codex-workspaces/document-test/openspec/methods/experience-notes.md:1)
  - 当前沉淀下来的高价值经验
- [heavy-task-template.md](/home/imax/codex-workspaces/document-test/openspec/methods/heavy-task-template.md:1)
  - 重任务的共享模板

## 和 OpenSpec change 的关系

关系应该是：

`这里管共享方法 -> change 里的战略卡管当前任务 -> wave note 管每轮证据`

一句话：

这个目录是“长期方法层”，不是“某次任务的当前作战卡”。
