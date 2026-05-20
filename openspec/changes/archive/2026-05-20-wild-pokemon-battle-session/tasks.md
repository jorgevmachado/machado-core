## 1. Modelagem e persistência

- [x] 1.1 Criar models ORM para `battle_session`, `battle_turn` e `battle_log`
- [x] 1.2 Definir enums de status e tipos de ação da batalha
- [x] 1.3 Criar migration para as novas tabelas e índices principais
- [x] 1.4 Registrar relacionamentos mínimos entre battle session, trainer e entidades de batalha
- [x] 1.5 Definir no model da sessão o snapshot corrente de HP e PP temporários
- [x] 1.6 Adicionar testes de model/repository para persistência básica da sessão

## 2. Estrutura do domínio de batalha

- [x] 2.1 Criar diretório do domínio `wild_pokemon_battle_session` na API com `__init__.py`, `schema.py`, `business.py`, `repository.py`, `service.py` e `route.py`
- [x] 2.2 Definir schemas normalizados para sessão ativa, lados da batalha, ações, turnos e logs
- [x] 2.3 Implementar repository com leitura da sessão ativa, criação da sessão, gravação de turnos e gravação de logs
- [x] 2.4 Implementar regra simples e explícita de dano para a V1
- [x] 2.5 Implementar regras puras para seleção do Pokémon inicial, consumo de PP, troca de Pokémon e transição de status
- [x] 2.6 Implementar escolha simples do movimento automático do Pokémon selvagem
- [x] 2.7 Implementar `WildPokemonBattleSessionService` com criação/retomada da sessão ativa do treinador

## 3. Ações da batalha

- [x] 3.1 Implementar endpoint/service para buscar batalha ativa do treinador
- [x] 3.2 Implementar endpoint/service para usar movimento durante a batalha
- [x] 3.3 Implementar endpoint/service para trocar o Pokémon ativo do treinador
- [x] 3.4 Implementar endpoint/service para fugir da batalha
- [x] 3.5 Implementar endpoint/service para listar logs da batalha em ordem cronológica
- [x] 3.6 Garantir que a ação do treinador processe também a resposta automática do selvagem no mesmo ciclo
- [x] 3.7 Cobrir com testes os cenários de PP insuficiente, troca inválida, fuga, resposta do selvagem e encerramento da sessão

## 4. Integração com exploration e domínios existentes

- [x] 4.1 Expor interface pública mínima para obter a party ativa do treinador para a batalha
- [x] 4.2 Expor interface pública mínima para obter os dados do Pokémon do treinador usados pela sessão
- [x] 4.3 Integrar `TrainerExplorationService.walk()` para criar ou retomar a battle session quando o evento for `WILD_POKEMON`
- [x] 4.4 Atualizar o payload normalizado do evento de exploração para incluir `battle_session_id`, `battle_status` e `has_active_battle`
- [x] 4.5 Garantir que múltiplas batalhas ativas simultâneas sejam impedidas por trainer
- [x] 4.6 Bloquear novas ações de `walk()` enquanto existir batalha ativa
- [x] 4.7 Ajustar invalidação de cache e orquestração entre exploration, home e battle session

## 5. BFF e camada web

- [x] 5.1 Criar rotas BFF em `machado-web/app/api` para batalha ativa, ação de movimento, troca, fuga e logs
- [x] 5.2 Criar/atualizar service client de trainer battle no frontend apontando para os novos endpoints backend
- [x] 5.3 Definir tipos TypeScript para sessão de batalha, logs e ações
- [x] 5.4 Criar hooks ou serviços de UI para carregar sessão ativa e executar ações da batalha
- [x] 5.5 Ajustar o fluxo de exploration/home para reconhecer a referência mínima de battle session no evento selvagem
- [x] 5.6 Adicionar testes unitários do BFF, service e hooks do frontend

## 6. Validação e documentação

- [x] 6.1 Adicionar testes de integração do backend para início automático da batalha a partir do `walk()`
- [x] 6.2 Validar os cenários de sessão única ativa, bloqueio de exploration, troca, fuga e encerramento por derrota
- [x] 6.3 Rodar `make lint` e `make test` em `machado-api`
- [x] 6.4 Rodar `yarn lint` e `yarn test` em `machado-web`
- [x] 6.5 Atualizar documentação operacional relevante (`AGENTS.md`, prompts ou docs de arquitetura) com o novo domínio de batalha
