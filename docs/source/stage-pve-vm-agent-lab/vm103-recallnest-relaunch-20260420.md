# VM103 RecallNest Relaunch 2026-04-20

## Purpose

This note is the bounded strategy card for relaunching and re-verifying
`RecallNest` on `VM103` after the `VM103` login-chain repair.

Use it like this:

`I open this card -> I see which old OpenSpec plan is being reused -> I run one bounded relaunch wave -> I write back one verdict instead of reopening broad governance`

This card is not:

- a new broad RecallNest architecture proposal
- a permission slip to reopen failed host config governance
- a permission slip to mix `opencode-box` execution into the same loop

## Reused Plan Sources

This relaunch wave reuses these earlier planning sources:

- archived deployment baseline:
  - [deploy-recallnest-shared-memory-on-vm103/proposal.md](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/proposal.md:1)
  - [deploy-recallnest-shared-memory-on-vm103/design.md](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/design.md:1)
  - [vm103-recallnest-memory-host spec](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/specs/vm103-recallnest-memory-host/spec.md:1)
  - [remote-recallnest-mcp-consumers spec](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/specs/remote-recallnest-mcp-consumers/spec.md:1)
  - [recallnest-continuity-operations spec](/home/imax/codex-workspaces/document-test/openspec/changes/archive/2026-04-19-deploy-recallnest-shared-memory-on-vm103/specs/recallnest-continuity-operations/spec.md:1)
- current incident handoff:
  - [vm103-login-chain-diagnostic-20260420.md](/home/imax/codex-workspaces/document-test/openspec/changes/stage-pve-vm-agent-lab/vm103-login-chain-diagnostic-20260420.md:1)
  - [vm103-login-chain-handoff-20260420.md](/home/imax/codex-workspaces/document-test/openspec/changes/stage-pve-vm-agent-lab/vm103-login-chain-handoff-20260420.md:1)

## Current Strategy Card

- Active plan: `stage-pve-vm-agent-lab`
- Current phase: `VM103` RecallNest bounded relaunch
- Window objective: re-prove that `VM103` RecallNest is healthy enough to act as
  the current shared-memory host
- Current wave: reuse the old deployment plan's minimum ops surface only
  (`mcp wrapper -> doctor -> seed:continuity -> checkpoint path proof`)
- Execution hook:
  - `ssh vm103-openclaw`
  - `/opt/ai-lab/bin/recallnest-mcp`
  - `bun run doctor`
  - `bun run seed:continuity`
- Not in this wave:
  - no RecallNest redesign
  - no host Codex config governance
  - no broad ingest program
  - no new secrets source
  - no `opencode-box`
- Output artifact: one relaunch note with one clear verdict and one next entry

## Wave 1 Commands

### 1. Wrapper and env proof

```bash
ssh vm103-openclaw "sed -n '1,220p' /opt/ai-lab/bin/recallnest-mcp"
ssh vm103-openclaw "bash -lc 'ls -l /opt/ai-lab/bin/recallnest-mcp /opt/ai-lab/config/recallnest.env'"
ssh vm103-openclaw "bash -lc 'cd /opt/recallnest && timeout 5 /opt/ai-lab/bin/recallnest-mcp >/tmp/recallnest-mcp-smoke.out 2>/tmp/recallnest-mcp-smoke.err; tail -n 60 /tmp/recallnest-mcp-smoke.err || true'"
```

### 2. Minimum health proof

```bash
ssh vm103-openclaw "bash -lc 'set -a; source /opt/ai-lab/config/recallnest.env; set +a; cd /opt/recallnest && timeout 20 bun run doctor'"
```

### 3. Continuity baseline proof

```bash
ssh vm103-openclaw "bash -lc 'set -a; source /opt/ai-lab/config/recallnest.env; set +a; cd /opt/recallnest && timeout 30 bun run seed:continuity'"
```

### 4. Data and checkpoint proof

```bash
ssh vm103-openclaw "bash -lc 'set -a; source /opt/ai-lab/config/recallnest.env; set +a; cd /opt/recallnest && timeout 20 bun run stats && timeout 20 bun run scope-inventory | sed -n \"1,120p\"'"
ssh vm103-openclaw "python3 -c \"import glob,json; p=sorted(glob.glob('/opt/recallnest/data/session-checkpoints/*.json'))[-1]; r=json.load(open(p,'r',encoding='utf-8')); print(p); print(r.get('checkpointId')); print(r.get('resolvedScope') or r.get('scope')); print(r.get('updatedAt'))\""
```

## Wave 1 Results

### Proven Healthy

- `/opt/ai-lab/bin/recallnest-mcp` exists and sources
  `/opt/ai-lab/config/recallnest.env`
- wrapper smoke reached MCP initialization and LLM client startup
- `bun run doctor` passed the core health checks:
  - Bun runtime present
  - config valid
  - `JINA_API_KEY` present
  - data directory present
  - embedding API configured
  - continuity baseline present
- `bun run seed:continuity` completed:
  - patterns existing and skipped
  - cases existing and skipped
  - memory seeds stored successfully
- `bun run stats` returned live index stats
- latest remote checkpoint file exists under:
  - `/opt/recallnest/data/session-checkpoints/`

### Still Imperfect But Not Blocking This Wave

- `scope-inventory` still reports `61` unresolved anomalies in
  `workflow-observations`
- this is a cleanliness problem, not a relaunch blocker

## Local Helper Repair Found In This Wave

This wave also exposed and repaired one local helper script issue:

- [scripts/recallnest/fetch_vm103_latest_checkpoint.sh](/home/imax/codex-workspaces/document-test/scripts/recallnest/fetch_vm103_latest_checkpoint.sh:1)

The repair was:

- stop using a remote heredoc that depends on the remote default shell
- pass the Python program over SSH stdin instead
- fix the local `python3` argument order when extracting `checkpointId` and
  `updatedAt`

Current boundary:

- the script logic and syntax are now corrected
- direct remote checkpoint proof succeeded in this wave
- the current tool wrapper still produced noisy `rtk` errors when invoking the
  whole helper script end-to-end, so the canonical proof for this wave remains
  the direct remote checkpoint read above

## Verdict

The relaunch verdict for this bounded wave is:

- `relaunch-ready`

More precisely:

- `RecallNest` on `VM103` did not need a broad redeploy
- it needed a bounded post-login-chain re-verification
- that re-verification now proves the wrapper, env path, doctor path,
  continuity baseline, stats path, and remote checkpoint store are all back in
  the normal range

## Next Entry

If the next window needs to continue this line, start here:

1. rerun the wrapper smoke
2. rerun `bun run doctor`
3. only if doctor fails, reopen a deeper `RecallNest` repair wave
4. if doctor passes, treat `VM103` RecallNest as the current shared-memory host
   and move on to the next bounded consumer-specific task

## Continue / Stop Gate

Continue when:

- the next loop is still only about `RecallNest` health or one consumer path
- one more loop can leave one clearer verdict

Stop and re-slice when:

- the next step starts reopening host config governance
- the next step starts redesigning RecallNest topology
- the next step becomes a broad ingest or cleanup campaign instead of one bounded relaunch follow-up
