## Why

As propostas anteriores entregaram a primeira versão da batalha, mas deixaram inconsistências entre spec, backend e frontend que hoje degradam o fluxo principal de exploração. Esta mudança corrige a direção arquitetural agora, antes que a nomenclatura, os contratos e a UX da batalha contaminem futuras extensões como dashboard de batalhas, captura e progressão.

## What Changes

- Normalizar o domínio de batalha para uma nomenclatura neutra e extensível, removendo o viés de que a sessão existe apenas para encontros `wild`.
- Alinhar contratos de status, payloads e semantics entre OpenSpec, backend, BFF, frontend e testes.
- Corrigir a leitura de sessão ativa para que ausência de batalha ativa não quebre Home, modal ou tela protegida.
- Substituir o handoff `encounter -> página /battle` por `encounter -> modal`, reutilizando o componente e hook de modal já existentes no `machado-web`.
- Manter `/battle` disponível para uso futuro como página de informações/resumo de batalhas, sem depender dela como entrypoint automático do encounter.
- Consolidar os endpoints de batalha sob namespace explícito `/trainer/battle/*` e documentar compatibilidade temporária se algum path legado ainda existir.
- Estender o payload agregado da Home com resumo confiável de batalha ativa para retomada do fluxo sem heurísticas locais frágeis.
- **BREAKING**: padronizar os estados finais canônicos da sessão e remover divergência entre nomes de enum/contrato atualmente usados.
- **BREAKING**: renomear o domínio/capability interno de batalha para um nome neutro, com plano explícito de transição em código, specs e testes.

## Capabilities

### New Capabilities
- _None_

### Modified Capabilities
- `wild-pokemon-battle-session`: os requisitos da sessão de batalha serão corrigidos para refletir nomenclatura neutra, estados finais canônicos, contrato estável de sessão ativa, UX via modal e namespace explícito de endpoints de batalha.
- `trainer-exploration`: os requisitos de walk/Home serão ajustados para refletir handoff de encounter para modal, resumo de batalha ativa no payload agregado e integração estável com a sessão de batalha em andamento.

## Impact

- API: refactor do domínio de batalha, revisão de enums, schemas, serviços, rotas e eventuais aliases de compatibilidade.
- Web: ajustes no handoff de encounter, reuso do modal existente, revisão da tela protegida de batalha e atualização da Home com resumo de batalha ativa.
- OpenSpec: atualização das specs existentes para refletir o contrato real desejado em vez da implementação atual.
- Testes: revisão e ampliação de testes backend, BFF e frontend para ausência de batalha ativa, modal de encounter, estados terminais e resumo da Home.
- Migração: necessidade de plano explícito para rename interno e compatibilidade de contratos já consumidos.
