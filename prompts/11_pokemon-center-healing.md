# Proposta: pokemon-center-healing

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

Já existem propostas anteriores relacionadas ao fluxo de exploração, batalha, captura, Home, Party, MyPokemon e Pokedex:

* `5_my-pokemon-features`
* `6_pokedex-features`
* `7_trainer-encounter-exploration`
* `8_trainer-home-and-party-refactor`
* `9_wild-pokemon-battle-session`
* `10_pokemon-capture-system`

Esta proposta NÃO deve:

* recriar o sistema de batalha;
* recriar o sistema de captura;
* recriar o sistema de exploração;
* recriar o sistema de party;
* recriar o sistema de Home;
* duplicar lógica de recuperação de HP e PP;
* criar implementação paralela de restauração.

Esta proposta deve apenas:

* implementar o centro Pokémon;
* restaurar HP e PP dos `my-pokemon`;
* integrar com o sistema de party;
* integrar com a Home agregadora;
* preparar estrutura futura para revive e cura avançada;
* manter separação correta entre battle, healing e progression.

---

# 1. Objetivo

Implementar o sistema de Centro Pokémon, permitindo que o treinador recupere completamente seus Pokémon da party principal.

O sistema deve:

* restaurar HP;
* restaurar PP dos movimentos;
* remover status temporários futuros;
* atualizar dados da Home;
* sincronizar cache;
* preparar arquitetura para futuras mecânicas de revive e cura avançada.

---

# 2. Contexto Atual

* Projeto: machado (monorepo API + WEB)
* Domínio afetado:

  * Trainer
  * TrainerParty
  * MyPokemon
  * BattleSession
  * PokemonCenter
* Comportamento atual:

  * existe sistema de batalha;
  * existe consumo de HP;
  * existe consumo de PP;
  * existe party principal;
  * existe Home agregadora;
* Problema identificado:

  * não existe sistema de restauração;
  * Pokémon permanecem danificados após batalhas;
  * PP dos movimentos não pode ser recuperado;
  * Home não possui fluxo de recuperação;
  * não existe domínio isolado para healing.

---

# 3. Comportamento Esperado

* O treinador deve conseguir acessar um Centro Pokémon.
* O sistema deve restaurar completamente o HP dos Pokémon da party principal.
* O sistema deve restaurar completamente o PP dos movimentos.
* O sistema deve atualizar o estado atual dos `my-pokemon`.
* O sistema deve impedir restauração parcial inconsistente.
* O sistema deve sincronizar os dados da Home.
* O sistema deve atualizar a party após restauração.
* O sistema deve registrar logs de healing.
* O sistema deve retornar payload normalizado para o frontend.
* O sistema deve preparar estrutura futura para:

  * revive;
  * status negativos;
  * itens de cura;
  * cooldown;
  * healing limitado.

---

# 4. Escopo

## Dentro do escopo

* Sistema de Centro Pokémon
* Restauração de HP
* Restauração de PP
* Integração com trainer-party
* Integração com Home
* Logs de healing
* Atualização de cache
* Estrutura futura para revive
* Estrutura futura para status negativos

---

## Fora do escopo

* Sistema de revive avançado
* Sistema de itens de cura
* Sistema de status negativos
* Sistema de enfermaria
* Sistema de cooldown
* Sistema de pagamento por healing
* Sistema de NPCs
* Sistema multiplayer
* Sistema competitivo

---

# 5. Regras de Negócio

## Regra 1 — Healing apenas para Pokémon do treinador

**Dado que:** o treinador possui Pokémon
**Quando:** solicitar healing
**Então:** apenas Pokémon pertencentes ao treinador devem ser restaurados

---

## Regra 2 — Healing da party principal

**Dado que:** existe uma party ativa
**Quando:** o healing for executado
**Então:** todos os Pokémon da party principal devem ser restaurados

---

## Regra 3 — Restauração de HP

