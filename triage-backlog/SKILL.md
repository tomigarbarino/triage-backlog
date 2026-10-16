---
name: triage-backlog
description: Relevar reportes de una app (bugs y pedidos de features o mejoras) o re-triagear un backlog de issues existente. Los toma pegados o de una fuente (issues del tracker, canal de Slack o Teams, formulario, pantalla de reportes de la app), revisa el código del front y del back, diagnostica la causa del bug o dónde entraría la feature, detecta lo que ya está resuelto o tiene PR, y arma un backlog en borrador priorizado (P0–P3) para que el dev a cargo lo apruebe antes de crear o cerrar nada. Incluye un setup con preguntas ("/triage-backlog setup") que la conecta a cualquier proyecto la primera vez. Usar cuando pidan "triageá este bug", "relevá los reportes de <fuente>", "revisá el backlog y priorizalo", "qué issues ya están resueltos", "armame el backlog", "cuánto cuesta esta feature y dónde entra", o peguen un reporte o pedido pidiendo análisis.
---

# Triage de reportes → backlog priorizado

De reportes o issues a un backlog en borrador. **Nada se crea, cierra ni cambia en el tracker hasta que el dev a cargo lo aprueba.** Esta skill no cambia código.

## Modos

- **`/triage-backlog setup`** (o "configurá triage-backlog para este proyecto"): conecta la skill a un proyecto. Una vez por proyecto, o de nuevo para cambiar la config.
- **`/triage-backlog`** con reportes pegados, una fuente, o nada (usa las fuentes de la config): corre el triage.

El triage tiene dos variantes, según la entrada:

- **Reportes nuevos** (mensajes, formularios, texto pegado): se convierten en ítems de backlog con ticket funcional.
- **Re-triage** (la fuente son issues que ya existen): el issue ya es el ticket. El valor está en verificar contra el código qué sigue pendiente, qué ya está hecho, qué tiene PR y qué está bloqueado, y en priorizar.

Si se pide un triage y el repo no tiene `.claude/triage.yml`, avisá y corré el setup primero.

## 0. Setup del proyecto

1. **Detectar.** Leé el repo: stack, si front y back están en el mismo repo o separados, carpetas de pantallas, componentes, endpoints, servicios y migraciones, rama principal, remoto y tracker (en GitHub: `gh repo view` y `gh label list`). Si hay otros repos involucrados, preguntá cuáles son y dónde están clonados. No inventes rutas.
2. **Preguntar** al usuario en un solo mensaje, con lo detectado como respuesta sugerida (si la herramienta de preguntas está disponible, usala):
   - **Fuentes:** de dónde salen los reportes. Issues del tracker (¿todos o con qué labels?), canal de Slack o Teams, formulario o planilla, pantalla de reportes de la app, o solo pegados.
   - **Tracker:** dónde viven y se crean los tickets: GitHub, Jira, Linear, otro, o ninguno (en ese caso el resultado es un archivo para importar o copiar).
   - **Aprobación:** quién aprueba el backlog (rol o nombre de pila, sin mail ni teléfono) y, si aplica, quién aprueba el alcance de las features.
   - **Prioridad:** los flujos críticos de la app, qué cuenta siempre como P0, y si el equipo ya usa otra escala (ej. alta/media/baja) que haya que mapear a P0–P3.
   - **Salida:** idioma de los tickets y dónde guardar el backlog (por defecto `docs/backlog/`, o fuera del repo si el equipo prefiere no versionarlo).
3. **Probar las fuentes.** Leé los últimos 3 reportes de cada fuente, solo lectura, y mostrá los títulos. Si una no responde (falta conector o permisos), decilo y dejala comentada en la config.
4. **Escribir** `.claude/triage.yml` desde [config-template.yml](config-template.yml) (ejemplos en [config-examples.md](config-examples.md)) y mostrarlo. Se guarda con el OK del usuario.
5. **Labels.** Si faltan las de tipo o prioridad en el tracker, listalas y crealas solo con OK.
6. **Cerrar** con un resumen de lo configurado y cómo correrlo: `/triage-backlog` o "relevá los reportes y armame el backlog". Sugerí commitear `.claude/` para que el equipo use la misma config.

## Antes de cada triage

- Leé `.claude/triage.yml`.
- `git fetch` y confirmá que revisás la rama viva (`rama_viva`) al día. Si el working tree está en otra rama, usá `git show origin/<rama_viva>:<ruta>` o un worktree aparte; no cambies la rama de trabajo de nadie.
- Anotá el commit que revisás: va en el encabezado del backlog.

## 1. Juntar los reportes

- **Pegados en el chat:** usá eso.
- **Desde una fuente** (pedido explícito o `fuentes` de la config), en modo solo lectura:
  - Issues de GitHub: `gh issue list --repo <repo> --state open --limit 200 --json number,title,body,labels,createdAt,comments`, con `--label <label>` por cada label de la fuente, o sin `--label` si la config dice `labels: todas`.
  - Jira, Linear u otro tracker: con el conector del agente o un export que pase el usuario.
  - Canal de Slack o Teams: con el conector del agente, si está disponible.
  - Formulario o planilla: el export (CSV, Excel) o el link que pase el usuario.
  - Pantalla o tabla de reportes de la app: solo lectura; nunca cambies el estado de un reporte desde acá.
