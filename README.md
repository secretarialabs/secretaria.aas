# Secretaria

Plataforma SaaS para crear asistentes inteligentes (orquestador de agentes).

## Descripción

Secretaria es una aplicación multi-canal que conecta Gmail, Calendar, WhatsApp, CRM y
otras fuentes para que agentes de IA clasifiquen, planifiquen y ejecuten tareas en nombre
del usuario. Los humanos aprueban las decisiones críticas; el resto se automatiza.

Este repositorio es un **monorepo con submódulos Git**. Cada subproyecto tiene su
propio `AGENTS.md` con reglas y stack detallados.

## Estructura

```
secretaria.aas/
├── AGENTS.md                 ← este directorio
├── README.md                 ← este archivo
├── docker-compose.yml        ← orquestación de web + bff
├── secretaria.web/           ← submodule: Next.js 16 (frontend)
└── secretaria.bff/           ← submodule: NestJS 10 (bff)
```

## Subproyectos

### `secretaria.web` — Frontend

Next.js 16 (App Router, Turbopack) + React 19 + TypeScript. Feature-Sliced Design (FSD).

**Tecnologías:**

- Next.js 16.1.1
- React 19.2.3
- TypeScript (strict)
- Tailwind CSS v4 + shadcn/ui (Radix UI)
- React Hook Form + Zod
- TanStack Query (server cache)
- Zustand (global state)
- Axios (HTTP)
- Next Themes (dark mode)
- Sonner (toasts)
- Vite (tests)
- ESLint flat config + Prettier
- pnpm 10 + Node 23

**Puerto dev:** `3210`

**Setup:**

```bash
cd secretaria.web
pnpm install
cp .env.example .env
pnpm dev
```

Más detalle: `secretaria.web/AGENTS.md` y `secretaria.web/README.md`.

### `secretaria.bff` — Backend For Frontend

NestJS 10 (Express) + TypeScript + Supabase (Auth + Postgres). Arquitectura Feature-First
por módulos NestJS. Pruebas con Vitest + SWC (para `emitDecoratorMetadata`).

**Tecnologías:**

- NestJS 10.4
- Node.js 23
- TypeScript (strict, no semicolons)
- Supabase (Auth + DB Postgres)
- Jose (verificación JWT)
- Zod + nestjs-zod (validación)
- cookie-parser (auth cookies httpOnly)
- Vitest + @swc/core + unplugin-swc
- ESLint flat config + Prettier
- pnpm 10

**Puerto dev:** `8080`

**Setup:**

```bash
cd secretaria.bff
pnpm install
cp .env.example .env
pnpm dev
```

Más detalle: `secretaria.bff/AGENTS.md` y `secretaria.bff/README.md`.

## Setup rápido (primera vez)

```bash
# 1. Clonar el repo padre con submodules
git clone --recurse-submodules <url-secretaria.aas>
cd secretaria.aas

# 2. O si ya tenés el clone sin submodules
git submodule update --init --recursive

# 3. Levantar en dev (ambos procesos en terminales separadas)
cd secretaria.bff && pnpm install && pnpm dev      # :8080
cd secretaria.web && pnpm install && pnpm dev      # :3210
```

O usando Docker Compose (build de ambos):

```bash
docker compose up --build
```

## Convenciones del monorepo

- **Branches:** `main` es la rama estable. Features en `feature/<nombre>`.
- **Commits:** Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`, etc.).
- **Node:** 23.x (gestionado vía `.nvmrc` en cada subproyecto si existe).
- **Package manager:** pnpm 10.x. Nunca npm/yarn/bun.
- **No commitear** `.env`, `node_modules/`, `dist/`, `coverage/`, `.tsbuildinfo`.

## Render

El monorepo incluye un blueprint raíz en [render.yaml](./render.yaml) para desplegar ambos servicios en `develop`:

- `secretaria-web-develop`
- `secretaria-bff-develop`

Cada servicio usa su propio `rootDir` (`secretaria.web` y `secretaria.bff`) para que Render trate el repo como monorepo y despliegue cada app por separado dentro del mismo Blueprint.

## Servicios externos

- **Supabase:** Auth (sign-up, sign-in, JWT) + DB Postgres gestionada.
  Configurar `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` y `SUPABASE_JWT_SECRET` en
  `secretaria.bff/.env`.

## Roles por subproyecto

| Subproyecto  | AGENTS.md propio                     | Stack docs                            |
|--------------|--------------------------------------|----------------------------------------|
| secretaria.web | `secretaria.web/AGENTS.md`         | `secretaria.web/README.md`            |
| secretaria.bff | `secretaria.bff/AGENTS.md`         | `secretaria.bff/README.md`            |

Si una tarea toca los dos, leer primero este archivo, después los dos `AGENTS.md`
de los subproyectos involucrados, y aplicar las reglas más restrictivas si hay
conflicto (prevalece el `AGENTS.md` del subproyecto específico).
