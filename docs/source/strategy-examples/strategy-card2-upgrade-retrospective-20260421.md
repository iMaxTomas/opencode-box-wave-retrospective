# Strategic Card 2 Upgrade Retrospective 2026-04-21

这份笔记不是新的长期规则。

它是对“战略卡2升级”这条线的一次结构化复盘，重点回答四件事：

1. 当前战略卡配置的薄弱点是什么
2. 整个战略卡2升级到底讨论了哪些内容
3. 如果原计划包太大，后面应该怎样分阶段升级
4. 为什么 `RecallNest H1-H4` 不能继续当后续验证主样本

---

## 1. 战略卡当前配置的薄弱点

🌳 **战略卡当前配置的薄弱点**
├── 🔴 **1. `Active plan` 约束还不够硬**
│   ├── 一个窗口只服务一个主线，这件事仍然容易漂
│   └── 字段有了，但约束力还不够强
├── 🟠 **2. `Current wave / Continue / Stop` 机制容易失真**
│   ├── wave 容易越跑越宽
│   ├── 该停的时候不停
│   └── 改着改着就开始改 contract / phase
├── 🟡 **3. 长期方法层 vs 当前任务卡的边界还不够稳**
│   ├── 长期方法层已经存在
│   ├── 当前 change 下的战略卡也存在
│   └── 但“什么该进长期规则，什么只该留在当前卡里”还不够清楚
├── 🟢 **4. 命名与口径一致性是薄弱点**
│   ├── 同一个对象容易被重命名、重包一层、再发明新别名
│   └── 配置项之间的概念边界还不够自解释
├── 🔵 **5. 稳定 artifact 命名和恢复锚点不够天然**
│   ├── 恢复很依赖 handoff / checkpoint / 人工记忆
│   └── 配置能跑，但恢复性还不够原生
├── 🟣 **6. 活跃面太容易膨胀**
│   ├── note 越挂越多
│   ├── handoff 越写越厚
│   └── 一条战略卡周围很容易长出一团说明层
├── ⚫ **7. 证据层有了，但“判生死层”还不够硬**
│   ├── 会记录，不等于会裁决
│   └── 会写 note，不等于会 closeout
└── ⚪ **8. 执行 hook 的选择逻辑还不够一眼明白**
    ├── `shell`、same-runtime、`VM103` 都出现了
    └── 但“什么时候必须升级执行路径”还不够傻瓜式

一句话：

`当前战略卡最薄弱的，不是没字段，而是字段有了，但约束力、分层边界、恢复性、裁决力还不够硬。`

---

## 2. 战略卡2升级完整讨论树

🌳 **战略卡2升级完整讨论树**
├── 🎯 **A. 最初想解决什么**
│   ├── 把样板案例里提炼出的战略卡规则升级成长期可复用基线
│   ├── 原本想一起推进：
│   │   `KPI` / `Main-Agent` / `Context-Sharing` / `Runtime Hygiene`
│   └── 想把它们当成一个统一的“战略卡2升级计划”
├── 🧱 **B. 后来明确出来的三层结构**
│   ├── **长期方法层**
│   │   `Goal` / `Non-Goals` / `Stop line` / `Evidence Status` / `Wave Micro-Batches`
│   ├── **受管会话治理层**
│   │   `Sidecar Boundary Rule` / `Preflight` / `Session Outcome Rule`
│   └── **当前任务卡层**
│       `Active plan` / `Current phase` / `Current wave` / `Execution hook` / `Evidence path`
├── 📏 **C. `KPI` 这条支线**
│   ├── `KPI` 不是目标，也不是 todo
│   ├── 它是评估“这轮跑法值不值得继续”的尺子
│   ├── 与 `OpenSpec` 的关系：
│   │   `Goal/Plan/Todo` 管“做什么”
│   │   `战略卡` 管“这一轮怎么跑”
│   │   `KPI` 管“这种跑法值不值得继续”
│   └── 这轮只证明到：`KPI` 作为评估面可用
├── 👤 **D. 协作对象这条支线**
│   ├── `Main-Agent`：谁冻结 slice、谁做最终判断
│   ├── `Context-Sharing`：brief / handoff / packet / 命名口径怎么传
│   └── `Runtime Hygiene`：哪步在哪个 runtime、边界有没有漂
├── 🖥️ **E. `host` / `VM103` 这条支线**
│   ├── 不是默认必须上 `VM103`
│   ├── 核心比较对象是：
│   │   `host-only` vs `host + VM103 bounded collaboration`
│   └── 当前 document-heavy、repo-local 的 slice 上，`host-only` 就够
├── 🧪 **F. `non-RecallNest sample` 这条支线**
│   ├── 不是禁用 RecallNest 工具
│   ├── 而是不要再拿 `RecallNest H1-H4` 当主样本
│   └── 后来样本切到：
│       `govern-multi-runtime-collaboration-and-subagent-configuration`
├── 📍 **G. Phase 3 实际怎么讨论和收口**
│   ├── `Phase 3 = bounded non-RecallNest validation phase`
│   ├── 入口卡、host-only verdict、closeout recommendation、final closeout 都落了
│   └── 当前结论：
│       `KPI` 这轮可用，`host-only` 足够，`host + VM103` 不必强补
├── ❌ **H. 对原“战略卡2升级计划”的反思**
│   ├── 包太大
│   ├── 成熟度不一致
│   ├── 把尺子和对象混在一起
│   ├── 样板味太重
│   ├── 验证 slice 太窄，承载不了整包升级
│   └── 名字本身也误导，天然让人按整包推进
├── 🩹 **I. 当前配置暴露出的薄弱点**
│   ├── `Active plan` 约束不够硬
│   ├── `Current wave / Continue / Stop` 容易漂
│   ├── 长期方法层与当前任务卡边界容易混
│   ├── 恢复锚点不够原生
│   ├── 文档活跃面容易膨胀
│   ├── 证据层有了，但裁决层还不够硬
│   └── `Execution hook` 路由规则还不够一眼明白
├── 🛠️ **J. 后来形成的新推进思路**
│   ├── 不再整包硬升
│   ├── 先做能跑完整闭环的战略卡配置 MVP
│   └── 再按薄弱点逐项加硬：
│       `壳子 -> 路由 -> 裁决 -> 最小协作包`
├── 📦 **K. “最小协作包 v1”**
│   ├── 候选对象仍是：
│   │   `KPI + Main-Agent + Context-Sharing + Runtime Hygiene`
│   ├── 但不再整包直接升格
│   └── 而是做短周期判生死的小 bundle
├── 🏛️ **L. “长期基线”**
│   ├── 长期基线 = 以后跨任务默认复用的正式规则
│   ├── 不是 handoff
│   ├── 不是一次性验证 note
│   └── 不是某条样板线里的局部经验
├── 🧭 **M. “战略卡运行逻辑”**
│   ├── `Active plan -> Current phase -> Current wave`
│   ├── `Execution hook -> Evidence path`
│   ├── `restore + orient -> collect + verify -> synthesize + handoff`
│   └── 最后 `continue / stop / closeout`
├── 🏢 **N. 公司比喻这条支线**
│   ├── `OpenSpec` 像项目立项书 / 甘特图
│   ├── `战略卡` 像这一轮项目怎么组织、谁主导、到哪停
│   ├── `KPI` 像复盘仪表盘
│   └── `Main-Agent / Context-Sharing / Runtime Hygiene`
│       像项目经理、共享资料流、流程纪律
└── 🚧 **O. 未来方向**
    ├── 一周内不现实的是“整套战略卡2全部升完”
    ├── 更现实的是把它压成可判生死的小包
    └── 当前最合理的下一步，是决定要不要新开 follow-up change

