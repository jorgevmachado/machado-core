# Tasks: trainer-home-and-party-refactor

## Dependências

```
1. audit-trainer-exploration
   ↓
2. create-trainer-party-domain
   ↓
3. extract-party-logic-from-exploration
   ├─→ 4. move-home-aggregation-to-trainer
   │
   └─→ 5. update-trainer-exploration-service
       ↓
6. update-web-layer (BFF + hooks)
   ↓
7. validation-and-tests
   ↓
8. docs-and-cleanup
```

---

## Task 1: audit-trainer-exploration
**Dependencies:** none

**Objective:** Mapear exatamente o que sai de `trainer_exploration`

**Steps:**
- [x] Listar todas as funções públicas em `trainer_exploration/service.py`
- [x] Marcar quais são **party** (update_party, get_party)
- [x] Marcar quais são **home** (get_home)
- [x] Marcar quais são **exploration** pura (walk, initialize_for_trainer, select_active_encounter, list_encounters)
- [x] Mapear testes por categoria
- [x] Mapear repository methods por categoria
- [x] Mapear rotas backend antigas `/trainer/exploration/home` e `/trainer/exploration/party`

**Deliverable:** documento textual ou comentários no código

---

## Task 2: create-trainer-party-domain
**Dependencies:** audit-trainer-exploration

**Objective:** Criar estrutura vazia de `trainer_party` domínio

**Steps:**
- [x] Criar diretório `machado-api/app/domain/trainer/trainer_party/`
- [x] Criar `__init__.py` vazio
- [x] Criar `service.py` com classe vazia `TrainerPartyService(BaseService)`
- [x] Criar `repository.py` vazio
- [x] Criar `schema.py` vazio
- [x] Criar `business.py` vazio
- [x] Criar `route.py` vazio
- [x] Adicionar exports mínimos necessários do novo domínio
- [x] Criar testes em `machado-api/tests/app/domain/trainer_party/`

**Deliverable:** diretório com struktura base

---

## Task 3: extract-party-logic-from-exploration
**Dependencies:** create-trainer-party-domain

**Objective:** Mover código de party de exploration → party

**Steps:**

### 3.1 Schema e Business
- [x] Copiar `TrainerPartyMemberSchema` de `trainer_exploration/schema.py` para `trainer_party/schema.py`
- [x] Copiar `UpdateTrainerPartySchema` de `trainer_exploration/schema.py` para `trainer_party/schema.py`
- [x] Copiar `MAX_PARTY_SIZE`, `validate_party_selection()` de `trainer_exploration/business.py` para `trainer_party/business.py`
- [x] Atualizar imports em ambos locais

### 3.2 Repository
- [x] Copiar métodos `list_active_party()`, `create_party()`, `soft_delete_active_party()` de `trainer_exploration/repository.py` para `trainer_party/repository.py`
- [x] Copiar `list_owned_my_pokemon()` para `trainer_party/repository.py`
- [x] **NÃO remover de exploration** ainda (feito em task 5)
- [x] Criar `TrainerPartyRepository(BaseRepository)` seguindo pattern do projeto

### 3.3 Service
- [x] Copiar `update_party()` de `trainer_exploration/service.py` para `trainer_party/service.py`
- [x] Copiar `get_party()` de `trainer_exploration/service.py` para `trainer_party/service.py`
- [x] Copiar métodos helper `_party_key()`, `to_party_schema()` 
- [x] Substituir invalidação direta da home por chamada explícita a `TrainerService.invalidate_home_cache()`
- [x] Adaptar imports e references (já não herda de TrainerExplorationService)
- [x] Expor método interno `get_party_by_trainer_id(trainer_id)`

### 3.4 Route
- [x] Criar route `PUT /trainer/party` que chama `trainer_party_service.update_party()`
- [x] Criar route `GET /trainer/party` que chama `trainer_party_service.get_party()`
- [x] Seguir padrão de imports e autenticação

