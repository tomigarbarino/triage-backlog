# Ejemplos de triage.yml

Puntos de partida para dos formas comunes de repo. Antes de guardarlos, confirmá las rutas leyendo el código actual.

## Monorepo (front y back en el mismo repo)

```yaml
app: mi-app
idioma_tickets: es
fuentes:
  - tipo: github_issues
    repo: mi-org/mi-app
    labels: [bug, enhancement]
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
  labels: { bug: bug, feature: enhancement, mejora: enhancement, prioridad: [P0, P1, P2, P3] }
aprobacion:
  dev_a_cargo: "dev a cargo de la app"
prioridad:
  flujos_criticos: [login, flujo principal del cliente]
  features_p1: [destraba una venta, renovación o compromiso con fecha]
  siempre_p0: [un usuario ve datos de otro cliente]
```

## Front y back en repos separados, reportes en un canal

```yaml
app: mi-app
idioma_tickets: es
fuentes:
  - tipo: canal
    nombre: "#soporte-mi-app"          # se lee con el conector de Slack o Teams
  - tipo: planilla
    ubicacion: "export del formulario de pedidos"
repos:
  front:
    github: mi-org/mi-app-web
    rama_viva: main
    carpetas: { pantallas: src/pages, componentes: src/components, cliente_api: src/api }
  back:
    github: mi-org/mi-app-api
    rama_viva: main
    carpetas: { endpoints: src/routes, servicios: src/services, datos: migrations }
tracker:
  tipo: github
  repo_issues: mi-org/mi-app-web      # o el repo de la capa afectada
  labels: { bug: bug, feature: enhancement, mejora: enhancement, prioridad: [P0, P1, P2, P3] }
aprobacion:
  dev_a_cargo: "dev a cargo de la app"
  producto: "responsable de producto"
prioridad:
  flujos_criticos: [checkout, alta de usuarios]
  siempre_p0: [datos personales expuestos]
```
