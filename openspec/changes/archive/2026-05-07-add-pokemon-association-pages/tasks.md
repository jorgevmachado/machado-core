## 1. Contract Review

- [x] 1.1 Verify current machado-api contracts for `/pokemon/type`, `/pokemon/ability`, and `/pokemon/move` list/detail responses and align frontend types to the available fields.
- [x] 1.2 Review existing Pokemon list/detail web implementation, DS components, loading/alert patterns, and `usePaginatedList` usage to reuse local conventions.
- [x] 1.3 Confirm the final protected route map for `/pokemon/type`, `/pokemon/ability`, `/pokemon/move`, and their `[identifier]` detail pages using `name` as the preferred detail identifier.
- [x] 1.4 Reconcile the OpenSpec scope with the implemented machado-api contract additions for Pokemon type descriptions and move sync timeout handling.

## 2. BFF And Services

- [x] 2.1 Add typed feature service methods for Pokemon type list/detail requests.
- [x] 2.2 Add typed feature service methods for Pokemon ability list/detail requests.
- [x] 2.3 Add typed feature service methods for Pokemon move list/detail requests.
- [x] 2.4 Add Next.js BFF route handlers for `/api/pokemon/type` and `/api/pokemon/type/[identifier]` with server-session authentication.
- [x] 2.5 Add Next.js BFF route handlers for `/api/pokemon/ability` and `/api/pokemon/ability/[identifier]` with server-session authentication.
- [x] 2.6 Add Next.js BFF route handlers for `/api/pokemon/move` and `/api/pokemon/move/[identifier]` with server-session authentication.
- [x] 2.7 Add route-handler tests covering authenticated delegation, unauthenticated 401 responses, query forwarding, and API error handling.

## 3. Feature Hooks And Types

- [x] 3.1 Expand Pokemon type frontend contracts for list item, detail, filters, and paginated response types.
- [x] 3.2 Expand Pokemon ability frontend contracts for list item, detail, filters, and paginated response types.
- [x] 3.3 Expand Pokemon move frontend contracts for list item, detail, filters, and paginated response types.
- [x] 3.4 Add resource-specific list hooks for type, ability, and move pages backed by `usePaginatedList`.
- [x] 3.5 Ensure list hooks keep filters and pagination as internal page state, build query parameters only for BFF requests, and preserve API response ordering.
- [x] 3.6 Add resource-specific detail hooks for type, ability, and move pages without introducing a new generic detail abstraction.
- [x] 3.7 Keep UI and helper code that is specific to type, ability, or move inside each corresponding feature module.
- [x] 3.8 Include `PokemonType.description` and related type description/badge fields in frontend type contracts for list, detail, strengths, and weaknesses.

## 4. List Pages

- [x] 4.1 Create a reusable association card pattern with shared navigation, focus, layout, and responsive behavior plus resource-specific content slots.
- [x] 4.2 Implement the protected `/pokemon/type` list page and view under the existing protected route group with cards that use `badge_url` as the primary visual when available, use other badge fields only as clear secondary/fallback visuals, fallback to color-based text when absent, filters, traditional pagination, loading, error, empty, and success states.
- [x] 4.3 Implement the protected `/pokemon/ability` list page and view under the existing protected route group with generic cards that prioritize `effect`, `short_effect`, and `flavor_text`, using iconography, metadata, badges, typography, and traditional pagination.
- [x] 4.4 Implement the protected `/pokemon/move` list page and view under the existing protected route group with generic cards that prioritize `short_effect` and `effect`, using iconography, move metadata, stat chips, badges, typography, and traditional pagination.
- [x] 4.5 Ensure list cards navigate to the correct detail route using each resource `name` as the preferred identifier.
- [x] 4.6 Add existing breadcrumbs component/pattern to the three list pages.
- [x] 4.7 Add focused component tests for the three list views, including existing-pattern empty states and Alert-only error states.

## 5. Detail Pages

