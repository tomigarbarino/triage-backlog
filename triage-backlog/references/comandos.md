# Comandos (GitHub CLI y git)

Reemplazá `<repo>` por `repos.<capa>.github` o `tracker.repo_issues` según el caso. Siempre con `--repo`: `gh` resuelve el repo por el remoto del directorio actual y, con repos separados, mira el equivocado sin avisar.

**Windows:** corré estos comandos en Git Bash. Si usás PowerShell 5.1: `&&` y `||` no existen (escribí `A; if ($LASTEXITCODE -eq 0) {'sí'} else {'no'}`); `>` escribe con BOM y `json.load` rechaza el archivo (usá `[IO.File]::WriteAllText("<ruta>", (gh ... | Out-String))`, o `Out-File -Encoding utf8NoBOM` en PowerShell 7); las comillas dobles dentro de `--jq` se escapan como `\"` (los filtros de `--jq` de abajo evitan comillas internas; los de `jq` en «Tandas» las usan: escapalas igual); `curl` es un alias de `Invoke-WebRequest`: escribí `curl.exe`.

## 1. Juntar

- Antes de bajar nada, creá la carpeta: `mkdir -p <salida.backlog>/trabajo` (PowerShell: `New-Item -ItemType Directory -Force <salida.backlog>/trabajo`). Sin esto, la redirección falla.
- **Contar** (setup 0.3 y tope del paso 1). Abiertos: `gh issue list --repo <repo> --state open --limit 500 --json number --jq length`. Por label: `gh issue list --repo <repo> --state open --limit 500 --json labels --jq '[.[].labels[].name] | group_by(.) | map({(.[0]): length}) | add'`.
- Issues abiertos, lista liviana a archivo (la coma dentro de `label:` es «cualquiera de estas»; nunca varios `--label` en una llamada, filtran por Y):
  `gh issue list --repo <repo> --state open --limit 500 --search 'label:bug,enhancement sort:updated-desc' --json number,title,labels,createdAt,updatedAt,url > <salida.backlog>/trabajo/issues.json`
  Sin filtro de label: sacá `label:...`. Solo lo actualizado desde una fecha: `--search 'updated:>=AAAA-MM-DD sort:updated-desc'`. `sort:updated-desc` ordena el listado; no sirve para partir tandas.
- Un issue completo (desde cada lote): `gh issue view <N> --repo <repo> --json title,body,labels,comments,state,url`
- PRs abiertos, lista liviana sin `body` (el cuerpo trae pasos de fallas y URLs internas; se lee por PR desde el lote con `gh pr view <N> --repo <repo> --json body`): `gh pr list --repo <repo> --state open --limit 200 --json number,title,headRefName,baseRefName,isDraft,reviewDecision,statusCheckRollup,mergeable,files,closingIssuesReferences,updatedAt > <salida.backlog>/trabajo/prs-abiertos.json`
- PRs mergeados desde una fecha: `gh pr list --repo <repo> --state merged --limit 200 --search 'merged:>=AAAA-MM-DD' --json number,title,mergedAt,mergeCommit,baseRefName,closingIssuesReferences > <salida.backlog>/trabajo/prs-mergeados.json`
- Repo y visibilidad: `gh repo view <repo> --json nameWithOwner,defaultBranchRef,hasIssuesEnabled,visibility`
- **Ramas extra** (setup 0.1). Candidatas: `git branch -r | grep -E 'release/|hotfix/|prod'`. Bases de los PRs: `gh pr list --repo <repo> --state all --limit 200 --json baseRefName --jq '[.[].baseRefName] | group_by(.) | map({(.[0]): length}) | add'`. Commits que no están en la rama viva: `git rev-list --count origin/<rama_viva>..origin/<rama>` (más de 0: preguntá si es una segunda línea desplegada).
- **Tandas** (paso 1). Criterio estable, nunca `updatedAt`. Partí sobre `issues.json` ya bajado, sin volver a llamar a `gh`:
  - por rango de números: `jq '[.[] | select(.number >= 1 and .number <= 80) | .number]' <salida.backlog>/trabajo/issues.json`;
  - por label: `jq '[.[] | select(any(.labels[]; .name == "bug")) | .number]' …` (en PowerShell, `\"bug\"`);
  - por antigüedad: `jq '[.[] | select(.createdAt < "2026-06-01") | .number]' …` (en PowerShell, `\"2026-06-01\"`).
  Escribí `<salida.backlog>/trabajo/tandas-<fecha>.json`:
  ```json
  { "fecha": "2026-10-17", "total": 70, "criterio": "rango de números, 10 por tanda", "tandas": [ { "n": 1, "de": 7, "issues": [12, 15, 18] } ] }
  ```
  Las tandas siguientes se leen de este archivo, no se recalculan; los issues nuevos van a la última.

