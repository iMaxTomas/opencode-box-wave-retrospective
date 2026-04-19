# VM103 Login Chain Diagnostic 2026-04-20

## Purpose

This note is the bounded diagnostic card for the current `VM103` login-chain
issue.

Use it like this:

`I open this note -> I see exactly which layer is already proven healthy -> I see exactly where the login chain is currently breaking -> I open only the next repair wave for that broken layer`

This note is not:

- a full `PVE` recovery program
- a permission slip to reopen broad host governance
- a permission slip to mix `opencode-box` work into the same loop

## Current Strategy Card

- Active plan: `stage-pve-vm-agent-lab`
- Current phase: `VM103` access-chain diagnosis
- Window objective: prove which layer of the `host -> PVE -> VM103 -> SSH shell`
  chain is currently broken
- Current wave: isolate the break between power state, L3 reachability, TCP 22,
  guest agent, and real SSH shell
- Execution hook: shell + `ssh pve` + direct `ssh vm103-openclaw`
- Not in this wave:
  - no host-config governance
  - no `opencode-box` execution
  - no broad guest repair
  - no credential rotation
- Output artifact: one diagnostic note with one clear next repair entry

## Commands Used In This Wave

### PVE state / config / guest-agent probe

```bash
ssh pve 'qm status 103; echo ---; qm config 103 | grep -E "^(name|agent|ipconfig|net0)" || true; echo ---; ping -c 1 -W 2 172.29.111.221 || true; echo ---; qm guest exec 103 -- /usr/bin/true'
```

### PVE TCP 22 probe

```bash
ssh pve "bash -lc 'if echo > /dev/tcp/172.29.111.221/22; then echo SSH_PORT_OPEN; else echo SSH_PORT_CLOSED; fi'"
```

### Host direct SSH shell probe

```bash
ssh vm103-openclaw -o BatchMode=yes -o ConnectTimeout=5 -o StrictHostKeyChecking=accept-new 'echo VM103_SSH_OK && hostname && whoami'
```

### PVE-to-guest SSH shell probe

```bash
ssh pve "ssh -o BatchMode=yes -o ConnectTimeout=5 root@172.29.111.221 'echo PVE_TO_VM103_SSH_OK && hostname'"
```

### PVE serial-console probe

```bash
ssh -tt pve qm terminal 103
```

Observed after one newline:

```text
debian-openclaw login:
```

## Results

### Proven Healthy

- `qm status 103` returned `status: running`
- `qm config 103` still shows:
  - `name: debian-openclaw`
  - `agent: enabled=1`
  - `ipconfig0: ip=172.29.111.221/24`
- `PVE -> 172.29.111.221` `ping` succeeded
- `PVE -> 172.29.111.221:22` returned `SSH_PORT_OPEN`
- `qm terminal 103` reached the guest login prompt

### Proven Broken

- `qm guest exec 103 -- /usr/bin/true` returned:
  - `QEMU guest agent is not running`
- direct host login probe returned:
  - `Connection timed out during banner exchange`
  - `Connection to 172.29.111.221 port 22 timed out`
- `PVE -> guest` SSH probe returned the same banner-exchange timeout

## What This Means

The break is not here:

- not `PVE` power state
- not guest IP assignment
- not basic L3 reachability
- not TCP port 22 visibility

The break is here:

- somewhere in the guest-side SSH service / pre-auth handshake / userland path
- and the missing guest agent confirms the guest userspace is not fully healthy

The current best repair entry is no longer guessed SSH.
It is the guest serial console as `root`.

This wave does **not** prove an operator credential problem.
The timeout happens before a normal usable shell is established.

## Next Repair Wave

The next bounded wave should be only this:

- inspect the guest from `PVE` console and check:
  - whether `sshd` is running cleanly
  - whether the guest is stuck before normal login service readiness
  - whether guest agent and SSH share the same userspace failure
  - whether `root` can repair `sshd` / `qemu-guest-agent` back into the normal range

Do not widen the next wave into:

- onboot policy redesign
- host config governance
- `opencode-box` work
- lane/provider/auth changes

## Continue / Stop Gate

Continue when:

- the next loop still only diagnoses or repairs the guest login chain
- one more loop can leave one clearer verdict

Stop and re-slice when:

- the next step starts mixing guest repair with unrelated host governance
- the next step needs a separate recovery change instead of one bounded repair

## Repair Wave Result

The bounded repair wave used this sequence:

```bash
ssh pve 'qm reboot 103'
ssh pve 'pvesh get /nodes/pve/tasks/UPID:pve:00132133:08A25426:69E5300F:qmreboot:103:root@pam:/status --output-format json'
ssh pve 'kill 1253683 && qm unlock 103 && qm reset 103'
```

Observed intermediate state:

- the first `qm reboot 103` created a stuck `qmreboot` task
- that task held `/run/lock/qemu-server/lock-103.conf`
- soft reboot did not restore SSH or guest agent

Bounded repair:

- kill the stuck `qmreboot` task
- `qm unlock 103`
- hard `qm reset 103`

## Post-Repair Verification

### Guest agent

