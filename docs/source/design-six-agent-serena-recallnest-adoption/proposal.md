## Why

The current six-agent chain already has accepted topology, transport, and bounded overlay evidence, but it does not yet have a formal contract for how `Serena` and `RecallNest` should be used together across those six agents. Right now the two layers are useful but uneven: `Serena` is only lightly onboarded on the host and is still underconfigured for this mixed workspace, while `RecallNest` is already acting as the continuity layer without a role-tiered adoption plan for every lane.

## What Changes

- Define one role-tiered adoption contract for how all six current agents may use `Serena`, `RecallNest`, or only bounded packets derived from them.
- Define one `Serena` project strategy for this mixed workspace so the host planning surface and major subprojects stop sharing one under-scoped `vue`-only semantic project.
- Define one bounded packet model that separates:
  - `Serena` anchor bundles for project-structure and code-shape context
  - `RecallNest` continuity briefs for resumable cross-window memory
  - host-owned acceptance of what may be shared onward
- Update readiness and rule-config evaluation so future serious-task verdicts explicitly say whether each agent has:
  - full `Serena`
  - scoped `Serena`
  - `RecallNest` full access
  - `RecallNest` brief-only access
  - packet-only access
- Keep this change planning-only. It does not yet expand `.serena/project.yml`, create all subproject configs, or widen any lane permissions.

## Capabilities

### New Capabilities
- `serena-recallnest-six-agent-adoption`: defines how the six-agent topology uses `Serena` and `RecallNest` by role, scope, and permission level.
- `serena-project-strategy`: defines how this mixed workspace should split host-planning and subproject `Serena` projects instead of relying on one under-scoped root project.
- `bounded-serena-recallnest-context-packets`: defines the allowed packet shapes for `Serena` anchor bundles and `RecallNest` continuity briefs across runtimes and lanes.

### Modified Capabilities
- `six-agent-end-to-end-readiness`: readiness verdicts must include the approved `Serena` and `RecallNest` access mode for each agent, not just lane role.
- `multi-runtime-rule-config-readiness`: rule/config readiness must include `Serena` project coverage and `RecallNest` access strategy for serious secondary-development work.

## Impact

- Affected systems:
  - host `Serena` onboarding and project-scope strategy
  - `VM103` remote execution planning and bounded semantic access
  - `RecallNest` shared continuity usage across host and `VM103`
  - six-agent readiness evaluation
  - future multiagent dispatch/review packet design
- Affected artifacts:
  - `.serena/project.yml` strategy and future subproject `.serena` configs
  - `RecallNest` scope and permission policy
  - OpenSpec readiness and packet contracts
- This change does not yet modify runtime lane identity, transport ownership, or `OpenClaw` overlay scope.
