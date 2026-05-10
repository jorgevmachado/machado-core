## Context

O Machado e um monorepo com `machado-web` em Next.js 16/React 19/TypeScript e `machado-api` em FastAPI/Python 3.13. A mudanca e transversal e deve melhorar confiabilidade e qualidade sem introduzir funcionalidades, novos contratos de API, novas telas ou alteracoes de banco.

O estado atual indicado no prompt e:

- `machado-web` precisa ampliar cobertura unitária e e2e/integracao.
- `machado-api` esta proximo de 100% de cobertura, com pendencia em `app/domain/pokemon/schema.py`.
- SonarQube local aponta muitas issues e precisa de configuracao reproduzivel com exclusoes para artefatos gerados.

## Goals / Non-Goals

**Goals:**

- Tornar a execucao de testes e cobertura reproduzivel em `machado-web` e `machado-api`.
- Provisionar ou documentar instalacao local do SonarQube/SonarScanner quando a ferramenta nao estiver disponivel no ambiente.
- Configurar analise local do SonarQube/SonarScanner para ler os relatorios de coverage relevantes.
- Criar um manual pratico para uso do Sonar no projeto, com comandos, variaveis, fluxo recomendado e interpretacao basica dos resultados.
- Excluir da analise local arquivos gerados, caches, builds, coverage e migrations.
- Corrigir issues de lint, ruff e Sonar em codigo-fonte mantido, preservando comportamento observavel.
- Priorizar testes em codigo de dominio, utilitarios, services, route handlers, server actions e componentes com comportamento.

**Non-Goals:**

- Implementar novas features.
- Alterar regras de negocio ou contratos HTTP para satisfazer cobertura.
- Criar novas tabelas, migrations ou mudancas de persistencia.
- Corrigir issues reportadas somente em artefatos gerados ou dependencias externas.
- Introduzir nova arquitetura de testes fora dos padroes ja usados por Jest, Playwright, Pytest, Ruff e ESLint.

## Decisions

1. Manter os runners existentes e ajustar configuracao quando necessario.

   `machado-web` deve continuar usando Jest/React Testing Library para testes unitarios e Playwright para e2e/integracao. `machado-api` deve continuar usando Pytest com `pytest-cov` e Ruff. Isso evita troca de ferramenta e mantem os comandos ja documentados nos AGENTS.md.

   Alternativa considerada: adicionar outro runner de testes. Rejeitada porque aumenta manutencao sem resolver o problema principal de cobertura.

2. Tratar SonarQube como ferramenta local de qualidade, nao como feature de runtime.

   A configuracao deve ficar em arquivos versionados e comandos locais, mas nenhum codigo produtivo deve depender de SonarQube em runtime. Se SonarQube/SonarScanner nao estiver instalado, a implementacao deve fornecer um caminho local reproduzivel, preferencialmente via Docker Compose ou script/documentacao de desenvolvimento. Tokens, URLs e credenciais devem continuar fora do repositorio e vir de variaveis de ambiente.

   Alternativa considerada: integrar Sonar diretamente no fluxo da aplicacao. Rejeitada porque Sonar e ferramenta de analise estatica, nao dependencia da API ou Web.

3. Documentar o uso do Sonar em um manual dedicado.

   O manual deve ficar em local versionado de documentacao do projeto e ensinar o fluxo completo: pre-requisitos, como iniciar Sonar local, como configurar token/URL por variaveis de ambiente, como gerar coverage, como executar scanner, onde ver resultados, como interpretar severidades e como lidar com falso positivo. O manual deve evitar segredos reais e usar placeholders.

   Alternativa considerada: deixar apenas comentarios em arquivos de configuracao. Rejeitada porque o objetivo inclui ensinar o uso e reduzir atrito operacional.

4. Usar exclusoes explicitas para artefatos gerados.

   A analise Sonar e coverage deve ignorar `.next/`, `node_modules/`, coverage HTML/LCOV gerado, caches, screenshots/traces de Playwright, `__pycache__/`, `.pytest_cache/`, migrations e outros outputs nao mantidos manualmente.

   Alternativa considerada: analisar o repositorio inteiro. Rejeitada porque gera ruido, issues nao acionaveis e risco de gastar tempo em arquivos gerados.

5. Corrigir Sonar por comportamento preservado e refactor pequeno.

   Issues devem ser resolvidas por testes, tipagem, reducao de duplicacao real, simplificacao local e correcoes de bug/code smell. Mudancas em codigo produtivo precisam manter os contratos existentes e devem receber testes de regressao quando houver risco.

   Alternativa considerada: silenciar issues em massa. Rejeitada porque mascara problemas reais; supressoes so devem ser usadas quando a issue for falso positivo ou houver justificativa tecnica clara.

6. Diferenciar meta de 100% de cobertura de cobertura util.

   A meta declarada e 100% para os escopos mensurados, mas o trabalho deve evitar testes que apenas exercitam linhas sem validar comportamento. Testes devem afirmar saidas, erros, chamadas de dependencia, estados de UI ou contratos relevantes.

   Alternativa considerada: cobrir linhas com smoke tests superficiais. Rejeitada porque nao reduz risco de regressao.

## Risks / Trade-offs

- 100% de cobertura pode exigir muitos testes de baixo valor em codigo glue -> Mitigacao: revisar exclusoes de coverage para arquivos gerados, barrels, tipos puros e wrappers sem comportamento, mantendo justificativa em configuracao.
- Corrigir todas as issues Sonar pode revelar mudancas grandes -> Mitigacao: aplicar correcoes em lotes pequenos, rodando lint/testes a cada bloco e mantendo refactors sem alteracao funcional.
- Sonar local pode depender de servico externo ou token -> Mitigacao: documentar execucao por variaveis de ambiente e manter credenciais fora do repositorio.
- Instalacao local do Sonar pode exigir Docker, Java ou download de scanner -> Mitigacao: detectar ferramentas existentes primeiro, preferir provisionamento isolado de desenvolvimento e pedir aprovacao antes de qualquer instalacao/download.
- Playwright pode ser sensivel a ambiente e snapshots -> Mitigacao: preservar configuracao atual de viewport, server local e tolerancias; atualizar snapshots apenas quando a UI esperada mudar.
- Ajustes para coverage podem ocultar arquivos importantes -> Mitigacao: limitar exclusoes a artefatos gerados, arquivos de configuracao sem logica de negocio e codigo explicitamente nao executavel.
