# Proposta: battle-session-normalization-and-web-stability

## 0. Contexto base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-API](../docs/architecture-api.md)
- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS API](../machado-api/AGENTS.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

Também é obrigatório revisar antes de propor qualquer mudança:

- `openspec/specs/wild-pokemon-battle-session/spec.md`
- `openspec/changes/archive/2026-05-20-wild-pokemon-battle-session/`
- `openspec/changes/wild-pokemon-battle-session-web-completion/`
- `prompts/9_wild-pokemon-battle-session.md`
- `prompts/9.1_wild-pokemon-battle-session-web-completion.md`

As decisões devem respeitar esses documentos, mas a nova proposta não deve assumir que a modelagem ou os contratos atuais estão corretos só porque já foram implementados.

---

## 1. Objetivo desta nova proposta

Criar uma proposta corretiva para normalizar a modelagem da sessão de batalha e estabilizar o fluxo web da batalha já implementada.

Esta nova proposta deve tratar explicitamente problemas deixados pelas propostas anteriores, em vez de apenas estender a implementação existente.

O resultado esperado é:

- nomenclatura de domínio, models, enums, specs e payloads sem acoplamento desnecessário ao conceito de `wild`;
- contratos backend/web coerentes entre spec, implementação e UI;
- endpoint `/battle/active` e fluxo equivalente no BFF tratados de forma resiliente quando não existir batalha ativa;
- Home exibindo de forma consistente o estado resumido de batalha ativa;
- encontro com Pokémon selvagem abrindo fluxo de batalha em modal, sem redirecionar imediatamente para uma página dedicada;
- frontend sem quebrar após finalizar uma batalha.

---

## 2. Problemas concretos já identificados e que a proposta deve corrigir

### 2.1 Nomenclatura ruim

A modelagem atual usa nomes como `wild_pokemon_battle_session` e `wild-pokemon-battle-session`, o que passa a ideia de um domínio exclusivo para batalha selvagem.

A nova proposta deve corrigir isso e adotar uma nomenclatura neutra para o aggregate/capability principal da batalha, preparada para futuras extensões.

Importante:

- não usar `wild` no nome principal do domínio, model ou capability;
- o fato de a V1 atual ser contra Pokémon selvagem deve aparecer como contexto do fluxo, não como identidade central do domínio;
- a proposta deve definir claramente a estratégia de rename em backend, spec, web, testes e migrations.

### 2.2 Contrato inconsistente entre spec e implementação

A spec atual descreve estados finais como:

- `ESCAPED`
- `WILD_POKEMON_DEFEATED`
- `TRAINER_DEFEATED`

Mas a implementação atual usa estados como:

- `FLED`
- `WON`
- `LOST`

A nova proposta deve escolher um contrato final consistente e exigir alinhamento completo entre:

- spec OpenSpec;
- enums do backend;
- schemas e responses;
- tipos do frontend;
- traduções e UI states;
- testes.

### 2.3 Fluxo web frágil ao consultar batalha ativa

Hoje, após finalizar a batalha, a leitura do endpoint `/active` quebra o fluxo da página com frequência.

A nova proposta deve definir explicitamente que:

- ausência de batalha ativa não é erro fatal de UX;
- o contrato deve retornar estado vazio explícito ou um tratamento BFF/UI equivalente, sem quebrar a tela;
- página de batalha, Home e demais consumidores não podem assumir que sempre existe sessão ativa;
- o polling e os refreshes pós-ação devem parar ou se adaptar corretamente quando a batalha terminar.

### 2.4 Home sem resumo/pendência de batalha ativa

A Home atual não apresenta um balanço/resumo consistente da batalha ativa para o treinador.

A nova proposta deve definir:

- se a Home vai receber um bloco resumido de batalha ativa no payload agregado;
- quais campos mínimos a Home precisa para retomar a batalha;
- como essa informação é invalidada/atualizada ao iniciar, encerrar ou abandonar uma batalha;
- como evitar que a Home dependa de chamadas frágeis extras para montar esse resumo.

### 2.5 Handoff de exploração para batalha com UX inadequada

Hoje o fluxo navega para a tela de batalha quando encontra um Pokémon selvagem. Essa decisão não é desejada para a V2.

A nova proposta deve definir explicitamente que:

- ao encontrar um Pokémon selvagem, o frontend deve abrir uma modal;
- essa modal deve reutilizar o componente de modal já existente no design system de `machado-web`;
- a implementação deve reutilizar o hook de modal já existente no projeto, em vez de criar infraestrutura paralela;
- a rota `/battle` não deve ser o destino automático desse fluxo, porque deve permanecer liberada para uso futuro como página de informações/histórico/resumo de batalhas.

### 2.6 Namespacing ruim dos endpoints de ação

As ações de batalha precisam deixar claro no contrato HTTP que pertencem ao contexto de batalha.

A nova proposta deve exigir que:

- endpoints de ação não usem paths genéricos/ambíguos como `/trainer/move`;
- o namespace de batalha fique explícito sob algo como `/trainer/battle/*`;
- a proposta decida com clareza a estrutura final dos endpoints de leitura e ação da batalha;
- qualquer compatibilidade temporária, se necessária, seja tratada como transição explícita e não como contrato definitivo.

---

## 3. Restrições obrigatórias

Esta proposta NÃO deve:

- tratar o problema como ajuste visual isolado;
- mascarar erro com `try/catch` sem corrigir contrato e fluxo;
- duplicar regras de batalha no frontend;
- criar uma modal nova se já existir componente/hook aderente no `machado-web`;
- manter a nomenclatura principal presa a `wild`;
- alterar migrations históricas já aplicadas;
- criar um segundo fluxo paralelo de batalha sem plano explícito de migração;
- quebrar compatibilidade sem mapear claramente impacto e transição.

Esta proposta DEVE:

- revisar o domínio atual como uma correção arquitetural;
- mapear rename de capability, domínio, models, rotas, schemas, types e testes;
- decidir explicitamente o comportamento canônico de “sem batalha ativa”;
- incluir impacto na Home agregadora;
- definir estratégia de transição segura para código já existente.

---

## 4. Perguntas que a proposta deve responder explicitamente

1. Qual será o novo nome canônico do domínio/capability?
2. O path HTTP continuará igual por compatibilidade ou será renomeado?
3. Como o backend representará “não existe batalha ativa”?
4. Como o BFF/web vão consumir esse caso sem quebrar a página?
5. Quais campos novos entram no payload da Home para resumir batalha ativa?
6. Como o fluxo de encounter abrirá modal usando a infraestrutura já existente de modal?
7. O que ficará na rota `/battle` agora e o que ficará explicitamente fora do escopo dela?
8. Como ficam os estados finais canônicos da sessão?
9. Quais endpoints finais de batalha serão expostos sob `/trainer/battle/*`?
10. Quais arquivos/tabelas/entidades serão renomeados e quais permanecerão só por compatibilidade?
11. Quais testes precisam ser adicionados para impedir regressão do `/active`, da modal e da Home?

---

## 5. Escopo funcional esperado

### Dentro do escopo

- normalização da nomenclatura do domínio de batalha;
- alinhamento entre OpenSpec, backend e frontend;
- correção do contrato/leitura de batalha ativa;
- correção do fluxo da tela de batalha após término;
- substituição do redirecionamento automático por abertura de modal no fluxo de encounter;
- preservação da rota `/battle` para uso futuro sem depender dela como entrypoint do encounter;
- normalização do namespace de endpoints de ação da batalha;
- atualização da Home para refletir batalha ativa em andamento ou recém-finalizada conforme a decisão da proposta;
- ajuste dos tipos, traduções e testes afetados;
- definição de estratégia de migração/refatoração incremental.

### Fora do escopo

- captura;
- experiência;
- level up;
- evolução;
- PvP;
- itens;
- efeitos avançados;
- WebSocket;
- redesign visual amplo fora do necessário para estabilizar o fluxo.

---

## 6. Requisitos obrigatórios para a proposta

### 6.1 Domínio e nomenclatura

- O nome principal da capability e do domínio deve ser neutro e extensível.
- O contexto “wild encounter” deve ficar como origem da sessão, não como nome do aggregate.
- A proposta deve explicar por que o novo nome escolhido é melhor para evolução futura.

### 6.2 Contrato de sessão ativa

- A proposta deve especificar um comportamento único e explícito para leitura da sessão ativa.
- “Sem sessão ativa” não pode continuar quebrando o fluxo do frontend.
- Se a decisão for manter `404`, a proposta deve obrigar BFF/UI a tratarem isso como empty state estável.
- Se a decisão for mudar o contrato para resposta vazia normalizada, a proposta deve detalhar o novo schema.

### 6.3 Home agregadora

- A proposta deve atualizar o contrato da Home para incluir visibilidade de batalha ativa.
- O payload agregado deve permitir retomar a batalha a partir da Home sem heurísticas frágeis no client.
- A proposta deve definir como o cache/invalidação da Home se comporta quando a batalha inicia, muda ou termina.

### 6.4 UX de modal e roteamento

- Ao encontrar um Pokémon selvagem, o fluxo principal deve abrir modal e não navegar automaticamente para `/battle`.
- A proposta deve referenciar explicitamente o reuso do componente/hook de modal já existente em `machado-web/app/ds/modal/`.
- A proposta deve definir como a modal consome o estado da batalha ativa e como ela é fechada ao finalizar/cancelar.
- A rota `/battle` deve permanecer livre para evolução futura e não pode continuar sendo o acoplamento principal do encounter.

### 6.5 Web resiliente

- A tela de batalha deve suportar loading, empty, active, terminal e error states reais.
- O frontend não deve redirecionar de forma prematura sem estabilizar o estado final.
- Polling e refresh pós-ação devem respeitar fim de batalha.
- A proposta deve prever testes cobrindo finalização da batalha seguida de nova leitura de `/active`.

### 6.6 Contrato HTTP de batalha

- Os endpoints de batalha devem deixar o contexto explícito no path.
- A proposta deve privilegiar estrutura namespaced como `/trainer/battle/active`, `/trainer/battle/move`, `/trainer/battle/switch`, `/trainer/battle/flee` e equivalentes definidos pela decisão final.
- A proposta não deve aceitar paths genéricos como contrato final para ações de batalha.
- Se existir endpoint legado, a proposta deve documentar estratégia de compatibilidade e remoção.

### 6.7 Testes e regressão

- Incluir testes backend para contrato de sessão ativa.
- Incluir testes BFF/web para ausência de batalha ativa.
- Incluir testes da Home para presença de resumo de batalha.
- Incluir testes do fluxo de encounter abrindo modal com o hook/componente existente.
- Incluir teste de fluxo: finalizar batalha -> acessar `/active` -> UI não quebra.

---

## 7. Orientação para o desenho da mudança

Ao gerar `proposal.md`, `design.md`, `tasks.md` e specs:

- priorizar correção arquitetural e estabilidade de fluxo;
- não assumir que a proposta anterior deve ser preservada como está;
- tratar rename, compatibilidade e transição como parte central do design;
- explicitar riscos de refactor e mitigação;
- quebrar as tasks em blocos independentes:
  - spec/contracts
  - backend rename/alignment
  - comportamento de active session
  - home aggregation
  - modal encounter flow
  - web resilience
  - tests

---

## 8. Resultado esperado da proposta OpenSpec

A proposta final deve sair pronta para implementação e deixar cristalino:

- o novo nome canônico do domínio de batalha;
- o contrato final de sessão ativa;
- o contrato final de estados/status;
- o comportamento oficial de encounter -> modal;
- a estratégia para manter `/battle` disponível para uso futuro;
- o namespace final dos endpoints de batalha;
- a participação da Home nesse fluxo;
- o plano de implementação incremental sem quebrar o que já existe.

Se houver ambiguidade entre compatibilidade e correção, a proposta deve explicitar a decisão, o impacto e a estratégia de migração.
