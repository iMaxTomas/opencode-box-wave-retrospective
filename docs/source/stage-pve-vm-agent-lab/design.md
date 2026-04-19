## Context

当前已经确认的现实基线是：

- `PVE` 宿主机可通过 `root@172.29.111.220` 访问
- 实验客体机是 `VM 103 (debian-openclaw)`
- `103` 当前有基线快照 `base-clean-20260325`
- `103` 当前 IPv4 为 `172.29.111.221`
- 本机已经可以通过 `ssh vm103-openclaw` 直接免密登录到客体机
- 客体机中已经建立了：
  - `/opt/ai-lab/bootstrap`
  - `/opt/ai-lab/checks`
  - `/opt/ai-lab/docs`
  - `/opt/ai-lab/logs`
  - `/srv/ai-artifacts`
- 独立的 secrets VM 已经建立为 `VM 104 (debian-bitwarden)`
- `104` 当前 IPv4 为 `172.29.111.222`
- `104` 当前已有快照：
  - `base-clean-20260325-bitwarden`
  - `pre-bitwarden-live-input-20260325-1315`
  - `pre-enable-https-20260325-1345`
  - `pre-custom-ca-20260325-1352`
- `104` 当前已经切换到 HTTPS，并发现 Bitwarden Lite 默认自动生成的自签证书不足以作为 `bw` CLI 的长期基线
- `103` 当前已经信任自定义 Bitwarden 根 CA，并已能把 `bw` CLI server 指向 `https://172.29.111.222`
- `103` 当前 root 用户已经安装 `codex-cli 0.116.0`，并且 `codex login status` 返回 `Logged in using ChatGPT`
- `103` 当前 root `Codex` home 已经包含 `/root/.codex/skills/smart-web-fetch`
- `103` 当前 root `Codex` 指令文件已存在 `/root/.codex/AGENTS.md -> @RTK.md` 与 `/root/.codex/RTK.md`
- `103` 当前已经把主线程最小真实 `Codex` 调用证据写入 `/opt/ai-lab/docs/codex-first-real-call-20260325.md`
- `103` 当前已经把显式 explorer subagent 的只读 side task 证据写入 `/opt/ai-lab/docs/codex-subagent-read-heavy-20260325.md`
- `103` 当前已经安装 `Claude Code 2.1.83`
- `103` 当前已经把最小真实 `Claude Code` 调用证据写入 `/opt/ai-lab/docs/claude-first-real-call-20260325.md`
- `103` 当前已经安装 `OpenClaw 2026.3.23-2`
- `103` 当前已经把最小 `OpenClaw` CLI 启动与帮助面证据写入 `/opt/ai-lab/docs/openclaw-minimal-check-20260325.md`
- `103` 当前已经安装 `codex-cli 0.116.0`，并且 `root` 下的 `Codex` 登录态可直接使用
- `103` 当前已经完成一次 `Codex` 主线程最小真实调用，证据落在 `/opt/ai-lab/docs/codex-first-real-call-20260325.md`
- `103` 当前已经完成一次显式 `Codex` read-heavy subagent side task，证据落在 `/opt/ai-lab/docs/codex-subagent-read-heavy-20260325.md`
- `103` 当前的 `codex-api-openai-prod` lane 已经能通过 `Bitwarden` 注入 `wantslabs-full-api`，并以 `provider base_url=https://api.wantslabs.com/v1` 完成最小真实调用；证据写入 `/opt/ai-lab/docs/codex-api-wantslabs-first-real-call-20260325.md`
- `103` 当前的 `Codex` 全局上下文里已经存在 `/root/.codex/AGENTS.md` 与 `/root/.codex/RTK.md`，因此 `RTK` 目前以全局 instruction mode 形式生效；lane-specific OAuth 隔离仍保留为后续收敛项

因此当前最合适的推进方式不是继续讨论抽象的“长期 AI 工作台”，而是先把现成的 `103` 变成一个稳定的实验室：先验证，后推广；先保留检查点，后做高风险改动；先明确阶段边界，后谈统一入口和控制平面。

