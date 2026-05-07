## Context

The existing Pokemon web feature has a protected Pokemon list and detail flow in `machado-web`, backed by Next.js route handlers that read the server-side session and delegate to authenticated machado-api endpoints. The API registers authenticated subroutes for `/pokemon/type`, `/pokemon/ability`, and `/pokemon/move`, each with list and detail endpoints, pagination parameters, and normalized schemas. Recent implementation work also enriches the `PokemonType` contract with descriptions sourced from PokeAPI move damage class data and makes `PokemonMove` synchronization tolerant of per-resource timeouts.

The requested change primarily implements the relationship-data experiences in `machado-web`, with tightly scoped `machado-api` contract/support changes where the new UI depends on richer normalized data.

## Goals / Non-Goals

**Goals:**

- Provide protected list and detail pages for Pokemon types, abilities, and moves.
- Place new pages under the existing protected App Router group and reuse the current authentication guard.
- Reuse the current Pokemon web patterns: App Router pages, BFF route handlers, `BaseServiceAbstract` feature services, typed feature contracts, `usePaginatedList` for list state, and feature-specific detail hooks.
- Reuse existing design system components for cards, filters, pagination, loading, alerts, badges, buttons, and text.
- Design detail views specifically for each entity's data shape while staying consistent with the existing design system.
- Use one reusable association card pattern for list pages, with resource-specific content slots.
- Use traditional pagination through the existing pagination component rather than incremental loading.
- Keep filters and pagination in internal page state; build query parameters only for BFF requests.
- Use API-provided default ordering for initial list data and avoid client-side reordering.
- Keep detail pages focused on user-facing fields; hide technical identifiers, raw URLs, and timestamps from the main layout unless useful as secondary metadata.
- Render `PokemonType` strengths and weaknesses as interactive links to related type detail pages using `name` as the preferred identifier.
- Surface `PokemonType.description` in list, detail, and related type cards when available.
- Treat `PokemonMove.type` as plain move metadata, not as a relation to `PokemonType`.
- Exclude reverse usage sections such as Pokemon that use a given type, ability, or move.
- Use the existing application navigation label convention for new sidebar children.
- Use the existing breadcrumbs component/pattern on list and detail pages.
- Use the existing Alert pattern for API/BFF errors without adding a dedicated retry button.
- Use the existing empty-state pattern for no-result lists without adding new custom actions unless the existing pattern already includes them.
- Add Playwright screenshot checks for desktop and mobile layout validation across success, loading, error, and empty states, following any existing project setup.
- For protected-page Playwright tests, prefer any existing project auth helper; if none exists, create an isolated mocked session/token setup in the browser context and mock network responses in the tests.
- Include a minimal Playwright navigation flow from list card to detail page for each entity in addition to deterministic screenshots.
- Keep entity-specific components inside each feature module; extract shared local UI only when it is truly generic.
- Version Playwright baseline screenshots when adding screenshot tests, unless an existing project convention says otherwise.
- Improve DS components only when composition cannot satisfy the requirement and the change can remain non-breaking.
- Add Pokemon child navigation entries for type, ability, and move listing pages.
- Keep route and link structure consistent with links that already exist in Pokemon detail cards: `/pokemon/type/:identifier`, `/pokemon/ability/:identifier`, and `/pokemon/move/:identifier`.
- Add narrowly scoped API support for Pokemon type descriptions and move sync timeout tolerance, with unit coverage.

**Non-Goals:**

- No unrelated machado-api changes beyond Pokemon type description enrichment and move sync timeout tolerance.
- No new tables, soft-delete changes, or synchronization jobs.
- No authentication changes.
- No global frontend architecture refactor.
- No new generic detail abstraction unless an equivalent pattern already exists before implementation.
- No evolution timeline on `PokemonType`, `PokemonAbility`, or `PokemonMove` detail pages.
- No incremental loading for association list pagination.
- No browser URL/query-string synchronization for list filters or page state.
- No frontend-forced default ordering beyond the existing API behavior.
- No reverse Pokemon usage sections for type, ability, or move details.

## Decisions

