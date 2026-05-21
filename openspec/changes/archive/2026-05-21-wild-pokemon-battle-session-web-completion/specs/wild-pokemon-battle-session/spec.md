## ADDED Requirements

### Requirement: Web battle session screen renders normalized active battle state
The system SHALL provide a protected web battle-session view that renders the trainer active Pokemon and wild Pokemon using normalized data from the existing battle-session contracts.

#### Scenario: Trainer opens an active battle session view
- **WHEN** an authenticated trainer navigates to the battle-session screen with an active session
- **THEN** the web interface MUST display session status, trainer and wild active Pokemon, battle HP values, available moves, and move PP from the current API/BFF response

#### Scenario: No active battle exists for the trainer
- **WHEN** an authenticated trainer reaches the battle view without an active session
- **THEN** the web interface MUST show an explicit empty state and MUST NOT fabricate local battle state

### Requirement: Web battle actions consume existing action endpoints
The system SHALL execute battle actions in the web interface exclusively through existing authenticated battle-session endpoints.

#### Scenario: Trainer uses a move from the action panel
- **WHEN** the trainer submits a valid move action in an active battle
- **THEN** the frontend MUST call the existing move-action endpoint and MUST refresh rendered battle state and logs from the response flow

#### Scenario: Move has no remaining PP in battle state
- **WHEN** the rendered move data indicates zero remaining PP
- **THEN** the frontend MUST render that move as disabled and MUST prevent dispatching the action request

#### Scenario: Battle is already finished
- **WHEN** the session status is terminal (`ESCAPED`, `WILD_POKEMON_DEFEATED`, or `TRAINER_DEFEATED`)
- **THEN** the frontend MUST block action controls for move, switch, and flee

### Requirement: Web battle flow supports switch, flee, and ordered battle logs
The system SHALL expose switch and flee actions in the battle UI and SHALL show normalized battle logs in chronological order.

#### Scenario: Trainer switches active party member
- **WHEN** the trainer selects a valid switch target in an active battle
- **THEN** the frontend MUST call the existing switch endpoint and MUST update the rendered active trainer Pokemon from the returned session state

#### Scenario: Trainer flees an active battle
- **WHEN** the trainer confirms the flee action
- **THEN** the frontend MUST call the existing flee endpoint and MUST render the resulting terminal session state

#### Scenario: Battle logs are rendered in order
- **WHEN** log data is loaded for the current session
- **THEN** the frontend MUST display log entries in chronological turn order provided by the API/BFF contracts

### Requirement: Web battle state stays synchronized during active sessions
The system SHALL keep the battle view synchronized while a battle is active via periodic refresh and explicit post-action synchronization.

#### Scenario: Polling runs only while battle is active
- **WHEN** the trainer remains on the battle screen and the session is active
- **THEN** the frontend MUST periodically refresh battle state and MUST stop polling when the battle reaches a terminal status or the view unmounts

#### Scenario: Battle request fails
- **WHEN** a battle read or action request fails
- **THEN** the frontend MUST present an explicit error state with retry capability and MUST preserve server authority for subsequent state recovery

### Requirement: Exploration handoff opens the existing battle session flow
The system SHALL connect exploration wild-encounter outcomes to the web battle-session screen using the normalized active-battle reference.

#### Scenario: Exploration response indicates active battle
- **WHEN** exploration returns `has_active_battle=true` with `battle_session_id`
- **THEN** the frontend MUST route the trainer to the battle-session screen associated with that active session
