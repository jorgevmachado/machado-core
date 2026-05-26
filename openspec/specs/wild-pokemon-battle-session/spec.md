## ADDED Requirements

### Requirement: Active battle sessions block Pokemon Center healing
The system SHALL prevent Pokemon Center healing from mutating trainer resources while a battle session is active.

#### Scenario: Active battle blocks healing attempts
- **WHEN** an authenticated trainer with an active battle session attempts to execute Pokemon Center healing
- **THEN** the system MUST reject the healing request as incompatible with the active battle state

#### Scenario: Blocked healing does not mutate battle state
- **WHEN** Pokemon Center healing is rejected because a battle session is active
- **THEN** the system MUST preserve the current battle session, turn state, and battle logs without side effects

### Requirement: Trainer can attempt capture during an active wild battle
The system SHALL allow the trainer to use capture as an authenticated battle action against the active wild Pokemon in the current battle session.

#### Scenario: Eligible capture attempt advances the battle turn
- **WHEN** an authenticated trainer uses the battle capture action during an active battle and the Pokemon is eligible for capture
- **THEN** the system MUST process the attempt as a trainer battle action within the session turn flow

#### Scenario: Failed eligible capture triggers wild response
- **WHEN** an eligible capture attempt fails and the battle remains active
- **THEN** the system MUST resolve one automatic response action from the wild Pokemon in the same turn cycle

#### Scenario: Capture success finishes the battle session
- **WHEN** a capture attempt succeeds during an active battle
- **THEN** the system MUST finish the battle session and MUST NOT process an automatic wild response after success

## MODIFIED Requirements

### Requirement: Trainer can escape and the battle can end with normalized final status
The system SHALL expose normalized end states for battle completion, capture completion, and escape using explicit public status names aligned across specs, backend, BFF, and frontend.

#### Scenario: Escaping finishes the session
- **WHEN** the trainer chooses to flee an active wild battle
- **THEN** the system MUST finish the session with status `ESCAPED`

#### Scenario: Wild Pokemon defeat finishes the session
- **WHEN** the wild Pokemon reaches zero battle HP
- **THEN** the system MUST finish the session with status `WILD_POKEMON_DEFEATED`

#### Scenario: Trainer defeat finishes the session
- **WHEN** the trainer no longer has a valid active battle Pokemon with remaining battle HP
- **THEN** the system MUST finish the session with status `TRAINER_DEFEATED`

#### Scenario: Successful capture finishes the session
- **WHEN** the trainer successfully captures the active wild Pokemon
- **THEN** the system MUST finish the session with status `CAPTURED`

#### Scenario: Public contracts do not expose legacy terminal names
- **WHEN** battle session data is returned by API, BFF, or web-facing contracts
- **THEN** the system MUST NOT expose ambiguous terminal names such as `WON`, `LOST`, or `FLED` as the canonical public contract

### Requirement: API exposes authenticated battle session endpoints
The system SHALL provide authenticated endpoints for battle session retrieval and actions under an explicit battle namespace.

#### Scenario: Trainer reads active battle session
- **WHEN** an authenticated trainer requests the active battle session
- **THEN** the API MUST return the normalized active session for that trainer or `404 Not Found` with an explicit "no active battle" detail as the canonical absence contract

#### Scenario: Trainer lists battle logs
- **WHEN** an authenticated trainer requests battle logs for an owned session
- **THEN** the API MUST return the normalized battle log entries in chronological turn order

#### Scenario: Battle actions use explicit namespace
- **WHEN** authenticated battle-session reads or actions are exposed
- **THEN** the API and BFF contracts MUST use explicit battle-scoped paths under `/trainer/battle/*`

#### Scenario: Capture action uses battle namespace
- **WHEN** the trainer attempts to capture the active wild Pokemon
- **THEN** the API and BFF contracts MUST expose that action under `/trainer/battle/capture`

#### Scenario: Unauthenticated battle access is rejected
- **WHEN** a battle session request or action arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose battle state
