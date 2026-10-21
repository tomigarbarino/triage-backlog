# Comandos (GitHub CLI, git y equivalencias)

Reemplazá `<repo>` por `repos.<capa>.github` o `tracker.repo_issues` según el caso. Siempre con `--repo`: `gh` resuelve el repo por el remoto del directorio actual y, con repos separados, mira el equivocado sin avisar. GitLab, Jira, Linear o sin tracker: §6.

**Texto de terceros fuera de la shell.** Nunca metas texto de un issue, un PR, un comentario o una rama de terceros en una línea de comando: un título o una rama con comillas, `$(…)` o `;` ejecuta en tu shell. Las palabras de búsqueda las elegís vos, solo alfanuméricas; las ramas de PRs se citan por SHA (`headRefOid`), no por nombre; los cuerpos van por archivo (`--body-file`, `-F body=@<archivo>`).

**Windows:** corré estos comandos en Git Bash. Si usás PowerShell 5.1: `&&` y `||` no existen (escribí `A; if ($LASTEXITCODE -eq 0) {'sí'} else {'no'}`); `>` escribe con BOM y `json.load` rechaza el archivo (usá `[IO.File]::WriteAllText("<ruta>", (gh ... | Out-String))`, o `Out-File -Encoding utf8NoBOM` en PowerShell 7); las comillas dobles dentro de `--jq` se escapan como `\"` (los filtros de `--jq` de abajo evitan comillas internas; los de `jq` en «Tandas» las usan: escapalas igual); `curl` es un alias de `Invoke-WebRequest`: escribí `curl.exe`.

## 1. Juntar

- Antes de bajar nada, creá la carpeta: `mkdir -p <salida.backlog>/trabajo` (PowerShell: `New-Item -ItemType Directory -Force <salida.backlog>/trabajo`). Sin esto, la redirección falla.
- **Contar** (setup 0.3 y tope del paso 1). Total real de abiertos, sin tope: `gh api graphql -F owner=<owner> -F name=<nombre> -f query='query($owner:String!,$name:String!){repository(owner:$owner,name:$name){issues(states:OPEN){totalCount}}}' --jq .data.repository.issues.totalCount`. Por label: `gh issue list --repo <repo> --state open --limit <total> --json labels --jq '[.[].labels[].name] | group_by(.) | map({(.[0]): length}) | add'`. Si una lista trae exactamente `--limit` ítems, está cortada: subí el límite por encima del total.
- Issues abiertos, lista liviana a archivo (la coma dentro de `label:` es «cualquiera de estas»; nunca varios `--label` en una llamada, filtran por Y):
  `gh issue list --repo <repo> --state open --limit <total> --search 'label:bug,enhancement sort:updated-desc' --json number,title,labels,createdAt,updatedAt,url > <salida.backlog>/trabajo/issues.json`
  Sin filtro de label: sacá `label:...`. Solo lo actualizado desde una fecha: `--search 'updated:>=AAAA-MM-DD sort:updated-desc'`. Solo lo creado desde un momento (post-release): `--search 'created:>=AAAA-MM-DDTHH:MM:SS-03:00 sort:created-asc'`. `sort:updated-desc` ordena el listado; no sirve para partir tandas. `--search` pasa por la API de búsqueda, que corta en 1000 resultados: con más, sacá `--search` y filtrá labels con jq sobre el archivo.
