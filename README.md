# coffee-ai-menu

An AI-powered coffee menu platform — Next.js 16 (App Router) + React 19.

## Architecture

Hybrid **feature-based** + **clean architecture**. Each feature in `src/modules/<feature>` is a vertical slice split into clean-architecture layers:

| Layer | Folder | Responsibility |
| --- | --- | --- |
| Domain | `domain/` | Entities, value objects, domain errors — pure, framework-free |
| Abstract (ports) | `abstract/` | Repository & service interfaces |
| Application | `application/` | Use-cases and DTOs (orchestration) |
| Infrastructure | `infrastructure/` | Repository/service implementations, mappers, datasources |
| Presentation | `presentation/` | Components, hooks, stores, queries, schemas, types, pages |

Dependencies point inward: `presentation → application → domain`, and `infrastructure` implements `abstract`.

Features: `auth`, `menu`, `products`, `orders`, `cart`, `ai-barista`, `dashboard`.

See [CLAUDE.md](./CLAUDE.md) for the full architecture guide.

## Structure

```
src/
  app/[locale]/        # routes (auth group, menu, dashboard) + api/ proxy routes
  modules/<feature>/   # feature slices (domain/abstract/application/infrastructure/presentation)
  common/              # design-system primitives (shadcn, forms, buttons, modals)
  shared/              # cross-feature utils, stores, components
  config/              # axios, react-query, i18n, swagger-apis
  containers/          # root-layout providers
messages/              # i18n message catalogs
```
