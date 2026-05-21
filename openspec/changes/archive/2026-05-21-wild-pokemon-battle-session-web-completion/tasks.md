## 1. Battle Feature Foundation

- [x] 1.1 Create `app/ui/features/battle` structure (types, service, hooks, and UI composition entrypoints) following existing feature conventions.
- [x] 1.2 Define typed battle DTOs/actions aligned with existing battle-session and battle-log contracts (no contract changes).
- [x] 1.3 Implement battle feature service methods for read session, list logs, use move, switch Pokemon, and flee using existing BFF/API endpoints.

## 2. BFF and Route Integration

- [x] 2.1 Add or complete protected BFF route handlers under `app/api` for battle read/actions/log retrieval using server session auth.
- [x] 2.2 Implement protected battle page route in `app/(protected)` and wire initial session loading + empty state behavior.
- [x] 2.3 Integrate exploration wild-encounter handoff so `has_active_battle`/`battle_session_id` navigates trainer into battle flow.

## 3. Battle Screen Interaction and Synchronization

- [x] 3.1 Build battle action panel and state panels (trainer/wild Pokemon, HP, moves with PP, switch, flee) using existing DS components.
- [x] 3.2 Enforce UI interaction constraints from server state (disable zero-PP moves, block all actions on terminal statuses, lock controls during in-flight actions).
- [x] 3.3 Implement periodic polling while battle is active and explicit post-action refresh to keep battle/session/log state synchronized.

## 4. UX States and Validation

- [x] 4.1 Implement loading, error, empty, and terminal battle states with retry and safe fallback navigation.
- [x] 4.2 Ensure battle logs render in chronological order from API/BFF response data.
- [x] 4.3 Add/update frontend tests (services/hooks/components/routes) covering action dispatch, state synchronization, and battle terminal-state behavior.
