## ADDED Requirements

### Requirement: Trainer capture flow is orchestrated as a battle-scoped trainer use case
The system SHALL expose wild Pokemon capture as a trainer-orchestrated use case reached through the battle namespace, with `trainer` coordinating `battle`, `my-pokemon`, and `pokedex` responsibilities without duplicating their internal rules.

#### Scenario: Battle capture uses the canonical trainer battle endpoint
- **WHEN** an authenticated trainer attempts to capture the active wild Pokemon during an active battle
- **THEN** the system MUST process the request through `POST /trainer/battle/capture` as the canonical public entrypoint for battle capture

#### Scenario: Capture success materializes owned Pokemon and trainer discovery in one flow
- **WHEN** a capture attempt succeeds
- **THEN** the system MUST create the owned `my-pokemon`, mark the Pokemon as discovered in the trainer's Pokedex, and finalize the battle outcome within the same orchestrated flow

#### Scenario: Capture failure does not materialize cross-domain success effects
- **WHEN** a capture attempt fails
- **THEN** the system MUST NOT create a `my-pokemon` and MUST NOT mark the Pokemon as discovered in the trainer's Pokedex

#### Scenario: Capture success updates trainer progression in the same flow
- **WHEN** a capture attempt succeeds
- **THEN** the system MUST update the trainer capture progression and the effective trainer `capture_rate` within the same orchestrated flow

### Requirement: Capture eligibility is gated by trainer and Pokemon capture rate
The system SHALL require trainer eligibility before a wild Pokemon can be captured, comparing the trainer's `capture_rate` against the wild Pokemon's required `capture_rate`.

#### Scenario: Eligible trainer may proceed to capture resolution
- **WHEN** `trainer.capture_rate` is greater than or equal to the active wild Pokemon `capture_rate`
- **THEN** the system MUST allow the capture attempt to proceed to capture resolution

#### Scenario: Ineligible trainer cannot capture the Pokemon
- **WHEN** `trainer.capture_rate` is lower than the active wild Pokemon `capture_rate`
- **THEN** the system MUST reject the capture as ineligible

#### Scenario: Ineligible attempt still consumes one pokeball
- **WHEN** a trainer submits a capture attempt for an ineligible Pokemon
- **THEN** the system MUST consume one pokeball even though the capture is rejected by the eligibility rule

### Requirement: Capture outcomes return normalized battle-aware results
The system SHALL return normalized responses that distinguish success, ineligibility, and probabilistic failure while preserving the active battle contract.

#### Scenario: Successful capture returns a terminal capture result
- **WHEN** a capture attempt succeeds
- **THEN** the response MUST indicate success, include the terminal battle status `CAPTURED`, include the created `my-pokemon`, and indicate that the Pokedex was updated

#### Scenario: Ineligibility returns a normalized failure reason
- **WHEN** a capture attempt is rejected because `trainer.capture_rate` is lower than the active wild Pokemon `capture_rate`
- **THEN** the response MUST indicate a normalized ineligibility result distinct from a probabilistic capture failure

#### Scenario: Probabilistic failure returns updated battle context
- **WHEN** a trainer is eligible to capture but the attempt fails by chance
- **THEN** the response MUST indicate failure and MUST include the updated battle state needed for the frontend to continue the active battle

### Requirement: Trainer capture rate progresses continuously after successful captures
The system SHALL increase the trainer effective `capture_rate` after successful captures using continuous progression points, with higher reward for species newly captured by that trainer than for repeated captures of a species already discovered.

#### Scenario: New species grants higher progression than repeated species
- **WHEN** a trainer successfully captures a Pokemon species not previously discovered by that trainer
- **THEN** the system MUST grant more capture progression points than it would for a repeated capture of a species already discovered by that same trainer

#### Scenario: Only successful captures increase capture progression
- **WHEN** a capture attempt fails or is rejected
- **THEN** the system MUST NOT increase the trainer capture progression points or effective `capture_rate`

#### Scenario: Effective capture rate grows continuously with diminishing returns
- **WHEN** the trainer accumulates successful capture progression points
- **THEN** the system MUST recalculate the effective trainer `capture_rate` using a continuous formula with diminishing returns and MUST cap the final value at `255`
