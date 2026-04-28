# Database Models

## 1. `User`
| Column                    | Type               | Constraints                                                          |
|---------------------------|--------------------|----------------------------------------------------------------------|
| `id`                      | UUID               | Primary key, default `uuid4`, not nullable                           |
| `name`                    | String             | Not nullable                                                         |
| `email`                   | String             | Unique, not nullable                                                 |
| `username`                | String             | Unique, not nullable                                                 |
| `password`                | String             | Not nullable — must always be stored hashed                          |
| `date_of_birth`           | DateTime           | Not nullable                                                         |
| `total_authentications`   | Integer            | Nullable, default `0`                                                |
| `authentication_success`  | Integer            | Nullable, default `0`                                                |
| `authentication_failures` | Integer            | Nullable, default `0`                                                |
| `last_authentication_at`  | DateTime           | Nullable, default `None`                                             |
| `created_at`              | DateTime           | Not nullable, default `utcnow`, **never updated after insert**       |
| `updated_at`              | DateTime           | Nullable, default `None`, **only set on second or later updates**    |
| `deleted_at`              | DateTime           | Nullable, default `None`, **only set when soft-delete is triggered** |
| `gender`                  | Enum(`GenderEnum`) | Not nullable                                                         |
| `status`                  | Enum(`StatusEnum`) | Not nullable, default `INACTIVE`                                     |

**`GenderEnum`**
- `MALE`
- `FEMALE`
- `OTHER`

**`StatusEnum`**
- `ACTIVE`
- `INACTIVE`

**Behavior rules for `User`:**
- `password` must **never** be stored in plain text. Always use `from app.core.security import get_password_hash` before persisting.
- `trainer` → one-to-one relationship to `Trainer` via foreign key `user_id` on `trainers`

---

## 2. `Trainer`

| Column           | Type                      | Constraints                                                           |
|------------------|---------------------------|-----------------------------------------------------------------------|
| `id`             | UUID                      | Primary key, default `uuid4`, not nullable                            |
| `user_id`        | UUID                      | Foreign key → `users.id`, not nullable, unique (one trainer per user) |
| `capture_rate`   | Integer                   | Not nullable                                                          |
| `pokeballs`      | Integer                   | Not nullable                                                          |
| `created_at`     | DateTime                  | Not nullable, default `utcnow`, **never updated after insert**        |
| `updated_at`     | DateTime                  | Nullable, default `None`, **only set on second or later updates**     |
| `deleted_at`     | DateTime                  | Nullable, default `None`, **only set when soft-delete is triggered**  |
| `pokedex_status` | Enum(`PokedexStatusEnum`) | Not nullable, default `EMPTY`                                         |

**`PokedexStatusEnum`**
- `EMPTY`
- `READY`
- `FAILED`
- `INITIALIZING`

**Behavior rules for `Trainer`:**
- One user can only have one trainer record (enforce via unique constraint on `user_id`).
- `pokedex_status` must be set to `READY` after the first successful pokedex generation.
- `user` → one-to-one relationship to `User` via foreign key `user_id` on `trainers`
- `pokedex_entries` → one-to-many relationship to `Pokedex` via foreign key `trainer_id` on `pokedex`
- `my_pokemons` → one-to-many relationship to `MyPokemon` via foreign key `trainer_id` on `my_pokemons`
- `battle_party_entries` → one-to-many relationship to `TrainerBattleParty` via foreign key `trainer_id` on `trainer_battle_party`
- `encounters` → one-to-many relationship to `Encounter` via foreign key `trainer_id` on `encounters`
- `battles` → one-to-many relationship to `Battle` via foreign key `trainer_id` on `battles`
- `escape_history` → one-to-many relationship to `TrainerEscapeHistory` via foreign key `trainer_id` on `trainer_escape_history`
- `battle_stats` → one-to-one relationship to `TrainerBattleStats` via foreign key `trainer_id` on `trainer_battle_stats` (enforced by `uq_trainer_battle_stats_trainer_id`)

---

