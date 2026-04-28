# Architecture

## mach-api

### Overview

**Stack principal:**
- Python 3.13 + FastAPI (async)
- SQLAlchemy 2.x async + PostgreSQL (driver: psycopg)
- Pydantic v2 (schemas/DTOs)
- Alembic (migrações de banco)
- Redis (cache)
- PyJWT + pwdlib[argon2] (autenticação JWT)
- fastapi-pagination (paginação)
- Poetry (gerenciador de dependências)

---

## ESTRUTURA DE PASTAS — EXPLICAÇÃO COMPLETA

```
mach-api/
├── app/
│   ├── main.py                     # Entry point: cria o FastAPI, registra routers
│   ├── core/                       # Infraestrutura compartilhada (não-domínio)
│   │   ├── settings.py             # Configurações via pydantic-settings (.env)
│   │   ├── database/
│   │   │   ├── base.py             # table_registry (SQLAlchemy registry)
│   │   │   └── database.py         # engine async + get_session (dependency)
│   │   ├── security/
│   │   │   └── security.py         # JWT: create_access_token, get_current_user, hash/verify
│   │   ├── cache/
│   │   │   ├── redis.py            # Conexão Redis async
│   │   │   ├── manager.py          # CacheManager: build_key, get_cache, set_cache
│   │   │   └── service.py          # CacheService: get/set list e single item com serialização
│   │   ├── repository/
│   │   │   └── base.py             # BaseRepository: list_all, find_by, save, update + paginação
│   │   ├── service/
│   │   │   └── base.py             # BaseService: list_all, find_one, find_one_cached, update
│   │   ├── exceptions/
│   │   │   └── exceptions.py       # handle_service_exception, AppHTTPException
│   │   ├── logging/
│   │   │   ├── logging.py          # configure_logging, log_service_success, log_service_exception
│   │   │   ├── middleware.py        # Middleware de request logging
│   │   │   └── schemas.py          # LoggingParams (dataclass para passar logger+service+operation)
│   │   ├── pagination/
│   │   │   ├── pagination.py       # get_limit_offset_params, exception_pagination, is_paginate
│   │   │   └── schemas.py          # CustomLimitOffsetPage
│   │   └── context/
│   │       └── request_context.py  # ContextVar request_id_ctx (rastreio por request)
│   ├── domain/                     # Cada subpasta = 1 domínio de negócio
│   │   └── <domain>/
│   │       ├── schema.py           # Pydantic DTOs: input, output, filtros
│   │       ├── repository.py       # Queries SQLAlchemy; herda BaseRepository
│   │       ├── service.py          # Orquestração de negócio; herda BaseService
│   │       ├── business.py         # Regras puras de domínio (sem infra)
│   │       └── route.py            # FastAPI router: endpoints, response_model, Depends
│   ├── models/                     # Entidades ORM (SQLAlchemy dataclass-mapped)
│   │   ├── enums.py                # Todos os Enums do sistema
│   │   ├── associations.py         # Tabelas de associação M2M
│   │   ├── user.py                 # Model User
│   │   ├── trainer.py              # Model Trainer
│   │   └── ...                     # Demais models
│   ├── shared/
│   │   ├── schemas.py              # FilterPage, Message (schemas base reutilizáveis)
│   │   └── utils/                  # Utilitários (ex: is_valid_uuid)
│   └── infrastructure/
│       └── external_api/
│           ├── pokeapi_client.py   # Client HTTP para PokeAPI (httpx async)
│           └── schemas.py          # Schemas da API externa
├── migrations/                     # Alembic: env.py + versions/
├── tests/                          # Espelha app/: tests/app/domain/<domain>/
├── pyproject.toml                  # Dependências e configurações (Poetry)
├── Makefile                        # Comandos: dev, test, lint, migrate
├── alembic.ini                     # Configuração do Alembic
├── Dockerfile
└── docker-compose.yml
```

---