## Goals / Non-Goals

**Goals:**
- 把 `103` 固化为当前的唯一实验客体机基线
- 把 `PVE` 宿主机职责限制在电源、快照、网络和资源管理
- 把本机到客体机的访问路径固定为可重复的 SSH 入口
- 按阶段推进 `Codex -> Claude Code -> Gemini -> OpenClaw`
- 为每个阶段定义进入条件、验证动作、退出条件和回滚点
- 为脚本、文档、日志和派生产物定义固定落盘位置
- 把“是否值得推广到正式环境”变成一个显式评审，而不是默认结果
- 让 `Codex` 的并行工作方式对齐官方 `Subagents / Worktrees` 标准，而不是继续停留在纯人工多会话约定

**Non-Goals:**
- 本 change 不重构 `PVE` 存储架构
- 本 change 不把 `nvme4n1` 纳入首轮实施路径
- 本 change 不要求一开始就统一 `Codex / Claude Code / Gemini / OpenClaw` 的控制平面
- 本 change 不直接把实验室配置回灌到正式主力机器
- 本 change 不要求当前就接入完整的 observability、memory 或 DVC 协作层

## Decisions

### 1. 当前阶段直接复用 `103`，而不是先克隆新 VM
你已经明确说明 `103` 是干净的，而且此前安装后没有继续漂移；同时基线快照也已经创建。因此当前第一轮不再额外克隆新 VM。

原因：
- 降低首轮推进成本
- 避免为“更干净的干净机”继续推迟验证
- 保留以后从该快照克隆新 VM 的空间

替代方案：
- 先从 `103` 克隆一台新 VM 再开始  
  放弃原因：当前不会带来实质性风险下降，只会增加操作和命名成本。

### 2. 访问链固定为“本机直连客体机”，不是“先登宿主机再跳转”
当前工作流固定为：

`我从本机执行 ssh vm103-openclaw -> SSH 直接进入 103 -> 我在 103 内执行安装和验证 -> 结果写入 103 内的固定目录 -> 下次我继续使用同一 SSH 入口复用这些产物`

不是这样：
- 不是每次都先 SSH 到 `PVE` 再手工进入 `103`
- 不是把宿主机当成日常工作面

原因：
- 降低日常进入成本
- 降低误操作宿主机的概率
- 更容易把“宿主机职责”和“客体机职责”分开

### 3. 宿主机和客体机分层治理
`PVE` 宿主机只负责：
- 启停
- 快照
- 资源配置
- 网络和 cloud-init 检查

`103` 只负责：
- 基础环境
- 四套工具的安装和验证
- 脚本、文档、日志和派生产物

原因：
- 防止宿主机配置和客体机实验同时漂移
- 回滚边界更清晰

替代方案：
- 直接在宿主机上安装和验证工具  
  放弃原因：会让实验风险直接落到基础设施层。

### 4. 使用阶段门控而不是“一次装完”
阶段顺序固定为：

1. 预检查与系统基线
2. 基础环境 bootstrap
3. 凭据与用户模型
4. `Codex`
5. `Claude Code`
6. `Gemini`
7. `OpenClaw`
8. 可选的工具间协作验证
9. 推广评审

每一阶段都必须满足：
- 进入前有明确快照
- 阶段内有最小验证动作
- 阶段结束后有结果记录

原因：
- 问题归因清晰
- 失败回滚便宜
- 不会因为四套工具互相覆盖依赖而失去定位能力

替代方案：
- 四套工具一起安装后统一排障  
  放弃原因：一旦失败，无法快速判断是依赖冲突、认证问题还是环境基线问题。

### 5. 实验室目录固定，所有可复用结果必须落盘
当前实验室目录固定为：
- `/opt/ai-lab/bootstrap`
- `/opt/ai-lab/checks`
- `/opt/ai-lab/docs`
- `/opt/ai-lab/logs`
- `/srv/ai-artifacts`

