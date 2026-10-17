# Backlog — Acme Tareas — 2026-10-15

> Ejemplo con datos ficticios. "Acme Tareas" es una app inventada de gestión de tareas para equipos (Next.js + Supabase en un monorepo). En un repo o tracker público, el #41 iría como "seguridad: ver canal privado", sin evidencia ni pasos.

**Estado:** borrador para aprobación de la líder técnica
**Fuentes:** 15 issues abiertos de `acme/tareas` (todos, con o sin label) y 2 PRs abiertos, al 15/10; 15 issues abiertos en el tracker. Código revisado sobre: front y back `acme/tareas@4f2a91c` (monorepo, `main`).
**Modo:** re-triage

## Lo más importante

1. **Un usuario invitado puede ver tareas de otros proyectos de la misma empresa** (#41). El arreglo está en el PR #52, con tests en verde y sin review.
2. **5 issues ya están resueltos en `main`** y siguen abiertos (#12, #18, #22, #27, #30). Cuatro citan la rama `feature/archivado`, que ya está integrada en `main` (commit `9be03d1`) aunque los comentarios dicen "falta mergear".
3. **La exportación a CSV que pide ventas (#35) depende de decidir qué columnas ve cada rol.** Sin eso no se puede estimar.

## Preguntas que bloquean

**Estado de producción** (el agente no lo puede ver):
- ¿La migración `20261001_task_archive.sql` está aplicada en producción? El código de archivar tareas (#27, ya en `main`) la usa.

**Decisiones de producto:**
- #35: ¿un invitado puede exportar, o solo los miembros?

## Resumen

Estado: **pendiente**, **parcial** (parte ya está en la rama viva), **hecho** (en la rama viva), **PR listo** (#N) u **obsoleto**; una dependencia va acá: "pendiente; depende de #N". Actual: la prioridad que el issue ya tenía (`—` si ninguna). Orden dentro de cada prioridad: PR listo, bugs, quick wins.

### P0 — ya, fuera del sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|
| #41 | bug | Un invitado ve tareas de otros proyectos de la empresa | PR listo: #52 | S (review) | alta | P1 |

### P1 — próximo sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|
| #44 | bug | Al mover una tarea entre columnas a veces se duplica | pendiente | M | media | — |
| #38 | feature | Recordatorio por mail el día antes del vencimiento | pendiente | M | alta | P2 |

### P2 — backlog normal

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|
| #33 | bug | La fecha de vencimiento se muestra en UTC | PR listo: #50 | S | alta | P2 |
| #29 | mejora | El buscador no encuentra tareas por texto de los comentarios | pendiente | S | alta | — |
| #36 | feature | Plantillas de proyecto | pendiente | L | media | — |

### P3 — cuando haya hueco

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|
| #31 | mejora | Modo oscuro en el tablero | pendiente | M | alta | — |
| #39 | deuda | Partir `lib/tasks.ts` (2.400 líneas) | pendiente | L | alta | — |

### A definir (falta una decisión de producto)

| ID | Tipo | Título | Qué hay que decidir |
|---|---|---|---|
| #35 | feature | Exportar tareas a CSV | Qué columnas ve cada rol y si los invitados pueden exportar |

### Sumar a issues existentes (duplicados)

| ID | Issue existente | Qué agregar |
|---|---|---|
| #40 | #38 | Un cliente más pide el recordatorio; sumar el pedido a #38 y cerrar #40 como duplicado |

### Cerrar después de verificar

| ID | Qué pasa (con evidencia) | Acción sugerida |
|---|---|---|
| #12 | El filtro por responsable ya está: `components/TaskFilters.tsx:88`, commit `c71e2d0` en `main` | Probar y cerrar |
| #18 | El contador de tareas vencidas ya se actualiza al cambiar la fecha: `hooks/useTaskCounters.ts:31`, commit `9be03d1` | Probar y cerrar |
| #22 | La pantalla de tareas archivadas ya existe: `app/projects/[id]/archive/page.tsx:1`, commit `9be03d1` | Probar y cerrar |
| #27 | Archivar tareas ya está en `main` (`lib/tasks.ts:1310`); el comentario del issue dice "falta mergear", pero el commit `9be03d1` está en `main` | Verificar la migración en producción y cerrar |
| #30 | Restaurar una tarea archivada ya está: `lib/tasks.ts:1342`, commit `9be03d1` | Probar y cerrar |

**Orden sugerido:** review y merge de #52 (el arreglo de #41 ya está en PR) → confirmar la migración de #27 → #44 → cerrar los 5 resueltos → #38.

---

## Detalle de P0 y P1

### #41 — Un invitado ve tareas de otros proyectos de la empresa (re-triage)

**Prioridad sugerida:** P0 — fuga de datos entre proyectos (`siempre_p0`); cualquier invitado lo puede hacer hoy desde la búsqueda. Sube de P1 porque el issue estaba subestimado: es fuga de datos, no un bug de un flujo.

La búsqueda global devuelve tareas de toda la empresa sin filtrar por los proyectos del usuario. Un invitado a un solo proyecto puede leer títulos y descripciones de todos.

- **Evidencia:** `app/api/search/route.ts:42` filtra solo por `company_id`; la policy de `tasks` permite leer por empresa (`supabase/migrations/20260810_tasks_rls.sql:15`). Sigue presente en `4f2a91c`.
- **Causa o propuesta:** filtrar por membresía en el endpoint y en la policy.  ·  **Riesgo:** la policy la usan también el tablero y los reportes.  ·  **Tests a agregar:** un invitado busca y solo ve sus proyectos.
- **PR:** #52 — checks verde, review no, migración sí (`supabase/migrations/20261012_tasks_rls_membership.sql`), cubre total, también ajusta el índice de búsqueda, deja afuera nada.
- **Falta:** review, merge y aplicar la migración.

### #44 — Al mover una tarea entre columnas a veces se duplica (re-triage)

**Prioridad sugerida:** P1 — afecta el flujo principal; pasa con conexiones lentas.

Al arrastrar una tarea a otra columna con una conexión lenta, a veces aparece dos veces y al recargar sigue duplicada. El issue ya describe el problema y el comportamiento esperado; no cambia.

- **Evidencia:** el cliente reintenta el `POST /api/tasks/move` sin clave de idempotencia (`hooks/useBoardDrag.ts:64`) y el endpoint inserta en vez de actualizar (`app/api/tasks/move/route.ts:27`). Confianza media: no reproducido, el código lo explica.
- **Causa o propuesta:** `update` por `task_id` y una clave de idempotencia por movimiento.  ·  **Riesgo:** el endpoint lo usan también los movimientos masivos.  ·  **Tests a agregar:** integración del endpoint con el mismo request repetido.
- **PR:** ninguno
- **Falta:** todo el arreglo.

### #38 — Recordatorio por mail el día antes del vencimiento (re-triage)

**Prioridad sugerida:** P1 — lo piden 3 clientes (con #40) y destraba una renovación con fecha. Sube de P2 por el compromiso con fecha.

El responsable de una tarea recibiría un mail el día anterior al vencimiento. El issue ya trae historia y criterios; se suma el pedido de #40.

- **Dónde entra:** el envío de mails (`lib/email.ts:12`) y el cron diario (`app/api/cron/daily/route.ts:20`) ya existen; falta la consulta de tareas por vencer y la preferencia por usuario. Nada de la feature está hecho: estado pendiente.
- **Causa o propuesta:** un job en el cron diario que consulte tareas que vencen mañana, mande el mail y marque "avisado"; preferencia en el perfil.  ·  **Riesgo:** mails duplicados si el cron corre dos veces.  ·  **Tests a agregar:** un solo mail por tarea aunque el cron corra dos veces.
- **PR:** ninguno
- **Falta:** todo; esfuerzo M.

---

## Decisiones

Completá la columna **Decisión** de esta tabla (es la única fuente de verdad): **aprobado**, **cambiar prioridad a ___**, **pedir más info**, **cerrar** o **descartar**. Con lo aprobado, la skill aplica labels, cierres, comentarios en duplicados, épicas y la revisión de los PR listos, siempre mostrando antes la lista de acciones y con OK en el chat. Mergear pide OK aparte, por PR. Las migraciones en producción no las corre la skill: deja escrito cuáles son y en qué orden. **Aplicado** lo completa la skill (link o fecha); las filas con Aplicado lleno no se vuelven a aplicar.

| ID | Título | Decisión | Quién y cuándo | Aplicado | Notas |
|---|---|---|---|---|---|
| #41 | Un invitado ve tareas de otros proyectos de la empresa | | | | |
| #44 | Al mover una tarea entre columnas a veces se duplica | | | | |
| #38 | Recordatorio por mail el día antes del vencimiento | | | | |
| #33 | La fecha de vencimiento se muestra en UTC | | | | |
| #29 | El buscador no encuentra tareas por texto de los comentarios | | | | |
| #36 | Plantillas de proyecto | | | | |
| #31 | Modo oscuro en el tablero | | | | |
| #39 | Partir `lib/tasks.ts` | | | | |
| #35 | Exportar tareas a CSV | | | | |
| #40 | Duplicado de #38 | | | | |
| #12 | Filtro por responsable (cerrar) | | | | |
| #18 | Contador de tareas vencidas (cerrar) | | | | |
| #22 | Pantalla de tareas archivadas (cerrar) | | | | |
| #27 | Archivar tareas (cerrar) | | | | |
| #30 | Restaurar tarea archivada (cerrar) | | | | |
