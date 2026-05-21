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
- **THEN** the API MUST select one persisted Pokémon available in the active encounter, MUST create or resume the trainer's active battle session, and MUST return the event payload with a normalized reference containing at least `battle_session_id`, `battle_status`, and `has_active_battle`

#### Scenario: Walking can generate a Pokeball event
- **WHEN** the random exploration outcome resolves to a Pokeball find
- **THEN** the API MUST add a randomized quantity of pokeballs to the trainer inventory and return the awarded quantity in the event payload

#### Scenario: Exploration event is persisted for future evolution
- **WHEN** a walk action succeeds
- **THEN** the system MUST persist an exploration event record containing the trainer context, event type, and normalized payload

#### Scenario: Walking is blocked while battle is active
- **WHEN** an authenticated trainer already has an active battle session and requests to walk again
- **THEN** the API MUST prevent creation of a conflicting new exploration outcome and MUST preserve the existing active battle session

#### Scenario: Encounter handoff opens modal instead of route navigation
- **WHEN** the web frontend consumes a walk response with `has_active_battle=true`
- **THEN** the primary UX handoff MUST open the battle flow in a modal instead of automatically navigating to `/battle`

### Requirement: Trainer Home exposes exploration-ready summary data
The system SHALL expose a Home summary contract for authenticated trainers with the data needed to render the first exploration dashboard.

#### Scenario: Home returns trainer summary and active encounter
- **WHEN** an authenticated trainer requests the Home payload
- **THEN** the API MUST return the trainer summary together with the current active known encounter when one exists

#### Scenario: Home returns the active main party
- **WHEN** an authenticated trainer requests the Home payload
- **THEN** the API MUST return the persisted active party ordered by slot

#### Scenario: Home returns active battle summary when one exists
- **WHEN** an authenticated trainer requests the Home payload while a battle session is active
- **THEN** the API MUST include a normalized battle summary sufficient to resume or reopen that active battle flow

#### Scenario: Home reads are cached and invalidated after exploration changes
- **WHEN** onboarding, active encounter selection, party changes, or walking mutate trainer exploration state
- **THEN** the system MUST invalidate affected Home and exploration cache entries before subsequent reads

#### Scenario: Home reads are invalidated after battle lifecycle changes
- **WHEN** battle session creation, relevant battle progress, or battle termination changes the trainer's active battle summary
- **THEN** the system MUST invalidate affected Home cache entries before subsequent reads
