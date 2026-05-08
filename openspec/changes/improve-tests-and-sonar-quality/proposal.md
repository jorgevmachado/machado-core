## Why

O monorepo Machado precisa elevar a confiabilidade antes de novas evolucoes: a cobertura atual do `machado-web` e o ponto pendente em `machado-api/app/domain/pokemon/schema.py` deixam regressao funcional e de contrato mais provavel. A analise local via SonarQube tambem aponta um volume alto de issues, exigindo um fluxo reproduzivel para identificar, priorizar e corrigir problemas de qualidade sem alterar comportamento de negocio.

## What Changes

- Adicionar e ajustar testes unitarios no `machado-web` ate atingir 100% de cobertura para o escopo mensurado pelo projeto.
- Adicionar e ajustar testes de integracao/e2e no `machado-web` ate atingir 100% de cobertura para fluxos cobertos pela suite.
- Adicionar testes unitarios no `machado-api`, com foco inicial em `app/domain/pokemon/schema.py`, ate atingir 100% de cobertura para o escopo mensurado pelo projeto.
- Instalar ou provisionar localmente o SonarQube/SonarScanner quando ausente no ambiente, preferencialmente por configuracao reproduzivel e documentada, sem commitar segredos.
- Configurar execucao local do SonarQube/SonarScanner para o monorepo, excluindo artefatos gerados como coverage, caches, builds e migrations.
- Criar um manual de uso do Sonar no projeto, cobrindo instalacao/provisionamento, configuracao de variaveis, execucao, leitura dos resultados e troubleshooting.
- Corrigir bugs, vulnerabilidades e code smells reportados pelo SonarQube que sejam aplicaveis ao codigo-fonte mantido no repositorio.
- Ajustar lint/ruff no `machado-api` e lint/build/test no `machado-web` conforme os padroes existentes.
- Nao implementar novas features, nao alterar contratos HTTP por requisito funcional e nao criar novas tabelas ou migrations.

## Capabilities

### New Capabilities

- `quality-gates-and-coverage`: Define requisitos transversais para cobertura de testes, execucao local do SonarQube, exclusoes de arquivos gerados e criterios de qualidade para API e Web.

### Modified Capabilities

- Nenhuma. As capacidades existentes de dominio Pokemon nao terao requisitos funcionais alterados.

## Impact

- Codigo afetado: testes e eventuais refactors sem mudanca comportamental em `machado-web/` e `machado-api/`.
- Configuracao afetada: arquivos de configuracao de testes, lint, coverage e SonarQube/SonarScanner, incluindo provisionamento local e manual de uso quando necessario.
- Validacao afetada: `yarn test --coverage`, `yarn test:e2e`, `yarn lint`, `yarn build`, `make test`, `make lint` e analise local do SonarQube.
- APIs/UI: sem novos endpoints, telas ou regras de negocio; qualquer ajuste em codigo produtivo deve preservar contratos e comportamento observavel.
- Dependencias: evitar novas dependencias de runtime; adicionar ou documentar ferramentas locais de desenvolvimento somente se indispensavel para executar Sonar ou testes no padrao do projeto.
