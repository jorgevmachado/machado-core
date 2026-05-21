## Why

The wild battle backend and contracts already exist, but the web frontend flow was left incomplete, so trainers cannot execute battle actions from the UI. Completing this now unblocks the exploration loop and enables the current battle capability to be fully usable end-to-end.

## What Changes

- Implement the battle session frontend in `machado-web` using existing API/BFF contracts, without reimplementing battle rules in the client.
- Add battle UI composition for active trainer Pokemon, wild Pokemon, HP/PP, available moves, party switching, flee action, and ordered battle logs.
- Integrate exploration outcomes that return `battle_session_id` into navigation and battle screen loading.
- Add battle state refresh/polling, plus loading, empty, terminal, and error states for battle interactions.
- Keep contracts backward-compatible and avoid backend/domain-rule changes in this change.

## Capabilities

### New Capabilities
- _None_

### Modified Capabilities
- `wild-pokemon-battle-session`: Extend requirements to cover web/BFF battle-session completion, including action UX behavior, state refresh, and integration from exploration into battle flow while preserving API-owned battle rules.

## Impact

- Affected specs: `openspec/specs/wild-pokemon-battle-session/spec.md`
- Affected code (expected): `machado-web/app/(protected)/**`, `machado-web/app/api/**`, `machado-web/app/ui/features/**`
- API endpoints/contracts remain the source of truth; no new battle engine behavior is introduced client-side.