## ARQUIVO POR ARQUIVO — PAPEL E REGRAS

### `app/main.py`
- Instancia `FastAPI()`
- Chama `configure_logging()` antes de tudo
- Registra cada router com `app.include_router(router, prefix='/...', tags=[...])`
- Chama `add_pagination(app)` do fastapi-pagination
- Define a rota raiz `GET /` que retorna `{"message": "Hello World!"}`

### `app/core/settings.py`
- Herda `BaseSettings` do pydantic-settings
- Lê `.env` e `.env.local` automaticamente
- Campos obrigatórios: `SECRET_KEY`, `DATABASE_URL`, `ALGORITHM`, `REDIS_HOST`, `REDIS_PORT`, `ACCESS_TOKEN_EXPIRE_MINUTES`
- Campos opcionais: `REDIS_URL`, `REDIS_CACHE_TTL_SECONDS` (default 3600), `POKEAPI_BASE_URL`
- **Nunca hardcode** valores; sempre via variável de ambiente

### `app/core/database/base.py`
- Define `table_registry = registry()` (SQLAlchemy)
- Define `default_lazy = 'selectin'` (lazy loading padrão dos relacionamentos)
- Todos os models usam `@table_registry.mapped_as_dataclass`

### `app/core/database/database.py`
- Cria `engine = create_async_engine(Settings().DATABASE_URL)`
- Define `get_session()`: gerador async que fornece `AsyncSession` para injeção de dependência

### `app/core/security/security.py`
- `get_password_hash(password)` → hash argon2 via pwdlib
- `verify_password(plain, hashed)` → bool
- `create_access_token(data: dict)` → JWT string com expiração de `ACCESS_TOKEN_EXPIRE_MINUTES`
- `get_current_user(session, token)` → dependency async que decodifica JWT e retorna `User`; lança 401 se inválido

### `app/core/cache/redis.py`
- Define `redis_client` async (aioredis)

### `app/core/cache/manager.py` — `CacheManager`
- `build_key(prefix, *parts)` → string de cache normalizada e determinística
- `get_cache(key)` → `dict | None` (deserializa JSON do Redis)
- `set_cache(key, value, ttl)` → serializa JSON e salva no Redis com TTL

### `app/core/cache/service.py` — `CacheService`
- Inicializado com `alias`, `prefix`, `logger_params`, `schema_class`
- `build_key_list(page_filter)` → chave para lista
- `build_key_one(param)` → chave para item único
- `get_list(key)` / `set_list(key, data)` → cache de listas, listas paginadas e custom-paginadas
- `get_one(key)` / `set_one(key, data)` → cache de item único
- Deserializa usando `schema_class.model_validate()`

### `app/core/repository/base.py` — `BaseRepository[ModelT]`
- Atributos de classe: `model`, `relations` (tuple de `selectinload`), `default_order_by`
- `list_all(page_filter)` → filtra, ordena, pagina; suporta `CustomLimitOffsetPage`
- `find_by(**kwargs)` → busca por colunas; suporta `pokemon_name` como filtro especial
- `save(entity)` → add + commit + refresh
- `update(entity)` → merge + commit + refresh
- `total()` → count total

### `app/core/service/base.py` — `BaseService[RepositoryT, ModelT]`
- Inicializado com: `alias`, `repository`, `logger_params`, `schema_class`, `cache_prefix`
- `list_all(page_filter, user_request, trainer_id)` → sem cache; captura exceções com `handle_service_exception`
- `list_all_cached(...)` → verifica Redis antes de chamar `list_all`
- `find_one(param)` → busca por UUID (id) ou string (name); lança 404 se não encontrado
- `find_one_cached(param)` → verifica Redis antes de `find_one`
- `find_by(**kwargs)` → delega ao repository
- `update(param, update_schema)` → busca + aplica `model_dump(exclude_unset=True)` + salva

