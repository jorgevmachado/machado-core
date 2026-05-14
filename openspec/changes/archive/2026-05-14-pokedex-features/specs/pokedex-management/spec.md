## ADDED Requirements

### Requirement: Trainer owns one Pokedex entry per base Pokemon
The system SHALL persist a `pokedex` entity representing one trainer-specific Pokédex entry linked to one authenticated trainer and one base `pokemon`.

#### Scenario: Pokedex belongs to the authenticated trainer
- **WHEN** the system creates or updates a `pokedex`
- **THEN** the record MUST remain scoped to the current authenticated trainer

#### Scenario: Pokedex references exactly one base Pokemon
- **WHEN** a `pokedex` entry is persisted
- **THEN** it MUST be linked to exactly one existing base `pokemon`

#### Scenario: Trainer cannot duplicate Pokedex entries for the same Pokemon
- **WHEN** a trainer already has a `pokedex` entry for a base `pokemon`
- **THEN** the system MUST prevent creating a second active entry for the same `trainer` and `pokemon`

#### Scenario: Pokedex supports nickname, discovery fields, and soft delete
- **WHEN** a `pokedex` entry is persisted
- **THEN** the API MUST store optional `nickname`, `discovered`, `discovered_at`, timestamps, and soft delete metadata

### Requirement: Pokedex entries generate individualized progression attributes
The system SHALL generate individualized progression attributes for each newly created `pokedex` entry based on the selected base Pokémon and controlled randomization rules implemented in the API.

#### Scenario: API generates initial attributes for Pokedex
- **WHEN** the API creates a `pokedex`
- **THEN** it MUST generate and persist `hp`, `max_hp`, `speed`, `attack`, `defense`, `special_attack`, `special_defense`, `experience`, and `level`

#### Scenario: Pokedex attributes derive from the base Pokemon
- **WHEN** the API computes initial `pokedex` attributes
- **THEN** the formula MUST use persisted data from the selected base Pokémon as the reference instead of arbitrary standalone values

#### Scenario: Creation fails when the base Pokemon is unavailable
- **WHEN** the system attempts to create a `pokedex` entry for a base `pokemon` that does not exist
- **THEN** the API MUST reject the request and MUST NOT persist a partial `pokedex`

### Requirement: Trainer onboarding initializes the full Pokedex collection
The system SHALL create the trainer Pokédex collection during onboarding using the local Pokémon catalog as the source of truth.

#### Scenario: Onboarding creates one Pokedex entry per base Pokemon
- **WHEN** an authenticated user without an existing `trainer` completes onboarding
- **THEN** the API MUST create one `pokedex` entry for each base `pokemon` currently persisted in the catalog and link them to the new trainer

#### Scenario: Onboarding discovers only the selected starter
- **WHEN** onboarding creates the trainer Pokédex collection
- **THEN** the entry for the chosen starter Pokémon MUST be persisted with `discovered=true` and `discovered_at` set, while all remaining entries MUST start with `discovered=false` and `discovered_at=null`

#### Scenario: Onboarding returns the created trainer aggregate with Pokedex
- **WHEN** onboarding succeeds
- **THEN** the API MUST return the created trainer together with the created `my-pokemons` and `pokedex` entries in the same normalized response

### Requirement: Pokedex API exposes explicit discovery, list, and detail flows
The system SHALL expose authenticated API contracts for discovering, listing, and detailing trainer-owned Pokédex entries.

#### Scenario: Authenticated user lists owned Pokedex entries
- **WHEN** an authenticated user requests the `pokedex` list
- **THEN** the API MUST return only the current trainer's `pokedex` records with pagination metadata and supported filters

#### Scenario: Authenticated user filters Pokedex list
- **WHEN** an authenticated user lists `pokedex`
- **THEN** the API MUST support filtering by base `pokemon_name`, nickname, and discovery status

#### Scenario: Authenticated user opens Pokedex detail
- **WHEN** an authenticated user requests one owned `pokedex` by `pokemon.name`
- **THEN** the API MUST return normalized detail data including trainer summary, base Pokémon summary, generated attributes, discovery state, and discovery timestamp

#### Scenario: Authenticated user explicitly discovers a Pokedex entry
- **WHEN** an authenticated user calls the explicit discovery endpoint for one trainer-owned `pokedex` entry
- **THEN** the API MUST mark that entry as discovered and MUST set `discovered_at` if it was not already set

#### Scenario: Discovery endpoint is idempotent
- **WHEN** an authenticated user calls the discovery endpoint for an already discovered `pokedex` entry
- **THEN** the API MUST preserve the existing discovered state and MUST NOT create duplicate records

#### Scenario: Unauthenticated access is rejected
- **WHEN** a list, detail, or discovery request for `pokedex` arrives without valid authentication
- **THEN** the API and BFF layers MUST reject the request and MUST NOT expose trainer-owned data

### Requirement: Pokedex reads use cache and efficient queries
The system SHALL optimize trainer-owned Pokédex reads with Redis caching and efficient query behavior.

#### Scenario: List response is cached per trainer and filters
- **WHEN** an authenticated trainer requests the `pokedex` list with pagination and filters
- **THEN** the API MUST use a deterministic Redis cache key scoped to the trainer and request parameters

#### Scenario: Detail response is cached per trainer and base Pokemon name
- **WHEN** an authenticated trainer requests `pokedex` detail
- **THEN** the API MUST use a deterministic Redis cache key scoped to the trainer-owned resource and the base `pokemon.name`

#### Scenario: Cache invalidates after discovery or onboarding initialization
- **WHEN** the system initializes or updates trainer-owned `pokedex` state
- **THEN** it MUST invalidate affected list and detail cache entries before subsequent reads

#### Scenario: Reads avoid N plus one queries
- **WHEN** the API builds list or detail responses for `pokedex`
- **THEN** repositories and services MUST load trainer and base Pokémon relationships efficiently without avoidable N plus one query behavior

### Requirement: Pokedex web experience uses the existing protected architecture
The system SHALL provide protected web flows for listing and detailing trainer-owned Pokédex entries using the existing `machado-web` BFF, service, hook, and design system patterns.

#### Scenario: Protected routes expose Pokedex pages
- **WHEN** an authenticated user opens `/pokedex` or `/pokedex/[name]`
- **THEN** the web app MUST render protected pages for Pokédex list and detail using the existing protected App Router structure

#### Scenario: Web list and detail consume normalized contracts
- **WHEN** the BFF returns `pokedex` list or detail data
- **THEN** the web app MUST render nickname, base Pokémon image, level, experience, HP/max HP, generated attributes, discovery status, and discovery date using existing design system patterns

#### Scenario: Web list supports pagination and filters
- **WHEN** the user paginates or filters the owned Pokédex roster
- **THEN** the page MUST use the existing paginated list hook pattern and MUST render loading, error, empty, and success states consistent with the rest of the application

#### Scenario: Web keeps manual discovery UI out of scope for this change
- **WHEN** the frontend consumes the new `pokedex` capability
- **THEN** it MUST support list and detail flows without introducing a dedicated manual discovery interface in this implementation
