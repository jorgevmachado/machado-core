## Context

O backend já possui exploração stateful por treinador, party principal, MyPokemon e Home agregadora, mas ainda trata o encontro com Pokémon selvagem apenas como um evento de exploração. A nova mudança precisa introduzir uma sessão de batalha persistida, com integração clara entre `trainer_exploration`, `trainer_party`, `my_pokemon` e uma futura captura, sem recriar regras que já existem nesses domínios.

O frontend hoje consegue consumir Home, party e eventos de exploração. Para batalha, ele precisará de um contrato dedicado e normalizado para ler a sessão ativa, executar ações e renderizar logs/turnos.

## Goals / Non-Goals

**Goals:**
- Criar um domínio backend próprio para sessão de batalha PvE contra Pokémon selvagem.
- Persistir estado de batalha, turnos, logs, HP temporário, PP temporário, Pokémon ativo e status final.
- Integrar `walk()` com criação automática da sessão quando houver evento de Pokémon selvagem.
- Fechar uma V1 de combate simples, determinística e pequena o suficiente para implementação incremental.
- Expor endpoints mínimos para consultar batalha ativa, usar movimento, trocar Pokémon, fugir e listar logs.
- Preparar o modelo para captura, progressão e XP futuras sem implementar esses fluxos agora.

**Non-Goals:**
- Implementar captura, uso de pokeballs, XP, level up ou evolução.
- Criar IA avançada, weather, status negativos, itens, PvP ou batalhas entre treinadores.
- Implementar prioridade complexa, accuracy avançada, efeitos secundários de moves ou múltiplas ações simultâneas por turno.
- Reescrever os domínios de party, home, my-pokemon ou exploration.

## Decisions

### 1. Criar um novo domínio `wild_pokemon_battle_session` na API

A sessão de batalha é um boundary próprio e não deve ficar acoplada a `trainer_exploration`. O domínio novo terá `repository.py`, `service.py`, `schema.py`, `business.py` e `route.py`, seguindo o padrão do projeto.

Alternativas consideradas:
- Embutir a batalha em `trainer_exploration`: rejeitado por misturar evento randômico com estado de combate persistido.
- Criar tudo em `trainer`: rejeitado porque `trainer` deve continuar como agregador/orquestrador, não engine de batalha.

### 2. Separar persistência em sessão, turnos e logs

O design assume pelo menos três entidades:
- `battle_session`: estado corrente, treinador, encounter/evento de origem, status, Pokémon ativos, snapshot mínimo do HP.
- `battle_turn`: ações efetivamente processadas por turno.
- `battle_log`: eventos normalizados consumíveis pelo frontend.

Essa separação reduz acoplamento entre estado corrente e histórico, além de facilitar futuras features de replay, captura e XP.

Alternativa considerada:
- Manter tudo em um JSON único na sessão: rejeitado por dificultar queries, auditoria e evolução incremental.

### 3. Reutilizar party e my-pokemon por interfaces públicas, não por queries cruzadas em repository

O Pokémon inicial do treinador virá da party ativa via service público do domínio de party. Dados necessários do `my-pokemon` também devem ser carregados por boundary apropriado do domínio, evitando queries cruzadas fora do aggregate owner.

Alternativa considerada:
- Reconsultar diretamente tabelas de party e my-pokemon dentro do repository da batalha: rejeitado por violar os boundaries documentados no `AGENTS.md`.

### 4. Integrar `walk()` com criação automática da batalha, mas manter a exploração responsável apenas pelo gatilho

`TrainerExplorationService.walk()` continuará responsável por gerar o evento de exploração. Quando o resultado for `WILD_POKEMON`, ele deve delegar explicitamente à battle session a criação/retorno da sessão ativa. A lógica de turnos, HP, PP, troca e fuga fica somente no domínio de batalha.

