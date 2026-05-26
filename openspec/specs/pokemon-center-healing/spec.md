## ADDED Requirements

### Requirement: Trainer can heal the active main party at the Pokemon Center
The system SHALL allow an authenticated trainer to execute a Pokemon Center healing operation that restores the entire active main party in one atomic action.

#### Scenario: Healing restores the full active party
- **WHEN** an authenticated trainer executes the Pokemon Center healing action with a valid active main party and no active battle session
- **THEN** the system MUST restore every party member to full persisted HP and full persisted move PP in the same successful operation

#### Scenario: Healing requires an active party
- **WHEN** an authenticated trainer executes the Pokemon Center healing action without a valid active main party
- **THEN** the system MUST reject the request and MUST NOT create healing records

#### Scenario: Healing is blocked during an active battle
- **WHEN** an authenticated trainer executes the Pokemon Center healing action while a battle session is active
- **THEN** the system MUST reject the request with a normalized business-rule error and MUST NOT mutate HP, PP, or healing history

### Requirement: Pokemon Center healing revives fainted party members
The system SHALL include revive behavior as part of the canonical Pokemon Center healing flow.

#### Scenario: Fainted Pokemon are revived during healing
- **WHEN** one or more active party members have `current_hp = 0` and the trainer completes a valid healing operation
- **THEN** the system MUST revive those party members and MUST restore their persisted HP to full

#### Scenario: Revive does not require a separate endpoint
- **WHEN** the system supports Pokemon Center healing
- **THEN** the canonical healing action MUST include revive behavior instead of requiring a parallel revive-only flow

### Requirement: Pokemon Center healing records a summarized operation and itemized history
The system SHALL persist both an operation summary and itemized healing history for each successful healing event.

#### Scenario: Successful healing creates an operation summary
- **WHEN** a Pokemon Center healing operation succeeds
- **THEN** the system MUST persist one `pokemon_center_healing` record containing the trainer context and aggregate restored totals for that operation

#### Scenario: Successful healing creates itemized Pokemon history
- **WHEN** a Pokemon Center healing operation succeeds
- **THEN** the system MUST persist one `healing_log` record per restored party member linked to the summarized healing operation

#### Scenario: History can be listed item by item
- **WHEN** an authenticated trainer requests Pokemon Center healing history
- **THEN** the system MUST return a chronological itemized history keyed by restored Pokemon entries rather than only grouped operation summaries

### Requirement: Pokemon Center healing returns normalized payloads and invalidates trainer caches
The system SHALL expose normalized healing responses and invalidate affected trainer caches after successful healing.

#### Scenario: Successful healing returns normalized restored data
- **WHEN** a Pokemon Center healing operation succeeds
- **THEN** the API MUST return a normalized payload containing the operation summary and the restored Pokemon entries without requiring frontend-side healing calculations

#### Scenario: Successful healing invalidates affected caches
- **WHEN** a Pokemon Center healing operation succeeds
- **THEN** the system MUST invalidate affected Home, party, `my-pokemon`, and healing-history cache entries before subsequent reads

#### Scenario: Unauthenticated Pokemon Center access is rejected
- **WHEN** a healing or healing-history request arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose trainer healing data
