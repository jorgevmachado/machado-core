## Why

O loop atual de exploração, batalha e captura deixa os `my-pokemon` persistidos com HP e PP consumidos, mas ainda não oferece um fluxo canônico de recuperação. Isso quebra a continuidade da party principal, reduz a utilidade da Home como hub do treinador e impede a evolução do domínio para revive, cura de status e outras mecânicas de suporte sem duplicar lógica.

## What Changes

- Criar a capability de Centro Pokémon com healing autenticado da party principal, restauração de HP e PP, revive de Pokémon desmaiados e trilha de auditoria da operação.
- Expor endpoint canônico para executar healing da party e endpoint para consultar histórico item a item de Pokémon restaurados.
- Persistir um resumo por operação em `pokemon_center_healing` e uma trilha detalhada por Pokémon restaurado em `healing_log`.
- Bloquear healing quando houver `battle-session` ativa, retornando erro de regra de negócio normalizado e sem efeitos colaterais.
- Invalidar os caches de Home, party, `my-pokemon` e healing após uma restauração concluída.
- Evoluir o payload da Home para destacar o último evento de cura, além de refletir imediatamente os valores restaurados da party.
- Preparar o design para extensões futuras de status negativos, itens de cura e cooldown sem introduzir schema desnecessário nesta change.

## Capabilities

### New Capabilities
- `pokemon-center-healing`: Healing da party principal no Centro Pokémon, incluindo restauração total de HP e PP, revive de Pokémon desmaiados, histórico detalhado e contratos web/BFF associados.

### Modified Capabilities
- `my-pokemon-management`: Os `my-pokemon` passam a suportar restauração canônica de estado persistido de HP e PP e revive por fluxo autenticado do Centro Pokémon.
- `trainer-exploration`: A Home do treinador passa a expor o último evento de cura e a refletir imediatamente a party restaurada após healing.
- `wild-pokemon-battle-session`: Healing no Centro Pokémon passa a ser incompatível com battle session ativa e o contrato público deve expor esse bloqueio de forma normalizada.

## Impact

- API: mudanças em `machado-api/app/domain/trainer/`, `trainer_party`, `trainer_home`/agregador equivalente, `my_pokemon` e possíveis novos domínios/rotas de `pokemon_center_healing` e `healing_log`.
- Web: nova tela de Centro Pokémon e histórico, integração com BFF, feedback visual de restauração e atualização da Home.
- Banco e persistência: novas estruturas `pokemon_center_healing` e `healing_log`, atualização transacional de `my_pokemon.current_hp` e `my_pokemon_move.current_pp`, com suporte a soft delete seguindo o padrão do projeto.
- Cache e contratos: invalidação explícita de `trainer:home:{trainer_id}`, `trainer:party:{trainer_id}`, `trainer:my-pokemon:{trainer_id}` e `trainer:healing:{trainer_id}`, além de payloads normalizados para healing bem-sucedido, histórico e bloqueio por batalha ativa.