### `app/core/exceptions/exceptions.py`
- `AppHTTPException(HTTPException)` → exceção base da aplicação
- `UnauthorizedException` → 401 padrão
- `handle_service_exception(exception, *, logger, service, operation, raise_exception, user_request)`:
  - Mapeia tipo de exceção → `HTTPStatus`
  - Loga com `log_service_exception`
  - Se `raise_exception=True` → lança `AppHTTPException`
  - Se `raise_exception=False` → retorna `(status_code, message)`

### `app/core/logging/logging.py`
- `configure_logging()` → configura handlers/formatters uma única vez
- `log_service_success(logger_params, ...)` → `logger.info` com campos extras estruturados
- `log_service_exception(logger_params, ...)` → `logger.error` ou `logger.warning` conforme status

### `app/core/logging/schemas.py` — `LoggingParams`
- Dataclass com: `logger`, `service`, `operation`, `message`, `status_code`, `user_request`
- Passado para `BaseService` e usado em todos os logs estruturados

### `app/shared/schemas.py`
- `Message` → `{ "message": str }` (resposta genérica)
- `FilterPage` → base de paginação/filtro: `page`, `offset`, `limit`, `order_by`
  - `FilterPage.build(page_filter, **extras)` → constrói dinamicamente com campos extras
  - `with_updates(**updates)` → cria cópia com novos valores

---

## COMO IMPLEMENTAR UM DOMÍNIO — PASSO A PASSO

### 1. Model ORM (`app/models/<entity>.py`)

```python
from __future__ import annotations
from datetime import datetime, timezone
from uuid import UUID, uuid4
from sqlalchemy import String, DateTime
from sqlalchemy.orm import Mapped, mapped_column, relationship
from app.core.database.base import table_registry

def _utcnow() -> datetime:
    return datetime.now(timezone.utc)

@table_registry.mapped_as_dataclass
class MyEntity:
    __tablename__ = 'my_entities'

    # Campos obrigatórios (sem default) — primeiro no __init__
    name: Mapped[str] = mapped_column(String, nullable=False)

    # Campos com default
    status: Mapped[str] = mapped_column(String, default='active')

    # Auto-gerados — init=False
    id: Mapped[UUID] = mapped_column(primary_key=True, default_factory=uuid4, init=False)
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), default_factory=_utcnow, init=False
    )
```

**Regras do model:**
- Sempre usar `@table_registry.mapped_as_dataclass`
- Campos obrigatórios (sem default) ANTES dos opcionais
- Campos `id`, `created_at`, `updated_at`, `deleted_at` com `init=False`
- Relacionamentos com `lazy='selectin'` e `init=False`
- Enums definidos em `app/models/enums.py`

### 2. Schema Pydantic (`app/domain/<domain>/schema.py`)

```python
from uuid import UUID
from datetime import datetime
from pydantic import BaseModel, ConfigDict

# Schema de entrada (criação)
class MyEntityCreateSchema(BaseModel):
    name: str

# Schema de saída (resposta)
class MyEntitySchema(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: UUID
    name: str
    created_at: datetime

# Schema de filtro/paginação (herda FilterPage)
class MyEntityFilterPageSchema(FilterPage):
    name: str | None = None
    status: str | None = None
```

**Regras do schema:**
- Schemas de saída SEMPRE têm `model_config = ConfigDict(from_attributes=True)`
- Schemas de entrada NÃO têm `from_attributes`
- Use `EmailStr` do pydantic para emails
- Validators com `@field_validator` quando necessário
- Nunca acoplar schema ao ORM (sem imports de models)

### 3. Repository (`app/domain/<domain>/repository.py`)

```python
from sqlalchemy.orm import selectinload
from app.core.repository.base import BaseRepository
from app.models.my_entity import MyEntity

class MyEntityRepository(BaseRepository[MyEntity]):
    model = MyEntity
    relations = (selectinload(MyEntity.related_objects),)
    default_order_by = 'name'

    async def get_by_name(self, name: str) -> MyEntity | None:
        return await self.find_by(name=name)

    async def create(self, data: dict) -> MyEntity:
        entity = MyEntity(**data)
        return await self.save(entity)
```

