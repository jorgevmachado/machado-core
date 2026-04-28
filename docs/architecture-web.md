# Architecture

## mach-web

### Overview

**Stack principal:**
- Next.js 16 (App Router) + React 19
- TypeScript 5 (strict — sem `any`)
- Tailwind CSS v4 + SASS (`.scss`)
- Jest 30 + React Testing Library (testes de componente)
- ESLint 9 + eslint-config-next (lint)
- npm / Node.js

---

## ESTRUTURA DE PASTAS — EXPLICAÇÃO COMPLETA

```
mach-web/
├── app/
│   ├── layout.tsx                   # Root layout: providers globais (UserProvider)
│   ├── page.tsx                     # Rota raiz: redireciona para /home ou /login
│   ├── globals.css                  # CSS global + variáveis Tailwind
│   ├── favicon.ico
│   ├── (protected)/                 # Grupo de rotas autenticadas
│   │   ├── layout.tsx               # Guard: redireciona para /login se não autenticado
│   │   ├── home/page.tsx
│   │   ├── pokemon/
│   │   │   ├── page.tsx             # Listagem de pokémons
│   │   │   └── [name]/page.tsx      # Detalhe do pokémon
│   │   ├── pokedex/...
│   │   └── my-pokemon/...
│   ├── (public)/                    # Grupo de rotas públicas
│   │   ├── layout.tsx               # Guard: redireciona para /home se já autenticado
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── api/                         # Next.js Route Handlers (BFF — lê cookie servidor)
│   │   ├── auth/me/route.ts         # GET /api/auth/me → chama mach-api /auth/me
│   │   ├── pokemon/route.ts         # GET /api/pokemon → chama mach-api /pokemon
│   │   ├── pokemon/[name]/route.ts
│   │   ├── pokedex/...
│   │   ├── my-pokemon/...
│   │   └── trainers/...
│   ├── actions/                     # Server Actions (formulários, mutações)
│   │   ├── auth.ts                  # loginAction, registerAction, logoutAction
│   │   └── trainer.ts
│   ├── ds/                          # Design System: componentes reutilizáveis puros
│   │   ├── alert/
│   │   ├── autocomplete/
│   │   ├── badge/
│   │   ├── bar-chart/
│   │   ├── breadcrumb/
│   │   ├── button/
│   │   ├── card/
│   │   ├── filters/
│   │   ├── image/
│   │   ├── input/
│   │   ├── loading/
│   │   ├── modal/
│   │   ├── pagination/
│   │   ├── text/
│   │   └── index.ts                 # Barrel: exporta tudo do DS
│   ├── shared/                      # Infraestrutura compartilhada (não-feature)
│   │   ├── index.ts
│   │   ├── cookies/
│   │   │   └── cookies.ts           # Helpers de cookie client-side
│   │   ├── lib/
│   │   │   └── auth/
│   │   │       ├── action-state.ts  # AuthActionState + INITIAL_AUTH_ACTION_STATE
│   │   │       ├── constants.ts     # AUTH_COOKIE_NAME, AUTH_TOKEN_MAX_AGE_IN_SECONDS
│   │   │       ├── server.ts        # Re-exporta: setAuthCookie, clearAuthCookie, getServerSession
│   │   │       ├── index.ts
│   │   │       ├── session/
│   │   │       │   └── session.ts   # setAuthCookie, clearAuthCookie, getServerSession (server-only)
│   │   │       ├── token/
│   │   │       │   └── token.ts     # isValidAuthToken, getAuthTokenExpiration, extractAuthToken
│   │   │       └── validation/
│   │   │           └── validation.ts # isValidEmail, isStrongPassword
│   │   └── services/
│   │       ├── index.ts
│   │       ├── http/
│   │       │   └── http.ts          # Classe Http abstrata: get, post, path, remove, send
│   │       └── service/
│   │           ├── service.ts       # BaseServiceAbstract extends Http (token → Authorization header)
│   │           └── types.ts         # IServiceConfig, IBaseResponse, IQueryParameters, IPaginate
│   ├── ui/                          # Features de UI (por domínio)
│   │   ├── index.ts
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   │   ├── service/         # AuthService extends BaseServiceAbstract
│   │   │   │   ├── types.ts         # SignInParams, SignUpParams, LoginResponsePayload, TUser
│   │   │   │   └── user/            # UserContext, UserProvider, useUser, server.ts
│   │   │   ├── pokemon/
│   │   │   │   ├── service/         # PokemonService extends BaseServiceAbstract
│   │   │   │   ├── list/            # usePokemonList (usa usePaginatedList)
│   │   │   │   ├── detail/
│   │   │   │   └── types.ts
│   │   │   ├── pokedex/
│   │   │   │   ├── service/
│   │   │   │   ├── list/
│   │   │   │   ├── detail/
│   │   │   │   └── types.ts
│   │   │   ├── my-pokemon/
│   │   │   │   ├── service/
│   │   │   │   ├── list/
│   │   │   │   ├── detail/
│   │   │   │   └── types.ts
│   │   │   ├── trainer/
│   │   │   │   ├── service/
│   │   │   │   └── types.ts
│   │   │   └── navigation/
│   │   │       ├── Navbar.tsx
│   │   │       ├── Sidebar.tsx
│   │   │       ├── NavigationFrame.tsx
│   │   │       └── constants.ts
│   │   └── hooks/
│   │       └── list/
│   │           └── usePaginatedList.ts  # Hook genérico de listagem paginada com filtros
│   └── utils/                       # Utilitários puros (sem side effects)
│       ├── array/
│       ├── color/
│       ├── join-class/
│       ├── number/
│       ├── object/
│       ├── pagination/
│       ├── string/
│       ├── tailwind/
│       ├── url/
│       ├── window/
│       ├── constants.ts
│       ├── types.ts
│       └── index.ts
├── public/                          # Assets estáticos
├── next.config.ts
├── tsconfig.json
├── eslint.config.mjs
├── jest.config.ts
├── jest.setup.ts
├── postcss.config.mjs
└── package.json
```

