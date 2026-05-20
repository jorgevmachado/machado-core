# Change: trainer-home-and-party-refactor

## Resumo
Desacoplar gerenciamento de party e Home de `trainer-exploration`. Extrair `party` para um novo domínio API (`trainer-party`) e reposicionar `GET /trainer/home` como um caso de uso agregador do domínio `trainer`, preservando o contrato atual do payload e reduzindo acoplamento estrutural.

## O Problema

Atualmente em `trainer_exploration/service.py`:
- `update_party()` — altera party do treinador (**pertence a trainer-party**)
- `get_party()` — lê party do treinador (**pertence a trainer-party**)
- `get_home()` — agrega dados da Home (**pertence ao domínio trainer**)
- `walk()`, `select_active_encounter()` — exploração genuína (**fica em trainer-exploration**)

O service ficou grande, misturando **três responsabilidades diferentes** no mesmo lugar. Quando party muda, Home é invalidada; quando encounter muda, ambas são invalidadas. Risco de regressão e crescimento descontrolado.

## Solução Proposta

**Preservar comportamento e contrato externo, redistribuindo responsabilidades internas:**

1. **`trainer-party` (novo domínio API)**
   - **Responsável:** seleção/gerenciamento da party principal
   - **Origem:** extrair `update_party()`, `get_party()`, business rules de validação de party (MAX 6, sem duplicatas) de `trainer_exploration`
   - **Reutiliza:** model `trainer_party.py`, migration existente e lógica atual de validação/serialização
   - **Novo:** criar repository/service/schema/route próprios em `trainer_party/`
   - **Endpoints backend canônicos:** `GET /trainer/party`, `PUT /trainer/party`

2. **`trainer` (agregador de Home)**
   - **Responsável:** expor `GET /trainer/home` como visão agregada do treinador
   - **Origem:** mover `get_home()` de `trainer_exploration` para `TrainerService`
   - **Contrato:** preservar exatamente o shape atual do payload, incluindo `active_encounter` e `latest_discoveries`
   - **Dependências:** orquestra dados de `trainer-party`, `trainer-exploration` e `pokedex` por interfaces públicas explícitas
   - **Endpoint backend canônico:** `GET /trainer/home`

3. **`trainer-exploration` (refatorado, reduzido)**
   - **Responsável:** apenas exploração genuína
   - **Mantém:** `initialize_for_trainer()`, `list_encounters()`, `select_active_encounter()`, `walk()`
   - **Remove:** lógica de party e Home
   - **Cache:** apenas para encounters
   - **Rotas restantes:** `/trainer/exploration/encounters`, `/trainer/exploration/encounters/active`, `/trainer/exploration/walk`

## Mapa de Responsabilidades

```
Antes (acoplado):
┌─────────────────────────────────────────┐
│    trainer-exploration (grande)         │
├─────────────────────────────────────────┤
│ • Encounters ✓                          │
│ • Party gerenciamento ✗ (aqui!)        │
│ • Home agregação ✗ (aqui!)             │
│ • Walk/eventos ✓                        │
│ • Cache (3 tipos)                       │
└─────────────────────────────────────────┘

Depois (desacoplado):
┌──────────────────┐  ┌─────────────────────┐  ┌──────────────────┐
│  trainer-party   │  │ trainer-exploration │  │     trainer      │
├──────────────────┤  ├─────────────────────┤  ├──────────────────┤
│ • Update party   │  │ • Encounters        │  │ • Home aggregate │
│ • Get party      │  │ • Walk/events       │  │ • Trainer data   │
│ • Validate 6     │  │ • Select encounter  │  │ • Cache home     │
│ • Soft delete    │  │ • Cache encounter   │  │ • Orchestration  │
│ • Cache party    │  │                     │  │                  │
└──────────────────┘  └─────────────────────┘  └──────────────────┘
        ↑                        ↑                      │
        └────────────────────────┴──────────────────────┘
                   trainer agrega party + exploration + pokedex
```

## O que NÃO muda

- ✓ Modelo de dados `trainer_party` — já existe, já tem migração
- ✓ Contrato do payload de Home — preservado (`active_encounter`, `latest_discoveries`, `party`, `trainer`)
- ✓ Validações de negócio de party (MAX 6, sem duplicatas) — preservadas
- ✓ BFF Next continua respondendo em `/api/trainer/home` e `/api/trainer/party`
- ✓ Migração existente de `trainer_party` — reaproveitada

## Arquivos/Áreas Impactadas

| Arquivo | Tipo | O que muda |
|---------|------|-----------|
| `machado-api/app/domain/trainer/trainer_party/` | NEW | Novo domínio (repository, service, business, schema, route, tests) |
| `machado-api/app/domain/trainer/service.py` | REFATOR | Passa a agregar `GET /trainer/home` |
| `machado-api/app/domain/trainer/route.py` | REFATOR | Registra `GET /trainer/home` e router de `trainer_party` |
| `machado-api/app/domain/trainer/trainer_exploration/` | REFATOR | Remove `update_party()`, `get_party()`, `get_home()` e foca em exploração |
| `machado-api/app/models/trainer_party.py` | NENHUMA | Model existente — apenas reutilizado |
| `machado-api/migrations/versions/` | NENHUMA | Migração existente funciona como-é |
| `machado-web/app/ui/features/trainer/services/service/service.ts` | REFATOR | Client da web passa a chamar `/trainer/home` e `/trainer/party` |
| `machado-web/app/api/trainer/` | COMPAT | BFF externo mantém URLs estáveis |
| `machado-web/app/ui/features/trainer/home/` | COMPAT | Hooks consumem mesmo contrato |

## Critérios de Aceite

- [ ] Domínio `trainer-party` implementado, testado, com party endpoints funcionando
- [ ] `GET /trainer/home` implementado em `trainer` com payload preservado
- [ ] `trainer-exploration` reduzido (remove party/home, mantém encounters e walk)
- [ ] `GET /trainer/home` retorna os campos `trainer`, `active_encounter`, `party` e `latest_discoveries`
- [ ] `PUT /trainer/party` e `GET /trainer/party` funcionam via `trainer_party`
- [ ] Cache correto: update de party invalida home + party; mudança de encounter invalida home + encounters
- [ ] `trainer_exploration/repository.py` não contém mais queries de party nem latest discoveries
- [ ] Limite 6 Pokémon validado em testes
- [ ] Testes para soft-delete e edge cases
- [ ] BFF `/api/trainer/home` e `/api/trainer/party` continuam funcionando sem mudança de contrato