**Regras do repository:**
- Herda `BaseRepository[ModelT]`
- Declara `model`, `relations` e `default_order_by`
- Apenas queries SQLAlchemy — nenhuma lógica de negócio
- Reutilize `save`, `update`, `find_by`, `list_all` da base

### 4. Service (`app/domain/<domain>/service.py`)

```python
import logging
from app.core.service.base import BaseService
from app.core.logging.schemas import LoggingParams
from app.domain.my_entity.repository import MyEntityRepository
from app.domain.my_entity.schema import MyEntitySchema

logger = logging.getLogger(__name__)

class MyEntityService(BaseService[MyEntityRepository, 'MyEntity']):
    def __init__(self, repository: MyEntityRepository):
        super().__init__(
            alias='MyEntity',
            repository=repository,
            logger_params=LoggingParams(
                logger=logger,
                service='MyEntityService',
                operation='init',
                message='',
            ),
            schema_class=MyEntitySchema,
            cache_prefix='my_entity',
        )

    async def create(self, data: MyEntityCreateSchema, user_request: str | None = None):
        try:
            entity = await self.repository.create(data.model_dump())
            return entity
        except Exception as exc:
            handle_service_exception(
                exc,
                logger=logger,
                service='MyEntityService',
                operation='create',
                raise_exception=True,
                user_request=user_request,
            )
        finally:
            log_service_success(
                self.logger_params,
                operation='create',
                message='MyEntity created successfully',
                user_request=user_request,
            )
```

**Regras do service:**
- Herda `BaseService[RepositoryT, ModelT]` para CRUD padrão
- Sempre usa `handle_service_exception` no `except`
- Sempre usa `log_service_success` no `finally`
- Use `list_all_cached` / `find_one_cached` quando o recurso é cacheável
- Nunca faz queries diretas ao banco

### 5. Route (`app/domain/<domain>/route.py`)

```python
from http import HTTPStatus
from typing import Annotated
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import get_session
from app.core.security import get_current_user
from app.domain.my_entity.repository import MyEntityRepository
from app.domain.my_entity.schema import MyEntityCreateSchema, MyEntitySchema
from app.domain.my_entity.service import MyEntityService
from app.models.user import User

router = APIRouter()
Session = Annotated[AsyncSession, Depends(get_session)]

def get_service(session: Session) -> MyEntityService:
    return MyEntityService(MyEntityRepository(session))

@router.get('/', response_model=list[MyEntitySchema], status_code=HTTPStatus.OK)
async def list_entities(
    service: Annotated[MyEntityService, Depends(get_service)],
    current_user: Annotated[User, Depends(get_current_user)],
):
    return await service.list_all_cached(user_request=str(current_user.id))

@router.post('/', response_model=MyEntitySchema, status_code=HTTPStatus.CREATED)
async def create_entity(
    data: MyEntityCreateSchema,
    service: Annotated[MyEntityService, Depends(get_service)],
    current_user: Annotated[User, Depends(get_current_user)],
):
    return await service.create(data, user_request=str(current_user.id))

@router.get('/{param}', response_model=MyEntitySchema, status_code=HTTPStatus.OK)
async def get_entity(
    param: str,
    service: Annotated[MyEntityService, Depends(get_service)],
    current_user: Annotated[User, Depends(get_current_user)],
):
    return await service.find_one_cached(param, user_request=str(current_user.id))
```

**Regras do route:**
- APENAS delegação — nenhuma lógica de negócio
- SEMPRE `response_model` explícito
- Rotas protegidas usam `Depends(get_current_user)`
- Factory function `get_service(session)` que instancia service + repository
- `user_request` é sempre `str(current_user.id)` para rastreio de logs

### 6. Registrar o router em `app/main.py`

