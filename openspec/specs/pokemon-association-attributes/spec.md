# pokemon-association-attributes Specification

## Purpose
Protected web list/detail experiences for Pokemon relationship attributes, with scoped API support for type descriptions and move sync resilience.

## Requirements
### Requirement: PokemonType list page
The system SHALL provide a protected web list page for `PokemonType` using the existing API pagination and filter contract.

#### Scenario: User accesses PokemonType list
- **WHEN** an authenticated user opens `/pokemon/type`
- **THEN** the web app MUST render a page with filters and a generic card-based `PokemonType` listing with PokemonType-specific content

#### Scenario: PokemonType card uses badge image
- **WHEN** a `PokemonType` item includes `badge_url`
- **THEN** the card MUST use the badge image as the primary visual and MUST NOT repeat the type name when the badge image already contains it

#### Scenario: PokemonType card avoids badge clutter
- **WHEN** a `PokemonType` item includes multiple badge image fields
- **THEN** the card MUST treat `badge_url` as the primary visual and MUST use other badge fields only as secondary or fallback visuals when they add clarity without cluttering the UI

#### Scenario: PokemonType card falls back without badge image
- **WHEN** a `PokemonType` item does not include `badge_url`
- **THEN** the card MUST render a textual fallback using the type name and available `background_color` and `text_color` values instead of showing a broken or missing image state

#### Scenario: PokemonType card displays description
- **WHEN** a `PokemonType` item includes `description`
- **THEN** the card MUST display that description as the preview copy and MUST use readable fallback copy when the description is absent or empty

#### Scenario: PokemonType list supports pagination and filters
- **WHEN** an authenticated user changes pagination or applies supported filters on the `PokemonType` list page
- **THEN** the web app MUST use the existing pagination component, request the matching BFF route with normalized query parameters, and render the paginated API response

#### Scenario: PokemonType list handles request states
- **WHEN** the `PokemonType` list page is loading, receives an error, receives no results, or receives results
- **THEN** the page MUST render the corresponding loading, error, empty, or success state using existing web patterns

#### Scenario: PokemonType empty state follows existing pattern
- **WHEN** the `PokemonType` list page receives no results
- **THEN** the page MUST render the existing system empty-state pattern and MUST NOT introduce custom empty-state actions unless already provided by that pattern

#### Scenario: PokemonType list error uses Alert only
- **WHEN** the `PokemonType` list page receives an API or BFF error
- **THEN** the page MUST display the error through the existing Alert pattern and MUST NOT add a dedicated retry action

### Requirement: PokemonType detail page
The system SHALL provide a protected web detail page for `PokemonType` using the existing API detail contract.

#### Scenario: User opens PokemonType detail from list card
- **WHEN** an authenticated user clicks a `PokemonType` card from the list page
- **THEN** the web app MUST navigate to `/pokemon/type/{name}` using `name` as the preferred identifier and render a rich normalized `PokemonType` detail page with an entity-specific layout consistent with the design system

#### Scenario: PokemonType detail shows normalized data
- **WHEN** the `PokemonType` detail response contains type identity, order, colors, badges, description, strengths, weaknesses, and timestamps
- **THEN** the page MUST display the available information in a consistent, responsive, and readable layout

#### Scenario: PokemonType strengths and weaknesses are navigable
- **WHEN** the `PokemonType` detail page displays strengths or weaknesses
- **THEN** each related type MUST be a clickable link to that related `PokemonType` detail page using `name` as the preferred route identifier

#### Scenario: PokemonType related cards show descriptions
- **WHEN** a `PokemonType` strength or weakness includes `description`
- **THEN** the related type card MUST display that description and MUST fall back to concise explanatory copy when absent

#### Scenario: PokemonType detail keeps technical fields secondary
- **WHEN** the `PokemonType` detail response contains technical fields such as `id`, raw `url`, `created_at`, `updated_at`, or `deleted_at`
- **THEN** the page MUST hide them from the primary UI unless they provide user-facing value, and MUST NOT show `deleted_at` in normal flows

### Requirement: PokemonAbility list page
The system SHALL provide a protected web list page for `PokemonAbility` using the existing API pagination and filter contract.

