## ADDED Requirements

### Requirement: VM103 local tools SHALL consume local RecallNest MCP
The system SHALL allow `VM103` local `Codex` and `Claude Code` runtimes to consume the `VM103` RecallNest host through a local MCP command path, so they operate against the same authoritative memory store.

#### Scenario: Codex on VM103 uses local MCP
- **WHEN** the operator opens `Codex` on `VM103`
- **THEN** the configured RecallNest MCP command SHALL run on `VM103`
- **AND** the memory operations SHALL target the authoritative `VM103` RecallNest data

#### Scenario: Claude Code on VM103 uses the same host
- **WHEN** the operator opens `Claude Code` on `VM103`
- **THEN** its RecallNest continuity integration SHALL target the same `VM103` memory host
- **AND** the result SHALL NOT create a second isolated memory store on `VM103`

### Requirement: Local-machine Codex SHALL consume VM103 RecallNest through SSH-remote MCP
The system SHALL let the local machine's `Codex` consume the same RecallNest host by launching the RecallNest MCP server on `VM103` over SSH. This path SHALL be treated as the default shared-memory route for local-machine Codex in this deployment.

#### Scenario: Local Codex uses SSH-remote MCP
- **WHEN** the operator opens `Codex` on the local machine and invokes RecallNest memory tools
- **THEN** the MCP command SHALL start the RecallNest MCP server on `VM103` over SSH
- **AND** the memory operations SHALL use the same authoritative `VM103` memory data

#### Scenario: Local setup does not create a second local memory host
- **WHEN** the local machine is configured for this shared-memory deployment
- **THEN** the local Codex setup SHALL avoid creating a separate default local RecallNest host for the same workflow
- **AND** the shared-memory path SHALL remain inspectable from the local Codex MCP configuration
