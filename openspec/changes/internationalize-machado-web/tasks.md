## 1. Setup and locale foundation

- [x] 1.1 Escolher e instalar a biblioteca de `i18n` compatível com Next.js 16/React 19 no `machado-web`
- [x] 1.2 Criar o módulo central de locale com idiomas suportados, fallback `en`, chave de persistência e normalização de idioma do navegador
- [x] 1.3 Criar a estrutura inicial dos arquivos de tradução `pt-BR`, `en` e `es` organizada por contexto de interface

## 2. Provider and runtime behavior

- [x] 2.1 Implementar o provider client de i18n responsável por resolver locale inicial, aplicar fallback e expor a API de tradução
- [x] 2.2 Integrar o provider de i18n ao `machado-web/app/layout.tsx` sem quebrar os providers globais já existentes
- [x] 2.3 Implementar a persistência local da escolha de idioma e a restauração do locale salvo em novos acessos

## 3. Navigation and UI translation

- [x] 3.1 Criar ou ajustar o `LanguageSwitcher` no navbar com opções para Brasil, Estados Unidos e Espanha
- [x] 3.2 Garantir que o seletor indique o idioma ativo, funcione por teclado e tenha nome acessível para tecnologias assistivas
- [x] 3.3 Internacionalizar textos fixos da navegação, autenticação, home e fluxos principais de listagem e detalhe
- [x] 3.4 Internacionalizar estados compartilhados de loading, erro, vazio, paginação e demais mensagens fixas prioritárias

## 4. Hardcoded text cleanup and fallback safety

- [x] 4.1 Remover textos hardcoded visíveis dos principais fluxos cobertos pela change, substituindo-os por chaves de tradução
- [x] 4.2 Garantir fallback seguro para locale inválido, locale não suportado e chaves de tradução ausentes
- [x] 4.3 Preservar a renderização de dados dinâmicos vindos da API sem traduzir valores de domínio no frontend

## 5. Validation

- [x] 5.1 Criar ou ajustar testes para resolução do locale inicial, fallback `en`, troca manual de idioma e persistência local
- [x] 5.2 Criar ou ajustar testes para o seletor de idioma no navbar, incluindo estado ativo e acessibilidade básica
- [ ] 5.3 Validar manualmente desktop e mobile nos fluxos principais para confirmar consistência visual e ausência de regressões
- [x] 5.4 Executar `yarn lint`, `yarn build` e a suíte de testes relevante do `machado-web`
