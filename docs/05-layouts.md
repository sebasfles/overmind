# Punto 5: Layouts de proyecto (single repo, monorepo, multirepo)

Estado: acordado el 2026-08-29.
Depende de: [02-orquestacion.md](02-orquestacion.md), [03-skills.md](03-skills.md), [04-operacion.md](04-operacion.md).
Donde este documento contradice a los anteriores, manda este.

## Objetivo

Un solo flujo para los tres layouts que Sebastian tiene:

- Single repo: un repo, una app.
- Monorepo: un repo con varias apps dentro (`/backend`, `/frontend`, `/infra`, o turborepo).
- Multirepo: una carpeta de proyecto con varios repos hermanos (`diy/diy-platform`, `diy/diy-infra`, ...).

Los tres se describen con la misma abstracción, y single y mono son casos con listas de un elemento.

## Abstracción

Un proyecto tiene `root`, `repos` y workspaces.

| Concepto | Single | Monorepo | Multirepo |
|---|---|---|---|
| `root`: cwd del om-manager, dueño de `docs/`, siempre un repo git | el repo | el repo | la carpeta del proyecto, convertida en repo git de docs con los clones ignorados |
| `repos`: repos de código | `[.]` | `[.]` | `[diy-platform, diy-infra, ...]` |
| Workspace de una task | `.workspaces/{{task}}/{{repo}}/` | idem | `.workspaces/{{task}}/{{repo}}/` por repo tocado, más `.workspaces/{{task}}/{{root}}/` |
| `docs/` y `docs/tasks/` | en el root | en el root | en el root |
| Targets de `verify-task` | 1 | N, uno por app, declarados en el TRD | N, uno por repo, cada uno con su sección del TRD |
| PRs de código | 1 | 1 | uno por repo tocado |
| PR de la task (lleva el resumen) | el mismo PR de código | el mismo | el PR del repo root |

## El repo root en multirepo

La carpeta del proyecto se inicializa como repo git que trackea solo `CLAUDE.md`, `docs/` y `.gitignore`.
Los clones de código y `.workspaces/` van en `.gitignore`; git no mira dentro de rutas ignoradas, así que los repos anidados no molestan.
Remoto privado personal, nombrado `{{proyecto}}-docs` (por ejemplo `fless/diy-docs`); la carpeta local sigue llamándose `{{proyecto}}`.
`add-project` lo crea cuando detecta un multirepo sin `.git` en la raíz, preguntando el remoto.

```
diy/                      # repo diy-docs
  .gitignore              # diy-platform/ diy-infra/ diy-pocs/ .workspaces/
  CLAUDE.md
  docs/
  diy-platform/           # clon, ignorado
  diy-infra/              # clon, ignorado
  .workspaces/            # ignorado
```

En single y mono no aplica: el repo de código ya es el root.

## Workspaces

`{{root}}/.workspaces/{{id}}_{{title}}/` es el cwd del om-reviewer y del om-developer de la task.
Contiene un worktree por repo tocado, todos en la misma rama `{{prefix}}/{{id}}_{{title}}`, y en multirepo además el worktree del root.
Reemplaza a `{{repo}}/.claude/worktrees/`.

- Se crean con `git -C {{repo}} worktree add -b {{rama}} {{root}}/.workspaces/{{task}}/{{repo}} origin/{{base}}`.
  Un worktree comparte `.git` con su clon; no es una copia.
- En single y mono el workspace queda dentro del repo; `.workspaces/` va al `.gitignore` del repo.
- Bootstrap por worktree, en `consolidate-task`: copiar o enlazar los archivos no versionados que el repo necesita (`.env*` y lo que el TRD liste en `Workspace files`) y correr el comando de instalación del TRD.
  Sin esto, la primera `verify-task` falla por razones ajenas a la task.
- Rutas absolutas siempre: el shell no conserva el `cd` entre comandos.
- Al limpiar: `git -C {{repo}} worktree remove {{ruta}}` por cada uno y borrar la carpeta del workspace.

## Documentación

`docs/` pertenece al root y describe la aplicación completa, no un repo.
Los módulos son de la aplicación: `docs/modules/billing/trd.md` tiene una sección por repo o app que participa.
El TRD general tiene una sección por repo o app: stack, rama base, `Verification targets`, `Workspace files`, instalación.

La regla de escritura de la carpeta de la task no cambia, solo se lee sobre el root:

- Antes de `delegate-task`: en el checkout del root.
- `consolidate-task` termina con el commit `docs(tasks): {{task}} planned` en la rama base del root, con aprobación de Sebastian.
- `delegate-task` rebasea **el worktree del root** sobre `origin/{{base}}`; en single y mono ese worktree es el del código.
- Después: en el worktree del root dentro del workspace. Viaja en el PR del root.

## Verificación

`verify-task` itera los `Verification targets` del TRD: cada uno con nombre, ruta (relativa al workspace) y comandos de lint, typecheck, unit y e2e con su flag serial.
Single: un target.
Monorepo: uno por app.
Multirepo: uno por repo tocado.
Un bloque por target en `verify.log`.

## Publicación

`publish-task` abre un PR por repo tocado, más el del root en multirepo.
El resumen (Intent, What changed, Decisions, Risk, Pipeline por target) vive en **el PR del repo root**: en single y mono es el mismo PR de código; en multirepo es el PR de `{{proyecto}}-docs`.
Los PRs de código en multirepo llevan un body de una línea con el link al PR del root.
`What changed` enlaza cada PR de código.
Si hay orden de merge entre repos (infra antes que plataforma), va en `Decisions`; Sebastian mergea en ese orden y al final el del root.
No existe el concepto de repo primario.

## Estado derivado

Igual que en el `02`, evaluado sobre todos los PRs de la task:

- `in_review`: algún PR abierto.
- `merged`: todos mergeados, incluido el del root, y el workspace aún existe.
- `done`: todos mergeados y sin workspace.

## Registro (`portfolio/projects.yaml`)

```yaml
projects:
  - name: diy
    root: /home/fless/dev/designli/projects/diy
    tmux: diy
    status: active
    repos:
      - name: diy-platform
        base_branch: develop
      - name: diy-infra
        base_branch: main
  - name: backend-school
    root: /home/fless/dev/designli/projects/backend-school
    tmux: backend-school
    status: active
    repos:
      - name: .
        base_branch: main
```

`task.md` gana `repos:` con los repos que toca (default: todos los de la lista cuando hay uno solo; obligatorio elegir en multirepo).

## Cambios en lo escrito

- `02`: carpeta de task, worktree, ciclo y limpieza se leen sobre root y workspaces; `.claude/worktrees/` desaparece.
- `03`: `verify-task` por targets; `publish-task` PR por repo con resumen en el root.
- `04`: registro con `root` y `repos`; `resume-project` abre el root.
- Skills: `add-project`, `resume-project`, `setup`, `write-trd`, `create-task`, `consolidate-task`, `delegate-task`, `start-task`, `verify-task`, `publish-task`, `check-task`, `clean-task`.
- Agentes: regla de rutas absolutas en `om-manager`, `om-reviewer`, `om-developer`; cwd del workspace en `om-reviewer` y `om-developer`.
