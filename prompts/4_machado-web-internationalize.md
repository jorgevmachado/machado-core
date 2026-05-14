# Proposta: machado-web-internationalize

## 0. Contexto Base obrigatório

Antes de gerar a proposta, considerar obrigatoriamente:

- [ARCHITECTURE-WEB](../docs/architecture-web.md)
- [AGENTS WEB](../machado-web/AGENTS.md)

As decisões devem respeitar esses documentos.
Não criar novos padrões se já existir um padrão definido no projeto.

> Observação importante: esta proposta deve considerar **somente o projeto `machado-web`**. Não alterar, criar ou refatorar nada no projeto `machado-api`.

---

## 1. Objetivo

Internacionalizar o projeto `machado-web`, padronizando os textos da interface com suporte a português, inglês e espanhol.

Adicionar um seletor de idioma no navbar usando bandeiras, permitindo que o usuário altere o idioma da aplicação, com detecção inicial baseada no idioma padrão do navegador.

---

## 2. Contexto Atual

Hoje a aplicação `machado-web` possui textos misturados entre português e inglês em vários pontos da interface.

- Projeto: `machado-web`
- Domínio afetado: interface web
- Arquivos/pastas relevantes:
  - `machado-web/app`
  - `machado-web/app/ui`
  - `machado-web/app/ui/components`
  - `machado-web/app/ui/features`
  - `machado-web/app/layout.tsx`
  - `machado-web/app/providers`, caso exista
  - componente atual de `Navbar`, `Header` ou equivalente
- Comportamento atual: textos hardcoded e mistura de idiomas na interface
- Problema identificado: ausência de uma estratégia centralizada de internacionalização

---

## 3. Comportamento Esperado

Após a mudança:

- O sistema deve utilizar a biblioteca `i18n` para controlar os textos da interface.
- O sistema deve suportar inicialmente os idiomas:
  - Português do Brasil (`pt-BR`)
  - Inglês (`en`)
  - Espanhol (`es`)
- O sistema deve usar como idioma inicial o idioma padrão do navegador do usuário, quando suportado.
- Caso o idioma do navegador não seja suportado, o sistema deve usar português do Brasil como fallback.
- O usuário deve conseguir alterar o idioma manualmente pelo navbar.
- A escolha manual do usuário deve ser persistida localmente, para que o idioma escolhido seja mantido em novos acessos.
- A interface deve exibir um seletor de idioma com bandeiras do Brasil, Estados Unidos e Espanha.
- Todas as páginas e componentes do `machado-web` devem renderizar textos a partir dos arquivos de tradução.
- O projeto deve ficar preparado para receber novos idiomas no futuro com baixo impacto.

---

## 4. Escopo

### Dentro do escopo

- Configurar a biblioteca `i18n` no projeto `machado-web`.
- Criar estrutura centralizada para arquivos de tradução.
- Criar arquivos iniciais de tradução para:
  - `pt-BR`
  - `en`
  - `es`
- Criar ou ajustar provider necessário para disponibilizar o idioma na aplicação.
- Adicionar seletor de idioma no navbar com bandeiras.
- Detectar idioma padrão do navegador quando o usuário ainda não tiver escolhido um idioma manualmente.
- Persistir a escolha de idioma do usuário no navegador.
- Substituir textos hardcoded da interface por chaves de tradução.
- Padronizar textos atualmente misturados entre português e inglês.
- Garantir fallback para textos não encontrados.

### Fora do escopo

- Não alterar o projeto `machado-api`.
- Não criar endpoints na API.
- Não alterar contratos de API.
- Não modificar regras de negócio do backend.
- Não implementar novas features de domínio.
- Não alterar layout geral das páginas além do necessário para incluir o seletor de idioma.
- Não internacionalizar dados vindos da API, salvo textos fixos da interface ao redor desses dados.
- Não traduzir nomes próprios, nomes de Pokémon, tipos vindos da API ou dados de domínio que já venham prontos do backend, a menos que já exista padrão para isso no projeto.

---

## 5. Regras de Negócio

### Regra 1 — Idioma inicial baseado no navegador

