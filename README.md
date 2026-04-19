# OpenCode Box Wave Retrospective

这是一份公开复盘，记录我在 `document-test` / `opencode-box` 这条变更里，如何把一个长任务拆成多轮次 `Wave`，以及为什么 `Wave 1` 很顺、`Wave 2` 出现性能下降。

## 背景

目标不是一次做完整的 `OpenCode` 运行时集成，而是先把最小可复用边界做出来：

- 先证明 `VM103` 上的 `OpenCode` box 能启动
- 再收口稳定 operator entry
- 再定义 provider ingress contract
- 更后面才考虑 `OpenAI OAuth`、host API inheritance、Hermes reuse、Docker

这个顺序后来被证明是对的，但只做到了前半段。

## 结果摘要

### Wave 0

证明了：

- `VM103` 上可以启动最小空盒子
- `systemd-run` + 隔离 `XDG_*` roots 可行
- 这一步不需要 provider auth / Hermes / Docker / host API inheritance

### Wave 1

这是整轮里手感最好的一次。

完成了：

- 选定稳定入口
  - host: `scripts/opencode/run_vm103_box.sh`
  - VM103 local: `planning/opencode-box/run_vm103_box.sh`
- 在 `VM103` 上用 `--version` 证明日常 operator path 可用
- 明确了 durable roots 放在 `/opt/ai-lab/opencode-box`

### Wave 2

这一轮没有失败，但性能明显下降。

最终完成了：

- launcher 默认改成 child env 清空
- provider ingress 改成显式 allowlist
- 新增 `round-02-vm103-provider-env-probe.env`
- 在 `VM103` 上证明：
  - stable round 默认不泄漏 ambient `OPENAI_*`
  - round-02 允许 `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`OPENCODE_API_KEY`
  - `opencode providers list --pure` 在 round-02 下显示
    - `0 credentials`
    - `3 environment variables`
    - `OpenAI`
    - `OpenCode Zen`
    - `OpenCode Go`

## 为什么 Wave 1 快，而 Wave 2 变慢

### Wave 1 快的原因

- 前面已经做了多轮梳理
- 轮次边界清楚
- 只有一个 blocking proof
- 这轮明确知道“不做什么”
- 产物只有一个：稳定入口证明

本质上，`Wave 1` 不是边执行边设计，而是：

`先把设计压实 -> 再跑一个很小的 proof -> 回填 observation -> 结束`

### Wave 2 变慢的原因

不是上下文大小本身，而是这一轮在执行时混进了太多不同层级的事情：

- provider / auth contract 分析
- launcher 实现修正
- VM103 远端验证
- Volta / PATH / clean-env 兼容问题
- OpenSpec 回填

也就是说，`Wave 2` 表面上叫一个 wave，但实际承担了一个小 phase 的工作量。

## 这次最大的教训

只把 `Wave 1` 规划好，不等于把整个任务周期规划好了。

如果长任务只在战术层切得漂亮，而在战略层没有把后续 phase 切开，那么：

- 前一轮会很顺
- 下一轮一旦碰到新 authority surface
- 就会从“单事实 proof”滑成“设计 + 修复 + 验证 + 回填”混合轮

这就是 `Wave 2` 的真实情况。

## 我后来收口出的战略框架

不要只定义 `Wave`，而要同时定义：

1. `Campaign`
   - 这个 change 在本周期到底做到哪里算完成
2. `Phase`
   - 每个阶段只解决一类问题
3. `Wave`
   - 每轮只证明一个事实
4. `Checkpoint`
   - 每轮结束后的压缩点

如果没有 `Campaign / Phase / Wave` 三层一起存在，就很容易出现：

- `Wave 1` 很漂亮
- `Wave 2` 开始发散

## 我最终采用的判断规则

继续执行的条件：

- 这一轮还能用一句话说明白
- 只剩一个 blocking proof
- 本轮只会产出一个 observation / verdict

必须停下来重编排的条件：

- 一轮里同时出现“设计边界 + 修实现 + 做验证”
- 同一轮出现连续两次以上意外修复
- 一轮跑了 10 到 15 分钟还没有单一结论
- 开始碰到新的 authority surface

## 当前结论

这次经历说明：

- `Wave 1` 的模式是有效的
- `Checkpoint + 压缩 + 新 session` 也是有效的
- 但前提是 checkpoint 里必须锁死“下一轮唯一目标”
- 更重要的是，要先把整个大任务周期的 `Phase` 切好

否则，压缩只是减少上下文，不会自动带来更高性能。

## 后续更新

后面继续推进时，又收出一个更细的分层：

- OpenSpec 薄底座
- 方法层
- 具体 change 的当前战略卡
- 配置真相源层

也就是说，后来发现还不能只说 `Campaign / Phase / Wave`。

还得分清：

- 哪些是长期经验
- 哪些是当前任务卡
- 哪些是真实配置来源

否则越往后做，越容易把：

- 当前任务策略
- 长期经验
- 配置真相

重新混在一起。

这也带来了一个新的修正：

有些当时看起来很“重要”的规则，其实不该抬成全局核心，  
它们更适合被降回：

- 重任务经验
- 方法层模板
- 当前阶段常用启发式

而不是装进最上层底座。

## 仓库内容

- [TIMELINE.md](./TIMELINE.md): 事情经过
- [FRAMEWORK.md](./FRAMEWORK.md): 之后可复用的长任务编排方法
- [DOCS-INDEX.md](./DOCS-INDEX.md): 这次补进去的原始文档审查包索引

## 审查包

这次不只更新了概述。

我还把相关原始文档一并打包放进了：

- `docs/source/openspec-methods/`
- `docs/source/stage-pve-vm-agent-lab/`
- `docs/source/deploy-recallnest-shared-memory-on-vm103/`
- `docs/source/design-six-agent-serena-recallnest-adoption/`

这样外部审查者不只看到总结，还能直接看：

- 方法层怎么长出来
- 当前 `RecallNest` / `VM103` 任务卡怎么写
- 旧部署基线长什么样
- 风险 / 反驳证据从哪里来
