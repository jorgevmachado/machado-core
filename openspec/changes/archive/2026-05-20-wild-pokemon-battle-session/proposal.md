## Why

O fluxo de exploração já consegue encontrar Pokémon selvagens, mas ainda não existe uma sessão de batalha stateful entre o evento do encounter e os sistemas futuros de captura e progressão. Isso cria um vazio arquitetural: o frontend não recebe um estado normalizado de batalha e a API não possui um boundary próprio para turnos, HP temporário, PP e troca de Pokémon.

## What Changes

- Criar um domínio de sessão de batalha contra Pokémon selvagem com estado persistido, turnos, logs e status de encerramento.
- Iniciar automaticamente uma batalha quando o evento de exploração gerar encontro com Pokémon selvagem.
- Expor endpoints para consultar a batalha ativa, usar movimento, trocar Pokémon, fugir e listar logs.
- Restringir a V1 a uma batalha determinística simples: turno do treinador seguido de resposta automática do selvagem, sem efeitos secundários complexos.
- Reutilizar party, my-pokemon, pokedex e exploration existentes sem recriar suas regras centrais.
- Preparar a sessão para extensões futuras de captura, experiência e progressão sem implementar esses fluxos agora.

## Capabilities

### New Capabilities
- `wild-pokemon-battle-session`: Sessão stateful de batalha PvE contra Pokémon selvagem, incluindo estado, turnos, ações do treinador, HP temporário, PP e logs.

### Modified Capabilities
- `trainer-exploration`: O evento de encontro com Pokémon selvagem passa a iniciar automaticamente uma sessão de batalha ativa e a devolver referência normalizada para essa sessão.

## Impact

- API: novo domínio de batalha no backend, novas entidades/modelos, rotas e serviços orquestrando integração com `trainer`, `trainer_party`, `my_pokemon` e `trainer_exploration`.
- Web: novos contratos e fluxos de UI para carregar batalha ativa, executar ações e refletir estado/log da batalha.
- Banco: novas tabelas para sessão, turnos e logs de batalha.
- Fluxos existentes: `walk()` e eventos de exploração passam a se integrar com a criação/consulta de batalha ativa.
- Contratos: o evento selvagem de exploration passa a devolver uma referência normalizada mínima da battle session.