#### Scenario: User accesses PokemonAbility list
- **WHEN** an authenticated user opens `/pokemon/ability`
- **THEN** the web app MUST render a page with filters and a generic card-based `PokemonAbility` listing with PokemonAbility-specific content

#### Scenario: PokemonAbility card compensates for missing image
- **WHEN** a `PokemonAbility` item has no image field
- **THEN** the card MUST prioritize `effect`, `short_effect`, and `flavor_text`, using representative iconography, slot, hidden status, order, badges, or typography as supporting visual hierarchy without relying on an image

#### Scenario: PokemonAbility list supports pagination and filters
- **WHEN** an authenticated user changes pagination or applies supported filters on the `PokemonAbility` list page
- **THEN** the web app MUST use the existing pagination component, request the matching BFF route with normalized query parameters, and render the paginated API response

#### Scenario: PokemonAbility list handles request states
- **WHEN** the `PokemonAbility` list page is loading, receives an error, receives no results, or receives results
- **THEN** the page MUST render the corresponding loading, error, empty, or success state using existing web patterns

#### Scenario: PokemonAbility empty state follows existing pattern
- **WHEN** the `PokemonAbility` list page receives no results
- **THEN** the page MUST render the existing system empty-state pattern and MUST NOT introduce custom empty-state actions unless already provided by that pattern

#### Scenario: PokemonAbility list error uses Alert only
- **WHEN** the `PokemonAbility` list page receives an API or BFF error
- **THEN** the page MUST display the error through the existing Alert pattern and MUST NOT add a dedicated retry action

### Requirement: PokemonAbility detail page
The system SHALL provide a protected web detail page for `PokemonAbility` using the existing API detail contract.

#### Scenario: User opens PokemonAbility detail from list card
- **WHEN** an authenticated user clicks a `PokemonAbility` card from the list page
- **THEN** the web app MUST navigate to `/pokemon/ability/{name}` using `name` as the preferred identifier and render a rich normalized `PokemonAbility` detail page with an entity-specific layout consistent with the design system

#### Scenario: PokemonAbility detail shows normalized data
- **WHEN** the `PokemonAbility` detail response contains ability identity, order, slot, hidden status, effect, short effect, flavor text, and timestamps
- **THEN** the page MUST display the available information in a consistent, responsive, and readable layout

#### Scenario: PokemonAbility detail keeps technical fields secondary
- **WHEN** the `PokemonAbility` detail response contains technical fields such as `id`, raw `url`, `created_at`, `updated_at`, or `deleted_at`
- **THEN** the page MUST hide them from the primary UI unless they provide user-facing value, and MUST NOT show `deleted_at` in normal flows

### Requirement: PokemonMove list page
The system SHALL provide a protected web list page for `PokemonMove` using the existing API pagination and filter contract.

#### Scenario: User accesses PokemonMove list
- **WHEN** an authenticated user opens `/pokemon/move`
- **THEN** the web app MUST render a page with filters and a generic card-based `PokemonMove` listing with PokemonMove-specific content

#### Scenario: PokemonMove card compensates for missing image
- **WHEN** a `PokemonMove` item has no image field
- **THEN** the card MUST prioritize `short_effect` and `effect`, using representative iconography, move type/damage metadata, stat chips, badges, or typography as supporting visual hierarchy without relying on an image

#### Scenario: PokemonMove list supports pagination and filters
- **WHEN** an authenticated user changes pagination or applies supported filters on the `PokemonMove` list page
- **THEN** the web app MUST use the existing pagination component, request the matching BFF route with normalized query parameters, and render the paginated API response

#### Scenario: PokemonMove list handles request states
- **WHEN** the `PokemonMove` list page is loading, receives an error, receives no results, or receives results
- **THEN** the page MUST render the corresponding loading, error, empty, or success state using existing web patterns

#### Scenario: PokemonMove empty state follows existing pattern
- **WHEN** the `PokemonMove` list page receives no results
- **THEN** the page MUST render the existing system empty-state pattern and MUST NOT introduce custom empty-state actions unless already provided by that pattern

#### Scenario: PokemonMove list error uses Alert only
- **WHEN** the `PokemonMove` list page receives an API or BFF error
- **THEN** the page MUST display the error through the existing Alert pattern and MUST NOT add a dedicated retry action

