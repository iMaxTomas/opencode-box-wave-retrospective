# VM103 Login Chain Handoff 2026-04-20

## Status

This incident is closed at the issue level.

The repaired chain is:

- host -> PVE -> VM103 -> SSH shell
- `qemu-guest-agent`
- `RecallNest MCP` wrapper startup on `VM103`

## What Happened

The stronger time-order reading from this session is:

1. a long-lived interactive root SSH session overlapped with recorded OpenCode use
2. that session degraded during teardown / cleanup
3. `systemd-logind` and related userspace cleanup began timing out
4. only after that did new SSH handshakes begin failing
5. a stuck `PVE`-side `qmreboot:103` task later complicated recovery

This means:

- `ssh` was a downstream symptom
- the stuck `qmreboot` task was part of the later recovery failure
- the fault was not explained by a simple static `sshd_config` break

## Recovery Path Used

The effective bounded repair path in this session was:

1. confirm `VM103` was still `running`
2. confirm `qm guest exec` was failing and SSH shell was failing
3. identify the stuck `qmreboot:103` task holding the guest lock
4. clear that task
5. unlock `103`
6. hard-reset the guest
7. re-verify:
   - `ssh vm103-openclaw`
   - `qm guest exec 103 -- /usr/bin/true`
   - `RecallNest MCP` wrapper startup

## Canonical Note

The full diagnostic + repair note is:

- [vm103-login-chain-diagnostic-20260420.md](/home/imax/codex-workspaces/document-test/openspec/changes/stage-pve-vm-agent-lab/vm103-login-chain-diagnostic-20260420.md:1)

## Reopen Conditions

Reopen this topic only if one of these fails again:

- `ssh vm103-openclaw` no longer returns a real shell
- `ssh pve 'qm guest exec 103 -- /usr/bin/true'` fails again
- the `VM103` RecallNest wrapper path stops starting after guest recovery

## Next Entry

If this is reopened, start with:

1. `ssh vm103-openclaw 'echo VM103_SSH_OK && hostname && whoami'`
2. `ssh pve 'qm guest exec 103 -- /usr/bin/true'`
3. only if one of those fails, return to the deeper `PVE` / guest-console repair path

## Checkpoint Note

RecallNest `checkpoint_session` was attempted for this handoff, but the tool
returned `Transport closed` in this window. This file is the fallback handoff
artifact for the same close-window state.
