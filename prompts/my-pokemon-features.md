# Proposta: my-pokemon-features

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

Implementar a entidade `my-pokemon`, vinculada a um `trainer` e a um `pokemon`, permitindo que cada treinador possua Pokémon próprios com atributos, nickname, movimentos e progressão individual.

---

## 2. Contexto Atual

- Projeto: machado (monorepo API + WEB)
- Domínio afetado: Trainer / Pokémon / My-Pokemon
- Comportamento atual: existe proposta para Pokémon base e Trainer, mas ainda não existe Pokémon pertencente ao treinador
- Problema identificado: ausência de uma entidade que represente Pokémon capturados, com atributos próprios e evolução futura

---

## 3. Comportamento Esperado

- O sistema deve permitir criar um `my-pokemon` vinculado a um `trainer`
- Cada `my-pokemon` deve estar relacionado a apenas um `pokemon`
- O usuário deve poder definir um nickname
- A API deve gerar atributos aleatórios para o `my-pokemon`
- A API deve selecionar 4 movimentos distintos do Pokémon base
- Cada movimento deve possuir PP próprio
- A estrutura deve ficar preparada para evolução, batalha, level up e futura troca de movimentos

---

## 4. Escopo

### Dentro do escopo

- Criação da entidade `my-pokemon`
- Relacionamento com `trainer`
- Relacionamento com `pokemon`
- Geração aleatória de atributos
- Registro de `captured_at`
- Definição opcional de nickname
- Associação inicial de 4 movimentos distintos
- Controle inicial de PP por movimento
- Estrutura preparada para futura troca de movimentos no level up

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
**Quando:** um `my-pokemon` for criado  
**Então:** ele deve ser vinculado ao `trainer` do usuário atual

---

### Regra 2 — Relacionamento com Pokémon base

**Dado que:** existe um Pokémon base cadastrado  
**Quando:** um `my-pokemon` for criado  
**Então:** ele deve possuir relacionamento com apenas um `pokemon`

---

### Regra 3 — Nickname

**Dado que:** o usuário está criando um `my-pokemon`  
**Quando:** informar um nickname  
**Então:** o nickname deve ser persistido no `my-pokemon`

---

### Regra 4 — Data de captura

**Dado que:** um `my-pokemon` é criado  
**Quando:** for persistido  
**Então:** deve registrar `created_at` e `captured_at`

---

### Regra 5 — Geração de atributos

**Dado que:** um `my-pokemon` está sendo criado  
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

### Regra 6 — Movimentos iniciais

**Dado que:** o Pokémon base possui movimentos disponíveis  
**Quando:** um `my-pokemon` for criado  
**Então:** o sistema deve selecionar aleatoriamente 4 movimentos distintos

---

### Regra 7 — PP dos movimentos

**Dado que:** os movimentos foram selecionados  
**Quando:** forem associados ao `my-pokemon`  
**Então:** cada movimento deve receber um PP aleatório entre 5 e 15

---

### Regra 8 — Controle de uso de movimento

**Dado que:** um movimento possui PP  
**Quando:** ele for usado em batalha futuramente  
**Então:** o PP deverá diminuir para impedir uso infinito

---

### Regra 9 — Preparação para troca futura de movimentos

**Dado que:** o `my-pokemon` subir de nível futuramente  
**Quando:** uma troca de movimento for permitida  
**Então:** o sistema deverá conseguir sugerir um movimento aleatório diferente dos movimentos atuais

---

## 6. Escopo Técnicos

### 6.1 API

Toda regra de negócio deve estar na API.  
O frontend deve ser apenas consumidor.

#### Funcionalidade

- endpoint para criar `my-pokemon`
- endpoint para listar `my-pokemon` do treinador autenticado
- endpoint para detalhar `my-pokemon`
- relacionamento com `trainer`
- relacionamento com `pokemon`
- relacionamento com movimentos selecionados

#### Fluxos

1. treinador escolhe Pokémon base
2. usuário informa nickname, se desejar
3. API cria `my-pokemon`
4. API gera atributos aleatórios
5. API seleciona 4 movimentos distintos do Pokémon base
6. API gera PP entre 5 e 15 para cada movimento
7. API persiste dados e retorna contrato normalizado

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
- captured_at
- created_at
- updated_at
- deleted_at

Para movimentos do `my-pokemon`, persistir:

- my_pokemon_id
- move_id
- pp
- created_at
- updated_at
- deleted_at

#### 6.1.3 Cache

Não necessário neste momento.

#### 6.1.4 Consistência e performance

- Garantir que cada `my-pokemon` pertença a um único `trainer`
- Garantir que cada `my-pokemon` tenha apenas um `pokemon`
- Garantir 4 movimentos distintos
- Garantir PP mínimo 5 e máximo 15
- Evitar duplicação de movimentos para o mesmo `my-pokemon`
- Utilizar constraints únicas quando necessário
- Evitar N+1 queries
- Garantir soft delete nas novas tabelas

#### 6.1.5 Contrato da API

A API deve ser a fonte única de verdade.

A API deve retornar dados normalizados contendo:

- dados do `my-pokemon`
- dados básicos do `pokemon`
- dados básicos do `trainer`
- atributos gerados
- movimentos associados
- PP atual de cada movimento

### 6.2 WEB

#### 6.2.1 Páginas

- listagem dos meus Pokémon
- detalhe do meu Pokémon
- criação/captura de my-pokemon

#### 6.2.2 Componentes

- cards de my-pokemon
- formulário/input de nickname
- seleção de Pokémon base
- exibição de atributos
- lista de movimentos com PP
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
- movimentos
- PP de cada movimento
- data de captura

#### 6.2.4 Interações

- criar my-pokemon
- escolher nickname
- visualizar detalhe
- navegar para listagem
- visualizar movimentos e PP

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

- [ ] criação de `my-pokemon` funcional
- [ ] relacionamento com `trainer` funcionando
- [ ] relacionamento com `pokemon` funcionando
- [ ] nickname persistido corretamente
- [ ] `captured_at` registrado corretamente
- [ ] atributos aleatórios gerados corretamente
- [ ] 4 movimentos distintos atribuídos
- [ ] PP gerado entre 5 e 15
- [ ] movimentos persistidos separadamente
- [ ] listagem funcionando
- [ ] detalhe funcionando
- [ ] API retornando dados normalizados
- [ ] sem duplicação de movimentos
- [ ] sem regressões
- [ ] novas tabelas possuem suporte a soft delete
- [ ] consultas padrão ignoram registros deletados logicamente
- [ ] nenhuma exclusão física foi implementada sem justificativa

## 9. Plano de Validação

### Validação manual

1. Criar um treinador
2. Selecionar um Pokémon base
3. Criar um `my-pokemon` com nickname
4. Validar atributos gerados
5. Validar `captured_at`
6. Validar 4 movimentos distintos
7. Validar PP entre 5 e 15
8. Acessar listagem
9. Acessar detalhe

### Testes automatizados

- criação de `my-pokemon`
- relacionamento com `trainer`
- relacionamento com `pokemon`
- geração de atributos
- seleção de movimentos distintos
- geração de PP
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