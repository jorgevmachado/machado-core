# Proposta: my-pokemon-level-progression

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

Já existem propostas anteriores relacionadas ao fluxo de exploração, batalha, captura, healing, Home, Party, MyPokemon e Pokedex:

* `5_my-pokemon-features`
* `6_pokedex-features`
* `7_trainer-encounter-exploration`
* `8_trainer-home-and-party-refactor`
* `9_wild-pokemon-battle-session`
* `10_pokemon-capture-system`
* `11_pokemon-center-healing`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de captura;
* recriar o sistema de healing;
* recriar o sistema de exploração;
* recriar o sistema de party;
* duplicar lógica de atributos;
* criar implementação paralela de progressão.

Esta proposta deve apenas:

* implementar progressão de level;
* implementar ganho de experiência;
* recalcular atributos dos `my-pokemon`;
* integrar com battle-session;
* integrar com Home e Party;
* preparar estrutura futura para evolução e novos movimentos.

---

# 1. Objetivo

Implementar o sistema de progressão de nível dos `my-pokemon`, permitindo que Pokémon do treinador ganhem experiência após batalhas e evoluam seus atributos.

O sistema deve:

* adicionar experiência;
* calcular level up;
* recalcular atributos;
* atualizar HP e MAX_HP;
* registrar progressão;
* sincronizar Home e Party;
* preparar arquitetura para evolução futura;
* preparar arquitetura para aprendizado de movimentos.

---

# 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado:

  * MyPokemon
  * BattleSession
  * TrainerParty
  * TrainerHome
  * PokemonProgression
* Comportamento atual:

  * existe battle-session;
  * existe captura de Pokémon;
  * existe MyPokemon;
  * existe Party;
  * existe Home agregadora;
* Problema identificado:

  * Pokémon não ganham experiência;
  * level não evolui;
  * atributos permanecem estáticos;
  * não existe progressão persistida;
  * não existe domínio isolado de progressão.

---

# 3. Comportamento Esperado

* O Pokémon deve ganhar experiência após batalhas.
* O sistema deve calcular level up automaticamente.
* O sistema deve recalcular atributos ao subir de nível.
* O HP atual deve respeitar o novo MAX_HP.
* O sistema deve registrar histórico de progressão.
* O sistema deve impedir overflow de level.
* O sistema deve sincronizar os dados da Home.
* O sistema deve atualizar a Party após level up.
* O sistema deve preparar estrutura futura para:

  * evolução;
  * aprendizado de movimentos;
  * habilidades passivas;
  * boosts temporários.

---

# 4. Escopo

## Dentro do escopo

* Sistema de experiência
* Sistema de level up
* Recalculo de atributos
* Integração com battle-session
* Integração com Home
* Integração com Party
* Logs de progressão
* Histórico de level
* Estrutura futura para evolução
* Estrutura futura para novos movimentos

---

## Fora do escopo

* Sistema de evolução
* Sistema de novos movimentos
* Sistema competitivo
* Sistema de IV/EV
* Sistema de habilidades passivas
* Sistema de prestige
* Sistema de rebirth
* Sistema de árvore de talentos

---

# 5. Regras de Negócio

## Regra 1 — Ganho de experiência

**Dado que:** uma batalha foi finalizada
**Quando:** o treinador vencer a batalha
**Então:** os Pokémon participantes devem receber experiência

---

## Regra 2 — Level up automático

**Dado que:** o Pokémon atingiu a experiência necessária
**Quando:** a experiência for processada
**Então:** o sistema deve utilizar a fórmula existente em `pokemon.growth_rate.formula` para validar se o Pokémon possui experiência suficiente para subir de nível e então aumentar o level automaticamente

---

## Regra 3 — Recalculo de atributos

**Dado que:** o Pokémon subiu de nível
**Quando:** o level up ocorrer
**Então:** os atributos devem ser recalculados

---

## Regra 4 — Atualização de HP

**Dado que:** o MAX_HP foi alterado
**Quando:** o level up finalizar
**Então:** o HP atual deve ser atualizado proporcionalmente

