## 1. Domain and data modeling

- [x] 1.1 Create `my-pokemon` and trainer-owned move persistence models with soft delete support, timestamps, and relational constraints
- [x] 1.2 Add or update ORM associations between `trainer`, `pokemon`, `my-pokemon`, and owned moves
- [x] 1.3 Generate and review the database migration for the new tables, indexes, and uniqueness guarantees

## 2. API business logic and contracts

- [x] 2.1 Create `schema.py`, `business.py`, `repository.py`, `service.py`, and `route.py` for the new `my-pokemon` domain following the existing layered pattern
- [x] 2.2 Implement `my-pokemon` creation flow with authenticated trainer ownership, nickname handling, capture timestamps, individualized attribute generation, and transactional persistence
- [x] 2.2.1 Apply nickname fallback in the API so empty or whitespace nickname persists as the selected Pokémon `name`
- [x] 2.2.2 Generate a public `name` slug from the effective nickname and ensure uniqueness per trainer for route/detail access
- [x] 2.3 Implement owned move selection with up to 4 distinct moves, persisted `pp` and `max_pp` copied from the base move, and support for Pokémon with fewer than 4 eligible moves
- [x] 2.4 Implement normalized list and detail endpoints for `my-pokemon` with pagination, filters, efficient relationship loading, and ownership scoping
- [x] 2.5 Implement the initial attribute formula in `business.py` using base Pokémon stats plus controlled random variation and expose detail by `name`
- [x] 2.6 Implement the onboarding API flow that creates `trainer` and first `my-pokemon` together when the user still has no `trainer`
- [x] 2.6.1 Expose onboarding under the `trainer` domain with a response payload that includes the created trainer and initial `my-pokemons`
- [x] 2.6.2 Apply admin-only overrides for initial `pokeballs` and `capture_rate`, keeping standard defaults for non-admin users

## 3. Cache and API integration hardening

- [x] 3.1 Add Redis cache keys and cache service integration for `my-pokemon` list and detail responses
- [x] 3.2 Invalidate affected `my-pokemon` cache entries after create or other relevant state changes
- [x] 3.2.1 Support explicit read-cache cleanup for list requests when operational debugging or forced refresh is needed
- [x] 3.3 Ensure standard queries ignore soft-deleted `my-pokemon` and owned moves while avoiding N+1 query patterns

## 4. Web BFF and feature implementation

- [x] 4.1 Add Next.js BFF route handlers for onboarding create, list, and detail `my-pokemon`, reusing the existing Pokémon list BFF as the admin source for first-Pokémon selection
- [x] 4.1.1 Route onboarding through the `trainer` web service and `/api/trainer/onboarding` BFF endpoint instead of reusing the `my-pokemon` create service
- [x] 4.2 Create `my-pokemon` web service types and hooks for list/detail flows following the existing feature structure
- [x] 4.3 Implement protected `my-pokemon` list page with pagination, filters, and roster cards showing key trainer-owned Pokémon data
- [x] 4.4 Implement protected `my-pokemon` detail page showing attributes, moves, PP state, capture date, and base Pokémon summary
- [x] 4.5 Implement the home onboarding flow so non-admin users choose one starter among Bulbasaur/Charmander/Squirtle, admin users choose freely via autocomplete with preview card, and all users can optionally inform a nickname
- [x] 4.5.1 Render onboarding directly from `home` when `user.trainer` is absent and refresh the authenticated user context after successful creation

## 5. Validation

- [x] 5.1 Add or update automated API tests for creation, ownership, move selection, PP persistence, soft delete behavior, cache behavior, and error scenarios
- [x] 5.2 Add or update web tests for BFF handlers, list/detail/create flows, and shared UI states
- [x] 5.3 Run `make lint`, relevant API tests, `yarn lint`, and relevant `machado-web` tests for the new feature
- [ ] 5.4 Validate manually the main trainer roster flows, including create, list, detail, and protected access behavior
