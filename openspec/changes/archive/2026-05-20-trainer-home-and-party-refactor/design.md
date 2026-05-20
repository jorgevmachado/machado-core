# Design: trainer-home-and-party-refactor

## Objetivo
Descrever como preservar o comportamento atual enquanto redistribui responsabilidades: extrair `party` de `trainer-exploration` para `trainer_party` e mover a agregação de `home` para o domínio `trainer`.

---

## 1. Estrutura de Domínios

### 1.1 `trainer-party` (novo)

**Localização:** `machado-api/app/domain/trainer/trainer_party/`

**Estrutura:**
```
trainer_party/
├── __init__.py              # exports
├── service.py               # TrainerPartyService (extracted from trainer_exploration)
├── repository.py            # TrainerPartyRepository (reusos methods de trainer_exploration.repository)
├── schema.py                # TrainerPartyMemberSchema, UpdateTrainerPartySchema (moved from trainer_exploration)
├── business.py              # validate_party_selection, MAX_PARTY_SIZE (moved from trainer_exploration)
└── route.py                 # PUT/GET /trainer/party
```

**Responsabilidades:**
- `update_party(current_user, payload: UpdateTrainerPartySchema)` → list[TrainerPartyMemberSchema]
- `get_party(current_user)` → list[TrainerPartyMemberSchema]
- Validações: MAX 6, sem duplicatas
- Cache: `trainer:party:{trainer_id}`
- Soft delete via `deleted_at`

**Reutilização:**
- Model: `trainer_party.py` (já existe)
- Lógica de validação/serialização atual
- Testes existentes de party como base para adaptação

---

### 1.2 `trainer` como agregador de Home

**Localização:** `machado-api/app/domain/trainer/`

**Estrutura:**
```
trainer/
├── service.py               # TrainerService agrega get_home()
├── route.py                 # GET /trainer/home
└── tests/...                # testes do use case agregador
```

**Responsabilidades:**
- `get_home(current_user)` → `TrainerHomeSchema`
  - Resolve o trainer atual
  - Orquestra party, encounter ativo e latest discoveries por interfaces públicas explícitas
  - Preserva exatamente o payload atual: `trainer`, `active_encounter`, `party`, `latest_discoveries`
- Cache: `trainer:home:{trainer_id}`
- Invalidação: após alterações em party, encounter ou pokedex

**Decisão arquitetural:**
- Não criar `trainer_home` como domínio novo nesta fase
- `home` é tratado como um use case agregador do agregado `trainer`, não como um subdomínio separado

---

### 1.3 `trainer-exploration` (refatorado)

**Localização:** `machado-api/app/domain/trainer/trainer_exploration/`

**Mantém:**
- `initialize_for_trainer()` 
- `list_encounters()` 
- `select_active_encounter()` 
- `walk()` 
- `create_event()`, `find_active_trainer_encounter()`
- Cache apenas para encounters e eventos

**Remove:**
- `update_party()` → mover para `trainer_party`
- `get_party()` → mover para `trainer_party`
- `get_home()` → mover para `trainer`
- Schemas, business rules, repository methods de party → mover para `trainer_party`
- Queries de latest discoveries → sair de `trainer_exploration`

**Ajustes:**
- Repository: fica restrito a encounter/event
- Service: não agrega mais home nem gerencia party
- Cache invalidation: invalida encounters localmente e aciona interface pública de invalidação da home

---

## 2. Fluxos de Dados

### Before (acoplado)
```
GET /api/trainer/home
  ↓
TrainerExplorationService.get_home()
  ├─ await repository.find_active_encounter()
  ├─ await repository.list_active_party()  ← party aqui
  ├─ await repository.list_latest_discoveries()
  └─ return TrainerHomeSchema
```

### After (desacoplado)
```
GET /trainer/home
  ↓
TrainerService.get_home()
  ├─ await self.get_by_user_id()
  ├─ await trainer_party_service.get_party_by_trainer_id()
  ├─ await trainer_exploration_service.get_active_encounter_by_trainer_id()
  ├─ await pokedex_service.list_latest_discoveries()
  └─ return TrainerHomeSchema (agregado, cache)

PUT /trainer/party
  ↓
TrainerPartyService.update_party()
  ├─ validate_party_selection()
  ├─ await repository.list_owned_my_pokemon()
  ├─ await repository.soft_delete_active_party()
  ├─ await repository.create_party()
  ├─ await trainer_service.invalidate_home_cache(trainer_id)
  └─ return list[TrainerPartyMemberSchema]
```

---

## 3. Cache & Invalidation

**Keys:**
- `trainer:home:{trainer_id}` — em `trainer.service`
- `trainer:party:{trainer_id}` — em `trainer_party.service`
- `trainer:encounters:{trainer_id}` — em `trainer_exploration.service` (mantido)

**Invalidation rules:**
| Evento | Chaves invalidadas |
|--------|--------------------|
| update_party() | `trainer:home:{id}`, `trainer:party:{id}` |
| select_active_encounter() | `trainer:home:{id}`, `trainer:encounters:{id}` |
| walk() (pokeball) | `trainer:home:{id}` |
| pokedex entry created | `trainer:home:{id}` |