原因：
- 下次进入时不需要重新想“脚本应该放哪里”
- 能把成功步骤和失败证据都变成可复用资产

替代方案：
- 临时散落在 `/root`、shell history、聊天记录里  
  放弃原因：复用成本高，且难以判断当前状态。

### 6. Secrets 不进入脚本正文，并以 `Bitwarden self-host` 作为主方案
第一轮部署中，脚本只保存 `.env.example`、说明文档、变量名或 secret templates，不直接写入真实密钥。主 secrets 方案选择 `Bitwarden self-host`；`103` 当前阶段只作为 secrets 消费方，不承担 secrets 服务端本体。

原因：
- 四套工具的认证方式不同
- 用户明确把“方便本地部署”作为首要约束
- `Bitwarden` 官方支持完整自托管部署，适合后续多端同步和本地控制
- 避免把 secrets 服务与实验客体机 `103` 绑定在一起
- 降低快照、脚本和模板泄漏真实密钥的风险

替代方案：
- 把密钥直接写进脚本或 cloud-init  
  放弃原因：后续替换、轮换和清理都很麻烦。
- 使用 `1Password + 1Password CLI` 作为首选  
  放弃原因：虽然官方支持 CLI、Connect Server 和运行时注入，但它不是完整的本地自托管 vault，和当前“本地部署优先”的约束不完全一致。

### 7. 日常操作用户当前保留 `root`，但把低级误操作控制当成显式风险
首轮执行中，`103` 继续使用 `root` 作为默认 SSH 和操作用户，不额外引入 `imax + sudo` 用户切换。

原因：
- 当前已经跑通 `root` 免密 SSH
- 先减少用户模型切换，优先把时间花在环境和工具落地上
- 可以把“用户模型调整”留到后续需要时再做

替代方案：
- 立即切换到 `imax + sudo`  
  放弃原因：会增加首轮迁移和验证变量，不符合当前“先跑通再收敛”的目标。

### 8. 首轮磁盘容量直接扩到 `80G`
当前不再等待“第一轮后再判断”，而是直接把 `103` 从 `40G` 扩到 `80G`。

原因：
- 当前 `PVE` 还有空间
- 这能降低 Node、Python、Docker、日志和工具缓存对首轮验证的阻塞
- 比起在快装完时才被空间问题打断，提前扩容更省时

替代方案：
- 继续以 `40G` 开始，首轮后再决定  
  放弃原因：当前已知容量偏紧，没有必要让可预见问题继续变成中途阻塞。

### 9. 工具接入采用 `Codex-first`
当前执行优先级固定为：

`我先把 103 的基础环境准备好 -> 我优先把 Codex 配好并完成最小真实任务验证 -> 之后再按顺序处理 Claude Code、Gemini、OpenClaw`

原因：
- `Codex` 是当前最直接的执行器
- 先用一套工具跑通基础链路，比四套同时推进更容易定位问题

替代方案：
- 四套工具平均推进  
  放弃原因：会稀释当前时间窗口，并增加排障维度。

### 9A. `Codex` 并行遵循官方 `Subagents` 语义，而不是把多个终端窗口当成正式团队模式
当前 `Codex` 并行链固定为：

`我先在主线程锁定当前 slice 的目标 -> 我先完成主线程最小真实调用 -> 只有当某个子任务真的能独立并行时，我才显式要求 Codex spawn subagents -> 子代理只返回摘要、证据或 diff -> 主线程决定是否集成`

不是这样：
- 不是默认开很多窗口就算官方 multi-agent
- 不是主线程一上来就把阻塞性的下一步也丢给子代理

原因：
- 官方 `Codex` 文档明确写了 subagents 只有在显式要求时才会生成
- 官方也明确把 subagents 的优势放在 exploration、tests、triage、summarization 这类读重任务上

替代方案：
- 继续把“main / research / review”只当成人工命名会话  
  放弃原因：这能继续提供操作提示，但还不等于官方支持的 subagent workflow。

### 9B. 当前 `103` 先使用内置 `default / explorer / worker`，暂不扩张自定义 agent 体系
当前 `Codex` 角色基线固定为：

