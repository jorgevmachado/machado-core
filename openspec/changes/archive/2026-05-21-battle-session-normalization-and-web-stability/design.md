## Context

O monorepo já possui uma V1 funcional de batalha iniciada pela exploração, mas a mudança anterior introduziu três problemas estruturais: o domínio foi nomeado como se fosse intrinsecamente `wild`, a spec principal divergiu dos enums e payloads efetivamente implementados, e o fluxo web ficou frágil ao depender de redirecionamento imediato para `/battle` e de leituras de `/active` que podem falhar após o fim da batalha.

Além disso, o `machado-web` já possui infraestrutura adequada de modal em `app/ds/modal/`, incluindo `Modal` e `useModal`, então criar uma segunda infraestrutura para o handoff da batalha aumentaria acoplamento e inconsistência visual. A Home também já é o agregado natural do fluxo de exploração, mas hoje não expõe um resumo confiável da batalha ativa.

## Goals / Non-Goals

**Goals:**
- Corrigir a identidade do domínio de batalha para um nome neutro e preparado para futuras formas de batalha.
- Alinhar contrato canônico de status, active session e endpoints entre spec, backend, BFF e frontend.
- Tornar ausência de batalha ativa um caso suportado, não um erro fatal de UX.
- Trocar o handoff de encounter para abertura de modal reutilizando o DS/hook existente.
- Acrescentar resumo de batalha ativa na Home para retomada confiável do fluxo.
- Preservar `/trainer/battle/*` como namespace explícito de APIs de batalha e manter `/battle` livre para uso futuro como página informativa.

**Non-Goals:**
- Implementar captura, XP, level up, evolução, PvP, itens ou engine avançada.
- Redesenhar o design system de modal ou criar nova infraestrutura de overlay.
- Criar dashboard final de batalhas em `/battle`; a rota fica reservada, não detalhada nesta mudança.
- Introduzir websocket ou sincronização em tempo real fora do polling já compatível com o escopo.

## Decisions

### Categoria 1. Domínio e nomenclatura

1. **O nome canônico do domínio deve ser neutro, mantendo `wild` apenas como contexto da origem atual da sessão**
   - **Why:** o aggregate representa uma sessão de batalha, não uma mecânica exclusiva de encontro selvagem. Isso reduz retrabalho para futuras batalhas PvE adicionais e evita contaminação semântica em models, services e specs.
   - **Alternative considered:** manter `wild_pokemon_battle_session` e documentar melhor. Rejeitado porque a ambiguidade já está produzindo design ruim e confusão de escopo.

2. **A capability OpenSpec existente continuará sendo modificada no diretório atual, mas com plano de rename interno no código**
   - **Why:** o spec principal existente já está em `openspec/specs/wild-pokemon-battle-session/spec.md`; a mudança precisa corrigir o comportamento agora sem depender de uma migração estrutural imediata do repositório de specs.
   - **Alternative considered:** renomear imediatamente o diretório da spec principal. Rejeitado nesta mudança por aumentar o custo operacional sem benefício proporcional para o apply imediato.

### Categoria 2. Contratos backend e compatibilidade

3. **Os estados finais canônicos devem seguir a spec semântica (`ESCAPED`, `WILD_POKEMON_DEFEATED`, `TRAINER_DEFEATED`)**
   - **Why:** esses nomes carregam semântica de negócio explícita e evitam a ambiguidade de `WON`/`LOST`/`FLED` em payloads públicos.
   - **Alternative considered:** simplificar em `WON`/`LOST`/`FLED`. Rejeitado porque isso perde contexto e já conflitou com a spec aprovada.

4. **A leitura da sessão ativa deve ter um comportamento único e explícito para “nenhuma batalha ativa”**
   - **Why:** o fluxo atual quebra porque a API/BFF/UI tratam esse caso de formas diferentes. O contrato final fica fechado em `404` no endpoint de active battle, com tratamento de empty state estável no BFF/frontend.
   - **Alternative considered:** manter a ambiguidade atual e “tratar no client”. Rejeitado porque perpetua regressões.

5. **Os endpoints públicos de batalha devem permanecer sob `/trainer/battle/*`**
   - **Why:** o namespace já comunica contexto corretamente e evita paths genéricos de ação. A mudança deve consolidar isso como contrato definitivo e documentar qualquer alias legado apenas como transição.
   - **Alternative considered:** expor ações sob `/trainer/move`, `/trainer/flee` ou paths sem namespace. Rejeitado por baixa clareza semântica.

### Categoria 3. UX web e roteamento

6. **O handoff de encounter para batalha deve abrir modal em vez de navegar automaticamente para `/battle`**
   - **Why:** a batalha ativa nasce dentro do fluxo de exploração e precisa manter o usuário no contexto principal. O projeto já possui `Modal` e `useModal`, então a solução mais consistente é reusar essa infraestrutura.
   - **Alternative considered:** manter redirecionamento para página dedicada. Rejeitado porque acopla demais a jornada de exploração a uma rota que o produto quer reservar para uso futuro.

