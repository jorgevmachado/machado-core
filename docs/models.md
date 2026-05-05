# Database Models

Este documento descreve os models ORM atualmente presentes em `machado-api/app/models`.

## Convenções

- Todos os models usam `@table_registry.mapped_as_dataclass`.
- `id` usa UUID com `uuid4`, salvo nas tabelas de associação compostas.
- `created_at` usa `utcnow`.
- `updated_at` inicia como `None`.
- `deleted_at` representa soft delete quando existir.
- Relacionamentos usam `default_lazy` (`selectin`) quando aplicável.

## Enums

**`GenderEnum`**
- `MALE`
- `FEMALE`
- `OTHER`

**`StatusEnum`**
- `ACTIVE`
- `INACTIVE`
- `INCOMPLETE`

**`RoleEnum`**
- `USER`
- `ADMIN`

**`PokemonStatusEnum`**
- `COMPLETE`
- `INCOMPLETE`

---

## 1. `User` (`users`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `name` | String | Not nullable |
| `email` | String | Unique, not nullable |
| `username` | String | Unique, not nullable |
| `gender` | Enum(`GenderEnum`) | Not nullable |
| `password` | String | Not nullable, must be hashed |
| `date_of_birth` | DateTime | Not nullable |
| `status` | Enum(`StatusEnum`) | Not nullable, default `INACTIVE` |
| `role` | Enum(`RoleEnum`) | Not nullable, default `USER` |
| `total_authentications` | Integer | Nullable, default `0` |
| `authentication_success` | Integer | Nullable, default `0` |
| `authentication_failures` | Integer | Nullable, default `0` |
| `last_authentication_at` | DateTime | Nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Rules**
- `password` must never be persisted in plain text.
- Use `app.core.security.get_password_hash` before saving passwords.

---

## 2. `Trainer` (`trainers`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `user_id` | UUID | Foreign key -> `users.id`, not nullable |
| `pokeballs` | Integer | Not nullable |
| `capture_rate` | Integer | Not nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

---

## 3. `Pokemon` (`pokemons`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `external_image` | String | Not nullable |
| `status` | Enum(`PokemonStatusEnum`) | Not nullable, default `INCOMPLETE` |
| `hp` | Integer | Nullable, default `0` |
| `speed` | Integer | Nullable, default `0` |
| `height` | Integer | Nullable, default `0` |
| `weight` | Integer | Nullable, default `0` |
| `attack` | Integer | Nullable, default `0` |
| `defense` | Integer | Nullable, default `0` |
| `special_attack` | Integer | Nullable, default `0` |
| `special_defense` | Integer | Nullable, default `0` |
| `base_experience` | Integer | Nullable, default `0` |
| `description` | Text | Nullable |
| `capture_rate` | Integer | Nullable, default `0` |
| `is_baby` | Boolean | Nullable, default `False` |
| `is_mythical` | Boolean | Nullable, default `False` |
| `is_legendary` | Boolean | Nullable, default `False` |
| `gender_rate` | Integer | Nullable, default `0` |
| `hatch_counter` | Integer | Nullable, default `0` |
| `base_happiness` | Integer | Nullable, default `0` |
| `evolution_chain` | String | Nullable |
| `evolves_from_species` | String | Nullable |
| `has_gender_differences` | Boolean | Nullable, default `False` |
| `growth_rate_id` | UUID | Foreign key -> `pokemon_growth_rates.id`, nullable |
| `habitat_id` | UUID | Foreign key -> `pokemon_habitats.id`, nullable |
| `shape_id` | UUID | Foreign key -> `pokemon_shapes.id`, nullable |
| `images_id` | UUID | Foreign key -> `pokemon_images.id`, nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `growth_rate` -> many-to-one `PokemonGrowthRate`
- `habitat` -> many-to-one `PokemonHabitat`
- `shape` -> many-to-one `PokemonShape`
- `images` -> many-to-one `PokemonImage`
- `types` -> many-to-many `PokemonType` through `pokemon_type_links`
- `moves` -> many-to-many `PokemonMove` through `pokemon_move_links`
- `abilities` -> many-to-many `PokemonAbility` through `pokemon_ability_links`
- `encounters` -> one-to-many `PokemonEncounter`
- `evolutions` -> self many-to-many through `pokemon_evolution_links`

**Rules**
- Initial sync stores only `name`, `order`, `external_image`, and `INCOMPLETE`.
- Detail access enriches incomplete records and changes `status` to `COMPLETE`.
- `external_image` is derived from the PokeAPI list URL order using four digits.

