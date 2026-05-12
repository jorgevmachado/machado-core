## Context

O `machado-web` usa Next.js 16 com App Router, mistura Server e Client Components e já possui providers globais registrados em [machado-web/app/layout.tsx](/home/jorge.machado/MY_PROJECTS/MACHADO/machado-core/machado-web/app/layout.tsx). Hoje os textos da interface estão espalhados entre páginas, componentes de navegação, views de feature, actions e mensagens de erro, sem uma camada centralizada de locale. O navbar existente fica em [machado-web/app/ui/features/navigation/Navbar.tsx](/home/jorge.machado/MY_PROJECTS/MACHADO/machado-core/machado-web/app/ui/features/navigation/Navbar.tsx), com navegação configurada em [machado-web/app/ui/features/navigation/constants.ts](/home/jorge.machado/MY_PROJECTS/MACHADO/machado-core/machado-web/app/ui/features/navigation/constants.ts), o que o torna o ponto natural para incluir o seletor de idioma.

As restrições são claras: a mudança deve atingir apenas `machado-web`, sem criar novas regras no backend, sem alterar contratos da `machado-api` e sem introduzir uma arquitetura paralela à já adotada pelo projeto. O fallback oficial desta change passa a ser `en`, e os artefatos precisam tratar esse comportamento como a regra padrão de resolução de locale.

## Goals / Non-Goals

**Goals:**
- Centralizar a configuração de internacionalização do frontend em um único módulo de locale.
- Suportar `pt-BR`, `en` e `es` com resolução inicial por prioridade: locale persistido, locale do navegador, fallback `en`.
- Disponibilizar tradução para Client e Server Components sem espalhar lógica de idioma entre features.
- Incluir um seletor de idioma acessível no navbar, com atualização imediata da interface e persistência local.
- Migrar textos fixos prioritários da UI para arquivos de tradução organizados por contexto.

**Non-Goals:**
- Alterar `machado-api`, contratos HTTP, payloads do backend ou dados dinâmicos vindos da API.
- Traduzir nomes de domínio já fornecidos pelo backend, como nomes próprios de Pokémon e tipos.
- Reestruturar navegação, autenticação ou o design system além do necessário para suportar locale.
- Implementar roteamento por locale na URL, subpaths (`/en/...`) ou middleware de i18n nesta change.

## Decisions

### 1. Usar uma camada de i18n client-first com provider global e mensagens locais
Será adicionada uma biblioteca de i18n no `machado-web`, encapsulada por um provider próprio registrado no root layout junto dos providers já existentes. A configuração ficará centralizada em um módulo dedicado, com lista de locales suportados, fallback, chave de persistência e função de normalização do locale do navegador.

Racional:
- Mantém a responsabilidade de locale fora das features de domínio.
- Permite que os componentes continuem consumindo uma API simples de tradução, como hook `useTranslation` ou equivalente.
- Evita espalhar `localStorage`, `navigator.language` e mapas de fallback por vários arquivos.

Alternativas consideradas:
- `next-intl` com routing por locale: descartado porque adiciona uma camada maior de roteamento e não é necessária para o escopo atual.
- Contexto manual sem biblioteca de i18n: descartado porque aumenta o custo de pluralização, fallback e manutenção futura.

### 2. Persistir locale no navegador usando `localStorage`
A escolha manual do usuário será armazenada em uma chave explícita, como `machado-web:locale`. Na inicialização client-side, o provider resolverá o locale nessa ordem: valor persistido válido, idioma do navegador mapeado para um locale suportado, fallback `en`.

Racional:
- É a opção mais simples e coerente com o requisito de persistência local sem tocar backend.
- Evita acoplamento com cookies de autenticação já existentes.
- Mantém a decisão de locale no frontend, que é onde a interface é renderizada.

Alternativas consideradas:
- Cookie: aceitável, mas menos direto para um requisito puramente visual e exigiria coordenação adicional entre SSR e hidratação.

