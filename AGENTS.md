# Machado Monorepo

## Project Snapshot
Monorepo com frontend Next.js 16/React 19/TypeScript 5 (`machado-web/`) e backend FastAPI/Python 3.13/SQLAlchemy async (`machado-api/`). Cada app tem seu próprio AGENTS.md detalhado.

## Root Setup Commands
```bash
# Instalar dependências de todos os apps
cd machado-web && yarn install
cd ../machado-api && poetry install
# Build/check all
cd ../machado-web && yarn build
cd ../machado-api && make lint
# Test all
cd ../machado-web && yarn test
cd ../machado-api && make test
```

## Universal Conventions
- Código limpo: Prettier, ESLint (web), Ruff (api)
- Commits: Conventional Commits
- Branches: `main` (prod), `dev` (integração), feature branches
- PR: Testes e lint obrigatórios, revisão de pelo menos 1 dev

## Security & Secrets
- Nunca commite `.env*` ou segredos
- Use variáveis de ambiente para tokens, senhas, URLs
- Dados sensíveis/PII: criptografar e mascarar logs

## JIT Index

### Package Structure
- Web UI: `machado-web/` → [veja machado-web/AGENTS.md](machado-web/AGENTS.md)
- API: `machado-api/` → [veja machado-api/AGENTS.md](machado-api/AGENTS.md)
- Docs: `docs/` (arquitetura, modelos)
- Especificações: `openspec/`

### Quick Find Commands
- Buscar função: `rg -n "def |function " machado-api/app/ machado-web/app/`
- Buscar componente: `find machado-web/app/ds -name "*.tsx"`
- Buscar rota API: `rg -n "@router" machado-api/app/domain/`

## Definition of Done
- Lint e testes verdes em ambos apps
- PR revisado e aprovado
- Documentação atualizada se necessário
