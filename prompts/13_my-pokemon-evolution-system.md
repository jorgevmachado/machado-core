# Proposta: my-pokemon-evolution-system

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

Já existem propostas anteriores relacionadas ao fluxo de exploração, batalha, captura, healing, progressão, Home, Party, MyPokemon e Pokedex:

* `5_my-pokemon-features`
* `6_pokedex-features`
* `7_trainer-encounter-exploration`
* `8_trainer-home-and-party-refactor`
* `9_wild-pokemon-battle-session`
* `10_pokemon-capture-system`
* `11_pokemon-center-healing`
* `12_my-pokemon-level-progression`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de captura;
* recriar o sistema de healing;
* recriar o sistema de progressão;
* recriar o sistema de party;
* duplicar lógica de atributos;
* criar implementação paralela de evolução.

Esta proposta deve apenas:

* implementar evolução de `my-pokemon`;
* integrar com growth_rate e progression;
* atualizar Pokémon base vinculado;
* recalcular atributos após evolução;
* sincronizar Home e Party;
* preparar estrutura futura para novos movimentos e formas especiais.

---

# 1. Objetivo

Implementar o sistema de evolução dos `my-pokemon`, permitindo que Pokémon do treinador evoluam automaticamente após atingirem os requisitos necessários.

O sistema deve:

* validar requisitos de evolução;
* atualizar Pokémon base vinculado;
* recalcular atributos;
* atualizar HP e MAX_HP;
* registrar histórico de evolução;
* sincronizar Home e Party;
* preparar arquitetura para novas formas de evolução;
* preparar arquitetura para aprendizado de movimentos.

---

# 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado:

  * MyPokemon
  * Pokemon
  * PokemonEvolution
  * PokemonProgression
  * TrainerParty
  * TrainerHome
* Comportamento atual:

  * existe progressão de level;
  * existe growth_rate;
  * existe battle-session;
  * existe Party;
  * existe Home agregadora;
* Problema identificado:

  * Pokémon não evoluem;
  * Pokémon base permanece estático;
  * atributos não acompanham evolução;
  * não existe histórico de evolução;
  * não existe domínio isolado de evolução.

---

# 3. Comportamento Esperado

* O Pokémon deve evoluir automaticamente ao atingir os requisitos necessários.
* O sistema deve validar regras de evolução.
* O sistema deve atualizar o Pokémon base vinculado ao `my-pokemon`.
* O sistema deve recalcular atributos após evolução.
* O HP atual deve respeitar o novo MAX_HP.
* O sistema deve registrar histórico de evolução.
* O sistema deve impedir evolução duplicada.
* O sistema deve sincronizar os dados da Home.
* O sistema deve atualizar a Party após evolução.
* O sistema deve preparar estrutura futura para:

  * mega evolution;
  * evolução por item;
  * evolução por troca;
  * evolução regional;
  * formas especiais.

---

# 4. Escopo

## Dentro do escopo

* Sistema de evolução
* Validação de requisitos
* Atualização do Pokémon base
* Recalculo de atributos
* Integração com progression
* Integração com Home
* Integração com Party
* Logs de evolução
* Histórico de evolução
* Estrutura futura para formas especiais
* Estrutura futura para novos movimentos

---

## Fora do escopo

* Mega evolution
* Evolução regional
* Evolução por item
* Evolução por troca
* Evolução temporária
* Sistema competitivo
* Sistema de IV/EV
* Sistema de breeding
* Sistema de prestige

---

# 5. Regras de Negócio

## Regra 1 — Evolução automática

**Dado que:** o Pokémon atingiu os requisitos de evolução
**Quando:** o sistema processar a progressão
**Então:** o Pokémon deve evoluir automaticamente

---

## Regra 2 — Uso da cadeia de evolução

**Dado que:** o Pokémon possui evolução cadastrada
**Quando:** a evolução for processada
**Então:** o sistema deve utilizar os relacionamentos já existentes da cadeia evolutiva persistida localmente

---

## Regra 3 — Atualização do Pokémon base

**Dado que:** o Pokémon evoluiu
**Quando:** a evolução finalizar
**Então:** o `my-pokemon` deve passar a apontar para o novo Pokémon base evoluído

---

## Regra 4 — Recalculo de atributos