---

## 4. `PokemonType` (`pokemon_types`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `badge_url` | String | Not nullable |
| `badge_icon_url` | String | Not nullable |
| `badge_shield_url` | String | Not nullable |
| `badge_legends_url` | String | Not nullable |
| `badge_legend_icon_url` | String | Not nullable |
| `badge_shield_icon_url` | String | Not nullable |
| `status` | Enum(`PokemonStatusEnum`) | Not nullable, default `INCOMPLETE` |
| `text_color` | String | Not nullable, default `#111827` |
| `background_color` | String | Not nullable, default `#E5E7EB` |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemons` -> many-to-many `Pokemon` through `pokemon_type_links`
- `weaknesses` -> self many-to-many through `pokemon_type_weaknesses`
- `strengths` -> self many-to-many through `pokemon_type_strengths`

---

## 5. `PokemonAbility` (`pokemon_abilities`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `slot` | Integer | Not nullable |
| `effect` | Text | Not nullable |
| `flavor_text` | Text | Not nullable |
| `short_effect` | Text | Not nullable |
| `is_hidden` | Boolean | Not nullable, default `False` |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemons` -> many-to-many `Pokemon` through `pokemon_ability_links`

---

## 6. `PokemonMove` (`pokemon_moves`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `pp` | Integer | Not nullable |
| `url` | String | Not nullable |
| `type` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `power` | Integer | Not nullable |
| `target` | String | Not nullable |
| `effect` | Text | Not nullable |
| `priority` | Integer | Not nullable |
| `accuracy` | Integer | Not nullable |
| `flavor_text` | Text | Not nullable |
| `short_effect` | Text | Not nullable |
| `damage_class` | String | Not nullable |
| `effect_chance` | Integer | Nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemons` -> many-to-many `Pokemon` through `pokemon_move_links`

---

## 7. `PokemonImage` (`pokemon_images`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `order` | Integer | Unique, not nullable |
| `images` | Text | Not nullable; JSON string with treated image URLs |
| `back_image` | String | Not nullable |
| `front_image` | String | Not nullable |
| `back_source` | String | Not nullable, default `back_default` |
| `front_source` | String | Not nullable, default `front_default` |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Rules**
- One `Pokemon` references the selected image set through `Pokemon.images_id`.
- `PokemonImageSchema` normalizes `images` from JSON string to `list[str]` for API responses.

---

## 8. `PokemonGrowthRate` (`pokemon_growth_rates`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `formula` | Text | Not nullable |
| `description` | Text | Not nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemons` -> one-to-many `Pokemon`

---

## 9. `PokemonHabitat` (`pokemon_habitats`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemons` -> one-to-many `Pokemon`

---

## 10. `PokemonShape` (`pokemon_shapes`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemons` -> one-to-many `Pokemon`

---

## 11. `PokemonEncounter` (`pokemon_encounters`)

| Column | Type | Constraints |
|---|---|---|
| `id` | UUID | Primary key, default `uuid4` |
| `url` | String | Not nullable |
| `name` | String | Unique, not nullable |
| `order` | Integer | Unique, not nullable |
| `chance` | Integer | Not nullable |
| `method` | String | Not nullable |
| `version` | String | Not nullable |
| `min_level` | Integer | Not nullable |
| `max_level` | Integer | Not nullable |
| `condition` | String | Not nullable |
| `max_chance` | Integer | Not nullable |
| `pokemon_id` | UUID | Foreign key -> `pokemons.id`, not nullable |
| `created_at` | DateTime | Not nullable |
| `updated_at` | DateTime | Nullable |
| `deleted_at` | DateTime | Nullable |

**Relationships**
- `pokemon` -> many-to-one `Pokemon`

---

## Association Tables

| Table | Columns | Purpose |
|---|---|---|
| `pokemon_type_links` | `pokemon_id`, `type_id` | Pokemon <-> Type |
| `pokemon_ability_links` | `pokemon_id`, `ability_id` | Pokemon <-> Ability |
| `pokemon_move_links` | `pokemon_id`, `move_id` | Pokemon <-> Move |
| `pokemon_evolution_links` | `pokemon_id`, `evolution_id` | Pokemon <-> Pokemon evolutions |
| `pokemon_type_weaknesses` | `pokemon_type_id`, `pokemon_type_weakness_id` | Type weaknesses |
| `pokemon_type_strengths` | `pokemon_type_id`, `pokemon_type_strength_id` | Type strengths |

All association columns are foreign keys with `ondelete='CASCADE'` and compose the primary key.
