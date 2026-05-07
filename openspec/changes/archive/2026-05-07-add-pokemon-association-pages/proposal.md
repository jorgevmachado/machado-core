## Why

`PokemonType`, `PokemonAbility` and `PokemonMove` have API contracts for pagination and detail, but `machado-web` does not expose exclusive list and detail pages for those relationship attributes. This change closes that UI gap, adds the small API contract enrichment needed for richer Pokemon type descriptions, and follows the existing web architecture and design-system patterns.

## What Changes

### Resumo da mudança

- Add exclusive protected list and detail pages for `PokemonType`, `PokemonAbility`, and `PokemonMove`.
- Consume existing machado-api pagination and detail endpoints through Next.js BFF route handlers.
- Place all new pages under the existing protected route group and require the same authentication behavior as `/pokemon`.
- Add collapsible child sidebar navigation under Pokemon for type, ability, and move list pages.
- Prefer `name` as the detail route identifier for type, ability, and move links.
- Reuse existing Pokemon page patterns, cards, filters, pagination, loading, alert, and detail UI conventions.
- Treat automatic synchronization as existing machado-api behavior consumed by the web app.
- Enrich `PokemonType` API data with a `description` field sourced from PokeAPI move damage class descriptions.
- Keep `PokemonMove` synchronization resilient by skipping timed-out move resources instead of aborting the whole sync.

### Problema identificado

- Users cannot browse or inspect Pokemon relationship attributes directly.
- Existing Pokemon detail links point to relationship resources, but there are no complete list/detail experiences for those resources.

### Solução proposta

- Implement web-only feature modules for Pokemon types, abilities, and moves.
- Keep entity-specific components and layouts inside each corresponding feature module.
- Add resource-specific services, typed contracts, list hooks, and detail hooks.
- Add list pages with filters, card-based results, pagination, and all expected states: loading, error, empty, and success.
- Use traditional pagination with the existing pagination component; do not implement incremental loading for list pagination.
- Keep list filters and pagination as internal page state; do not require browser URL/query-string synchronization.
- Respect the existing API default ordering for initial lists; do not force additional client-side ordering.
- Add detail pages with rich normalized information from the existing API response, using entity-specific layouts that remain consistent with the design system.
- Build list-card and relationship links with `name` as the preferred detail identifier.
- Keep business rules and automatic synchronization behavior in the API; limit backend work to the contract/support changes required by the new pages.

### Arquivos impactados

- `machado-web/app/(protected)/pokemon/type/**`
- `machado-web/app/(protected)/pokemon/ability/**`
- `machado-web/app/(protected)/pokemon/move/**`
- `machado-web/app/api/pokemon/type/**`
- `machado-web/app/api/pokemon/ability/**`
- `machado-web/app/api/pokemon/move/**`
- `machado-web/app/ui/features/pokemon/type/**`
- `machado-web/app/ui/features/pokemon/ability/**`
- `machado-web/app/ui/features/pokemon/move/**`
- `machado-web/app/ui/features/navigation/**`
- `machado-api/app/domain/pokemon/type/**`
- `machado-api/app/domain/pokemon/move/service.py`
- `machado-api/app/infrastructure/external_api/**`
- `machado-api/app/models/pokemon_type.py`
- `machado-api/migrations/versions/b2c3d4e5f6a7_pokemon_catalog.py`
- `machado-api/tests/app/domain/pokemon/**`
- `machado-api/tests/app/infrastructure/**`
- Existing DS components only if non-breaking reuse improvements are needed.
- Shared association UI should be extracted only when it is genuinely generic; entity-specific UI belongs inside each feature.

### Plano de implementação

- Review current API schemas and existing Pokemon web patterns.
- Add BFF route handlers and service methods for type, ability, and move list/detail reads.
- Add typed hooks and views for the three list pages.
- Add typed hooks and views for the three detail pages.
- Update sidebar navigation to render the relationship links as Pokemon children.
- Add Pokemon type description support to the API schema/model/sync path and expose related type descriptions to the web UI.
- Add move sync timeout handling and unit coverage for the new API branches.
- Add focused tests and run web lint/build/test validation.

### Alterações de modelo

- `PokemonType` gains a persisted `description` field and corresponding schema exposure for full type records and damage relation items.
- The existing Pokemon catalog migration includes the `pokemon_types.description` column for this in-flight catalog schema.
- Frontend TypeScript types will be aligned with existing API contracts for `PokemonType`, `PokemonAbility`, and `PokemonMove`.
- Automatic synchronization remains an API responsibility; the web app consumes the resulting API contracts.

### Alterações de UI

