## Why

O sistema já possui catálogo base de Pokémon, treinador e elenco próprio em `my-pokemon`, mas ainda não representa a coleção de Pokédex de cada treinador. Isso bloqueia uma visão de progresso de descoberta por treinador e impede evoluções futuras como percentuais de coleção, recompensas e jornadas de descoberta.

## What Changes

- Introduzir a entidade `pokedex` vinculada a `trainer` e `pokemon`, com atributos individualizados, nickname opcional, `discovered`, `discovered_at` e soft delete.
- Criar na API o fluxo de inicialização completa da Pokédex do treinador durante o onboarding, gerando uma entrada de `pokedex` para cada Pokémon base já persistido.
- Marcar como descoberto, no onboarding, apenas o Pokémon escolhido pelo usuário, registrando `discovered=true` e `discovered_at` para essa entrada específica.
- Expor endpoints autenticados para listar paginadamente, detalhar e marcar descoberta de `pokedex` usando `pokemon.name` como identificador externo do recurso.
- Adicionar cache Redis para listagem e detalhe de `pokedex`, com invalidação coerente após descoberta ou outras atualizações relevantes.
- Incluir a lista de `pokedex` no contrato retornado pelo onboarding de `trainer`, junto com a lista já existente de `my-pokemons`.
- Implementar no `machado-web` as páginas protegidas de listagem e detalhe de `pokedex` consumindo a API via BFF existente.
- Exibir no frontend nickname, imagem do Pokémon base, atributos, level, experiência, HP/max HP, status de descoberta e data de descoberta com estados de loading/erro/vazio/sucesso seguindo os padrões já existentes.
- Preparar o domínio para futuras extensões de descoberta, sem ativar nesta implementação sincronizações adicionais além do onboarding.

## Capabilities

### New Capabilities
- `pokedex-management`: gerenciamento da Pokédex do treinador, incluindo inicialização no onboarding, descoberta explícita, listagem, detalhe, cache e experiência web protegida.

### Modified Capabilities
- `my-pokemon-management`: o onboarding do treinador passa a retornar também a coleção `pokedex`, preservando compatibilidade contratual do fluxo inicial do treinador.

## Impact

- `machado-api`: novos models, associações, schemas, regras de negócio, repositories, services, routes, cache keys e migrações para `pokedex`, além da ampliação do onboarding de `trainer`.
- `machado-web`: novos BFF route handlers, services, hooks e telas/fluxos em `app/(protected)/pokedex/` e `app/api/pokedex/`.
- Banco de dados PostgreSQL: nova tabela e constraints para garantir uma única entrada de `pokedex` por `trainer` + `pokemon`.
- Redis: novas entradas de cache para listagem e detalhe do domínio `pokedex`.
