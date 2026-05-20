# Proposta: trainer-progression-and-rewards

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

Já existem propostas anteriores relacionadas ao fluxo de exploração, batalha, captura, healing, progressão, evolução, gerenciamento de movimentos, Home, Party, MyPokemon e Pokedex:

* `5_my-pokemon-features`
* `6_pokedex-features`
* `7_trainer-encounter-exploration`
* `8_trainer-home-and-party-refactor`
* `9_wild-pokemon-battle-session`
* `10_pokemon-capture-system`
* `11_pokemon-center-healing`
* `12_my-pokemon-level-progression`
* `13_my-pokemon-evolution-system`
* `14_my-pokemon-move-management`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de progressão dos Pokémon;
* recriar o sistema de evolução;
* recriar o sistema de movimentos;
* recriar o sistema de party;
* duplicar lógica de recompensas;
* criar implementação paralela de progressão do treinador.

Esta proposta deve apenas:

* implementar progressão do treinador;
* implementar recompensas globais;
* integrar com battle-session e captura;
* integrar com Home;
* sincronizar estatísticas gerais do treinador;
* preparar arquitetura para achievements e badges futuras.

---

# 1. Objetivo

Implementar o sistema de progressão do treinador, permitindo que o treinador ganhe experiência global, suba de nível e receba recompensas por batalhas, capturas e progresso geral.

O sistema deve:

* adicionar experiência ao treinador;
* calcular level do treinador;
* desbloquear recompensas;
* atualizar estatísticas globais;
* registrar histórico de progressão;
* sincronizar Home;
* preparar arquitetura para achievements, badges e missões futuras.

---

# 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado:

  * Trainer
  * TrainerProgression
  * BattleSession
  * PokemonCapture
  * TrainerHome
  * TrainerReward
* Comportamento atual:

  * existe battle-session;
  * existe captura de Pokémon;
  * existe Home agregadora;
  * existe progressão dos Pokémon;
* Problema identificado:

  * treinador não possui progressão própria;
  * treinador não ganha experiência;
  * não existem recompensas globais;
  * estatísticas gerais não evoluem;
  * não existe domínio isolado para progressão do treinador.

---

# 3. Comportamento Esperado

* O treinador deve ganhar experiência global.
* O sistema deve calcular level do treinador automaticamente.
* O sistema deve desbloquear recompensas.
* O sistema deve atualizar estatísticas globais.
* O sistema deve registrar histórico de progressão.
* O sistema deve impedir overflow de level.
* O sistema deve sincronizar os dados da Home.
* O sistema deve atualizar informações globais do treinador.
* O sistema deve preparar estrutura futura para:

  * badges;
  * achievements;
  * quests;
  * ranking;
  * sistema competitivo.

---

# 4. Escopo

## Dentro do escopo

* Sistema de experiência do treinador
* Sistema de level do treinador
* Sistema de recompensas
* Estatísticas globais
* Integração com battle-session
* Integração com captura
* Integração com Home
* Logs de progressão
* Histórico de recompensas
* Estrutura futura para badges e achievements

---

## Fora do escopo

* Sistema de achievements
* Sistema de badges
* Sistema competitivo
* Sistema de quests
* Sistema de ranking
* Sistema de guildas
* Sistema multiplayer
* Sistema de temporadas

---

# 5. Regras de Negócio

## Regra 1 — Ganho de experiência do treinador

**Dado que:** uma batalha foi finalizada ou uma captura foi concluída
**Quando:** o sistema processar as recompensas
**Então:** o treinador deve receber experiência global

---

## Regra 2 — Level up automático do treinador

**Dado que:** o treinador atingiu a experiência necessária
**Quando:** a experiência for processada
**Então:** o sistema deve aumentar automaticamente o level do treinador

---

## Regra 3 — Recompensas globais

**Dado que:** o treinador subiu de nível
**Quando:** o processamento finalizar
**Então:** o sistema deve desbloquear recompensas configuradas para aquele level

---

## Regra 4 — Atualização de estatísticas

**Dado que:** ocorreu batalha, captura ou progressão
**Quando:** o processamento finalizar
**Então:** o sistema deve atualizar as estatísticas globais do treinador

---

## Regra 5 — Histórico de progressão

