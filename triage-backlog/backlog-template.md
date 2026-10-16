# Backlog — <app> — <fecha>

**Estado:** borrador para aprobación de <dev a cargo>
**Fuentes:** <cantidad> reportes o issues (<fuentes leídas>), hasta <fecha>. Código revisado sobre <repo@commit>.
**Modo:** reportes nuevos / re-triage

## Lo más importante

1. <hallazgo que cambia el plan, ej. "3 fallas explotables hoy tienen el arreglo en PR, sin review">
2. <ej. "8 issues ya están resueltos en main">
3. <ej. "riesgo: migraciones que el código usa y quizás no están en producción">

## Preguntas que bloquean

**Estado de producción** (el agente no lo puede ver):
- <ej. ¿la migración X está aplicada en producción?>

**Decisiones de producto:**
- <ej. ¿quién puede generar contenido pago?>

## Resumen

Estado: **pendiente**, **parcial**, **hecho** (en la rama viva), **PR listo** (#N) u **obsoleto**.

### P0 — ya, fuera del sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Decisión |
|---|---|---|---|---|---|---|
| R1 | bug | <título en lenguaje de usuario> | PR listo: #N | S | alta | pendiente |

### P1 — próximo sprint

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Decisión |
|---|---|---|---|---|---|---|

### P2 — backlog normal

| ID | Tipo | Título | Estado | Esfuerzo | Confianza | Decisión |
|---|---|---|---|---|---|---|

### P3 — cuando haya hueco

| ID | Tipo | Título | Estado | Esfuerzo | Confianza |
|---|---|---|---|---|---|

### A definir (falta una decisión de producto)

| ID | Tipo | Título | Qué hay que decidir |
|---|---|---|---|

### Cerrar después de verificar

| ID | Qué pasa (con evidencia) | Acción sugerida |
|---|---|---|
| #N | <ya está en main: `archivo:línea`> | <probar X y cerrar> |

**Orden sugerido:** <R1 → R3 → R2, y por qué en una línea>

---

## Detalle de P0 y P1

### R1 — <título> (bug, reporte nuevo)

**Prioridad sugerida:** P0 — <una línea de por qué>
**Reportes:** <fuente y fecha; sin datos personales>  ·  **Duplicado de:** — / #N

**Ticket funcional.**
- **Problema:** <qué pasa, a quién afecta, desde cuándo>
- **Comportamiento esperado:** <en lenguaje de usuario>
- **Criterios de aceptación:**
  - [ ] <verificable por alguien que no lee código>

**Notas técnicas.**
- **Causa probable** (confianza alta/media/baja): <explicación> — `ruta/archivo.ts:123`
- **Arreglo propuesto:** <1–3 líneas>  ·  **Esfuerzo:** S/M/L  ·  **Riesgo:** <migración, permisos, otros flujos>
- **Tests a agregar:** <...>
- **Cómo se reprodujo:** <pasos, o "no reproducido: falta X">

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

### #N — <título> (re-triage)

**Prioridad sugerida:** P0 — <una línea de por qué>

<2–3 líneas: qué cambia para el usuario y por qué importa ahora.>

- **Evidencia:** `ruta/archivo.ts:123` (sigue presente en <commit>)
- **PR:** #M — <checks, migraciones, review> / ninguno
- **Falta:** <qué queda para cerrarlo>

---

## Decisiones

Por ítem: **aprobado**, **cambiar prioridad a ___**, **pedir más info**, **cerrar** o **descartar**.

| ID | Decisión | Quién y cuándo | Notas |
|---|---|---|---|
| | | | |
