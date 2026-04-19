# VM103 RecallNest Heavy Repair Strategy 2026-04-20

## Purpose

This note is the heavy-task strategy card for the next `RecallNest` repair and
stabilization cycle.

Use it like this:

`I open this card -> I see the design intent -> I see the strongest rebuttals -> I see the current factual state -> I choose one bounded repair wave instead of reopening everything at once`

This card is not:

- a permission slip to do a broad redesign in one window
- a permission slip to mix host config governance into this repair line
- a claim that `RecallNest` is fully clean just because it is currently usable

## Current Position

Current status is mixed:

- `RecallNest` on `VM103` is usable now
- but the deployment baseline is not clean
- and the source-of-truth story is split

So the real problem is no longer:

- `can RecallNest start at all?`

The real problem now is:

- `can I turn the current usable-but-drifting setup into a clean, bounded, repeatable baseline?`

## Strategy Card

- Active plan: `stage-pve-vm-agent-lab`
- Current phase: `RecallNest` heavy repair preparation
- Window objective:
  - build one evidence-backed repair plan before touching the deployment again
- Current wave:
  - combine design intent, rebuttal/risk, and real-state evidence into one
    bounded plan
- Not in this wave:
  - no broad redeploy
  - no host config governance
  - no lane redesign
  - no full ingest campaign
  - no upstream sync yet
- Output artifact:
  - one heavy repair strategy card
  - one next-wave menu with go/no-go gates

## Evidence Chain

This card is built from three evidence lines:

### 1. Design intent

From earlier OpenSpec:

- [deploy-recallnest-shared-memory-on-vm103/proposal.md](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/proposal.md:1)
- [deploy-recallnest-shared-memory-on-vm103/design.md](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/design.md:1)
- [vm103-recallnest-memory-host spec](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/specs/vm103-recallnest-memory-host/spec.md:1)
- [remote-recallnest-mcp-consumers spec](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/specs/remote-recallnest-mcp-consumers/spec.md:1)
- [recallnest-continuity-operations spec](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/specs/recallnest-continuity-operations/spec.md:1)

Intended design is:

- `VM103` is the authoritative shared-memory host
- `VM103` local tools use local MCP
- local-machine `Codex` uses SSH-remote MCP
- code, data, and operational evidence stay separate
- ops surface is bounded:
  - wrapper
  - `doctor`
  - continuity seed
  - selective ingest
  - bounded inspection
  - backup

### 2. Rebuttal / risk

From earlier OpenSpec and current repo design:

- SSH-remote MCP depends on SSH plus remote Bun stability
- secret injection can be “service up, recall weak”
- root runtime inherits root risk
- dual local integrations can silently fork memory hosts
- `RecallNest` can become a catch-all dump if briefs and scope rules drift
- current code has some design hotspots:
  - config loading is thin JSON parse without a validation layer
  - retrieval pipeline is powerful but tuning-heavy
  - checkpoint logic prefers `rich` over strictly newest
  - `doctor` depends on continuity seeds being kept current

### 3. Real-state evidence

From this session:

- [vm103-recallnest-relaunch-20260420.md](/home/imax/codex-workspaces/document-test/openspec/changes/stage-pve-vm-agent-lab/vm103-recallnest-relaunch-20260420.md:1)
- [vm103-recallnest-status-audit-20260420.md](/home/imax/codex-workspaces/document-test/openspec/changes/stage-pve-vm-agent-lab/vm103-recallnest-status-audit-20260420.md:1)

Current facts:

- two local repo copies exist
- both are behind upstream `HEAD`
- current `VM103` deployed commit matches:
  - `/home/imax/document/test/recallnest`
- `VM103` runtime health is good enough to use now:
  - wrapper smoke works
  - `doctor` passes
  - `seed:continuity` works
  - `stats` works
  - checkpoint data exists
- but the deployed repo is not clean:
  - detached `HEAD`
  - many modified tracked files
  - many untracked files
  - sync leftovers and backup leftovers
- `scope-inventory` still reports:
  - `61` unresolved `workflow-observations` anomalies
- no persistent API/UI listener is currently running on:
  - `4317`
  - `4318`