**Dado que:** o usuário acessa o `machado-web` pela primeira vez e ainda não possui idioma salvo localmente.  
**Quando:** a aplicação é carregada.  
**Então:** o sistema deve tentar usar o idioma padrão do navegador, desde que seja um idioma suportado.

---

### Regra 2 — Fallback para português

**Dado que:** o idioma padrão do navegador não é suportado pela aplicação.  
**Quando:** a aplicação é carregada.  
**Então:** o sistema deve usar `en` como idioma padrão.

---

### Regra 3 — Seleção manual de idioma

**Dado que:** o usuário está navegando no sistema.  
**Quando:** ele seleciona uma bandeira no navbar.  
**Então:** os textos da interface devem ser atualizados para o idioma selecionado.

---

### Regra 4 — Persistência da escolha do usuário

**Dado que:** o usuário selecionou manualmente um idioma.  
**Quando:** ele recarrega a página ou acessa o sistema novamente.  
**Então:** o sistema deve manter o idioma escolhido anteriormente.

---

### Regra 5 — Idiomas disponíveis no seletor

**Dado que:** o usuário visualiza o navbar.  
**Quando:** o seletor de idioma é exibido.  
**Então:** devem estar disponíveis as opções com bandeiras para Brasil, Estados Unidos e Espanha.

---

### Regra 6 — Textos hardcoded

**Dado que:** existem textos fixos escritos diretamente nos componentes.  
**Quando:** a internacionalização for implementada.  
**Então:** esses textos devem ser movidos para arquivos de tradução e consumidos via chave i18n.

---

## 6. Escopo Técnico

### 6.1 API

Não haverá alteração no projeto `machado-api`.

Esta proposta deve ser limitada ao `machado-web`.

Não criar:

- novos endpoints;
- novas tabelas;
- migrations;
- alterações de schema;
- regras de negócio no backend;
- cache no backend;
- novas integrações externas.

Caso algum texto venha da API, o frontend deve apenas renderizar o valor recebido, mantendo o contrato atual.

---

### 6.2 WEB

#### 6.2.1 Biblioteca e configuração

Implementar internacionalização usando a ferramenta `i18n`.

A implementação deve respeitar a estrutura atual do `machado-web`, especialmente:

- padrões definidos em `ARCHITECTURE-WEB`;
- orientações do `AGENTS WEB`;
- organização atual de `app`, `ui`, `components`, `features` e `providers`, caso existam.

Criar uma configuração centralizada para:

- idiomas suportados;
- idioma padrão;
- fallback;
- carregamento dos arquivos de tradução;
- função/hook de tradução;
- persistência da escolha do idioma.

Sugestão de estrutura, podendo ser ajustada conforme a arquitetura real do projeto:

```txt
machado-web/
  app/
    i18n/
      config.ts
      locales/
        pt-BR.json
        en.json
        es.json
    providers/
      I18nProvider.tsx
```

Caso o projeto já possua uma pasta ou padrão para providers, configuração global ou libs compartilhadas, reutilizar o padrão existente em vez de criar uma estrutura paralela.

---

#### 6.2.2 Páginas

Todas as páginas do `machado-web` devem ter seus textos fixos preparados para internacionalização.

Priorizar inicialmente:

- página inicial;
- páginas de listagem;
- páginas de detalhe;
- páginas de erro, vazio e loading;
- textos de navegação;
- botões;
- labels;
- títulos;
- descrições;
- mensagens auxiliares.

O fluxo de navegação não deve ser alterado.

---

#### 6.2.3 Componentes

Criar ou ajustar o componente de seletor de idioma no navbar.

O componente deve:

- exibir as opções de idioma com bandeiras;
- permitir seleção de português, inglês e espanhol;
- refletir visualmente o idioma selecionado;
- ser acessível por teclado;
- possuir label ou aria-label adequado;
- manter boa apresentação em desktop e mobile;
- usar padrões visuais já existentes no projeto.

Sugestão de componente, ajustando nomes e caminhos conforme o projeto:

```txt
machado-web/app/ui/components/language-switcher/LanguageSwitcher.tsx
```

Ou, caso o projeto organize componentes por feature/layout:

