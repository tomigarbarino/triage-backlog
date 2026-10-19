# Ejemplos de triage.yml

Puntos de partida para dos formas comunes de repo. Antes de guardarlos, confirmá las rutas leyendo el código actual. Opcionales: `generado`, `repos.otros`, `repos.<capa>.local` (vacío = el repo ancla; obligatorio si esa capa vive en otro repo), `repos.<capa>.ramas_extra`, `fuentes` (`fuentes: []` si los reportes siempre se pegan en el chat), `aprobacion.producto`, `aprobacion.canal`, `prioridad.features_p1`, `prioridad.notas`, `limites`, `salida`, `entornos` (`prod`, `staging` y `chequeos_get`; sin `prod` no hay chequeo de producción y se dice en el encabezado). El resto es obligatorio. Los comentarios de la config describen reglas, no conteos ni fechas.

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
    carpetas: { pantallas: app, componentes: components }
  back:
    github: mi-org/mi-app
    rama_viva: main
    ramas_extra: [{ rama: release/cliente-x, criterio: "issues con label cliente-x" }]   # segunda línea desplegada: hecho en main + backport; en un monorepo, repetila en cada capa que comparte el repo
    carpetas: { endpoints: app/api, servicios: lib, datos: supabase/migrations }
  otros:
    - nombre: worker
      github: mi-org/mi-app
      rama_viva: main
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
