# Timeline

## 0. 起点

最开始并不想直接做完整环境，而是先做一个最小 `OpenCode` box。

问题被拆成：

- 最小启动是否成立
- 稳定入口怎么定
- provider / auth contract 怎么定
- 哪些内容必须后置

## 1. 前期梳理

在真正执行 `Wave 1` 之前，先做了多轮梳理：

- 明确 `VM103` 是主证明面
- 明确 host wrapper 保持薄
- 明确 `OpenSpec` 要按 wave 回填
- 明确不是“整套 playbook 全做完再补文档”

这一阶段虽然不产生“功能完成”，但它把 `Wave 1` 成功所需的不确定性提前消耗掉了。

## 2. Wave 0

`Wave 0` 只证明一件事：

`VM103` 上的最小空盒子能否启动。

结果：

- 证明成功
- `/tmp` 根下的隔离 roots 成立
- 不需要 provider / Docker / Hermes / host API

## 3. Wave 1

`Wave 1` 只解决 operator entry。

做法：

- 不再让 operator 记 proof round
- 给出一个稳定 host entry
- 给出一个 VM103 local entry
- 用一次 `--version` 证明整条入口链成立

为什么它顺：

- 目标单一
- 产物单一
- 成功标准单一
- deferred scopes 明确

## 4. Wave 2

`Wave 2` 一开始名义上是 provider 验证，但实际进入执行后，逐渐混入了：

- contract 发现
- launcher hardening
- env passthrough 设计
- Volta shim / node 路径兼容
- VM103 proof
- 文档更新

所以它不再是纯 wave，而是一个小 phase 被直接装进 wave 里执行。

## 5. Wave 2 的真实收获

虽然性能下降，但最后还是拿到了关键结论：

- stable round 默认 provider-clean
- provider ingress 必须显式 allowlist
- host wrapper 不是 secret bridge
- tool-native auth 仍然 box-local

这使得后续 `Wave 3` 可以更清楚地只做：

- live provider request proof
或
- OAuth migration semantics

## 6. 复盘后的转向

复盘后最大的认识不是“某个命令怎么写”，而是：

长任务不能只在 wave 层做规划。

必须提前定义：

- 整个周期的完成点
- 每个 phase 的问题类型
- 每个 phase 的中止点
- 每个 wave 的单一 proof

否则就会重复出现：

- 前一轮很好
- 下一轮开始混层

