## Context

O backend já concentra a orquestração transversal do treinador em `app/domain/trainer/service.py`, com `TrainerPartyService` cuidando da party ativa, `BattleSessionService` da battle session e `TrainerService.get_home()` do payload agregado da Home. O estado persistido relevante para healing já existe em `my_pokemon.current_hp` e `my_pokemon_move.current_pp`, e a battle session já bloqueia outros fluxos do treinador quando está ativa.

O Centro Pokémon introduz um caso de uso cross-domain: ler a party principal do treinador, validar ausência de batalha ativa, restaurar HP e PP, reviver Pokémon com `current_hp = 0`, registrar uma operação resumida e logs detalhados, invalidar caches e enriquecer a Home com o último evento de cura. O design precisa respeitar os boundaries do projeto: regras de negócio na API, repositories donos apenas dos próprios agregados, soft delete nas novas tabelas e frontend consumindo payloads já normalizados pelo backend/BFF.

## Goals / Non-Goals

**Goals:**
- Introduzir um fluxo canônico autenticado de healing da party principal no Centro Pokémon.
- Restaurar completamente HP e PP dos Pokémon da party, incluindo revive de Pokémon desmaiados.
- Persistir um registro resumido por operação em `pokemon_center_healing` e uma trilha item a item em `healing_log`.
- Bloquear o healing quando houver battle session ativa, com erro de regra de negócio normalizado e sem efeitos colaterais.
- Atualizar a Home para refletir a party restaurada e destacar explicitamente o último evento de cura.
- Reaproveitar os serviços existentes de `trainer`, `trainer_party`, `my_pokemon` e cache sem criar duplicação de regras.

**Non-Goals:**
- Implementar cura parcial, cooldown, custo monetário, NPCs, enfermaria ou healing limitado.
- Implementar cura de status negativos, itens de cura ou revive consumível; apenas deixar pontos de extensão no design.
- Reescrever o engine de batalha, o agregador da Home ou o fluxo de captura além do necessário para o bloqueio de healing durante batalha ativa.
- Introduzir consumo de PokéAPI, novas dependências externas ou uma arquitetura paralela de healing no frontend.

## Decisions

### 1. Criar um subdomínio próprio de Centro Pokémon sob `trainer`, com orquestração fina no `TrainerService`

O contrato público deve viver sob o namespace autenticado do treinador, com rotas como `POST /trainer/pokemon-center/heal` e `GET /trainer/pokemon-center/healing-history`. A implementação terá um subdomínio `app/domain/trainer/pokemon_center/` para schemas, repository e service do agregado `pokemon_center_healing`, enquanto `TrainerService` continuará como ponto de composição com `TrainerPartyService`, `BattleSessionService` e o serviço de logs detalhados.

Isso mantém o padrão atual do projeto: `trainer/route.py` agrega subrouters e o service raiz do treinador coordena fluxos transversais.

Alternativas consideradas:
- Colocar toda a lógica em `TrainerPartyService`: rejeitado porque mistura gestão de composição da party com healing, histórico e Home.
- Colocar tudo em `TrainerService` sem subdomínio: rejeitado porque centralizaria persistência e contratos demais em um service já transversal.

### 2. Tratar o healing como operação atômica sobre a party ativa inteira

O Centro Pokémon sempre atua sobre a party principal completa. O fluxo será:

1. validar que o treinador possui party ativa;
2. validar que não existe `battle-session` ativa;
3. carregar a party com `my_pokemon` e moves necessários;
4. calcular diffs de restauração por Pokémon e por move;
5. aplicar revive quando `current_hp = 0`, restaurando HP para o máximo;
6. restaurar `current_hp` para `max_hp` e `current_pp` para `max_pp`;
7. persistir `pokemon_center_healing` com totais agregados;
8. persistir `healing_log` item a item;
9. invalidar caches e retornar payload normalizado.

Essa semântica elimina healing parcial inconsistente e simplifica a Home, a party e a trilha de auditoria.

Alternativas consideradas:
- Healing parcial por Pokémon: rejeitado porque contraria o prompt e aumenta risco de inconsistência entre Home, party e histórico.
- Revive em fluxo separado: rejeitado por decisão funcional explícita do usuário e por quebrar a expectativa do Centro Pokémon como restauração total.

