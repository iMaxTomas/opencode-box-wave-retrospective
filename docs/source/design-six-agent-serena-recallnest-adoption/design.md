## Context

The current six-agent topology already has accepted role, transport, and bounded overlay contracts:

- host `Codex` is the sole control plane
- `VM103` `Codex` is the sole remote execution owner
- `NFS` is the machine-facing default route
- `SMB` is a bounded human-facing export side port
- the current `MiniMax M2.7` path is bounded and lane-specific

What is still missing is a formal contract for semantic context and continuity:

- `Serena` is now onboarded for this workspace and already has project memories, but the current root config only declares `vue`
- `RecallNest` already acts as the shared continuity layer, but six-agent access is still implicit rather than role-tiered
- cheap lanes and reviewer lanes should not automatically receive the same semantic and memory access as host or `VM103`

This makes the next design problem cross-cutting:

- how each agent should consume `Serena`
- how each agent should consume `RecallNest`
- how the two systems should meet through bounded packets instead of open-ended shared context

## Goals / Non-Goals

**Goals:**

- Define one role-tiered adoption contract for `Serena` and `RecallNest` across the current six-agent chain.
- Define one `Serena` project strategy for this mixed workspace so planning and subproject work stop depending on one under-scoped root semantic project.
- Define bounded packet shapes for passing `Serena` structure context and `RecallNest` continuity context across runtimes and lanes.
- Update readiness and rule-config reviews so future serious-task verdicts explicitly say which semantic/memory access mode each agent is allowed to use.
- Keep the plan compatible with the current host / `VM103` topology and current transport rules.

**Non-Goals:**

- Expanding `.serena/project.yml` in this change.
- Creating all subproject `.serena` configs in this change.
- Giving every lane direct full access to both `Serena` and `RecallNest`.
- Replacing `RecallNest` with `Serena`, or replacing `Serena` with `RecallNest`.
- Reopening current lane identity, transport ownership, or overlay scope.

## Decisions

### Decision: `Serena` and `RecallNest` stay separate by function

`Serena` is the semantic project layer:

- onboarding
- project memories
- symbol-aware navigation
- local structure and file-shape understanding

`RecallNest` is the continuity layer:

- resume context
- checkpoint
- cases and patterns
- cross-window and cross-runtime memory reuse

Why this split:

- `Serena` is strongest when it stays project-local and semantically scoped
- `RecallNest` is strongest when it stays shared and continuity-oriented
- merging them conceptually would blur code semantics, memory, and evidence

Alternative considered:

- treat `Serena` as both semantic and continuity layer
  - rejected because that would overload project-local onboarding with cross-runtime memory duties
- treat `RecallNest` as both memory and semantic navigation layer
  - rejected because it does not replace symbol/project structure tools

### Decision: Six-agent adoption SHALL use permission tiers rather than equal access

The six agents will not all receive equal `Serena` and `RecallNest` access.

The allowed tiers are:

- `full`
  - may directly use the layer as a first-class tool
- `scoped-full`
  - may directly use the layer, but only inside a bounded runtime or project scope
- `brief-only`
  - may consume host-owned or commander-owned summaries, not the full layer
- `packet-only`
  - may only consume bounded packets derived from the layer

Planned mapping:

- host `Codex`
  - `Serena = full`
  - `RecallNest = full`
- `VM103` `Codex`
  - `Serena = full`
  - `RecallNest = scoped-full`
- gateway `gpt-5.4`
  - `Serena = read-first/scoped`
  - `RecallNest = brief-only`
- gateway `gpt-5.3-codex`
  - `Serena = read-only/scoped`
  - `RecallNest = brief-only`
- gateway `Claude` reviewer
  - `Serena = VM103-scoped review-only`
  - `RecallNest = brief-only`
- `MiniMax M2.7`
  - `Serena = packet-only`
  - `RecallNest = packet-only`

Why this split:

- avoids over-granting cheap lanes
- keeps interpretation and continuity ownership with host / `VM103`
- still allows every current agent to participate in the trial matrix

Alternative considered:

- full `Serena` + full `RecallNest` for every lane
  - rejected because it would widen drift and blur ownership

### Decision: `Serena` project strategy SHALL split host planning from major subprojects

The current root `document-test` `Serena` project is useful for planning and OpenSpec, but it is too narrow to act as the single semantic project for this whole mixed workspace.

The target strategy is two-layered:

- host planning `Serena` project
  - OpenSpec
  - host scripts
  - markdown / yaml / bash / typescript planning surfaces
- subproject `Serena` projects
  - `recallnest/`
  - `paperclip/`
  - `openclaw/`
  - `openclaw-a2a-gateway/`
  - `vibe-kanban/`
  - `serena/`

Why this split:

- one mixed root project will stay noisy and under-modeled
- different subprojects have different languages and validation paths
- host planning still needs its own semantic surface

Alternative considered:

- expand only the one root `document-test` `Serena` project
  - rejected because that still leaves too much semantic breadth under one config

### Decision: Cross-runtime semantic and continuity exchange SHALL use bounded packets

The contract will define two packet types:

- `Serena` anchor bundle
  - structure-oriented
  - project-local
  - intended to answer “what files/symbols/areas matter”
- `RecallNest` continuity brief
  - memory-oriented
  - shared continuity
  - intended to answer “what background/decision/open loop matters”

The host or `VM103` commander remains the packet producer and acceptance owner.

Why this model:

- external lanes should not need direct unrestricted access to both tools
- packets keep semantic context and continuity context distinct
- packet use fits the already-accepted artifact-access routing contract

Alternative considered:

- let every lane directly call both systems
  - rejected because it would bypass the bounded artifact model

## Risks / Trade-offs

- [Risk] `Serena` remains underused if the project split never gets implemented.
  - Mitigation: keep the design explicit about host-planning vs subproject rollout order.
- [Risk] cheap lanes may lose useful context if brief-only mode is too thin.
  - Mitigation: define packet contracts carefully and allow later widening based on executed evidence.
- [Risk] too many `Serena` projects could create operator friction.
  - Mitigation: keep host planning project as the front door and switch to subproject `Serena` only when the bounded task is already frozen.
- [Risk] `RecallNest` could be overused as a catch-all context dump.
  - Mitigation: keep continuity briefs short and distinct from `Serena` anchor bundles.

## Migration Plan

1. Accept the planning contract for six-agent `Serena` / `RecallNest` adoption.
2. Record the approved access tier for each current agent.
3. Record the target host-planning and subproject `Serena` project layout.
4. Record the bounded packet contract for semantic and continuity exchange.
5. Use that contract to drive a later execution change:
   - first expand `Serena` project coverage
   - then run one bounded six-agent trial
   - then write a host-owned adoption verdict

## Open Questions

- Whether the host planning `Serena` project should remain one root project or be split further into planning vs scripts.
- Which subproject should receive the first full `Serena` rollout after host planning:
  - `openclaw/`
  - `recallnest/`
  - `paperclip/`
- Whether `gpt-5.4` should eventually move from scoped read-first to stronger direct `Serena` access.
