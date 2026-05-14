# Proposta: listagem-e-detalhe-pokemon

## 0. Contexto obrigatório (IMPORTANTE)

Antes de propor ou implementar, ler e respeitar:

- `../docs/architecture-api.md`
- `../docs/architecture-web.md`
- `../machado-api/AGENTS.md`
- `../machado-web/AGENTS.md`

Não criar novos padrões se já existir padrão definido nesses documentos.

---

## 1. Objetivo

Implementar listagem e detalhamento de Pokémon com sincronização automática via PokéAPI e uso de cache.

A feature deve permitir:

- listar Pokémon com paginação e filtros;
- visualizar detalhes de um Pokémon;
- importar dados básicos da PokéAPI quando a base local estiver vazia ou incompleta;
- completar os dados de um Pokémon quando acessado em detalhe;
- utilizar cache Redis para otimizar consultas.

---

## 2. Contexto Atual
- Projeto: machado (monorepo API + WEB)
- Domínio afetado: Pokémon
- Comportamento atual: não existe listagem nem detalhamento
- Problema identificado: ausência de dados locais e dependência total de fonte externa

---

## 3. Comportamento Esperado
- O sistema deve listar Pokémon com paginação e filtros
- O usuário deve conseguir visualizar detalhes completos de um Pokémon
- A API deve sincronizar dados automaticamente com a PokéAPI
- A interface deve exibir informações ricas e normalizadas

---

## 4. Escopo
### 4.1 Dentro do escopo
- Listagem paginada de Pokémon
- Detalhamento completo
- Sincronização com PokéAPI
- Enriquecimento de dados
- Implementação de cache Redis

### 4.2 Fora do escopo
- Autenticação
- Refatorações globais
- Mudanças de arquitetura

---

## 5. Regras de Negócio

### Regra 1 — Sincronização inicial

**Dado que:** a base está vazia
**Quando:** a listagem for chamada
**Então:** importar todos os Pokémon da PokéAPI como INCOMPLETE

---

### Regra 2 — Base parcial

**Dado que:** existem dados mas incompletos
**Quando:** houver divergência de total
**Então:** importar apenas os Pokémon ausentes

---

### Regra 3 — Enriquecimento

**Dado que:** Pokémon com status INCOMPLETE
**Quando:** for acessado no detalhe
**Então:** completar dados e atualizar status

---

### Regra 4 — Cache

**Dado que:** dados já foram consultados
**Quando:** nova requisição ocorrer
**Então:** retornar dados do cache quando válido

---

## 6. Escopo Técnicos

### 6.1 API
Toda regra de negócio deve estar na API.

O frontend deve ser apenas consumidor.

---
#### Funcionalidade
- endpoint de listagem paginada
- endpoint de detalhe
- filtros: nome, ordem, status, tipo

---


#### Fluxos
- base vazia → importar todos
- base parcial → completar
- base completa → usar cache

---

#### 6.1.1 Integrações externas

Endpoints:

https://pokeapi.co/api/v2/pokemon?offset=0&limit=1350
https://pokeapi.co/api/v2/pokemon/{name}
https://pokeapi.co/api/v2/pokemon/{encounter_order}/encounters
https://pokeapi.co/api/v2/pokemon-species/{name}
https://pokeapi.co/api/v2/move/{move_order}
https://pokeapi.co/api/v2/type/{type_order}
https://pokeapi.co/api/v2/ability/{ability_order}
https://pokeapi.co/api/v2/growth-rate/{growth_rate_name}
https://pokeapi.co/api/v2/evolution-chain/{evolution_chain_order}

Regras:

- Usar endpoints para enriquecer o domínio
- Priorizar dados completos
- Evitar chamadas redundantes

---

#### 6.1.2 Enriquecimento de dados

Persistir quando disponível:

- descrição
- altura
- peso
- taxa de captura
- estatísticas (hp, attack, defense, etc)
- experiência
- crescimento
- habitat
- tipos
- habilidades
- movimentos
- cadeia evolutiva
- fraquezas e vantagens

---

#### 6.1.3 Cache
Chaves:

pokemon:meta
pokemon:list:{filters}:{pagination}
pokemon:detail:{id|name}

Regras:

- TTL > 2h
- invalidar após atualização
- manter consistência

---

#### 6.1.4 Consistência e performance
- operações idempotentes
- evitar duplicação
- constraints únicas
- paralelismo controlado
- limitar chamadas externas
- fallback em falhas
- evitar N+1 queries
---

#### 6.1.5 Contrato da API
A API deve ser pensada como fonte única de verdade (single source of truth).
A API deve:
- retornar dados normalizados
- evitar transformação no frontend
- manter consistência de naming

---

#### 6.1.6 Ideia de Modelo de dados

##### **pokemon**:
File location: `machado-api/app/models/pokemon.py`

Create a SQLAlchemy async model named `Pokemon` mapped to the table `pokemons` with the following columns:


| Column                   | Type                      | Constraints                                                          |
|--------------------------|---------------------------|----------------------------------------------------------------------|
| `id`                     | UUID                      | Primary key, default `uuid4`, not nullable                           |
| `name`                   | String                    | Not nullable                                                         |
| `order`                  | Integer                   | Not nullable                                                         |
| `external_image`         | String                    | Not nullable                                                         |
| `status`                 | Enum(`PokemonStatusEnum`) | Not nullable — must always be stored hashed                          |
| `hp`                     | Integer                   | Nullable, default `0`                                                |
| `image_id`               | UUID                      | Foreign key → `pokemon_images.id`, Nullable                          |
| `speed`                  | Integer                   | Nullable, default `0`                                                |
| `height`                 | Integer                   | Nullable, default `0`                                                |
| `weight`                 | Integer                   | Nullable, default `0`                                                |
| `attack`                 | Integer                   | Nullable, default `0`                                                |
| `defense`                | Integer                   | Nullable, default `0`                                                |
| `habitat_id`             | UUID                      | Foreign key → `pokemon_habitat.id`, Nullable                         |
| `shape_id`               | UUID                      | Foreign key → `pokemon_shape.id`, Nullable                           |
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
- `habitat` → many-to-one relationship to `PokemonHabitat` via foreign key `habitat_id` on `pokemons` (optional, nullable)
- `shape` → many-to-one relationship to `PokemonShape` via foreign key `shape_id` on `pokemons` (optional, nullable)
- `moves` → many-to-many relationship to `PokemonMove` via association table `pokemon_pokemon_moves`
- `encounters` → many-to-many relationship to `PokemonEncounter` via association table `pokemon_pokemon_encounters`
- `abilities` → many-to-many relationship to `PokemonAbility` via association table `pokemon_pokemon_abilities`
- `types` → many-to-many relationship to `PokemonType` via association table `pokemon_pokemon_types`
- `evolutions` → many-to-many self-referential relationship on `pokemons` via association table `pokemon_evolutions` (both directions accessible via `pokemon.evolutions`)
- `image` → one-to-one relationship to `PokemonImage` via foreign key `pokemon_id` on `pokemon_images` (optional, nullable)

---

##### **PokemonGrowthRate**:

File location: `machado-api/app/models/pokemon_growth_rate.py`

Create a SQLAlchemy async model named `PokemonGrowthRate` mapped to the table `pokemon_growth_rates`:

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

##### **PokemonMove**:

File location: `machado-api/app/models/pokemon_move.py`

Create a SQLAlchemy async model named `PokemonMove` mapped to the table `pokemon_moves`:

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

---

##### **PokemonAbility**:
File location: `machado-api/app/models/pokemon_ability.py`

Create a SQLAlchemy async model named `PokemonAbility` mapped to the table `pokemon_abilities`:

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

##### **PokemonType**:
File location: `machado-api/app/models/pokemon_type.py`

Create a SQLAlchemy async model named `PokemonType` mapped to the table `pokemon_types`:

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

##### **PokemonImage**:
File location: `machado-api/app/models/pokemon_image.py`

Create a SQLAlchemy async model named `PokemonImage` mapped to the table `pokemon_images`:

| Column       | Type     | Constraints                                                   | description                                                                                            |
|--------------|----------|---------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| `id`         | UUID     | Primary key, default `uuid4`, not nullable                    | -------------                                                                                          |
| `pokemon_id` | UUID     | Foreign key → `pokemon.id`, Nullable                          | -------------                                                                                          |
| `url`        | String   | Not nullable                                                  | -------------                                                                                          |
| `order`      | Integer  | Not nullable                                                  | -------------                                                                                          |
| `source`     | String   | Not nullable                                                  | image family such as `default`, `official-artwork`, `home`, `dream-world`, `showdown`, or `version`.   |
| `variant`    | String   | Not nullable                                                  | sprite variant such as `front_default`, `front_shiny`, `front_female`, `back_default`, or `back_shiny` |
| `generation` | String   | nullable                                                      | generation name for versioned sprites.                                                                 |
| `game`       | String   | nullable                                                      | game/version name for versioned sprites.                                                               |
| `media_type` | String   | nullable                                                      | media type hint such as `png`, `svg`, or `gif`.                                                        |
| `is_primary` | Boolean  | Not nullable                                                  | flag indicating the preferred catalog image                                                            |
| `images`     | String   | Not nullable                                                  | list of all url image                                                                                  |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert    | -------------                                                                                          |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates | -------------                                                                                          |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete             | -------------                                                                                          |

**Relationships on `PokemonImage`:**
- `pokemon` → one-to-one relationship to `Pokemon` via foreign key `pokemon_id`

---

##### **PokemonEncounter**:

File location: `machado-api/app/models/pokemon_encounter.py`

Create a SQLAlchemy async model named `PokemonEncounter` mapped to the table `pokemon_encounters`:

| Column       | Type     | Constraints                                                   |
|--------------|----------|---------------------------------------------------------------|
| `id`         | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`        | String   | Not nullable                                                  |
| `order`      | Integer  | Not nullable                                                  |
| `created_at` | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at` | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at` | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonEncounter`:**
- `locations` → many-to-many relationship to `PokemonLocation` via association table `pokemon_encounter_location`

---

##### **PokemonLocation**:
File location: `machado-api/app/models/pokemon_location.py`

Create a SQLAlchemy async model named `PokemonLocation` mapped to the table `pokemon_locations`:

| Column                 | Type     | Constraints                                                   |
|------------------------|----------|---------------------------------------------------------------|
| `id`                   | UUID     | Primary key, default `uuid4`, not nullable                    |
| `pokemon_encounter_id` | UUID     | Foreign key → `pokemon_encounter.id`, Not Nullable            |
| `url`                  | String   | Not nullable                                                  |
| `name`                 | String   | Not nullable                                                  |
| `order`                | Integer  | Not nullable                                                  |
| `chance`               | Integer  | Not nullable                                                  |
| `condition`            | String   | Not nullable                                                  |
| `condition_url`        | String   | Not nullable                                                  |
| `max_level`            | Integer  | Not nullable                                                  |
| `min_level`            | Integer  | Not nullable                                                  |
| `method`               | String   | Not nullable                                                  |
| `created_at`           | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`           | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`           | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonLocation`:**
- `encounter` → many-to-one relationship to `PokemonEncounter` via foreign key `pokemon_encounter_id` on `pokemon_encounters`
---

---

##### **PokemonHabitat**:
File location: `machado-api/app/models/pokemon_habitat.py`

Create a SQLAlchemy async model named `PokemonHabitat` mapped to the table `pokemon_habitats`:

| Column        | Type     | Constraints                                                   |
|---------------|----------|---------------------------------------------------------------|
| `id`          | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`         | String   | Not nullable                                                  |
| `name`        | String   | Unique, not nullable                                          |
| `created_at`  | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`  | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`  | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonHabitat`:**
- `pokemons` → one-to-many relationship to `Pokemon` via foreign key `habitat_id` on `pokemons`

---

##### **PokemonShape**:
File location: `machado-api/app/models/pokemon_shape.py`

Create a SQLAlchemy async model named `PokemonShape` mapped to the table `pokemon_shapes`:

| Column        | Type     | Constraints                                                   |
|---------------|----------|---------------------------------------------------------------|
| `id`          | UUID     | Primary key, default `uuid4`, not nullable                    |
| `url`         | String   | Not nullable                                                  |
| `name`        | String   | Unique, not nullable                                          |
| `created_at`  | DateTime | Not nullable, default `utcnow`, never updated after insert    |
| `updated_at`  | DateTime | Nullable, default `None`, only set on second or later updates |
| `deleted_at`  | DateTime | Nullable, default `None`, only set on soft-delete             |

**Relationships on `PokemonShape`:**
- `pokemons` → one-to-many relationship to `Pokemon` via foreign key `habitat_id` on `pokemons`

---

---

#### 6.1.7 External API Client
File location: `machado-api/app/infrastructure/external_api/pokeapi_client.py`

Create a dedicated async HTTP client class `PokeApiClient` responsible for all calls to `https://pokeapi.co`. This class must be injected via FastAPI dependency injection and must NOT be mixed into domain services or repositories. Methods:

This service should implement the following methods:
- `list_pokemon(offset: int = 0, limit: int = 1350) -> list[PokemonExternalBaseSchema]` — returns the list of pokemons from the external API with pagination support. The default limit should be set to 1350 to retrieve all pokemons in one call.
- `get_pokemon(name: str) -> PokemonExternalSchema` — returns the pokemon details by name from the external API.
- `get_pokemon_species(name_or_id: str) -> PokemonExternalSpecieSchema` — returns the pokemon species details by name or id from the external API.
- `get_move(order: int) -> PokemonExternalMoveSchema` — returns the pokemon move details by order from the external API.
- `get_type(order: int) -> PokemonExternalTypeSchema` — returns the pokemon type details by order from the external API.
- `get_growth_rate(order: int) -> PokemonExternalGrowthRateSchema` — returns the pokemon growth rate details by order from the external API.
- `get_evolution_chain(url: str) -> PokemonExternalEvolutionSchema` — returns the pokemon evolution chain details by url from the external API.
- `get_pokemon_by_name(pokemon: PokemonSchema) -> PokemonByNameResponseSchema` — returns the pokemon details by name from the external API.
- `get_encounter(url: str) -> PokemonExternalEncounterSchema` — returns the pokemon encounter details by url from the external API.


### 6.2 WEB

#### 6.2.1 Páginas
- listagem de Pokémon
- detalhe do Pokémon

---

#### 6.2.2 Componentes
- cards de Pokémon (Usar Componente de Card existente, se ncessário melhorar.)
- filtros (Usar Componente de Filtro existente, se necessário melhorar.)
- badges (Usar Componente de Badges existente, se necessário melhorar.)
- timeline de evolução

Estados:
- loading
- erro
- vazio
- sucesso

---

#### 6.2.3 Renderização
<Badge style={{ color: type.text_color, backgroundColor: type.background_color }}>
    {type.name}
</Badge>

---

#### 6.2.4 Interações
- paginação (Usar Component de Paginação existente, se necessário melhorar.)
- clique para detalhe
- botão “ver mais” para movimentos
- carregamento incremental

---

## 7. Restrições Arquiteturais
- Não criar nova arquitetura
- Não duplicar lógica
- Reutilizar padrões existentes
- Toda regra de negócio na API
- Toda nova tabela criada no banco de dados deve suportar soft delete.
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto.
- Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`.
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário.
- Ao criar um repository no projeto api sempre consumir o BaseRepository `from app.core.repository.base import BaseRepository`.
- Ao criar um service no projeto api sempre consumir o BaseService `from app.core.service.base import BaseService`.
- Ao criar uma tabela sempre crie um arquivo por tabela.
- Sempre Crie um domain por entidade. 1 domain = 1 tabela.
- Cada domain deve ter no minimo um `repository.py`, `service.py` e um `schema.py`.
- Se houver necessidade de regras específicas criar no arquivo `business.py`
- Se houver `endpoints` criar arquivo `routes.py`
---

## 8. Critérios de aceite
- [ ] listagem funcional
- [ ] filtros funcionando
- [ ] sincronização automática
- [ ] cache funcionando
- [ ] sem duplicação
- [ ] detalhe completo
- [ ] UI consistente
- [ ] sem regressões
- [ ] Cobertura de testes adequada ao padrão do projeto
- [ ] Testes cobrindo fluxos principais e cenários críticos
- [ ] Novas tabelas possuem suporte a soft delete
- [ ] Consultas padrão ignoram registros deletados logicamente
- [ ] Nenhuma exclusão física foi implementada sem justificativa