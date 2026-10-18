# Sala Onirica — Plataforma de venda d'entrades

Monorepo pnpm per a la plataforma de venda d'entrades de la Sala Onirica. Arquitectura dual de backend: **NestJS** per a temps real (WebSockets) i **Laravel** per a accés a base de dades i autenticació. Frontend **Nuxt 3** (SPA + SSR). Nginx com a proxy invers únic.

---

## Estructura del projecte

```
prj-entrades/
├── src/
│   ├── frontend/              # Nuxt 3 — SPA + SSR (port 3000)
│   ├── backend/
│   │   ├── node-service/      # NestJS — WebSockets + cron (port 3001)
│   │   └── laravel-service/   # Laravel — BD + auth REST API (port 8000)
│   └── shared/                # Tipus TypeScript compartits (no és runtime)
│       └── types/
│           ├── seat.types.ts
│           ├── event.types.ts
│           ├── socket.types.ts
│           └── auth.types.ts
├── doc/                       # PRD i backlog
├── openspec/                  # Especificacions i historials de canvi
├── pnpm-workspace.yaml
├── package.json               # Scripts globals
└── .nvmrc                     # Node.js LTS 20
```

---

## Stack tecnològic

| Capa | Tecnologia | Port |
|---|---|---|
| Frontend | Nuxt 3 (Vue 3, SPA + SSR) | 3000 |
| Node service | NestJS + Socket.IO | 3001 |
| Laravel service | Laravel 11 + Sanctum | 8000 |
| Base de dades | PostgreSQL | 5432 |
| Proxy invers | Nginx | 80 |
| Testing | Vitest (tots els workspaces JS) | — |
| Gestió de paquets | pnpm workspaces | — |

---

## Requisits previs

- **Node.js** >= 20 (recomanat: `nvm use`)
- **pnpm** >= 9 (`npm install -g pnpm`)
- **PHP** >= 8.2 + Composer (per a laravel-service)
- **Docker** + Docker Compose (per a PostgreSQL i entorn complet)

---

## Instal·lació

```bash
# Clonar el repositori
git clone <url> && cd prj-entrades

# Usar la versió correcta de Node.js
nvm use

# Instal·lar totes les dependències JS (frontend + node-service + shared)
pnpm install

# Instal·lar dependències PHP (quan laravel-service estigui disponible)
cd src/backend/laravel-service && composer install
```

---

## Scripts globals

Executa des de l'arrel del monorepo:

| Comanda | Descripció |
|---|---|
| `pnpm dev` | Arrenca frontend i node-service en paral·lel |
| `pnpm test` | Executa Vitest als 3 workspaces JS |
| `pnpm build` | Compila tots els workspaces JS |
| `pnpm lint` | Lint a tots els workspaces |
| `pnpm type-check` | TypeScript check a tots els workspaces |

Per a un workspace concret:

```bash
pnpm --filter frontend dev
pnpm --filter node-service test
pnpm --filter shared test
```

---

## Àlies TypeScript `@shared/*`

Tant `frontend` com `node-service` poden importar els tipus compartits sense rutes relatives:

```typescript
import { EstatSeient, ISeient } from '@shared/seat.types'
import { IEvent } from '@shared/event.types'
import { SeientCanviEstatPayload } from '@shared/socket.types'
import { UserRole, IUser, IJwtPayload } from '@shared/auth.types'
```

El paquet `shared` no té dependències de runtime i no requereix etapa de build.

---

## Arquitectura de comunicació

```
Browser
  │
  ▼
Nginx :80
  ├── /          → Nuxt 3       :3000  (SSR + assets)
  ├── /api/*     → Laravel      :8000  (REST API, auth, BD)
  └── /ws        → NestJS       :3001  (WebSockets, temps real)
                      │
                      └── HTTP intern → Laravel :8000
                          (reserves, expiració, stats)
```

**Flux d'autenticació:**
1. Client fa `POST /api/auth/login` → Laravel emet JWT
2. Client usa JWT al handshake WebSocket (`auth: { token }`)
3. NestJS valida el JWT localment (secret compartit) sense consultar BD
4. Per a operacions de BD, NestJS crida Laravel via HTTP intern

---

## Tests

```bash
# Tots els workspaces
pnpm test

# Només shared (types)
pnpm --filter shared test
```

Tests actuals: **14 tests** a `src/shared/types/` que verifiquen la integritat dels enums i la forma de les interfícies.

---

## Variables d'entorn

Copia `.env.example` a `.env` a cada servei i omple els valors:

| Variable | Servei | Descripció |
|---|---|---|
| `DATABASE_URL` | laravel | Connexió PostgreSQL |
| `JWT_SECRET` | laravel + node | Secret compartit per a JWT |
| `APP_KEY` | laravel | Clau d'aplicació Laravel |
| `NODE_SERVICE_URL` | laravel | URL interna del node-service |
| `LARAVEL_SERVICE_URL` | node | URL interna del laravel-service |

---

## Documentació

- [PRD v2.0](PRD.md) — Product Requirements Document
- [Backlog](JIRA-BACKLOG.md) — Historial i ordre de tasques
- [OpenSpec](../openspec/) — Especificacions i historial de canvis
