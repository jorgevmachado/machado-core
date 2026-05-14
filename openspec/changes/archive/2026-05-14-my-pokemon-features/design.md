## Context

O monorepo já separa claramente API e web, com toda regra de negócio concentrada em `machado-api` e `machado-web` atuando como consumidor via BFF e hooks específicos por recurso. A mudança adiciona um novo agregado de domínio que depende de dados já persistidos de `trainer`, `pokemon` e `moves`, sem consumir a PokéAPI diretamente, e precisa deixar a estrutura pronta para futuras capacidades como batalha, level up, evolução e troca de movimentos.

## Goals / Non-Goals

**Goals:**
- Introduzir um domínio `my-pokemon` persistente, ligado a um único treinador e a um único Pokémon base.
- Gerar atributos individuais e conjunto inicial de até 4 movimentos distintos no momento da criação.
- Persistir PP atual e máximo por movimento para suportar consumo futuro em batalha e restauração posterior.
- Expor contratos normalizados para criação, listagem e detalhe, com cache Redis e invalidação consistente.
- Entregar experiência web de onboarding na `home` para criação de `trainer` + primeiro `my-pokemon`, com nickname opcional e fallback automático para o nome do Pokémon, além da experiência protegida para listar e visualizar detalhe de `my-pokemon` reutilizando padrões existentes de BFF, hooks e design system.

**Non-Goals:**
- Implementar batalha, evolução automática, level up em batalha ou troca ativa de movimentos.
- Consumir novamente a PokéAPI para criação de `my-pokemon`.
- Introduzir nova arquitetura paralela para fluxo de posse de Pokémon.

## Decisions

### 1. Criar `my-pokemon` como novo agregado com tabela própria e associação explícita de movimentos
`my-pokemon` precisa ter ciclo de vida e atributos independentes do Pokémon base. A melhor modelagem é uma tabela principal para o indivíduo do treinador e uma tabela associativa própria para os movimentos equipados, contendo `pp`, `max_pp` e timestamps/soft delete.

Alternativas consideradas:
- Reutilizar tabela de associação genérica `pokemon_moves`: descartado porque mistura catálogo base com estado individual do treinador.
- Armazenar movimentos como JSON em `my-pokemon`: descartado por dificultar constraints, consultas e evolução futura do domínio.

### 2. Centralizar toda geração de atributos, nickname efetivo e seleção de movimentos na API
A API já é a fonte de verdade do domínio e o frontend não deve duplicar regras. A criação de `my-pokemon` deve buscar o Pokémon base e seus movimentos locais, definir o nickname efetivo, calcular atributos derivados com fator aleatório controlado e persistir até 4 movimentos distintos em transação única. Quando o Pokémon base tiver menos de 4 movimentos elegíveis, a criação permanece válida com a quantidade disponível.

Alternativas consideradas:
- Sortear atributos no frontend: descartado por quebrar consistência e segurança.
- Persistir apenas referência ao Pokémon base e gerar o restante on-read: descartado por aumentar custo de leitura e dificultar evolução individual.

### 2.1 Aplicar fallback de nickname no backend
O onboarding deve aceitar nickname opcional para qualquer perfil. Quando o usuário não informar nickname, ou enviar somente espaços, a API deve persistir como nickname o `name` do Pokémon selecionado, garantindo consistência independentemente do comportamento do client.

Alternativas consideradas:
- Exigir nickname obrigatório: descartado por aumentar atrito no onboarding.
- Fazer fallback apenas no frontend: descartado por permitir inconsistências entre clientes.

### 2.2 Gerar `name` público estável e único por treinador a partir do nickname efetivo
O frontend navega e busca detalhe por `name`, então o backend precisa derivar esse identificador de forma consistente. A implementação deve fazer slug do nickname efetivo, remover acentos e caracteres especiais e garantir unicidade por treinador com sufixos incrementais quando necessário.

Alternativas consideradas:
- Expor diretamente o nickname na URL sem normalização: descartado por fragilidade com espaços, acentos e colisões.
- Expor UUID na URL: descartado por pior UX e menor aderência ao padrão já usado em outros domínios.

### 3. Usar fórmula simples baseada no Pokémon base com variação aleatória limitada
Os atributos precisam refletir o Pokémon base e ainda diferenciar indivíduos. A implementação deve usar os stats persistidos do Pokémon base como referência, aplicar um multiplicador aleatório pequeno por atributo e derivar `max_hp`, `hp`, `experience` inicial e `level` inicial de forma determinística dentro da regra de negócio.

