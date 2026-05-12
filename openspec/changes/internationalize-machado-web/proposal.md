## Why

O `machado-web` ainda expõe textos fixos misturados entre português e inglês, o que gera inconsistência de UX e torna a evolução da interface mais cara. A internacionalização precisa ser centralizada agora para suportar `pt-BR`, `en` e `es` com troca de idioma previsível, persistência local e baixo acoplamento à arquitetura atual do App Router.

## What Changes

- Configurar uma base de internacionalização no `machado-web` com idiomas suportados, fallback, resolução inicial de locale e persistência da escolha do usuário.
- Adicionar um provider/hook de tradução compatível com a estrutura atual de Server e Client Components, concentrando a lógica de idioma em um único ponto.
- Criar arquivos de tradução iniciais para `pt-BR`, `en` e `es`, com organização previsível por contexto de UI.
- Incluir um seletor de idioma no navbar com opções para Brasil, Estados Unidos e Espanha, acessível por teclado e consistente em desktop e mobile.
- Substituir textos fixos da interface por chaves de tradução nas páginas, componentes e estados visuais prioritários do `machado-web`.
- Garantir fallback para locale não suportado, valor persistido inválido e chaves ausentes sem alterar contratos da `machado-api`.

## Capabilities

### New Capabilities
- `web-internationalization`: internacionalização centralizada do `machado-web`, incluindo resolução de locale, persistência da preferência do usuário, seletor de idioma no navbar e consumo de traduções na UI.

### Modified Capabilities

## Impact

- Código afetado: `machado-web/app/layout.tsx`, navegação autenticada, componentes/layout do navbar, providers e páginas/componentes com textos hardcoded.
- Dependências: adição de biblioteca `i18n` compatível com Next.js 16/React 19 e arquivos de locale do frontend.
- APIs e backend: sem mudanças no `machado-api`, sem novos endpoints e sem alteração de contratos.
- Testes: ajustes e novos testes para locale inicial, fallback, persistência e troca manual de idioma.
