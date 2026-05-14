## ADDED Requirements

### Requirement: Trainer can own individual Pokemon records
The system SHALL persist a `my-pokemon` entity representing a single trainer-owned Pokémon linked to one authenticated trainer and one base `pokemon`.

#### Scenario: MyPokemon belongs to the authenticated trainer
- **WHEN** an authenticated user creates a `my-pokemon`
- **THEN** the API MUST persist the record with the current authenticated trainer as owner

#### Scenario: MyPokemon references one base Pokemon
- **WHEN** a `my-pokemon` is created
- **THEN** the API MUST link it to exactly one existing base `pokemon`

#### Scenario: MyPokemon supports nickname and capture timestamps
- **WHEN** a `my-pokemon` is persisted
- **THEN** the API MUST store optional `nickname`, `created_at`, and `captured_at`

#### Scenario: Empty nickname falls back to the Pokemon name
- **WHEN** the create flow receives an empty, missing, or whitespace-only nickname
- **THEN** the API MUST persist the selected base Pokémon `name` as the effective nickname

#### Scenario: Public name is derived from the effective nickname
- **WHEN** the API persists a trainer-owned Pokémon
- **THEN** it MUST generate a stable public `name` identifier from the effective nickname for use in list/detail routes

#### Scenario: Public name remains unique per trainer
- **WHEN** the generated public `name` collides with another owned Pokémon of the same trainer
- **THEN** the API MUST generate a unique suffixed variant instead of overwriting or rejecting the existing record

#### Scenario: MyPokemon uses soft delete
- **WHEN** a `my-pokemon` or a trainer-owned move association is removed from active use
- **THEN** the system MUST use soft delete fields and MUST exclude soft-deleted rows from standard queries

### Requirement: MyPokemon creation generates individualized progression state
The system SHALL generate individualized progression attributes for each newly created `my-pokemon` based on the selected base Pokémon and controlled randomization rules implemented in the API.

#### Scenario: API generates initial battle attributes
- **WHEN** the API creates a `my-pokemon`
- **THEN** it MUST generate and persist `hp`, `max_hp`, `speed`, `attack`, `defense`, `special_attack`, `special_defense`, `experience`, and `level`

#### Scenario: Generated attributes derive from the base Pokemon
- **WHEN** the API computes initial `my-pokemon` attributes
- **THEN** the formula MUST use persisted data from the selected base Pokémon as the reference instead of arbitrary standalone values

#### Scenario: Creation fails when the base Pokemon is unavailable
- **WHEN** a user requests creation with a base `pokemon` that does not exist or is unavailable for selection
- **THEN** the API MUST reject the request and MUST NOT persist a partial `my-pokemon`

### Requirement: MyPokemon starts with up to four distinct moves and individual PP state
The system SHALL assign each new `my-pokemon` up to four distinct moves derived from the selected base Pokémon and store PP state independently from the base move catalog.

#### Scenario: Creation selects up to four distinct moves
- **WHEN** a `my-pokemon` is created from a base Pokémon with available move data
- **THEN** the API MUST persist up to four distinct associated moves for that `my-pokemon`

#### Scenario: Move PP is individualized
- **WHEN** a move is associated to a `my-pokemon`
- **THEN** the API MUST persist both `pp` and `max_pp` on the trainer-owned move association by copying the base move `pp` value without mutating the base move record

#### Scenario: Creation continues when fewer than four moves are available
- **WHEN** the selected base Pokémon provides fewer than four eligible local moves
- **THEN** the API MUST create the `my-pokemon` with the available distinct move set instead of failing solely because the count is below four

#### Scenario: Structure supports future move replacement
- **WHEN** future level-up or battle systems need to replace a `my-pokemon` move
- **THEN** the persisted model MUST allow updating the trainer-owned move association without altering the base Pokémon catalog contract

### Requirement: MyPokemon API exposes normalized create, list, and detail flows
The system SHALL expose authenticated API contracts for creating, listing, and detailing trainer-owned Pokémon with normalized payloads ready for frontend consumption.

#### Scenario: Authenticated user creates MyPokemon
- **WHEN** an authenticated user submits a valid create request with a base Pokémon and optional nickname
- **THEN** the API MUST create the `my-pokemon`, persist generated state, and return a normalized representation of the created resource

#### Scenario: Initial onboarding can create trainer and first MyPokemon together
- **WHEN** an authenticated user without an existing `trainer` completes the initial starter selection flow
- **THEN** the system MUST create the `trainer` and the first `my-pokemon` as part of the same onboarding flow

#### Scenario: Onboarding is exposed by the trainer domain
- **WHEN** the web or another client starts the initial trainer setup flow
- **THEN** the canonical API entrypoint MUST be `POST /trainer/onboarding` rather than `POST /my-pokemon`

#### Scenario: Onboarding accepts optional nickname for any role
- **WHEN** a user completes the onboarding flow as admin or non-admin
- **THEN** the flow MUST accept an optional nickname field for the first `my-pokemon`

#### Scenario: Admin onboarding may override trainer defaults
- **WHEN** an admin user completes onboarding and provides `pokeballs` or `capture_rate`
- **THEN** the API MUST accept those initial trainer values within validation limits

#### Scenario: Non-admin onboarding uses default trainer values
- **WHEN** a non-admin user completes onboarding
- **THEN** the API MUST ignore custom trainer inventory parameters and initialize the trainer with standard default values

#### Scenario: Onboarding response returns the created trainer aggregate
- **WHEN** onboarding succeeds
- **THEN** the API MUST return the created trainer together with the initial `my-pokemons` roster entries created in the same transaction