```txt
machado-web/app/ui/features/layout/language-switcher/LanguageSwitcher.tsx
```

Não duplicar componente se já existir um padrão de seleção, dropdown ou menu no projeto.

---

#### 6.2.4 Renderização

Os textos devem ser renderizados por chave de tradução.

Exemplo conceitual:

```tsx
const { t } = useTranslation();

return <h1>{t('home.title')}</h1>;
```

Exemplo de organização de chaves:

```json
{
  "common": {
    "loading": "Carregando...",
    "error": "Ocorreu um erro",
    "empty": "Nenhum registro encontrado"
  },
  "navigation": {
    "home": "Início",
    "pokemon": "Pokémon",
    "pokedex": "Pokédex"
  },
  "language": {
    "label": "Selecionar idioma",
    "ptBR": "Português",
    "en": "Inglês",
    "es": "Espanhol"
  }
}
```

As chaves devem ser organizadas de forma previsível, evitando nomes genéricos demais.

Preferir agrupamento por contexto:

- `common`
- `navigation`
- `language`
- `home`
- `pokemon`
- `pokedex`
- `errors`
- `pagination`
- `forms`, se existir

---

#### 6.2.5 Interações

O usuário deve conseguir alterar o idioma pelo navbar.

Ao alterar o idioma:

1. A aplicação deve atualizar os textos imediatamente.
2. O idioma selecionado deve ser salvo localmente.
3. O seletor deve indicar qual idioma está ativo.
4. A alteração não deve recarregar a página inteira, salvo se a biblioteca escolhida exigir e isso estiver alinhado à arquitetura do projeto.

---

#### 6.2.6 Estados da interface

Os estados também devem ser internacionalizados:

- loading;
- erro;
- vazio;
- sucesso;
- validações visuais;
- mensagens de fallback;
- textos de botões;
- textos de paginação.

Exemplos:

```txt
Carregando...
Nenhum Pokémon encontrado.
Erro ao carregar os dados.
Tentar novamente.
Próxima página.
Página anterior.
```

Todos esses textos devem sair dos arquivos de tradução.

---

#### 6.2.7 Persistência do idioma

Persistir a escolha do usuário no navegador.

Preferencialmente usar uma abordagem simples e compatível com o projeto, como:

- `localStorage`; ou
- cookie, caso o projeto já utilize estratégia baseada em cookies.

A chave de persistência deve ser clara, por exemplo:

```txt
machado-web:locale
```

A prioridade de definição do idioma deve ser:

1. idioma salvo pelo usuário;
2. idioma padrão do navegador, se suportado;
3. fallback `pt-BR`.

---

#### 6.2.8 Acessibilidade

O seletor de idioma deve:

- possuir `aria-label` ou label visível;
- permitir navegação por teclado;
- não depender exclusivamente da bandeira para comunicar o idioma;
- exibir também o nome ou abreviação do idioma, quando necessário;
- manter contraste e legibilidade.

As bandeiras podem ser usadas como apoio visual, mas não devem ser a única forma de identificação do idioma.

---

#### 6.2.9 Performance e organização

A implementação deve:

- evitar duplicação de arquivos de tradução;
- evitar espalhar lógica de idioma por vários componentes;
- centralizar configuração e tipos relacionados a idioma;
- manter os arquivos de tradução organizados;
- evitar re-renderizações desnecessárias;
- manter compatibilidade com Server Components e Client Components do Next.js, conforme a estrutura atual do projeto.

Caso seja necessário usar hooks ou estado de idioma no client-side, isolar essa responsabilidade em um provider ou componente client específico.

---

## 7. Restrições Arquiteturais

- Não criar nova arquitetura.
- Não alterar o projeto `machado-api`.
- Não duplicar lógica entre componentes.
- Não espalhar configurações de idioma em múltiplos pontos da aplicação.
- Reutilizar padrões existentes do `machado-web`.
- Minimizar impacto no código atual.
- Não implementar novas features além da internacionalização.
- Não alterar contratos de API.
- Não traduzir dados dinâmicos vindos da API, salvo quando já houver padrão definido para isso no projeto.
- Não criar tabelas, migrations ou alterações de banco de dados.
- Não criar lógica de internacionalização no backend.
- Respeitar a estrutura atual de componentes, providers, hooks e features.
- Não remover textos sem garantir chave correspondente nos arquivos de tradução.
- Não deixar textos visíveis hardcoded quando forem textos fixos da interface.

