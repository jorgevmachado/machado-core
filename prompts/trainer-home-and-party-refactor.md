# Proposta: trainer-home-and-party-refactor

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-API](../docs/architecture-api.md)
- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS API](../machado-api/AGENTS.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já exister um definido.

---

## Observação importante

Já existe uma proposta anterior chamada `7_trainer-encounter-exploration`.

Esta proposta NÃO deve:

- recriar regras de exploração;
- recriar regras de caminhada;
- recriar regras de eventos aleatórios;
- recriar regras de encounter;
- duplicar endpoints já existentes;
- criar nova implementação paralela.

Esta proposta deve apenas:

- reorganizar responsabilidades;
- desacoplar Home de Exploration;
- centralizar gerenciamento da party do treinador;
- transformar a Home em uma camada agregadora;
- evitar crescimento excessivo de services;
- evitar acoplamento excessivo entre Home, Encounter e Party.

Caso já existam implementações criadas pela proposta anterior, elas devem ser reorganizadas e reutilizadas, evitando retrabalho e duplicação.

---

## 1. Objetivo

Refatorar e reorganizar a estrutura de Home e Party do treinador para separar responsabilidades entre exploração, gerenciamento de party e agregação de dados da Home.

---

## 2. Contexto Atual

- Projeto: machado (monorepo API + WEB)
- Domínio afetado:
  - Trainer
  - TrainerEncounter
  - MyPokemon
  - Pokedex
  - Home
- Arquivos/pastas relevantes:
  - `machado-api/app/domain`
  - `machado-api/app/models`
  - `machado-web/app/ui/features`
  - `machado-web/app`
- Comportamento atual:
  - existe implementação inicial da feature `trainer-encounter-exploration`
  - parte da lógica de Home e Party foi implementada dentro da feature de exploração
- Problema identificado:
  - responsabilidades misturadas entre Home, Party e Exploration
  - risco de crescimento excessivo de services
  - risco de duplicação de lógica
  - risco de acoplamento excessivo
  - Home contendo regras de negócio que deveriam estar em domínios específicos
  - gerenciamento da party do treinador sem domínio próprio

---

## 3. Comportamento Esperado

- O sistema deve possuir um domínio específico para gerenciamento da party do treinador
- O sistema deve possuir uma camada agregadora específica para Home
- O domínio de exploração deve permanecer responsável apenas por exploração
- O domínio de party deve permanecer responsável apenas pelo time principal do treinador
- A Home deve apenas agregar dados vindos de outros domínios
- A Home não deve possuir regras de negócio complexas
- O treinador deve continuar conseguindo selecionar até 6 Pokémon
- A Home deve continuar exibindo:
  - dados do treinador
  - encounter selecionado
  - últimos Pokémon descobertos
  - party principal
- O sistema não deve duplicar lógica já existente
- O sistema deve reutilizar implementações já criadas anteriormente

---

## 4. Escopo

### Dentro do escopo

- Refatoração estrutural da Home
- Refatoração estrutural da Party
- Criação do domínio `trainer-party`
- Criação do domínio `trainer-home`
- Reorganização de responsabilidades
- Reutilização da implementação existente
- Separação de responsabilidades
- Ajuste de endpoints se necessário
- Ajuste de services e repositories
- Ajuste de contratos se necessário

### Fora do escopo

- Sistema de batalha
- Sistema de captura
- Sistema de experiência
- Sistema de level up
- Evolução de Pokémon
- IA de Pokémon selvagem
- Refatoração global da arquitetura
- Reescrita completa da feature anterior
- Reimplementação completa de exploration

---

## 5. Regras de Negócio

### Regra 1 — Exploration deve permanecer isolado

**Dado que:** existe o domínio de exploração  
**Quando:** ocorrer qualquer ação de exploração  
**Então:** apenas o domínio `trainer-encounter-exploration` deve ser responsável pela lógica

---

### Regra 2 — Party deve possuir domínio próprio

**Dado que:** o treinador possui Pokémon principais  
**Quando:** gerenciar a party  
**Então:** toda regra relacionada à party deve existir apenas no domínio `trainer-party`

---

### Regra 3 — Limite da party

**Dado que:** o treinador está gerenciando sua party  
**Quando:** tentar adicionar Pokémon  
**Então:** o sistema deve continuar limitando a 6 Pokémon

---

### Regra 4 — Home agregadora

**Dado que:** a Home do treinador é carregada  
**Quando:** os dados forem retornados  
**Então:** a Home deve apenas agregar dados de outros domínios

---

### Regra 5 — Home sem regras complexas

**Dado que:** existe lógica de negócio  
**Quando:** a Home consumir dados  
**Então:** a lógica deve permanecer nos domínios específicos

---

### Regra 6 — Últimos descobertos

**Dado que:** o treinador possui Pokémon descobertos  
**Quando:** a Home for carregada  
**Então:** a Home deve continuar retornando os 3 últimos Pokémon descobertos

---

### Regra 7 — Encounter selecionado

**Dado que:** existe encounter selecionado  
**Quando:** a Home for carregada  
**Então:** a Home deve consumir os dados do domínio de exploration sem duplicar lógica

---

### Regra 8 — Reutilização da implementação anterior

**Dado que:** já existe implementação anterior  
**Quando:** ocorrer a refatoração  
**Então:** reutilizar ao máximo estruturas já existentes

---

## 6. Escopo Técnicos

### 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

Funcionalidade

- reorganizar domínio de party
- reorganizar domínio de Home
- criar agregador da Home
- reutilizar endpoints existentes quando possível
- desacoplar Home de Exploration

Fluxos

1. trainer acessa Home
2. Home consulta serviços específicos
3. Home agrega dados
4. Home retorna payload normalizado

---

#### 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

#### 6.1.2 Enriquecimento de dados

### trainer_home

A Home deve agregar:

- dados do trainer
- party principal
- encounter selecionado
- últimos descobertos da pokedex

### trainer_party

Persistir:

- trainer_id
- my_pokemon_id
- slot
- is_active
- created_at
- updated_at
- deleted_at

---

#### 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:party:{trainer_id}
```

Regras:

- invalidar cache da Home após alteração da party
- invalidar cache da Home após alteração de encounter
- invalidar cache da Home após descoberta da pokedex

---

#### 6.1.4 Consistência e performance

- evitar duplicação de lógica
- evitar services gigantes
- evitar acoplamento excessivo
- evitar N+1 queries
- reutilizar estruturas existentes
- manter integridade referencial
- utilizar soft delete
- manter operações idempotentes quando possível

---

#### 6.1.5 Contrato da API

A API deve:

- retornar dados normalizados
- evitar transformação no frontend
- manter consistência de naming

Exemplo conceitual da Home:

```json
{
  "trainer": {},
  "selected_encounter": {},
  "party": [],
  "last_discovered_pokemon": []
}
```

A Home não deve conter lógica de negócio complexa.

A Home deve apenas agregar dados.

---

### 6.2 WEB

#### 6.2.1 Páginas

- Home do treinador
- gerenciamento da party principal

---

#### 6.2.2 Componentes

- card do treinador
- lista da party
- seletor da party
- cards de últimos descobertos
- card de encounter selecionado

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

A renderização deve consumir apenas o payload agregador da Home.

---

#### 6.2.4 Interações

- selecionar Pokémon da party
- remover Pokémon da party
- visualizar Home
- visualizar encounter selecionado

---

## 7. Restrições Arquiteturais

- Não criar nova arquitetura
- Não duplicar lógica entre API e WEB
- Toda regra de negócio deve estar na API
- Reutilizar padrões existentes
- Minimizar impacto no código atual
- Não reimplementar exploration
- Não duplicar regras de exploration
- Não criar services gigantes
- Toda nova tabela criada no banco de dados deve suportar soft delete.
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto.
- Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`.
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário.

---

## 8. Critérios de Aceite

- [ ] domínio `trainer-party` criado
- [ ] Home desacoplada de exploration
- [ ] Home funcionando como agregadora
- [ ] regras de negócio permanecem em domínios específicos
- [ ] sem duplicação de lógica
- [ ] limite de 6 Pokémon funcionando
- [ ] encounter selecionado continua funcionando
- [ ] últimos descobertos continuam funcionando
- [ ] payload da Home normalizado
- [ ] sem regressões
- [ ] cache funcionando
- [ ] services organizados corretamente
- [ ] sem services excessivamente grandes
- [ ] novas tabelas possuem suporte a soft delete
- [ ] consultas padrão ignoram registros deletados logicamente
- [ ] nenhuma exclusão física foi implementada sem justificativa

---

## 9. Plano de Validação

Validação manual

1. Acessar Home do treinador
2. Validar dados agregados
3. Validar encounter selecionado
4. Validar últimos descobertos
5. Selecionar Pokémon da party
6. Validar limite de 6 Pokémon
7. Validar atualização da Home
8. Validar ausência de regressões

Testes automatizados

- agregação da Home
- seleção da party
- limite da party
- consumo dos dados de exploration
- atualização da Home
- cenários de erro
- edge cases
- validação de soft delete

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
