# Proposta: Atributos de relacionamentos de Pokemon

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

Implementar páginas de listagem e detalhes exclusivas para `PokemonType`, `PokemonAbility` e `PokemonMove`.
A feature deve permitir:

- listar PokémonType com paginação e filtros;
- visualizar detalhes de um PokémonType;
- listar PokémonMove com paginação e filtros;
- visualizar detalhes de um PokémonMove;
- listar PokémonAbility com paginação e filtros;
- visualizar detalhes de um PokémonAbility;
- Não Alterar o machado-api sómente o machado-web.
---

## 2. Contexto Atual
- Projeto: machado (monorepo API)
- Domínio afetado: PokemonType, PokemonAbility, PokemonMove
- Comportamento atual: não existe listagem nem detalhamento
- Problema identificado: ausência de páginas de listagem e detalhes para esses relacionamentos

---

## 3. Comportamento Esperado

- O sistema deve listar PokémonType com paginação e filtros
- O sistema deve listar PokémonMove com paginação e filtros
- O sistema deve listar PokémonAbility com paginação e filtros
- O usuário deve conseguir visualizar detalhes completos de um PokémonType
- O usuário deve conseguir visualizar detalhes completos de um PokémonMove
- O usuário deve conseguir visualizar detalhes completos de um PokémonAbility
- A interface deve exibir informações ricas e normalizadas

---

## 4. Escopo

### Dentro do escopo

- Consumo de API já existente de paginação de PokemonType, PokemonAbility e PokemonMove
- Consumo de API já existente de detalhamento de PokemonType, PokemonAbility e PokemonMove

### Fora do escopo

- Autenticação
- Refatorações globais
- Mudanças de arquitetura
- Alteração na API (machado-api)

---

## 5. Regras de Negócio

### Regra 1 — Acesso a página de listagem de tipos de pokémon

**Dado que:** Carrega a página de listagem de tipos de pokémon  
**Quando:** O Usuário acessa a página de listagem de tipos de pokémon  
**Então:** Será apresentada uma página com filtros e uma listagem  em formato de cards similar a página de listagem de Pokémon Atual  

---

### Regra 2 — Ao clicar em um card dentro da listagem de tipos de pokémon

**Dado que:** Redireciona para a página de detalhes do tipo de pokémon  
**Quando:** O Usuário clica em um card dentro da listagem de tipos de pokémon  
**Então:** Será apresentada uma página de detalhes do tipo de pokémon com informações ricas e normalizadas, similar a página de detalhes de Pokémon Atual

---

### Regra 3 — Acesso a página de listagem de Habilidades de pokémon

**Dado que:** Carrega a página de listagem de Habilidades de pokémon  
**Quando:** O Usuário acessa a página de listagem de Habilidades de pokémon  
**Então:** Será apresentada uma página com filtros e uma listagem  em formato de cards similar a página de listagem de Pokémon Atual  

---

### Regra 4 — Ao clicar em um card dentro da listagem de Habilidades de pokémon

**Dado que:** Redireciona para a página de detalhes de Habilidades de pokémon  
**Quando:** O Usuário clica em um card dentro da listagem de Habilidades de pokémon  
**Então:** Será apresentada uma página de detalhes de Habilidades de pokémon com informações ricas e normalizadas, similar a página de detalhes de Pokémon Atual

---

### Regra 5 — Acesso a página de listagem de Movimentos de pokémon

**Dado que:** Carrega a página de listagem de Movimentos de pokémon  
**Quando:** O Usuário acessa a página de listagem de Movimentos de pokémon  
**Então:** Será apresentada uma página com filtros e uma listagem  em formato de cards similar a página de listagem de Pokémon Atual  

---

### Regra 6 — Ao clicar em um card dentro da listagem de Movimentos de pokémon

**Dado que:** Redireciona para a página de detalhes de Movimentos de pokémon  
**Quando:** O Usuário clica em um card dentro da listagem de Movimentos de pokémon  
**Então:** Será apresentada uma página de detalhes de Movimentos de pokémon com informações ricas e normalizadas, similar a página de detalhes de Pokémon Atual

---

### Regra 7 — Apresentação no sidebar

**Dado que:** O Usuário acessa qualquer página do sistema
**Quando:** Carrega qualquer página.  
**Então:** Deve apresentar no sidebar de forma que seja filho de pokémon os links das listagens de tipos, habilidades e movimentos de pokémon com ícones que representem cada um.

---
## 6. Escopo Técnicos

### 6.1 API
Consumir API já existente de paginação de PokemonType, PokemonAbility e PokemonMove

---
### 6.2 WEB
#### 6.2.1 Páginas
- listagem de tipos de Pokémon
- detalhe de tipo de Pokémon
- listagem de movimentos de Pokémon
- detalhe de movimento de Pokémon
- listagem de habilidades de Pokémon
- detalhe de habilidade de Pokémon

---

#### 6.2.2 Componentes
- cards de Pokémon (Usar Componente de Card existente, se ncessário melhorar.)
- filtros (Usar Componente de Filtro existente, se necessário melhorar.)
- timeline de evolução

Estados:
- loading
- erro
- vazio
- sucesso

---

#### 6.2.3 Interações
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
- [ ] Página de listagem funcional de Tipos de Pokémon
- [ ] Página de listagem funcional de Habilidades de Pokémon
- [ ] Página de listagem funcional de Movimentos de Pokémon
- [ ] Página de detalhes funcional de Tipos de Pokémon
- [ ] Página de detalhes funcional de Habilidades de Pokémon
- [ ] Página de detalhes funcional de Movimentos de Pokémon
- [ ] filtros funcionando
- [ ] sincronização automática
- [ ] sem duplicação
- [ ] detalhe completo
- [ ] UI consistente
- [ ] sem regressões
- [ ] Cobertura de testes adequada ao padrão do projeto
- [ ] Testes cobrindo fluxos principais e cenários críticos
- [ ] Novas tabelas possuem suporte a soft delete
- [ ] Consultas padrão ignoram registros deletados logicamente
- [ ] Nenhuma exclusão física foi implementada sem justificativa

## 9. Plano de Validação

Testes automatizados
- cenário principal
- erro
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
7. Alterações de UI  
8. Estratégia de testes  
9. Riscos e pontos de atenção