### 3. Modelar `pokemon_center_healing` como resumo da operação e `healing_log` como detalhe por Pokémon

Serão persistidos dois níveis:

- `pokemon_center_healing`: `trainer_id`, `healed_pokemon_quantity`, `restored_hp`, `restored_pp`, timestamps e `deleted_at`.
- `healing_log`: `trainer_id`, `my_pokemon_id`, `pokemon_center_healing_id`, `action_type`, `payload`, timestamps e `deleted_at`.

O histórico principal exposto ao frontend será item a item, então o endpoint de listagem pode ser orientado a `healing_log`, mas cada linha continuará ligada ao evento resumido para auditoria, agrupamento futuro e destaque da Home.

Alternativas consideradas:
- Persistir apenas `healing_log`: rejeitado porque dificultaria destacar um “último evento de cura” coerente na Home.
- Persistir apenas `pokemon_center_healing`: rejeitado porque perderia o histórico item a item exigido pelo usuário.

### 4. Reusar `TrainerPartyService` para leitura da party e criar regras puras de healing em `business.py`

Os repositories devem respeitar ownership por domain. Por isso, a leitura da party continuará saindo de `TrainerPartyService`/`TrainerPartyRepository`, enquanto a mutação de `my_pokemon` e `my_pokemon_move` deve acontecer por services/repositories owners desses agregados. As regras puras de cálculo ficarão em `app/domain/trainer/pokemon_center/business.py`, incluindo:

- identificar Pokémon que serão revividos;
- calcular `restored_hp`;
- calcular `restored_pp`;
- gerar payloads normalizados de log;
- decidir se o healing alterou algo ou foi no-op.

Alternativas consideradas:
- Repository do Centro Pokémon fazer join e update direto em `MyPokemon`/`MyPokemonMove`: rejeitado por violar o boundary do AGENTS da API.
- Colocar cálculo no frontend: rejeitado porque toda regra de negócio deve ficar na API.

### 5. Bloquear healing com batalha ativa usando erro de regra de negócio enxuto

Quando houver battle session ativa, o endpoint deve falhar antes de qualquer mutação, retornando erro normalizado com um código/razão explícita como `ACTIVE_BATTLE_BLOCKS_HEALING`. O payload pode incluir um booleano ou resumo mínimo (`has_active_battle: true`), mas não deve acoplar o Centro Pokémon ao contrato completo da batalha.

Isso mantém o boundary limpo e permite ao frontend apenas redirecionar/reabrir a batalha usando a Home ou o fluxo de battle existente.

Alternativas consideradas:
- Retornar a battle session completa no erro: rejeitado por acoplamento excessivo entre healing e batalha.
- Permitir healing fora da batalha mesmo com sessão ativa: rejeitado por inconsistência do loop principal.

### 6. Evoluir explicitamente o payload da Home com um resumo do último evento de cura

O contrato de Home deve ganhar um bloco explícito, por exemplo `last_healing`, contendo no mínimo:

- `healing_id`
- `healed_pokemon_quantity`
- `restored_hp`
- `restored_pp`
- `created_at`

Além disso, a party retornada pela Home deve refletir os `current_hp` e `current_pp` restaurados logo após a invalidação de cache. O dado da Home não deve obrigar o frontend a reconstruir o destaque da cura buscando o histórico separadamente.

Alternativas consideradas:
- Só invalidar cache sem mudar o payload: rejeitado por decisão explícita do usuário.
- Enviar lista completa de logs dentro da Home: rejeitado por inflar o agregador sem necessidade.

### 7. Expor o histórico principal como listagem item a item orientada a `healing_log`

O histórico do Centro Pokémon será consumido principalmente por item de Pokémon curado. O endpoint deve devolver entradas cronológicas com:

- referência do Pokémon curado;
- deltas restaurados de HP e PP;
- indicação se houve revive;
- referência ao evento resumido (`pokemon_center_healing_id`);
- timestamp.

Isso atende a necessidade funcional sem impedir agrupamentos futuros por operação na UI.

