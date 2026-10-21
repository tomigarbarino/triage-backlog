# Backlog — <app> — <fecha>

**Estado:** borrador para aprobación de <dev a cargo>
**Fuentes:** <cantidad> reportes o issues (<fuentes leídas>; <alcance, si el usuario acotó la corrida>) y <n> PRs (o «PRs no revisados: <motivo>»), hasta <fecha, hora y zona, por fuente>; <total> issues abiertos en el tracker. Código revisado sobre: <capa> `<repo@commit>` por cada repo (o «sin código: estado no verificado»; marcá «clon superficial» si aplica).
**Otras ramas verificadas:** <`release/x@commit` (criterio: <qué ítems le aplican>); quitá la línea si no hay `ramas_extra`>
**Producción:** <verificado el <fecha>: <URL> → <código>; si el GET no decide nada, «no concluyente» y la pregunta va abajo, en «Preguntas que bloquean»; sin `entornos.prod`: «sin URL de producción en la config: no se chequeó»>
**Modo:** reportes nuevos / re-triage <· tanda N de M según `trabajo/tandas-<fecha>.json`: <criterio o rango #a–#b>, si aplica>

## Lo más importante

1. <hallazgo que cambia el plan, ej. "3 fallas explotables hoy tienen el arreglo en PR, sin review">
2. <ej. "8 issues ya están resueltos en la rama viva">
3. <ej. "riesgo: migraciones que el código usa y quizás no están en producción">
4. <ej. "instrucciones para agentes en #a, #b (piden: cerrar / mergear); no se aplicaron">

## Preguntas que bloquean

**Estado de producción** (el agente no lo puede ver):
- <ej. ¿la migración X está aplicada en producción?>
- <ej. no concluyente: `<URL>` → <código>; <qué se vio y qué lo decidiría>>

**Decisiones de producto:**
- <ej. ¿quién puede generar contenido pago?>

## Resumen