### 3.5 Tests
- [x] Criar `machado-api/tests/app/domain/trainer_party/test_trainer_party_service.py`
- [x] Criar `machado-api/tests/app/domain/trainer_party/test_trainer_party_route.py`
- [x] Adaptar imports (agora usam `trainer_party` service)
- [x] Cobrir validação de limite, duplicidade, soft delete e cache invalidation
- [x] Rodar: `pytest machado-api/tests/app/domain/trainer_party/`

**Deliverable:** `trainer_party/` completo com testes verdes

---

## Task 4: move-home-aggregation-to-trainer
**Dependencies:** create-trainer-party-domain

**Objective:** Mover a agregação de home para `TrainerService`

**Steps:**

### 4.1 Schema
- [x] Preservar `TrainerHomeSchema` com os campos `trainer`, `active_encounter`, `party`, `latest_discoveries`
- [x] Decidir se o schema permanece em `trainer_exploration/schema.py` temporariamente ou é movido para `trainer/schema.py`

### 4.2 Service
- [x] Mover `get_home()` para `TrainerService`
- [x] Criar `TrainerService.invalidate_home_cache(trainer_id)`
- [x] Adicionar/ajustar dependências explícitas:
  - `trainer_party_service.get_party_by_trainer_id()`
  - `trainer_exploration_service.get_active_encounter_by_trainer_id()`
  - `pokedex_service.list_latest_discoveries()`
- [x] Garantir que `TrainerService` agregue sem consultar repository incorreto
- [x] Implementar cache `trainer:home:{trainer_id}` em `TrainerService`

### 4.3 Route
- [x] Registrar `GET /trainer/home` em `app/domain/trainer/route.py`
- [x] Manter temporariamente compatibilidade para consumidores antigos, se necessário

### 4.4 Tests
- [x] Criar `machado-api/tests/app/domain/trainer/test_trainer_home_service.py`
- [x] Criar `machado-api/tests/app/domain/trainer/test_trainer_home_route.py`
- [x] Mock das dependências (`trainer_party_service`, `trainer_exploration_service`, `pokedex_service`)
- [x] Testar cache hit/miss
- [x] Validar shape exato do payload (`active_encounter`, `latest_discoveries`)

**Deliverable:** `TrainerService.get_home()` implementado com testes

---

## Task 5: update-trainer-exploration-service
**Dependencies:** extract-party-logic-from-exploration, move-home-aggregation-to-trainer

**Objective:** Reduzir exploration removendo party e home, delegando conforme necessário

**Steps:**

### 5.1 Remove from Service
- [x] Remover `update_party()` (já em trainer_party)
- [x] Remover `get_party()` (já em trainer_party)
- [x] Remover `get_home()` (agora em trainer)
- [x] Remover helpers: `_party_key()`, `to_party_schema()`
- [x] Remover `party_cache_service` e `home_cache_service`
- [x] **Manter:** `encounter_cache_service`
- [x] Adicionar método público mínimo `get_active_encounter_by_trainer_id(trainer_id)`

### 5.2 Remove from Repository
- [x] **Verificar:** `list_active_party()`, `create_party()`, `soft_delete_active_party()`, `list_owned_my_pokemon()` — são usados **apenas** por party service?
  - Se sim: remover
  - Se não: manter
- [x] Remover `list_latest_discoveries()` de `trainer_exploration/repository.py`
- [x] Garantir que `trainer_exploration/repository.py` fique restrito a encounter/event

### 5.3 Update Service Dependencies
- [x] Evitar dependência nova de `trainer_party_service`
- [x] Limitar dependências a orquestração mínima para invalidação de home

### 5.4 Update Cache Invalidation
- [x] `_invalidate_cache()` agora apenas invalida encounters
- [x] Chamar `TrainerService.invalidate_home_cache()` após `walk()`, `select_active_encounter()` e `initialize_for_trainer()` quando aplicável
- [x] Documentar ownership das chaves de cache

### 5.5 Update Tests
- [x] Remover testes de `update_party()`, `get_party()`, `get_home()` (já em domínios específicos)
- [x] Manter/ajustar testes de walk, encounters
- [x] Criar cobertura para `get_active_encounter_by_trainer_id()`
- [x] Rodar: `pytest machado-api/tests/app/domain/trainer_exploration/`