Alternativas consideradas:
- Fazer o frontend iniciar a batalha em uma chamada separada após o evento: rejeitado porque abre janela de inconsistência entre evento e sessão stateful.
- Criar a sessão fora do fluxo de `walk()`: rejeitado porque quebra a regra de início automático.

### 5. Persistir HP e PP temporários como estado de batalha, sem alterar o agregado permanente de `my-pokemon`

O dano e o consumo de PP da batalha devem existir como estado temporário da sessão. O agregado real de `my-pokemon` não deve ser mutado como se fosse estado permanente de progressão.

Alternativa considerada:
- Atualizar diretamente HP/PP do `my-pokemon`: rejeitado porque mistura estado transiente de batalha com estado persistente de progressão do treinador.

### 6. Definir uma V1 de dano e turno simples

A primeira versão deve usar uma regra de dano simples e explícita, suficiente para validar o loop de batalha sem introduzir engine complexa. O fluxo de turno será:
- ação do treinador
- resolução do dano/efeito simples
- verificação de encerramento
- resposta automática do selvagem com um movimento válido
- nova verificação de encerramento

O Pokémon selvagem responderá automaticamente no mesmo ciclo da ação do treinador, usando uma escolha determinística simples entre movimentos com PP disponível. Não haverá prioridade, efeitos secundários complexos, accuracy avançada ou status negativos na V1.

Alternativas consideradas:
- Deixar fórmula e resposta do selvagem em aberto para a implementação: rejeitado porque gera retrabalho e ambiguidade desnecessária.
- Fazer o selvagem agir em chamada separada: rejeitado porque complica o contrato do frontend e o fluxo stateful.

### 7. Bloquear exploration enquanto houver batalha ativa

Enquanto existir uma wild battle session ativa para o treinador, novas ações de exploration que possam abrir outro encontro devem ser rejeitadas ou redirecionadas para a sessão corrente. Isso reforça a regra de sessão única ativa e evita duplicidade de estado.

Alternativas consideradas:
- Permitir novos `walk()` durante a batalha: rejeitado porque cria conflito de ownership entre encounter ativo e sessão de combate.

### 8. Expor um contrato normalizado e mínimo para o frontend

O frontend precisa receber um payload consistente com:
- dados da sessão
- status
- lado do treinador e lado selvagem
- Pokémon ativo de cada lado
- movimentos disponíveis e PP atual
- logs e turno atual

Isso evita que a UI tenha de reconstruir estado a partir de múltiplas respostas heterogêneas.

No caso específico do `walk()` com evento selvagem, a referência normalizada mínima deve incluir pelo menos:
- `battle_session_id`
- `battle_status`
- `has_active_battle`

### 9. Snapshot temporário de HP e PP fica centralizado na sessão

O snapshot corrente de HP e PP deve ficar centralizado na sessão ativa, enquanto turnos e logs registram o histórico das transições. Isso simplifica leitura do estado atual e reduz custo de recomposição do frontend.

Alternativas consideradas:
- Distribuir todo o estado corrente apenas nos turnos/logs: rejeitado porque aumenta complexidade de leitura e reconstrução.

## Risks / Trade-offs

- [Sessão de batalha virar god object] → Mitigar separando business rules puras, estado corrente e histórico em camadas e entidades distintas.
- [Acoplamento indevido com exploration e party] → Mitigar usando interfaces públicas mínimas entre services e proibindo queries cross-domain em repository.
- [Migração inicial de schema ficar grande] → Mitigar dividindo entidades em tabelas pequenas e explícitas, com enums simples de status.
- [Estado temporário de HP/PP divergir do frontend] → Mitigar com payload normalizado da sessão e logs por turno.
- [V1 crescer para uma engine de batalha completa cedo demais] → Mitigar limitando explicitamente fórmula, resposta do selvagem e mecânicas avançadas fora de escopo.
- [Bloqueio futuro para captura/XP] → Mitigar persistindo origem do encounter, estado final e logs suficientes para extensões posteriores.
