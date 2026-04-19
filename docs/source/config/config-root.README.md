# Config Source

这个目录是仓库里的**配置真相源层**。

它和 `openspec/methods/` 不一样。

`openspec/methods/` 放的是：

- 经验
- 模板
- 方法

这里放的是：

- 真正的 durable config
- 结构化配置模板
- 可长期维护的配置来源

## 这里不放什么

这里不放：

- 当前任务诊断卡
- 战略卡经验
- handoff
- wave note

那些内容继续留在 `openspec/changes/` 或 `openspec/methods/`。

## 当前子目录

- [codex/](./codex/)
- [ko/](./ko/)
- [daks/](./daks/)

一句话：

这个目录是“配置真相源”，不是“经验层”。
