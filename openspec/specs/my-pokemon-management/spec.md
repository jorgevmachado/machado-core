## MODIFIED Requirements

### Requirement: MyPokemon API exposes normalized create, list, and detail flows
The system SHALL expose authenticated API contracts for creating, listing, detailing, and selecting trainer-owned Pokémon for the trainer's main party with normalized payloads ready for frontend consumption.

#### Scenario: Authenticated user creates MyPokemon
- **WHEN** an authenticated user submits a valid create request with a base Pokémon and optional nickname
- **THEN** the API MUST create the `my-pokemon`, persist generated state, and return a normalized representation of the created resource

#### Scenario: Initial onboarding can create trainer and first MyPokemon together
- **WHEN** an authenticated user without an existing `trainer` completes the initial starter selection flow
- **THEN** the system MUST create the `trainer`, the first `my-pokemon`, the initial `pokedex` collection, and the initial known encounter collection as part of the same onboarding flow

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
- **THEN** the API MUST return the created trainer together with the initial `my-pokemons`, `pokedex`, and known encounter entries created in the same transaction

#### Scenario: Authenticated user lists owned MyPokemon
- **WHEN** an authenticated user requests the `my-pokemon` list
- **THEN** the API MUST return only the current trainer's `my-pokemon` records with pagination metadata and supported filters

#### Scenario: List filters by owned name and base Pokemon name
- **WHEN** an authenticated user lists `my-pokemon`
- **THEN** the API MUST support filtering by the trainer-owned public `name` and by the base `pokemon_name`

#### Scenario: Authenticated user opens MyPokemon detail
- **WHEN** an authenticated user requests one owned `my-pokemon` by `name`
- **THEN** the API MUST return normalized detail data including trainer summary, base Pokémon summary, generated attributes, owned moves, PP state, and capture timestamp

#### Scenario: Trainer can assign owned Pokemon to the main party
- **WHEN** an authenticated trainer updates the main party selection
- **THEN** the system MUST accept only trainer-owned active `my-pokemon` entries as party members

#### Scenario: Unauthenticated access is rejected
- **WHEN** a create, list, or detail request for `my-pokemon` arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose trainer-owned data