---

## ARQUIVO POR ARQUIVO — PAPEL E REGRAS

### `app/layout.tsx` (Root Layout)
- Provider raiz: envolve a aplicação com `UserProvider`
- Inicializa sessão server-side e passa `initialUser`, `isAuthenticated`, `tokenExpiresAt` para o provider
- Define `<html>`, `<body>`, fontes globais e metadados

### `app/(protected)/layout.tsx`
- Server Component assíncrono
- Chama `getServerSession()` e redireciona para `/login` se não autenticado
- **Nunca** contém lógica além do guard de autenticação

### `app/(public)/layout.tsx`
- Server Component assíncrono
- Chama `getServerSession()` e redireciona para `/home` se já autenticado
- **Nunca** contém lógica além do guard

### `app/api/<recurso>/route.ts` (Route Handlers — BFF)
- Lê o cookie de sessão server-side via `getServerSession()`
- Retorna 401 se não autenticado
- Instancia o Service correspondente com o token e delega a chamada para a mach-api
- Retorna `NextResponse.json(data)`
- **Nunca** contém lógica de negócio ou validação além da autenticação

### `app/actions/<domínio>.ts` (Server Actions)
- Marcados com `'use server'`
- Assinatura: `async function xyzAction(prevState: AuthActionState, formData: FormData): Promise<AuthActionState>`
- Padrão:
  1. Ler payload do `FormData` com helpers locais (`getStringValue`)
  2. Validar com funções puras; retornar `{ status: 'error', message }` se inválido
  3. Chamar o Service correspondente
  4. Em caso de sucesso: `redirect(...)` ou retornar `{ status: 'success', message }`
  5. Em caso de erro: retornar `{ status: 'error', message }` mapeado de `ResponseError`
- **Nunca** usados para queries de leitura — apenas mutações e autenticação

### `app/shared/lib/auth/session/session.ts`
- Marcado com `'server-only'`
- `setAuthCookie(token)` → define cookie `httpOnly`, `secure` em produção, `sameSite: 'lax'`
- `clearAuthCookie()` → remove o cookie de sessão
- `getServerSession()` → retorna `{ isAuthenticated, token? }` lendo o cookie e validando com `isValidAuthToken`

### `app/shared/lib/auth/token/token.ts`
- `isValidAuthToken(token)` → decodifica payload JWT base64url e verifica `exp`
- `getAuthTokenExpiration(token)` → retorna `exp * 1000` (ms) ou `undefined`
- `extractAuthToken(payload)` → extrai `access_token` de `LoginResponsePayload`; lança `Error` se ausente
- `createMockAuthToken()` → gera token falso para testes

### `app/shared/lib/auth/validation/validation.ts`
- `isValidEmail(email)` → regex simples
- `isStrongPassword(password)` → verifica contra `PASSWORD_PATTERN` (definido em `constants.ts`)

### `app/shared/services/http/http.ts` — `Http`
- Classe abstrata base para todas as chamadas HTTP
- Métodos: `get<T>`, `post<B, T>`, `path<B, T>` (PUT), `remove<T>`
- Privado `send<T>`: monta headers (`content-type: application/json`), chama `fetch`, trata erros com `handle` e `errorMessage`
- `ResponseError`: `{ error, message, statusCode }`
- Usa `formatUrl(url, path, params)` de `@/app/utils` para montar a URL com query params

