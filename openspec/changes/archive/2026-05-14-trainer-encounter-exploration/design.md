## Context

O monorepo já possui onboarding de `trainer`, catálogo local de `pokemon` com `pokemon_encounters`, roster de `my-pokemon` e descoberta de `pokedex`, mas a aplicação ainda não tem um agregado operacional que conecte esses domínios em um loop principal de exploração. A mudança é transversal: afeta persistência do trainer, onboarding, contratos autenticados da API, cache, BFFs do web e a composição da Home protegida.

Há restrições claras do projeto:
- toda regra de negócio deve permanecer na API;
- a web deve consumir contratos normalizados via BFF;
- o projeto já usa padrões de cache por domínio, paginação comum e arquitetura em 5 camadas;
- a change deve preparar integração futura com batalha e desbloqueio de encounters sem ativar esses fluxos agora.

## Goals / Non-Goals

**Goals:**
- introduzir posse de encounters pelo treinador com seleção de encounter ativo e unicidade do ativo;
- permitir caminhada autenticada no encounter ativo conhecido, retornando um evento de exploração normalizado;
- suportar dois eventos iniciais de exploração: encontro de Pokémon selvagem e achado de pokebolas;
- adicionar party principal persistida com até 6 `my-pokemon`;
- expor um payload de Home do trainer com summary, encounter ativo, party principal e últimos 3 descobertos da `pokedex`;
- preparar modelos e contratos para futura evolução de batalha e desbloqueio de encounters.

**Non-Goals:**
- sistema de batalha, dano, turnos, IA, captura e consumo de pokebolas em batalha;
- progressão de experiência/level-up decorrente da exploração;
- desbloqueio efetivo de novos encounters por vitória;
- múltiplos tipos complexos de evento além de wild Pokémon e item de pokebola.

## Decisions

### 1. Representar encounters conhecidos do trainer com associação dedicada
Criar uma associação persistida entre `trainer` e `pokemon_encounter` para modelar posse, estado ativo e timestamps, em vez de derivar “encounters conhecidos” dinamicamente do roster atual.

Rationale:
- preserva histórico e prepara desbloqueio futuro de novos encounters;
- permite garantir apenas um encounter ativo por trainer;
- simplifica filtros, cache e payload de Home.

Alternativas consideradas:
- derivar encounters conhecidos a cada request a partir dos `my-pokemon`.
  Rejeitada porque não preserva estado ativo nem desbloqueios futuros.

### 2. Inicializar encounters no onboarding a partir do Pokémon inicial
No onboarding, após criar trainer, `my-pokemon` inicial e `pokedex`, a API também criará os encounters vinculados ao Pokémon inicial e marcará um encounter ativo automaticamente de forma determinística.

Rationale:
- evita estado inicial inválido na Home;
- reduz atrito antes do primeiro walk;
- mantém toda a inicialização principal do trainer no mesmo fluxo transacional.

Alternativas consideradas:
- exigir seleção manual do primeiro encounter.
  Rejeitada por adicionar um passo obrigatório sem ganho funcional real.

### 3. Modelar a party principal como entidade persistida por slot
Persistir `trainer_party` com `trainer_id`, `my_pokemon_id`, `slot`, `is_active` e soft delete, tratando a party como seleção explícita do roster em vez de lista embutida em `trainer`.

Rationale:
- encaixa no padrão relacional já usado no projeto;
- permite ordenação estável por slot;
- facilita validação de limite de 6 e futuras trocas/reordenações.

Alternativas consideradas:
- armazenar ids da party em campo JSON no `trainer`.
  Rejeitada por piorar integridade referencial, queries e evolução futura.

### 4. Tratar caminhada como caso de uso no domínio do trainer/exploration
Adicionar um domínio novo de exploração do trainer com serviço responsável por validar encounter ativo, selecionar evento aleatório, atualizar pokeballs quando necessário e retornar um payload normalizado.

Rationale:
- concentra o loop de exploração em um único domínio orquestrador;
- evita espalhar regras entre `trainer`, `pokemon_encounter`, `pokedex` e `my-pokemon`;
- facilita evolução futura para batalha sem mudar a API pública principal.

Alternativas consideradas:
- distribuir endpoints entre domínios existentes (`trainer`, `pokemon/encounter`, `my-pokemon`).
  Rejeitada por gerar acoplamento operacional e contratos fragmentados.

### 5. Persistir `exploration_event` como estrutura mínima para auditoria e evolução
Persistir eventos de exploração com `event_type`, `payload`, timestamps e vínculo ao trainer, mesmo que a Home ou listagem futura não dependam disso nesta primeira entrega.

Rationale:
- prepara rastreabilidade e futuras timelines;
- evita redesenho de contrato quando novos tipos de evento forem introduzidos;
- permite idempotência e depuração melhores em fluxos de walk.

Alternativas consideradas:
- retornar evento só em memória, sem persistência.
  Rejeitada por limitar auditoria e evolução futura.

### 6. Expor Home agregada via endpoint dedicado do trainer
Adicionar um endpoint autenticado de Home do trainer que agregue summary do trainer, encounter ativo, party principal e os 3 últimos descobertos da `pokedex`, com cache próprio.

Rationale:
- reduz round-trips do frontend;
- centraliza a composição do dashboard na API;
- mantém a web como consumidora simples de um contrato pronto.

Alternativas consideradas:
- montar a Home no frontend combinando vários endpoints.
  Rejeitada por duplicar composição e aumentar acoplamento do web.

## Risks / Trade-offs

- [Transação de onboarding mais complexa] → Mitigar com criação orquestrada em uma única sessão e testes de rollback cobrindo trainer, my-pokemon, pokedex e encounters.
- [Exploração aleatória dificulta testes determinísticos] → Mitigar isolando regras puras de sorteio no `business.py` com pontos de controle mockáveis.
- [Cache da Home pode ficar stale após múltiplas mutações] → Mitigar definindo invalidação explícita após onboarding, troca de encounter ativo, caminhada e atualização da party.
- [Party pode referenciar `my-pokemon` removido/inválido] → Mitigar com validação de ownership, soft delete exclusion e constraints relacionais.
- [Modelo de evento persistido pode crescer rápido] → Mitigar mantendo payload mínimo nesta change e sem criar leituras históricas complexas por enquanto.
