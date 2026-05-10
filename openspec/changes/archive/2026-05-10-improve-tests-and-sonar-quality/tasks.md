## 1. Baseline e Configuracao

- [x] 1.1 Executar baseline de `machado-api` com `make lint` e `make test`, registrando arquivos sem cobertura e falhas atuais.
- [x] 1.2 Executar baseline de `machado-web` com `yarn lint`, `yarn build`, `yarn test --coverage` e Playwright, registrando arquivos/fluxos sem cobertura e falhas atuais.
- [x] 1.3 Ajustar scripts/configuracoes de coverage do `machado-web` para emitir relatorio consumivel pelo Sonar e aplicar thresholds de 100% no escopo mensurado.
- [x] 1.4 Ajustar configuracoes de coverage do `machado-api` para emitir relatorio consumivel pelo Sonar e aplicar thresholds de 100% no escopo mensurado.
- [x] 1.5 Verificar se SonarQube/SonarScanner ja estao disponiveis localmente e registrar a estrategia de execucao.
- [x] 1.6 Instalar, provisionar ou documentar SonarQube/SonarScanner local quando ausente, preferencialmente via Docker Compose ou ferramenta de desenvolvimento isolada, sem commitar segredos.
- [x] 1.7 Criar ou ajustar configuracao local do SonarQube/SonarScanner para analisar API e Web sem commitar segredos.
- [x] 1.8 Configurar exclusoes Sonar/coverage para arquivos gerados, dependencias, caches, builds, coverage, Playwright artifacts e migrations.

## 2. Cobertura da API

- [x] 2.1 Adicionar testes unitarios para `machado-api/app/domain/pokemon/schema.py`, cobrindo validacao, serializacao e defaults aplicaveis.
- [x] 2.2 Revisar lacunas restantes de coverage em `machado-api/app/` e adicionar testes focados em comportamento.
- [x] 2.3 Corrigir falhas de Ruff/lint introduzidas ou reveladas pelos testes da API.
- [x] 2.4 Confirmar `make test` com 100% de coverage no escopo mensurado da API.

## 3. Cobertura Unitaria do Web

- [x] 3.1 Adicionar testes para utilitarios e bibliotecas compartilhadas do `machado-web` com asserts de entradas, saidas e erros.
- [x] 3.2 Adicionar testes para services, route handlers e server actions do `machado-web`, mockando dependencias externas conforme padrao existente.
- [x] 3.3 Adicionar testes para hooks e componentes do design system com comportamento, estados e acessibilidade observavel.
- [x] 3.4 Adicionar testes para features de UI Pokemon, Pokedex, My Pokemon, Trainer, Auth e Navigation conforme lacunas de coverage.
- [x] 3.5 Confirmar `yarn test --coverage` com 100% de statements, branches, functions e lines no escopo mensurado do Web.

## 4. Cobertura E2E/Integracao do Web

- [x] 4.1 Mapear fluxos publicos e autenticados existentes que devem ser cobertos por Playwright sem criar novas features.
- [x] 4.2 Adicionar ou ajustar testes Playwright para navegacao, login/registro, listagens, detalhes e fluxos Pokemon existentes.
- [x] 4.3 Garantir que testes Playwright sejam estaveis nos projetos desktop e mobile configurados.
- [x] 4.4 Confirmar execucao completa da suite e2e/integracao do Web com o comando documentado.

## 5. SonarQube e Qualidade

- [x] 5.1 Executar analise local do SonarQube/SonarScanner usando variaveis de ambiente para URL/token.
- [x] 5.2 Corrigir bugs, vulnerabilidades e code smells aplicaveis em codigo-fonte mantido da API.
- [x] 5.3 Corrigir bugs, vulnerabilidades e code smells aplicaveis em codigo-fonte mantido do Web.
- [x] 5.4 Documentar falsos positivos ou supressoes estritamente necessarias com justificativa tecnica.
- [x] 5.5 Reexecutar Sonar local e confirmar ausencia de issues aplicaveis no escopo analisado.

## 6. Validacao Final

- [x] 6.1 Executar `make lint` e `make test` em `machado-api`.
- [x] 6.2 Executar `yarn lint`, `yarn build`, `yarn test --coverage` e suite Playwright em `machado-web`.
- [x] 6.3 Conferir que relatorios de coverage e Sonar ignoram artefatos gerados e migrations.
- [x] 6.4 Criar manual versionado de uso do Sonar no projeto, cobrindo pre-requisitos, instalacao/provisionamento local, variaveis de ambiente, geracao de coverage, execucao do scanner, leitura dos resultados e troubleshooting.
- [x] 6.5 Atualizar documentacao local de comandos de Sonar/testes se novos scripts ou variaveis forem adicionados.
