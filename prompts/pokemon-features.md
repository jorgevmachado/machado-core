# Proposta: listagem-e-detalhe-pokemon

## 0. Contexto obrigatório

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

### 6.2 WEB

#### 6.2.1 Páginas
- listagem de Pokémon
- detalhe do Pokémon

---

#### 6.2.2 Componentes
- cards de Pokémon
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