Sugestão inicial para implementação:
- `level`: iniciar em `1`
- `experience`: iniciar em `0`
- Para `attack`, `defense`, `speed`, `special_attack`, `special_defense`: `floor(base_stat * roll)` com `roll` entre `0.90` e `1.10`, garantindo mínimo `1`
- Para `max_hp`: `floor(base_hp * roll_hp) + 5` com `roll_hp` entre `0.95` e `1.15`, garantindo mínimo `10`
- Para `hp`: iniciar igual a `max_hp`

Essa fórmula é simples, coerente com o Pokémon base e fácil de evoluir depois sem redesenhar o domínio.

Alternativas consideradas:
- Fórmula fiel aos jogos principais: descartada nesta fase por aumentar complexidade sem necessidade do MVP.
- Valores totalmente aleatórios sem base no Pokémon: descartado por produzir resultados incoerentes com a espécie base.

### 4. Manter cache Redis somente para listagem e detalhe
O fluxo de criação altera estado e deve persistir primeiro; listagem e detalhe são os alvos claros de otimização. O domínio deve seguir o padrão já usado na API com chaves determinísticas por trainer, paginação, filtros e identificador do `my-pokemon`, invalidando cache após criação ou atualização relevante.

Alternativas consideradas:
- Não usar cache inicialmente: descartado porque o prompt pede explicitamente Redis para leitura.
- Cachear criação: descartado por não trazer benefício real.

### 5. Seguir o padrão web existente com BFF + hooks específicos
No `machado-web`, a listagem e o detalhe devem viver em `app/(protected)/my-pokemon/`, com route handlers em `app/api/my-pokemon/`, service próprio e hooks de listagem/detalhe seguindo o padrão de `pokemon`, `type`, `ability` e `move`. O fluxo inicial deve começar na `home` e só aparecer quando o usuário ainda não tiver `trainer`: qualquer usuário pode informar nickname opcional; usuários comuns escolhem um starter entre `Bulbasaur`, `Charmander` e `Squirtle`, enquanto usuários `admin` podem pesquisar livremente na lista existente de Pokémon com autocomplete e preview em card antes do salvamento.

Alternativas consideradas:
- Chamar a API diretamente do client: descartado por fugir do padrão BFF autenticado.
- Criar hook genérico novo para detalhe/criação: descartado por adicionar abstração desnecessária.

### 5.1 Colocar o onboarding em `trainer`, não em `my-pokemon`
O onboarding inicial cria um agregado de treinador e já retorna o primeiro Pokémon do elenco. Por isso, o endpoint canônico deve pertencer ao domínio `trainer` (`POST /trainer/onboarding`), enquanto `POST /my-pokemon` continua reservado para criações posteriores de um treinador já existente. A BFF do frontend segue a mesma separação.

Alternativas consideradas:
- Reutilizar `POST /my-pokemon` para onboarding: descartado por misturar pré-condições de criação de treinador com criação recorrente de Pokémon possuído.
- Criar endpoint isolado fora de qualquer domínio: descartado por enfraquecer a modelagem e a descoberta da API.

### 6. Usar `name` como identificador externo do domínio no fluxo web
Para a navegação e a API web consumida por BFF, o identificador externo preferencial deve ser `name`, seguindo o padrão já usado em outros domínios do projeto e evitando UUID exposto na URL da interface.

Alternativas consideradas:
- Expor UUID no path: descartado por pior UX e menor consistência com as rotas existentes.
- Criar slug adicional: descartado por não agregar valor nesta fase.

## Risks / Trade-offs

- **[Risco]** Fórmula simples de atributos pode não refletir bem o balanceamento futuro de batalha. → **Mitigação:** encapsular cálculo em `business.py` para permitir refinamento posterior sem quebrar contratos externos.
- **[Risco]** Misturar onboarding inicial e captura avançada no mesmo fluxo pode confundir a UX. → **Mitigação:** limitar esta change ao onboarding inicial sem `trainer`, com regra simples por role.
- **[Risco]** Reutilizar a listagem existente de Pokémon para seleção livre do admin pode crescer demais no futuro. → **Mitigação:** limitar o payload aos campos necessários para autocomplete/card preview e reavaliar paginação, busca dedicada ou carregamento incremental se a base aumentar.
- **[Risco]** Colisões de nickname podem gerar URLs ambíguas no elenco do mesmo treinador. → **Mitigação:** gerar slug estável e acrescentar sufixos incrementais por treinador durante a criação.
- **[Risco]** Cache inconsistente após criação/edição pode exibir elenco desatualizado. → **Mitigação:** invalidar domínio por trainer e detalhe individual na camada de service ao final da transação.
- **[Trade-off]** Guardar PP atual e máximo por movimento agora adiciona modelagem extra antes da batalha existir. → **Justificativa:** reduz retrabalho futuro e atende a preparação pedida pelo domínio.