**Dado que:** o treinador recebeu experiência ou recompensa
**Quando:** o processamento finalizar
**Então:** o sistema deve registrar histórico da progressão

---

## Regra 6 — Limite de level

**Dado que:** o treinador atingiu o level máximo
**Quando:** receber experiência
**Então:** o sistema deve impedir progressão adicional de level

---

## Regra 7 — Battle-session ativa

**Dado que:** existe uma battle-session ativa
**Quando:** o sistema processar recompensas
**Então:** o processamento deve ocorrer apenas após encerramento da batalha

---

# 6. Escopo Técnicos

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint para consultar progressão do treinador
* endpoint para histórico de recompensas
* endpoint para estatísticas globais
* integração com battle-session
* integração com captura
* integração com Home

### Fluxos

1. batalha ou captura é finalizada
2. sistema calcula experiência global
3. sistema processa recompensas
4. sistema valida level up
5. sistema atualiza estatísticas
6. sistema atualiza Home
7. sistema registra logs

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

### trainer

Atualizar:

* level
* experience
* total_battles
* total_captures
* total_wins
* updated_at

---

### trainer_progression_log

Persistir:

* trainer_id
* old_level
* new_level
* gained_experience
* payload
* created_at

---

### trainer_reward

Persistir:

* trainer_id
* reward_type
* reward_value
* source_type
* source_id
* created_at

---

### trainer_statistics

Persistir:

* trainer_id
* total_battles
* total_wins
* total_losses
* total_captures
* total_experience
* updated_at

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:progression:{trainer_id}
trainer:statistics:{trainer_id}
trainer:rewards:{trainer_id}
```

Regras:

* invalidar cache após level up
* invalidar cache após recompensa
* invalidar cache da Home
* invalidar cache de estatísticas

---

## 6.1.4 Consistência e performance

* evitar recompensas duplicadas
* evitar inconsistência de experiência
* evitar inconsistência de estatísticas
* manter integridade referencial
* utilizar soft delete
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre progressão do treinador e progressão dos Pokémon
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
  "trainer_progression_result": {
    "success": true,
    "level_up": true,
    "old_level": 3,
    "new_level": 4,
    "experience_gained": 500,
    "rewards": []
  }
}
```

---

# 6.2 WEB

## 6.2.1 Páginas

* Home do treinador
* histórico de recompensas
* estatísticas do treinador

---

## 6.2.2 Componentes

* barra de experiência do treinador
* indicador de level
* resumo de recompensas
* card de estatísticas
* histórico de progressão
* histórico de recompensas

Estados:

* loading
* erro
* vazio
* sucesso

---

## 6.2.3 Renderização

A Home do treinador deve exibir:

* level atual do treinador
* experiência atual
* recompensas desbloqueadas
* estatísticas globais
* histórico de progressão
* resumo de batalhas e capturas

---

## 6.2.4 Interações

* visualizar progressão
* visualizar recompensas
* visualizar estatísticas
* atualizar Home

---

# 7. Restrições Arquiteturais

* Não criar nova arquitetura
* Não duplicar lógica entre API e WEB
* Toda regra de negócio deve estar na API
* Reutilizar padrões existentes
* Minimizar impacto no código atual
* Não reimplementar progressão dos Pokémon
* Não duplicar regras de recompensas
* Não criar services gigantes
* Não duplicar regras de estatísticas
* O domínio de progressão do treinador deve consumir as estruturas já existentes
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] experiência do treinador funcionando
* [ ] cálculo de level funcionando
* [ ] recompensas funcionando
* [ ] estatísticas globais funcionando
* [ ] atualização da Home funcionando
* [ ] logs funcionando
* [ ] payload normalizado
* [ ] integração com battle-session funcionando
* [ ] integração com captura funcionando
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
2. Entrar em batalha
3. Capturar Pokémon
4. Validar ganho de experiência global
5. Validar level up do treinador
6. Validar recompensas
7. Validar atualização da Home
8. Validar estatísticas globais
9. Validar logs de progressão

---

## Testes automatizados

* ganho de experiência global
* cálculo de level do treinador
* desbloqueio de recompensas
* atualização de estatísticas
* integração com Home
* integração com battle-session
* integração com captura
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
15. Estratégia futura para achievements, badges, ranking e quests
