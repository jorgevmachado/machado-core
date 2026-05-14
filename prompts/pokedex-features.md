# Proposta: pokedex-features

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-API](../docs/architecture-api.md)
- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS API](../machado-api/AGENTS.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já existir um definido.

---

## 1. Objetivo

Implementar a entidade `pokedex`, vinculada a um `trainer` e a um `pokemon`, permitindo que cada treinador possua uma lista de pokedex ex com Pokémon próprios com atributos, nickname e progressão individual.
Onde trainer deverá ter uma lista de `pokedex` no retorno da API.
Implementar listagem e detalhamento de Pokedex do treinador.

A feature deve permitir:

- listar Pokedex de treinador com paginação e filtros;
- visualizar detalhes de uma Pokedex do treinador, incluindo atributos e data de descoberta;
- utilizar cache Redis para otimizar consultas.

---

## 2. Contexto Atual

- Projeto: machado (monorepo API + WEB)
- Domínio afetado: Trainer / Pokémon / Pokedex
- Comportamento atual: existe proposta para Pokémon base e Trainer, mas ainda não existe Pokedex pertencente ao treinador
- Problema identificado: ausência de uma entidade que represente Pokedex de um treinador, com atributos próprios e evolução futura

---

## 3. Comportamento Esperado

- O sistema deve permitir criar um `pokedex` vinculado a um `trainer`
- Cada `pokedex` deve estar relacionado a apenas um `pokemon`
- A API deve gerar atributos aleatórios para o `pokedex` (Pode seguir a abordagem de `my-pokemon`)
- Ao iniciar o serviço `onboarding` de `trainer` a aplicação deve percorrer a lista de `pokemon` e criar uma `pokedex` para cada um, vinculada ao `trainer` recém-criado, para representar a coleção inicial do treinador, deve pegar o `pokemon` escolhido pelo usuário e adicionar a lista de pokedex com a data que foi encontrado e flagando a flag `discovered` como `true` 
- O usuário deve ter a opção de definir um nickname para cada `pokedex`, mas isso deve ser opcional
- A estrutura deve ficar preparada para evolução, batalha, level up
- O sistema deve listar Pokémons do treinador com paginação e filtros
- O usuário deve conseguir visualizar detalhes completos de uma entrada da Pokedex do treinador, incluindo atributos e data de descoberta
- O domínio deve ficar preparado para futuramente sincronizar descoberta da `pokedex` com novos eventos de captura, mas isso não deve ser ativado nesta implementação

---

## 4. Escopo

### Dentro do escopo

- Criação da entidade `pokedex`
- Relacionamento com `trainer`
- Relacionamento com `pokemon`
- Geração aleatória de atributos (Pode seguir a abordagem de `my-pokemon`)
- Registro de `discovered_at` quando o pokémon for descoberto, sendo o valor inicial como `None`
- Registro de `discovered` quando o pokémon for descoberto, sendo o valor inicial como `False`
- Definição opcional de nickname
- Estrutura preparada para futura level up
- Listagem paginada de Pokedex do treinador
- Detalhamento completo de Pokedex do treinador
- Endpoint explícito para descoberta de pokemon da pokedex do treinador

### Fora do escopo

- Sistema de batalha
- Evolução automática
- Level up em batalha
- Tela de troca de movimentos
- Compra/captura com pokebolas
- Implementação da lógica completa de sugestão de novo movimento

---

## 5. Regras de Negócio

### Regra 1 — Relacionamento com Trainer

**Dado que:** existe um treinador autenticado  
**Quando:** um `pokedex` for criado  
**Então:** ele deve ser vinculado ao `trainer` do usuário atual

---

### Regra 2 — Relacionamento com Pokémon base

**Dado que:** existe um Pokémon base cadastrado  
**Quando:** um `pokedex` for criado  
**Então:** ele deve possuir relacionamento com apenas um `pokemon`

---

### Regra 3 — Nickname

**Dado que:** o usuário está revelando uma `pokedex`  
**Quando:** informar um nickname  
**Então:** o nickname deve ser persistido no `pokedex`

---

### Regra 4 — Data de descoberta e flag de descoberta

**Dado que:** um `pokedex` foi registrado como descoberto  
**Quando:** for persistido  
**Então:** deve registrar `discovered_at` e `discovered` como `true`

---

### Regra 5 — Geração de atributos

**Dado que:** um `pokedex` está sendo criado  
**Quando:** a API processar a criação  
**Então:** deve gerar aleatoriamente os atributos:

- HP
- SPEED
- ATTACK
- DEFENSE
- SPECIAL_ATTACK
- SPECIAL_DEFENSE
- EXPERIENCE
- MAX_HP
- LEVEL

---

### Regra 6 — Inicialização de uma lista de pokedex

**Dado que:** Existe uma lista de Pokémon base cadastrados 
**Quando:** um `trainer` for criado e passar pelo processo de onboarding
**Então:** deve ser criada uma `pokedex` para cada Pokémon base, vinculada ao `trainer`, com a data de descoberta registrada e a flag de descoberta marcada como `true` para o Pokémon escolhido pelo usuário, e os demais com a data de descoberta como `None` e a flag de descoberta como `false`.

---

### Regra 7 - Descoberta explícita de um pokémon da pokedex

**Dado que:** existe uma entrada de `pokedex` vinculada ao treinador autenticado
**Quando:** a API processar uma solicitação explícita de descoberta para aquele `pokemon`
**Então:** deve atualizar o `pokemon` da `pokedex` com a data de descoberta registrada e a flag de descoberta marcada como `true`

---

## 6. Escopo Técnicos

### 6.1 API

Toda regra de negócio deve estar na API.  
O frontend deve ser apenas consumidor.

#### Funcionalidade

- endpoint para descobrir `pokedex`
- endpoint para listar `pokedex` do treinador autenticado
- endpoint para detalhar `pokedex` do treinador autenticado
- relacionamento com `trainer`
- relacionamento com `pokemon`

#### Fluxos

1. treinador é inicializado através do `onboarding`
2. treinador escolhe Pokémon base
3. usuário informa nickname, se desejar
4. API cria `my-pokemon`
5. API gera atributos aleatórios para `my-pokemon`
6. API percorre a lista de `pokemon` e cria `pokedex` para cada um, vinculada ao `trainer`, com a data de descoberta registrada e a flag de descoberta marcada como `true` para o Pokémon escolhido pelo usuário, e os demais com a data de descoberta como `None` e a flag de descoberta como `false`
7. API gera atributos aleatórios para cada `pokedex` criado
8. API persiste dados e retorna contrato normalizado

#### 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Usar os dados já persistidos pela feature de Pokémon.

#### 6.1.2 Enriquecimento de dados

Durante o processamento, a API deve persistir:

- trainer_id
- pokemon_id
- nickname
- hp
- max_hp
- speed
- attack
- defense
- special_attack
- special_defense
- experience
- level
- discovered
- discovered_at
- created_at
- updated_at
- deleted_at
- Logica para calculo dos atributos (HP, SPEED, ATTACK, DEFENSE, SPECIAL_ATTACK, SPECIAL_DEFENSE, EXPERIENCE, LEVEL) deve ser implementada na API, utilizando uma fórmula simples baseada no Pokémon base e um fator aleatório para garantir diversidade entre as `pokedex` (usar como base a lógica de `my-pokemon`).

#### 6.1.3 Cache

Usar o padrão do sistema, lembrando que para este caso o cache não precisa ser tão grande quanto a feature de Pokémon.

#### 6.1.4 Consistência e performance

- Garantir que cada `pokedex` pertença a um único `trainer`
- Garantir que cada `pokedex` tenha apenas um `pokemon`
- Utilizar constraints únicas quando necessário
- Evitar N+1 queries
- Garantir soft delete nas novas tabelas

#### 6.1.5 Contrato da API

A API deve ser a fonte única de verdade.

A API deve retornar dados normalizados contendo:

- dados da `pokedex`
- dados básicos do `pokemon`
- dados básicos do `trainer` com lista de `my-pokemon` e `pokedex`
- atributos gerados

### 6.2 WEB

#### 6.2.1 Páginas

- listagem da Pokedex do treinador
- detalhe da Pokedex do treinador
- descoberta de pokemon da pokedex do treinador

#### 6.2.2 Componentes

- cards de pokedex
- formulário/input de nickname
- seleção de Pokémon base
- exibição de atributos
- indicadores visuais de level e experiência

Estados:

- loading
- erro
- vazio
- sucesso

#### 6.2.3 Renderização

Exibir:

- nickname
- imagem do Pokémon base
- level
- experiência
- HP / MAX_HP
- atributos de batalha
- data de descoberta

#### 6.2.4 Interações

- descobrir pokemon da pokedex do treinador
- escolher nickname
- visualizar detalhe
- navegar para listagem

---

## 7. Restrições Arquiteturais

- Não criar nova arquitetura
- Não duplicar lógica entre API e WEB
- Toda regra de negócio deve estar na API
- Reutilizar padrões existentes
- Minimizar impacto no código atual
- Toda nova tabela criada no banco de dados deve suportar soft delete
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
- Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

## 8. Critérios de Aceite

- [ ] criação de lista de `pokedex` funcional junto ao serviço de `onboarding`
- [ ] relacionamento com `trainer` funcionando
- [ ] relacionamento com `pokemon` funcionando
- [ ] nickname persistido corretamente
- [ ] `discoverd_at` registrado corretamente quando o pokemon for descoberto
- [ ] `discoverd` registrado corretamente quando o pokemon for descoberto
- [ ] atributos aleatórios gerados corretamente
- [ ] listagem funcionando
- [ ] detalhe funcionando
- [ ] API retornando dados normalizados
- [ ] sem regressões
- [ ] novas tabelas possuem suporte a soft delete
- [ ] consultas padrão ignoram registros deletados logicamente
- [ ] nenhuma exclusão física foi implementada sem justificativa

## 9. Plano de Validação

### Validação manual

1. Criar um treinador
2. Escolher um pokemon base como `my-pokemon`
3. Validar a criação da lista de `pokedex` com o mesmo tamanho da lista de `pokemon` base
4. Validar a descoberta do pokemon escolhido pelo treinador e que está marcada corretamente na `pokedex`
5. Validar atributos gerados
6. Validar `discoverd_at` quando o pokemon for descoberto
7. Validar `discoverd` quando o pokemon for descoberto
8. Acessar listagem de Pokedex do treinador
9. Acessar detalhe de Pokedex do treinador

### Testes automatizados

- criação de lista de `pokedex` do treinador na hora do `onboarding`
- relacionamento com `trainer`
- relacionamento com `pokemon`
- geração de atributos
- validação de soft delete
- cenário de erro quando Pokémon não existir
- cenário de erro quando Trainer não existir

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
