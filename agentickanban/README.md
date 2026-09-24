# agentickanban (plugin de ZCode)

Plugin para **desarrollar, operar y contribuir** a
[agentic-kanban](https://github.com/p-wegner/agentic-kanban) — tablero kanban local-first
para tareas de código impulsadas por IA (re-implementación cleanroom de vibe-kanban).

## Componentes

| Tipo | Nombre | Para qué |
|------|--------|----------|
| Skill | `agentickanban-dev` | Setup, arquitectura de los 6 paquetes, convenciones obligatorias, gates de calidad, fuentes de conocimiento (graphify/mempalace) |
| Skill | `agentickanban-contribute` | Flujo fork→upstream: sincronización con rebase, convenciones de PR `ak-<N>`, gates antes del PR |
| Skill | `agentickanban-operate` | Instalar (npx/Docker), modo dos tableros, `pnpm promote`, env vars `KANBAN_*`, service stacks, worker fleet, troubleshooting |
| Agente | `agentickanban-expert` | Subagente experto que orquesta las skills y las fuentes de conocimiento del proyecto |

Cada skill lleva `references/` con documentación destilada del upstream y del workspace.

## Fuente de la información

Compilada el 2026-09-24 desde: docs del upstream (`docs/decisions`, `docs/domain`, `CLAUDE.md`,
`docs/env-vars.md`), la guía del workspace (`docs/DEVELOPMENT-GUIDE.md`), y research web
(vibe-kanban/Bloop, hilo de HN, reviews, loop engineering).

## Uso

Tras instalar el plugin desde el marketplace local, invoca las skills desde el composer
(`/agentickanban-dev`, etc.) o deja que el agente `agentickanban-expert` se active con
cualquier pregunta sobre agentic-kanban.
