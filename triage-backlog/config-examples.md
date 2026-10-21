# Ejemplos de triage.yml

Puntos de partida para formas comunes de repo. Antes de guardarlos, confirmá las rutas leyendo el código actual. Opcionales: `generado`, `repos.otros`, `repos.<capa>.local` (vacío = el repo ancla; obligatorio si esa capa vive en otro repo), `repos.<capa>.ramas_extra`, `repos.<capa>.remoto` (en lugar de `github` fuera de GitHub), `fuentes` (`fuentes: []` si los reportes siempre se pegan en el chat), `fuentes[].ventana_inicial`, `tracker.sitio`, `tracker.prioridad_en`, `tracker.transiciones`, `tracker.labels.descartado`, `tracker.labels.espera`, `aprobacion.producto`, `aprobacion.canal`, `prioridad.features_p1`, `prioridad.notas`, `prioridad.peso_por_plan`, `limites` (`max_items_por_corrida`, `dias_sin_respuesta`, `dias_sin_actividad`, `backlog_vivo`), `salida` (`backlog`, `detalle`, `idioma`), `entornos` (`prod`, `staging` y `chequeos_get`; sin `prod` no hay chequeo de producción y se dice en el encabezado), `seguridad`, `versiones`, `planificacion`. `repos` lleva al menos una capa de nombre libre, o `{}` sin código. El resto es obligatorio. Más casos (GitLab, sin código, open source, mobile, planificación de sprint), con su snippet: [references/adaptaciones.md](references/adaptaciones.md). Los comentarios de la config describen reglas, no conteos ni fechas.

## Monorepo (front y back en el mismo repo)

```yaml
app: mi-app
idioma_tickets: es
fuentes:
  - tipo: github_issues
    repo: mi-org/mi-app
    labels: [bug, enhancement]          # cualquiera de estas
repos:
  front:
    github: mi-org/mi-app
    rama_viva: main
    ramas_extra: [{ rama: release/cliente-x, criterio: "issues con label cliente-x" }]   # segunda línea desplegada: hecho en main + backport; en un monorepo, repetila en cada capa que comparte el repo
    carpetas: { pantallas: app, componentes: components }
  back:
    github: mi-org/mi-app
    rama_viva: main
    ramas_extra: [{ rama: release/cliente-x, criterio: "issues con label cliente-x" }]
    carpetas: { endpoints: app/api, servicios: lib, datos: supabase/migrations }
  otros:
    - nombre: worker
      github: mi-org/mi-app
      rama_viva: main
      ramas_extra: [{ rama: release/cliente-x, criterio: "issues con label cliente-x" }]
      carpetas: { entrada: worker/src, endpoints: worker/src/routes, servicios: worker/src/services }   # las claves de back más `entrada`, todas carpetas
tracker:
  tipo: github
  repo_issues: mi-org/mi-app
  labels: { bug: bug, feature: enhancement, mejora: enhancement, deuda: tech-debt, epica: epic }
aprobacion:
  dev_a_cargo: "dev a cargo de la app"
prioridad:
  escala: { P0: P0, P1: P1, P2: P2, P3: P3 }
  flujos_criticos: [login, flujo principal del cliente]
  features_p1: ["destraba una venta, renovación o compromiso con fecha"]
  siempre_p0: [un usuario ve datos de otro cliente]
salida:
  backlog: docs/backlog/                 # ignorada entera por git si el equipo no versiona backlogs
entornos:
  prod: https://mi-app.example.com
  chequeos_get: [{ nombre: worker, url: https://worker.mi-app.example.com/health }]
```

## Front y back en repos separados, reportes en un canal

El agente está abierto en el repo del front (repo ancla); el back vive en otro checkout.

```yaml
app: mi-app
idioma_tickets: es
fuentes:
  - tipo: canal
    nombre: "#soporte-mi-app"          # se lee con el conector de Slack o Teams
  - tipo: planilla
    ubicacion: pedir                   # el export del formulario se pide en cada corrida
repos:
  front:
    github: mi-org/mi-app-web
    rama_viva: main
    carpetas: { pantallas: src/pages, componentes: src/components, cliente_api: src/api }
  back:
    github: mi-org/mi-app-api
    local: ../mi-app-api               # checkout del back, relativo al repo ancla
    rama_viva: main
    carpetas: { endpoints: src/routes, servicios: src/services, datos: migrations }
tracker:
  tipo: github
  repo_issues: mi-org/mi-app-web      # los tickets se crean siempre acá; si tocan el back, se anota en el cuerpo
  labels: { bug: bug, feature: enhancement, mejora: enhancement, deuda: tech-debt, epica: epic }
aprobacion:
  dev_a_cargo: "dev a cargo de la app"
  producto: "responsable de producto"
prioridad:
  escala: { P0: "prioridad: urgente", P1: "prioridad: alta", P2: "prioridad: media", P3: "prioridad: baja" }
  flujos_criticos: [checkout, alta de usuarios]
  siempre_p0: [datos personales expuestos]
salida:
  backlog: docs/backlog/
```

## Repo único (CLI, librería o API), sin entornos

Una sola capa con nombre libre y sin labels. Sin `entornos`, el encabezado dice que no se chequeó producción.

```yaml
app: mi-cli
idioma_tickets: es
fuentes: []                            # los reportes se pegan en el chat
repos:
  app:
    github: mi-org/mi-cli
    rama_viva: main
    carpetas: { codigo: src, tests: test }
tracker:
  tipo: github
  repo_issues: mi-org/mi-cli
  labels: {}                           # sin labels de tipo
aprobacion:
  dev_a_cargo: "mantenedor"
prioridad:
  escala: {}                           # sin labels de prioridad: va en la primera línea del cuerpo
  flujos_criticos: [comando principal, instalación]
  siempre_p0: [pérdida de datos del usuario]
salida:
  backlog: docs/backlog/
  detalle: todos                       # pocos ítems: detalle para todos
```

## Código en GitHub, tickets en Jira

```yaml
app: mi-app
idioma_tickets: es
fuentes:
  - tipo: jira
    proyecto: APP
    filtro: "project = APP AND statusCategory != Done ORDER BY Rank"   # JQL: fija el alcance
repos:
  app:
    github: mi-org/mi-app
    rama_viva: main
    carpetas: { pantallas: src/pages, endpoints: src/api, servicios: src/services, datos: migrations }
tracker:
  tipo: jira
  sitio: jira.example.com              # sin tokens
  repo_issues: APP
  prioridad_en: campo                  # Priority es un campo, no una label
  labels: { bug: Bug, feature: Story, mejora: Story, deuda: Task, epica: Epic }   # tipos de issue; el setup no los crea
  transiciones: { hecho: Done, descartado: "Won't Do" }
aprobacion:
  dev_a_cargo: "dev a cargo de la app"
  producto: "responsable de producto"
prioridad:
  escala: { P0: Highest, P1: High, P2: Medium, P3: Low }   # valores del campo Priority
  flujos_criticos: [login, checkout]
  siempre_p0: [un usuario ve datos de otro cliente]
salida:
  backlog: docs/backlog/
  detalle: [P0, P1, a_definir]
```