7. **A rota `/battle` deve continuar existindo, mas desacoplada do encounter como destino automático**
   - **Why:** isso preserva espaço para um dashboard futuro de batalhas, histórico ou resumo sem invalidar a rota protegida atual.
   - **Alternative considered:** remover `/battle`. Rejeitado porque elimina um ponto útil para evolução futura e exigiria refactor adicional depois.

8. **A Home deve expor um resumo mínimo de batalha ativa**
   - **Why:** o usuário precisa poder retomar batalha pela Home mesmo sem depender do `lastEvent` local ou de polling oportunista.
   - **Alternative considered:** deixar a Home como está e fazer uma chamada separada sempre que precisar. Rejeitado por fragilidade, complexidade desnecessária e acoplamento de UI.

### Categoria 4. Sugestões de implementação

1. **Sugestões para API**
   - Introduzir schema resumido de batalha ativa para a Home em vez de reutilizar o payload completo da sessão.
   - Concentrar a decisão de “sem batalha ativa” em um único service/BFF mapping reutilizável.
   - Manter compatibilidade apenas como shim interno de import se algum código ainda depender do package legado; não reintroduzir endpoints HTTP legados.

2. **Sugestões para Web**
   - Encapsular a battle modal em feature própria, mas renderizada a partir da Home/exploration flow via `useModal`.
   - Evitar `router.replace('/home')` automático ao terminar a batalha; primeiro estabilizar estado terminal, depois fechar modal/atualizar Home.
   - Manter `/battle` como leitura complementar, não como passo obrigatório do encounter.

3. **Sugestões para testes**
   - Cobrir explicitamente `walk -> has_active_battle -> openModal`.
   - Cobrir `terminal action -> refresh active session -> empty state estável`.
   - Cobrir invalidação de Home ao iniciar, atualizar e finalizar batalha.

## Risks / Trade-offs

- **[Risk] Rename interno atravessa backend, web e testes ao mesmo tempo** → **Mitigation:** executar em camadas, começando por contratos e aliases estáveis antes de remover nomes antigos.
- **[Risk] Compatibilidade parcial entre enums antigos e novos durante a transição** → **Mitigation:** definir mapping temporário explícito e remover a ambiguidade no contrato público antes de concluir a mudança.
- **[Risk] Modal de batalha introduzir estado duplicado entre Home e feature de batalha** → **Mitigation:** manter o servidor como autoridade única e usar resumo da Home apenas para descoberta/retomada.
- **[Risk] Reservar `/battle` sem redefinir seu uso imediato parecer trabalho incompleto** → **Mitigation:** documentar claramente que a rota permanece válida, mas fora do handoff automático do encounter nesta mudança.
- **[Risk] Escolha errada para “sem batalha ativa” continuar causando regressão** → **Mitigation:** exigir contrato único com cobertura de backend, BFF e frontend para esse caso.

## Resolved Contract

1. **No active battle**
   - API: `GET /trainer/battle/active` retorna `404 Not Found` com detalhe explícito.
   - BFF: propaga o `404` como resposta conhecida do domínio de batalha.
   - Frontend: trata `404` como ausência suportada de batalha ativa, preservando snapshot terminal quando existir e interrompendo polling.
   - Home: usa `active_battle: null` quando não há sessão ativa.

2. **Battle endpoints**
   - Endpoints canônicos: `GET /trainer/battle/active`, `GET /trainer/battle/logs`, `POST /trainer/battle/move`, `POST /trainer/battle/switch`, `POST /trainer/battle/flee`.
   - `GET /trainer/battle/logs` pode retornar logs da sessão terminal mais recente quando a sessão ativa já terminou, permitindo refresh estável pós-batalha.

3. **Legacy compatibility**
   - Não existe path HTTP legado canônico fora de `/trainer/battle/*`.
   - Compatibilidade temporária, quando necessária, fica restrita a aliases internos de import para evitar duplicação do domínio neutro de battle session.

## Migration Plan

1. Atualizar proposal/specs para fixar o contrato desejado e impedir que a implementação atual continue sendo a referência semântica.
2. Introduzir alinhamento de enums/status e comportamento canônico de active session no backend.
3. Ajustar BFF e tipos do frontend para o contrato final.
4. Mover o handoff de encounter para modal com reuso de `Modal`/`useModal`.
5. Acrescentar resumo de batalha ativa na Home e revisar invalidação de cache.
6. Manter `/battle` acessível, mas retirar o redirecionamento automático do encounter.
7. Executar testes de regressão para active session, Home e modal antes de remover compatibilidades transitórias.

## Open Questions

- O rename estrutural de models/tabelas históricas deve acontecer em uma mudança futura dedicada, já que esta iteração fecha o contrato e elimina a duplicação ativa do domínio sem reescrever persistência histórica.