## The Real Repair Question

The next repair cycle should answer this question:

`Do I want to stabilize the current VM103 RecallNest as the continuing authoritative host, or do I want to rebuild a cleaner deployment baseline first?`

Right now the evidence says:

- authoritative-host intent is still valid
- runtime health is already strong enough
- the weak point is baseline cleanliness and source alignment

So the next repair cycle should target:

- baseline cleanliness
- source-of-truth clarity
- consumer proof

Not:

- a brand-new architecture

## Design Claim vs Rebuttal vs Current Fact

### Claim A

`VM103` should keep being the authoritative shared-memory host.

### Rebuttal A

That only makes sense if the deployed repo and ops path are clean enough to
trust.

### Current fact

The runtime is healthy enough to use, but the repo worktree is dirty and the
local source story is split.

### Decision

Do not abandon `VM103` host design.
First clean and align the baseline.

---

### Claim B

The current deployment can be treated as “already fixed”.

### Rebuttal B

Usable runtime is not the same as a clean deployment baseline.

### Current fact

`doctor` passes, but:

- repo is dirty
- local copies are behind upstream
- deployment source is ambiguous
- inventory anomalies remain

### Decision

Treat current state as:

- `runtime usable`
- `baseline not clean`

---

### Claim C

The next step should be to sync everything to upstream latest.

### Rebuttal C

That could erase or trample the only currently working deployment path if done
blindly.

### Current fact

`VM103` currently matches the standalone local copy, not the workspace copy, and
the deployed repo has heavy local drift.

### Decision

Do not start with an upstream pull.
Start with bounded source-alignment analysis and deployment hygiene first.

## Wave Menu

The next repair cycle should be split into bounded waves.

### Wave H1: Canonical source decision

Goal:

- decide which local repo copy is the current deployment source of truth

Questions:

- does `/home/imax/document/test/recallnest` become the canonical deployment
  source for this line?
- is the workspace copy only a development fork now?

Output:

- one explicit source-of-truth decision

Stop if:

- this starts turning into a broad multi-repo governance discussion

### Wave H2: VM103 repo hygiene audit

Goal:

- explain exactly what changed in `/opt/recallnest`

Questions:

- which local modifications are intentional?
- which are sync leftovers?
- which are generated artifacts that should move out of the repo root?

Output:

- one artifact-level cleanliness report
- one decision: clean in place, snapshot then clean, or redeploy from a chosen
  source

Stop if:

- this starts rewriting the deployment in the same wave

### Wave H3: Config and continuity hardening

Goal:

- check whether runtime config, retrieval config, checkpoint config, and doctor
  expectations are explicit and safe enough

Focus:

- config validation gap
- retrieval tuning assumptions
- checkpoint retention and quality preference
- anomaly handling for workflow observations

Output:

- one “safe enough / not safe enough” verdict on current config strategy

Stop if:

- this becomes a broad algorithm redesign

### Wave H4: Consumer-path proof

Goal:

- prove one real consumer path end to end against the stabilized baseline

Possible consumers:

- `VM103` local Codex via local MCP
- host `Codex` via SSH-remote MCP

Output:

- one bounded real-consumer proof

Stop if:

- this reopens every consumer at once

## Go / No-Go Gates

### Go

Continue the repair cycle when:

- one next wave still has one narrow objective
- the next wave produces one clear artifact or verdict
- runtime usability is preserved while checking cleanliness

### No-Go

Stop and re-slice when:

- one wave tries to do source alignment, repo cleanup, config redesign, and
  consumer proof all at once
- host config governance gets mixed back into this line
- “latest upstream” becomes an excuse to skip deployment hygiene evidence

## Current Recommendation

The best next wave is:

- `Wave H1: Canonical source decision`

Reason:

- right now the biggest ambiguity is not health
- it is which repo copy actually owns the next clean deployment move

Without that decision, every later repair step can drift again.

## Next Entry

If the next window continues this heavy repair line, start here:

1. compare the two local repo copies directly
2. compare each against the `VM103` deployed worktree
3. decide one canonical deployment source
4. only after that, open the hygiene audit wave
