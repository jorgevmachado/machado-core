## Why

O projeto ainda nao possui uma experiencia local e normalizada para consultar Pokemon. A aplicacao depende de uma fonte externa para dados de catalogo, mas precisa persistir um catalogo proprio para listagem, detalhe, cache e futuras features relacionadas.

## What Changes

- Adicionar catalogo de Pokemon na API com listagem paginada, filtros e detalhe.
- Sincronizar automaticamente a lista basica da PokeAPI na primeira chamada da listagem quando a base local estiver vazia.
- Persistir Pokemon importados inicialmente como `INCOMPLETE`, com `name`, `order` e `external_image` derivados da listagem externa.
- Enriquecer Pokemon sob demanda quando o detalhe for acessado e o registro estiver `INCOMPLETE`.
- Persistir dados enriquecidos de stats, species, tipos, habilidades, movimentos, imagens, encontros, crescimento, habitat, shape e evolucao quando disponiveis.
- Adicionar cache Redis para respostas de listagem e detalhe, com invalidacao apos atualizacao/enriquecimento.
- Adicionar interface web protegida para listagem e detalhe de Pokemon consumindo apenas contratos normalizados da API.
- Manter fora do escopo my-pokemon, pokedex, captura, batalha, trainer-owned Pokemon e evolucao de Pokemon do treinador.

## Capabilities

### New Capabilities

- `pokemon-catalog`: Catalogo local de Pokemon com sincronizacao inicial, enriquecimento sob demanda, cache, endpoints de listagem/detalhe e UI de consumo.

### Modified Capabilities

- None.

## Impact

- API: novos models, migrations, domains fragmentados por entidade Pokemon, client externo PokeAPI, endpoints protegidos e testes.
- Web: novas rotas protegidas `/pokemon` e `/pokemon/[identifier]`, BFF route handlers, service, tipos, hooks e componentes de UI.
- Infraestrutura: uso de Redis existente para cache e configuracao existente para variaveis de ambiente da PokeAPI.
- Banco de dados: novas tabelas com suporte a soft delete onde aplicavel e constraints para evitar duplicacao.