- [x] 5.1 Implement the protected `/pokemon/type/[identifier]` detail page and entity-specific view with colors, badges when available, and clickable strengths/weaknesses linking to related type details by `name`.
- [x] 5.2 Implement the protected `/pokemon/ability/[identifier]` detail page and entity-specific view with slot, hidden status, effect, short effect, flavor text, and timestamps where useful.
- [x] 5.3 Implement the protected `/pokemon/move/[identifier]` detail page and entity-specific view with type as non-relational metadata, power, accuracy, PP, target, damage class, effect chance, and effects.
- [x] 5.4 Keep technical fields such as `id`, raw `url`, and timestamps out of the primary detail UI unless they add user-facing value, and do not show `deleted_at` in normal flows.
- [x] 5.5 Ensure detail pages do not include reverse Pokemon usage sections or links.
- [x] 5.6 Add existing breadcrumbs component/pattern to the three detail pages.
- [x] 5.7 Add accessible back navigation from each detail page to its corresponding list page when it complements breadcrumbs.
- [x] 5.8 Add focused component tests for the three detail views, including loading, Alert-only error, and long-text rendering.
- [x] 5.9 Render Pokemon type descriptions in type list/detail and related type cards with readable fallback copy when descriptions are absent.

## 6. Navigation And Existing Pokemon Detail Links

- [x] 6.1 Extend authenticated navigation types to support optional child menu items without breaking existing flat menu items.
- [x] 6.2 Update the sidebar to render Pokemon as an expandable/collapsible group where the Pokemon label/link navigates to `/pokemon` and a separate arrow control expands/collapses child links for types, abilities, and moves.
- [x] 6.3 Auto-expand the Pokemon sidebar group when the active pathname is a type, ability, or move list/detail route.
- [x] 6.4 Verify existing Home, Pokemon, Pokedex, My Pokemons, and Battle sidebar behavior still works.
- [x] 6.5 Ensure existing Pokemon detail relationship links point to the implemented type, ability, and move detail routes.
- [x] 6.6 Add or update navigation tests for child rendering, expand/collapse behavior, auto-expanded child routes, and active route behavior.

## 7. UI Polish And Validation

- [x] 7.1 Check mobile and desktop layouts for the new list and detail pages to prevent overlapping text, cards, badges, filters, navigation, or actions.
- [x] 7.2 Truncate long ability and move text previews in cards and provide a clear "ver mais" path to the full detail pages.
- [x] 7.3 Verify whether Playwright is already configured in `machado-web`; if absent, add the smallest project-consistent setup for screenshot tests.
- [x] 7.4 Add Playwright auth/test setup using existing helpers when available, or isolated mocked session/token setup when absent, with deterministic network mocks.
- [x] 7.5 Add Playwright screenshot tests for success, loading, error, and empty states of the new type, ability, and move list/detail pages in representative desktop and mobile viewports.
- [x] 7.6 Add one Playwright list-card-to-detail navigation flow per entity using mocked network responses.
- [x] 7.7 Version stable Playwright baseline screenshots according to project convention, or with the tests if no convention exists.
- [x] 7.8 Make DS component improvements only if composition is insufficient, keeping changes small and non-breaking.
- [x] 7.9 Run `yarn lint` in `machado-web` after fixing current implementation formatting errors.
- [x] 7.10 Re-run `yarn build` in `machado-web` after the latest implementation updates.
- [x] 7.11 Run `yarn test` in `machado-web`.
- [x] 7.12 Re-run the Playwright visual/screenshot test command in `machado-web` after the latest implementation updates.
- [x] 7.13 Document any confirmed API contract gaps or intentionally deferred enhancements in the implementation notes.

## 8. Machado-API Contract Support

- [x] 8.1 Add `PokemonType.description` to the model and in-flight Pokemon catalog migration.
- [x] 8.2 Expose `description` in `PokemonTypeSchema` and `PokemonTypeDamageSchema`.
- [x] 8.3 Extend external PokeAPI type schemas with `move_damage_class` and add move damage class description parsing.
- [x] 8.4 Add `PokeApiClient.get_move_damage_class_by_url`.
- [x] 8.5 Update `PokemonTypeService` to populate missing descriptions from move damage class descriptions and keep empty fallback behavior when external data is absent.
- [x] 8.6 Update incomplete type enrichment to fill missing descriptions before damage relation completion.
- [x] 8.7 Update `PokemonMoveService.sync_from_resources` to skip individual `httpx.TimeoutException` failures and log skipped move order/url.
- [x] 8.8 Add/update API unit tests for Pokemon type description branches, move timeout handling, PokeAPI move damage class parsing, and full coverage of the touched services.
