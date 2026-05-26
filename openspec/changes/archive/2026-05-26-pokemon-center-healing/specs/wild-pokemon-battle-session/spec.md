## ADDED Requirements

### Requirement: Active battle sessions block Pokemon Center healing
The system SHALL prevent Pokemon Center healing from mutating trainer resources while a battle session is active.

#### Scenario: Active battle blocks healing attempts
- **WHEN** an authenticated trainer with an active battle session attempts to execute Pokemon Center healing
- **THEN** the system MUST reject the healing request as incompatible with the active battle state

#### Scenario: Blocked healing does not mutate battle state
- **WHEN** Pokemon Center healing is rejected because a battle session is active
- **THEN** the system MUST preserve the current battle session, turn state, and battle logs without side effects