- `default`: 主线程与一般任务兜底
- `explorer`: 只读探索、盘点、定位
- `worker`: 边界明确的执行任务

原因：
- 当前实验室主线的缺口在流程落地，不在 agent 数量
- 先用官方内置角色，更容易和官方文档、CLI 行为以及后续复盘保持一致

替代方案：
- 先为 lane、provider、工具分别定义大量自定义 agents  
  放弃原因：当前主线还没完成首个真实 `Codex` 调用，过早扩张代理层只会增加配置面。

### 9C. 写重并行默认不进入共享工作区；需要并行写时，优先进入 Git worktree
当前并行写规则固定为：

`我先判断这是读重还是写重任务 -> 读重任务可以交给 subagents 并行 -> 写重任务默认留在主线程或单 worker -> 只有当目标目录是 Git 仓库且确实需要并行写时，我才进入 worktree / handoff 路径`

不是这样：
- 不是多个 agent 直接同时改同一个共享目录
- 不是把额外开几个终端当作 worktree 的替代品

原因：
- 官方 worktree 文档的重点就是“并行任务互不干扰”
- 当前 `103` 上的 `/opt/ai-lab` 和部分实验目录还没有统一成 Git worktree 驱动，所以写并行要更保守

替代方案：
- 完全靠口头约定“别撞文件”  
  放弃原因：这在复杂阶段下容易把冲突和恢复成本重新推高。

### 9D. 当前先接受 root-level `Codex` 主线程验证，再把 lane-local OAuth 作为下一认证隔离里程碑
当前 `Codex` 验证链固定为：

`我先在 103 上直接运行已登录的 root-level Codex -> Codex 在主线程里完成一个最小真实只读任务 -> 结果写入 /opt/ai-lab/docs/codex-first-real-call-20260325.md -> 然后我再显式要求它 spawn 一个 explorer subagent 做读重 side task -> 结果写入 /opt/ai-lab/docs/codex-subagent-read-heavy-20260325.md -> 下次我继续沿这些证据推进 lane-local OAuth 或 API lane 的隔离验证`

不是这样：
- 不是等 lane-local OAuth 全部补完后，当前 `Codex` 才算第一次真实可用
- 不是把 root-level 已登录状态和 lane-local 运行态隔离混成同一个验收点

原因：
- 你已经明确说明 `103` 上现有 `Codex` 登录可以直接使用
- 当前 `4.1` 的目标是先证明客体机里的主线程 `Codex` 能执行真实任务
- lane-local OAuth 仍然重要，但它属于下一层身份隔离和复用治理，不应继续阻塞第一个真实调用

替代方案：
- 先停下来迁移现有登录到 lane runtime root，再允许主线程真实调用  
  放弃原因：会把“第一次可用”与“后续隔离收敛”绑死，拖慢主线推进。

### 10. 实验室可复用部分采用单独 Git 仓库存放，运行态与产物分离
`/opt/ai-lab` 中的脚本、模板、文档和检查逻辑应进入单独 Git 仓库；日志、缓存、真实 secrets、运行态文件和大体积产物继续留在 Git 外。

原因：
- 这最适合未来复制到新 VM
- 也最适合回头把经过验证的部分推广到正式环境
- 能避免把噪音文件混进可复用模板

替代方案：
- 全部都不进 Git  
  放弃原因：后续复制和对比成本太高。
- 把日志和运行态也放进 Git  
  放弃原因：会让仓库迅速污染且不利于复用。

### 11. 推广到正式环境必须晚于实验室通过
这条链固定为：

`我先在 103 完成阶段验证 -> 103 里的脚本和记录产生产物 -> 我根据产物决定哪些步骤值得推广 -> 只有通过推广评审的部分才进入正式机器`

不是这样：
- 不是实验室里“装成了”就自动等于正式方案
- 不是把宿主机和主力电脑当作第一验证场

原因：
- 保护正式环境
- 保留试错自由
- 让“反哺学习”建立在真实可复盘的记录上