- Cuerpo y comentarios de cada ítem. Los baja el agente principal; los lotes los leen con Read y no usan `gh`: `mkdir -p <salida.backlog>/trabajo/cuerpos` y `gh issue view <N> --repo <repo> --json title,body,labels,comments,state,url > <salida.backlog>/trabajo/cuerpos/<N>.json`. El de un PR, si hace falta: `gh pr view <N> --repo <repo> --json body > <salida.backlog>/trabajo/cuerpos/pr-<N>.json`. `cuerpos/` se borra al terminar la corrida.
- PRs abiertos, lista liviana sin `body` (el cuerpo trae pasos de fallas y URLs internas; si hace falta, va a `cuerpos/pr-<N>.json`, arriba): `gh pr list --repo <repo> --state open --limit 200 --json number,title,headRefName,headRefOid,isCrossRepository,baseRefName,isDraft,reviewDecision,statusCheckRollup,mergeable,files,closingIssuesReferences,updatedAt > <salida.backlog>/trabajo/prs-abiertos.json`. `isCrossRepository: true` = PR de fork: solo diff y CI. «PR listo» exige `mergeable` distinto de `CONFLICTING` y `updatedAt` de los últimos 90 días.
- PRs mergeados desde una fecha: `gh pr list --repo <repo> --state merged --limit 200 --search 'merged:>=AAAA-MM-DD' --json number,title,mergedAt,mergeCommit,baseRefName,closingIssuesReferences > <salida.backlog>/trabajo/prs-mergeados.json`
- ¿Hay PRs? (modo rápido): `gh pr list --repo <repo> --state all --limit 1 --json number --jq length`. Si da 0, no se bajan los JSON de PRs y el encabezado lo dice.
- **Commits que citan issues** (hechos viejos que no cerraron el issue): `git log origin/<rama_viva> --date=short --format='%h %ad %s %b' -E --grep='#[0-9]+'`. Cruzá los números con `issues.json`: cada coincidencia es «candidato a hecho (el commit X cita #N)» y el lote la verifica en el código.
- Repo y visibilidad: `gh repo view <repo> --json nameWithOwner,defaultBranchRef,hasIssuesEnabled,visibility`
- **Reporte privado de vulnerabilidades** (repos públicos, adaptaciones §6): `gh api repos/<repo>/private-vulnerability-reporting --jq .enabled` (`true` = habilitado). Si da 403 o 404 (requiere permisos de admin), buscá `SECURITY.md` en la raíz, `.github/` o `docs/` con `git show origin/<rama_viva>:<ruta>` y anotá «no verificado» en el setup.
- **Ramas extra** (setup 0.1). Candidatas: `git branch -r | grep -E 'release/|hotfix/|prod'`. Bases de los PRs: `gh pr list --repo <repo> --state all --limit 200 --json baseRefName --jq '[.[].baseRefName] | group_by(.) | map({(.[0]): length}) | add'`. Commits que no están en la rama viva: `git rev-list --count origin/<rama_viva>..origin/<rama>` (más de 0: preguntá si es una segunda línea desplegada).
- **Tandas** (paso 1). Criterio estable: `updatedAt` solo congelado como lista de números al partir, nunca recalculado. Con Jira, el orden del backlog (Rank) congelado como lista de claves. Los candidatos a duplicado (§2) van enteros a la misma tanda. Partí sobre `issues.json` ya bajado, sin volver a llamar a `gh`:
  - por rango de números: `jq '[.[] | select(.number >= 1 and .number <= 80) | .number]' <salida.backlog>/trabajo/issues.json`;
  - por label: `jq '[.[] | select(any(.labels[]; .name == "bug")) | .number]' …` (en PowerShell, `\"bug\"`);
  - por antigüedad: `jq '[.[] | select(.createdAt < "2026-06-01") | .number]' …` (en PowerShell, `\"2026-06-01\"`).
  Escribí `<salida.backlog>/trabajo/tandas-<fecha>.json`:
  ```json
  { "fecha": "2026-10-17", "total": 70, "criterio": "rango de números, 10 por tanda", "tandas": [ { "n": 1, "de": 7, "issues": [12, 15, 18], "backlog": null, "aplicada": null } ], "consolidado": null }
  ```
  Las tandas siguientes se leen de este archivo, no se recalculan; los issues nuevos van a la última. La skill completa `backlog` (archivo) al guardar el backlog de cada tanda y `aplicada` (fecha) al terminar su paso 8. Al leer una tanda, sacá los issues que ya no están abiertos y decilo en el encabezado. Cuando todas las tandas tienen `aplicada`, la corrida de consolidación (adaptaciones §7, «Cierre de la limpieza») completa `consolidado` con el archivo del backlog único; con `consolidado` lleno, la limpieza terminó.

## 2. Duplicados

`gh issue list --repo <repo> --state all --search '<palabras> in:title' --limit 20 --json number,title,state,url`, en `tracker.repo_issues` y en cada `fuentes[].repo` de tipo `github_issues`.

- Buscá con 2 o 3 variantes: palabras del usuario, pantalla y término técnico, en el idioma de los títulos del tracker (`idioma_tickets`); si difiere del reporte, probá en los dos. Las palabras las elegís vos, solo alfanuméricas.
- Contra el backlog anterior: los `R<n>` y `#N` de su tabla Decisiones.
- Re-triage con muchos ítems: no busques de a uno contra la API (admite unas 30 búsquedas por minuto). Listá `jq -r '.[] | [.number, .title] | @tsv' <salida.backlog>/trabajo/issues.json`, agrupá por tema en la pasada de metadatos y mandá cada grupo entero a la misma tanda. Entre dos abiertos se conserva el más antiguo, salvo que el otro tenga PR o más detalle; la fila lo dice.

## 3. Verificar lo que citan los comentarios

- Un PR: `gh pr view <N> --repo <repo> --json state,mergedAt,mergeCommit,baseRefName,headRefName`
- Un commit está en la rama viva: `git merge-base --is-ancestor <sha> origin/<rama_viva>`; exit code 0 = sí, 1 = no (bash: `echo $?`; PowerShell: `$LASTEXITCODE`). Para una rama extra, lo mismo con `origin/<rama_extra>`: si está en la viva y no en la extra, el estado es «hecho en <rama_viva>; backport pendiente en <rama_extra>».
- Ramas ya integradas: `git branch -r --merged origin/<rama_viva>`
- Cuándo entró o salió un símbolo: `git log origin/<rama_viva> -S'<símbolo>' --oneline -- <ruta>`
- Commits recientes en una zona: `git log origin/<rama_viva> --since=30.days --oneline -- <ruta>`
- Qué cambió desde el backlog anterior: `git diff --stat <commit_anterior>..origin/<rama_viva> -- <rutas>`
- Leer un archivo de la rama viva sin cambiar de rama: `git show origin/<rama_viva>:<ruta>` (ruta relativa al root del repo)
- Worktree para los lotes: `git fetch origin <rama_viva> && git worktree add --detach ../<app>-<capa>-triage origin/<rama_viva>` (uno por repo: `<capa>` = front, back o el `nombre` de `repos.otros`); al terminar, `git worktree remove ../<app>-<capa>-triage`
- Clonar una capa que falta (setup 0.1, con OK): `git clone --filter=blob:none --branch <rama_viva> <url> <carpeta>`: historia completa, archivos a demanda, sin `gh`. Nunca `--depth`: en un clon superficial, `log -S`, `merge-base`, los tags y `release/*` dan resultados falsos. ¿Superficial? `git rev-parse --is-shallow-repository` (`true` = sí; se arregla con `git fetch --unshallow`). Tags (versiones de una app): `git fetch --tags`; una ruta en una versión: `git show <tag>:<ruta>`; primera versión que trae un commit: `git tag --contains <sha> --sort=v:refname` (el primero de la lista).
- **Regresión o preexistente** (después de un release, adaptaciones §8; `v2.3.0` = tag anterior, `v2.4.0` = tag desplegado): `git fetch --tags`; qué cambió en el rango: `git diff --stat v2.3.0 v2.4.0` (solo migraciones: `-- <carpetas.datos>`); commits del rango en una ruta: `git log --oneline v2.3.0..v2.4.0 -- <ruta>`; la ruta en cada versión: `git show v2.3.0:<ruta>` y `git show v2.4.0:<ruta>`; commits de la rama viva que no están en el tag: `git rev-list --count v2.4.0..origin/<rama_viva>`; un arreglo ya está en el tag: `git merge-base --is-ancestor <sha> v2.4.0` (0 = sí, 1 = no).
- Merge local de varios PRs (revisión del paso 8), nunca en la rama de trabajo: `git worktree add --detach ../<app>-merge-check origin/<base>`; por cada PR, `git fetch origin pull/<N>/head` y `git -C ../<app>-merge-check merge --no-edit <headRefOid>`. GitLab: `git fetch origin refs/merge-requests/<iid>/head` y merge del `sha` del MR. Un conflicto = PRs que chocan. Al terminar, `git worktree remove --force ../<app>-merge-check`. PRs de forks: no se mergean localmente ni se corren sus tests.

## 4. Aplicar (solo con OK, paso 8)

- Crear: `gh issue create --repo <repo> --title "<título>" --label <tipo> --label <prioridad> --body-file <archivo>`
- Labels de prioridad: `gh issue edit <N> --repo <repo> --add-label P1 --remove-label P2`
- Cerrar hecho: `gh issue close <N> --repo <repo> --reason completed --comment "Triage <fecha>: resuelto en <commit> (archivo:línea), verificado contra <rama_viva>"`
- Cerrar obsoleto o descartado: `gh issue close <N> --repo <repo> --reason "not planned" --comment "Triage <fecha>: <motivo en una línea>"`
- Cerrar duplicado (solo con decisión del dev): `gh issue close <N> --repo <repo> --duplicate-of <M> --comment "Triage <fecha>: duplicado de #M"` (con un gh viejo sin `--duplicate-of`: `--reason "not planned"`)
- Comentar (duplicados, pedir más info): `gh issue comment <N> --repo <repo> --body-file <archivo>`. Al pedir más info, si existe `tracker.labels.espera`: `gh issue edit <N> --repo <repo> --add-label <espera>` (nunca se saca desde la skill).
- Asignar a un sprint (acción «sprint:», solo filas aprobadas en «Entra»): `gh issue edit <N> --repo <repo> --milestone "<nombre>"`. El milestone tiene que existir (`gh api repos/<repo>/milestones --jq '.[].title'`); si no existe, avisá y no lo crees sin OK. Nunca saques un ítem de un milestone en curso sin que la Decisión lo diga.
- Verificar: `gh issue view <N> --repo <repo> --json state,labels,url`
- **En bloques.** De a 20 a 50 acciones, con el avance a la vista. Verificá cada acción y completá su fila de Aplicado en ese momento. Ante un 403, un 429 u otro error, frená y decí en qué fila quedó; no reintentes en loop (GitHub corta las escrituras seguidas). Al retomar, verificá cada acción antes de repetirla.
- **Notificaciones.** Cada cierre con comentario notifica al autor y a los suscriptores: con más de 50 cierres, la lista de acciones lo avisa antes del OK. El spam se cierra sin comentario: `gh issue close <N> --repo <repo> --reason "not planned"`.
- **Antes del OK,** revisá cada archivo de cuerpo: sin texto de reportes, nombres ni valores de las columnas del export que identifican clientes o personas. En GitLab, ninguna línea empieza con `/` (§6).
- Labels (setup y paso 8): `gh label list --repo <repo> --limit 200 --json name,description`; crear una por una: `gh label create <nombre> --repo <repo> --color <hex> --description "<...>"`. Nunca `--force`, `gh label edit` ni `gh label delete`.

## 5. Chequeos GET a producción (solo lectura, sin sesión)

- Código de respuesta de una URL pública (GET, sin sesión). Bash (macOS, Linux, Git Bash): `curl -s -o /dev/null -w '%{http_code}' <url>`. PowerShell: `curl.exe -s -o NUL -w '%{http_code}' <url>`. No uses `-I` ni `-Method Head`: mandan HEAD, no GET, y algunos servicios responden distinto. Evitá `Invoke-WebRequest` en PowerShell 5.1: con 4xx o 5xx tira una excepción y no devuelve el código.
- Corré uno por `entornos.prod` y por cada `entornos.chequeos_get[].url`. Anotá «verificado el <fecha>: <url> → <código>». Un 200 solo prueba que responde: si no confirma ni descarta lo que se busca, va a «Preguntas que bloquean» como «no concluyente: <qué se vio>», nunca como hecho.

## 6. Equivalencias por tracker y hosting

Todo termina en los mismos archivos de `trabajo/` (`issues.json`, `cuerpos/<ID>.json`, `prs-*.json`), así tope, tandas y duplicados funcionan igual. Jira y Linear se leen con el conector del agente o un export: los nombres de abajo son campos a pedir, no comandos exactos. Bitbucket u otro hosting sin CLI: sin PRs (paso 1).

**Leer (triage, solo lectura)**

| Paso | GitHub | GitLab | Jira | Linear | Sin tracker |
|---|---|---|---|---|---|
| Contar abiertos | `totalCount` (§1) | header `X-Total` (G1) | total del JQL en el conector o el export | total del filtro en el conector | reportes pegados o filas del export |
| Lista liviana → `issues.json` | §1 | G2 | clave, resumen, tipo, estado, prioridad, labels, épica, estimación, fechas; `number` = clave | identificador, título, estado, prioridad, labels, estimación, fechas | una entrada por reporte; `number` = `R<n>` |
| Acotar la corrida | `--search` (§1) | `labels=`, `created_after=` (G2) | `fuentes[].filtro` (JQL) | filtro del conector (equipo, estado, label) | — |
| Cuerpo → `cuerpos/` | §1 | G3 | descripción y comentarios por clave | descripción y comentarios | el reporte |
| Duplicados | §2 | `issues?search=<palabras>&in=title&state=all` | JQL `project = <CLAVE> AND summary ~ "<palabras>"` | búsqueda del conector | solo contra el backlog anterior |
| PRs o MRs | §1 | G4 | los del hosting del código | los del hosting del código | los del hosting del código |
| Qué cubre un PR o MR | `closingIssuesReferences` → `closes #N` → rama o título → contenido («probable») | `closes_issues` → patrón de cierre en la descripción → número inicial de la rama → contenido | clave en rama, título o commits (J1) | identificador en rama, título o commits (J1) | — |
| Cómo se escribe | issue y PR `#N` | issue `#N`, MR `!N` | `CLAVE-N` | `EQP-N` | `R<n>` |

**Escribir (solo con OK, paso 8)**

| Acción | GitHub | GitLab | Jira | Linear | Sin tracker |
|---|---|---|---|---|---|
| Crear | §4 | G5 | conector: tipo de `tracker.labels`, Priority de `prioridad.escala` | conector, prioridad nativa | ticket en markdown, en el chat o un archivo |
| Prioridad | label (§4) | G6 | campo Priority (`prioridad_en: campo`) | campo prioridad (Urgent, High, Medium, Low) | en el backlog |
| Cerrar hecho | `--reason completed` + comentario | comentario + cierre (G7) | transición `tracker.transiciones.hecho` | estado completado del equipo | — |
| Cerrar obsoleto o descartado | `--reason "not planned"` + comentario | comentario + cierre; label `tracker.labels.descartado` si existe | transición `tracker.transiciones.descartado` | estado cancelado | — |
| Duplicado | comentario en el original + `--duplicate-of` | comentario + cierre; `/duplicate #M` solo si el dev lo pide (excepción de G8) | link «duplicates» + transición `descartado`, si el conector lo permite | marcar duplicado, si el conector lo permite; si no, comentario + cancelado | — |
| Comentar | `--body-file` | G8; ninguna línea empieza con `/` | Jira Service Management: siempre nota interna (si el conector no puede, no se comenta y se avisa); Jira Software: comentario normal o con visibilidad restringida a un rol del proyecto | lo ve todo el workspace: sin detalle de seguridad | «Respuestas sugeridas» del backlog |
| Verificar | `gh issue view --json state,labels,url` | `GET` del issue (`state`, `labels`) | leer el ítem (estado, resolución) | leer el ítem | — |
| Label o tipo faltante | crear una por una, con OK (§4) | `POST projects/<ruta>/labels`, con OK | no se crea: avisá para que lo cree un admin | con OK, desde el conector | — |
| Asignar a sprint (adaptaciones §12) | milestone (§4) | milestone: `glab api -X PUT 'projects/<ruta>/issues/<iid>' -f milestone_id=<id>` | campo Sprint, vía conector; el sprint tiene que existir | Cycle, vía conector; el cycle tiene que existir | en el backlog |
| Mergear (solo con OK que nombre el PR o MR) | `gh pr merge <N> --repo <repo>` con el método del repo; nunca `--admin` | G9 | — | — | — |

Si `tracker.transiciones` falta, se pregunta al aplicar y la lista de acciones nombra la transición exacta. En Jira, una transición puede mandar mails a watchers o disparar automatizaciones: avisalo en la lista.

**GitLab.** Token `read_api` en la variable de entorno `GITLAB_TOKEN` para el triage; nunca en el chat, un comando, el brief ni la config. El de escritura lo carga el usuario recién en el paso 8. `<ruta>` es `grupo/subgrupo/proyecto` codificada (`grupo%2Fsubgrupo%2Fproyecto`); con instancia propia, sumá `--hostname <tracker.sitio>`. Nunca guardes la respuesta cruda: normalizá con jq antes de escribir.

- **G0 Repo:** `glab api 'projects/<ruta>' | jq '{default_branch, visibility, issues_enabled}'`. `internal` cuenta como público (lo ve toda la instancia).
- **G1 Contar:** `glab api -i 'projects/<ruta>/issues?state=opened&per_page=1'` y leé el header `X-Total` (con más de 10 000 ítems GitLab no lo manda: contá el export paginado).
- **G2 Lista liviana:** `glab api --paginate 'projects/<ruta>/issues?state=opened&per_page=100' | jq -s 'add | map({number: .iid, title, labels: [.labels[] | {name: .}], createdAt: .created_at, updatedAt: .updated_at, url: .web_url})' > <salida.backlog>/trabajo/issues.json`. `labels=a,b` filtra por Y: para «cualquiera», una llamada por label y `jq -s 'add | unique_by(.number)'`. Desde un momento: `&created_after=<ISO>`.
- **G3 Cuerpo:** `glab api 'projects/<ruta>/issues/<iid>' | jq '{title, body: .description, labels, state, confidential, url: .web_url}' > .../cuerpos/<iid>.json` y `glab api --paginate 'projects/<ruta>/issues/<iid>/notes?per_page=100' | jq -s 'add | map(select(.system | not) | {author: .author.username, createdAt: .created_at, internal, body})' > .../cuerpos/<iid>-notas.json`. Un issue confidencial o una nota interna va al backlog como «#N (confidencial)» y nunca se cita en otro issue.
- **G4 MRs:** `glab api --paginate 'projects/<ruta>/merge_requests?state=opened&per_page=100' | jq -s 'add | map({number: .iid, title, sourceBranch: .source_branch, headRefOid: .sha, baseRefName: .target_branch, isDraft: .draft, mergeStatus: .detailed_merge_status, updatedAt: .updated_at, isCrossRepository: (.source_project_id != .target_project_id), author: .author.username})' > .../prs-abiertos.json`. `isCrossRepository: true` = MR de fork: solo diff y CI, como en GitHub.
  - Archivos de un MR: `glab api --paginate 'projects/<ruta>/merge_requests/<iid>/diffs?per_page=100' | jq -s 'add | map(.new_path)'` (solo rutas, nunca el diff a disco). Sirven para marcar «migración» (archivos bajo `carpetas.datos`).
  - Uno: `projects/<ruta>/merge_requests/<iid>`. Checks = `head_pipeline.status`; review = `.../approvals` (`approved`); conflictos = `detailed_merge_status` igual a `conflict` o `broken_status`; qué cubre = `.../closes_issues`.
  - Mergeados: `glab api --paginate 'projects/<ruta>/merge_requests?state=merged&updated_after=<ISO>&per_page=100' | jq -s 'add | map(select(.merged_at >= "<ISO>")) | map({number: .iid, title, mergedAt: .merged_at, mergeCommit: (.merge_commit_sha // .squash_commit_sha // .sha), baseRefName: .target_branch})' > .../prs-mergeados.json`. Con squash, el commit en la rama viva es `squash_commit_sha`; con fast-forward, `sha`.
- **G5 Crear:** `glab api -X POST 'projects/<ruta>/issues' -f title='<título>' -f labels='<tipo>,<prioridad>' -F description=@<archivo>`
- **G6 Prioridad:** `glab api -X PUT 'projects/<ruta>/issues/<iid>' -f add_labels=P1 -f remove_labels=P2`
- **G7 Cerrar:** primero el comentario (G8), después `glab api -X PUT 'projects/<ruta>/issues/<iid>' -f state_event=close`. No hay motivo de cierre: lo dice el comentario.
- **G8 Comentar:** `glab api -X POST 'projects/<ruta>/issues/<iid>/notes' -F body=@<archivo>`. **Quick actions:** GitLab ejecuta cualquier línea que empiece con `/` (`/close`, `/move`, `/label`), también por API. Ninguna línea de lo que escribe la skill empieza con `/` (salvo `/duplicate #M` con las condiciones de adaptaciones §5); si hay que citar algo así, va en bloque de código. La lista de acciones muestra el texto final tal cual se manda.
- **G9 Mergear** (solo con OK que nombre `!N`, después de la revisión del paso 8): `glab api -X PUT 'projects/<ruta>/merge_requests/<iid>/merge' -f sha=<sha revisado>`. Sin `merge_when_pipeline_succeeds` ni saltear aprobaciones; nunca `should_remove_source_branch` sin pedirlo. `#N` es un issue: un OK que dice «#N» sobre un MR es una pregunta, no un OK.

**Jira o Linear con código en GitHub.** `closingIssuesReferences` viene vacío.

- **J1 Cobertura:** un PR cubre el ítem si la clave está en la rama o el título (`jq '[.[] | select((.title + " " + .headRefName) | test("APP-[0-9]+"))]' <salida.backlog>/trabajo/prs-abiertos.json`) o en sus commits (`git log origin/<rama_viva> --oneline -E --grep='APP-[0-9]+'`), con la clave de tu proyecto en lugar de `APP`.
