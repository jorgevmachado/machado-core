## MODIFIED Requirements

### Requirement: Trainer Home exposes exploration-ready summary data
The system SHALL expose a Home summary contract for authenticated trainers with the data needed to render the first exploration dashboard and the latest healing outcome.

#### Scenario: Home returns trainer summary and active encounter
- **WHEN** an authenticated trainer requests the Home payload
- **THEN** the API MUST return the trainer summary together with the current active known encounter when one exists

#### Scenario: Home returns the active main party
- **WHEN** an authenticated trainer requests the Home payload
- **THEN** the API MUST return the persisted active party ordered by slot

#### Scenario: Home returns active battle summary when one exists
- **WHEN** an authenticated trainer requests the Home payload while a battle session is active
- **THEN** the API MUST include a normalized battle summary sufficient to resume or reopen that active battle flow

#### Scenario: Home returns the latest healing summary when one exists
- **WHEN** an authenticated trainer requests the Home payload after at least one successful Pokemon Center healing operation
- **THEN** the API MUST include an explicit normalized summary of the latest healing event with aggregate restored totals and timestamp

#### Scenario: Home reads are cached and invalidated after exploration changes
- **WHEN** onboarding, active encounter selection, party changes, or walking mutate trainer exploration state
- **THEN** the system MUST invalidate affected Home and exploration cache entries before subsequent reads

#### Scenario: Home reads are invalidated after battle lifecycle changes
- **WHEN** battle session creation, relevant battle progress, or battle termination changes the trainer's active battle summary
- **THEN** the system MUST invalidate affected Home cache entries before subsequent reads

#### Scenario: Home reads are invalidated after healing changes
- **WHEN** a successful Pokemon Center healing operation changes the trainer party state or latest healing summary
- **THEN** the system MUST invalidate affected Home cache entries before subsequent reads
