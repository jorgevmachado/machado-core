## Purpose
Define the canonical trainer exploration, encounter selection, walking, party, and Home summary behavior.
## Requirements
### Requirement: Trainer owns known encounters and one active encounter
The system SHALL persist trainer-specific known encounters derived from local encounter data and enforce at most one active encounter per trainer.

#### Scenario: Onboarding creates known encounters from the starter Pokemon
- **WHEN** an authenticated user completes onboarding with a valid starter Pokémon
- **THEN** the API MUST create trainer-owned encounter records for the persisted encounters linked to that starter Pokémon

#### Scenario: Onboarding activates one known encounter automatically
- **WHEN** onboarding creates more than one known encounter for the trainer
- **THEN** the API MUST mark exactly one encounter as active using a deterministic catalog ordering rule

#### Scenario: Selecting an active encounter deactivates the previous one
- **WHEN** an authenticated trainer selects a different known encounter as active
- **THEN** the API MUST persist the new active encounter and MUST ensure no other trainer-owned encounter remains active

#### Scenario: Unknown encounters cannot be selected
- **WHEN** an authenticated trainer attempts to select an encounter not owned by that trainer
- **THEN** the API MUST reject the request and MUST NOT change the active encounter state

### Requirement: Trainer exploration exposes authenticated list and selection flows
The system SHALL expose authenticated contracts for listing trainer-owned encounters and selecting the active encounter.

#### Scenario: Authenticated user lists trainer encounters
- **WHEN** an authenticated trainer requests the encounter list
- **THEN** the API MUST return only that trainer's known encounters with normalized encounter and active-state data

#### Scenario: Encounter list identifies the active encounter
- **WHEN** the encounter list response is returned
- **THEN** the payload MUST explicitly identify which trainer-owned encounter is active

#### Scenario: Unauthenticated encounter access is rejected
- **WHEN** a list or select-encounter request arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose trainer exploration data

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

### Requirement: Trainer maintains a main party of up to six owned Pokemon
The system SHALL allow authenticated trainers to manage a main party composed only of their owned `my-pokemon` entries, limited to six active slots.

#### Scenario: Trainer sets the active party from owned Pokemon
- **WHEN** an authenticated trainer submits a valid party selection using owned `my-pokemon` identifiers
- **THEN** the API MUST persist the selected party ordered by slot and scoped to that trainer

#### Scenario: Party cannot exceed six Pokemon
- **WHEN** an authenticated trainer attempts to persist more than six active party members
- **THEN** the API MUST reject the request and MUST NOT save the invalid selection

#### Scenario: Party cannot include foreign or deleted Pokemon
- **WHEN** an authenticated trainer attempts to include a `my-pokemon` not owned by that trainer or not active
- **THEN** the API MUST reject the request and MUST NOT save the invalid selection

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
