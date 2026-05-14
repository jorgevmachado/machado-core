## Context

O monorepo já possui um domínio de catálogo base (`pokemon`), um domínio de treinador (`trainer`) e um domínio de posse individual (`my-pokemon`). A nova feature precisa introduzir uma camada intermediária de coleção por treinador, onde cada Pokémon base tenha uma entrada persistida na Pokédex daquele treinador, independente de fluxos de posse e sem copiar responsabilidades de captura para este domínio.

O fluxo já existente de onboarding do treinador é o melhor ponto para popular a coleção inicial, porque ele já cria `trainer` e o primeiro `my-pokemon` em uma única transação de negócio. A implementação precisa respeitar os padrões existentes: API como fonte única de verdade, arquitetura em 5 camadas, cache Redis para leitura, BFF autenticado no frontend e hooks específicos por feature.

## Goals / Non-Goals

**Goals:**
- Introduzir um domínio `pokedex` persistente, ligado a um único treinador e a um único Pokémon base.
- Gerar atributos individuais para cada entrada de `pokedex` usando a mesma abordagem base de `my-pokemon`.
- Persistir estado de descoberta com `discovered` e `discovered_at`.
- Popular toda a Pokédex do treinador durante o onboarding, marcando como descoberto apenas o Pokémon inicial escolhido.
- Expor contratos autenticados para descobrir, listar e detalhar `pokedex`, com cache Redis e invalidação consistente.
- Retornar `pokedex` no payload de onboarding de `trainer`.
- Entregar experiência web protegida para listagem e detalhe de `pokedex`.

**Non-Goals:**
- Implementar batalha, evolução, level up em combate ou troca de movimentos.
- Implementar tela específica para descoberta manual no frontend nesta change.
- Ativar nesta implementação sincronizações automáticas adicionais fora do onboarding.
- Consumir a PokéAPI diretamente para criação de `pokedex`.

## Decisions

### 1. Modelar `pokedex` como agregado próprio por treinador e Pokémon base
Cada treinador precisa ter exatamente uma entrada de Pokédex para cada Pokémon base. A modelagem mais simples e consistente é uma tabela própria de `pokedex`, com constraint única por `trainer_id` + `pokemon_id`, contendo os atributos individualizados, nickname opcional e estado de descoberta.

Alternativas consideradas:
- Reutilizar `my-pokemon` como fonte da Pokédex: descartado porque posse individual e coleção são conceitos diferentes e esta change deve manter responsabilidades separadas.
- Armazenar progresso da Pokédex em JSON dentro de `trainer`: descartado por dificultar filtros, constraints, paginação e evolução futura.

### 2. Inicializar toda a Pokédex no onboarding do treinador
O onboarding já é responsável por criar o primeiro estado jogável do treinador. Nesta change, ele deve também percorrer o catálogo local de Pokémon e criar uma linha de `pokedex` para cada espécie, marcando apenas a escolhida como descoberta. Isso mantém a coleção pronta desde o primeiro acesso e evita populações lazily inconsistentes.

Alternativas consideradas:
- Criar entradas de Pokédex sob demanda no primeiro acesso à listagem: descartado por aumentar complexidade de leitura e por dificultar contratos determinísticos.
- Criar apenas a entrada do Pokémon inicial: descartado porque o objetivo do domínio é representar a coleção inteira do treinador.

### 3. Reutilizar a fórmula base de atributos de `my-pokemon`
O prompt pede diversidade individual e compatibilidade futura com progressão. A implementação deve seguir a abordagem já adotada em `my-pokemon`, usando stats persistidos do Pokémon base como referência e variação aleatória controlada. Isso reduz retrabalho conceitual e mantém coerência entre domínios individuais do treinador.

Alternativas consideradas:
- Criar fórmula diferente para `pokedex`: descartado por aumentar divergência sem ganho funcional nesta fase.
- Não gerar atributos na Pokédex: descartado porque a proposta exige estrutura pronta para evolução futura.

### 4. Expor descoberta explícita por endpoint, mas usar apenas onboarding nesta fase
O domínio deve ter um endpoint explícito para marcar uma entrada de `pokedex` como descoberta, porque isso simplifica integrações futuras com eventos ou missões. Porém, nesta implementação, o único fluxo que deve invocar esse comportamento é o onboarding do `trainer`.

Alternativas consideradas:
- Não criar endpoint agora: descartado porque o próprio requisito já antecipa uma ação explícita de descoberta.
- Já conectar outros fluxos automáticos de descoberta além do onboarding: descartado nesta change para limitar escopo e evitar acoplamento adicional antes de validar a base da Pokédex.

### 5. Usar `pokemon.name` como identificador externo de `pokedex`
Como existe uma única entrada de `pokedex` por Pokémon base para cada treinador, `pokemon.name` é um identificador externo estável e consistente com o padrão já adotado em outros domínios do projeto. Isso evita expor UUID na URL e simplifica a navegação web.

Alternativas consideradas:
- Expor UUID de `pokedex`: descartado por pior UX e menor consistência com as rotas existentes.
- Criar slug novo por Pokédex: descartado por duplicar a identidade já estável do Pokémon base.

### 6. Manter cache Redis em listagem e detalhe
Listagem e detalhe são os alvos naturais de cache e seguem o padrão já usado por `pokemon` e `my-pokemon`. A descoberta manual e o onboarding alteram estado, então precisam invalidar listagem e detalhe afetados após commit.

Alternativas consideradas:
- Não usar cache inicialmente: descartado porque a proposta pede explicitamente otimização de consultas.
- Cachear onboarding: descartado por não trazer benefício significativo.

### 7. Tratar o onboarding de `trainer` como o ponto de ampliação do contrato
O retorno do onboarding já inclui `my-pokemons`. Para evitar criar um fluxo paralelo, o mesmo contrato deve passar a retornar também `pokedex`, permitindo que web e clientes futuros recebam o estado inicial completo do treinador em uma única resposta.

Alternativas consideradas:
- Criar endpoint adicional para buscar a Pokédex logo após o onboarding: descartado por adicionar roundtrip e acoplamento de cliente.
- Não alterar o contrato de onboarding: descartado porque o prompt exige `trainer` com lista de `pokedex` no retorno.

## Risks / Trade-offs

- **[Risco]** Inicializar a Pokédex inteira no onboarding pode aumentar o tempo da transação à medida que o catálogo crescer. → **Mitigação:** usar queries eficientes, criação em lote quando apropriado e manter a lógica concentrada na API.
- **[Risco]** Duplicar fórmulas de atributos entre `my-pokemon` e `pokedex` pode causar drift futuro. → **Mitigação:** compartilhar ou espelhar a lógica em `business.py` com regras claramente encapsuladas.
- **[Risco]** O endpoint de descoberta existir antes do uso pleno pode criar confusão de escopo. → **Mitigação:** documentar claramente que apenas o onboarding o utiliza nesta fase.
- **[Trade-off]** Retornar `pokedex` junto com `my-pokemons` no onboarding aumenta o payload inicial. → **Justificativa:** evita chamadas extras e entrega o estado completo necessário do treinador.
- **[Trade-off]** Usar `pokemon.name` como identificador externo acopla rotas à identidade do catálogo base. → **Justificativa:** a relação é 1:1 por treinador + Pokémon e o projeto já usa `name` como identificador preferencial.
