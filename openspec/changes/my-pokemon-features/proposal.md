## Why

O sistema já possui catálogo base de Pokémon e estrutura de treinador, mas ainda não representa Pokémon capturados e personalizados por treinador. Isso bloqueia fluxos futuros como progressão individual, batalha, evolução e gestão de elenco próprio, então a mudança precisa introduzir esse domínio agora com contratos normalizados e cache adequado.

## What Changes

- Introduzir a entidade `my-pokemon` vinculada a `trainer` e `pokemon`, com nickname opcional no onboarding, fallback automático para o `name` do Pokémon quando vazio, `captured_at`, atributos individuais e soft delete.
- Criar fluxo de criação de `my-pokemon` na API com geração de atributos, seleção de até 4 movimentos distintos do Pokémon base, persistência de `pp` e `max_pp` copiando o valor base de cada move e geração de `name` público único por treinador a partir do nickname efetivo.
- Expor endpoints autenticados para criar, listar paginadamente e detalhar `my-pokemon` usando `name` como identificador de rota.
- Adicionar cache Redis para listagem e detalhe de `my-pokemon`, com invalidação coerente após criação ou atualização, incluindo suporte a limpeza explícita de cache de leitura.
- Implementar no `machado-web` o onboarding de criação inicial a partir da `home`, exibido apenas quando o usuário ainda não possui `trainer`: qualquer usuário pode informar nickname opcional; usuários comuns escolhem entre `Bulbasaur`, `Charmander` e `Squirtle`, enquanto usuários `admin` podem escolher livremente um Pokémon da lista existente com autocomplete e preview em card antes de salvar; o fluxo deve usar `POST /trainer/onboarding` e retornar o `trainer` já com o primeiro `my-pokemon`.
- Implementar no `machado-web` as páginas protegidas de listagem e detalhe de `my-pokemon` consumindo a API via BFF existente.
- Exibir no frontend nickname, imagem do Pokémon base, level, experiência, HP/max HP, atributos, movimentos, PP, metadados básicos dos moves, resumo do treinador e data de captura, com estados de loading/erro/vazio/sucesso seguindo os padrões já existentes.

## Capabilities

### New Capabilities
- `my-pokemon-management`: gerenciamento completo de Pokémon do treinador, incluindo criação, listagem, detalhe, contratos normalizados, cache e experiência web protegida.

### Modified Capabilities
<!-- None. Existing specs do not currently define trainer-owned Pokémon behavior. -->

## Impact

- `machado-api`: novos models, associações, schemas, regras de negócio, repositories, services, routes, cache keys e migrações para `my-pokemon` e movimentos do `my-pokemon`, além de suporte ao fluxo inicial em `trainer` que cria `trainer` e primeiro `my-pokemon` no onboarding.
- `machado-web`: novos BFF route handlers, services, hooks e telas/fluxos em `app/(protected)/my-pokemon/` e `app/(protected)/home/`, incluindo o onboarding condicionado por `user.trainer` e o service de `trainer` para a criação inicial.
- Banco de dados PostgreSQL: novas tabelas/relacionamentos com suporte a soft delete e constraints para evitar duplicidade de movimentos.
- Redis: novas entradas de cache para listagem e detalhe do domínio `my-pokemon`.
