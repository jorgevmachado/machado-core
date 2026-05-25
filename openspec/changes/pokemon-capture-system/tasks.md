## 1. API contract and battle flow

- [x] 1.1 Add the authenticated `POST /trainer/battle/capture` route and request/response schemas using the existing battle namespace
- [x] 1.2 Extend battle session public contracts to support the terminal status `CAPTURED`
- [x] 1.3 Add normalized capture result semantics for success, ineligibility by `capture_rate`, and probabilistic failure
- [x] 1.4 Ensure battle logs and battle-facing payloads can represent capture attempt, capture success, capture failure, and failure reason

## 2. Trainer capture orchestration

- [x] 2.1 Implement the capture use case orchestration in `trainer/service.py`
- [x] 2.2 Validate active battle ownership, session state, and duplicate-success protection before processing capture
- [x] 2.3 Consume one pokeball for every submitted capture attempt, including ineligible attempts
- [x] 2.4 Enforce `trainer.capture_rate >= pokemon.capture_rate` as the capture eligibility gate
- [x] 2.5 Reuse `my-pokemon` domain creation rules to materialize the captured Pokemon only on success
- [x] 2.6 Reuse `pokedex` domain discovery rules to mark the captured Pokemon as discovered only on success
- [x] 2.7 Add and persist trainer capture progression fields needed to support continuous `capture_rate` growth
- [x] 2.8 Apply higher progression reward for newly discovered species than for repeated captures
- [x] 2.9 Recalculate and persist the effective trainer `capture_rate` after each successful capture using the agreed diminishing-returns formula
- [x] 2.10 Commit capture success effects transactionally so `my-pokemon`, `pokedex`, trainer progression, and battle status do not diverge

## 3. Battle engine integration

- [x] 3.1 Extend battle service logic to support capture as a trainer action during an active session
- [x] 3.2 Make eligible capture attempts consume the trainer turn within the battle session
- [x] 3.3 Keep the battle session active after probabilistic capture failure and process the automatic wild response in the same cycle
- [x] 3.4 Finalize the battle session with status `CAPTURED` after successful capture and prevent any further capture attempts in that session
- [x] 3.5 Preserve normalized logs and active battle reads after capture failures so the frontend can continue the battle safely

## 4. Existing endpoint and compatibility cleanup

- [x] 4.1 Review the current `POST /trainer/my-pokemon` behavior and remove or restrict its role as a battle capture entrypoint
- [x] 4.2 Update any internal callers or documentation that still treat `POST /trainer/my-pokemon` as the canonical battle capture flow
- [x] 4.3 Document the compatibility strategy for clients affected by the entrypoint change

## 5. Web and BFF integration

- [x] 5.1 Add the BFF/backend client support for `POST /trainer/battle/capture`
- [x] 5.2 Update the battle UI to expose the capture action with modal, animation, pokeball count, and normalized success/failure feedback
- [x] 5.3 Keep the battle UI stable when capture fails by ineligibility or probabilistic resolution, including the returned updated battle state
- [x] 5.4 Refresh Home, party, and Pokedex views after successful capture

## 6. Validation

- [x] 6.1 Add backend tests for successful capture, ineligible capture, no-pokeball rejection, finished-session rejection, and duplicate-success rejection
- [x] 6.2 Add backend tests proving probabilistic capture failure keeps the battle active and triggers the wild automatic response
- [x] 6.3 Add backend tests proving successful capture creates `my-pokemon`, updates the trainer-owned Pokedex, and finalizes the battle with `CAPTURED`
- [x] 6.4 Add backend tests proving new species grant more trainer progression than repeated captures and that failed captures do not change progression
- [x] 6.5 Add frontend or BFF tests for normalized capture result handling and stable battle continuation after failure
- [x] 6.6 Run the relevant lint and test suites for `machado-api` and `machado-web`
