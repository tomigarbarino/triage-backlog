# Backlog — Acme Tareas — 2026-10-15

> Ejemplo con datos ficticios. "Acme Tareas" es una app inventada de gestión de tareas para equipos (Next.js + Supabase en un monorepo).

**Estado:** borrador para aprobación de la líder técnica
**Fuentes:** 23 issues abiertos de `acme/tareas` (todos, con o sin label) y 3 PRs abiertos, al 15/10. Código revisado sobre `main` @ `4f2a91c`.
**Modo:** re-triage

## Lo más importante

1. **Un usuario invitado puede ver tareas de otros proyectos de la misma empresa** (#41). El arreglo está en el PR #52, con tests en verde y sin review.
2. **5 issues ya están resueltos en `main`** y siguen abiertos (#12, #18, #22, #27, #30).
3. **La exportación a CSV que pide ventas (#35) depende de decidir qué columnas ve cada rol.** Sin eso no se puede estimar.

## Preguntas que bloquean

**Estado de producción:**
- ¿La migración `20261001_task_archive.sql` está aplicada en producción? El código de archivar tareas (#27, ya en `main`) la usa.

**Decisiones de producto:**
- #35: ¿un invitado puede exportar, o solo los miembros?

## Resumen

### P0 — ya, fuera del sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Decisión |
|---|---|---|---|---|---|---|
| #41 | bug | Un invitado ve tareas de otros proyectos de la empresa | PR listo: #52 | S (review) | alta | pendiente |

### P1 — próximo sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Decisión |
|---|---|---|---|---|---|---|
| #44 | bug | Al mover una tarea entre columnas a veces se duplica | pendiente | M | media | pendiente |
| #38 | feature | Recordatorio por mail el día antes del vencimiento | parcial | M | alta | pendiente |

### P2 — backlog normal

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Decisión |
|---|---|---|---|---|---|---|
| #29 | mejora | El buscador no encuentra tareas por texto de los comentarios | pendiente | S | alta | pendiente |
| #33 | bug | La fecha de vencimiento se muestra en UTC | PR listo: #50 | S | alta | pendiente |
| #36 | feature | Plantillas de proyecto | pendiente | L | media | pendiente |

### P3 — cuando haya hueco

| ID | Tipo | Título | Estado | Esfuerzo | Confianza |
|---|---|---|---|---|---|
| #31 | mejora | Modo oscuro en el tablero | pendiente | M | alta |
| #39 | deuda | Partir `lib/tasks.ts` (2.400 líneas) | pendiente | L | alta |

### A definir

| ID | Tipo | Título | Qué hay que decidir |
|---|---|---|---|
| #35 | feature | Exportar tareas a CSV | Qué columnas ve cada rol y si los invitados pueden exportar |

### Cerrar después de verificar

| ID | Qué pasa (con evidencia) | Acción sugerida |
|---|---|---|
| #12 | El filtro por responsable ya está: `components/TaskFilters.tsx:88` | Probar y cerrar |
| #27 | Archivar tareas ya está en `main` (`lib/tasks.ts:1310`); el comentario del issue dice "falta mergear", pero el commit `9be03d1` está en `main` | Verificar la migración en producción y cerrar |

**Orden sugerido:** revisar y mergear #52 → confirmar la migración de #27 → #44 → cerrar los 5 resueltos → #38.

---

## Detalle de P0 y P1

### #41 — Un invitado ve tareas de otros proyectos de la empresa (re-triage)

**Prioridad sugerida:** P0 — fuga de datos entre proyectos; cualquier invitado lo puede hacer hoy desde la búsqueda.

La búsqueda global devuelve tareas de toda la empresa sin filtrar por los proyectos del usuario. Un invitado a un solo proyecto puede leer títulos y descripciones de todos.

- **Evidencia:** `app/api/search/route.ts:42` filtra solo por `company_id`; la policy de `tasks` permite leer por empresa (`supabase/migrations/20260810_tasks_rls.sql:15`).
- **PR:** #52 filtra por membresía y ajusta la policy. Checks en verde, sin review, con una migración.
- **Falta:** review y aplicar la migración.

### #44 — Al mover una tarea entre columnas a veces se duplica (bug)

**Prioridad sugerida:** P1 — afecta el flujo principal; pasa con conexiones lentas.

**Ticket funcional.**
- **Problema:** al arrastrar una tarea a otra columna con una conexión lenta, a veces aparece dos veces, y al recargar sigue duplicada.
- **Comportamiento esperado:** la tarea queda una sola vez, en la columna de destino.
- **Criterios de aceptación:**
  - [ ] Con la red limitada a 3G, mover una tarea 20 veces no genera duplicados.
  - [ ] Si el movimiento falla, la tarea vuelve a su columna original con un aviso.

**Notas técnicas.**
- **Causa probable** (confianza media): el cliente reintenta el `POST /api/tasks/move` sin clave de idempotencia y el endpoint inserta en vez de actualizar — `app/api/tasks/move/route.ts:27`, `hooks/useBoardDrag.ts:64`.
- **Arreglo propuesto:** usar `update` por `task_id` y una clave de idempotencia por movimiento.
- **Esfuerzo:** M  ·  **Riesgo:** toca el endpoint que usan también los movimientos masivos.
- **Tests a agregar:** integración del endpoint con el mismo request repetido.
- **Cómo se reprodujo:** no reproducido; el código lo explica.

### #38 — Recordatorio por mail el día antes del vencimiento (feature)

**Prioridad sugerida:** P1 — lo piden 3 clientes y destraba una renovación con fecha.

**Ticket funcional.**
- **Historia:** Como responsable de una tarea quiero un mail el día antes de que venza para no olvidarme.
- **Problema de fondo:** hoy los equipos se enteran del vencimiento cuando ya pasó.
- **Propuesta funcional:** un mail a las 9 h (hora del usuario) el día anterior al vencimiento, con link a la tarea.
- **Fuera de alcance:** recordatorios configurables y notificaciones push.
- **Criterios de aceptación:**
  - [ ] El responsable recibe un solo mail por tarea, el día anterior.
  - [ ] Se puede desactivar desde el perfil.

**Notas técnicas.**
- **Dónde entra:** ya existe el envío de mails (`lib/email.ts`) y un cron diario (`app/api/cron/daily/route.ts`); falta la consulta de tareas por vencer y la preferencia por usuario.
- **Dependencias:** ninguna.  ·  **Esfuerzo:** M (confianza alta)  ·  **Riesgo:** mails duplicados si el cron corre dos veces; usar una marca de "avisado".

---

## Decisiones

| ID | Decisión | Quién y cuándo | Notas |
|---|---|---|---|
| | | | |
