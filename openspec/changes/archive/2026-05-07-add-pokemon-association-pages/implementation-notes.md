## Implementation Notes

- Implementation now spans `machado-web` plus tightly scoped `machado-api` contract support.
- The web layer consumes Pokemon type, ability, and move list/detail API contracts through Next.js BFF routes.
- `PokemonType` API records now expose `description`, including related type items in strengths and weaknesses.
- `PokemonTypeService` populates descriptions from PokeAPI move damage class descriptions when creating or enriching incomplete types, with empty-string fallback when description data is unavailable.
- `PokemonMoveService.sync_from_resources` skips and logs individual `httpx.TimeoutException` failures so one timed-out move does not abort the whole sync batch.
- Detail navigation uses `name` as the preferred identifier.
- `PokemonMove.type` is rendered as move metadata only; it is not linked to `PokemonType`.
- Reverse Pokemon usage sections were intentionally kept out of scope.
- Playwright visual tests use deterministic route mocks and a mocked auth cookie. Chromium must be installed locally before running `yarn test:visual`.
- A small non-breaking DS loading fix was added so delayed loading timers are cleared on unmount; this prevents Next.js dev overlay failures during rapid Playwright navigation.
- API unit coverage was updated for Pokemon type description enrichment, move timeout handling, and PokeAPI move damage class client parsing. The focused API coverage run reports 100% for `app/domain/pokemon/type/service.py` and `app/domain/pokemon/move/service.py`.
- Current verification note: `machado-api` unit tests and coverage passed; `machado-web` unit tests passed after updating `PokemonTypeDetailView` expectations for the accessible link labels. `machado-web` lint still reports formatting errors in implementation files that should be fixed before marking pre-PR validation complete.
