# VM103 RecallNest Status Audit 2026-04-20

## Purpose

This note is the bounded strategy card for the current `RecallNest` status
audit.

Use it like this:

`I open this card -> I see which local repo copy is current -> I see whether it is on upstream HEAD -> I see whether VM103 RecallNest is actually healthy`

This card is not:

- a broad RecallNest redesign
- a permission slip to reopen failed host config governance
- a permission slip to mix `opencode-box` work into the same loop

## Current Strategy Card

- Active plan: `stage-pve-vm-agent-lab`
- Current phase: `RecallNest` status audit after relaunch verification
- Window objective:
  - identify the current local repo copy
  - check whether local copies are on upstream latest state
  - check the real health of `VM103` RecallNest
- Current wave:
  - local repo identity
  - upstream HEAD comparison
  - `VM103` runtime + repo health proof
- Not in this wave:
  - no broad redeploy
  - no host config governance
  - no large ingest campaign
  - no topology redesign
- Output artifact: one current-state verdict with one next-entry

## Local Repo Copies Found

Two local copies exist:

1. workspace copy
   - `/home/imax/codex-workspaces/document-test/recallnest`
2. standalone local copy
   - `/home/imax/document/test/recallnest`

## Local / Upstream State

### Workspace copy

- path:
  - `/home/imax/codex-workspaces/document-test/recallnest`
- branch:
  - `main`
- local HEAD:
  - `b74bb719e895e18db313ddd3bc6b73961b8e2b69`
- origin HEAD:
  - `194cd656d9ab1ad6ef41c993d1711f013e8fd45e`
- local status:
  - `## main...origin/main`
  - untracked `.serena/`

Interpretation:

- this copy is not at the current upstream HEAD
- it also does not match the current `VM103` deployment copy

### Standalone local copy

- path:
  - `/home/imax/document/test/recallnest`
- branch:
  - `main`
- local HEAD:
  - `82c4f5a85935375bc2336127c7d4a76decb58bb6`
- origin HEAD:
  - `194cd656d9ab1ad6ef41c993d1711f013e8fd45e`
- local status:
  - `## main...origin/main`

Interpretation:

- this copy is also behind upstream HEAD
- but this is the copy that matches the current `VM103` deployment commit

## Serena Activation

`Serena` was activated and onboarding was confirmed on:

- `/home/imax/codex-workspaces/document-test/recallnest`

Project memory confirms the repo is the `RecallNest` TypeScript/Bun shared
memory layer with MCP, API, UI, doctor, seed, and checkpoint primitives.

## VM103 Repo State

Current deployed repo root:

- `/opt/recallnest`

Observed state:

- branch:
  - `HEAD (no branch)`
- deployed HEAD:
  - `82c4f5a85935375bc2336127c7d4a76decb58bb6`
- remote:
  - `https://github.com/AliceLJY/recallnest.git`
- worktree state:
  - many modified tracked files
  - many untracked files
  - local `data/`
  - backup/sync leftovers such as `src.bak.*` and `src.pre-sync.*`

Interpretation:

- `VM103` is not running a clean checkout
- it is on the same commit family as `/home/imax/document/test/recallnest`
- but it has significant in-place drift

## VM103 Runtime Health

### Proven Healthy

- `bun` present on `VM103`
  - `1.3.11`
- wrapper exists:
  - `/opt/ai-lab/bin/recallnest-mcp`
- env exists:
  - `/opt/ai-lab/config/recallnest.env`
- wrapper smoke reaches MCP init and LLM client startup
- `bun run doctor` passes core health checks
- `bun run stats` returns live memory stats
- `bun run scope-inventory` runs
- session checkpoint store exists:
  - `344` checkpoint files
- memory data exists:
  - `59M` under `/srv/ai-memory/recallnest`

### Proven Imperfect

- `scope-inventory` still reports:
  - `61` unresolved anomalies in `workflow-observations`
- no long-running API/UI listener was observed on:
  - `4317`
  - `4318`

Interpretation:

- the on-demand MCP path is healthy
- the data path is healthy
- the continuity baseline is healthy
- but this is not currently a clean, service-managed deployment with a clean repo

## Current Verdict

The current task-state verdict is:

- local repo situation: `split`
- upstream freshness: `behind`
- VM103 runtime health: `usable`
- VM103 deployment cleanliness: `not clean`

More precisely:

- the current operationally-relevant local copy is
  `/home/imax/document/test/recallnest`, because it matches the `VM103` commit
- neither local copy is at upstream latest HEAD
- `VM103` RecallNest is healthy enough to use now
- but the deployed repo on `VM103` has heavy local drift and should not be
  treated as a clean baseline

## Next Entry

If the next window continues this line, choose exactly one follow-up:

1. repo alignment wave
   - decide whether `/home/imax/document/test/recallnest` becomes the canonical
     local deployment source
   - then plan a clean bounded sync to `VM103`
2. VM103 hygiene wave
   - clean or snapshot the dirty `/opt/recallnest` worktree without changing the
     memory data path
3. consumer verification wave
   - keep the current deployment as-is
   - verify one concrete consumer path end to end against the current healthy MCP
     wrapper

## Continue / Stop Gate

Continue when:

- the next loop still focuses on one bounded follow-up from above
- one more loop can leave one clearer verdict

Stop and re-slice when:

- the next step mixes repo alignment with broad topology redesign
- the next step tries to fold host config governance into this status audit
