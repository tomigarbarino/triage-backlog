# Comandos (GitHub CLI y git)

Reemplazá `<repo>` por `repos.<capa>.github` o `tracker.repo_issues` según el caso. Siempre con `--repo`: `gh` resuelve el repo por el remoto del directorio actual y, con repos separados, mira el equivocado sin avisar.

## 1. Juntar

- Issues abiertos, lista liviana a archivo (la coma dentro de `label:` es «cualquiera de estas»; nunca varios `--label` en una llamada, filtran por Y):
  `gh issue list --repo <repo> --state open --limit 500 --search 'label:bug,enhancement sort:updated-desc' --json number,title,labels,createdAt,updatedAt,url > <salida.backlog>/trabajo/issues.json`
  Sin filtro de label: sacá `label:...`. Solo lo actualizado desde una fecha: `--search 'updated:>=AAAA-MM-DD sort:updated-desc'`.
- Un issue completo (desde cada lote): `gh issue view <N> --repo <repo> --json title,body,labels,comments,state,url`
- PRs abiertos: `gh pr list --repo <repo> --state open --limit 200 --json number,title,body,headRefName,baseRefName,isDraft,reviewDecision,statusCheckRollup,mergeable,files,closingIssuesReferences,updatedAt`
- PRs mergeados desde una fecha: `gh pr list --repo <repo> --state merged --limit 200 --search 'merged:>=AAAA-MM-DD' --json number,title,mergedAt,mergeCommit,closingIssuesReferences`
- Repo y visibilidad: `gh repo view <repo> --json nameWithOwner,defaultBranchRef,hasIssuesEnabled,visibility`

## 2. Duplicados

`gh issue list --repo <repo> --state all --search '<palabras> in:title' --limit 20 --json number,title,state,url`, en `tracker.repo_issues` y en cada `fuentes[].repo` de tipo `github_issues`.

## 3. Verificar lo que citan los comentarios

- Un PR: `gh pr view <N> --repo <repo> --json state,mergedAt,mergeCommit,baseRefName,headRefName`
- Un commit está en la rama viva: `git merge-base --is-ancestor <sha> origin/<rama_viva> && echo sí || echo no`
- Ramas ya integradas: `git branch -r --merged origin/<rama_viva>`
- Cuándo entró o salió un símbolo: `git log origin/<rama_viva> -S'<símbolo>' --oneline -- <ruta>`
- Commits recientes en una zona: `git log origin/<rama_viva> --since=30.days --oneline -- <ruta>`
- Qué cambió desde el backlog anterior: `git diff --stat <commit_anterior>..origin/<rama_viva> -- <rutas>`
- Leer un archivo de la rama viva sin cambiar de rama: `git show origin/<rama_viva>:<ruta>` (ruta relativa al root del repo)
- Worktree para los lotes: `git fetch origin <rama_viva> && git worktree add --detach ../<app>-triage origin/<rama_viva>`; al terminar, `git worktree remove ../<app>-triage`

## 4. Aplicar (solo con OK, paso 8)

- Crear: `gh issue create --repo <repo> --title "<título>" --label <tipo> --label <prioridad> --body-file <archivo>`
- Labels de prioridad: `gh issue edit <N> --repo <repo> --add-label P1 --remove-label P2`
- Cerrar hecho: `gh issue close <N> --repo <repo> --reason completed --comment "Triage <fecha>: resuelto en <commit> (archivo:línea), verificado contra <rama_viva>"`
- Cerrar obsoleto o descartado: `gh issue close <N> --repo <repo> --reason "not planned" --comment "Triage <fecha>: <motivo en una línea>"`
- Cerrar duplicado (solo con decisión del dev): `gh issue close <N> --repo <repo> --reason "not planned" --comment "Triage <fecha>: duplicado de #M"`
- Comentar (duplicados, pedir más info): `gh issue comment <N> --repo <repo> --body-file <archivo>`
- Verificar: `gh issue view <N> --repo <repo> --json state,labels,url`
- Labels (setup y paso 8): `gh label list --repo <repo> --limit 200 --json name,description`; crear una por una: `gh label create <nombre> --repo <repo> --color <hex> --description "<...>"`. Nunca `--force`, `gh label edit` ni `gh label delete`.