### 12. 恢复策略采用“双层”：快照负责快速回滚，备份负责长期恢复
当前恢复策略固定为：

`我在高风险改动前创建 pre-* 快照 -> 我在 VM 103 中完成改动和验证 -> 我只把验证通过的阶段状态送入长期备份 -> 出现小范围问题时优先恢复快照 -> 出现快照不可用、历史过深或宿主机侧问题时再从备份恢复`

不是这样：
- 不是把快照当作长期备份体系
- 不是要求每次 pre-* 快照在同一秒同步成备份

原因：
- 快照更适合高频、低延迟的本机回滚
- 备份更适合长期保存、跨时间点恢复和应对快照失效
- 只有验证通过后的状态才值得进入长期备份链，避免把未验证或错误状态反复固化

替代方案：
- 每次快照都立即视作备份  
  放弃原因：会混淆回滚点和长期恢复点，并放大 thin-pool 压力。
- 只做备份、不做快照  
  放弃原因：日常实验回滚成本太高，不适合当前高频迭代。

### 13. `Bitwarden` 服务端与 `103` 分离，当前先采用独立 `VM 104`
当前 secrets 路径固定为：

`我在 103 上启动某个 lane wrapper -> wrapper 向独立的 Bitwarden 实例请求这一条 lane 允许使用的 secrets -> secrets 只注入当前这次进程 -> 工具开始运行 -> 下次我继续用同一条 lane`

不是这样：
- 不是把 Bitwarden 服务端装进 `103`
- 不是让工作机同时扮演 secrets 服务端和 secrets 消费端
- 不是把 `PVE` 宿主机直接当 Bitwarden 业务机

当前首选部署形态为：
- `PVE` 宿主机保留控制平面职责
- `VM 103` 保留 AI 工作机职责
- 新的 `VM 104` 保留 Bitwarden self-host 职责

当前已经落地的实施状态是：
- `104` 已从 Debian cloud image 创建
- `104` 已安装 Docker Engine 和 Docker Compose plugin
- Bitwarden Lite 官方 upstream 模板已下载到 `/opt/bitwarden-lite/upstream`
- 仍故意停在“等待安装 ID、安装 key、域名和首次注册”这一步

原因：
- 避免 `103` 损坏时出现“工作机坏了，但 secrets 也在工作机里面”的循环依赖
- 避免把宿主机继续业务化
- 当前 `PVE` 已有现成 Debian cloud image，创建一台小型 VM 的成本很低

替代方案：
- 把 Bitwarden 放进 `103`  
  放弃原因：恢复边界和职责边界都变差。
- 把 Bitwarden 直接装在 `PVE` 宿主机  
  放弃原因：会污染宿主机职责。

### 14. 使用 lane 作为账号、provider 和运行态的隔离单位
当前运行模型固定为：

`我不直接裸跑 codex 或 claude -> 我只运行一个明确命名的 lane wrapper -> wrapper 固定它自己的 runtime root、日志目录、provider 变量和 secrets 取用方式 -> 结果只写回这一条 lane`

当前 stable layout 增补为：
- `/opt/ai-lab/bin`
- `/opt/ai-lab/templates`
- `/opt/ai-lab/docs/lanes`
- `/srv/ai-runtime/<lane>`
- `/srv/ai-artifacts/lanes/<lane>`

lane 命名格式固定为：
- `<tool>-<authmode>-<provider>-<label>`

示例：
- `codex-oauth-work`
- `codex-oauth-personal`
- `codex-api-openai-prod`
- `claude-api-anthropic-prod`

原因：
- `Codex` OAuth 身份和 API/provider 差异不是同一层问题
- `Claude Code` 的 provider 切换也不应和别的 lane 共用同一批用户态 state
- 让“下次我该跑哪个入口”变得明确，而不是继续记忆一组全局环境变量

### 15. 先完成 non-secret skeleton，再要求用户输入 live credentials
当前 handoff 规则固定为：