一句话：

`这条线讨论的，不只是 KPI 或 Phase 3，而是战略卡本体如何分层、如何裁决、如何从整包升级改成分阶段升级。`

---

## 3. 分阶段升级总思路

🌳 **分阶段升级总思路**
├── 🔵 **核心改法**
│   ├── 不是按概念拆
│   └── 而是按成熟度拆：
│       `先升壳子 -> 再升路由 -> 再升裁决 -> 最后才升协作包`
├── 🟩 **Phase 1：控制壳子 MVP**
│   ├── 固化：
│   │   `Active plan` / `Current phase` / `Current wave` / `Stop line` /
│   │   `Output artifact` / `Evidence path`
│   ├── 目标：
│   │   `先让战略卡能稳定跑完一轮闭环`
│   └── 解决：
│       主线漂移、wave 失控、恢复入口不稳、命名混乱
├── 🟨 **Phase 2：执行路由 MVP**
│   ├── 固化：
│   │   `Execution hook` / `Preflight` / `Sidecar Boundary Rule`
│   ├── 规则：
│   │   `shell first -> same-runtime -> VM103 only when needed`
│   └── 解决：
│       什么时候上 sidecar、什么时候上 `VM103`、什么时候必须单 owner
├── 🟧 **Phase 3：证据与裁决 MVP**
│   ├── 固化：
│   │   `Evidence Status` / `Entry gate` / `Exit gate` / `verdict` / `closeout`
│   ├── 目标：
│   │   `让战略卡不只会记录，还会判定`
│   └── 解决：
│       证据很多但结论太软、会写 note 不会收口
└── 🟥 **Phase 4：最小协作包 v1**
    ├── 处理对象：
    │   `KPI` + `Main-Agent` + `Context-Sharing` + `Runtime Hygiene`
    ├── 升级方式：
    │   用 `1-2` 个样本做短周期判生死
    └── 退出标准：
        `进长期基线 / 退回候选`

一句话：

`针对“包太大”的正确改法，不是把大包均匀切碎，而是按成熟度分四层升级，先壳子、后路由、再裁决、最后协作包。`

---

## 4. RecallNest H1-H4 主样本讨论树

🌳 **RecallNest H1-H4 主样本讨论树**
├── 📍 **1. 它原来扮演什么角色**
│   ├── 这是早期最重要的大样板案例
│   ├── 很多战略卡2的想法，都是从这条线里提炼出来的
│   └── `KPI` 也一度主要从这条样板里长出来
├── ⚠️ **2. 为什么后来不再继续拿它当主样本**
│   ├── 样板味太重
│   ├── 和 `RecallNest` 本身绑定太深
│   ├── 容易把样板经验误当成跨任务通用规则
│   └── 会变成“拿同一条线反复证明自己”
├── 📏 **3. 它和 `KPI` 的关系**
│   ├── `KPI` 很大程度上就是从 `RecallNest H1-H4` 的协作经验里长出来的
│   └── 所以后来担心：
│       `如果继续只拿 H1-H4 验 KPI，就会太自指`
├── 🔄 **4. 后来如何重新定位它**
│   ├── 不再把它当后续 Phase 3 的主验证样本
│   ├── 而把它当成：
│   │   来源案例 / 提炼母体 / 已经用过的重样板
│   └── 后续样本切到：
│       `govern-multi-runtime-collaboration-and-subagent-configuration`
└── 🧠 **5. 这条支线真正留下的反思**
    ├── `RecallNest H1-H4` 适合拿来提炼规则
    └── 但不适合一直拿来证明规则普适

一句话：

`RecallNest H1-H4` 不是没价值，而是更适合当“规则来源样板”，不再适合继续当“后续验证主样本”。`
