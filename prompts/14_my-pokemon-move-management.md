# Proposta: my-pokemon-move-management

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

* [ARCHITECTURE-API](../docs/architecture-api.md)
* [ARCHITECTURE-WEB](../docs/architecture-web.md)
* [AGENTS API](../machado-api/AGENTS.md)
* [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já existir um definido.

---

## Observação importante

Já existem propostas anteriores relacionadas ao fluxo de exploração, batalha, captura, healing, progressão, evolução, Home, Party, MyPokemon e Pokedex:

* `5_my-pokemon-features`
* `6_pokedex-features`
* `7_trainer-encounter-exploration`
* `8_trainer-home-and-party-refactor`
* `9_wild-pokemon-battle-session`
* `10_pokemon-capture-system`
* `11_pokemon-center-healing`
* `12_my-pokemon-level-progression`
* `13_my-pokemon-evolution-system`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de progressão;
* recriar o sistema de evolução;
* recriar o sistema de party;
* duplicar lógica de movimentos;
* criar implementação paralela de gerenciamento de moves.

Esta proposta deve apenas:

* implementar gerenciamento de movimentos;
* implementar aprendizado de novos movimentos;
* implementar substituição de movimentos;
* integrar com progression e evolution;
* sincronizar Home e Party;
* preparar arquitetura para habilidades especiais futuras.

---

# 1. Objetivo

Implementar o sistema de gerenciamento de movimentos dos `my-pokemon`, permitindo que Pokémon aprendam, esqueçam e substituam movimentos durante sua progressão.

O sistema deve:

* permitir aprendizado de novos movimentos;
* limitar quantidade máxima de movimentos;
* permitir substituição de movimentos antigos;
* controlar PP e MAX_PP;
* registrar histórico de aprendizado;
* sincronizar Home e Party;
* preparar arquitetura para habilidades especiais futuras.

---

# 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado:

  * MyPokemon
  * PokemonMove
  * PokemonProgression
  * PokemonEvolution
  * TrainerParty
  * TrainerHome
* Comportamento atual:

  * existe sistema de batalha;
  * existe controle de PP;
  * existe progressão de level;
  * existe evolução;
  * existe MyPokemon;
* Problema identificado:

  * Pokémon não aprendem novos movimentos;
  * não existe substituição de movimentos;
  * não existe limite formal de gerenciamento;
  * não existe histórico de aprendizado;
  * não existe domínio isolado para gerenciamento de moves.

---

# 3. Comportamento Esperado

* O Pokémon deve conseguir aprender novos movimentos.
* O sistema deve limitar a quantidade máxima de movimentos ativos.
* O sistema deve permitir substituição de movimentos antigos.
* O sistema deve impedir movimentos duplicados.
* O sistema deve controlar PP e MAX_PP.
* O sistema deve registrar histórico de aprendizado.
* O sistema deve sincronizar os dados da Home.
* O sistema deve atualizar a Party após alteração de movimentos.
* O sistema deve preparar estrutura futura para:

  * TM/HM;
  * habilidades especiais;
  * movimentos exclusivos;
  * efeitos secundários avançados.

---

# 4. Escopo

## Dentro do escopo

* Sistema de aprendizado de movimentos
* Sistema de substituição de movimentos
* Controle de limite de movimentos
* Controle de PP
* Controle de MAX_PP
* Integração com progression
* Integração com evolution
* Integração com Home
* Integração com Party
* Histórico de movimentos
* Estrutura futura para habilidades especiais

---

## Fora do escopo

* Sistema de TM/HM
* Sistema de breeding
* Sistema competitivo
* Sistema de habilidades passivas
* Sistema de movimentos especiais lendários
* Sistema de crafting de movimentos
* Sistema de árvore de habilidades

---

# 5. Regras de Negócio

## Regra 1 — Limite máximo de movimentos

**Dado que:** o Pokémon possui movimentos ativos
**Quando:** tentar aprender um novo movimento
**Então:** o sistema deve limitar a quantidade máxima de movimentos ativos em 4

---

## Regra 2 — Aprendizado automático

**Dado que:** o Pokémon atingiu os requisitos necessários
**Quando:** a progressão for processada
**Então:** o sistema deve validar se o Pokémon pode aprender novos movimentos

---

## Regra 3 — Substituição de movimento

**Dado que:** o Pokémon já possui 4 movimentos ativos
**Quando:** tentar aprender um novo movimento
**Então:** o sistema deve permitir substituir um movimento existente

---

## Regra 4 — Movimentos duplicados

**Dado que:** o Pokémon já possui determinado movimento
**Quando:** tentar aprender o mesmo movimento novamente
**Então:** o sistema deve impedir duplicação

---

## Regra 5 — Controle de PP

**Dado que:** um movimento foi aprendido
**Quando:** o aprendizado finalizar
**Então:** o sistema deve inicializar `current_pp` e `max_pp`

---

## Regra 6 — Histórico de aprendizado

**Dado que:** o Pokémon aprendeu ou substituiu um movimento
**Quando:** o processamento finalizar
**Então:** o sistema deve registrar histórico da alteração

---

## Regra 7 — Battle-session ativa

**Dado que:** existe uma battle-session ativa
**Quando:** o gerenciamento de movimentos for executado
**Então:** o sistema deve impedir alteração de movimentos durante a batalha

---

# 6. Escopo Técnicos

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint para listar movimentos
* endpoint para aprender movimento
* endpoint para substituir movimento
* endpoint para histórico de movimentos
* integração com progression
* integração com evolution
* integração com Home
* integração com Party

### Fluxos

1. Pokémon sobe de nível
2. sistema valida novos movimentos disponíveis
3. sistema processa aprendizado
4. sistema valida limite de movimentos
5. sistema substitui movimento se necessário
6. sistema atualiza Home e Party
7. sistema registra logs

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

### my_pokemon_move

Atualizar:

* move_id
* current_pp
* max_pp
* slot
* updated_at

---

### pokemon_move_learn_log

Persistir:

* trainer_id
* my_pokemon_id
* old_move_id
* new_move_id
* action_type
* payload
* created_at

---

### pokemon_move_requirement

Consumir:

* minimum_level
* move_id
* learn_method

A implementação deve reutilizar a estrutura já existente dos movimentos persistidos localmente.

O sistema não deve criar regras hardcoded de aprendizado.

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:party:{trainer_id}
trainer:my-pokemon:{trainer_id}
trainer:moves:{trainer_id}
```

Regras:

* invalidar cache após alteração de movimentos
* invalidar cache da Home
* invalidar cache da Party
* invalidar cache dos MyPokemon

---

## 6.1.4 Consistência e performance

* evitar movimentos duplicados
* evitar inconsistência de PP
* evitar inconsistência de slots
* manter integridade referencial
* utilizar soft delete
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre progression, evolution e move-management
* evitar services gigantes

---

## 6.1.5 Contrato da API

A API deve:

* retornar dados normalizados;
* evitar transformação no frontend;
* manter consistência de naming.

Exemplo conceitual:

```json
{
  "move_management_result": {
    "success": true,
    "learned_move": {},
    "replaced_move": {}
  }
}
```

---

# 6.2 WEB

## 6.2.1 Páginas

* detalhe do my-pokemon
* gerenciamento de movimentos
* histórico de movimentos

---

## 6.2.2 Componentes

* lista de movimentos
* seletor de substituição
* indicador de PP
* indicador de MAX_PP
* histórico de movimentos
* feedback de aprendizado

Estados:

* loading
* erro
* vazio
* sucesso

---

## 6.2.3 Renderização

A tela de gerenciamento de movimentos deve exibir:

* movimentos atuais
* PP atual
* MAX_PP
* movimentos disponíveis
* histórico de aprendizado
* resumo das alterações

---

## 6.2.4 Interações

* aprender movimento
* substituir movimento
* visualizar histórico
* atualizar Home
* atualizar Party

---

# 7. Restrições Arquiteturais

* Não criar nova arquitetura
* Não duplicar lógica entre API e WEB
* Toda regra de negócio deve estar na API
* Reutilizar padrões existentes
* Minimizar impacto no código atual
* Não reimplementar progression
* Não reimplementar evolution
* Não duplicar regras de movimentos
* Não criar services gigantes
* Não duplicar regras de aprendizado
* O domínio de gerenciamento de movimentos deve consumir as estruturas já existentes
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] aprendizado de movimentos funcionando
* [ ] substituição de movimentos funcionando
* [ ] limite de 4 movimentos funcionando
* [ ] controle de PP funcionando
* [ ] controle de MAX_PP funcionando
* [ ] integração com Party funcionando
* [ ] atualização da Home funcionando
* [ ] logs funcionando
* [ ] payload normalizado
* [ ] integração com my-pokemon funcionando
* [ ] sem duplicação de lógica
* [ ] sem regressões
* [ ] cache funcionando
* [ ] services organizados corretamente
* [ ] sem services excessivamente grandes
* [ ] novas tabelas possuem suporte a soft delete
* [ ] consultas padrão ignoram registros deletados logicamente
* [ ] nenhuma exclusão física foi implementada sem justificativa

---

# 9. Plano de Validação

## Validação manual

1. Criar treinador
2. Criar Pokémon
3. Subir de nível
4. Validar aprendizado de movimento
5. Validar limite de 4 movimentos
6. Validar substituição
7. Validar PP
8. Validar atualização da Home
9. Validar atualização da Party
10. Validar logs de movimentos

---

## Testes automatizados

* aprendizado de movimentos
* substituição de movimentos
* limite de movimentos
* controle de PP
* integração com my-pokemon
* integração com Home
* atualização de cache
* cenários de erro
* edge cases

---

# 10. Saída Esperada do OpenSpec

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
10. Estratégia de cache
11. Estratégia de rollback
12. Riscos da implementação
13. Dependências entre domínios
14. Estratégia de separação de responsabilidades
15. Estratégia futura para TM/HM, habilidades especiais e efeitos avançados