### `app/shared/services/service/service.ts` — `BaseServiceAbstract`
- Herda `Http`
- Construtor recebe `baseUrl`, `pathUrl`, `token?`
- Se `token` presente → injeta `Authorization: Bearer <token>` nos headers
- Todas as features de serviço herdam esta classe

### `app/ui/features/<feature>/service/service.ts` — Feature Service
- Herda `BaseServiceAbstract`
- Construtor: `super(baseUrl, '<pathUrl>', token?)`
- Métodos tipados com os tipos definidos em `types.ts` da feature
- **Nunca** contém lógica de validação ou negócio

### `app/ui/features/auth/user/UserProvider.tsx`
- `'use client'`
- Gerencia estado global do usuário: `user`, `isLoading`
- `refreshUser()` → chama `/api/auth/me` (Route Handler BFF)
- Auto-invalida sessão via `setTimeout` quando `tokenExpiresAt` é atingido
- Refresca usuário ao `focus` e `visibilitychange`
- Expõe contexto via `UserContext`

### `app/ui/hooks/list/usePaginatedList.ts`
- Hook genérico `'use client'` para listagens paginadas
- Parâmetros: `endpoint`, `initialFilters`, `initialInputFilters`, `fetchErrorMessage`, `normalizeFilters`, `buildQueryString?`
- Estado interno: `items`, `meta` (`TPaginatedMeta`), `isLoading`, `errorMessage`
- Controla race conditions via `requestIdRef`
- Expõe: `goToPage`, `applyFilters`, `applyInputFilters`, `clearFilters`, `clearInputFilters`, `updateInputFilters`, `reload`
- Faz `fetch` direto ao Route Handler BFF (ex.: `/api/pokemon?page=1&limit=12`)

### `app/ds/<componente>/`
- Componentes React puros sem lógica de negócio
- Cada componente tem: `ComponentName.tsx`, `types.ts`, `index.ts` (barrel)
- Componentes com testes têm `ComponentName.spec.tsx`
- **Proibido** importar features ou serviços dentro do DS

---

## FLUXO DE AUTENTICAÇÃO

```
Login Form (Client)
  → loginAction (Server Action)
    → valida payload
    → AuthService.login() → POST mach-api /auth/login
    → setAuthCookie(token)  ← cookie httpOnly
    → redirect('/home')

Página protegida (Server Component)
  → (protected)/layout.tsx
    → getServerSession()
      → lê cookie → isValidAuthToken()
    → se !isAuthenticated → redirect('/login')

Dados do usuário (Client)
  → UserProvider.refreshUser()
    → GET /api/auth/me  (Route Handler)
      → getServerSession() → token do cookie
      → AuthService.me() → GET mach-api /auth/me
    → atualiza UserContext

Logout
  → logoutAction (Server Action)
    → clearAuthCookie()
```

---

## COMO IMPLEMENTAR UMA FEATURE — PASSO A PASSO

### 1. Tipos (`app/ui/features/<feature>/types.ts`)

```typescript
// Tipos de resposta da API
export type TMyEntity = {
  id: string;
  name: string;
  createdAt: string;
};

// Parâmetros de query para listagem
export type MyEntityListQuery = {
  name?: string;
  page?: number;
  limit?: number;
};

// Resultado do hook de listagem (re-exporta usePaginatedList result)
export type UseMyEntityListResult = UsePaginatedListResult<TMyEntity, MyEntityFilters>;
```

**Regras:**
- Nunca usar `any` — sempre tipar completamente
- Tipos de resposta espelham os schemas Pydantic da mach-api
- Separar tipos de entrada (params) dos tipos de saída (response)

### 2. Service (`app/ui/features/<feature>/service/service.ts`)

```typescript
import { BaseServiceAbstract } from '@/app/shared/services/service/service';
import { TMyEntity, MyEntityListQuery } from '../types';
import { TPaginatedListResponse } from '@/app/ds';
import { omitUndefined } from '@/app/utils';

export class MyEntityService extends BaseServiceAbstract {
  constructor(baseUrl: string, token?: string) {
    super(baseUrl, 'my-entities', token);
  }

  public async list(params: MyEntityListQuery = {}): Promise<TPaginatedListResponse<TMyEntity>> {
    const sanitizedParams = omitUndefined(params);
    return await this.get<TPaginatedListResponse<TMyEntity>>(this.pathUrl, {
      params: { ...sanitizedParams },
    });
  }

  public async detail(id: string): Promise<TMyEntity> {
    return await this.get<TMyEntity>(`${this.pathUrl}/${id}`);
  }
}

// Factory function (padrão de instanciação)
export const myEntityService = (token?: string) =>
  new MyEntityService(process.env.NEXT_PUBLIC_API_URL!, token);
```

