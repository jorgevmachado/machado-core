# Proposta: trainer-features

---

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

* `../docs/architecture-api.md`
* `../docs/architecture-web.md`
* `../machado-api/AGENTS.md`
* `../machado-web/AGENTS.md`

As decisões devem respeitar esses documentos.
Não criar novos padrões se já existir um definido.

---

## 1. Objetivo

Implementar a entidade Trainer vinculada ao usuário, responsável por representar a progressão, captura e estatísticas do jogador.

---

## 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado: Trainer / User / My-Pokemon
* Comportamento atual: não existe entidade de treinador
* Problema identificado: ausência de camada de progressão e controle do jogador

---

## 3. Comportamento Esperado

* O sistema deve criar automaticamente um Trainer para cada usuário
* O Trainer deve possuir progressão (level)
* O Trainer deve controlar capacidade de captura (capture_rate)
* O Trainer deve possuir pokebolas para captura
* O Trainer deve armazenar estatísticas de uso e progresso
* O Trainer deve ser a entidade central para relacionamentos futuros (my-pokemon, batalhas, pokedex)

---

## 4. Escopo

### Dentro do escopo

* Criação da entidade Trainer
* Relacionamento 1:1 com User
* Controle de level
* Controle de capture_rate
* Controle de pokebolas
* Estrutura inicial de estatísticas
* Inicialização automática do treinador

### Fora do escopo

* Sistema de batalha
* Sistema de compra de pokebolas
* Sistema de pokedex (apenas preparar estrutura)
* Progressão avançada de level

---

## 5. Regras de Negócio

### Regra 1 — Criação automática

**Dado que:** um usuário é criado
**Quando:** finaliza cadastro
**Então:** deve ser criado automaticamente um Trainer vinculado

---

### Regra 2 — Relacionamento

**Dado que:** existe um usuário
**Quando:** possuir Trainer
**Então:** deve haver relação 1:1

* Um usuário → um trainer
* Um trainer → um usuário

---

### Regra 3 — Level inicial

**Dado que:** o Trainer é criado
**Quando:** inicializado
**Então:** deve começar com level base (ex: 1)

---

### Regra 4 — Progressão

**Dado que:** o Trainer participa de batalhas (futuro)
**Quando:** ganha experiência
**Então:** seu level deve aumentar

---

### Regra 5 — Capture rate

**Dado que:** o Trainer é criado
**Quando:** inicializado
**Então:** deve possuir `capture_rate = 45`

---

### Regra 6 — Pokebolas iniciais

**Dado que:** o Trainer é criado
**Quando:** inicializado
**Então:** deve possuir 5 pokebolas

---

### Regra 7 — Consumo de pokebola (preparação)

**Dado que:** um Pokémon for capturado (futuro)
**Quando:** ação executada
**Então:** deve consumir 1 pokebola

---

### Regra 8 — Pokedex

**Dado que:** o Trainer é criado
**Quando:** inicializado
**Então:** deve possuir uma pokedex vazia (estrutura futura)

---

### Regra 9 — Estatísticas

**Dado que:** o Trainer utiliza o sistema
**Quando:** interage (batalhas, capturas, etc)
**Então:** deve armazenar dados para estatísticas

---

## 6. Escopo Técnicos

---

### 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

---

#### Funcionalidade

* endpoint para obter Trainer do usuário autenticado
* endpoint para atualização básica (se necessário)
* criação automática no fluxo de usuário

---

#### Fluxos

* criação de usuário → criação automática de trainer
* acesso → retornar trainer
* futuras atualizações → refletir progressão

---

#### 6.1.1 Integrações externas

Não há integração externa neste momento.

---

#### 6.1.2 Enriquecimento de dados

Persistir:

* user_id
* level
* capture_rate
* pokeballs
* created_at
* estatísticas (campos preparados para expansão futura)

Exemplo de estatísticas futuras:

* total_capturas
* total_batalhas
* total_vitorias
* total_derrotas

---

#### 6.1.3 Cache

Não necessário neste momento.

---

#### 6.1.4 Consistência e performance

* garantir relacionamento 1:1 (constraint única)
* evitar duplicação de trainer por usuário
* operações idempotentes na criação
* evitar N+1 queries
* garantir integridade referencial

---

#### 6.1.5 Contrato da API

A API deve ser a fonte única de verdade.

Deve retornar:

* dados do trainer
* informações básicas do usuário (quando necessário)
* estatísticas do trainer

---

### 6.2 WEB

---

#### 6.2.1 Páginas

* perfil do treinador
* visualização de informações

---

#### 6.2.2 Componentes

* card do treinador
* exibição de level
* exibição de pokebolas
* estatísticas

Estados:

* loading
* erro
* vazio
* sucesso

---

#### 6.2.3 Renderização

Exibir:

* level
* quantidade de pokebolas
* capture_rate
* estatísticas

---

#### 6.2.4 Interações

* visualizar dados do treinador
* futuras interações (batalha, captura, loja)

---

## 7. Restrições Arquiteturais

- Não criar nova arquitetura
- Não duplicar lógica
- Reutilizar estruturas existentes
- Toda regra de negócio deve estar na API
- Toda nova tabela criada no banco de dados deve suportar soft delete.
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto.
- Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`.
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário.

---

## 8. Critérios de Aceite

- [ ] trainer criado automaticamente com usuário
- [ ] relacionamento 1:1 garantido
- [ ] level inicial correto
- [ ] capture_rate inicial correto
- [ ] pokebolas iniciais corretas
- [ ] estrutura de estatísticas criada
- [ ] endpoint de leitura funcionando
- [ ] sem duplicação de dados
- [ ] sem regressões
- [ ] testes cobrindo fluxos principais
- [ ] Novas tabelas possuem suporte a soft delete
- [ ] Consultas padrão ignoram registros deletados logicamente
- [ ] Nenhuma exclusão física foi implementada sem justificativa
---

## 9. Plano de Validação

### Validação manual

1. criar usuário
2. validar criação automática de trainer
3. validar dados iniciais
4. acessar endpoint de trainer

---

### Testes automatizados

* criação automática
* relacionamento 1:1
* valores iniciais
* erro de duplicação

---

## 10. Saída Esperada do OpenSpec

Gerar:

1. Resumo
2. Problema
3. Solução
4. Arquivos impactados
5. Plano de implementação
6. Alterações de modelo
7. Alterações de API
8. Alterações de UI
9. Testes
10. Riscos