- New pages: type list/detail, ability list/detail, and move list/detail.
- List cards should use a generic card pattern with small content variations per entity.
- `PokemonType` cards should use `badge_url` as the primary visual when available and should not repeat the type name when the badge image already contains it.
- `PokemonType` cards should fallback to a textual color-based presentation when `badge_url` is absent.
- `PokemonType` list cards and relationship cards should display the API-provided `description` when available and fallback to concise explanatory copy when absent.
- Other PokemonType badge fields should be secondary/fallback visuals only when they add clarity and do not clutter the UI.
- `PokemonAbility` and `PokemonMove` cards do not have images, so they should compensate with clear iconography, compact metadata, badges, and typography using the design system.
- `PokemonAbility` cards should prioritize `effect`, `short_effect`, and `flavor_text`, with slot, hidden status, and order as secondary metadata.
- `PokemonMove` cards should prioritize `short_effect` and `effect`, with type, damage class, power, accuracy, and PP as secondary metadata.
- Long ability and move card text should be truncated to preserve a stable grid, with a clear "ver mais" path to the full detail page.
- Detail layouts should be specific to each entity instead of copying the Pokemon detail layout directly.
- Detail pages should hide technical fields such as `id`, raw `url`, and timestamps from the primary UI unless they add user-facing value; `deleted_at` should not be shown in normal flows.
- `PokemonType` strengths and weaknesses should render as clickable links to other `PokemonType` detail pages, preferring `name` as the route identifier.
- `PokemonMove.type` is move metadata and must not be treated as a clickable relationship to `PokemonType`.
- Reverse usage sections or links, such as Pokemon that use a given type, ability, or move, are out of scope.
- New list and detail pages should use the existing breadcrumbs component/pattern.
- Error states should display through the existing Alert pattern without adding a dedicated retry action.
- Empty states should follow the existing system pattern without introducing new actions unless already part of that pattern.
- Filters and traditional pagination using existing components and hooks, managed as internal page state.
- Ability and move previews must truncate long text and provide a clear "ver mais" path to the full detail.
- "Ver mais" is a detail-navigation affordance for truncated text, not an incremental list-loading mechanism.
- Sidebar must show type, ability, and move list links as collapsible Pokemon children with representative icons.
- Sidebar labels for the new links should follow the application's existing navigation label convention.
- The Pokemon sidebar parent must keep direct navigation to `/pokemon`; a separate arrow control expands or collapses the child links.
- The Pokemon sidebar group should auto-expand when the active route is a type, ability, or move child route.
- Evolution timeline is out of scope for `PokemonType`, `PokemonAbility`, and `PokemonMove` detail pages.

### Estratégia de testes

- Route-handler tests for authenticated delegation, unauthenticated access, query forwarding, and API errors.
- Component tests for list pages: success, loading, error, empty, filters, and pagination.
- Component tests for detail pages: success, loading, error, and long content.
- Navigation tests for child links and active states.
- API unit tests for `PokemonType` description enrichment, `PokemonType` response schema fields, PokeAPI move damage class client parsing, and `PokemonMove` sync timeout handling.
- Playwright visual/screenshot checks for success, loading, error, and empty states of the new list/detail pages on desktop and mobile viewports.
- Playwright tests should prefer existing auth helpers when available; otherwise they should use isolated mocked session/token setup and network mocks.
- Playwright should also cover one minimal list-to-detail navigation flow per entity.
- Playwright snapshots should follow the project convention; if adding Playwright for this change, version stable baseline screenshots with the tests.

### Riscos e pontos de atenção

- API contract mismatch: adapt frontend types to actual API fields and document gaps instead of changing the API.
- API catalog schema changed while the catalog migration is still in flight: verify migration state before applying in shared environments.
- PokeAPI move damage class descriptions can be missing: API should return an empty description and the UI should keep a readable fallback.
- Move sync timeout tolerance may produce partial move association data: log skipped resources with order/url for later retry through existing sync behavior.
- Sidebar currently uses a flat menu type: evolve it with optional children without breaking existing navigation.
- Similar resources can lead to duplication: reuse current hooks and local helpers where it reduces meaningful repetition.
- Long move effects can break card/detail layouts: constrain previews and validate responsive layouts.
- Visual regressions can slip through component tests: add Playwright screenshot coverage with versioned stable baselines when the project test setup supports it or when adding the setup for this change.
- DS changes should be minimal and only made when existing components cannot support the required UI through composition.

## Capabilities

### New Capabilities

- `pokemon-association-attributes`: Protected web list and detail experiences for `PokemonType`, `PokemonAbility`, and `PokemonMove`.

### Modified Capabilities

<!-- None. This change introduces a dedicated web capability for Pokemon relationship attributes. -->

## Impact

- Affected app: `machado-web`.
- Expected areas: `app/(protected)/pokemon`, `app/api`, `app/ui/features`, `app/ui/features/navigation`, and focused DS reuse or small DS improvements if the existing components need non-breaking generalization.
- API impact: consumes machado-api pagination and detail endpoints for `PokemonType`, `PokemonAbility`, and `PokemonMove`; enriches `PokemonType` schema/model with `description`; adds PokeAPI move damage class parsing; and makes move sync skip timed-out resources.
- Authentication impact: uses the existing protected layout and BFF server-session authentication; no auth behavior changes.
- Dependencies: no new external dependencies expected.
- Risks: implementation is blocked or must be narrowed if the assumed machado-api endpoints or normalized response fields are missing or incompatible with the existing frontend patterns.
