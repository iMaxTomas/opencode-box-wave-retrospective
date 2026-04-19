## 1. Baseline And Access

- [x] 1.1 Record the current `PVE` host, `VM 103`, snapshot, IP, and SSH entry facts in the change artifacts
- [x] 1.2 Define the snapshot naming scheme and rollback notes for each future phase
- [x] 1.3 Decide whether the guest keeps `root` as the daily operator user or adds `imax + sudo`

## 2. Guest Workspace And Base Environment

- [x] 2.1 Confirm and document the stable guest directory layout under `/opt/ai-lab` and `/srv/ai-artifacts`
- [x] 2.2 Bootstrap the base guest environment, including locale cleanup, core packages, shell tools, Node, Python, and `uv`
- [x] 2.3 Capture baseline health checks and write them under `/opt/ai-lab/checks` and `/opt/ai-lab/docs`

## 3. Credential And Safety Model

- [x] 3.1 Implement `Bitwarden self-host` on a separate instance as the primary secrets path contract for `Codex`, `Claude Code`, `Gemini`, and `OpenClaw`, with `103` acting only as a consumer and without embedding live secrets in reusable scripts
- [x] 3.2 Define the lane wrapper layout, which files hold templates, which files are ignored, where runtime state lives, how secrets are rendered or loaded, and where the operator handoff pauses for live credential input
- [x] 3.3 Define the point at which a new snapshot is required before high-risk changes
- [x] 3.4 Provision a custom CA and SAN-valid server certificate on `VM 104` for Bitwarden HTTPS
- [x] 3.5 Trust the Bitwarden CA on `VM 103` and validate `bw` CLI server reachability before live vault use
- [x] 3.6 Update the operator handoff so the next-step order is: Web login -> API item entry -> lane-specific OAuth
- [x] 3.7 Define the official `Codex` subagent baseline for `103`, including explicit spawn rules and built-in role usage
- [x] 3.8 Define when shared-directory work stays serial and when Git worktree isolation is required before parallel writes

## 4. Tool Onboarding

- [x] 4.1 Install and validate `Codex` with a minimal real task inside the guest main thread
- [x] 4.2 Validate one official `Codex` subagent read-heavy side task and record the evidence and handoff path
- [x] 4.3 Install and validate `Claude Code` with a minimal real task inside the guest
- [ ] 4.4 If `Gemini` returns to scope, install and validate it with a minimal real task inside the guest
- [x] 4.5 Install and validate `OpenClaw` with a minimal startup and usability check inside the guest

## 5. Evidence And Iteration

- [x] 5.1 Write bootstrap scripts, check scripts, and operator notes into the stable guest directory layout
- [x] 5.2 Record logs, failures, workarounds, and residual risks for each phase
- [x] 5.3 Expand `103` storage to `80G`, verify guest visibility, and record whether further growth is needed after the first full pass

## 6. Promotion And Follow-up Planning

- [x] 6.1 Review which parts of the lab setup are safe to promote to a main environment and which remain lab-only
- [x] 6.2 Identify which remaining work belongs in this change versus a narrower follow-up change
- [x] 6.3 Resolve the open questions around user model, secret handling, OpenClaw scope, and post-validation cloning strategy
- [x] 6.4 Review `PVE` thin-pool headroom and define a snapshot retention rule before broader tool onboarding
- [x] 6.5 Define the validated-state backup cadence for `103`, including when backups run relative to pre-* snapshots and which storage target receives them