## 3. `Pokemon`
| Column                   | Type                      | Constraints                                                          |
|--------------------------|---------------------------|----------------------------------------------------------------------|
| `id`                     | UUID                      | Primary key, default `uuid4`, not nullable                           |
| `name`                   | String                    | Not nullable                                                         |
| `order`                  | Integer                   | Not nullable                                                         |
| `external_image`         | String                    | Not nullable                                                         |
| `status`                 | Enum(`PokemonStatusEnum`) | Not nullable — must always be stored hashed                          |
| `hp`                     | Integer                   | Nullable, default `0`                                                |
| `image`                  | String                    | Nullable, default `None`                                             |
| `speed`                  | Integer                   | Nullable, default `0`                                                |
| `height`                 | Integer                   | Nullable, default `0`                                                |
| `weight`                 | Integer                   | Nullable, default `0`                                                |
| `attack`                 | Integer                   | Nullable, default `0`                                                |
| `defense`                | Integer                   | Nullable, default `0`                                                |
| `habitat`                | String                    | Nullable, default `None`                                             |
| `is_baby`                | Boolean                   | Nullable, default `False`                                            |
| `shape_url`              | String                    | Nullable                                                             |
| `shape_name`             | String                    | Nullable                                                             |
| `is_mythical`            | Boolean                   | Nullable, default `False`                                            |
| `gender_rate`            | Integer                   | Nullable, default `0`                                                |
| `is_legendary`           | Boolean                   | Nullable, default `False`                                            |
| `capture_rate`           | Integer                   | Nullable, default `0`                                                |
| `hatch_counter`          | Integer                   | Nullable, default `0`                                                |
| `base_happiness`         | Integer                   | Nullable, default `0`                                                |
| `special_attack`         | Integer                   | Nullable, default `0`                                                |
| `base_experience`        | Integer                   | Nullable, default `0`                                                |
| `special_defense`        | Integer                   | Nullable, default `0`                                                |
| `evolution_chain`        | String                    | Nullable                                                             |
| `evolves_from_species`   | String                    | Nullable                                                             |
| `has_gender_differences` | Boolean                   | Nullable, default `False`                                            |
| `growth_rate_id`         | UUID                      | Foreign key → `pokemon_growth_rates.id`, Nullable                    |
| `created_at`             | DateTime                  | Not nullable, default `utcnow`, **never updated after insert**       |
| `updated_at`             | DateTime                  | Nullable, default `None`, **only set on second or later updates**    |
| `deleted_at`             | DateTime                  | Nullable, default `None`, **only set when soft-delete is triggered** |

**`PokemonStatusEnum`**
- `COMPLETE`
- `INCOMPLETE`

**Behavior rules for `Pokemon`:**
- `growth_rate` → many-to-one relationship to `PokemonGrowthRate` via foreign key `growth_rate_id` on `pokemons` (optional, nullable)
- `moves` → many-to-many relationship to `PokemonMove` via association table `pokemon_pokemon_moves`
- `abilities` → many-to-many relationship to `PokemonAbility` via association table `pokemon_pokemon_abilities`
- `types` → many-to-many relationship to `PokemonType` via association table `pokemon_pokemon_types`
- `evolutions` → many-to-many self-referential relationship on `pokemons` via association table `pokemon_evolutions` (both directions accessible via `pokemon.evolutions`)
- `my_pokemons` → one-to-many relationship to `MyPokemon` via foreign key `pokemon_id` on `my_pokemons`
- `pokedex_entries` → one-to-many relationship to `Pokedex` via foreign key `pokemon_id` on `pokedex`

---

## 4. `PokemonType`

| Column             | Type     | Constraints                                                   |
|--------------------|----------|---------------------------------------------------------------|
| `id`               | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`              | String   | Not nullable                                                  |
| `order`            | Integer  | Not nullable                                                  |
| `name`             | String   | Unique, not nullable                                          |
| `text_color`       | String   | Not nullable                                                  |
| `background_color` | String   | Not nullable                                                  |
| `created_at`       | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`       | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`       | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonType`:**
- `pokemons` → many-to-many relationship to `Pokemon` via association table `pokemon_pokemon_types`
- `weaknesses` → many-to-many self-referential relationship to `PokemonType` via association table `pokemon_type_weaknesses` (types that deal double or half damage to this type)
- `strengths` → many-to-many self-referential relationship to `PokemonType` via association table `pokemon_type_strengths` (types this type deals double or half damage to)

