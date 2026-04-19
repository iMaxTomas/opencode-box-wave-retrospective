## 1. VM103 Host Preparation

- [x] 1.1 Define and create the stable `RecallNest` code, data, and log paths on `VM103`
- [x] 1.2 Install or verify the required runtime dependencies on `VM103`, including Bun and any system prerequisites needed by RecallNest
- [x] 1.3 Sync the local RecallNest repository to `VM103` in a way that preserves a clear deployment root

## 2. Host Configuration And Health

- [x] 2.1 Render the non-secret RecallNest config on `VM103` so it points at the chosen data location
- [x] 2.2 Connect the required live secret path for RecallNest on `VM103` without storing live keys in the repository
- [x] 2.3 Run the first health checks on `VM103`, including doctor and a minimal API or MCP smoke proof

## 3. VM103 Local Consumers

- [x] 3.1 Configure `Codex` on `VM103` to consume the local RecallNest MCP server
- [x] 3.2 Configure `Claude Code` on `VM103` to consume the same local RecallNest MCP server and continuity rules
- [x] 3.3 Verify that both `VM103` local consumers resolve to the same authoritative RecallNest memory host

## 4. Local-Machine Shared Consumer

- [x] 4.1 Define the SSH-remote MCP command for the local machine's `Codex` so it runs the RecallNest MCP server on `VM103`
- [x] 4.2 Update the local machine's `Codex` configuration to use the SSH-remote RecallNest MCP path instead of a local isolated host
- [x] 4.3 Verify that the local machine's `Codex` can call the remote RecallNest MCP path successfully

## 5. Continuity Proof

- [x] 5.1 Seed the continuity baseline on `VM103`
- [x] 5.2 Run a bounded first ingest plan for the current lab workflow instead of ingesting all history blindly
- [x] 5.3 Prove cross-machine continuity by saving context from one configured consumer and recovering it from another configured consumer

## 6. Operations And Recovery

- [x] 6.1 Define the bounded inspection path for the RecallNest API and UI, including the local-machine access method
- [x] 6.2 Record the minimum operator runbook for start, stop, doctor, ingest, and continuity verification
- [x] 6.3 Define how RecallNest state on `VM103` is covered by the existing snapshot and backup rhythm without confusing it with artifact or trace storage