`我先把 Bitwarden 实例、103 的 consumer 工具、lane 目录、wrapper、模板和失败时的提示都准备好 -> 我验证这些 wrapper 在 secrets 缺失时会干净失败 -> 只有到这个阶段结束时，才要求用户输入安装 ID、完成 Web 注册、录入 API key，或在各自 lane 中完成 OAuth 登录`

不是这样：
- 不是一边搭目录一边让用户立刻填 secrets
- 不是先把 live credentials 写进去，再回头整理结构

原因：
- 用户输入点更少
- 更容易回滚
- 更容易判断问题是“环境还没好”还是“真实凭据本身有问题”

补充说明：

### 15A. 当前 `Claude Code` 先接受“Bitwarden item -> 当前进程环境注入 -> wantslabs gateway”的最小验证
当前 `Claude` 验证链固定为：

`我先在 103 上从 Bitwarden 读取 wantslabs-full-api -> 当前 shell 只给这次 Claude 进程注入 ANTHROPIC_API_KEY 和 ANTHROPIC_BASE_URL -> Claude 用最小非交互调用返回 OK -> 结果写入 /opt/ai-lab/docs/claude-first-real-call-20260325.md -> 下次我再把这条 provider 路径收敛回 lane wrapper`

不是这样：
- 不是当前 `/opt/ai-lab/bin/claude-lane` 已经自动完成了这条 provider 路径
- 不是把 gateway key 永久写进全局 shell 配置

原因：
- 现有 wantslabs gateway 已经能提供最快的真实调用证明
- `Claude Code` 安装与认证验证不需要再等待 wrapper 重构完成
- 这样可以把“工具已可用”和“lane wrapper 仍待收敛”分开记录

替代方案：
- 等 `claude-lane` 完整接上 Bitwarden 和 provider 后，再允许 `Claude Code` 计入本阶段验证  
  放弃原因：会把真实可用性继续拖到 wrapper 收敛之后，不符合当前主线的推进优先级。
- 这一步结束后，handoff 还必须明确告诉操作员下一步顺序是“先完成 Web 注册，再录 API items，再做 Codex OAuth lane”
- `Codex OAuth` token 仍然保留在各自 lane 的本地运行态里，不作为 Bitwarden 主运行态保存

### 15B. 当前 `Codex API lane` 已接受“Bitwarden item -> lane wrapper -> wantslabs /v1 responses”的最小验证
当前 `Codex API lane` 验证链固定为：

`我先在 103 上运行 codex-api-openai-prod -> lane wrapper 从 Bitwarden 读取 wantslabs-full-api -> wrapper 把 provider base_url 固定到 https://api.wantslabs.com/v1 -> Codex 直接对 /v1/responses 发起最小真实调用 -> 结果写入 /opt/ai-lab/docs/codex-api-wantslabs-first-real-call-20260325.md -> 下次我继续复用同一个 lane`

不是这样：
- 不是 wantslabs 不支持 `Responses` 或 SSE
- 不是 `Bitwarden` 注入失败
- 不是 `Codex` 本体 OAuth 登录态损坏

当前已确认的根因：
- 之前 lane 里的 `PROVIDER_BASE_URL` 少了 `/v1`
- `Codex` 因此实际打到了 `https://api.wantslabs.com/responses`
- 该路径返回的是网关前台 HTML，而不是 API stream

原因：
- 这个根因比“上游流式协议坏掉”更具体，也更容易稳定复用
- 修正后已经能用同一个 lane 返回 `OK`
- 因此当前不再把 `wantslabs` 兼容性当成主线阻塞

### 16. `Gemini` 设计保持兼容，但当前实施主线先不要求立即接入
当前 change 仍保留 `Gemini` 作为未来兼容目标，但首轮执行主线固定为：

`Bitwarden -> Codex lane skeleton -> Codex 真实验证 -> Claude Code lane skeleton -> Claude Code 真实验证 -> OpenClaw scope review`

原因：
- 当前对话已经明确“暂时没有 Gemini”
- 继续把 `Gemini` 放在主线里只会增加当前阻塞面
- 但 lane 和 secrets 设计仍然需要对未来 `Gemini` 兼容

