## 1. API Data Model

- [x] 1.1 Add `PokemonStatusEnum` with `COMPLETE` and `INCOMPLETE`.
- [x] 1.2 Create SQLAlchemy models for Pokemon, PokemonType, PokemonAbility, PokemonMove, PokemonImage, PokemonGrowthRate, PokemonHabitat, PokemonShape, PokemonEncounter, PokemonLocation, and PokemonEvolution.
- [x] 1.3 Create association/link models for Pokemon types, abilities, moves, encounters, and evolutions where required.
- [x] 1.4 Add `deleted_at` soft delete support to new tables where applicable.
- [x] 1.5 Register new models in `app/models/__init__.py`.
- [x] 1.6 Generate and review Alembic migration for all new catalog tables, indexes, foreign keys, and unique constraints.

## 2. External API Integration

- [x] 2.1 Add PokeAPI settings for base URL and request behavior using existing settings patterns.
- [x] 2.2 Create external PokeAPI schemas for list, detail, species, move, type, ability, growth-rate, encounters, and evolution-chain payloads.
- [x] 2.3 Implement async PokeAPI client methods for required external endpoints.
- [x] 2.4 Add tests for PokeAPI client URL construction, success responses, and error propagation.

## 3. Domain Structure

- [x] 3.1 Create `pokemon` domain with `repository.py`, `service.py`, `schema.py`, `business.py`, and `route.py`.
- [x] 3.2 Create fragment domains for PokemonType, PokemonAbility, PokemonMove, PokemonImage, PokemonGrowthRate, PokemonHabitat, PokemonShape, PokemonEncounter, PokemonLocation, and PokemonEvolution.
- [x] 3.3 Ensure every new repository extends `BaseRepository`.
- [x] 3.4 Ensure every new service extends `BaseService`.
- [x] 3.5 Ensure cross-domain interactions use services only and repositories are not imported outside their owning domain.
- [x] 3.6 Add pure business helpers for order extraction, four-digit image order formatting, and `external_image` generation.

## 4. Initial Synchronization

- [x] 4.1 Implement local catalog empty check in Pokemon service.
- [x] 4.2 Implement first-list-call sync using only the PokeAPI list endpoint.
- [x] 4.3 Persist minimal Pokemon records with `name`, derived `order`, generated `external_image`, and `INCOMPLETE` status.
- [x] 4.4 Make initial sync idempotent with get-or-create behavior and unique constraints.
- [x] 4.5 Add tests proving initial sync does not call detail/species/move/type/ability/encounter/evolution endpoints.

## 5. Pokemon Listing

- [x] 5.1 Implement paginated Pokemon list repository query.
- [x] 5.2 Add filters for name, order, status, and type.
- [x] 5.3 Ensure list queries ignore soft-deleted rows by default.
- [x] 5.4 Expose authenticated `GET /pokemon` route with response model.
- [x] 5.5 Add route, service, and repository tests for list pagination and filters.

## 6. Detail Enrichment

- [x] 6.1 Implement Pokemon detail lookup by id, name, or order.
- [x] 6.2 Implement enrichment flow for `INCOMPLETE` Pokemon using external detail endpoints.
- [x] 6.3 Persist enriched stats, species metadata, growth rate, habitat, shape, types, abilities, moves, encounters, evolution data, weaknesses, and strengths through the owning domain services.
- [x] 6.4 Update Pokemon status to `COMPLETE` after successful enrichment.
- [x] 6.5 Avoid external enrichment calls for already `COMPLETE` Pokemon.
- [x] 6.6 Expose authenticated `GET /pokemon/{identifier}` route with response model.
- [x] 6.7 Add tests for incomplete enrichment, complete detail without external calls, and not found behavior.

## 7. Pokemon Image Rules

- [x] 7.1 Implement `pokemon_image/business.py` to flatten nested `sprites` payloads.
- [x] 7.2 Infer `source`, `variant`, `generation`, `game`, and `media_type` from sprite paths and URLs.
- [x] 7.3 Filter invalid or empty sprite URLs.
- [x] 7.4 Select primary image using preferred official artwork, home artwork, then default sprites.
- [x] 7.5 Store all treated image entries in `PokemonImage.images` as a JSON string.
- [x] 7.6 Add focused unit tests for sprite flattening, metadata inference, primary image selection, and JSON string output.

## 8. Cache

- [x] 8.1 Add Redis cache integration for Pokemon list using key `pokemon:list:{filters}:{pagination}`.
- [x] 8.2 Add Redis cache integration for Pokemon detail using key `pokemon:detail:{identifier}`.
- [x] 8.3 Configure TTL greater than 2 hours using existing cache patterns.
- [x] 8.4 Invalidate affected list and detail cache entries after sync or enrichment.
- [x] 8.5 Add tests for cache hit, cache miss, set, and invalidation behavior.

## 9. Web BFF And Services

- [x] 9.1 Add web route handler `GET /api/pokemon` that reads server session and delegates to the API.
- [x] 9.2 Add web route handler `GET /api/pokemon/[identifier]` that reads server session and delegates to the API.
- [x] 9.3 Create Pokemon web service extending `BaseServiceAbstract`.
- [x] 9.4 Create TypeScript types for Pokemon list item, detail, type, ability, move, image, encounter, evolution, and paginated responses.
- [x] 9.5 Add tests for route handlers and service URL/query behavior.

## 10. Web UI

- [x] 10.1 Create protected `/pokemon` page.
- [x] 10.2 Create protected `/pokemon/[identifier]` page.
- [x] 10.3 Implement `usePokemonList` by consuming the existing shared `usePaginatedList` hook.
- [x] 10.4 Implement `usePokemonDetail` as a Pokemon-specific detail hook without creating a generic `useDetail` abstraction.
- [x] 10.5 Build responsive Tailwind layouts for list and detail pages using the existing design system with polished mobile and desktop behavior.
- [x] 10.6 Wire page/content loading states through the existing Loading component using `useLoading`.
- [x] 10.7 Wire fetch, BFF, and API error states through the existing Alert component using `useAlert`.
- [x] 10.8 Render Pokemon cards with image, name, order, type badges, filters, pagination, and loading/error/empty/success states.
- [x] 10.9 Render Pokemon detail with normalized API data, including images, stats, abilities, moves, evolution timeline, encounters, weaknesses, and strengths.
- [x] 10.10 Add component/page tests for list and detail states.

## 11. Verification

- [x] 11.1 Run API lint and tests.
- [x] 11.2 Run web lint, build, and tests.
- [x] 11.3 Run OpenSpec validation/status checks for the change.
- [x] 11.4 Update documentation if implementation decisions diverge from the proposal.