---

## 5. `PokemonMove`

| Column          | Type     | Constraints                                                   |
|-----------------|----------|---------------------------------------------------------------|
| `id`            | UUID     | Primary key, default `uuid4`, not nullable                    |
| `pp`            | Integer  | Not nullable                                                  |
| `url`           | String   | Not nullable                                                  |
| `type`          | String   | Not nullable                                                  |
| `name`          | String   | Unique, not nullable                                          |
| `order`         | Integer  | Not nullable                                                  |
| `power`         | Integer  | Not nullable                                                  |
| `target`        | String   | Not nullable                                                  |
| `effect`        | String   | Not nullable                                                  |
| `priority`      | Integer  | Not nullable                                                  |
| `accuracy`      | Integer  | Not nullable                                                  |
| `short_effect`  | String   | Not nullable                                                  |
| `damage_class`  | String   | Not nullable                                                  |
| `effect_chance` | Integer  | Nullable                                                      |
| `created_at`    | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`    | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`    | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonMove`:**
- `pokemons` → many-to-many relationship to `Pokemon` via association table `pokemon_pokemon_moves`
- `my_pokemons` → many-to-many relationship to `MyPokemon` via association table `my_pokemon_pokemon_moves`
- `my_pokemon_move_states` → one-to-many relationship to `MyPokemonMoveState` via foreign key `pokemon_move_id` on `my_pokemon_move_states`

---

## 6. `PokemonAbility`

| Column       | Type     | Constraints                                                   |
|--------------|----------|---------------------------------------------------------------|
| `id`         | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`        | String   | Not nullable                                                  |
| `order`      | Integer  | Not nullable                                                  |
| `name`       | String   | Unique, not nullable                                          |
| `slot`       | Integer  | Not nullable                                                  |
| `is_hidden`  | Boolean  | Not nullable                                                  |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonAbility`:**
- `pokemons` → many-to-many relationship to `Pokemon` via association table `pokemon_pokemon_abilities`

---

## 7. `PokemonGrowthRate`

| Column        | Type     | Constraints                                                   |
|---------------|----------|---------------------------------------------------------------|
| `id`          | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`         | String   | Not nullable                                                  |
| `name`        | String   | Unique, not nullable                                          |
| `formula`     | String   | Not nullable                                                  |
| `description` | String   | Not nullable                                                  |
| `created_at`  | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`  | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`  | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonGrowthRate`:**
- `pokemons` → one-to-many relationship to `Pokemon` via foreign key `growth_rate_id` on `pokemons`

---

## 8. `MyPokemon`

| Column               | Type     | Constraints                                                          |
|----------------------|----------|----------------------------------------------------------------------|
| `id`                 | UUID     | Primary key, default `uuid4`, not nullable                           |
| `hp`                 | Integer  | not nullable                                                         |
| `iv_hp`              | Integer  | not nullable                                                         |
| `ev_hp`              | Integer  | not nullable                                                         |
| `wins`               | Integer  | not nullable                                                         |
| `level`              | Integer  | not nullable                                                         |
| `losses`             | Integer  | not nullable                                                         |
| `max_hp`             | Integer  | not nullable                                                         |
| `battles`            | Integer  | not nullable                                                         |
| `nickname`           | String   | not nullable                                                         |
| `speed`              | Integer  | not nullable                                                         |
| `iv_speed`           | Integer  | not nullable                                                         |
| `ev_speed`           | Integer  | not nullable                                                         |
| `attack`             | Integer  | not nullable                                                         |
| `iv_attack`          | Integer  | not nullable                                                         |
| `ev_attack`          | Integer  | not nullable                                                         |
| `defense`            | Integer  | not nullable                                                         |
| `iv_defense`         | Integer  | not nullable                                                         |
| `ev_defense`         | Integer  | not nullable                                                         |
| `experience`         | Integer  | not nullable                                                         |
| `special_attack`     | Integer  | not nullable                                                         |
| `iv_special_attack`  | Integer  | not nullable                                                         |
| `ev_special_attack`  | Integer  | not nullable                                                         |
| `special_defense`    | Integer  | not nullable                                                         |
| `iv_special_defense` | Integer  | not nullable                                                         |
| `ev_special_defense` | Integer  | not nullable                                                         |
| `formula`            | String   | not nullable                                                         |
| `capture_at`         | DateTime | not nullable, default `utcnow`, **never updated after insert**       |
| `created_at`         | DateTime | Not nullable, default `utcnow`, **never updated after insert**       |
| `updated_at`         | DateTime | Nullable, default `None`, **only set on second or later updates**    |
| `deleted_at`         | DateTime | Nullable, default `None`, **only set when soft-delete is triggered** |
| `pokemon_id`         | UUID     | Foreign key → `pokemons.id`, Not nullable                            |
| `trainer_id`         | UUID     | Foreign key → `trainers.id`, Not nullable                            |

**Relationships on `MyPokemon`:**
- `pokemon` → many-to-one relationship to `Pokemon` via foreign key `pokemon_id` on `my_pokemons`
- `trainer` → many-to-one relationship to `Trainer` via foreign key `trainer_id` on `my_pokemons`
- `moves` → many-to-many relationship to `PokemonMove` via association table `my_pokemon_pokemon_moves`
- `move_states` → one-to-many relationship to `MyPokemonMoveState` via foreign key `my_pokemon_id` on `my_pokemon_move_states`
- `battle_party_entries` → one-to-many relationship to `TrainerBattleParty` via foreign key `my_pokemon_id` on `trainer_battle_party`

**Behavior rules for `MyPokemon`:**
- `captured_at` is set only when the record is created and must never be changed afterwards.

---

## 9. `MyPokemonMoveState`

| Column                             | Type     | Constraints                                                                            |
|------------------------------------|----------|----------------------------------------------------------------------------------------|
| `my_pokemon_id`, `pokemon_move_id` | UUID     | Unique key, default `uuid4`, not nullable, `uq_my_pokemon_move_states_my_pokemon_move` |
| `current_pp`                       | Integer  | not nullable                                                                           |
| `created_at`                       | DateTime | Not nullable, default `utcnow`, **never updated after insert**                         |
| `updated_at`                       | DateTime | Nullable, default `None`, **only set on second or later updates**                      |
| `deleted_at`                       | DateTime | Nullable, default `None`, **only set when soft-delete is triggered**                   |
| `my_pokemon_id`                    | UUID     | Foreign key → `my_pokemons.id`, Not nullable                                           |
| `pokemon_move_id`                  | UUID     | Foreign key → `moves.id`, Not nullable                                                 |

**Relationships on `MyPokemonMoveState`:**
- `my_pokemon` → many-to-one relationship to `MyPokemon` via foreign key `my_pokemon_id` on `my_pokemon_move_states`
- `pokemon_move` → many-to-one relationship to `PokemonMove` via foreign key `pokemon_move_id` on `my_pokemon_move_states`

---

## 10. `Pokedex`

| Column               | Type     | Constraints                                                           |
|----------------------|----------|-----------------------------------------------------------------------|
| `id`                 | UUID     | Primary key, default `uuid4`, not nullable                            |
| `hp`                 | Integer  | not nullable                                                          |
| `iv_hp`              | Integer  | not nullable                                                          |
| `ev_hp`              | Integer  | not nullable                                                          |
| `wins`               | Integer  | not nullable                                                          |
| `level`              | Integer  | not nullable                                                          |
| `losses`             | Integer  | not nullable                                                          |
| `max_hp`             | Integer  | not nullable                                                          |
| `battles`            | Integer  | not nullable                                                          |
| `nickname`           | String   | not nullable                                                          |
| `speed`              | Integer  | not nullable                                                          |
| `iv_speed`           | Integer  | not nullable                                                          |
| `ev_speed`           | Integer  | not nullable                                                          |
| `attack`             | Integer  | not nullable                                                          |
| `iv_attack`          | Integer  | not nullable                                                          |
| `ev_attack`          | Integer  | not nullable                                                          |
| `defense`            | Integer  | not nullable                                                          |
| `iv_defense`         | Integer  | not nullable                                                          |
| `ev_defense`         | Integer  | not nullable                                                          |
| `experience`         | Integer  | not nullable                                                          |
| `special_attack`     | Integer  | not nullable                                                          |
| `iv_special_attack`  | Integer  | not nullable                                                          |
| `ev_special_attack`  | Integer  | not nullable                                                          |
| `special_defense`    | Integer  | not nullable                                                          |
| `iv_special_defense` | Integer  | not nullable                                                          |
| `ev_special_defense` | Integer  | not nullable                                                          |
| `discovered`         | Boolean  | Nullable, default `False`                                             |
| `formula`            | String   | not nullable                                                          |
| `discovered_at`      | DateTime | Nullable, default `None`, **only set on change discovered attribute** |
| `created_at`         | DateTime | Not nullable, default `utcnow`, **never updated after insert**        |
| `updated_at`         | DateTime | Nullable, default `None`, **only set on second or later updates**     |
| `deleted_at`         | DateTime | Nullable, default `None`, **only set when soft-delete is triggered**  |
| `pokemon_id`         | UUID     | Foreign key → `pokemons.id`, Not nullable                             |
| `trainer_id`         | UUID     | Foreign key → `trainers.id`, Not nullable                             |

**Relationships on `Pokedex`:**
- `pokemon` → many-to-one relationship to `Pokemon` via foreign key `pokemon_id` on `pokedex`
- `trainer` → many-to-one relationship to `Trainer` via foreign key `trainer_id` on `pokedex`

**Behavior rules for `Pokedex`:**
- `discovered_at` is set only when `discovered` attribute is changed from `False` to `True`.

---

## 11. `TrainerBattleStats`

| Column            | Type     | Constraints                                                                                      |
|-------------------|----------|--------------------------------------------------------------------------------------------------|
| `id`              | UUID     | Primary key, default `uuid4`, not nullable                                                       |
| `trainer_id`      | UUID     | Foreign key → `trainers.id`, not nullable, unique (`uq_trainer_battle_stats_trainer_id`)         |
| `total_victories` | Integer  | Not nullable, default `0`                                                                        |
| `total_defeats`   | Integer  | Not nullable, default `0`                                                                        |
| `total_escapes`   | Integer  | Not nullable, default `0`                                                                        |
| `total_captures`  | Integer  | Not nullable, default `0`                                                                        |
| `total_battles`   | Integer  | Not nullable, default `0`                                                                        |
| `created_at`      | DateTime | Not nullable, default `utcnow`, **never updated after insert**                                   |
| `updated_at`      | DateTime | Nullable, default `None`, **only set on second or later updates**                                |

**Relationships on `TrainerBattleStats`:**
- `trainer` → one-to-one relationship to `Trainer` via foreign key `trainer_id` on `trainer_battle_stats` (enforced by `uq_trainer_battle_stats_trainer_id`)

---

## 12. `TrainerBattleParty`

| Column          | Type     | Constraints                                                                                                                                                  |
|-----------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`            | UUID     | Primary key, default `uuid4`, not nullable                                                                                                                   |
| `trainer_id`    | UUID     | Foreign key → `trainers.id`, not nullable — unique together with `slot` (`uq_trainer_battle_party_trainer_slot`)                                             |
| `my_pokemon_id` | UUID     | Foreign key → `my_pokemons.id`, not nullable — unique together with `trainer_id` (`uq_trainer_battle_party_trainer_my_pokemon`)                              |
| `slot`          | Integer  | Not nullable                                                                                                                                                 |
| `created_at`    | DateTime | Not nullable, default `utcnow`, **never updated after insert**                                                                                               |
| `updated_at`    | DateTime | Nullable, default `None`, **only set on second or later updates**                                                                                            |

