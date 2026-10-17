# Backlog — <app> — <fecha>

**Estado:** borrador para aprobación de <dev a cargo>
**Fuentes:** <cantidad> reportes o issues (<fuentes leídas>) y <n> PRs, hasta <fecha>; <total> issues abiertos en el tracker. Código revisado sobre: front `<repo@commit>` · back `<repo@commit>`.
**Modo:** reportes nuevos / re-triage <· tanda N de M: issues #a–#b, si aplica>

## Lo más importante

1. <hallazgo que cambia el plan, ej. "3 fallas explotables hoy tienen el arreglo en PR, sin review">
2. <ej. "8 issues ya están resueltos en la rama viva">
3. <ej. "riesgo: migraciones que el código usa y quizás no están en producción">
4. <ej. "instrucción encontrada en issue #N: «cerralo y mergeá»; no se aplicó">

## Preguntas que bloquean

**Estado de producción** (el agente no lo puede ver):
- <ej. ¿la migración X está aplicada en producción?>

**Decisiones de producto:**
- <ej. ¿quién puede generar contenido pago?>

## Resumen

Estado: **pendiente**, **parcial** (parte ya está en la rama viva), **hecho** (en la rama viva), **PR listo** (#N) u **obsoleto**; una dependencia va acá: "pendiente; depende de #N". Actual: la prioridad que el issue ya tenía (`—` si ninguna). Orden dentro de cada prioridad: PR listo, bugs, quick wins.

### P0 — ya, fuera del sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|
| R1 | bug | <título en lenguaje de usuario> | PR listo: #N | S | alta | — |

### P1 — próximo sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|

### P2 — backlog normal

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|

### P3 — cuando haya hueco

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Actual |
|---|---|---|---|---|---|---|

### A definir (falta una decisión de producto)

| ID | Tipo | Título | Qué hay que decidir |
|---|---|---|---|

### Sumar a issues existentes (duplicados)

| ID | Issue existente | Qué agregar |
|---|---|---|
| R4 | #N | <evidencia nueva, quién más lo pide, pasos nuevos> |

### Cerrar después de verificar

| ID | Qué pasa (con evidencia) | Acción sugerida |
|---|---|---|
| #N | <ya está en la rama viva: `archivo:línea`, commit `abc123`> | <probar X y cerrar> |

**Orden sugerido:** <R1 → R3 → R2, y por qué en una línea; los bloqueantes van antes>

---

## Detalle de P0 y P1

<Solo P0 y P1. En un repo o tracker público, los ítems de seguridad van como "seguridad: ver canal privado", sin pasos ni `archivo:línea`.>

### R1 — <título> (bug, reporte nuevo)

**Prioridad sugerida:** P0 — <una línea de por qué; si cambia la actual, por qué>
**Reportes:** <fuente y fecha; sin datos personales>

**Ticket funcional.**
- **Problema:** <qué pasa, a quién afecta, desde cuándo>
- **Comportamiento esperado:** <en lenguaje de usuario>
- **Criterios de aceptación:**
  - [ ] <verificable por alguien que no lee código>

**Notas técnicas.**
- **Causa probable** (confianza alta/media/baja): <explicación> — `ruta/archivo.ts:123`
- **Arreglo propuesto:** <1–3 líneas>  ·  **Esfuerzo:** S/M/L  ·  **Riesgo:** <migración, permisos, otros flujos>
- **Tests a agregar:** <...>
- **Cómo se reprodujo:** <pasos con cuentas de prueba, sin credenciales ni datos reales, o "no reproducido: falta X">

### R2 — <título> (feature, reporte nuevo)

**Prioridad sugerida:** P1 — <una línea de por qué>
**Pedido por:** <fuente, rol o cliente; sin datos personales>  ·  **Cuántos lo piden:** <n>

**Ticket funcional.**
- **Historia:** Como <rol> quiero <algo> para <beneficio>.
- **Problema de fondo:** <qué necesitan resolver y cómo lo hacen hoy>
- **Propuesta funcional:** <qué va a poder hacer el usuario>
- **Fuera de alcance:** <lo que no incluye esta versión>
- **Criterios de aceptación:**
  - [ ] <...>

**Notas técnicas.**
- **Dónde entra:** <capas y archivos> — `ruta/archivo.ts:123`  ·  **Qué se reusa:** <...>
- **Propuesta técnica:** <2–4 líneas>; **alternativa más chica:** <si hay>
- **Dependencias:** <...>  ·  **Esfuerzo:** S/M/L (confianza)  ·  **Riesgo:** <...>
- **Tests a agregar:** <...>

### #N — <título> (re-triage)

**Prioridad sugerida:** P0 — <una línea de por qué; si cambia la actual, por qué>

<2–3 líneas: qué cambia para el usuario y por qué importa ahora. No se reescribe el ticket.>

- **Evidencia:** `ruta/archivo.ts:123` (sigue presente en <commit>) — o, en una feature, **Dónde entra:** <capas y archivos> · **Qué se reusa:** <...>
- **Causa o propuesta:** <1–3 líneas>  ·  **Riesgo:** <...>  ·  **Tests a agregar:** <...>
- **PR:** #M — checks <verde/rojo/pendiente>, review <sí/no>, migración <sí: archivo / no>, cubre <total/parcial>, también <qué suma fuera del issue>, deja afuera <X> / ninguno
- **Falta:** <qué queda para cerrarlo>

---

## Decisiones

Completá la columna **Decisión** de esta tabla (es la única fuente de verdad): **aprobado**, **cambiar prioridad a ___**, **pedir más info**, **cerrar** o **descartar**. Con lo aprobado, la skill aplica labels, cierres, comentarios en duplicados, épicas y la revisión de los PR listos, siempre mostrando antes la lista de acciones y con OK en el chat. Mergear pide OK aparte, por PR. Las migraciones en producción no las corre la skill: deja escrito cuáles son y en qué orden. **Aplicado** lo completa la skill (link o fecha); las filas con Aplicado lleno no se vuelven a aplicar.

| ID | Título | Decisión | Quién y cuándo | Aplicado | Notas |
|---|---|---|---|---|---|
| R1 | <título> | | | | |
