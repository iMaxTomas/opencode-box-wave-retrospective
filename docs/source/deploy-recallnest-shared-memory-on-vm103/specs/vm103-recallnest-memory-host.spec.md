## ADDED Requirements

### Requirement: VM103 SHALL host the authoritative RecallNest runtime and data
The system SHALL deploy RecallNest on `VM103` as the authoritative shared memory host for the current lab. The deployment SHALL separate code, data, and operational evidence so the runtime can be backed up and inspected without conflating repository files and memory state.

#### Scenario: Code and data are stored separately
- **WHEN** RecallNest is deployed on `VM103`
- **THEN** the deployment SHALL place code and runtime entrypoints in a stable code location
- **AND** the deployment SHALL place memory data in a separate durable data location
- **AND** the operator SHALL be able to identify where logs and evidence are saved

#### Scenario: Health checks are available after deployment
- **WHEN** the RecallNest runtime is installed on `VM103`
- **THEN** the operator SHALL be able to run a health check such as doctor or API smoke validation
- **AND** the result SHALL be recordable as deployment evidence

### Requirement: RecallNest live secrets SHALL remain outside the repository
The system SHALL keep live RecallNest secrets outside the repository and outside reusable committed templates. Live keys required for embedding or related runtime behavior SHALL be injected through the existing secrets path instead of being stored in tracked files.

#### Scenario: Repository files remain non-secret
- **WHEN** the operator inspects the deployed RecallNest repository on `VM103`
- **THEN** tracked config examples MAY exist
- **AND** live secrets SHALL NOT appear in committed repository files

#### Scenario: Runtime receives the needed live secret
- **WHEN** the operator starts or validates the RecallNest runtime
- **THEN** the runtime SHALL receive the required live secret through the deployment secret path
- **AND** the health check SHALL reflect whether the secret path is valid
