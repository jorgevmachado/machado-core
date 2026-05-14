## Why

O projeto já permite onboarding do treinador, posse de `my-pokemon` e descoberta de `pokedex`, mas ainda não existe um loop principal de exploração para sustentar progressão, navegação por encounters e eventos aleatórios. Essa change introduz a primeira camada jogável desse fluxo, preparando a base para integrações futuras com batalha e desbloqueio de novos encounters.

## What Changes

- Adicionar posse de encounters por treinador com seleção de encounter ativo e garantia de apenas um ativo por vez.
- Inicializar os encounters conhecidos do treinador no onboarding a partir do Pokémon inicial, com ativação automática determinística de um encounter inicial.
- Adicionar fluxo autenticado de exploração com ação de caminhada e retorno de evento normalizado para o frontend.
- Implementar eventos aleatórios locais para exploração, inicialmente com encontro de Pokémon selvagem e achado de pokebolas.
- Adicionar gestão da party principal do treinador com seleção de até 6 `my-pokemon` ativos.
- Adicionar payload de Home do treinador com encounter ativo, dados principais do trainer, últimos 3 Pokémon descobertos e party principal.
- Preparar persistência e contratos para futura integração com batalha e desbloqueio de novos encounters sem ativar esses fluxos agora.

## Capabilities

### New Capabilities
- `trainer-exploration`: exploração autenticada do treinador com encounters conhecidos, encounter ativo, caminhada, eventos aleatórios, party principal e payload de Home.

### Modified Capabilities
- `my-pokemon-management`: requisitos de `my-pokemon` passam a suportar seleção de party principal do treinador e consumo do roster principal em fluxos de exploração.
- `pokedex-management`: requisitos de `pokedex` passam a incluir exposição dos últimos Pokémon descobertos na Home do treinador.

## Impact

- API: novos modelos e relacionamentos para encounters do treinador, party principal e eventos de exploração; novos endpoints autenticados em domínios de trainer/exploration; ajustes no onboarding.
- Web: novas rotas BFF e atualização da Home protegida para consumir summary do trainer, encounter ativo, últimos descobertos e party principal.
- Dados e cache: novas tabelas/associações, invalidação de cache da Home/encounters/party após onboarding, seleção de encounter, caminhada e atualização da party.