---

## Regra 5 — Limite de level

**Dado que:** o Pokémon atingiu o level máximo
**Quando:** receber experiência
**Então:** o sistema deve impedir evolução adicional de level

---

## Regra 6 — Histórico de progressão

**Dado que:** o Pokémon recebeu experiência
**Quando:** o processamento finalizar
**Então:** o sistema deve registrar o histórico da progressão

---

## Regra 7 — Battle-session ativa

**Dado que:** a batalha ainda está ativa
**Quando:** a experiência for calculada
**Então:** o sistema deve aguardar o encerramento da battle-session

---

# 6. Escopo Técnicos

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint para consultar progressão
* endpoint para histórico de level
* integração com battle-session
* integração com my-pokemon
* integração com Home
* integração com Party

### Fluxos

1. batalha é encerrada
2. sistema calcula experiência
3. sistema adiciona XP
4. sistema verifica level up
5. sistema recalcula atributos
6. sistema atualiza Home e Party
7. sistema registra logs

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

### my_pokemon

Atualizar:

* level
* experience
* hp
* max_hp
* attack
* defense
* special_attack
* special_defense
* speed
* updated_at

---

### pokemon_progression_log

Persistir:

* trainer_id
* my_pokemon_id
* old_level
* new_level
* gained_experience
* payload
* created_at

---

### growth_rate

Consumir:

* formula

A fórmula presente em `pokemon.growth_rate.formula` deve ser utilizada como fonte única de verdade para validar progressão de experiência e level up.

A implementação deve reutilizar a estrutura já existente do domínio `growth_rate`, evitando duplicação de fórmulas no domínio de progressão.

O sistema não deve criar fórmulas hardcoded para cálculo de level.

---

### battle_reward

Persistir:

* battle_session_id
* my_pokemon_id
* experience_gained
* created_at

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:party:{trainer_id}
trainer:my-pokemon:{trainer_id}
trainer:progression:{trainer_id}
```

Regras:

* invalidar cache após level up
* invalidar cache da Home
* invalidar cache da Party
* invalidar cache dos MyPokemon

---

## 6.1.4 Consistência e performance

* evitar level up duplicado
* evitar inconsistência de atributos
* evitar inconsistência de experiência
* manter integridade referencial
* utilizar soft delete
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre battle e progression
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
  "progression_result": {
    "success": true,
    "level_up": true,
    "old_level": 5,
    "new_level": 6,
    "experience_gained": 120
  }
}
```

---

# 6.2 WEB

## 6.2.1 Páginas

* detalhe do my-pokemon
* histórico de progressão
* resultado da batalha

---

## 6.2.2 Componentes

* barra de experiência
* indicador de level
* animação de level up
* card de atributos
* histórico de progressão
* resumo de recompensa

Estados:

* loading
* erro
* vazio
* sucesso

---

## 6.2.3 Renderização

A tela de progressão deve exibir:

* level atual
* experiência atual
* experiência necessária
* atributos atualizados
* histórico de level up
* recompensa da batalha

---

## 6.2.4 Interações

* visualizar experiência
* visualizar histórico de progressão
* visualizar level up
* atualizar Home
* atualizar Party

---

# 7. Restrições Arquiteturais

* Não criar nova arquitetura
* Não duplicar lógica entre API e WEB
* Toda regra de negócio deve estar na API
* Reutilizar padrões existentes
* Minimizar impacto no código atual
* Não reimplementar battle-session
* Não duplicar regras de progressão
* Não duplicar fórmulas de growth_rate no domínio de progressão
* O domínio de progressão deve consumir o growth_rate já existente
* Não criar services gigantes
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] ganho de experiência funcionando
* [ ] cálculo de level funcionando
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
4. Vencer batalha
5. Validar ganho de experiência
6. Validar level up
7. Validar atualização de atributos
8. Validar atualização da Home
9. Validar atualização da Party
10. Validar logs de progressão

---

## Testes automatizados

* ganho de experiência
* cálculo de level
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
15. Estratégia futura para evolução, novos movimentos e habilidades passivas