**Implementação:**
- `TrainerService.invalidate_home_cache(trainer_id)` é a interface pública de invalidação da home
- `TrainerPartyService` chama essa interface após mudanças de party
- `TrainerExplorationService` chama essa interface após mudanças de encounter relevantes
- `PokedexService` chama essa interface quando latest discoveries afetarem a home

---

## 4. Reutilização Concreta

| Componente | Origem | Destino | Tipo |
|------------|--------|---------|------|
| `TrainerPartyMemberSchema` | trainer_exploration/schema.py | trainer_party/schema.py | copy as-is |
| `UpdateTrainerPartySchema` | trainer_exploration/schema.py | trainer_party/schema.py | copy as-is |
| `MAX_PARTY_SIZE` | trainer_exploration/business.py | trainer_party/business.py | copy as-is |
| `validate_party_selection()` | trainer_exploration/business.py | trainer_party/business.py | copy as-is |
| `list_active_party()` | trainer_exploration/repository | trainer_party/repository | mover/adaptar |
| `create_party()` | trainer_exploration/repository | trainer_party/repository | mover/adaptar |
| `soft_delete_active_party()` | trainer_exploration/repository | trainer_party/repository | mover/adaptar |
| `list_owned_my_pokemon()` | trainer_exploration/repository | trainer_party/repository | mover/adaptar |
| `TrainerHomeSchema` | trainer_exploration/schema.py | trainer/schema or reuse import | preservar contrato |
| latest discoveries access | trainer_exploration/repository | pokedex service/repository | mover para boundary correto |

---

## 5. Contrato API

**GET /trainer/home** (novo endpoint canônico no backend)
```json
{
  "trainer": {
    "id": "uuid",
    "pokeballs": 10,
    "capture_rate": 0.5
  },
  "active_encounter": {
    "id": "uuid",
    "pokemon_encounter": { ... }
  },
  "party": [
    {
      "id": "uuid",
      "slot": 1,
      "my_pokemon": { ... }
    }
  ],
  "latest_discoveries": [ ... ]
}
```

**PUT /trainer/party** (novo endpoint canônico no backend)
```json
// request
{
  "my_pokemon_ids": ["uuid1", "uuid2", "uuid3"]
}

// response: list[TrainerPartyMemberSchema]
```

**GET /trainer/party**
```json
// Same as party array in /home
[...]
```

**Compatibilidade Web**
- O BFF Next continua expondo `/api/trainer/home` e `/api/trainer/party`
- O client web passa a consumir `/trainer/home` e `/trainer/party` na API backend
- Rotas antigas `/trainer/exploration/home` e `/trainer/exploration/party` podem existir temporariamente como aliases durante a migração, mas deixam de ser canônicas

---

## 6. Model de Dados

**Nenhuma mudança no banco:**
- `trainer_party` table já existe (migration `trainer_exploration.py`)
- Schema `trainer_id`, `my_pokemon_id`, `slot`, `is_active`, `created_at`, `updated_at`, `deleted_at`
- Indexes: `(trainer_id, deleted_at)`, `(trainer_id, slot, is_active)` existem

---

## 7. Padrões & Convenções

- Seguir padrão existente de BaseService, repository, schema, route
- Cache: usar `CacheService` pattern do projeto
- Logging: usar `LoggingParams` pattern
- Soft delete: ignorar registros com `deleted_at` em queries padrão
- Tests: usar fixtures e mocks do projeto
- `trainer_exploration/repository.py` não deve continuar consultando `TrainerParty` ou `Pokedex`
- `TrainerService` agrega `home`, mas não absorve regras de party ou exploration
- Evitar ciclos de dependência: expor interfaces públicas mínimas entre services

## 8. Interfaces Públicas Internas

Para viabilizar o desacoplamento sem consultas cruzadas indevidas em repository:

- `TrainerPartyService.get_party_by_trainer_id(trainer_id: UUID) -> list[TrainerPartyMemberSchema]`
- `TrainerExplorationService.get_active_encounter_by_trainer_id(trainer_id: UUID) -> TrainerEncounterSchema | None`
- `PokedexService.list_latest_discoveries(trainer_id: UUID, limit: int = 3) -> list[PokedexSchema]`
- `TrainerService.invalidate_home_cache(trainer_id: str) -> None`

## 9. Riscos e Mitigação

| Risco | Mitigação |
|-------|-----------|
| `TrainerService` virar god object | Limitar `home` a orquestração de leitura; party e exploration continuam com suas regras próprias |
| Cache invalidation breaks | Documentar regras claras; testar cada cenário |
| Frontend quebra (contrato muda) | Preservar shape de `GET /trainer/home` e manter BFF estável |
| Regressões em exploração | Manter testes existentes de walk/encounters; rodá-los antes de merge |
| Boundary de repository continuar incorreto | Mover queries de party e latest discoveries para os domínios corretos |