1. Keep relationship pages under `/pokemon`.

   The current Pokemon detail UI already links related types and abilities with `/pokemon/type/{name}` and `/pokemon/ability/{name}`. The implementation should complete that route family with list pages at `/pokemon/type`, `/pokemon/ability`, and `/pokemon/move`, plus detail pages at `/pokemon/type/[identifier]`, `/pokemon/ability/[identifier]`, and `/pokemon/move/[identifier]`. List-card and relationship links should prefer `name` as the identifier, while keeping `id` compatible only if the existing API already supports it.

   Alternative considered: top-level routes such as `/pokemon-types`. This would avoid nested paths but would conflict with existing links and the requested sidebar hierarchy under Pokemon.

2. Add BFF route handlers for each relationship resource.

   The web app should call local Next.js route handlers such as `/api/pokemon/type` and `/api/pokemon/type/[identifier]`. Each handler reads the server session, returns 401 when unauthenticated, instantiates a typed service with the bearer token, and delegates to the matching machado-api endpoint.

   Alternative considered: direct client calls to machado-api. This would expose auth/session concerns to client code and diverge from the existing Pokemon list/detail implementation.

3. Keep pages inside the protected route group.

   All list and detail pages should live under the existing protected App Router group, matching `/pokemon` behavior. Authentication remains owned by the current protected layout and BFF server-session checks.

   Alternative considered: page-level auth checks in each new page. That would duplicate the existing protected layout behavior.

4. Use resource-specific feature modules instead of a generic association framework.

   Types, abilities, and moves share list/detail mechanics but have different fields and presentation needs. The implementation should use small resource-specific services, hooks, views, types, and detail layouts. Entity-specific components should live inside each corresponding feature module. Shared local UI should be extracted only when it is genuinely generic and avoids meaningful duplication.

   Alternative considered: one generic association page renderer driven by configuration. That would introduce a new abstraction before the UI requirements prove it is needed.

5. Use existing breadcrumbs for page orientation.

   New list and detail pages should use the existing breadcrumbs component/pattern to match the rest of the system. Detail pages may also include a local back action when it improves flow, but breadcrumbs are the baseline navigation pattern.

   Alternative considered: back link only. That would skip an existing system-level orientation pattern.

6. Use a reusable generic card pattern with resource-specific content.

   List cards should share a consistent structure for navigation, focus states, metadata, and responsive sizing. `PokemonType` should use `badge_url` as the main visual when available and avoid duplicating the name when the image already displays it. Other PokemonType badge fields should be secondary or fallback visuals only when they add clarity without clutter. When `badge_url` is absent, the type card should fallback to a textual presentation using `background_color` and `text_color` when available. `PokemonAbility` should prioritize `effect`, `short_effect`, and `flavor_text`, with slot, hidden status, and order as secondary metadata. `PokemonMove` should prioritize `short_effect` and `effect` in the card body, with type, damage class, power, accuracy, and PP as secondary metadata. Long text in ability and move cards should be truncated to keep grid heights stable, with a clear "ver mais" link to the full detail page.

   Alternative considered: fully separate card components for each resource. That would make the UI easier to tailor but would duplicate interaction, layout, and state behavior across three very similar list pages.

7. Treat existing API contracts as the source of truth.

   The frontend types should align to current API schemas: `PokemonTypeSchema`, `PokemonAbilitySchema`, and `PokemonMoveSchema`. Optional fields should be modeled where the API contract allows nullable values. Existing automatic synchronization remains API-owned behavior that the web app consumes through those endpoints. The `PokemonType` contract now includes `description` on full type records and damage relation items so the new UI can render meaningful type copy without hardcoding it in the frontend.

   Alternative considered: designing richer frontend-only contracts. That would hide backend gaps and make implementation dependent on data that does not exist in the current scope.

8. Keep technical fields out of the primary detail UI.

   Fields such as `id`, raw `url`, `created_at`, and `updated_at` may remain in TypeScript contracts but should not dominate the user-facing detail layout. They can be omitted or placed as subtle secondary metadata if useful. `deleted_at` should not be shown in normal flows because the existing API behavior should exclude soft-deleted records by default.

9. Link related PokemonType damage relations.

   `strengths` and `weaknesses` in `PokemonType` detail should be interactive badges or list items that navigate to the related type detail page. Links should prefer the related type `name`, matching the route identifier decision for detail pages.

   Alternative considered: static visual badges only. That would show the relationship but would not support the expected browse flow across type relationships.

10. Keep PokemonMove type non-relational.

   `PokemonMove.type` should be displayed as metadata only. It must not link to `/pokemon/type/{name}` because it is not part of the `PokemonType` relationship model for this feature.

   Alternative considered: linking move type to PokemonType by matching text. That would create a false relationship in the UI.