Alternativas consideradas:
- Listagem agrupada por operação com detalhe lazy: rejeitado porque a exigência principal já foi definida como item a item.

### 8. Invalidar caches por domínio concluindo a transação antes da invalidação

O fluxo de healing deve usar transação única para mutações e logs. Apenas após commit bem-sucedido devem ser invalidadas as chaves:

- `trainer:home:{trainer_id}`
- `trainer:party:{trainer_id}`
- `trainer:my-pokemon:{trainer_id}`
- `trainer:healing:{trainer_id}`

O serviço de healing deve reaproveitar `CacheManager`/`CacheService` existentes e evitar invalidar por padrão mais amplo do que o necessário.

Alternativas consideradas:
- Invalidar antes do commit: rejeitado por risco de cache refletir operação que falhou.
- Recalcular Home síncrona e persistir snapshot dedicado: rejeitado por duplicar o padrão atual de cache sob demanda.

### 9. Preparar extensibilidade futura sem ampliar o schema agora além do necessário para revive

O prompt pede estrutura futura para status negativos e cura avançada. Nesta change, a preparação será feita por:

- enums/tipos de `action_type` em `healing_log` que aceitem categorias futuras;
- payload estruturado em `healing_log.payload` para acomodar status curados no futuro;
- design de service separado (`business.py` + `service.py`) para crescer sem virar service gigante.

Não serão adicionadas colunas novas só para “reservar lugar” de mecânicas futuras que ainda não têm regra fechada.

Alternativas consideradas:
- Adicionar campos nulos para status/revive avançado agora: rejeitado por schema prematuro.
- Ignorar totalmente extensibilidade: rejeitado porque aumentaria retrabalho na próxima change de suporte/itens.

## Risks / Trade-offs

- [Healing tocar vários domínios e crescer demais em `TrainerService`] → Mitigar com subdomínio próprio de Centro Pokémon e regras puras em `business.py`.
- [Repositories cruzarem boundaries de forma indevida] → Mitigar delegando leitura da party e mutações de `my_pokemon`/moves aos services owners.
- [Home ficar pesada com novo bloco de healing] → Mitigar incluindo apenas o último evento resumido, não o histórico completo.
- [Revive alterar expectativas do loop de derrota em batalha] → Mitigar limitando revive ao contexto do Centro Pokémon, nunca dentro da battle session ativa.
- [Histórico item a item gerar volume maior de dados] → Mitigar usando paginação e mantendo `pokemon_center_healing` como resumo auditável por operação.
- [Healing no-op causar ruído de logs] → Mitigar definindo explicitamente se operações sem restauração geram registro resumido; a recomendação é registrar apenas quando houver ao menos um delta real.
- [Caches inconsistentes após erro parcial] → Mitigar com commit único antes da invalidação e rollback completo em qualquer falha.

## Migration Plan

1. Adicionar models, enums e migration para `pokemon_center_healing` e `healing_log`, ambos com `deleted_at`.
2. Implementar subdomínio `trainer/pokemon_center` com schemas, repository, business, service e rotas autenticadas.
3. Ajustar `my_pokemon`/moves owners para suportar atualização em lote do estado persistido usado pelo healing.
4. Integrar o fluxo no `TrainerService` e na Home, incluindo `last_healing` no contrato agregado.
5. Adicionar BFF/serviços web, página protegida do Centro Pokémon e histórico item a item.
6. Cobrir API e web com testes de sucesso, revive, bloqueio por batalha ativa, histórico e invalidação de cache.
7. Fazer rollout sem quebrar contratos existentes; o novo bloco da Home deve ser aditivo e opcional para clientes antigos.

## Open Questions

- Se a party ativa existir mas todos os Pokémon já estiverem com HP e PP máximos, a implementação deve retornar sucesso idempotente sem criar logs, ou registrar uma operação sem deltas? A recomendação deste design é tratar como sucesso idempotente sem persistir evento.
- Ainda precisa ser decidido se o histórico item a item será paginado pelo endpoint backend desde o primeiro release ou se o volume inicial é suficientemente pequeno para paginação simples padrão do projeto; a recomendação é já seguir o padrão paginado existente.
