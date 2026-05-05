# pokemon-catalog Specification

## Purpose
TBD - created by archiving change listagem-e-detalhe-pokemon. Update Purpose after archive.
## Requirements
### Requirement: Pokemon list initializes local catalog
The system SHALL initialize the local Pokemon catalog automatically when an authenticated Pokemon list request is received and the local catalog is empty.

#### Scenario: Empty catalog triggers initial sync
- **WHEN** an authenticated user requests the Pokemon list and there are no local Pokemon records
- **THEN** the API MUST call the PokeAPI Pokemon list endpoint only and persist minimal local Pokemon records as `INCOMPLETE`

#### Scenario: Initial sync stores minimal fields
- **WHEN** the API imports Pokemon from the external list response
- **THEN** each imported Pokemon MUST store `name`, derived `order`, derived `external_image`, and status `INCOMPLETE`

#### Scenario: Initial sync avoids detail calls
- **WHEN** the initial catalog sync runs from the Pokemon list endpoint
- **THEN** the API MUST NOT call external Pokemon detail, species, move, type, ability, growth-rate, encounter, or evolution-chain endpoints during that initial sync

### Requirement: Pokemon order and external image are derived from external URL
The system SHALL derive Pokemon `order` from the numeric identifier at the end of the PokeAPI list item URL and SHALL derive `external_image` from that order.

#### Scenario: Order is extracted from external URL
- **WHEN** a PokeAPI list item contains URL `https://pokeapi.co/api/v2/pokemon/25/`
- **THEN** the imported Pokemon MUST store `order` as `25`

#### Scenario: External image uses four digit order
- **WHEN** the imported Pokemon order is `25`
- **THEN** the imported Pokemon MUST store `external_image` as `https://www.pokemon.com/static-assets/content-assets/cms2/img/pokedex/detail/0025.png`

#### Scenario: External image normalizes leading zeros
- **WHEN** the derived order value is received as `01`
- **THEN** the image order segment MUST be normalized to `0001`

### Requirement: Pokemon list supports pagination and filters
The system SHALL expose an authenticated Pokemon list endpoint with pagination and filters.

#### Scenario: List returns paginated Pokemon
- **WHEN** an authenticated user requests the Pokemon list with pagination parameters
- **THEN** the API MUST return a paginated response containing Pokemon list items and pagination metadata

#### Scenario: List filters by name
- **WHEN** an authenticated user requests the Pokemon list with a name filter
- **THEN** the API MUST return only Pokemon whose name matches the requested filter

#### Scenario: List filters by order
- **WHEN** an authenticated user requests the Pokemon list with an order filter
- **THEN** the API MUST return only the Pokemon with the matching order

#### Scenario: List filters by status
- **WHEN** an authenticated user requests the Pokemon list with a status filter
- **THEN** the API MUST return only Pokemon with the matching status

#### Scenario: List filters by type
- **WHEN** an authenticated user requests the Pokemon list with a type filter
- **THEN** the API MUST return only Pokemon associated with the matching Pokemon type

### Requirement: Pokemon detail enriches incomplete records
The system SHALL enrich a local Pokemon record when its detail is requested and its status is `INCOMPLETE`.

#### Scenario: Incomplete detail triggers enrichment
- **WHEN** an authenticated user requests the detail of a Pokemon with status `INCOMPLETE`
- **THEN** the API MUST fetch the necessary external Pokemon detail data, persist enriched data, update the Pokemon status to `COMPLETE`, and return the complete detail response

#### Scenario: Complete detail avoids redundant enrichment
- **WHEN** an authenticated user requests the detail of a Pokemon with status `COMPLETE`
- **THEN** the API MUST return local data without repeating external enrichment calls

#### Scenario: Unknown Pokemon returns not found
- **WHEN** an authenticated user requests the detail of a Pokemon identifier that does not exist locally after synchronization rules are applied
- **THEN** the API MUST return a not found response

### Requirement: Pokemon detail returns normalized rich data
The system SHALL return Pokemon detail as a normalized API contract suitable for direct frontend consumption.

#### Scenario: Detail includes core Pokemon data
- **WHEN** an authenticated user requests Pokemon detail
- **THEN** the response MUST include Pokemon identity, order, name, status, external image, stats, height, weight, base experience, species metadata, timestamps, and available relationships

#### Scenario: Detail includes related catalog data
- **WHEN** enriched related data exists for the Pokemon
- **THEN** the response MUST include normalized types, abilities, moves, growth rate, habitat, shape, images, encounters, evolution information, weaknesses, and strengths