---

## 8. Critérios de Aceite

- [ ] O projeto `machado-web` possui configuração funcional de `i18n`.
- [ ] O projeto suporta `pt-BR`, `en` e `es`.
- [ ] O idioma inicial considera o idioma padrão do navegador.
- [ ] O fallback padrão é `pt-BR`.
- [ ] O usuário consegue alterar o idioma pelo navbar.
- [ ] O seletor de idioma exibe bandeiras do Brasil, Estados Unidos e Espanha.
- [ ] O seletor indica corretamente o idioma ativo.
- [ ] A escolha do idioma é persistida no navegador.
- [ ] Ao recarregar a página, o idioma escolhido anteriormente é mantido.
- [ ] Textos fixos da interface foram movidos para arquivos de tradução.
- [ ] Estados de loading, erro, vazio e sucesso foram internacionalizados.
- [ ] Botões, labels, títulos e textos de navegação foram internacionalizados.
- [ ] A implementação respeita a arquitetura atual do `machado-web`.
- [ ] Nenhuma alteração foi feita no `machado-api`.
- [ ] Não houve alteração de contrato de API.
- [ ] A UI continua consistente em desktop e mobile.
- [ ] O seletor de idioma possui acessibilidade básica.
- [ ] Não foram introduzidas novas features fora do escopo.
- [ ] Não existem textos fixos visíveis esquecidos nos principais fluxos da interface.
- [ ] O projeto continua executando sem regressões.

---

## 9. Plano de Validação

### Validação manual

1. Abrir o `machado-web` sem idioma salvo no navegador.
2. Verificar se o sistema tenta usar o idioma padrão do navegador.
3. Caso o idioma do navegador não seja suportado, verificar se o fallback é `pt-BR`.
4. Acessar o navbar.
5. Selecionar português pelo seletor de bandeiras.
6. Verificar se os textos aparecem em português.
7. Selecionar inglês pelo seletor de bandeiras.
8. Verificar se os textos aparecem em inglês.
9. Selecionar espanhol pelo seletor de bandeiras.
10. Verificar se os textos aparecem em espanhol.
11. Recarregar a página.
12. Confirmar que o idioma escolhido foi mantido.
13. Navegar pelas principais páginas.
14. Verificar se títulos, botões, labels, mensagens e estados foram traduzidos.
15. Testar o seletor em tela pequena/mobile.
16. Testar navegação por teclado no seletor de idioma.

### Testes automatizados

Criar ou ajustar testes para validar:

- renderização com idioma padrão;
- fallback para `pt-BR`;
- troca manual de idioma;
- persistência do idioma selecionado;
- renderização correta de textos traduzidos;
- presença das opções de idioma no navbar;
- comportamento acessível básico do seletor.

### Edge cases

Validar os seguintes cenários:

- idioma salvo inválido;
- idioma do navegador não suportado;
- chave de tradução inexistente;
- arquivo de tradução incompleto;
- renderização inicial antes da hidratação client-side;
- uso do seletor em mobile;
- usuário alternando idiomas rapidamente.

---

## 10. Saída Esperada do OpenSpec

Gerar uma proposta contendo:

1. Resumo da mudança
2. Problema identificado
3. Solução proposta
4. Arquivos impactados
5. Plano de implementação
6. Alterações de modelo
7. Alterações de API
8. Alterações de UI
9. Estratégia de testes
10. Riscos e pontos de atenção

A proposta gerada pelo OpenSpec deve deixar explícito que:

- a alteração é exclusiva do `machado-web`;
- o `machado-api` não deve ser alterado;
- a internacionalização deve usar `i18n`;
- os idiomas iniciais são português, inglês e espanhol;
- o seletor de idioma deve ficar no navbar;
- a escolha do usuário deve ser persistida;
- o idioma inicial deve considerar o idioma do navegador;
- `pt-BR` deve ser o fallback padrão.