**Dado que:** o Pokémon evoluiu
**Quando:** a evolução for concluída
**Então:** os atributos devem ser recalculados respeitando a nova forma evoluída

---

## Regra 5 — Atualização de HP

**Dado que:** o MAX_HP foi alterado
**Quando:** a evolução finalizar
**Então:** o HP atual deve ser atualizado proporcionalmente

---

## Regra 6 — Histórico de evolução

**Dado que:** o Pokémon evoluiu
**Quando:** o processamento finalizar
**Então:** o sistema deve registrar o histórico da evolução

---

## Regra 7 — Evolução única

**Dado que:** o Pokémon já evoluiu para determinada forma
**Quando:** o sistema validar evolução
**Então:** não deve ocorrer evolução duplicada

---

## Regra 8 — Battle-session ativa

**Dado que:** existe uma battle-session ativa
**Quando:** a evolução for validada
**Então:** o sistema deve aguardar o encerramento da battle-session

---

# 6. Escopo Técnicos

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint para consultar evolução
* endpoint para histórico de evolução
* integração com progression
* integração com my-pokemon
* integração com Home
* integração com Party

### Fluxos

1. batalha é encerrada
2. sistema calcula experiência
3. sistema processa progressão
4. sistema valida evolução
5. sistema atualiza Pokémon base
6. sistema recalcula atributos
7. sistema atualiza Home e Party
8. sistema registra logs

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

### my_pokemon

Atualizar:

* pokemon_id
* level
* hp
* max_hp
* attack
* defense
* special_attack
* special_defense
* speed
* updated_at

---

### pokemon_evolution_log

Persistir:

* trainer_id
* my_pokemon_id
* old_pokemon_id
* new_pokemon_id
* old_level
* new_level
* payload
* created_at

---

### evolution_requirement

Consumir:

* minimum_level
* evolution_chain
* evolution_trigger

A implementação deve reutilizar a estrutura já existente da cadeia evolutiva persistida localmente.

O sistema não deve criar regras hardcoded de evolução.

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:party:{trainer_id}
trainer:my-pokemon:{trainer_id}
trainer:evolution:{trainer_id}
```

Regras:

* invalidar cache após evolução
* invalidar cache da Home
* invalidar cache da Party
* invalidar cache dos MyPokemon

---

## 6.1.4 Consistência e performance

* evitar evolução duplicada
* evitar inconsistência de atributos
* evitar inconsistência de evolução
* manter integridade referencial
* utilizar soft delete
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre progression e evolution
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
  "evolution_result": {
    "success": true,
    "evolved": true,
    "old_pokemon": {},
    "new_pokemon": {}
  }
}
```

---

# 6.2 WEB

## 6.2.1 Páginas

* detalhe do my-pokemon
* histórico de evolução
* resultado da batalha

---

## 6.2.2 Componentes

* animação de evolução
* indicador de evolução
* card de atributos
* histórico de evolução
* resumo da evolução

Estados:

* loading
* erro
* vazio
* sucesso

---

## 6.2.3 Renderização

A tela de evolução deve exibir:

* Pokémon anterior
* Pokémon evoluído
* level atual
* atributos atualizados
* histórico de evolução
* resumo da evolução

---

## 6.2.4 Interações

* visualizar evolução
* visualizar histórico de evolução
* visualizar atributos atualizados
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
* Não duplicar regras de evolução
* Não criar services gigantes
* Não duplicar regras da cadeia evolutiva
* O domínio de evolução deve consumir a estrutura evolutiva já existente
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] evolução funcionando
* [ ] validação da cadeia evolutiva funcionando
* [ ] atualização do Pokémon base funcionando
* [ ] recalculo de atributos funcionando
* [ ] atualização de HP funcionando
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
2. Criar party
3. Entrar em batalha
4. Ganhar experiência
5. Validar level up
6. Validar evolução
7. Validar atualização do Pokémon base
8. Validar atualização de atributos
9. Validar atualização da Home
10. Validar atualização da Party
11. Validar logs de evolução

---

## Testes automatizados

* validação de evolução
* atualização do Pokémon base
* recalculo de atributos
* atualização de HP
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
15. Estratégia futura para mega evolution, formas especiais e novos movimentos