- **PRs abiertos:** en re-triage, listalos también (`gh pr list --state open --json number,title,body,headRefName`) y anotá qué issue cubre cada uno.

En modo lote, tomá solo lo nuevo desde la última corrida (la fecha está en el backlog anterior, o preguntala) y anotá qué fuentes leíste y hasta qué fecha.

Si no tenés acceso a la fuente, pedile al usuario que pegue o exporte los reportes. No completes huecos inventando.

## 2. Clasificar y normalizar

Cada ítem es de un **tipo**: **bug** (algo que existe no funciona como debería), **feature** (algo que no existe), **mejora** (funciona, pero se pide distinto o mejor), **deuda** (interno, sin cambio visible) o **épica** (agrupa otros).

Si no está claro (ej. "no puedo exportar a Excel": ¿se rompió o nunca existió?), mirá el código antes de clasificar. Si sigue la duda, marcalo y preguntá.

En reportes nuevos, sacá los hechos sin copiar datos personales de quien reporta:

- **Bug:** qué pasa y qué se esperaba; dónde (pantalla, URL, rol, entorno); desde cuándo y si es siempre; pasos para reproducir; evidencia.
- **Feature o mejora:** quién lo pide y para qué (el problema de fondo, no solo la solución que proponen); a quién más le serviría; cómo lo resuelven hoy; si hay fecha o compromiso con un cliente.

Agrupá lo que es el mismo problema o pedido. Buscá duplicados en el tracker (`gh issue list --state all --search "<palabras>"`): si ya hay un issue, el ítem pasa a "agregar evidencia o votos a #N".

## 3. Revisar el código

**Reparto del trabajo:**

- Pocos ítems (menos de ~15): por capa. Front (ruta → componente → estado → llamada a la API) y back (endpoint → servicio → datos y permisos → integraciones), en paralelo si tu agente soporta subagentes.
- Muchos ítems: lotes de 8 a 12 por tema (ej. seguridad, notificaciones, una épica), un subagente de solo lectura por lote, en paralelo. Cada lote recibe el texto completo de sus ítems y el checkout a revisar, y devuelve una tabla fija: tipo, estado, evidencia (`archivo:línea` o commit), dónde entra el cambio, esfuerzo, riesgo y confianza.

**Qué buscar por ítem:**

- **Estado real:** **pendiente**, **parcial**, **hecho** (ya está en la rama viva), **PR listo** (hay un PR abierto que lo cubre) u **obsoleto**.
- **Bug:** el flujo con `archivo:línea`, los sospechosos y los commits recientes en esa zona (`git log --since=30.days -- <ruta>`). Si se puede reproducir sin riesgo (local, staging o una lectura en prod), reproducilo. Nunca escribas en datos de producción.
- **Feature o mejora:** qué existe que se pueda reusar, dónde entra el cambio en cada capa, qué datos o permisos toca, y si choca con algo en curso.

**El código manda.** El estado sale del código y del historial de git, no de lo que dicen el issue o sus comentarios ("ya está mergeado", "falta X"). Si no coinciden, anotá la contradicción.

**Control:** antes de armar el backlog, verificá vos mismo en el código cada hallazgo que vaya a quedar P0 o P1, y cada "hecho" que proponga cerrar un issue. Los subagentes se equivocan.

## 4. Analizar

- **Bug:** causa probable con `archivo:línea` y **confianza**: alta (reproducido, o el código lo muestra sin duda), media (el código lo explica pero no se reprodujo), baja (hipótesis; decí qué la confirmaría). Arreglo propuesto en 1–3 líneas.
- **Feature o mejora:** propuesta de solución en 2–4 líneas, alternativas más chicas si las hay, dependencias (otras features, datos, decisiones pendientes) y confianza de la estimación.
- **Todos:** capa, **esfuerzo** S (< medio día), M (1–2 días), L (más; partirlo), y **riesgo** (migración, permisos, otros flujos que usan el mismo código).

## 5. Priorizar

Prioridad sugerida P0–P3. Las reglas de `triage.yml` pisan estas tablas. Si el equipo usa otra escala, mapeala y decilo.

**Bugs** (severidad × alcance):

| Prioridad | Cuándo |
|---|---|
| **P0** | Caída, pérdida o corrupción de datos, falla de seguridad o permisos, o un flujo crítico bloqueado para todos sin alternativa. Va ya, fuera del sprint. |
| **P1** | Flujo crítico roto para un grupo de usuarios, o con una alternativa mala. Próximo sprint como máximo. |
| **P2** | Funciona mal pero hay alternativa razonable, o afecta a pocos. Backlog normal. |
| **P3** | Cosmético, texto, caso borde raro. Cuando haya hueco. |

**Features y mejoras** (valor × alcance ÷ esfuerzo). Nunca son P0:

