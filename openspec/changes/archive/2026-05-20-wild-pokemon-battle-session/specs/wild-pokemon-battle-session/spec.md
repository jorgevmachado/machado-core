## ADDED Requirements

### Requirement: Wild Pokemon battle session starts automatically from exploration
The system SHALL create or return a trainer-scoped wild Pokemon battle session automatically when a wild encounter exploration event is produced.

#### Scenario: Walking into a wild Pokemon starts a battle session
- **WHEN** an authenticated trainer walks and the exploration outcome is a wild Pokemon encounter
- **THEN** the system MUST create a battle session linked to that trainer and return a normalized reference containing at least `battle_session_id`, `battle_status`, and `has_active_battle`

#### Scenario: Trainer cannot own multiple simultaneous active battle sessions
- **WHEN** an authenticated trainer already has an active wild Pokemon battle session
- **THEN** the system MUST reject creation of a second active session and MUST return the existing active session instead of duplicating state

### Requirement: Battle session persists normalized battle state
The system SHALL persist the active battle state needed to continue the fight across requests.

#### Scenario: Session tracks both sides and status
- **WHEN** a battle session is created or loaded
- **THEN** the payload MUST include a unique session identifier, session status, the active trainer Pokemon, and the active wild Pokemon

#### Scenario: Session tracks temporary HP and PP state
- **WHEN** the trainer or wild Pokemon enters battle
- **THEN** the system MUST persist temporary battle HP and movement PP state in the active session without mutating permanent progression state outside the battle session

### Requirement: Battle actions are turn-based and logged
The system SHALL process battle actions through ordered turns and persist normalized logs for the frontend.

#### Scenario: Using a move creates a turn entry
- **WHEN** the trainer uses a valid move during an active battle
- **THEN** the system MUST process the move as a new battle turn and MUST persist a normalized battle log entry for the resulting action

#### Scenario: Move usage consumes PP
- **WHEN** a move is successfully used in battle
- **THEN** the system MUST decrement the remaining PP for that move in the active battle state

#### Scenario: Move without PP is rejected
- **WHEN** the trainer attempts to use a move with no remaining PP in the active battle state
- **THEN** the system MUST reject the action and MUST NOT advance the battle turn

#### Scenario: Wild Pokemon responds automatically after the trainer action
- **WHEN** the trainer executes a valid action and the battle remains active
- **THEN** the system MUST resolve one automatic response action from the wild Pokemon in the same turn cycle

#### Scenario: V1 damage resolution remains simple and deterministic
- **WHEN** the battle engine resolves a move in the initial implementation
- **THEN** the system MUST use an explicit simple damage rule without advanced priority, weather, status effects, or secondary move effects

### Requirement: Trainer can switch party members during an active battle
The system SHALL allow the trainer to switch to another valid active party member while the battle session remains active.

#### Scenario: Switching changes the active trainer Pokemon
- **WHEN** the trainer selects another valid Pokemon from the active party during battle
- **THEN** the system MUST update the session so the selected party member becomes the active trainer Pokemon

#### Scenario: Invalid switch target is rejected
- **WHEN** the trainer attempts to switch to a Pokemon not present in the active party or unavailable for battle
- **THEN** the system MUST reject the action and MUST NOT change the active trainer Pokemon

### Requirement: Trainer can escape and the battle can end with normalized final status
The system SHALL expose normalized end states for battle completion and escape.

#### Scenario: Escaping finishes the session
- **WHEN** the trainer chooses to flee an active wild battle
- **THEN** the system MUST finish the session with status `ESCAPED`

#### Scenario: Wild Pokemon defeat finishes the session
- **WHEN** the wild Pokemon reaches zero battle HP
- **THEN** the system MUST finish the session with status `WILD_POKEMON_DEFEATED`

#### Scenario: Trainer defeat finishes the session
- **WHEN** the trainer no longer has a valid active battle Pokemon with remaining battle HP
- **THEN** the system MUST finish the session with status `TRAINER_DEFEATED`

### Requirement: API exposes authenticated battle session endpoints
The system SHALL provide authenticated endpoints for battle session retrieval and actions.

#### Scenario: Trainer reads active battle session
- **WHEN** an authenticated trainer requests the active battle session
- **THEN** the API MUST return the normalized active session for that trainer or an explicit empty result when none exists

#### Scenario: Trainer lists battle logs
- **WHEN** an authenticated trainer requests battle logs for an owned session
- **THEN** the API MUST return the normalized battle log entries in chronological turn order

#### Scenario: Unauthenticated battle access is rejected
- **WHEN** a battle session request or action arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose battle state

### Requirement: Exploration is blocked while a wild battle session is active
The system SHALL prevent the trainer from progressing exploration into a conflicting new encounter while a wild battle session remains active.

#### Scenario: Walking is blocked during an active battle
- **WHEN** an authenticated trainer with an active wild battle session requests a new exploration walk
- **THEN** the system MUST reject or short-circuit the exploration action and MUST preserve the existing active battle session