### 17. Bitwarden TLS 采用自定义 CA + SAN 证书，不使用默认自动证书作为 CLI 基线
当前 TLS 路径固定为：

`我打开 https://172.29.111.222 -> 104 返回一张带 IP/DNS SAN 的服务器证书 -> 103 先信任对应根 CA -> bw CLI 再连接并消费 vault -> lane wrapper 才能稳定把 secrets 注入当前进程`

不是这样：
- 不是浏览器能点过告警就等于 `bw` CLI 也能正常工作
- 不是把 `NODE_TLS_REJECT_UNAUTHORIZED=0` 之类的降级选项当成正式方案
- 不是继续依赖 Bitwarden Lite 默认自动生成的自签证书作为 CLI 长期基线

原因：
- `bw` CLI 对证书校验比浏览器更严格，要求访问端点和证书 `SAN` 可匹配
- 默认自动证书会导致“网页可访问、CLI 不可消费”的分裂状态
- 自定义 CA + SAN 证书可以同时覆盖浏览器访问、`103` CLI 消费和后续 lane 自动化

### 18. 当前 `OpenClaw` 首轮只收口到 CLI 可启动与帮助面可用，不在本 change 中展开完整 Gateway/Channel onboarding
当前 `OpenClaw` 验证链固定为：

`我先在 103 上安装 openclaw@latest -> 我验证 openclaw --version、openclaw --help、openclaw doctor --help 都可正常响应 -> 我把结果写入 /opt/ai-lab/docs/openclaw-minimal-check-20260325.md -> 下次如果要继续接入 Gateway、channels、workspace、daemon，再单独进入 follow-up`

不是这样：
- 不是当前就把 `OpenClaw` 的 gateway、pairing、channels、workspace bootstrap 一次做完
- 不是把 `OpenClaw` 的完整产品化接入和今天的实验室主线绑死

原因：
- 本 change 对 `OpenClaw` 的任务定义本来就是“minimal startup and usability check”
- 当前主线的目标是把多工具实验室跑到“可启动、可验证、可分叉”，不是把每一套工具都拉到完整运营态
- `OpenClaw` 的后续价值更偏长期控制平面和多渠道助手，不该继续拖慢当前 change 的收口

### 19. 推广评审当前先固定“哪些值得推广，哪些继续留在 lab”
当前推广判断固定为：

`我先把已经反复验证过的底座、目录规则、快照/备份节奏、Bitwarden 分层、Codex 官方 subagent/worktree 规则、Codex API lane 的 /v1 provider 模式、Claude/OpenClaw 的最小安装验证视为可推广候选 -> 我把 root 登录态、VM103 本机路径、临时 session 文件、以及任何未完成 lane 收敛的 provider path 继续留在 lab`

当前适合推广到后续正式环境或模板机的部分：
- `PVE host` / `VM103` / `VM104` 的职责分层
- `/opt/ai-lab` 与 `/srv/ai-artifacts` 的目录边界
- pre-* snapshot + post-validation backup 的节奏
- `Bitwarden` 服务端与工作机分离的策略
- `Codex` 官方 `Subagents / Worktrees` 基线
- `Codex API lane -> Bitwarden item -> /v1 responses provider` 的最小工作模式
- `Claude Code` 与 `OpenClaw` 的安装路径和最小验证方式

当前继续留在 lab 的部分：
- root 下现有 `Codex` OAuth 登录态
- `/run/ai-lab/bw-session.env` 这类机器本地运行态
- `RTK` 当前的 root-level instruction mode
- `OpenClaw` 尚未完成的 gateway / channel / daemon onboarding

### 20. 当前剩余工作拆分方式固定：`Gemini` 和更深的 provider/lane 收敛进入 follow-up
当前拆分判断固定为：

`我把今天已经证明可用的主线收口在本 change 内 -> 我把仍然需要额外 provider 兼容性、账号治理、或完整产品 onboarding 的部分拆到 follow-up change`

