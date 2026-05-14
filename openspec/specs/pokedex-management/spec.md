## MODIFIED Requirements

### Requirement: Pokedex web experience uses the existing protected architecture
The system SHALL provide protected web flows for listing and detailing trainer-owned Pokédex entries and SHALL expose discovery summaries needed by the trainer Home using the existing `machado-web` BFF, service, hook, and design system patterns.

#### Scenario: Protected routes expose Pokedex pages
- **WHEN** an authenticated user opens `/pokedex` or `/pokedex/[name]`
- **THEN** the web app MUST render protected pages for Pokédex list and detail using the existing protected App Router structure

#### Scenario: Web list and detail consume normalized contracts
- **WHEN** the BFF returns `pokedex` list or detail data
- **THEN** the web app MUST render nickname, base Pokémon image, level, experience, HP/max HP, generated attributes, discovery status, and discovery date using existing design system patterns

#### Scenario: Web list supports pagination and filters
- **WHEN** the user paginates or filters the owned Pokédex roster
- **THEN** the page MUST use the existing paginated list hook pattern and MUST render loading, error, empty, and success states consistent with the rest of the application

#### Scenario: Home exposes the latest discovered Pokedex entries
- **WHEN** an authenticated trainer opens the protected Home view
- **THEN** the API and BFF layers MUST provide the three most recently discovered `pokedex` entries for that trainer in the Home payload

#### Scenario: Web keeps manual discovery UI out of scope for this change
- **WHEN** the frontend consumes the new `pokedex` capability
- **THEN** it MUST support list and detail flows without introducing a dedicated manual discovery interface in this implementation