**Dado que:** um Pokémon possui HP reduzido
**Quando:** o healing for processado
**Então:** o HP atual deve voltar para MAX_HP

---

## Regra 4 — Restauração de PP

**Dado que:** um movimento possui PP reduzido
**Quando:** o healing for processado
**Então:** o PP atual deve voltar para MAX_PP

---

## Regra 5 — Consistência da party

**Dado que:** a party foi restaurada
**Quando:** o processo finalizar
**Então:** todos os Pokémon da party devem permanecer sincronizados

---

## Regra 6 — Atualização da Home

**Dado que:** o healing foi concluído
**Quando:** a Home for carregada
**Então:** os dados restaurados devem refletir corretamente

---

## Regra 7 — Battle-session ativa

**Dado que:** existe uma battle-session ativa
**Quando:** o treinador tentar realizar healing
**Então:** o sistema deve impedir a operação

---

# 6. Escopo Técnicos

## 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

### Funcionalidades

* endpoint para healing da party
* endpoint para consultar histórico de healing
* integração com trainer-party
* integração com my-pokemon
* integração com Home

### Fluxos

1. treinador acessa Centro Pokémon
2. sistema carrega party principal
3. sistema restaura HP
4. sistema restaura PP
5. sistema registra logs
6. sistema atualiza cache
7. sistema retorna payload normalizado

---

## 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

## 6.1.2 Enriquecimento de dados

### pokemon_center_healing

Persistir:

* trainer_id
* healed_pokemon_quantity
* restored_hp
* restored_pp
* created_at

---

### my_pokemon

Atualizar:

* current_hp
* updated_at

---

### my_pokemon_move

Atualizar:

* current_pp
* updated_at

---

### healing_log

Persistir:

* trainer_id
* my_pokemon_id
* action_type
* payload
* created_at

---

## 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

```txt
trainer:home:{trainer_id}
trainer:party:{trainer_id}
trainer:my-pokemon:{trainer_id}
trainer:healing:{trainer_id}
```

Regras:

* invalidar cache após healing
* invalidar cache da Home
* invalidar cache da Party
* invalidar cache dos MyPokemon

---

## 6.1.4 Consistência e performance

* evitar healing duplicado
* evitar inconsistência de HP
* evitar inconsistência de PP
* manter integridade referencial
* utilizar soft delete
* evitar N+1 queries
* reutilizar estruturas existentes
* manter separação clara entre battle e healing
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
  "healing_result": {
    "success": true,
    "restored_pokemon": []
  }
}
```

---

# 6.2 WEB

## 6.2.1 Páginas

* Centro Pokémon
* histórico de healing

---

## 6.2.2 Componentes

* botão de healing
* card da party
* indicador de HP
* indicador de PP
* feedback de restauração
* histórico de healing

Estados:

* loading
* erro
* vazio
* sucesso

---

## 6.2.3 Renderização

A tela do Centro Pokémon deve exibir:

* Pokémon da party
* HP atual
* PP atual
* botão de restauração
* feedback visual da cura
* histórico de healing

---

## 6.2.4 Interações

* restaurar party
* visualizar HP restaurado
* visualizar PP restaurado
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
* Não duplicar regras de healing
* Não criar services gigantes
* Toda nova tabela criada no banco de dados deve suportar soft delete
* Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
* Usar o padrão existente do projeto para soft delete, preferencialmente com `deleted_at`
* Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

# 8. Critérios de Aceite

* [ ] healing funcionando
* [ ] restauração de HP funcionando
* [ ] restauração de PP funcionando
* [ ] integração com party funcionando
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
4. Consumir HP
5. Consumir PP
6. Acessar Centro Pokémon
7. Validar restauração de HP
8. Validar restauração de PP
9. Validar atualização da Home
10. Validar atualização da Party

---

## Testes automatizados

* healing da party
* restauração de HP
* restauração de PP
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
15. Estratégia futura para revive, itens de cura e status negativos