#### Scenario: PokemonAbility and PokemonMove cards support complete detail access
- **WHEN** a `PokemonAbility` or `PokemonMove` card contains long `effect`, `short_effect`, or `flavor_text`
- **THEN** the page MUST truncate the preview to preserve a stable card grid and provide a clear "ver mais" path to the complete detail page

### Requirement: PokemonMove detail page
The system SHALL provide a protected web detail page for `PokemonMove` using the existing API detail contract.

#### Scenario: User opens PokemonMove detail from list card
- **WHEN** an authenticated user clicks a `PokemonMove` card from the list page
- **THEN** the web app MUST navigate to `/pokemon/move/{name}` using `name` as the preferred identifier and render a rich normalized `PokemonMove` detail page with an entity-specific layout consistent with the design system

#### Scenario: PokemonMove detail shows normalized data
- **WHEN** the `PokemonMove` detail response contains move identity, order, type, power, accuracy, PP, target, damage class, effect chance, effect, short effect, and timestamps
- **THEN** the page MUST display the available information in a consistent, responsive, and readable layout

#### Scenario: PokemonMove type is not linked to PokemonType
- **WHEN** the `PokemonMove` list or detail page displays the move `type` field
- **THEN** the page MUST treat it as move metadata and MUST NOT link it to a `PokemonType` detail page

#### Scenario: PokemonMove detail keeps technical fields secondary
- **WHEN** the `PokemonMove` detail response contains technical fields such as `id`, raw `url`, `created_at`, `updated_at`, or `deleted_at`
- **THEN** the page MUST hide them from the primary UI unless they provide user-facing value, and MUST NOT show `deleted_at` in normal flows

### Requirement: Pokemon relationship attribute BFF consumption
The system SHALL consume existing machado-api list and detail endpoints for `PokemonType`, `PokemonAbility`, and `PokemonMove` through Next.js BFF route handlers.

#### Scenario: BFF delegates authenticated list requests
- **WHEN** the web app requests `/api/pokemon/type`, `/api/pokemon/ability`, or `/api/pokemon/move` with a valid server-side session
- **THEN** the route handler MUST delegate to the corresponding existing machado-api pagination endpoint with the authenticated bearer token

#### Scenario: BFF delegates authenticated detail requests
- **WHEN** the web app requests `/api/pokemon/type/{identifier}`, `/api/pokemon/ability/{identifier}`, or `/api/pokemon/move/{identifier}` with a valid server-side session
- **THEN** the route handler MUST delegate to the corresponding existing machado-api detail endpoint with the authenticated bearer token

#### Scenario: BFF rejects unauthenticated requests
- **WHEN** a relationship attribute BFF route receives a request without a valid server-side session
- **THEN** it MUST return a 401 JSON response and MUST NOT call machado-api

#### Scenario: BFF keeps API as business-rule owner
- **WHEN** a relationship attribute page needs data synchronization, filtering, detail enrichment, or validation behavior
- **THEN** the web app MUST rely on the existing machado-api behavior and MUST NOT implement business rules in `machado-web`

#### Scenario: Web consumes existing automatic synchronization
- **WHEN** machado-api performs automatic synchronization for `PokemonType`, `PokemonAbility`, or `PokemonMove`
- **THEN** the web app MUST consume the resulting API response and MUST NOT implement a separate synchronization flow

### Requirement: PokemonType API description enrichment
The system SHALL enrich `PokemonType` API contracts with descriptions that can be consumed by the association pages.

#### Scenario: PokemonType schema exposes description
- **WHEN** the API returns a `PokemonType` list or detail record
- **THEN** the response schema MUST include `description` when available

#### Scenario: PokemonType damage schema exposes description
- **WHEN** the API returns `PokemonType` strengths or weaknesses
- **THEN** each related type schema MUST include `description` when available so the web UI can render related type preview copy

#### Scenario: PokemonType sync reads move damage class description
- **WHEN** `PokemonTypeService` creates a type from an external type payload with `move_damage_class.url`
- **THEN** it MUST fetch that move damage class payload, select the English `description`, and persist it on `PokemonType.description`

