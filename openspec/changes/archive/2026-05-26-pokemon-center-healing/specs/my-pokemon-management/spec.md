## ADDED Requirements

### Requirement: Pokemon Center healing can restore persisted MyPokemon battle resources
The system SHALL allow the canonical Pokemon Center healing flow to restore persisted `my-pokemon` state through the owning domain rules.

#### Scenario: Healing restores persisted HP for owned party Pokemon
- **WHEN** an authenticated trainer successfully heals the active party at the Pokemon Center
- **THEN** the system MUST update the owned party members so their persisted `current_hp` matches their maximum HP state

#### Scenario: Healing restores persisted PP for owned moves
- **WHEN** an authenticated trainer successfully heals the active party at the Pokemon Center
- **THEN** the system MUST update the owned moves of the healed party members so their persisted `current_pp` matches their maximum PP state

#### Scenario: Healing revives fainted owned Pokemon
- **WHEN** an authenticated trainer successfully heals the active party and one owned party member has `current_hp = 0`
- **THEN** the system MUST revive that owned Pokemon through the canonical `my-pokemon` state update rules instead of a parallel persistence flow
