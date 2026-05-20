## MODIFIED Requirements

### Requirement: Walking generates a normalized exploration event
The system SHALL allow authenticated trainers to walk in their active known encounter and receive one normalized exploration event per walk action.

#### Scenario: Walking requires an active known encounter
- **WHEN** an authenticated trainer requests to walk without a valid active known encounter
- **THEN** the API MUST reject the request and MUST NOT generate an exploration event

#### Scenario: Walking returns one normalized event payload
- **WHEN** an authenticated trainer walks in an active known encounter
- **THEN** the API MUST return exactly one normalized exploration event containing the event type and event-specific payload

#### Scenario: Walking can generate a wild Pokemon event
- **WHEN** the random exploration outcome resolves to a wild encounter
- **THEN** the API MUST select one persisted Pokémon available in the active encounter, MUST create or resume the trainer's active wild battle session, and MUST return the event payload with a normalized reference containing at least `battle_session_id`, `battle_status`, and `has_active_battle`

#### Scenario: Walking can generate a Pokeball event
- **WHEN** the random exploration outcome resolves to a Pokeball find
- **THEN** the API MUST add a randomized quantity of pokeballs to the trainer inventory and return the awarded quantity in the event payload

#### Scenario: Exploration event is persisted for future evolution
- **WHEN** a walk action succeeds
- **THEN** the system MUST persist an exploration event record containing the trainer context, event type, and normalized payload

#### Scenario: Walking is blocked while battle is active
- **WHEN** an authenticated trainer already has an active wild battle session and requests to walk again
- **THEN** the API MUST prevent creation of a conflicting new exploration outcome and MUST preserve the existing active battle session