## 2. Duplicados

`gh issue list --repo <repo> --state all --search '<palabras> in:title' --limit 20 --json number,title,state,url`, en `tracker.repo_issues` y en cada `fuentes[].repo` de tipo `github_issues`.

## 3. Verificar lo que citan los comentarios

- Un PR: `gh pr view <N> --repo <repo> --json state,mergedAt,mergeCommit,baseRefName,headRefName`
- Un commit está en la rama viva: `git merge-base --is-ancestor <sha> origin/<rama_viva>`; exit code 0 = sí, 1 = no (bash: `echo $?`; PowerShell: `$LASTEXITCODE`). Para una rama extra, lo mismo con `origin/<rama_extra>`: si está en la viva y no en la extra, el estado es «hecho en <rama_viva>; backport pendiente en <rama_extra>».
- Ramas ya integradas: `git branch -r --merged origin/<rama_viva>`
- Cuándo entró o salió un símbolo: `git log origin/<rama_viva> -S'<símbolo>' --oneline -- <ruta>`
- Commits recientes en una zona: `git log origin/<rama_viva> --since=30.days --oneline -- <ruta>`
- Qué cambió desde el backlog anterior: `git diff --stat <commit_anterior>..origin/<rama_viva> -- <rutas>`
- Leer un archivo de la rama viva sin cambiar de rama: `git show origin/<rama_viva>:<ruta>` (ruta relativa al root del repo)
- Worktree para los lotes: `git fetch origin <rama_viva> && git worktree add --detach ../<app>-<capa>-triage origin/<rama_viva>` (uno por repo: `<capa>` = front, back o el `nombre` de `repos.otros`); al terminar, `git worktree remove ../<app>-<capa>-triage`

## 4. Aplicar (solo con OK, paso 8)

- Crear: `gh issue create --repo <repo> --title "<título>" --label <tipo> --label <prioridad> --body-file <archivo>`
- Labels de prioridad: `gh issue edit <N> --repo <repo> --add-label P1 --remove-label P2`
- Cerrar hecho: `gh issue close <N> --repo <repo> --reason completed --comment "Triage <fecha>: resuelto en <commit> (archivo:línea), verificado contra <rama_viva>"`
- Cerrar obsoleto o descartado: `gh issue close <N> --repo <repo> --reason "not planned" --comment "Triage <fecha>: <motivo en una línea>"`
- Cerrar duplicado (solo con decisión del dev): `gh issue close <N> --repo <repo> --reason "not planned" --comment "Triage <fecha>: duplicado de #M"`
- Comentar (duplicados, pedir más info): `gh issue comment <N> --repo <repo> --body-file <archivo>`
- Verificar: `gh issue view <N> --repo <repo> --json state,labels,url`
- Labels (setup y paso 8): `gh label list --repo <repo> --limit 200 --json name,description`; crear una por una: `gh label create <nombre> --repo <repo> --color <hex> --description "<...>"`. Nunca `--force`, `gh label edit` ni `gh label delete`.

## 5. Chequeos GET a producción (solo lectura, sin sesión)

- Código de respuesta de una URL pública (GET, sin sesión). Bash (macOS, Linux, Git Bash): `curl -s -o /dev/null -w '%{http_code}' <url>`. PowerShell: `curl.exe -s -o NUL -w '%{http_code}' <url>`. No uses `-I` ni `-Method Head`: mandan HEAD, no GET, y algunos servicios responden distinto. Evitá `Invoke-WebRequest` en PowerShell 5.1: con 4xx o 5xx tira una excepción y no devuelve el código.
- Corré uno por `entornos.prod` y por cada `entornos.chequeos_get[].url`. Anotá «verificado el <fecha>: <url> → <código>». Un 200 solo prueba que responde: si no confirma ni descarta lo que se busca, va a «Preguntas que bloquean» como «no concluyente: <qué se vio>», nunca como hecho.