```bash
ssh pve 'qm guest exec 103 -- /usr/bin/true'
```

Result:

```json
{
  "exitcode": 0,
  "exited": 1
}
```

### SSH

```bash
ssh vm103-openclaw -o BatchMode=yes -o ConnectTimeout=5 -o StrictHostKeyChecking=accept-new 'echo VM103_SSH_OK && hostname && whoami'
```

Result:

```text
VM103_SSH_OK
debian-openclaw
root
```

### RecallNest MCP wrapper

Process check after SSH recovery showed the remote RecallNest wrapper path still
exists and can start on `VM103`:

```bash
ssh vm103-openclaw 'ps -ef | grep -i recallnest | grep -v grep || true'
ssh vm103-openclaw "bash -lc 'cd /opt/recallnest && timeout 3 /opt/ai-lab/bin/recallnest-mcp >/tmp/recallnest-mcp-smoke.out 2>/tmp/recallnest-mcp-smoke.err; tail -n 40 /tmp/recallnest-mcp-smoke.err || true'"
```

Observed startup evidence:

```text
[INFO] LLM client initialized: claude-sonnet-4-6 @ https://api.wantslabs.com/v1
[MCP] Skipping workflow_observe (tier: governance)
[MCP] Skipping workflow_health (tier: governance)
...
```

Interpretation:

- the wrapper path `/opt/ai-lab/bin/recallnest-mcp` is valid
- the persisted env file is present and readable
- `bun run src/mcp-server.ts` starts far enough to initialize the LLM client and
  MCP server path
- no new RecallNest-specific repair was required after the VM103 reset

## Final Verdict

This bounded wave is complete.

The fault was no longer just inside guest SSH.
It also involved a stuck `PVE`-side `qmreboot` task holding the guest lock while
the guest stayed in a half-healthy state.

After killing the stuck reboot task, unlocking the guest, and forcing one hard
reset:

- guest userspace recovered enough for `qemu-guest-agent`
- SSH recovered to a normal shell
- the `RecallNest MCP` wrapper path on `VM103` can start again
- the `host -> PVE -> VM103 -> SSH shell` chain is healthy again

## Time-Order Recheck

This recheck was done to answer a narrower question:

- was this just a random SSH failure
- or did the failure begin while an active `OpenCode` session was still in use

### Evidence

Root `fish_history` on `VM103` recorded these `opencode` commands:

- `OPENCODE_SERVER_PORT=4096 opencode serve`
  - `2026-04-19T17:43:46+00:00`
- `opencode auth`
  - `2026-04-19T18:14:10+00:00`
- `opencode auth logout`
  - `2026-04-19T18:14:21+00:00`

The same boot also shows one long-lived SSH session:

- session 330 accepted at `2026-04-19T17:43:44+00:00`
- disconnect started at `2026-04-19T18:58:20+00:00`
- `pam_unix` close at `2026-04-19T18:58:50+00:00`
- `pam_systemd(sshd:session): Failed to release session: Transport endpoint is not connected`
  at `2026-04-19T18:59:25+00:00`
- `dbus-daemon`: `Connection has not authenticated soon enough`
  at `2026-04-19T18:59:28+00:00`
- `systemd-logind`: `Failed to abandon session scope, ignoring: Connection timed out`
  at `2026-04-19T19:00:30+00:00`
- multiple new inbound SSH attempts hit `Broken pipe [preauth]`
  at `2026-04-19T19:03:29+00:00`
- `systemd-journald.service: Watchdog timeout`
  at `2026-04-19T19:05:14+00:00`

### Interpretation

The strongest current reading is:

- a long-lived interactive root SSH session was active during `OpenCode` use
- that session disconnected first
- the failure began during session teardown / logind / dbus cleanup
- only after that did new SSH handshakes begin failing
- journald watchdog timeout appeared later, as part of the same degrading userspace state

This means the failure sequence is more precise than:

- "SSH randomly died"

and also more precise than:

- "`qmreboot` alone caused everything"

Current best explanation:

- the system first got stuck while cleaning up a long-lived interactive session
  that overlaps with recorded `OpenCode` use
- the resulting userspace degradation then broke new SSH handshakes and guest
  agent health
- the stuck `qmreboot` task was part of the later recovery failure, not the
  earliest trigger currently visible in the logs

## Next Entry

If a future session reopens this topic, start from:

- verify `ssh vm103-openclaw` still returns a real shell
- verify `qm guest exec 103 -- /usr/bin/true` still works

Do not reopen the deeper repair path unless one of those two checks fails again.

## Archive Decision

This incident is closed and may be treated as archived at the issue level.

Why:

- the `host -> PVE -> VM103 -> SSH shell` chain is healthy again
- `qemu-guest-agent` is healthy again
- the `RecallNest MCP` wrapper path on `VM103` can start again
- the fault sequence and the recovery path are both recorded in this note

Reopen only if one of these happens again:

- `ssh vm103-openclaw` stops returning a real shell
- `qm guest exec 103 -- /usr/bin/true` fails again
- the remote `RecallNest` wrapper path stops starting after guest recovery