**Relationships on `TrainerBattleParty`:**
- `trainer` → many-to-one relationship to `Trainer` via foreign key `trainer_id` on `trainer_battle_party`
- `my_pokemon` → many-to-one relationship to `MyPokemon` via foreign key `my_pokemon_id` on `trainer_battle_party`

**Behavior rules for `TrainerBattleParty`:**
- A trainer cannot have the same `slot` twice (enforced by `uq_trainer_battle_party_trainer_slot`).
- A trainer cannot have the same Pokémon twice in the party (enforced by `uq_trainer_battle_party_trainer_my_pokemon`).

---

## 13. `TrainerEscapeHistory`

| Column                  | Type                     | Constraints                                                          |
|-------------------------|--------------------------|----------------------------------------------------------------------|
| `id`                    | UUID                     | Primary key, default `uuid4`, not nullable                           |
| `trainer_id`            | UUID                     | Foreign key → `trainers.id`, not nullable                            |
| `source`                | Enum(`EscapeSourceEnum`) | Not nullable                                                         |
| `reason`                | Enum(`EscapeReasonEnum`) | Not nullable                                                         |
| `wild_pokemon_snapshot` | JSON                     | Not nullable                                                         |
| `encounter_id`          | UUID                     | Foreign key → `encounters.id`, nullable, default `None`              |
| `battle_id`             | UUID                     | Foreign key → `battles.id`, nullable, default `None`                 |
| `created_at`            | DateTime                 | Not nullable, default `utcnow`, **never updated after insert**       |

