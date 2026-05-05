## Context

O projeto e um monorepo com API FastAPI e web Next.js. A API deve ser a fonte unica de verdade para regras de negocio e dados normalizados, enquanto o frontend deve apenas consumir os contratos expostos.

O arquivo `prompts/pokemon-features.md` define listagem e detalhe de Pokemon com sincronizacao automatica via PokeAPI, cache Redis, enriquecimento de dados e interface web. A proposta fica limitada ao catalogo Pokemon. Nao inclui my-pokemon, pokedex, batalha, captura ou qualquer entidade pertencente ao treinador.

## Goals / Non-Goals

**Goals:**

- Criar um catalogo local de Pokemon com registros minimos importados automaticamente da PokeAPI.
- Expor listagem paginada com filtros por nome, ordem, status e tipo.
- Expor detalhe de Pokemon por identificador.
- Enriquecer Pokemon sob demanda quando o detalhe for acessado.
- Usar cache Redis para reduzir consultas repetidas de lista e detalhe.
- Criar UI web protegida para listagem e detalhe de Pokemon.
- Fragmentar a API por entidade para permitir rotas futuras de tipos, habilidades, movimentos e demais recursos.

**Non-Goals:**

- Criar my-pokemon, pokedex, captura, batalha ou progressao individual.
- Implementar autenticacao nova ou alterar o fluxo de auth existente.
- Criar arquitetura nova fora dos padroes `route.py -> service.py -> repository.py -> models.py`.
- Persistir o payload bruto completo da PokeAPI sem normalizacao.

## Decisions

### Fragmentar domains por entidade

A implementacao da API deve criar um domain por entidade/tabela relevante, por exemplo `pokemon`, `pokemon_type`, `pokemon_ability`, `pokemon_move`, `pokemon_image`, `pokemon_habitat`, `pokemon_shape`, `pokemon_growth_rate`, `pokemon_encounter`, `pokemon_location` e `pokemon_evolution`.

Cada domain deve ter pelo menos `repository.py`, `service.py` e `schema.py`. Domains que tiverem regras especificas devem ter `business.py`. Domains que expuserem endpoints devem ter `route.py`.

Alternativa considerada: concentrar todo o catalogo em `app/domain/pokemon`. Isso reduziria arquivos, mas dificultaria futuras rotas independentes como listagem de tipos e habilidades.

### Orquestrar sincronizacao pelo domain Pokemon

Mesmo com domains fragmentados, a orquestracao de listagem, sincronizacao inicial e enriquecimento deve ficar no service de Pokemon. Esse service pode coordenar outros domains, mas somente chamando seus services publicos.

Racional: o detalhe de um Pokemon e um agregado que toca varias tabelas. Espalhar a orquestracao por subdomains aumentaria acoplamento e duplicacao.

### Manter repositories privados ao proprio domain

Repositories nao devem ser acessados fora do domain ao qual pertencem. Se `PokemonService` precisar criar, buscar ou atualizar `PokemonMove`, ele deve chamar `PokemonMoveService`; nao deve importar nem chamar `PokemonMoveRepository` diretamente.

Esse limite preserva a independencia dos domains fragmentados:

`PokemonService -> PokemonMoveService -> PokemonMoveRepository`

e evita:

`PokemonService -> PokemonMoveRepository`

Alternativa considerada: permitir que o orquestrador Pokemon acesse repositories de subdomains para reduzir chamadas indiretas. Isso foi descartado porque quebra encapsulamento e espalha detalhes de persistencia entre domains.

### Importacao inicial leve pela listagem externa

Na primeira chamada de `GET /pokemon`, se a base local estiver vazia, a API deve chamar apenas `https://pokeapi.co/api/v2/pokemon?offset=0&limit=1350`.

Para cada item retornado, a API deve extrair `order` da URL externa, salvar `name`, montar `external_image` e persistir o status `INCOMPLETE`. Nenhum detalhe pesado deve ser buscado nesta etapa.

Alternativa considerada: enriquecer todos os Pokemon na primeira chamada. Isso foi descartado porque criaria muitas chamadas externas e deixaria a primeira listagem lenta.

### Derivar `external_image` pelo order

O campo `external_image` deve ser calculado durante a importacao inicial usando o formato:

`https://www.pokemon.com/static-assets/content-assets/cms2/img/pokedex/detail/{formatted_order}.png`

O `order` deve vir do identificador numerico no final da URL retornada pela listagem da PokeAPI. O `formatted_order` deve ter sempre 4 digitos com zero a esquerda. Exemplos: `1 -> 0001`, `01 -> 0001`, `25 -> 0025`, `100 -> 0100`.

### Enriquecimento sob demanda no detalhe

Quando `GET /pokemon/{identifier}` encontrar um Pokemon `INCOMPLETE`, a API deve buscar os detalhes externos necessarios, persistir dados enriquecidos e atualizar o status para `COMPLETE`.

O enriquecimento pode usar endpoints de Pokemon, species, encounters, move, type, ability, growth-rate e evolution-chain conforme necessario. A implementacao deve evitar chamadas redundantes usando get-or-create e constraints unicas.

