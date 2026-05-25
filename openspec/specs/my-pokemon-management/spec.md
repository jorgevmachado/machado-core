## ADDED Requirements

### Requirement: Battle capture can materialize owned MyPokemon through the domain owner
The system SHALL allow a successful wild battle capture to create a trainer-owned `my-pokemon` through the `my-pokemon` domain rules instead of duplicating creation logic in another domain.

#### Scenario: Successful battle capture creates owned Pokemon
- **WHEN** an authenticated trainer successfully captures the active wild Pokemon during battle
- **THEN** the system MUST create a trainer-owned `my-pokemon` using the canonical `my-pokemon` creation rules

#### Scenario: Failed battle capture does not create owned Pokemon
- **WHEN** a battle capture attempt fails or is rejected
- **THEN** the system MUST NOT create a trainer-owned `my-pokemon`

#### Scenario: Battle capture does not require a parallel public create flow
- **WHEN** the system supports creating `my-pokemon` from battle capture
- **THEN** the implementation MUST NOT require a separate conflicting public capture flow outside the canonical trainer battle capture entrypoint