```python
from app.domain.my_entity.route import router as my_entity_router
app.include_router(my_entity_router, prefix='/my-entities', tags=['MyEntity'])
```

### 7. Criar migração Alembic

```bash
make create-migration message="add my_entity table"
make migrate
```

---

## REGRAS DE NEGÓCIO PURAS (`business.py`)

Use quando há cálculos ou regras que não dependem de infra (banco, cache, HTTP):

```python
# app/domain/battle/business.py
def calculate_damage(attack: int, defense: int, base_power: int) -> int:
    return max(1, int((attack / defense) * base_power))

def is_effective(attacker_type: str, defender_types: list[str]) -> float:
    # lógica pura sem I/O
    ...
```

**Regras do business:**
- Funções puras (sem `async`, sem `import` de infra)
- Altamente testáveis de forma unitária
- Chamadas pelo `service.py`, nunca pelo `route.py`

---

## DOMÍNIOS EXISTENTES

| Domínio | Prefixo | Descrição |
|---|---|---|
| `auth` | `/auth` | Registro, login, /me |
| `trainer` | `/trainers` | Perfil do treinador |
| `pokemon` | `/pokemon` | Catálogo de pokémons (PokeAPI) |
| `pokedex` | `/pokedex` | Pokédex do treinador |
| `my_pokemon` | `/my-pokemon` | Pokémons capturados |
| `encounter` | `/battle` | Encontros com pokémons selvagens |
| `battle` | `/battle` | Batalhas |
| `battle_history` | `/battle/history` | Histórico de batalhas |
| `dashboard` | `/dashboard` | Estatísticas gerais |
| `pokemon_center` | `/pokemon-center` | Centro Pokémon (cura) |

---

## PADRÃO DE TESTE

Os testes espelham a estrutura `app/`:

```
tests/
└── app/
    ├── core/
    │   ├── security/test_security.py
    │   ├── cache/test_service.py
    │   └── repository/test_base.py
    └── domain/
        └── <domain>/
            ├── test_route.py      # testa endpoints (TestClient ou AsyncClient)
            ├── test_service.py    # testa service com mocks do repository
            └── test_business.py   # testa funções puras (sem mocks)
```

**Convenções de teste:**
- Usa `pytest-asyncio` com `pytest.mark.asyncio`
- `factory-boy` para criar fixtures de models
- `testcontainers` para banco e Redis reais em integração
- Mocks com `unittest.mock.AsyncMock` para repository nos testes de service
- `freezegun` para congelar tempo

---

## COMANDOS ESSENCIAIS

```bash
# Instalar dependências
poetry install

# Iniciar servidor de desenvolvimento
make dev

# Rodar todos os testes (lint + pytest + coverage)
make test

# Rodar apenas lint
make lint

# Rodar um arquivo de teste específico
make test-file file=tests/app/domain/pokemon/test_service.py

# Criar migração
make create-migration message="descrição da mudança"

# Aplicar migrações
make migrate

# Reverter última migração
make rollback-migration
```

---

## VARIÁVEIS DE AMBIENTE (`.env`)

```env
SECRET_KEY=sua_chave_secreta_jwt
DATABASE_URL=postgresql+psycopg://user:pass@localhost:5432/mach_db
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_CACHE_TTL_SECONDS=3600
POKEAPI_BASE_URL=https://pokeapi.co/api/v2
```

---

## CHECKLIST PRÉ-PR

```bash
make test   # deve passar lint + pytest + coverage
```

- [ ] Mudança feita na camada correta (route → service → repository)
- [ ] Nenhuma lógica de negócio no `route.py`
- [ ] Schemas com `from_attributes=True` nos de saída
- [ ] `handle_service_exception` em todo `except`
- [ ] `log_service_success` em todo `finally`
- [ ] `response_model` explícito em todo endpoint
- [ ] Se DB mudou: migração criada com `make create-migration`
- [ ] Testes para service e route atualizados
- [ ] Ruff limpo (`make lint`)