#### Scenario: PokemonType description degrades safely
- **WHEN** the external type has no move damage class URL or the move damage class payload cannot provide a description
- **THEN** `PokemonTypeService` MUST persist an empty description instead of failing type creation

#### Scenario: PokemonType incomplete enrichment fills missing description
- **WHEN** `PokemonTypeService.find_one` loads an incomplete type with an empty description
- **THEN** it MUST attempt to populate the description before completing damage relations

### Requirement: PokemonMove sync timeout tolerance
The system SHALL keep Pokemon move synchronization resilient when individual external move requests time out.

#### Scenario: Timed-out move resource is skipped
- **WHEN** `PokemonMoveService.sync_from_resources` encounters `httpx.TimeoutException` while syncing one move resource
- **THEN** it MUST log the skipped move order and URL, skip that resource, and continue syncing remaining move resources

#### Scenario: Move sync preserves successful resources
- **WHEN** some move resources sync successfully and another move resource times out
- **THEN** the service MUST return the successfully synced moves without raising the timeout for the whole batch

### Requirement: Pokemon relationship attribute sidebar navigation
The system SHALL present Pokemon type, ability, and move list links as collapsible child links of Pokemon in the authenticated sidebar.

#### Scenario: Sidebar expands Pokemon child links
- **WHEN** an authenticated user accesses any protected page with the sidebar expanded
- **THEN** the sidebar MUST allow the Pokemon item to expand and show links for Pokemon types, Pokemon abilities, and Pokemon moves as children of Pokemon with representative icons and labels that follow the application's existing navigation convention

#### Scenario: Pokemon parent link remains navigable
- **WHEN** an authenticated user clicks the Pokemon label or main Pokemon link in the sidebar
- **THEN** the sidebar MUST navigate to `/pokemon` and MUST NOT treat that click as only an expand/collapse action

#### Scenario: Pokemon arrow controls child visibility
- **WHEN** an authenticated user clicks the Pokemon sidebar arrow control
- **THEN** the sidebar MUST expand or collapse the Pokemon child links without navigating away from the current page

#### Scenario: Sidebar collapses Pokemon child links
- **WHEN** an authenticated user collapses the Pokemon group in the expanded sidebar
- **THEN** the sidebar MUST hide the Pokemon type, ability, and move child links without breaking navigation to the main Pokemon page

#### Scenario: Sidebar active state includes child routes
- **WHEN** an authenticated user opens a Pokemon type, ability, or move list/detail route
- **THEN** the sidebar MUST indicate the matching child route and the Pokemon parent according to the existing navigation styling conventions, expanding the Pokemon group when needed to reveal the active child

#### Scenario: Pokemon group auto-expands on child route
- **WHEN** an authenticated user lands directly on `/pokemon/type/{identifier}`, `/pokemon/ability/{identifier}`, or `/pokemon/move/{identifier}`
- **THEN** the sidebar MUST initialize the Pokemon group as expanded so the active child section is visible

#### Scenario: Existing sidebar navigation remains stable
- **WHEN** an authenticated user navigates through existing top-level sidebar links
- **THEN** Home, Pokemon, Pokedex, My Pokemons, and Battle MUST continue to render and navigate as before

### Requirement: Pokemon relationship attribute web architecture
The system SHALL implement Pokemon relationship attribute pages in `machado-web` without creating a new architecture or duplicating non-trivial logic.

#### Scenario: Pages use protected route group
- **WHEN** `PokemonType`, `PokemonAbility`, or `PokemonMove` list/detail pages are added
- **THEN** they MUST live under the existing protected App Router group and require the same authentication behavior as `/pokemon`

#### Scenario: Lists use existing pagination hook pattern
- **WHEN** a relationship attribute list page manages pagination, filters, loading, error, reload, or empty state
- **THEN** it MUST use a resource-specific hook backed by the existing shared `usePaginatedList` hook

#### Scenario: List state remains internal
- **WHEN** a relationship attribute list page applies filters or changes pagination
- **THEN** the page MUST keep filter and pagination state internally and MUST NOT require browser URL/query-string synchronization