**`EscapeSourceEnum`**
- `ENCOUNTER`
- `BATTLE`

**`EscapeReasonEnum`**
- `RUN_BUTTON`
- `MODAL_CLOSE`
- `PAGE_RELOAD`
- `ROUTE_CHANGE`
- `NEW_ENCOUNTER_OVERRIDE`

**Relationships on `TrainerEscapeHistory`:**
- `trainer` → many-to-one relationship to `Trainer` via foreign key `trainer_id` on `trainer_escape_history`
- `encounter` → many-to-one relationship to `Encounter` via foreign key `encounter_id` on `trainer_escape_history` (optional)
- `battle` → many-to-one relationship to `Battle` via foreign key `battle_id` on `trainer_escape_history` (optional)

---

## 14. `Encounter`

| Column                         | Type                        | Constraints                                                         |
|--------------------------------|-----------------------------|---------------------------------------------------------------------|
| `id`                           | UUID                        | Primary key, default `uuid4`, not nullable                          |
| `trainer_id`                   | UUID                        | Foreign key → `trainers.id`, not nullable                           |
| `wild_pokemon_id`              | UUID                        | Foreign key → `pokemons.id`, not nullable                           |
| `wild_current_hp`              | Integer                     | Not nullable                                                        |
| `wild_max_hp`                  | Integer                     | Not nullable                                                        |
| `wild_pokemon_snapshot`        | JSON                        | Not nullable                                                        |
| `wild_selected_moves_snapshot` | JSON                        | Not nullable, default `[]`                                          |
| `status`                       | Enum(`EncounterStatusEnum`) | Not nullable, default `OPEN`                                        |
| `created_at`                   | DateTime                    | Not nullable, default `utcnow`, **never updated after insert**      |
| `updated_at`                   | DateTime                    | Nullable, default `None`, **only set on second or later updates**   |
| `closed_at`                    | DateTime                    | Nullable, default `None`, **only set when the encounter is closed** |

