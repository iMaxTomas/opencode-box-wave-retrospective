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

## 后续修正：战略卡要分层

后来继续在 `RecallNest`、`Codex` 配置治理和 `OpenSpec` 结构上推进时，又暴露出一个新问题：

如果不继续分层，战略卡本身也会混乱。

会混在一起的有三类东西：

- 当前任务的作战卡
- 长期可复用的经验
- 真实配置真相源

所以后面收口出的更稳结构是：

### 1. OpenSpec 薄底座

只放最小的全局底座：

- `specs`
- `changes`
- 最小治理边界

不要把当前任务打法和长期经验都往这里塞。

### 2. 方法层

这层放：

- 战略卡经验
- 重任务模板
- 常用启发式
- 波次拆法

它更像方法库，不是配置层，也不是当前任务的即时战报。

### 3. 具体 change 层

这层放：

- `campaign.md`
- `execution-waves.md`
- `phase-*.md`
- `diagnostic note`
- `handoff`
- `wave note`

也就是每个任务自己的当前作战卡和当前情报。

### 4. 配置真相源层

这层单独存在。

它放：

- `.codex`
- `ko`
- `daks`
- 其他 durable config source

这层不该和方法层混。

## 当前推荐结构

```text
openspec
├── config.yaml
├── specs/
├── methods/
│   ├── README.md
│   ├── experience-notes.md
│   └── heavy-task-template.md
└── changes/
    └── <change>/
        ├── proposal.md
        ├── design.md
        ├── tasks.md
        ├── campaign.md
        ├── execution-waves.md
        ├── phase-*.md
        ├── diagnostic-*.md
        ├── handoff-*.md
        └── wave-*.md

config
├── README.md
├── codex/
├── ko/
└── daks/
```

这一版的意思是：

- `openspec/config.yaml`
  - 继续保持薄底座
- `openspec/methods/`
  - 放经验、模板、方法
- `openspec/changes/<change>/`
  - 放当前任务卡和当前情报
- `config/`
  - 放真实配置真相源

## 那 5 条经验后来怎么重新定位

后来发现，这 5 条虽然很有用，但不该被误抬成“全局核心规则”：

1. 先分清“运行健康”和“基线健康”
2. 重任务同时摆出“设计诉求 / 反驳 / 现场事实”
3. 先找最影响后续动作的那个模糊点
4. 先做波次菜单，再只开一个波次
5. sidecar 收证据，主线程定修复方向

它们更适合被理解成：

- 当前阶段提炼出来的高价值经验
- 重任务方法层的共享启发式

而不是：

- 所有任务都必须套用的上位核心规则

## 最后的更新结论

除了 `Campaign / Phase / Wave / Checkpoint` 之外，

还要再分清：

`底座 -> 方法 -> 当前任务卡 -> 配置真相源`

这样长任务后面才不会把：

- 当前任务策略
- 长期经验
- 配置治理

重新揉成一团。