11. Keep reverse Pokemon usage out of scope.

   Detail pages should not add sections such as "Pokemon that use this ability" or "Pokemon that use this move". Those views require additional relationship data or endpoint support and are outside the web-only scope.

   Alternative considered: deriving reverse usage in the frontend. That would create business/data behavior in `machado-web` and likely require unavailable data.

12. Evolve navigation minimally to support collapsible Pokemon children.

   `MenuItem` is currently flat. To render links as children of Pokemon, add an optional `children` property and update the sidebar rendering so Pokemon can expand/collapse its child links. The Pokemon label/link should still navigate directly to `/pokemon`; a separate arrow control should expand or collapse child links. The group should auto-expand when the active pathname is a type, ability, or move child route. The parent should be marked active for child routes, child active state should remain clear, collapsed-sidebar mode should remain accessible through titles/labels, and new child labels should follow the application's existing navigation label convention.

   Alternative considered: adding three more top-level menu items or showing child links always expanded. Those options would either miss the requested hierarchy or skip the requested open/close behavior.

13. Enrich PokemonType descriptions in the API.

   `PokemonTypeService` should read `move_damage_class.url` from the external type payload, fetch the move damage class by URL, select the English `description`, persist it on `PokemonType.description`, and expose it through both `PokemonTypeSchema` and `PokemonTypeDamageSchema`. Existing descriptions should be preserved when already present, and missing external description data should degrade to an empty string.

   Alternative considered: hardcoding type descriptions in `machado-web`. That would duplicate domain data in the UI and diverge from the API-owned synchronization model.

14. Skip timed-out PokemonMove resources during sync.

   `PokemonMoveService.sync_from_resources` should catch `httpx.TimeoutException` around individual move creation, log the skipped move order/url, and continue syncing the remaining moves. This keeps partial sync failures from aborting an entire Pokemon sync operation.

   Alternative considered: letting the timeout bubble. That would preserve strict failure semantics but makes a single slow PokeAPI move block unrelated sync progress.

## Risks / Trade-offs

- API response mismatch or missing fields -> Validate current route contracts during implementation and adjust UI to available normalized fields without changing the backend.
- PokemonType description data depends on PokeAPI move damage class payloads -> Keep the field nullable/empty-safe in UI and test fallback behavior.
- Updating an in-flight catalog migration can be risky if already applied elsewhere -> Confirm environment state before running migrations in shared databases.
- Move sync timeout tolerance can leave some moves incomplete until a later sync -> Log skipped move order/url and rely on existing retry/sync behavior.
- Collapsible nested sidebar change affects all authenticated navigation -> Keep `children` optional, preserve existing top-level item rendering, and test expand/collapse and active states for existing links.
- Three similar feature flows can create duplication -> Start with explicit resource modules, then extract only small local helpers if the implementation repeats non-trivial logic.
- Large lists or long effects/flavor text can break layouts -> Use constrained card/detail layouts, text wrapping, card truncation, "ver mais" detail links, and responsive checks for mobile and desktop.
- Existing DS components may not cover every card/detail need -> Prefer composition of existing DS primitives; only make non-breaking DS improvements when needed by multiple resources.
- Error recovery is passive -> Display clear Alert messages and rely on existing navigation/page refresh behavior instead of adding dedicated retry controls.
- Playwright may not be configured yet -> During implementation, first verify the existing test setup; if absent, add the smallest project-consistent Playwright setup needed for screenshot checks.
- Auth in visual tests can become brittle -> Reuse existing auth helpers when present, otherwise isolate a mocked session/token setup and avoid real backend/session dependencies.
- Full end-to-end flows can become broad -> Limit Playwright navigation coverage to one deterministic list-to-detail path per entity, with network mocks.
- Snapshot maintenance can become noisy -> Use deterministic mocked data and version only stable baseline screenshots.

## Migration Plan

The web portion ships as frontend routes, BFF route handlers, feature modules, navigation updates, and tests. The API portion adds `pokemon_types.description` to the in-flight catalog migration/model/schema and can be rolled back by reverting that contract addition before applying the migration in shared environments. Rollback of the web portion is removing the frontend additions and sidebar entries.

## Open Questions

<!-- None. Current decisions are captured in proposal, specs, and tasks. -->