**`EncounterStatusEnum`**
- `OPEN`
- `PROMOTED_TO_BATTLE`
- `ESCAPED`
- `EXPIRED`

**Relationships on `Encounter`:**
- `trainer` → many-to-one relationship to `Trainer` via foreign key `trainer_id` on `encounters`
- `wild_pokemon` → many-to-one relationship to `Pokemon` via foreign key `wild_pokemon_id` on `encounters`
- `battles` → one-to-many relationship to `Battle` via foreign key `encounter_id` on `battles`

---

## 15. `Battle`

| Column                         | Type                     | Constraints                                                          |
|--------------------------------|--------------------------|----------------------------------------------------------------------|
| `id`                           | UUID                     | Primary key, default `uuid4`, not nullable                           |
| `trainer_id`                   | UUID                     | Foreign key → `trainers.id`, not nullable                            |
| `encounter_id`                 | UUID                     | Foreign key → `encounters.id`, not nullable                          |
| `wild_pokemon_id`              | UUID                     | Foreign key → `pokemons.id`, not nullable                            |
| `active_my_pokemon_id`         | UUID                     | Foreign key → `my_pokemons.id`, nullable, default `None`             |
| `wild_pokemon_snapshot`        | JSON                     | Not nullable                                                         |
| `trainer_party_snapshot`       | JSON                     | Not nullable, default `[]`                                           |
| `selected_my_pokemon_snapshot` | JSON                     | Nullable, default `None`                                             |
| `battle_state`                 | JSON                     | Not nullable, default `{}`                                           |
| `status`                       | Enum(`BattleStatusEnum`) | Not nullable, default `OPEN`                                         |
| `result`                       | Enum(`BattleResultEnum`) | Nullable, default `None`                                             |
| `turn_number`                  | Integer                  | Not nullable, default `0`                                            |
| `is_capture_available`         | Boolean                  | Not nullable, default `False`                                        |
| `created_at`                   | DateTime                 | Not nullable, default `utcnow`, **never updated after insert**       |
| `updated_at`                   | DateTime                 | Nullable, default `None`, **only set on second or later updates**    |
| `finished_at`                  | DateTime                 | Nullable, default `None`, **only set when the battle ends**          |