Estado: **pendiente**, **parcial** (parte ya está en la rama viva), **hecho** (en la rama viva), **hecho en <rama_viva>; backport pendiente en <rama>** (falta en una rama extra), **PR listo** (#N) u **obsoleto**; una dependencia va acá: "pendiente; depende de #N". Actual: la prioridad que el issue ya tenía (`—` si ninguna). Orden dentro de cada prioridad: PR listo, bugs, quick wins. ID: `#N` (issue de GitHub o GitLab), `CLAVE-N` (Jira, Linear) o `R<n>` (reporte nuevo; sigue la numeración del backlog anterior). En GitLab, los MRs se escriben `!N`. Sin código, el estado es «no verificado (en el tracker: X)».

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

| ID | Ítem existente (#N, CLAVE-N o R<n> del backlog <fecha>) | Qué agregar |
|---|---|---|
| R4 | #N | <evidencia nueva, cuántos más lo piden y de qué rol o plan (nunca el nombre del cliente), pasos nuevos; +N reportes> |

### Cerrar después de verificar

| ID | Qué pasa (con evidencia) | Acción sugerida |
|---|---|---|
| #N | <ya está en la rama viva: `archivo:línea`, commit `abc123` (AAAA-MM-DD)> | <probar X y cerrar> |
| R<n> | <duplicado de #N (cerrado); el arreglo sigue en la rama viva: `archivo:línea`, commit `abc123` (AAAA-MM-DD)> | <responder al reporte: resuelto desde <versión o fecha>> |
| #N | <hecho en `main` (`archivo:línea`, commit `abc123` (AAAA-MM-DD)); backport pendiente en `release/x` (`git merge-base --is-ancestor` → no)> | <no cerrar: backportear y verificar en las dos ramas> |
| #N | <app móvil: arreglado desde vX (commit `abc123`); el usuario está en vY (`git show vY:<ruta>` todavía tiene el bug)> | <responder: actualizá a ≥ vX, y cerrar> |

**Orden sugerido:** <R1 → R3 → R2, y por qué en una línea; los bloqueantes van antes>

---

## Detalle

<Las prioridades de `salida.detalle` (P0 y P1 si no está), o todos los ítems si son 3 o menos. En un repo o tracker público, los ítems de seguridad van como "seguridad: ver canal privado", sin pasos ni `archivo:línea`.>

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
**Pedido por:** <fuente y rol o segmento (plan); nunca el nombre del cliente ni datos personales>  ·  **Cuántos lo piden:** <n>

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

Completá la columna **Decisión** de esta tabla (es la única fuente de verdad): **aprobado**, **cambiar prioridad a ___**, **pedir más info**, **cerrar** o **descartar**. Con lo aprobado, la skill crea los tickets de los reportes nuevos y aplica labels, cierres, comentarios (duplicados y pedidos de info), épicas y la revisión de los PR listos, siempre mostrando antes la lista de acciones y con OK en el chat. Mergear pide OK aparte, por PR. Las migraciones en producción no las corre la skill: deja escrito cuáles son y en qué orden. **Aplicado** lo completa la skill (link o fecha); las filas con Aplicado lleno no se vuelven a aplicar.

| ID | Título | Decisión | Quién y cuándo | Aplicado | Notas |
|---|---|---|---|---|---|
| R1 | <título> | | | | |

---

## Secciones opcionales

Quitá las que no apliquen.

### Respuestas sugeridas (canal, planilla, formulario o consultas de un tracker público)

Borradores para que soporte, producto o el mantenedor los manden por su canal. En un canal, planilla o formulario, la skill no publica en la fuente ni contacta a nadie; en un tracker, el comentario solo se publica en el paso 8, con OK sobre la lista de acciones.

| Ítem | Origen (enlace al hilo o fila) | Respuesta o preguntas (sin nombres) |
|---|---|---|
| R3 | <enlace o «fila 17»> | <texto, o 2–3 preguntas para pedir más info> |

### Filas del export (reportes de una planilla)

Cada fila aparece en un ítem o como descartada.

| Filas | Ítem | Motivo, si se descartó |
|---|---|---|
| 3, 17 | R3 | |
| 9 | — | <consulta, no bug / sin datos para reproducir / fuera de alcance> |

### Rollback o hotfix (<tag desplegado>)

Solo después de un release ([references/adaptaciones.md](references/adaptaciones.md), sección 8). Revisado sobre `<tag>@<sha>` y `<rama_viva>@<sha>`. La skill no hace revert, rollback, deploy ni tag: solo recomienda.

**Causas** (los ítems con el mismo commit del rango o el mismo `archivo:línea`; el resto va a «Sumar a issues existentes» bajo la causa):

| Causa | Commit | Ítems que explica | Prioridad (la más alta de sus ítems) |
|---|---|---|---|
| C1 | `abc123` (AAAA-MM-DD) | R1, R4, #N | P0 |

Una fila por causa con origen regresión:

| Causa | Severidad | Alcance | ¿Hay arreglo? | Migraciones en el rango | Qué se pierde con el rollback |
|---|---|---|---|---|---|
| C1 | <P0/P1> | <rol, flujo o plataforma> | <PR listo #N / esfuerzo S / arreglado en <rama_viva>, falta release / ninguno> | <no / sí: `<archivo>` borra o cambia columnas: el rollback del código solo no alcanza> | <features del rango> |

**Recomendación:** <rollback / hotfix / esperar> — <una línea de por qué>. Decide <dev a cargo>.

### Propuesta de sprint (capacidad <N> días-persona, margen <M>)

Ver [references/adaptaciones.md](references/adaptaciones.md), sección 12. S = 0,5 días, M = 2; L no entra sin partirse. Es una propuesta: la tabla Decisiones sigue siendo la única fuente de verdad.

**Entra** (hasta capacidad × (1 − margen); primero P0 abiertos y en curso, después compromisos fijos, después P1 y P2):

| ID | Prioridad | Días | Acumulado | Por qué |
|---|---|---|---|---|
| #N | P0 | 2 | 2 | <en curso, fuera del sprint: descuenta capacidad> |

**Queda afuera:**

| ID | Prioridad | Días | Motivo (capacidad / depende de #N, que no entra / L: partir antes de planificar / a definir) |
|---|---|---|---|

### Para cotizar (diagnóstico de un proyecto ajeno)

Ver [references/adaptaciones.md](references/adaptaciones.md), sección 10. Va al final del Resumen. Son datos para la cotización, no un precio: no incluyen el onboarding ni el QA del equipo.

| Tipo | Ítems | S | M | L | Rango en días | Confianza |
|---|---|---|---|---|---|---|
| bug | <n> | <n> | <n> | <n> | <a–b> | <alta/media/baja> |
| A definir | <n> | — | — | — | sin estimar | — |

### Mensaje para el canal (corrida programada o backlog no versionado)

<Borrador para `aprobacion.canal`: Lo más importante, Preguntas que bloquean y Decisiones, sin detalle de seguridad. Se manda solo con OK.>