**Deliverable:** exploration service reduzido, testes verdes

---

## Task 6: update-web-layer
**Dependencies:** update-trainer-exploration-service

**Objective:** Adaptar web para chamar novos endpoints

**Steps:**

### 6.1 BFF Routes
- [x] Revisar `machado-web/app/api/trainer/home/route.ts`
  - Mantém `/api/trainer/home` no BFF
  - Confirma que o client backend passa a chamar `/trainer/home`
- [x] Revisar `machado-web/app/api/trainer/party/route.ts`
  - Mantém `/api/trainer/party` no BFF
  - Confirma que o client backend passa a chamar `/trainer/party`

### 6.2 Hooks
- [x] `useTrainerHome()` — sem mudança, mesma URL `/api/trainer/home`, mesmo contrato
- [x] `useTrainerParty()` — verificar se existe; se não, pode deixar como-é ou criar

### 6.3 Client Service
- [x] Atualizar `machado-web/app/ui/features/trainer/services/service/service.ts`
  - `home()` → `/trainer/home`
  - `updateParty()` → `/trainer/party`
- [x] Decidir se haverá leitura `GET /trainer/party` no client nesta fase

### 6.4 Tests
- [x] Rodar testes web: `yarn test` em `machado-web/`
- [x] Ajustar testes que validam as URLs antigas `/trainer/exploration/home` e `/trainer/exploration/party`
- [x] Verificar se há regressões

**Deliverable:** BFF routes e hooks funcionando

---

## Task 7: validation-and-tests
**Dependencies:** update-web-layer

**Objective:** Verificar tudo está funcionando sem regressões

**Steps:**

### 7.1 Backend Tests
- [x] `cd machado-api && make test` (ou `pytest`)
- [x] Verificar cobertura de testes
- [x] Verificar soft delete behavior
- [x] Verificar cache invalidation

### 7.2 Frontend Tests
- [x] `cd machado-web && yarn test`
- [x] Verificar hooks de trainer/home e trainer/party
- [x] Validar client service apontando para `/trainer/home` e `/trainer/party`

### 7.3 Manual Testing (se backend/frontend local)
- [x] Acessar Home do treinador
- [x] Validar que dados aparecem (trainer, encounter, party, últimos descobertos)
- [x] Selecionar Pokémon da party
- [x] Validar que party atualiza
- [x] Verificar limite 6 Pokémon
- [x] Verificar cache (segunda requisição deve ser rápida)

### 7.4 Linting
- [x] `cd machado-api && make lint`
- [x] `cd machado-web && yarn lint`

**Deliverable:** todos testes verdes, sem erros de lint

---

## Task 8: docs-and-cleanup
**Dependencies:** validation-and-tests

**Objective:** Documentar changes e limpar

**Steps:**

### 8.1 Update Documentation
- [x] Revisar `prompts/trainer-home-and-party-refactor.md`
- [x] Atualizar com achados reais (houve surpresas? Documentar)

### 8.2 Update AGENTS.md
- [x] `machado-api/AGENTS.md` — adicionar ou atualizar seção `trainer-party`
- [x] Atualizar texto para refletir `trainer` como agregador de home, sem `trainer_home` como domain novo

### 8.3 OpenSpec Cleanup
- [x] Fechar `openspec/changes/trainer-home-and-party-refactor/` (marcar como done)
- [x] Mover para `openspec/changes/archive/` se workflow exigir

### 8.4 Verify No Unused Code
- [x] Verificar se `trainer_exploration/schema.py` ainda tem `TrainerPartyMemberSchema`, `UpdateTrainerPartySchema` após serem copiadas
  - Se sim e não mais usadas lá: remover
  - Se ainda usadas (deps): manter e documentar
- [x] Verificar se `TrainerHomeSchema` está no local correto e não causa dependência circular
- [x] Verificar se rotas antigas `/trainer/exploration/home` e `/trainer/exploration/party` foram removidas ou documentadas como alias temporário

**Deliverable:** documentação atualizada, cleanup completo