**`BattleStatusEnum`**
- `OPEN`
- `FINISHED`
- `ESCAPED`
- `ABANDONED`

**`BattleResultEnum`**
- `VICTORY`
- `DEFEAT`
- `ESCAPE`
- `CAPTURE`

**Relationships on `Battle`:**
- `trainer` → many-to-one relationship to `Trainer` via foreign key `trainer_id` on `battles`
- `encounter` → many-to-one relationship to `Encounter` via foreign key `encounter_id` on `battles`
- `wild_pokemon` → many-to-one relationship to `Pokemon` via foreign key `wild_pokemon_id` on `battles`
- `active_my_pokemon` → many-to-one relationship to `MyPokemon` via foreign key `active_my_pokemon_id` on `battles` (optional)
- `events` → one-to-many relationship to `BattleEvent` via foreign key `battle_id` on `battle_events`

---

## 16. `BattleEvent`

| Column            | Type                          | Constraints                                                          |
|-------------------|-------------------------------|----------------------------------------------------------------------|
| `id`              | UUID                          | Primary key, default `uuid4`, not nullable                           |
| `battle_id`       | UUID                          | Foreign key → `battles.id`, not nullable                             |
| `event_type`      | Enum(`BattleEventTypeEnum`)   | Not nullable                                                         |
| `turn_number`     | Integer                       | Not nullable, default `0`                                            |
| `actor_side`      | Enum(`BattleActorSideEnum`)   | Not nullable, default `SYSTEM`                                       |
| `actor_snapshot`  | JSON                          | Nullable, default `None`                                             |
| `target_side`     | Enum(`BattleActorSideEnum`)   | Nullable, default `None`                                             |
| `target_snapshot` | JSON                          | Nullable, default `None`                                             |
| `payload`         | JSON                          | Not nullable, default `{}`                                           |
| `created_at`      | DateTime                      | Not nullable, default `utcnow`, **never updated after insert**       |

**`BattleEventTypeEnum`**
- `ENCOUNTER_STARTED`
- `BATTLE_STARTED`
- `POKEMON_SELECTED`
- `MOVE_SELECTED`
- `ATTACK_HIT`
- `ATTACK_MISSED`
- `TYPE_EFFECTIVENESS`
- `DAMAGE_APPLIED`
- `POKEMON_SWITCHED`
- `POKEMON_FAINTED`
- `CAPTURE_ATTEMPTED`
- `CAPTURE_SUCCEEDED`
- `CAPTURE_FAILED`
- `ESCAPED`
- `LEVEL_UP`
- `BATTLE_FINISHED`

**`BattleActorSideEnum`**
- `TRAINER`
- `WILD`
- `SYSTEM`

**Relationships on `BattleEvent`:**
- `battle` → many-to-one relationship to `Battle` via foreign key `battle_id` on `battle_events`

---

## 17. General Rules
- `created_at` is set only at creation and must never be changed afterwards.
- `updated_at` starts as `None`. It must be set to the current UTC datetime only on the **second or later modification** of the record.
- `deleted_at` must only be set when a soft-delete operation is explicitly triggered. Soft-deleting a record must **not** physically remove the row.

---