#### Scenario: Lists respect API default ordering
- **WHEN** a relationship attribute list page loads initial results
- **THEN** the page MUST render the API response order and MUST NOT apply additional client-side default ordering

#### Scenario: Lists do not use incremental loading
- **WHEN** a relationship attribute list page renders paginated results
- **THEN** it MUST use traditional pagination and MUST NOT implement incremental loading as the list pagination mechanism

#### Scenario: Details use resource-specific hooks
- **WHEN** a relationship attribute detail page fetches data for an identifier
- **THEN** it MUST use a resource-specific detail hook and MUST NOT introduce a new generic detail abstraction for this change

#### Scenario: UI reuses existing components
- **WHEN** relationship attribute pages render cards, filters, pagination, loading, error, empty, or success states
- **THEN** they MUST reuse existing design-system and Pokemon UI patterns where applicable, improving components only through non-breaking changes when needed

#### Scenario: Entity-specific UI stays inside each feature
- **WHEN** UI, layout, or helper code is specific to `PokemonType`, `PokemonAbility`, or `PokemonMove`
- **THEN** it MUST be created inside the corresponding feature module instead of a broad shared abstraction

#### Scenario: DS changes are minimal
- **WHEN** existing design-system components cannot satisfy the required UI through composition
- **THEN** the implementation MAY make small non-breaking DS improvements, but MUST avoid broad DS refactors for this change

#### Scenario: List cards share a generic pattern
- **WHEN** relationship attribute list pages render cards for `PokemonType`, `PokemonAbility`, or `PokemonMove`
- **THEN** the cards MUST share a generic interaction and layout pattern while allowing small resource-specific content variations

#### Scenario: Detail layout follows entity shape
- **WHEN** a relationship attribute detail page renders `PokemonType`, `PokemonAbility`, or `PokemonMove`
- **THEN** it MUST use a layout tailored to that entity's fields while preserving design-system consistency

#### Scenario: Detail error uses Alert only
- **WHEN** a relationship attribute detail page receives an API or BFF error
- **THEN** the page MUST display the error through the existing Alert pattern and MUST NOT add a dedicated retry action

#### Scenario: Pages use existing breadcrumbs
- **WHEN** a relationship attribute list or detail page renders
- **THEN** it MUST use the existing breadcrumbs component or established breadcrumbs pattern for page orientation

#### Scenario: Pages have visual screenshot coverage
- **WHEN** `PokemonType`, `PokemonAbility`, or `PokemonMove` list/detail pages are implemented
- **THEN** they MUST have Playwright screenshot checks for success, loading, error, and empty states in representative desktop and mobile viewports, following the project's existing visual test setup or the smallest project-consistent setup if none exists

#### Scenario: Playwright baselines are stable and versioned
- **WHEN** Playwright screenshot tests are added for this change
- **THEN** they MUST use deterministic mocked data and version stable baseline screenshots according to project convention, or with the tests if no convention exists

#### Scenario: Playwright tests avoid backend dependency
- **WHEN** Playwright screenshot tests run for protected relationship attribute pages
- **THEN** they MUST use existing auth helpers when available or an isolated mocked session/token setup when absent, and MUST mock network responses for deterministic success, loading, error, and empty states

#### Scenario: Playwright tests cover list to detail navigation
- **WHEN** Playwright tests run for `PokemonType`, `PokemonAbility`, and `PokemonMove`
- **THEN** they MUST cover at least one deterministic list-card-to-detail navigation flow per entity using mocked network responses

#### Scenario: Backend changes remain scoped
- **WHEN** this change is implemented
- **THEN** machado-api changes MUST remain limited to Pokemon type description contract support, PokeAPI move damage class parsing, move sync timeout tolerance, and corresponding tests

#### Scenario: Evolution timeline remains out of scope
- **WHEN** a `PokemonType`, `PokemonAbility`, or `PokemonMove` detail page is rendered
- **THEN** the page MUST NOT include an evolution timeline as part of this change

#### Scenario: Reverse Pokemon usage remains out of scope
- **WHEN** a `PokemonType`, `PokemonAbility`, or `PokemonMove` detail page is rendered
- **THEN** the page MUST NOT include sections or links for Pokemon that use the type, ability, or move as part of this change