**Regras:**
- Herda `BaseServiceAbstract`
- Métodos públicos tipados com os tipos da feature
- Factory function exportada para facilitar instanciação em Route Handlers e Server Actions
- `omitUndefined` para não enviar query params vazios

### 3. Route Handler BFF (`app/api/<feature>/route.ts`)

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from '@/app/shared/lib/auth/server';
import { myEntityService } from '@/app/ui/features/my-entity/service';

export async function GET(request: NextRequest): Promise<NextResponse> {
  const session = await getServerSession();

  if (!session.isAuthenticated || !session.token) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const { searchParams } = new URL(request.url);
    const page = Number(searchParams.get('page') ?? 1);
    const limit = Number(searchParams.get('limit') ?? 12);
    const name = searchParams.get('name') ?? undefined;

    const data = await myEntityService(session.token).list({ page, limit, name });
    return NextResponse.json(data);
  } catch {
    return NextResponse.json({ message: 'Internal Server Error' }, { status: 500 });
  }
}
```

**Regras:**
- SEMPRE verificar autenticação antes de qualquer chamada à API
- Retornar `NextResponse.json(...)` com status HTTP explícito em erros
- Leitura de `searchParams` feita diretamente; sem validação complexa no Route Handler

### 4. Hook de listagem (`app/ui/features/<feature>/list/useMyEntityList.ts`)

```typescript
'use client';

import type { FiltersProps } from '@/app/ds';
import usePaginatedList from '@/app/ui/hooks/list';
import type { MyEntityFilters, TMyEntity, UseMyEntityListResult } from '../types';

const INITIAL_FILTERS: MyEntityFilters = { name: '' };

const INITIAL_INPUT_FILTERS: FiltersProps['filters'] = [
  {
    label: 'NAME',
    type: 'text',
    name: 'name',
    value: '',
    placeholder: 'Search by name',
  },
];

const useMyEntityList = (): UseMyEntityListResult => {
  return usePaginatedList<TMyEntity, MyEntityFilters>({
    endpoint: '/api/my-entities',
    initialFilters: INITIAL_FILTERS,
    initialInputFilters: INITIAL_INPUT_FILTERS,
    fetchErrorMessage: 'Could not fetch entries.',
    normalizeFilters: (f) => ({ name: f?.name?.trim() }),
  });
};

export default useMyEntityList;
```

**Regras:**
- `'use client'` obrigatório
- `endpoint` aponta sempre para o Route Handler BFF (`/api/...`)
- `normalizeFilters` deve trimar strings e remover valores vazios

### 5. Página de listagem (`app/(protected)/<feature>/page.tsx`)

```typescript
'use client';

import { Card, Filters, Pagination, Text } from '@/app/ds';
import { useMyEntityList } from '@/app/ui/features/my-entity';

export default function MyEntityPage() {
  const {
    items, meta, isLoading,
    inputFilters, applyInputFilters, clearInputFilters,
    errorMessage, goToPage,
  } = useMyEntityList();

  return (
    <main>
      <Filters
        ariaLabel="MyEntity filters"
        filters={inputFilters}
        onApply={applyInputFilters}
        onClear={clearInputFilters}
      />

      {errorMessage && <Card>...</Card>}

      <section>
        {items.map((entity) => (
          <Card key={entity.id}>
            <Text>{entity.name}</Text>
          </Card>
        ))}
      </section>

      <Pagination
        currentPage={meta.current_page}
        totalPages={meta.total_pages}
        onPageChange={goToPage}
        isLoading={isLoading}
      />
    </main>
  );
}
```

**Regras:**
- Páginas são **Client Components** quando usam hooks; Server Components quando apenas renderizam dados estáticos
- Toda UI de listagem usa `usePaginatedList` (via hook de feature específico)
- Componentes visuais sempre do Design System (`@/app/ds`)
- **Proibido** fazer `fetch` diretamente na página — sempre via hook

### 6. Server Action para mutação (`app/actions/<feature>.ts`)

```typescript
'use server';

import { redirect } from 'next/navigation';
import type { AuthActionState } from '@/app/shared/lib/auth/action-state';
import { getServerSession } from '@/app/shared/lib/auth/server';
import { myEntityService } from '@/app/ui/features/my-entity/service';