### Requirement: Pokemon images are processed by image business rules
The system SHALL process Pokemon sprite images through the `pokemon_image` domain business rules before persistence.

#### Scenario: Sprites are flattened into image records
- **WHEN** the Pokemon detail payload contains a nested `sprites` object
- **THEN** `pokemon_image/business.py` MUST extract valid image URLs and infer `source`, `variant`, `generation`, `game`, and `media_type` where available

#### Scenario: Images field stores serialized treated URLs
- **WHEN** sprite image URLs are processed
- **THEN** the Pokemon image record MUST store all treated image URL entries in the `images` field as a JSON string

#### Scenario: Primary image is selected from processed sprites
- **WHEN** processed sprite images include preferred official artwork or home artwork
- **THEN** the image business rules MUST select a primary image for catalog display

### Requirement: Pokemon catalog uses Redis cache
The system SHALL cache Pokemon list and detail responses in Redis with deterministic keys and a TTL greater than 2 hours.

#### Scenario: List response is served from cache
- **WHEN** a valid cached Pokemon list response exists for the requested filters and pagination
- **THEN** the API MUST return the cached response without querying the database for the list payload

#### Scenario: Detail response is served from cache
- **WHEN** a valid cached Pokemon detail response exists for the requested identifier
- **THEN** the API MUST return the cached detail response without rebuilding the detail payload

#### Scenario: Cache invalidates after enrichment
- **WHEN** a Pokemon is enriched or updated
- **THEN** the API MUST invalidate affected Pokemon list and detail cache entries before returning the updated response

### Requirement: Pokemon catalog data avoids duplicates and supports soft delete
The system SHALL enforce consistency for Pokemon catalog data with unique constraints, idempotent writes, and soft delete support where applicable.

#### Scenario: Repeated initial sync is idempotent
- **WHEN** the initial sync process runs more than once
- **THEN** the API MUST avoid duplicate Pokemon and related catalog records

#### Scenario: Soft deleted rows are excluded by default
- **WHEN** Pokemon catalog queries are executed
- **THEN** records with active `deleted_at` values MUST be excluded unless a query explicitly requires them

### Requirement: Pokemon domains remain independent
The system SHALL keep each Pokemon catalog domain independent by exposing cross-domain operations through services only.

#### Scenario: Pokemon service updates move data
- **WHEN** `PokemonService` needs to create, update, or read `PokemonMove` data during synchronization or enrichment
- **THEN** it MUST call `PokemonMoveService` and MUST NOT access `PokemonMoveRepository` directly

#### Scenario: Domain repository stays internal
- **WHEN** any domain needs data owned by another Pokemon catalog domain
- **THEN** it MUST use the owning domain service instead of importing the owning domain repository

### Requirement: Pokemon catalog exposes web list and detail pages
The system SHALL provide protected web pages for Pokemon list and detail using the normalized API contracts.

#### Scenario: User views Pokemon list page
- **WHEN** an authenticated user opens `/pokemon`
- **THEN** the web app MUST render Pokemon cards with image, name, order, type badges when available, filters, pagination, and loading/error/empty/success states

#### Scenario: Loading uses design system hook
- **WHEN** the Pokemon list or detail page is loading page or content data
- **THEN** the web app MUST use the existing Loading component through the `useLoading` hook

#### Scenario: Errors use design system alert hook
- **WHEN** the Pokemon list or detail page receives a fetch, BFF, or API error
- **THEN** the web app MUST show the error through the existing Alert component using the `useAlert` hook

#### Scenario: Pages are responsive and polished
- **WHEN** an authenticated user opens the Pokemon list or detail page on mobile or desktop
- **THEN** the web app MUST render an elegant responsive Tailwind-based layout using the existing design system without overlapping text, images, filters, cards, or actions

#### Scenario: List uses shared pagination hook
- **WHEN** the Pokemon list page manages pagination, filters, loading, or error state
- **THEN** it MUST use a Pokemon-specific hook backed by the existing shared `usePaginatedList` hook

#### Scenario: User opens Pokemon detail page
- **WHEN** an authenticated user selects a Pokemon from the list
- **THEN** the web app MUST navigate to the Pokemon detail page and render the normalized detail data returned by the API

#### Scenario: Detail uses feature-specific hook
- **WHEN** the Pokemon detail page fetches data for an identifier
- **THEN** it MUST use a Pokemon-specific detail hook and MUST NOT introduce a new generic `useDetail` abstraction for this change

#### Scenario: Web uses BFF route handlers
- **WHEN** the web app requests Pokemon list or detail data
- **THEN** it MUST call Next.js route handlers that read the server-side session and delegate to the API with the authenticated token

