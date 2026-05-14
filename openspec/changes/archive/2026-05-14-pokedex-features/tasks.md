## 1. Domain and data modeling

- [x] 1.1 Create the `pokedex` persistence model with `trainer` and `pokemon` relations, generated attributes, nickname, discovery state, timestamps, and soft delete
- [x] 1.2 Add or update ORM associations between `trainer`, `pokemon`, and `pokedex`
- [x] 1.3 Generate and review the database migration for the new table, indexes, and uniqueness guarantee on `trainer` + `pokemon`

## 2. API business logic and contracts

- [x] 2.1 Create `schema.py`, `business.py`, `repository.py`, `service.py`, and `route.py` for the new `pokedex` domain following the existing layered pattern
- [x] 2.2 Implement Pokedex attribute generation in the API using the existing `my-pokemon` approach as the baseline
- [x] 2.3 Implement authenticated list and detail endpoints for `pokedex` with pagination, filters, ownership scoping, and normalized responses
- [x] 2.4 Implement an authenticated explicit discovery endpoint for a trainer-owned `pokedex` entry identified by `pokemon.name`
- [x] 2.5 Extend the trainer onboarding flow to create one `pokedex` per base Pokémon, mark only the selected starter as discovered, and return `pokedex` in the onboarding payload
- [x] 2.6 Keep `my-pokemon` contracts and onboarding compatibility intact while preparing the codebase for future capture-to-pokedex synchronization without activating that flow now

## 3. Cache and query hardening

- [x] 3.1 Add Redis cache keys and cache service integration for `pokedex` list and detail responses
- [x] 3.2 Invalidate affected `pokedex` cache entries after onboarding initialization, explicit discovery, or other relevant updates
- [x] 3.3 Ensure standard queries ignore soft-deleted `pokedex` rows and avoid N+1 patterns in list and detail reads

## 4. Web BFF and feature implementation

- [x] 4.1 Add Next.js BFF route handlers for `pokedex` list, detail, and discovery, plus onboarding payload compatibility updates where needed
- [x] 4.2 Create `pokedex` web service types and hooks for list and detail flows following the existing feature structure
- [x] 4.3 Implement the protected `pokedex` list page with pagination, filters, discovery status indicators, and trainer collection cards
- [x] 4.4 Implement the protected `pokedex` detail page showing generated attributes, level, experience, HP state, nickname, discovery status, and discovery date
- [x] 4.5 Ensure the web consumes the expanded onboarding response without introducing duplicate business rules or a separate onboarding discovery UI in this change

## 5. Validation

- [x] 5.1 Add or update automated API tests for onboarding initialization, discovery endpoint behavior, ownership, filters, cache behavior, and error scenarios
- [x] 5.2 Add or update web tests for BFF handlers, `pokedex` list/detail flows, and onboarding response compatibility
- [x] 5.3 Run `make lint`, relevant API tests, `yarn lint`, and relevant `machado-web` tests for the new feature
- [x] 5.4 Validate manually the main trainer Pokedex flows, including onboarding initialization, list, detail, and protected access behavior