| Prioridad | Cuándo |
|---|---|
| **P1** | Destraba una venta, una renovación o un compromiso con fecha; o la piden varios clientes y el esfuerzo es S/M. |
| **P2** | Valor claro para un grupo de usuarios, sin fecha comprometida. |
| **P3** | Nice to have o valor incierto; o esfuerzo L sin un pedido fuerte detrás. |

- Cada prioridad lleva **una línea de por qué**.
- Con confianza baja, no subas la prioridad sin decirlo: "P1 si se confirma X".
- Un ítem con **PR listo** va primero dentro de su prioridad: es el trabajo de más valor por esfuerzo.
- Dentro de cada prioridad: primero bugs, después quick wins (S de alto valor).
- Si algo necesita una decisión de producto antes de estimarse, va a **"a definir"**, no con una prioridad inventada.
- Lo que está **hecho** u **obsoleto** no se prioriza: va a **"cerrar después de verificar"**.

## 6. Backlog en borrador

Armá el backlog con [backlog-template.md](backlog-template.md) y guardalo donde diga `salida.backlog` de la config. Estructura:

1. **Lo más importante:** 3 o 4 líneas con los hallazgos que cambian el plan (ej. fallas explotables con PR listo, issues ya resueltos, riesgos de producción).
2. **Preguntas que bloquean**, separadas en **estado de producción** (lo que el agente no puede ver: migraciones aplicadas, switches de configuración, datos reales) y **decisiones de producto**. Van arriba porque pueden cambiar prioridades.
3. **Tablas por prioridad** (P0, P1, P2, P3), más **"a definir"** y **"cerrar después de verificar"**, con: ID, tipo, título, estado, esfuerzo, confianza y decisión del dev.
4. **Detalle solo para P0 y P1:**
   - En reportes nuevos: ticket funcional (en `idioma_tickets`, sin rutas de archivo) y notas técnicas.
     - Bug: problema, a quién afecta, comportamiento esperado, criterios de aceptación.
     - Feature o mejora: historia ("Como <rol> quiero <algo> para <beneficio>"), problema de fondo, propuesta funcional, fuera de alcance, criterios de aceptación.
   - En re-triage: no reescribas el ticket; resumí en 2–3 líneas qué cambia para el usuario, y sumá la evidencia, el PR si hay y lo que falta.
   - Notas técnicas: causa o propuesta con `archivo:línea`, confianza, esfuerzo, riesgo y tests a agregar.
5. **Orden sugerido** en una línea, y una tabla de decisiones vacía para el dev.

P2 y P3 van solo como filas de tabla. Pasale al usuario el link y lo más importante, no el backlog entero en el chat.

## 7. Aprobación del dev

El dev a cargo (`aprobacion.dev_a_cargo`, o quien diga el usuario) decide por ítem: **aprobar**, **cambiar prioridad**, **pedir más info**, **cerrar** o **descartar**. Para features, si la config nombra a alguien de producto, esa persona aprueba el alcance.

- Si hay que mandarle el backlog, redactá el mensaje y pedí OK al usuario antes de enviar nada.
- Registrá en el backlog la decisión y quién la tomó. No cambies prioridades por tu cuenta después.

## 8. Aplicar lo aprobado

Solo con la aprobación:

- **Reportes nuevos:** un ticket por ítem aprobado (épica si son varios del mismo tema), con label de tipo y de prioridad. En GitHub: `gh issue create --repo <repo> --title ... --label ... --body-file ...` y verificá cada uno con `gh issue view`. En otro tracker, con su conector. Sin tracker, entregá un archivo listo para importar (CSV o markdown).
- **Re-triage:** aplicá las labels de prioridad, cerrá los issues aprobados como hechos con un comentario que cite la evidencia, y actualizá las épicas.
- Si las labels de la config no existen, pedí OK antes de crearlas.
- Actualizá el backlog con los links.

**Ítems con PR listo:** al cerrar, ofrecé revisarlos: diff, migraciones (que lo que borran o cambian exista y no lo use la app), conflictos entre PRs (merge local de todos juntos), tests y typecheck. Reportá si están para mergear y qué hay que chequear en producción antes. **Nunca mergees ni apliques migraciones sin OK explícito.**

## Trampas

- El backlog es una propuesta. La prioridad final es del dev; no la presentes como decidida.
- El código manda sobre los comentarios del issue (ver paso 3).
- El agente no ve producción: lo que depende de ella va como pregunta, no como hecho.
- Un pedido de feature trae una solución; buscá el problema de fondo antes de estimar.
- No mezcles análisis con implementación. Si piden "hacelo", el ítem aprobado es el alcance.
- Datos confidenciales (personales, salariales, keys, tokens) no van en el backlog, los tickets ni los ejemplos.
- Un build verde no prueba que el bug esté resuelto: se verifica donde se reportó.

## Al terminar

Pasá el link al backlog, lo más importante, las preguntas que bloquean y el orden sugerido. Si hay ítems con PR listo, ofrecé revisarlos.
