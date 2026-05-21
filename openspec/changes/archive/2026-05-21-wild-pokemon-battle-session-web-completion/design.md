## Context

The `wild-pokemon-battle-session` capability already defines API-side battle creation, turn resolution, logs, switching, and escape. The gap is in `machado-web`: trainers cannot complete battle actions in the UI after exploration creates or resumes an active battle session.

The web architecture constrains this change to existing patterns: protected routes, Next.js BFF route handlers in `app/api`, feature services extending `BaseServiceAbstract`, and no business-rule duplication in client code. Existing contracts from exploration and battle APIs must remain the source of truth.

## Goals / Non-Goals

**Goals:**
- Deliver a protected battle-session web flow that renders normalized battle state and logs.
- Enable move usage, party switch, and flee actions through existing BFF/API endpoints.
- Keep battle state consistent through refresh/polling and post-action synchronization.
- Integrate exploration outcome (`battle_session_id`, `has_active_battle`) with battle route navigation.
- Provide explicit loading, empty, terminal, and error states for battle UX.

**Non-Goals:**
- Reimplementing backend battle rules, progression, capture, or damage logic.
- Introducing websocket infrastructure or multiplayer behavior.
- Redesigning existing API contracts or adding new battle engine rules.

## Decisions

1. **Battle UI will be implemented as a dedicated web feature module (`app/ui/features/battle`)**
   - **Why:** Keeps parity with existing domain-feature organization (service/types/components/hooks) and avoids leaking battle concerns into generic UI layers.
   - **Alternative considered:** Embedding battle logic directly in route page components. Rejected due to lower reuse/testability and inconsistent project structure.

2. **All battle actions/read models go through existing BFF routes and typed feature services**
   - **Why:** Preserves auth/session handling in server-side cookies and keeps API URL/token logic centralized.
   - **Alternative considered:** Direct client calls to machado-api. Rejected because it bypasses established BFF/auth conventions.

3. **Battle state synchronization uses polling plus immediate refetch after mutations**
   - **Why:** Current scope explicitly excludes websockets; polling is enough to keep the trainer view current and aligns with existing constraints.
   - **Alternative considered:** Websocket subscription for realtime turns. Rejected as out of scope and introduces infra complexity.

4. **Client remains a pure consumer of backend rules**
   - **Why:** Rules for PP validation, turn progression, status transitions, and terminal outcomes already exist in API and must not be duplicated.
   - **Alternative considered:** Client-side optimistic rule simulation. Rejected due to drift risk and duplicated domain logic.

## Risks / Trade-offs

- **[Risk] Polling frequency can increase API load** → **Mitigation:** use bounded interval only while battle is active and stop polling in terminal states/unmounted views.
- **[Risk] Action race conditions (double clicks / stale state)** → **Mitigation:** disable action controls while mutation is in-flight and always reconcile with latest server response.
- **[Risk] Contract mismatches between BFF and UI rendering** → **Mitigation:** enforce typed DTOs in battle feature and normalize response mapping in one place.
- **[Risk] Incomplete exploration→battle handoff UX** → **Mitigation:** standardize navigation trigger on `has_active_battle`/`battle_session_id` and fallback to explicit empty/error states.

## Migration Plan

1. Add the battle feature module and BFF route handlers aligned with existing contracts.
2. Wire protected battle page and exploration handoff navigation.
3. Roll out action handling and polling with terminal-state stop conditions.
4. Validate UX states (loading/error/empty/terminal) and keep fallback navigation to Home/Exploration.
5. Rollback strategy: revert web-only feature wiring; backend contracts remain unchanged and compatible.

## Open Questions

- None at proposal time; implementation will reuse existing battle endpoint contracts without introducing schema changes.