### Tratar sprites no `pokemon_image/business.py`

O domain `pokemon_image` deve conter `business.py` para regras puras de imagem. Esse arquivo deve receber o objeto `sprites` do payload `pokemon/{name}`, percorrer sua estrutura nested, filtrar URLs validas, inferir metadados como `source`, `variant`, `generation`, `game` e `media_type`, escolher a imagem primaria e produzir uma representacao serializavel.

O campo `PokemonImage.images` deve guardar uma string com JSON serializado contendo todas as imagens tratadas. A feature nao deve salvar o objeto bruto de `sprites` sem tratamento.

Implementacao final: `PokemonImage` ficou como um recurso reutilizavel por `order`, sem `pokemon_id` direto. O `Pokemon` referencia o conjunto de imagens escolhido por `images_id`. Alem de `images`, a tabela guarda `front_image`, `back_image`, `front_source` e `back_source` para consumo direto pela API/web.

### Cache Redis

A listagem deve usar chave `pokemon:list:{filters}:{pagination}` e o detalhe deve usar `pokemon:detail:{identifier}`. O TTL deve ser maior que 2 horas. O enriquecimento ou qualquer atualizacao de Pokemon deve invalidar as chaves relacionadas antes de retornar dados atualizados.

Implementacao final: o cache ganhou invalidacao por chave e por padrao em `CacheManager`, alem de serializadores customizados em `CacheService` para respostas com `serialize()`. O catalogo Pokemon usa invalidacao por padrao para listas (`pokemon:list*`) e por chave para detalhes.

### Frontend como consumidor

O web deve criar rotas protegidas `/pokemon` e `/pokemon/[identifier]`. Route handlers BFF devem ler a sessao server-side, chamar a API com token e devolver JSON ao client. A UI deve usar os componentes existentes de Card, Filters, Badge, Pagination, Image, Loading e Alert.

Carregamentos de pagina ou conteudo devem usar o componente Loading pelo hook `useLoading`. Erros de listagem, detalhe ou BFF/API devem ser apresentados com o componente Alert pelo hook `useAlert`, sem criar sistema paralelo de loading ou alertas.

A listagem deve criar um hook especifico da feature, como `usePokemonList`, consumindo o hook compartilhado existente `usePaginatedList`. A pagina de listagem nao deve implementar paginacao/filtros por conta propria.

O detalhe deve criar um hook especifico da feature, como `usePokemonDetail`, para buscar `/api/pokemon/{identifier}`. Esta proposta nao deve criar um hook generico `useDetail`; essa abstracao so deve ser considerada quando houver mais de uma feature real repetindo o mesmo padrao.

As paginas de listagem e detalhe devem ter layout responsivo usando Tailwind CSS v4 e o design system existente. A experiencia visual deve ser elegante, bem acabada e adequada ao tema Pokemon, sem criar uma landing page: a primeira tela deve ser a experiencia utilizavel de catalogo. Os layouts devem funcionar bem em mobile e desktop, sem sobreposicao de textos, cards, filtros, imagens ou acoes.

### Rotas auxiliares expostas pela API

Implementacao final: o router principal `app/domain/pokemon/route.py` inclui rotas auxiliares protegidas para resources sincronizados:

- `/pokemon/ability`
- `/pokemon/move`
- `/pokemon/type`
- `/pokemon/habitat`
- `/pokemon/growth-rate`
- `/pokemon/encounter`

`shape` ficou como subdomain interno sem rota propria nesta change.

## Risks / Trade-offs

- [Primeira listagem depende da PokeAPI] -> Se a PokeAPI falhar e a base estiver vazia, retornar erro controlado e nao criar dados parciais inconsistentes.
- [Enriquecimento pode disparar muitas chamadas externas] -> Usar get-or-create, constraints unicas e chamadas somente quando o registro estiver `INCOMPLETE`.
- [JSON serializado em `images` reduz consultabilidade SQL] -> Aceitar o trade-off porque o objetivo e facilitar consumo/tratamento de URLs no frontend e nas regras do dominio de imagem.
- [Domains fragmentados aumentam quantidade de arquivos] -> Manter orquestracao no PokemonService, mas acessar outros domains apenas via services publicos.
- [Cache pode retornar dados desatualizados] -> Invalidar lista e detalhe apos sincronizacao ou enriquecimento.

## Migration Plan

- Criar novas models e migration Alembic para tabelas Pokemon e entidades relacionadas.
- Criar client externo PokeAPI e schemas de payload externo.
- Implementar domains fragmentados com repositories, services, schemas e regras puras.
- Registrar routers necessarios no `app/main.py`.
- Implementar BFF routes e telas web protegidas.
- Validar com lint, testes unitarios, testes de rota/service/repository e build web.

Rollback: reverter a migration criada pela change, remover routers registrados e remover telas/BFF routes web da feature.

## Open Questions

- Resolvido: a quantidade padrao da listagem externa permanece fixa em `1350` no client/service.
- Resolvido: endpoints auxiliares de tipos, habilidades, movimentos, habitats, growth rates e encounters foram incluidos na implementacao inicial; `shape` ficou preparado como subdomain interno.
