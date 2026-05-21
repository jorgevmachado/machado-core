# Proposta: wild-pokemon-battle-session

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-API](../docs/architecture-api.md)
- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS API](../machado-api/AGENTS.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já existir um definido.

---

## Observação importante

Já existem propostas anteriores relacionadas ao fluxo de exploração, Home, Party, MyPokemon e Pokedex:

- 5_my-pokemon-features
- 6_pokedex-features
- 7_trainer-encounter-exploration
- 8_trainer-home-and-party-refactor

Esta proposta NÃO deve:

- recriar o sistema de exploração;
- recriar o sistema de party;
- recriar a Home;
- recriar MyPokemon;
- recriar Pokedex;
- duplicar regras de encounter;
- duplicar lógica de seleção de Pokémon;
- criar implementação paralela de exploração.

Esta proposta deve apenas:

- implementar a sessão de batalha contra Pokémon selvagem;
- integrar com o sistema de exploração já existente;
- reutilizar estruturas já criadas anteriormente;
- preparar a arquitetura para captura futura;
- preparar a arquitetura para progressão futura;
- manter isolamento correto entre os domínios.

Caso já existam implementações anteriores relacionadas a encounters ou exploração, reutilizar ao máximo as estruturas existentes.

---

## 1. Objetivo

Implementar o sistema de sessão de batalha contra Pokémon selvagem, permitindo que eventos de exploração iniciem batalhas stateful entre o treinador e um Pokémon selvagem encontrado.

A sessão de batalha deve:

- controlar o estado da batalha;
- controlar turnos;
- controlar Pokémon ativo;
- controlar HP temporário;
- controlar uso de movimentos e PP;
- preparar estrutura futura para captura;
- preparar estrutura futura para experiência e level up;
- retornar eventos normalizados para o frontend.

---

## 2. Contexto Atual
- Projeto: machado (monorepo API + WEB)
- Domínio afetado:
- - Trainer
- - TrainerParty
- - TrainerEncounter
- - MyPokemon
- - Pokemon
- - BattleSession
- Arquivos/pastas relevantes:
- - machado-api/app/domain
- - machado-api/app/models
- - machado-web/app
- - machado-web/app/ui/features
- Comportamento atual:
- - existe sistema de exploração;
- - existe geração de eventos aleatórios;
- - existe evento de Pokémon selvagem;
- - existe MyPokemon;
- - existe Party do treinador;
- - existe Home agregadora;
- Problema identificado:
- - não existe sessão de batalha;
- - não existe controle de turnos;
- - não existe controle de batalha stateful;
- - não existe gerenciamento de Pokémon ativo;
- - não existe fluxo intermediário entre exploration e captura futura.

---

## 3. Comportamento Esperado
- O sistema deve iniciar automaticamente uma sessão de batalha quando um Pokémon selvagem for encontrado.
- A batalha deve ser stateful.
- A sessão deve possuir um identificador único.
- O treinador deve utilizar um Pokémon da sua party principal.
- O sistema deve selecionar inicialmente o primeiro Pokémon ativo da party.
- O Pokémon selvagem deve possuir HP próprio durante a batalha.
- O Pokémon do treinador deve possuir HP próprio durante a batalha.
- O sistema deve controlar turnos.
- O sistema deve controlar uso de movimentos.
- O sistema deve controlar PP dos movimentos.
- O sistema deve impedir uso de movimento sem PP.
- O sistema deve permitir troca de Pokémon da party durante a batalha.
- O sistema deve permitir fuga da batalha.
- O sistema deve retornar eventos normalizados da batalha.
- A sessão deve possuir status:
- - ACTIVE
- - FINISHED
- - ESCAPED
- - WILD_POKEMON_DEFEATED
- - TRAINER_DEFEATED
- O sistema deve preparar estrutura futura para captura.
- O sistema deve preparar estrutura futura para experiência.
- O sistema deve preparar estrutura futura para evolução.

---

## 4. Escopo
Dentro do escopo
- Criação da entidade battle-session
- Criação da entidade battle-turn
- Criação da entidade battle-log
- Controle de batalha stateful
- Controle de Pokémon ativo
- Controle de HP temporário
- Controle de turnos
- Controle de uso de movimentos
- Controle de PP
- Troca de Pokémon
- Encerramento de batalha
- Fuga da batalha
- Integração com exploração
- Integração com Party
- Integração com MyPokemon
- Integração com eventos de exploration
- Estrutura preparada para captura futura
- Estrutura preparada para XP futura

---

Fora do escopo
- Sistema de captura
- Consumo de pokebolas
- Sistema de level up
- Sistema de experiência
- Evolução de Pokémon
- IA avançada de batalha
- Batalha PvP
- Batalha entre treinadores
- Efeitos complexos de moves
- Status negativos
- Weather system
- Sistema competitivo

---

## 5. Regras de Negócio
### Regra 1 — Início automático da batalha

**Dado que:** o treinador encontrou um Pokémon selvagem
**Quando:** o evento de exploração for processado
**Então:** o sistema deve iniciar automaticamente uma battle-session

---

### Regra 2 — Apenas uma batalha ativa

**Dado que:** o treinador já possui uma batalha ativa
**Quando:** tentar iniciar uma nova batalha
**Então:** o sistema deve impedir múltiplas sessões simultâneas

---

### Regra 3 — Pokémon inicial da batalha

**Dado que:** o treinador possui party ativa
**Quando:** a batalha iniciar
**Então:** o primeiro Pokémon ativo da party deve ser utilizado inicialmente

---

### Regra 4 — Controle de HP

**Dado que:** a batalha está ativa
**Quando:** um Pokémon receber dano
**Então:** o HP temporário da batalha deve ser atualizado

---

### Regra 5 — Controle de PP

**Dado que:** um movimento foi utilizado
**Quando:** a ação for processada
**Então:** o PP do movimento deve diminuir

---

### Regra 6 — Movimento sem PP

**Dado que:** um movimento está sem PP
**Quando:** o treinador tentar utilizá-lo
**Então:** o sistema deve impedir a ação

---

### Regra 7 — Turnos

**Dado que:** a batalha está ativa
**Quando:** uma ação for executada
**Então:** o sistema deve registrar o turno da batalha

---

### Regra 8 — Troca de Pokémon

**Dado que:** o treinador possui outros Pokémon na party
**Quando:** solicitar troca
**Então:** o sistema deve permitir alterar o Pokémon ativo

---

### Regra 9 — Encerramento da batalha

**Dado que:** a batalha terminou
**Quando:** um dos lados não possuir mais HP
**Então:** a sessão deve ser finalizada

---

### Regra 10 — Fuga

**Dado que:** a batalha está ativa
**Quando:** o treinador fugir
**Então:** a sessão deve ser encerrada com status ESCAPED

---

## 6. Escopo Técnicos
### 6.1 API

Toda regra de negócio deve estar na API.
O frontend deve ser apenas consumidor.

---

Funcionalidades
- endpoint para iniciar batalha
- endpoint para buscar batalha ativa
- endpoint para utilizar movimento
- endpoint para trocar Pokémon
- endpoint para fugir da batalha
- endpoint para listar logs da batalha

---

Fluxos
- treinador caminha
- exploration gera evento WILD_POKEMON
- sistema cria battle-session
- sistema seleciona Pokémon ativo
- treinador executa ações
- sistema processa turnos
- batalha é encerrada

--- 
#### 6.1.1 Integrações externas

Não consumir PokéAPI diretamente nesta proposta.

Utilizar apenas dados persistidos localmente.

---

#### 6.1.2 Enriquecimento de dados
battle_session

Persistir:

- trainer_id
- wild_pokemon_id
- trainer_active_my_pokemon_id
- status
- started_at
- finished_at
- escaped_at
- created_at
- updated_at
- deleted_at

---

battle_turn

Persistir:

- battle_session_id
- turn_number
- actor_type
- actor_id
- move_id
- damage
- created_at
- updated_at
- deleted_at

---

battle_log

Persistir:

- battle_session_id
- event_type
- payload
- created_at
- updated_at
- deleted_at

---

battle_pokemon_state

Persistir:

- battle_session_id
- my_pokemon_id
- current_hp
- is_active
- fainted
- created_at
- updated_at
- deleted_at

---

wild_pokemon_state

Persistir:

- battle_session_id
- pokemon_id
- current_hp
- max_hp
- level
- fainted
- created_at
- updated_at
- deleted_at

---

#### 6.1.3 Cache

Utilizar o padrão existente do projeto.

Sugestão de chaves:

- trainer:battle:{trainer_id}
- battle:session:{battle_session_id}
- battle:logs:{battle_session_id}

Regras:

- invalidar cache após cada turno
- invalidar cache após troca de Pokémon
- invalidar cache após encerramento da batalha

---

#### 6.1.4 Consistência e performance
- garantir apenas uma batalha ativa por treinador
- evitar duplicação de sessões
- evitar N+1 queries
- reutilizar estruturas existentes
- manter integridade referencial
- utilizar soft delete
- manter operações idempotentes quando possível
- evitar services gigantes
- manter separação clara entre exploration e battle

---

#### 6.1.5 Contrato da API

A API deve:

- retornar dados normalizados;
- evitar transformação no frontend;
- manter consistência de naming.

Exemplo conceitual:
``` json
{
  "battle_session": {},
  "trainer_pokemon": {},
  "wild_pokemon": {},
  "logs": []
}
```

Exemplo de evento:
``` json
{
  "event_type": "MOVE_USED",
  "payload": {}
}
```
Tipos iniciais de eventos:

- MOVE_USED
- DAMAGE_DEALT
- POKEMON_SWITCHED
- POKEMON_FAINTED
- TRAINER_ESCAPED
- BATTLE_FINISHED

---

### 6.2 WEB
#### 6.2.1 Páginas
tela de batalha
histórico/log da batalha

---

#### 6.2.2 Componentes
- card do Pokémon do treinador
- card do Pokémon selvagem
- seletor de movimentos
- indicador de PP
- indicador de HP
- log da batalha
- botão de fuga
- botão de troca de Pokémon

Estados:

- loading
- erro
- vazio
- sucesso

---

#### 6.2.3 Renderização

A tela de batalha deve exibir:

- Pokémon ativo do treinador
- Pokémon selvagem
- HP atual
- PP dos movimentos
- movimentos disponíveis
- logs da batalha
- status da batalha

---

#### 6.2.4 Interações
utilizar movimento
trocar Pokémon
fugir
atualizar batalha em tempo real/polling
visualizar logs

---

## 7. Restrições Arquiteturais
- Não criar nova arquitetura
- Não duplicar lógica entre API e WEB
- Toda regra de negócio deve estar na API
- Reutilizar padrões existentes
- Minimizar impacto no código atual
- Não reimplementar exploration
- Não duplicar regras de exploration
- Não criar services gigantes
- Toda nova tabela criada no banco de dados deve suportar soft delete
- Não realizar exclusão física de registros, salvo se já existir padrão explícito no projeto
- Usar o padrão existente do projeto para soft delete, preferencialmente com deleted_at
- Consultas devem ignorar registros com soft delete ativo, salvo quando explicitamente necessário

---

## 8. Critérios de Aceite
- [ ] battle-session criada corretamente
- [ ] batalha inicia automaticamente após evento selvagem
- [ ] apenas uma batalha ativa por treinador
- [ ] Pokémon inicial da party funcionando
- [ ] controle de HP funcionando
- [ ] controle de PP funcionando
- [ ] troca de Pokémon funcionando
- [ ] fuga funcionando
- [ ] logs funcionando
- [ ] turnos funcionando
- [ ] payload normalizado
- [ ] integração com exploration funcionando
- [ ] integração com party funcionando
- [ ] sem duplicação de lógica
- [ ] sem regressões
- [ ] cache funcionando
- [ ] services organizados corretamente
- [ ] sem services excessivamente grandes
- [ ] novas tabelas possuem suporte a soft delete
- [ ] consultas padrão ignoram registros deletados logicamente
- [ ] nenhuma exclusão física foi implementada sem justificativa

---

## 9. Plano de Validação
Validação manual
1. Criar treinador
2. Criar party
3. Caminhar em encounter
4. Encontrar Pokémon selvagem
5. Validar criação da battle-session
6. Utilizar movimento
7. Validar consumo de PP
8. Validar dano
9. Trocar Pokémon
10. Fugir da batalha
11. Finalizar batalha
12. Validar logs
13. Validar status final

---

Testes automatizados
- criação da battle-session
- apenas uma batalha ativa
- seleção do Pokémon inicial
- uso de movimentos
- consumo de PP
- troca de Pokémon
- fuga
- encerramento da batalha
- geração de logs
- integração com exploration
- integração com party
- cenários de erro
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
7. Alterações de API  
8. Alterações de UI  
9. Estratégia de testes
10. Estratégia de cache
11. Estratégia de rollback
12. Riscos da implementação
13. Dependências entre domínios
14. Estratégia de separação de responsabilidades
15. Estratégia para evolução futura de captura, XP e level up
16. Riscos e pontos de atenção





