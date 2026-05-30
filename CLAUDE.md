# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`coffee-ai-menu` — an AI-powered coffee menu platform. Next.js 16 (App Router) + React 19 frontend that talks to a separate backend API. The frontend never calls the backend directly from the browser; it proxies through its own Next.js API routes.

## Architecture — Hybrid (Feature modules × Clean Architecture)

Application code lives in `src/modules/<feature>`. The codebase mixes **feature-based architecture** (vertical slices, one folder per feature) with **clean architecture** (horizontal layers inside each feature). Every feature is self-contained and split into the same layers, with dependencies pointing **inward** (presentation → application → domain; infrastructure depends on abstract/domain, never the reverse).

```
modules/<feature>/
  domain/                # Enterprise rules — pure, framework-free
    entities/            #   business entities / models
    value-objects/       #   immutable typed values
    errors/              #   domain-specific errors
  abstract/              # Ports — contracts the outer layers implement
    repositories/        #   repository interfaces (abstract classes)
    services/            #   service interfaces
  application/           # Application rules — orchestration
    use-cases/           #   one class/fn per business action
    dtos/                #   input/output data shapes
  infrastructure/        # Adapters — concrete implementations of `abstract/`
    repositories/        #   repository implementations
    services/            #   *Service classes: static methods + ENDPOINT map, call axios
    mappers/             #   DTO <-> entity mapping
    datasources/         #   remote/local data sources
  presentation/          # Delivery — React/Next UI
    components/          #   feature UI (often nested by sub-area)
    hooks/               #   thin wrappers over queries-mutations
    stores/              #   Zustand stores
    queries-mutations/   #   TanStack Query config + query-key factories
    schemas/             #   Zod schemas
    types/               #   *.d.ts ambient types
    layouts/ pages/ constants/
  index.ts               # public exports for the module
```

### Dependency rule
- `domain/` depends on nothing.
- `abstract/` may depend on `domain/`.
- `application/` depends on `domain/` + `abstract/` (never on `infrastructure/`).
- `infrastructure/` implements `abstract/` and may use `domain/`.
- `presentation/` consumes `application/` use-cases and `domain/` types.

Features: `auth`, `menu`, `products`, `orders`, `cart`, `ai-barista`, `dashboard`.

### Shared vs common vs config
- `src/common/` — design-system primitives. shadcn/ui components live in `common/shared/`, form fields in `common/forms/`, buttons in `common/buttons/`, the dynamic modal system in `common/models/`.
- `src/shared/` — cross-feature utilities, stores, and components.
- `src/config/` — app wiring: axios instances, React Query setup, i18n, session, shadcn utils, generated `swagger-apis/`.
- `src/containers/` — top-level providers mounted in the root layout.

Path alias: `@/*` → `src/*`.

### Data flow (two-hop proxy)
The browser never hits the backend directly: **browser → Next.js API route (`src/app/api/**`) → backend (`BACKEND_API_URL`)**. Feature `infrastructure/services/*` import the axios context switcher; API route handlers forward to the real backend.

### Routing & i18n
- Routes under `src/app/[locale]/`; auth pages grouped in `(auth)`.
- i18n via `next-intl` with locale-prefixed routes; messages in `/messages/{locale}.json`.

## Conventions
- TypeScript `strict`. Ambient types use `*.d.ts` inside each feature's `presentation/types/`.
- Keep the dependency rule above — never import `infrastructure` from `domain`/`application`.
