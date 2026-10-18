# Sala Onírica 🎟️

**Plataforma de venta de entradas con selección de asientos en tiempo real.** Varias personas pueden competir por el mismo asiento a la vez: el sistema garantiza que nunca se venda dos veces y todos ven los cambios al instante.

Proyecto individual del CFGS de Desarrollo de Aplicaciones Web (Institut Pedralbes, curso 2025-26). Lo diseñé y desarrollé de principio a fin: requisitos (PRD), arquitectura, frontend, los dos backends, tests e infraestructura.

![Nuxt](https://img.shields.io/badge/Nuxt_3-00DC82?logo=nuxtdotjs&logoColor=white)
![Vue](https://img.shields.io/badge/Vue_3-4FC08D?logo=vuedotjs&logoColor=white)
![Pinia](https://img.shields.io/badge/Pinia-FFD859?logo=pinia&logoColor=black)
![NestJS](https://img.shields.io/badge/NestJS-E0234E?logo=nestjs&logoColor=white)
![Laravel](https://img.shields.io/badge/Laravel-FF2D20?logo=laravel&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)
![Socket.io](https://img.shields.io/badge/Socket.io-010101?logo=socketdotio&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Vitest](https://img.shields.io/badge/Vitest-6E9F18?logo=vitest&logoColor=white)

---

## El reto técnico

Una venta de entradas tiene un problema clásico de **concurrencia**: si dos personas pulsan el mismo asiento en el mismo instante, solo una puede quedárselo. Lo resolví así:

- **Bloqueo pesimista en PostgreSQL** (`SELECT ... FOR UPDATE` dentro de una transacción) al reservar y al comprar. Aunque lleguen dos peticiones a la vez, la segunda espera, ve que el asiento ya no está libre y se rechaza con un mensaje claro.
- **El servidor es la única fuente de verdad.** El cliente nunca decide el estado de un asiento, solo lo muestra.
- **Difusión en tiempo real con Socket.IO.** Cada cambio (disponible → reservado → vendido) llega a todas las personas que están viendo el evento.
- **Reservas que caducan.** Una tarea programada revisa cada 30 segundos las reservas vencidas y devuelve los asientos a la venta.
- **Un test de integración de concurrencia** demuestra que, con dos reservas simultáneas del mismo asiento, exactamente una tiene éxito.

---

## Funcionalidades

**Comprador**
- Listado de eventos, renderizado en el servidor (SSR) para que cargue rápido y sea indexable.
- Mapa de asientos con su estado en tiempo real.
- Reserva de varios asientos, con cuenta atrás visible.
- Checkout y consulta de las entradas compradas.

**Administrador**
- Crear, editar y eliminar eventos, con categorías de precio y aforo.
- Panel en tiempo real: asientos disponibles, reservados y vendidos, y personas conectadas.
- Informes: recaudación por categoría y porcentaje de ocupación.

---

## Arquitectura

Un monorepo con **dos backends**, cada uno con una responsabilidad clara:

```
Navegador
   │
   ▼
 Nginx :80
   ├── /        →  Nuxt 3          (frontend, SSR)
   ├── /api/*   →  Laravel         (API REST, autenticación, base de datos)
   └── /ws      →  NestJS          (WebSockets y tareas programadas)
                      │
                      └── HTTP interno → Laravel
```

- **Laravel** gestiona la base de datos, la autenticación y las operaciones críticas (reserva y compra con bloqueo).
- **NestJS** mantiene las conexiones en tiempo real y la tarea de caducidad de reservas.
- **Autenticación compartida:** Laravel emite un JWT que el cliente envía al conectarse por WebSocket. NestJS lo valida localmente, sin consultar la base de datos.
- **Tipos TypeScript compartidos** (`src/shared`) entre el frontend y NestJS, para que los dos hablen exactamente el mismo idioma.

---

## Stack

| Capa | Tecnologías |
|---|---|
| **Frontend** | Nuxt 3 (Vue 3, SSR), Pinia, Socket.IO client, TypeScript |
| **Backend en tiempo real** | NestJS, Socket.IO, tareas programadas |
| **API y datos** | Laravel, Sanctum, PostgreSQL |
| **Infraestructura** | Docker Compose, Nginx (proxy inverso), Mailpit (pruebas de email) |
| **Calidad** | Vitest, PHPUnit, ESLint, Prettier, GitHub Actions (CI) |
| **Monorepo** | pnpm workspaces |

---

## Cómo ejecutarlo en local

**Requisitos:** Node.js 20+, pnpm 9+, PHP 8.2+ con Composer, y Docker.

```bash
git clone https://github.com/valeriazv99/sala-onirica-tickets.git
cd sala-onirica-tickets
nvm use
pnpm install
cd src/backend/laravel-service && composer install && cd -
```

Copia los archivos de entorno y revisa sus valores:

```bash
cp .env.example .env
cp src/backend/laravel-service/.env.example src/backend/laravel-service/.env
cp src/backend/node-service/.env.example src/backend/node-service/.env
```

Arranca todo con Docker (la app queda en http://localhost y Mailpit en http://localhost:8025):

```bash
docker compose up
```

O, para desarrollar, arranca el frontend y NestJS en paralelo:

```bash
pnpm dev
```

| Comando | Qué hace |
|---|---|
| `pnpm test` | Tests con Vitest en todos los workspaces |
| `pnpm lint` | Lint de todo el monorepo |
| `pnpm type-check` | Comprobación de tipos TypeScript |

---

## Metodología

Desarrollo *spec-driven* con [OpenSpec](https://github.com/Fission-AI/OpenSpec): cada funcionalidad empieza con una especificación y una propuesta de cambio antes de escribir código, y queda registrada en [`openspec/`](openspec/). Usé Claude como agente de código dentro de este flujo, con ramas por funcionalidad, pull requests y un backlog en Jira.

---

## Documentación

- [Documentación técnica](doc/README-tecnico.md) (en catalán): estructura, scripts y variables de entorno.
- [PRD](doc/PRD.md): requisitos, esquema de base de datos, endpoints, protocolo Socket.IO y gestión de concurrencia.
- [Backlog](doc/JIRA-BACKLOG.md)

---

Desarrollado por **Valeria Zavaleta** ([@valeriazv99](https://github.com/valeriazv99)) · [LinkedIn](https://www.linkedin.com/in/valeria-zavaleta-vasquez)
