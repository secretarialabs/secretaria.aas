# Secretaria

Secretaria es una plataforma SaaS para crear asistentes inteligentes
(orquestador de agentes).

## Arquitectura

Monorepo con submódulos Git.

- `web/` → Next.js 16 + React 19 + TypeScript
- `bff/` → NestJS 10 + Express + TypeScript + Supabase

## Principios

- TypeScript estricto.
- pnpm obligatorio.
- Node 23 obligatorio.
- Código limpio antes que optimización.
- Preferir composición sobre herencia.
- No duplicar lógica.
- Mantener arquitectura simple para el MVP.

## Convenciones

- camelCase para variables.
- PascalCase para componentes y clases.
- ESLint obligatorio.
- Prettier obligatorio.
- Commits convencionales.

## Estructura

```
web/   → secretaria.web (Next.js)
bff/   → secretaria.bff (NestJS)
```

Cada subproyecto posee su propio `AGENTS.md` con reglas específicas.

## Cómo trabajar

Cuando una tarea involucre más de un proyecto:

1. leer primero este archivo
2. luego leer el `AGENTS.md` correspondiente
3. respetar las reglas de ambos

Si existe conflicto:

- prevalece el `AGENTS.md` del subproyecto.
