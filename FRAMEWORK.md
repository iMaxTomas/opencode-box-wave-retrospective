# Framework

## 目标

这份框架不是为了让单个 `wave` 更漂亮，而是为了让整个长任务周期可以按天迭代，同时保持执行效率。

## 四层结构

### 1. Campaign

周期级目标。

要回答：

- 这个 change 本周期做到哪里算完成
- 哪些内容本周期明确不做
- 哪些 phase 必须有先后顺序

### 2. Phase

阶段级目标。

规则：

- 一个 phase 只解决一类问题
- 不允许把不同 authority surface 混在同一个 phase

例如：

- minimal startup boundary
- stable operator entry
- provider ingress contract
- live provider proof
- OAuth migration semantics

### 3. Wave

执行级单位。

规则：

- 一个 wave 只证明一个事实
- 只允许一个 blocking 目标
- 只允许一个 exit artifact

如果一句话里出现两个以上“并且”，这个 wave 往往就太大了。

### 4. Checkpoint

压缩与恢复单位。

一个好 checkpoint 只应该包含：

- 已验证事实
- 当前边界
- 下一轮唯一目标
- 本轮明确不碰什么
- 成功 / 失败判据

## 继续还是停止

### 继续

满足这些条件时继续：

- 当前目标还能一句话说清
- 当前只剩一个 proof
- 当前不会引入新的 authority surface

### 停止并重编排

满足任一条就停：

- 一轮执行超过 10 到 15 分钟还没有单一结论
- 同一轮里既在设计边界，又在修实现，又在做验证
- 同一个意外修复出现两次以上
- 新问题已经不属于本 phase

## subagent 的位置

subagent 很有用，但应该放在侧翼：

- 旧证据检索
- 源码事实提取
- 文档回填
- 一致性 review

不要把主 proof 交给 subagent，除非这条 proof 和主执行链完全解耦。

## worktree 的位置

worktree 不是默认加速器。

适用场景：

- 并行写代码
- 高风险改动隔离
- 多分支长线改造

不适用场景：

- 只是想让单个 wave 更快
- 只是为了心理上更“正式”

## 最后一句

高性能执行的关键不是“上下文够大”，也不是“开了多少 agent”。

真正关键的是：

`前置战略切分清楚 -> phase 不混层 -> wave 只做单事实 proof -> checkpoint 只保留硬结论`

