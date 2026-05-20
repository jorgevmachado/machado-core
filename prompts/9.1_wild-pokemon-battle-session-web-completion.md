# Proposta: wild-pokemon-battle-session-web-completion

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

Já existe uma proposta anterior chamada:

- `9_wild-pokemon-battle-session`

A implementação anterior criou corretamente a estrutura de backend da batalha, porém a implementação do frontend não foi concluída.

Esta proposta NÃO deve:

- recriar backend;
- recriar battle-session;
- recriar endpoints;
- recriar regras de batalha;
- recriar regras de progressão;
- recriar regras de captura;
- alterar contratos existentes sem necessidade;
- duplicar lógica da API no frontend.

Esta proposta deve apenas:

- implementar o frontend da batalha no `machado-web`;
- consumir os endpoints já existentes;
- criar fluxo visual da batalha;
- criar componentes visuais da batalha;
- integrar exploration → battle-session;
- integrar battle-session → capture;
- integrar battle-session → progression;
- reutilizar contratos já implementados.

---

# 1. Objetivo

Implementar a interface web da batalha Pokémon no `machado-web`, permitindo que o treinador visualize e interaja com batalhas selvagens já implementadas na API.

O frontend deve:

- consumir a battle-session já existente;
- exibir Pokémon selvagem;
- exibir Pokémon do treinador;
- exibir HP;
- exibir PP;
- exibir moves disponíveis;
- permitir utilização de movimentos;
- permitir troca de Pokémon;
- permitir fuga;
- exibir logs da batalha;
- atualizar estado da batalha em tempo real;
- preparar integração futura com captura e progressão.

---

# 2. Contexto Atual

- Projeto: machado (monorepo API + WEB)
- Domínio afetado:
  - BattleSession
  - TrainerParty
  - MyPokemon
  - TrainerEncounter
  - TrainerHome
- Arquivos/pastas relevantes:
  - `machado-web/app`
  - `machado-web/app/ui/features`
  - `machado-web/app/services`
  - `machado-web/app/hooks`
- Comportamento atual:
  - backend da battle-session já existe;
  - endpoints da batalha já existem;
  - sistema de exploration já existe;
  - encounter já cria battle-session;
- Problema identificado:
  - frontend da batalha não foi implementado;
  - usuário não consegue visualizar batalha;
  - usuário não consegue utilizar moves;
  - usuário não consegue trocar Pokémon;
  - usuário não consegue fugir da batalha;
  - logs não são exibidos.

---

# 3. Comportamento Esperado

- O treinador deve conseguir visualizar uma batalha ativa.
- O sistema deve exibir o Pokémon selvagem.
- O sistema deve exibir o Pokémon ativo do treinador.
- O sistema deve exibir HP atual.
- O sistema deve exibir PP dos movimentos.
- O sistema deve exibir moves disponíveis.
- O treinador deve conseguir utilizar movimentos.
- O treinador deve conseguir trocar Pokémon.
- O treinador deve conseguir fugir da batalha.
- O sistema deve exibir logs da batalha.
- O sistema deve atualizar o estado da batalha após ações.
- O sistema deve exibir loading states.
- O sistema deve exibir estados de erro.
- O sistema deve preparar estrutura visual futura para:
  - captura;
  - level up;
  - evolução;
  - efeitos especiais;
  - status negativos.

---

# 4. Escopo

## Dentro do escopo

- Tela da battle-session
- Componentes visuais da batalha
- Integração com endpoints existentes
- Exibição de HP
- Exibição de PP
- Lista de movimentos
- Troca de Pokémon
- Fuga da batalha
- Logs da batalha
- Polling/refresh do estado
- Loading states
- Error states
- Empty states

---

## Fora do escopo

- Reimplementação do backend
- Sistema de captura
- Sistema de level up
- Sistema de evolução
- Sistema de status negativos
- Sistema competitivo
- Websocket realtime
- Animações complexas
- Sistema multiplayer

---

# 5. Regras de Negócio

## Regra 1 — Frontend apenas consumidor

**Dado que:** a battle-session já existe  
**Quando:** o frontend consumir os dados  
**Então:** toda regra de negócio deve permanecer na API

---

## Regra 2 — Atualização da batalha

**Dado que:** uma ação foi executada  
**Quando:** o frontend receber resposta da API  
**Então:** a interface deve atualizar o estado da batalha

---

## Regra 3 — Moves indisponíveis

**Dado que:** um move está sem PP  
**Quando:** o frontend renderizar os movimentos  
**Então:** o move deve aparecer desabilitado

---

## Regra 4 — Battle finalizada

**Dado que:** a batalha foi encerrada  
**Quando:** o frontend atualizar o estado  
**Então:** as ações devem ser bloqueadas

---

## Regra 5 — Polling

**Dado que:** a batalha está ativa  
**Quando:** o usuário permanecer na tela  
**Então:** o frontend deve atualizar o estado periodicamente

---

## Regra 6 — Logs da batalha

**Dado que:** a batalha possui eventos  
**Quando:** os logs forem carregados  
**Então:** o frontend deve exibir os eventos ordenados corretamente

---

# 6. Escopo Técnicos

## 6.1 WEB

Toda lógica de negócio deve permanecer na API.  
O frontend deve apenas consumir os contratos existentes.

---

### Funcionalidades

- tela de batalha
- listagem de moves
- troca de Pokémon
- fuga da batalha
- exibição de logs
- atualização do estado da batalha
- integração com encounter
- integração com Home

---

### Fluxos

1. usuário explora área
2. exploration gera encounter
3. encounter retorna battle-session
4. frontend navega para tela da batalha
5. usuário executa ações
6. frontend atualiza estado da batalha
7. batalha é encerrada

---

## 6.1.1 Integrações

Consumir apenas os endpoints já existentes.

Não criar regras duplicadas no frontend.

---

## 6.1.2 Estrutura sugerida

### Features

Criar estrutura similar aos padrões já existentes:

```txt
app/ui/features/battle/