#### Scenario: Authenticated user lists owned MyPokemon
- **WHEN** an authenticated user requests the `my-pokemon` list
- **THEN** the API MUST return only the current trainer's `my-pokemon` records with pagination metadata and supported filters

#### Scenario: List filters by owned name and base Pokemon name
- **WHEN** an authenticated user lists `my-pokemon`
- **THEN** the API MUST support filtering by the trainer-owned public `name` and by the base `pokemon_name`

#### Scenario: Authenticated user opens MyPokemon detail
- **WHEN** an authenticated user requests one owned `my-pokemon` by `name`
- **THEN** the API MUST return normalized detail data including trainer summary, base Pokémon summary, generated attributes, owned moves, PP state, and capture timestamp

#### Scenario: Unauthenticated access is rejected
- **WHEN** a create, list, or detail request for `my-pokemon` arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose trainer-owned data

### Requirement: MyPokemon list supports cache, filters, and efficient reads
The system SHALL optimize trainer-owned Pokémon reads with Redis caching and efficient query behavior.

#### Scenario: List response is cached per trainer and filters
- **WHEN** an authenticated trainer requests the `my-pokemon` list with pagination and filters
- **THEN** the API MUST use a deterministic Redis cache key scoped to the trainer and request parameters

#### Scenario: Detail response is cached per owned Pokemon name
- **WHEN** an authenticated trainer requests `my-pokemon` detail
- **THEN** the API MUST use a deterministic Redis cache key scoped to that trainer-owned Pokémon resource and its external `name`

#### Scenario: Cache invalidates after create or relevant update
- **WHEN** the system creates or changes a `my-pokemon` or its owned move state
- **THEN** it MUST invalidate affected list and detail cache entries before subsequent reads

#### Scenario: Operators can force a list cache refresh
- **WHEN** a list request explicitly asks to bypass stale cached data
- **THEN** the API MUST allow cache cleanup before rebuilding the list response

#### Scenario: Reads avoid N plus one queries
- **WHEN** the API builds list or detail responses for `my-pokemon`
- **THEN** repositories and services MUST load trainer, base Pokémon, and owned move relationships efficiently without avoidable N plus one query behavior

### Requirement: MyPokemon web experience uses the existing protected architecture
The system SHALL provide protected web flows for listing and detailing trainer-owned Pokémon and a home-based capture flow using the existing `machado-web` BFF, service, hook, and design system patterns.

#### Scenario: Protected routes expose MyPokemon pages
- **WHEN** an authenticated user opens `/my-pokemon` or `/my-pokemon/[name]`
- **THEN** the web app MUST render protected pages for roster list and detail using the existing protected App Router structure

#### Scenario: Home starts the initial trainer onboarding flow
- **WHEN** an authenticated user without an existing `trainer` opens the home page
- **THEN** the page MUST expose the initial onboarding flow to create the trainer and choose the first `my-pokemon`

#### Scenario: Non-admin onboarding offers only the three starters
- **WHEN** a non-admin user without `trainer` opens the onboarding flow
- **THEN** the web app MUST offer only `Bulbasaur`, `Charmander`, and `Squirtle` as valid first-Pokémon choices

#### Scenario: Onboarding form exposes optional nickname
- **WHEN** the onboarding flow is displayed
- **THEN** the web app MUST show an optional nickname input for the first `my-pokemon`

#### Scenario: Admin onboarding allows free Pokemon selection
- **WHEN** an admin user without `trainer` opens the onboarding flow
- **THEN** the web app MUST allow free selection from the existing Pokémon list using autocomplete

#### Scenario: Admin selection uses preview card
- **WHEN** an admin selects a Pokémon from autocomplete during onboarding
- **THEN** the web app MUST show a preview card similar to the Pokémon list card for the selected option

#### Scenario: Web creates MyPokemon through BFF
- **WHEN** the frontend submits the onboarding form with the selected Pokémon and any allowed fields for the current user role
- **THEN** it MUST call a Next.js BFF route handler that authenticates server-side and delegates to `machado-api` to create the trainer-owned Pokémon

#### Scenario: Home onboarding uses the trainer BFF endpoint
- **WHEN** the home onboarding form is submitted
- **THEN** the frontend MUST call `/api/trainer/onboarding`, which delegates to the trainer onboarding API contract

#### Scenario: Existing Pokemon list remains the admin source for onboarding selection
- **WHEN** an admin uses the onboarding autocomplete to choose the first Pokémon
- **THEN** the web app MUST source the options from the existing Pokémon list contract instead of creating a separate Pokémon-selection API

#### Scenario: Web list and detail consume normalized contracts
- **WHEN** the BFF returns `my-pokemon` list or detail data
- **THEN** the web app MUST render nickname, base Pokémon image, level, experience, HP/max HP, battle attributes, moves, PP, move metadata, trainer summary, and capture date using existing DS patterns

#### Scenario: Web list supports pagination and filters
- **WHEN** the user paginates or filters the owned roster
- **THEN** the page MUST use the existing paginated list hook pattern and MUST render loading, error, empty, and success states consistent with the rest of the application

#### Scenario: Web does not duplicate business rules
- **WHEN** the web app needs creation validation, attribute generation, move selection, or PP rules
- **THEN** it MUST rely on the API contract and MUST NOT implement duplicate business logic in the frontend

#### Scenario: Home replaces the default welcome view for users without trainer
- **WHEN** an authenticated user has no `trainer` associated in the user context
- **THEN** the protected home page MUST render the onboarding experience instead of the standard post-login content