export async function createMyEntityAction(
  _: AuthActionState,
  formData: FormData,
): Promise<AuthActionState> {
  const name = (formData.get('name') as string)?.trim();

  if (!name || name.length < 2) {
    return { status: 'error', message: 'Name must have at least 2 characters.' };
  }

  const session = await getServerSession();

  if (!session.isAuthenticated || !session.token) {
    return { status: 'error', message: 'Unauthorized.' };
  }

  try {
    await myEntityService(session.token).create({ name });
  } catch {
    return { status: 'error', message: 'Could not create. Please try again.' };
  }

  redirect('/my-entities');
}
```

**Regras:**
- `'use server'` obrigatório
- Retorna sempre `AuthActionState` (`{ status, message }`)
- Valida antes de chamar o service
- Lê token da sessão server-side (`getServerSession`)
- `redirect()` após mutações de sucesso

### 7. Componente DS (`app/ds/<componente>/MyComponent.tsx`)

```typescript
import React from 'react';
import { MyComponentProps } from './types';

export function MyComponent({ label, variant = 'default', onClick }: MyComponentProps) {
  return (
    <button
      type="button"
      className={`rounded px-4 py-2 ${variant === 'primary' ? 'bg-blue-600 text-white' : 'bg-gray-100'}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
}
```

**Regras:**
- Props tipadas em `types.ts` separado
- Sem imports de features, services ou shared/lib
- Sempre acessível (ARIA quando necessário)
- Testes em `MyComponent.spec.tsx` com React Testing Library

---

## FEATURES EXISTENTES

| Feature | Rota web | Endpoint BFF | Domínio mach-api |
|---|---|---|---|
| `auth` | `/login`, `/register` | `/api/auth/me` | `/auth` |
| `pokemon` | `/pokemon`, `/pokemon/[name]` | `/api/pokemon` | `/pokemon` |
| `pokedex` | `/pokedex` | `/api/pokedex` | `/pokedex` |
| `my-pokemon` | `/my-pokemon` | `/api/my-pokemon` | `/my-pokemon` |
| `trainer` | — | `/api/trainers` | `/trainers` |

---

## PADRÃO DE TESTE

Os testes ficam junto ao arquivo testado com sufixo `.spec.tsx` (componentes) ou `.spec.ts` (hooks/utils):

```
app/
├── ds/
│   └── button/
│       ├── Button.tsx
│       └── Button.spec.tsx          # testa renderização e interações
├── ui/
│   └── hooks/
│       └── list/
│           ├── usePaginatedList.ts
│           └── usePaginatedList.spec.tsx  # testa lógica de paginação com fetch mock
└── shared/
    └── lib/
        └── auth/
            └── session/
                ├── session.ts
                └── session.spec.ts
```

**Convenções de teste:**
- `@testing-library/react` para componentes e hooks
- `jest.fn()` / `jest.spyOn()` para mocks de `fetch` e módulos externos
- `@testing-library/jest-dom` para matchers semânticos (`toBeInTheDocument`, `toHaveAttribute`)
- Nunca testar detalhes de implementação — testar comportamento visível
- Server Actions e Route Handlers testados via integração quando possível

---

## COMANDOS ESSENCIAIS

```bash
# Instalar dependências
npm install

# Iniciar servidor de desenvolvimento
npm run dev

# Build de produção
npm run build

# Rodar testes (jest --passWithNoTests)
npm run test

# Rodar lint (ESLint)
npm run lint
```

---

## VARIÁVEIS DE AMBIENTE (`.env.local`)

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

- Variáveis com `NEXT_PUBLIC_` são expostas ao browser
- Variáveis sem prefixo estão disponíveis apenas no servidor (Server Components, Route Handlers, Server Actions)
- **Nunca** commitar `.env.local` ou `.env`

---

## CHECKLIST PRÉ-PR

```bash
npm run lint    # deve passar sem erros
npm run build   # build de produção sem erros de tipo
npm run test    # todos os testes passando
```

- [ ] Nenhum `any` em TypeScript
- [ ] Componente DS sem imports de features ou serviços
- [ ] Route Handler BFF verifica autenticação antes de chamar a API
- [ ] Server Action valida payload antes de chamar o service
- [ ] Pages de listagem usam `usePaginatedList` via hook de feature
- [ ] Cookie de autenticação gerenciado exclusivamente em `app/shared/lib/auth/session/session.ts`
- [ ] Variáveis de ambiente de servidor **sem** prefixo `NEXT_PUBLIC_`
- [ ] Testes `.spec.tsx` atualizados para componentes e hooks alterados
- [ ] `npm run lint` e `npm run build` passando sem warnings
