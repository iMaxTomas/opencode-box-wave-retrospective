## ADDED Requirements

### Requirement: The deployment SHALL define a minimal RecallNest operations surface
The system SHALL define the minimum operational actions required to keep RecallNest usable in the lab. This operations surface SHALL include continuity seeding, health checks, history ingest guidance, and inspection access for the API or UI without expanding RecallNest into a general network service.

#### Scenario: Continuity baseline can be seeded and verified
- **WHEN** RecallNest is first deployed or reset on `VM103`
- **THEN** the operator SHALL be able to seed the continuity baseline
- **AND** the operator SHALL be able to verify that the continuity path is ready for fresh windows

#### Scenario: Existing history can be selectively ingested
- **WHEN** the operator wants RecallNest to know about prior terminal or project history
- **THEN** the deployment SHALL define an ingest path
- **AND** the operator SHALL be able to choose a bounded initial ingest scope instead of being forced to ingest all history

### Requirement: API and UI inspection SHALL use a bounded exposure path
The system SHALL keep the RecallNest API and UI on a bounded operational exposure path. Direct lab-network exposure SHALL NOT be the default. Inspection from the local machine SHALL work through the chosen bounded path.

#### Scenario: API inspection stays bounded
- **WHEN** the operator needs to inspect the RecallNest API from outside `VM103`
- **THEN** the access path SHALL use the bounded exposure method defined by the deployment
- **AND** the API SHALL NOT require the deployment to become a general LAN-facing service by default

#### Scenario: Shared memory can be proven across machines
- **WHEN** the operator stores or checkpoints context through one configured consumer and later resumes from another configured consumer
- **THEN** the second consumer SHALL be able to recover the same shared-memory context from the authoritative `VM103` host
- **AND** the deployment SHALL save evidence of that cross-machine continuity proof
