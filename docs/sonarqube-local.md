# SonarQube local

Este manual explica como usar SonarQube/SonarScanner no monorepo Machado para analisar `machado-api` e `machado-web` com os relatorios de cobertura locais.

## Pre-requisitos

- Docker e Docker Compose.
- Dependencias da API instaladas com `poetry install`.
- Dependencias do Web instaladas com `yarn install`.
- Um token local do SonarQube. Nunca commite tokens, senhas ou URLs privadas.

## Subir o SonarQube

Na raiz do repositorio:

```bash
docker compose -f docker-compose.sonar.yml up -d
```

Acesse `http://localhost:9000`. No primeiro acesso, use o login padrao `admin` / `admin` e troque a senha quando solicitado.

Crie um projeto local com a chave:

```text
machado
```

Depois gere um token de usuario ou de projeto e exporte em uma variavel de ambiente:

```bash
export SONAR_HOST_URL="http://localhost:9000"
export SONAR_TOKEN="<seu-token-local>"
```

## Gerar coverage

API:

```bash
cd machado-api
make test
```

Esse comando gera `machado-api/coverage.xml`, que e lido pelo Sonar.

Web:

```bash
cd machado-web
yarn test:coverage
```

Esse comando gera `machado-web/coverage/lcov.info`, que e lido pelo Sonar.

## Executar o scanner

Se voce ja tiver `sonar-scanner` instalado:

```bash
sonar-scanner \
  -Dsonar.host.url="$SONAR_HOST_URL" \
  -Dsonar.token="$SONAR_TOKEN"
```

Se nao tiver scanner instalado, use o container oficial:

```bash
docker run --rm \
  --network host \
  -e SONAR_HOST_URL="$SONAR_HOST_URL" \
  -e SONAR_TOKEN="$SONAR_TOKEN" \
  -v "$PWD:/usr/src" \
  sonarsource/sonar-scanner-cli
```

O scanner usa `sonar-project.properties` na raiz do repositorio.

## Ler os resultados

Abra `http://localhost:9000/dashboard?id=machado`.

Priorize nesta ordem:

1. Bugs.
2. Vulnerabilidades.
3. Security hotspots.
4. Code smells.
5. Duplicacoes e coverage.

Para cada issue, corrija com a menor mudanca que preserve comportamento. Quando a correcao altera codigo produtivo, adicione ou ajuste teste de regressao.

## Falsos positivos e supressoes

Use supressao somente quando houver justificativa tecnica clara. Prefira:

- Marcar como falso positivo no SonarQube local quando a issue nao for real.
- Adicionar comentario curto no codigo apenas quando o motivo nao for obvio.
- Limitar qualquer supressao ao menor trecho possivel.

Nao use supressoes para esconder falhas corrigiveis de lint, tipo, teste ou seguranca.

## Arquivos ignorados

A configuracao ignora artefatos gerados e nao mantidos manualmente, incluindo:

- `.next/`
- `node_modules/`
- `coverage/`
- `htmlcov/`
- caches de pytest/ruff/Python
- screenshots e reports do Playwright
- `migrations/`

## Troubleshooting

Se o SonarQube encerrar logo na inicializacao com erro de Elasticsearch sobre `vm.max_map_count`, o host precisa permitir o valor minimo exigido pelo Docker/SonarQube. Em ambientes Linux, isso normalmente requer permissao de administrador para ajustar o `sysctl` do host:

```bash
sudo sysctl -w vm.max_map_count=262144
```

Se o comando acima falhar com permissao negada, o ambiente nao permite subir o SonarQube localmente sem ajuda do administrador ou sem mover a execucao para um host controlado por voce.

Se `http://localhost:9000` nao abrir, verifique os containers:

```bash
docker compose -f docker-compose.sonar.yml ps
docker compose -f docker-compose.sonar.yml logs -f sonarqube
```

Se o scanner falhar por autenticacao, gere um novo token e atualize `SONAR_TOKEN`.

Se o scanner nao encontrar coverage, rode novamente `make test` em `machado-api` e `yarn test:coverage` em `machado-web` antes da analise.

Para parar o Sonar local:

```bash
docker compose -f docker-compose.sonar.yml down
```
