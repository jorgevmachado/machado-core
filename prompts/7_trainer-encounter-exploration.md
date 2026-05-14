# Proposta: trainer-encounter-exploration

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-API](../docs/architecture-api.md)
- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS API](../machado-api/AGENTS.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já exister um definido.

---

## 1. Objetivo

Implementar o sistema de exploração do treinador, permitindo que o usuário possua encounters conhecidos, selecione um encounter ativo, caminhe entre encounters descobertos e encontre eventos aleatórios como Pokémon selvagens e pokebolas.

---

## 2. Contexto Atual

- Projeto: machado (monorepo API + WEB)
- Domínio afetado: Trainer / PokemonEncounter / Pokedex / MyPokemon
- Arquivos/pastas relevantes:
  - `machado-api/app/domain`
  - `machado-api/app/models`
  - `machado-web/app/ui/features`
  - `machado-web/app`
- Comportamento atual:
  - `Trainer` já possui estrutura inicial
  - `Pokémon` já possui encounters
  - `Pokedex` já possui fluxo de descoberta
  - `MyPokemon` já possui vínculo com Trainer
- Problema identificado:
  - não existe sistema de exploração
  - não existe encounter ativo do treinador
  - não existe sistema de caminhada
  - não existe sistema de eventos aleatórios
  - treinador ainda não consegue desbloquear novos encounters

---

## 3. Comportamento Esperado

- O sistema deve permitir que o treinador possua encounters conhecidos
- O sistema deve permitir selecionar um encounter ativo
- O treinador deve conseguir caminhar apenas em encounters conhecidos
- O sistema deve gerar eventos aleatórios ao caminhar
- O sistema deve permitir encontrar Pokémon selvagens
- O sistema deve permitir encontrar pokebolas aleatoriamente
- O sistema deve retornar um evento normalizado para o frontend
- O sistema deve desbloquear novos encounters ao vencer Pokémon futuramente
- A interface deve exibir o encounter selecionado do treinador
- A interface deve exibir os últimos Pokémon descobertos da pokedex
- A interface deve exibir os dados principais do treinador na Home
- O treinador deve conseguir selecionar até 6 `my-pokemon` como party principal

---

## 4. Escopo

### Dentro do escopo

- Relacionamento entre `trainers` e `pokemon_encounters` onde um `trainer`pode ter muitos `pokemon_encounters`
- Controle de encounter selecionado
- Sistema de caminhada
- Sistema de eventos aleatórios
- Evento de Pokémon selvagem
- Evento de encontrar pokebola
- Seleção da party principal do treinador
- Exibição dos dados do treinador na Home
- Exibição dos últimos Pokémon descobertos da pokedex
- Estrutura preparada para futura integração com batalha

### Fora do escopo

- Sistema de batalha
- Captura de Pokémon
- Consumo de pokebolas
- Sistema de level up
- Sistema de experiência
- Evolução de Pokémon
- Troca de Pokémon durante batalha
- Sistema de dano
- Sistema de turno
- IA de Pokémon selvagem

---

## 5. Regras de Negócio

### Regra 1 — Trainer encounter inicial

**Dado que:** um treinador foi criado  
**Quando:** finalizar onboarding  
**Então:** o sistema deve vincular ao treinador os encounters do Pokémon inicial escolhido

---

### Regra 2 — Encounter selecionado

**Dado que:** o treinador possui encounters conhecidos  
**Quando:** selecionar um encounter  
**Então:** apenas um encounter pode ficar selecionado por vez

---

### Regra 3 — Caminhar apenas em encounter conhecido

**Dado que:** o treinador possui encounters conhecidos  
**Quando:** tentar caminhar  
**Então:** o sistema deve permitir caminhar apenas no encounter selecionado e conhecido

---

### Regra 4 — Evento aleatório

**Dado que:** o treinador caminhou  
**Quando:** o sistema processar a caminhada  
**Então:** deve retornar um evento aleatório

---

### Regra 5 — Evento de Pokémon selvagem

**Dado que:** um evento aleatório foi gerado  
**Quando:** o evento for do tipo Pokémon selvagem  
**Então:** o sistema deve selecionar aleatoriamente um Pokémon pertencente ao encounter do treinador

---

### Regra 6 — Evento de pokebola

**Dado que:** um evento aleatório foi gerado  
**Quando:** o evento for do tipo item  
**Então:** o treinador deve receber uma quantidade aleatória de pokebolas

---

### Regra 7 — Party principal

**Dado que:** o treinador possui `my-pokemon`  
**Quando:** selecionar sua party principal  
**Então:** deve conseguir selecionar até 6 Pokémon

---

### Regra 8 — Limite da party

**Dado que:** o treinador já possui 6 Pokémon selecionados  
**Quando:** tentar adicionar um novo Pokémon na party  
**Então:** o sistema deve impedir a operação até que um Pokémon seja removido

---

### Regra 9 — Últimos descobertos

**Dado que:** o treinador possui Pokémon descobertos na pokedex  
**Quando:** acessar a Home  
**Então:** o sistema deve retornar os 3 últimos Pokémon descobertos

---

### Regra 10 — Estrutura futura de desbloqueio

**Dado que:** o treinador vencer um Pokémon futuramente  
**Quando:** o Pokémon possuir encounters não conhecidos pelo treinador  
**Então:** os novos encounters deverão poder ser desbloqueados futuramente

---

## 6. Escopo Técnicos

### 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

Funcionalidade

- endpoint para listar encounters do treinador autenticado
- endpoint para selecionar encounter ativo
- endpoint para caminhar
- endpoint para selecionar party principal
- endpoint para listar Home do treinador
- relacionamento entre `trainer` e `pokemon_encounters` onde um `trainer`pode ter muitos `pokemon_encounters`

Fluxos

1. trainer é criado
2. trainer recebe encounters do Pokémon inicial
3. trainer seleciona encounter
4. trainer caminha
5. sistema gera evento aleatório
6. sistema retorna evento normalizado

---

#### 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados já persistidos localmente.

---

#### 6.1.2 Enriquecimento de dados

Durante o processamento, a API deve persistir:

### trainer_party

- trainer_id
- my_pokemon_id
- slot
- is_active
- created_at
- updated_at
- deleted_at

### exploration_event

Estrutura preparada para eventos futuros:

- event_type
- payload
- created_at

---

#### 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:encounters:{trainer_id}
trainer:party:{trainer_id}
```
Regras:

- invalidar cache após alteração de encounter selecionado
- invalidar cache após alteração da party
- invalidar cache após caminhada****

---

#### 6.1.4 Consistência e performance
- Garantir apenas um encounter selecionado por treinador
- Garantir no máximo 6 Pokémon na party
- Evitar duplicação de encounters
- Evitar N+1 queries
- Utilizar soft delete
- Garantir integridade referencial
- Operações devem ser idempotentes quando possível

---
#### 6.1.5 Contrato da API

A API deve:

- retornar dados normalizados
- evitar transformação no frontend
- manter consistência de naming

Exemplo de resposta de caminhada:

```json
{
  "event_type": "WILD_POKEMON",
  "data": {}
}
```
Exemplo de evento de item:
```json
{
  "event_type": "ITEM_FOUND",
  "data": {
    "item": "pokeball",
    "quantity": 2
  }
} 
```
Tipos iniciais de evento:

- WILD_POKEMON
- ITEM_FOUND
- NOTHING
---

### 6.2 WEB

#### 6.2.1 Páginas
- Home do treinador
- seleção de encounters
- seleção da party principal

---

#### 6.2.2 Componentes
- card do treinador
- seletor de encounter
- lista da party principal
- botão de caminhar
- cards dos últimos descobertos
- indicador de evento encontrado

Estados:

- loading
- erro
- vazio
- sucesso

---

#### 6.2.3 Renderização

A Home deve exibir:

- level do treinador
- quantidade de pokebolas
- capture_rate
- encounter selecionado
- party principal
- últimos Pokémon descobertos
- botão de caminhar

Eventos devem ser renderizados de forma visualmente distinta:

- Pokémon selvagem
- item encontrado
- nenhum evento

---

#### 6.2.4 Interações
- selecionar encounter
- selecionar até 6 Pokémon da party
- caminhar
- visualizar evento encontrado
- atualizar Home após caminhada

---

## 7. Restrições Arquiteturais
- Não criar nova arquitetura
- Não duplicar lógica entre API e WEB
- Toda regra de negócio deve estar na API
- Reutilizar padrões existentes
- Minimizar impacto no código atual
- Toda nova tabela criada no banco de dados deve suportar soft delete.
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto.
- Usar o padrão existente do projeto para soft delete, preferencialmente com deleted_at.
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário.

---

## 8. Critérios de Aceite
- [ ]  trainer possui lista de encounters conhecidos
- [ ]  trainer consegue selecionar encounter ativo
- [ ]  apenas um encounter pode ficar selecionado
- [ ]  trainer consegue caminhar
- [ ]  sistema gera eventos aleatórios
- [ ]  evento de Pokémon selvagem funcionando
- [ ]  evento de pokebola funcionando
- [ ]  trainer consegue selecionar até 6 Pokémon
- [ ]  limite de 6 Pokémon funcionando
- [ ]  Home retorna dados completos do treinador
- [ ]  Home retorna últimos descobertos da pokedex
- [ ]  cache funcionando
- [ ]  sem duplicação de encounters
- [ ]  sem regressões
- [ ]  novas tabelas possuem suporte a soft delete
- [ ]  consultas padrão ignoram registros deletados logicamente
- [ ]  nenhuma exclusão física foi implementada sem justificativa

---

## 9. Plano de Validação

### Validação manual

1. Criar treinador
2. Escolher Pokémon inicial
3. Validar encounters iniciais
4. Selecionar encounter
5. Caminhar
6. Validar geração de evento
7. Validar evento de Pokémon selvagem
8. Validar evento de item
9. Selecionar party principal
10. Validar limite de 6 Pokémon
11. Validar Home do treinador

### Testes automatizados

- criação de trainer encounters
- seleção de encounter
- validação de apenas um encounter selecionado
- geração de evento aleatório
- evento de Pokémon selvagem
- evento de item
- limite da party
- listagem da Home
- validação de soft delete
- cenários de erro
- edge cases

---

## 10. Saída Esperada do OpenSpec

Gerar uma proposta contendo:

1. Resumo da mudança
2. Problema identificado
3. Solução proposta
4. Arquivos impactados
5. Plano de implementação
6. Alterações de modelo
7. Alterações de API
8. Alterações de UI
9. Estratégia de testes
10. Riscos e pontos de atenção