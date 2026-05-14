## 1. Data modeling and persistence

- [x] 1.1 Create the persistence structures for trainer-known encounters, trainer main party slots, and exploration events with soft delete, timestamps, and relational integrity
- [x] 1.2 Add or update ORM associations between `trainer`, `pokemon_encounter`, `my_pokemon`, and the new exploration-related entities
- [x] 1.3 Generate and review the database migration covering uniqueness for the active encounter, party slot integrity, and duplicate-prevention for trainer-owned encounters

## 2. Onboarding and trainer exploration API

- [x] 2.1 Implement API domain modules for trainer exploration, including schemas, business rules, repository queries, services, and authenticated routes
- [x] 2.2 Extend onboarding so the trainer receives the starter-derived known encounters and one deterministic active encounter in the same transactional flow
- [x] 2.3 Implement authenticated list and selection endpoints for trainer-owned encounters with ownership checks and normalized responses
- [x] 2.4 Implement the authenticated walk endpoint that validates the active known encounter, generates one normalized random event, persists it, and updates trainer pokeballs when applicable
- [x] 2.5 Implement authenticated main-party management with validation for ownership, active state, slot ordering, and maximum size of six `my-pokemon`

## 3. Home aggregation, cache, and query hardening

- [x] 3.1 Implement the authenticated trainer Home summary endpoint returning trainer summary, active encounter, main party, and the three latest discovered `pokedex` entries
- [x] 3.2 Add cache keys and cache-service integration for trainer Home, trainer encounters, and trainer party reads using the project cache pattern
- [x] 3.3 Invalidate affected cache entries after onboarding initialization, encounter selection, walking, and party updates
- [x] 3.4 Ensure exploration, Home, and party queries avoid N+1 patterns and exclude soft-deleted records from standard reads

## 4. Web BFF and protected experience

- [x] 4.1 Add Next.js BFF route handlers for trainer encounter list, active encounter selection, walking, party selection, and trainer Home summary
- [x] 4.2 Create trainer exploration service types and hooks for encounters, walking events, party management, and Home payload consumption following the existing feature structure
- [x] 4.3 Update the protected Home page to render trainer summary, active encounter, latest discovered `pokedex` entries, and the current main party using existing design-system patterns
- [x] 4.4 Implement the protected trainer exploration interactions needed in this change, including selecting an active encounter, walking, and managing the main party without embedding business logic in the frontend

## 5. Validation

- [x] 5.1 Add or update automated API tests for onboarding encounter initialization, encounter ownership and selection, walking outcomes, party constraints, Home payload, cache invalidation, and error scenarios
- [x] 5.2 Add or update web tests for BFF handlers, Home rendering, encounter selection, walking event handling, and party management flows
- [x] 5.3 Run `make lint`, relevant API tests, `yarn lint`, and relevant `machado-web` tests for the new feature
- [ ] 5.4 Validate manually the main trainer exploration flows, including onboarding initialization, Home summary rendering, encounter selection, walking, and main-party updates
