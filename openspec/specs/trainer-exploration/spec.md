## ADDED Requirements

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
- **THEN** the API MUST select one persisted Pokémon available in the active encounter and return it in the event payload

#### Scenario: Walking can generate a Pokeball event
- **WHEN** the random exploration outcome resolves to a Pokeball find
- **THEN** the API MUST add a randomized quantity of pokeballs to the trainer inventory and return the awarded quantity in the event payload

#### Scenario: Exploration event is persisted for future evolution
- **WHEN** a walk action succeeds
- **THEN** the system MUST persist an exploration event record containing the trainer context, event type, and normalized payload

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
The system SHALL expose a Home summary contract for authenticated trainers with the data needed to render the first exploration dashboard.

#### Scenario: Home returns trainer summary and active encounter
- **WHEN** an authenticated trainer requests the Home payload
- **THEN** the API MUST return the trainer summary together with the current active known encounter when one exists

#### Scenario: Home returns the active main party
- **WHEN** an authenticated trainer requests the Home payload
- **THEN** the API MUST return the persisted active party ordered by slot

#### Scenario: Home reads are cached and invalidated after exploration changes
- **WHEN** onboarding, active encounter selection, party changes, or walking mutate trainer exploration state
- **THEN** the system MUST invalidate affected Home and exploration cache entries before subsequent reads