### 3. Isolar a hidratação de locale em um provider client específico
Como `navigator.language` e `localStorage` só existem no client, a resolução final do idioma será encapsulada em um provider client. O layout raiz continuará server-side, mas passará a envolver a árvore com esse provider para entregar estado e função de troca de idioma aos demais componentes.

Racional:
- Preserva a compatibilidade com o modelo atual de Server Components.
- Reduz o risco de hydration mismatch, concentrando a diferença entre render inicial e locale resolvido em uma única camada.
- Facilita testes unitários do fluxo de resolução e persistência.

Alternativas consideradas:
- Resolver locale diretamente em cada componente client: descartado por duplicação.
- Migrar o layout inteiro para client component: descartado por impacto arquitetural desnecessário.

### 4. Criar um `LanguageSwitcher` dedicado dentro da feature de navegação
O seletor será tratado como parte da navegação autenticada, provavelmente próximo ao `Navbar`, e não como componente do design system. Ele exibirá bandeira e rótulo/abreviação do idioma, sinalizará o item ativo e seguirá os padrões visuais já existentes da navegação.

Racional:
- O comportamento é específico do layout e depende do contexto global de locale.
- Evita contaminar o design system com um componente altamente contextual.
- Permite encaixe direto no navbar já existente sem criar uma abstração genérica desnecessária.

Alternativas consideradas:
- Componente DS genérico de select/dropdown só para idioma: descartado nesta change por não haver evidência de reutilização real fora da navegação.

### 5. Traduzir por domínio de interface e não por payload do backend
Os arquivos de locale serão organizados por áreas previsíveis como `common`, `navigation`, `auth`, `pokemon`, `errors` e `pagination`. Apenas textos fixos controlados pelo frontend serão movidos; dados vindos da API seguem sendo renderizados como recebidos.

Racional:
- Mantém a fronteira entre UI e dados de domínio.
- Reduz risco de divergência semântica entre frontend e backend.
- Facilita revisar lacunas de hardcoded text por área funcional.

## Risks / Trade-offs

- [Hydration mismatch entre locale inicial e locale resolvido no client] → Mitigar com provider client único, fallback inicial estável em `en` e atualização pós-hidratação controlada.
- [Volume alto de textos hardcoded espalhados pelo projeto] → Mitigar com rollout orientado pelos principais fluxos visíveis e checklist de busca por strings remanescentes.
- [Seletor no navbar impactar layout mobile] → Mitigar reutilizando padrões visuais já existentes da navegação e cobrindo viewport menor em testes.
- [Mensagens de erro e validação ficarem parcialmente fora do i18n] → Mitigar incluindo ações, estados visuais e mensagens de fallback no escopo de migração.
- [Nova dependência de i18n aumentar custo de bundle] → Mitigar escolhendo biblioteca enxuta e evitando carregar lógica adicional de roteamento por locale.

## Migration Plan

1. Adicionar a dependência de i18n e criar o módulo central de configuração de locale.
2. Introduzir o provider global de i18n no layout raiz sem alterar fluxos de navegação.
3. Criar os arquivos iniciais `pt-BR`, `en` e `es` com namespaces principais.
4. Implementar o `LanguageSwitcher` no navbar e integrar persistência local.
5. Migrar textos fixos prioritários de navegação, autenticação, páginas principais, estados de loading/erro/vazio e paginação.
6. Ajustar ou criar testes unitários dos componentes e do provider.
7. Validar manualmente o comportamento de locale inicial, troca de idioma, persistência e responsividade.

Rollback:
- Remover o provider e o seletor do navbar, preservando a navegação atual.
- Como não há mudança de backend nem migração de dados, o rollback é apenas de código frontend.

## Open Questions

- A biblioteca final de i18n deve ser validada contra a base atual do projeto na implementação, priorizando compatibilidade com Next.js 16 e mínimo atrito com App Router.
- Algumas mensagens podem hoje estar misturadas entre actions server-side e componentes client-side; isso deve ser mapeado na implementação para decidir quais strings ficam em camada compartilhada do frontend.
