## ADDED Requirements

### Requirement: Successful battle capture automatically discovers the Pokemon for the trainer
The system SHALL automatically mark a Pokemon as discovered in the trainer-owned Pokedex when that trainer successfully captures the Pokemon during battle.

#### Scenario: Capture success updates only the authenticated trainer Pokedex
- **WHEN** an authenticated trainer successfully captures a wild Pokemon
- **THEN** the system MUST mark that Pokemon as discovered only in the Pokedex owned by that trainer

#### Scenario: Capture failure does not discover the Pokemon
- **WHEN** a battle capture attempt fails or is rejected
- **THEN** the system MUST NOT mark the Pokemon as discovered in the trainer-owned Pokedex

#### Scenario: Automatic discovery does not require manual web interaction
- **WHEN** the trainer completes a successful battle capture
- **THEN** the system MUST update the trainer-owned Pokedex automatically without requiring a separate manual discovery action in the web flow