留在本 change 的完成标准：
- `Codex` 主线程与官方 subagent 已验证
- `Codex API lane` 通过 `wantslabs /v1 responses` 的最小真实调用已验证
- `Claude Code` 最小真实调用已验证
- `OpenClaw` 最小启动与帮助面已验证
- 推广边界和 follow-up 边界已经写清楚

进入 follow-up change 的工作：
- `Gemini` 如果重新回到 scope，再单独接入
- `Codex` lane-local OAuth 收敛
- `claude-lane` 自动消费同一 provider 路径
- `OpenClaw` 的 gateway / channels / daemon / workspace onboarding

### 21. 之前的开放问题现在按当前阶段收敛为固定结论
- 用户模型：当前实验室继续使用 `root`；推广时再决定是否切到专用用户
- secret handling：API keys 继续以 `Bitwarden on VM104` 为主；OAuth token 继续留在各工具本地运行态
- `OpenClaw` scope：当前 change 只做到 CLI 最小启动验证，不扩展到完整个人助手流程
- cloning strategy：当前不立即克隆新机；等 post-validation snapshot + backup 更稳定后，再按需要分叉新 VM

## Risks / Trade-offs

- [风险] `80G` 仍可能被大型缓存、镜像和日志逐步侵蚀 -> 缓解：把空间巡检纳入基线检查，并把运行态和可复用模板分开
- [风险] 继续以 `root` 作为日常登录用户会扩大误操作和密钥暴露影响面 -> 缓解：保留 `root` 仅作为当前阶段决策，同时把高风险操作前快照和脚本化执行作为约束
- [风险] 四套工具需要的系统依赖或运行时版本互相冲突 -> 缓解：按阶段安装、按阶段快照、按阶段验证
- [风险] 快照越来越多但没有命名规则和用途记录 -> 缓解：把快照命名和用途写入 `/opt/ai-lab/docs`
- [风险] `PVE` 的 `lvmthin` 在快照增多时会出现 thin-pool 余量压力和 overcommit 警告 -> 缓解：记录 snapshot retention 规则、监控 thin-pool 使用率、避免把快照当作长期备份
- [风险] 如果备份时点和验证时点混乱，长期备份中会混入未验证状态 -> 缓解：只在阶段验证通过后触发长期备份，明确 pre-* 快照与 post-validation backup 的先后关系
- [风险] 把 secrets 服务直接部署到 `103` 会污染实验客体机边界 -> 缓解：`103` 只消费 secrets，secrets 服务本体单独规划
- [风险] self-host secrets 会引入额外运维负担 -> 缓解：先把消费方接口和模板定住，再单独规划服务端部署
- [风险] `OpenClaw` 最后接入，可能暴露更复杂的运行时需求 -> 缓解：将其放在最后一阶段，并允许拆出 follow-up change

## Migration Plan

1. 固定当前基线事实，并把访问路径、快照名、目录结构写入本 change
2. 扩容 `103` 到 `80G`，并完成 guest 侧识别与文件系统扩展
3. 在 `103` 内完成基础环境 bootstrap 和系统级最小整理
4. 明确 secrets 注入方式，并把模板、运行态、Git 跟踪范围分开
4. 依次安装并验证 `Codex`、`Claude Code`、`Gemini`、`OpenClaw`
5. 每一阶段完成后记录验证结果、残留问题和回滚点
6. 在四套工具最小验证完成后，决定是否需要克隆或拆分 follow-up changes
7. 最后才评估哪些内容值得推广到正式主力环境
8. 为 `103` 建立“快照前置、验证后备份”的固定节奏，并把长期备份迁到独立于运行盘的存储上

## Open Questions

- `Bitwarden` 服务端应单独部署在独立 VM、容器，还是未来专门的 secrets 节点
- 可复用 Git 仓库最终是只留在 `103`，还是同步到外部远端以便跨机器恢复
- `238G` 盘最终是直接作为本地备份目标，还是进一步承载 PBS datastore
