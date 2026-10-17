# Ejemplos de triage.yml

Puntos de partida para dos formas comunes de repo. Antes de guardarlos, confirmá las rutas leyendo el código actual. Opcionales: `repos.otros`, `repos.<capa>.local` (vacío = el repo ancla; obligatorio si esa capa vive en otro repo), `fuentes` (`fuentes: []` si los reportes siempre se pegan en el chat), `aprobacion.producto`, `aprobacion.canal`, `prioridad.features_p1`, `prioridad.notas`, `limites`, `salida`, `entornos`. El resto es obligatorio.

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
    carpetas: { endpoints: app/api, servicios: lib, datos: supabase/migrations }
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
  backlog: docs/backlog/
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
