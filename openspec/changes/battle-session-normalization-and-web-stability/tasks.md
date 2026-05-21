## 1. Spec and Contract Alignment

- [ ] 1.1 Normalize the proposal/design/spec artifacts so battle naming, final statuses, and active-session semantics are defined consistently.
- [ ] 1.2 Decide and document the canonical behavior for “no active battle” across API, BFF, and frontend consumers.
- [ ] 1.3 Document the final `/trainer/battle/*` endpoint contract and any temporary legacy compatibility that must exist during migration.

## 2. Backend Battle Domain Refactor

- [ ] 2.1 Refactor the battle domain internals toward a neutral battle-session identity without breaking approved persistence history.
- [x] 2.2 Align backend enums, schemas, serializers, and service outputs with the canonical terminal statuses.
- [x] 2.3 Implement the canonical active-session read behavior and add backend coverage for the “no active battle” case.

## 3. Home and Exploration Integration

- [x] 3.1 Extend the trainer Home contract with a normalized active-battle summary for resume/reopen behavior.
- [ ] 3.2 Update exploration walk responses and invalidation rules to keep Home and battle summary state synchronized.
- [x] 3.3 Add tests covering walk while battle is active, battle-summary presence in Home, and cache invalidation on battle lifecycle changes.

## 4. Web Modal Flow and Route Stability

- [x] 4.1 Replace the automatic encounter redirect with modal opening using the existing `Modal` component and `useModal` hook.
- [x] 4.2 Keep `/battle` accessible as a future-facing protected page while removing it as the primary encounter handoff path.
- [x] 4.3 Make the battle UI resilient to empty, active, terminal, and post-terminal refresh states without premature redirects.

## 5. Frontend Contract and Namespace Alignment

- [x] 5.1 Update BFF routes, feature services, and frontend types to the canonical battle-session contract and namespaced endpoints.
- [x] 5.2 Remove assumptions that `/active` always returns an active session and stop polling when the battle becomes absent or terminal.
- [x] 5.3 Add regression tests for `encounter -> modal`, `finalize battle -> read /active`, and Home-driven battle resume behavior.
