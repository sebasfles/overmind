# Punto 3: Skills por rol

Estado: acordado el 2026-08-28.
Depende de: [01-documentacion.md](01-documentacion.md), [02-orquestacion.md](02-orquestacion.md).

## Principios

1. Todas las skills viven en `~/.claude/skills/` (globales).
   El flujo es la convención de Sebastian, no del proyecto.
   El detalle por stack va en `references/{{stack}}.md` dentro de cada skill y en el TRD del proyecto.
2. Cada rol solo puede invocar sus skills.
   La definición del agente en `~/.claude/agents/{{rol}}.md` restringe qué skills tiene disponibles.
   El om-developer no puede correr `review-task`; el om-reviewer no puede correr `execute-task`.
   Es por diseño, no por confianza.
3. Las skills del om-manager las dispara Sebastian.
   Las que cambian estado (`consolidate-task`, `delegate-task`, `reiterate-task`, `clean-task`, `clean-work`) se marcan como solo invocables por el usuario.
   `check-task` y `check-work` pueden ser invocadas por el om-manager cuando Sebastian pregunta por el estado.
4. Las skills del om-reviewer y del om-developer se ejecutan automáticamente.
   om-reviewer y om-developer son máquinas de estado dirigidas por eventos.
   La máquina vive en el system prompt del agente; nadie invoca las skills, el agente reacciona.
5. Una skill por procedimiento, no por ronda.
   No existen variantes `re-*`.
   El input (estado de la carpeta de la task, hallazgos, retakes) determina el modo.
   Dos skills que comparten el 80% del texto se desincronizan con el tiempo.

## om-manager

| Skill | Qué hace |
|---|---|
| `setup` | Extrae la documentación de un proyecto, esté donde esté (READMEs, wikis, ADRs, comentarios, código, Sebastian), hacia la convención del punto 1. No la espera como entrada: es su salida. Inventaría, confirma módulos contigo, documenta módulos en paralelo como Workflow (`om-setup-worker`), luego TRD, PRD y ARD (vacío si no hay historia), `CLAUDE.md` corto. Idempotente. |
| `write-prd` / `write-trd` / `write-ard` | Producen o actualizan cada documento. Los usa `setup`; `document-task` los reutiliza a nivel de módulo. |
| `plan-task` | Conversación de planning con Sebastian siguiendo la ruta de lectura de `docs/`. Termina ofreciendo `create-task`. |
| `create-task` | Crea la carpeta `docs/tasks/{{id}}_{{title}}/` en el checkout principal con `task.md` (`type`, `Goal`, `Scope`, `Acceptance`), `replication.md` si es bug y, en el raro caso de fases, un `phase_N.md` por fase. No commitea. Termina ofreciendo `consolidate-task`. |
| `consolidate-task` | Sincroniza la rama base, crea worktree y rama según `type` desde `origin/{{base}}`, abre la ventana de tmux y lanza solo al om-reviewer dentro del worktree. Media las dudas del om-reviewer con Sebastian hasta que `Context & decisions` está escrito en la copia del checkout principal. Con aprobación de Sebastian commitea y pushea `docs(tasks): {{id}}_{{title}} planned` (plan más decisiones; único commit de docs de la task). Pregunta si se delega ahora; si no, detiene la sesión del om-reviewer (conservando la conversación) y cierra la ventana de tmux; worktree y rama se quedan. |
| `delegate-task` | Verifica `depends_on`. Reabre la sesión del om-reviewer si estaba detenida (`claude attach` o `claude -r`); solo si se perdió lanza una nueva. Hace `git fetch` y `git rebase origin/{{base}}` en el worktree para que la rama reciba la carpeta con las decisiones. Manda "delegated, start". No lanza om-developers. |
| `reiterate-task` | Anota los comentarios de Sebastian sobre el PR, fechados, en `retakes.md` y relanza el par sobre el mismo branch, worktree y PR. |
| `check-task` | Deriva el estado de una task: `planned` (sin worktree), `consolidating` (worktree sin `Context & decisions`), `consolidated` (worktree con `Context & decisions`, sin commits ni om-developer), `in_progress` (commits por delante de la base o sesión de om-developer), `in_review` (PR abierto según `gh`), `merged` (PR mergeado, worktree aún existe), `done` (PR mergeado, sin worktree). No consulta a las sesiones para preguntarles nada; solo comprueba si existen. |
| `check-work` | `check-task` sobre todas las tasks del proyecto. Es el tablero de Sebastian. |
| `clean-task` | Si el PR de la task está mergeado a la rama base: `git pull` en el checkout principal, borra sesiones de Claude, worktree, rama local y remota, ventana de tmux. No commitea nada; sin worktree la task se deriva como `done`. |
| `clean-work` | Recorre todos los worktrees, detecta los mergeados y corre `clean-task` en cada uno. |

## om-reviewer

| Skill | Evento que la dispara | Qué hace |
|---|---|---|
| `analyze-task` | Nace la sesión | Lee la carpeta de la task y `docs/` de los módulos. Si `Context & decisions` está vacío, hace el ping-pong con om-manager y Sebastian una sola vez y lo escribe en la copia del checkout principal (la única que se escribe antes de la delegación); puede ajustar `Scope`, `Acceptance` y las fases. Si ya está escrito (solo pasa cuando la sesión original se perdió), lo lee y no vuelve a preguntar. Si hay `retakes.md` nuevo, lo incorpora. Avisa al om-manager "consolidated" y espera "delegated, start". |
| `start-task` | Mensaje del om-manager "delegated, start" | Abre el pane derecho de la ventana de tmux, lanza `om-{{id}}-developer` (o `-developer-phase-1`) con cwd en el worktree y le manda "context ready, start". |
| `review-task` | Mensaje "ronda N" del om-developer | Corre el Pipeline sobre el worktree: intent, rebase, `verify-task`, review, documentation. Si hay issues, los manda al om-developer con archivo:línea, error y lo esperado. Si no hay issues, corre `publish-task`. |
| `publish-task` | `review-task` sin issues | Push de cada rama del workspace, un PR por repo tocado (más el del root en multirepo), y el único comentario resumen en el PR del root con Intent, What changed (con links a cada PR), Decisions (incluido orden de merge), Risk assessment y Pipeline por target. No escribe ningún estado; avisa al om-manager "PRs ready". |
| `next-phase` | Mensaje del om-manager "phase N merged, continue" | Mata la sesión del om-developer de la fase N, crea la rama de la fase N+1 desde `origin/{{base}}` en el worktree, escribe `Result` de la fase N en `phase_N.md` (viaja en el PR de la fase N+1) y corre `start-task`. |

El om-reviewer nunca modifica código.
En tasks con fases es el único que vive toda la task; los om-developers cambian por fase.
Sí puede pushear: comparte el worktree con el om-developer y publicar no es escribir código.
Es el único punto de publicación.

## om-developer

| Skill | Evento que la dispara | Qué hace |
|---|---|---|
| `execute-task` | "context ready, start" del om-reviewer, o mensaje con hallazgos | Modo implementar si no hay código de la task; modo corregir si hay hallazgos o retakes. Rebase desde `origin/{{base}}`, implementa, `verify-task`, `document-task`, aplasta en un commit, escribe su nota de cierre (qué hizo, qué dejó pendiente) en `task.md` o `phase_N.md`, avisa al om-reviewer "ronda N". |
| `document-task` | Al final de `execute-task` | Actualiza `prd.md`, `trd.md`, `ard.md`, `database.md` y `flows.md` del módulo tocado con `updated` y `source` (punto 1). |

El om-developer nunca toca el remoto.
Su trabajo termina en un commit local y un mensaje al om-reviewer.
Nunca habla con el om-manager ni con Sebastian.

## Compartida

| Skill | Quién | Qué hace |
|---|---|---|
| `verify-task` | om-reviewer y om-developer | Itera los `Verification targets` del TRD (uno por repo o app que la task toca). Corre lint → typecheck → tests, uno a uno y con `--runInBand`. Para `type: docs` no corre tests. Un bloque por target en `verify.log`: qué corrió, cuándo, resultado y sobre qué commit. |

El registro de `verify-task` es lo que permite al om-reviewer comprobar que lint y tests corrieron después del último fix.
El om-reviewer además la re-corre sobre el commit final, lo que hace irrelevante el orden en que la corrió el om-developer.

## Máquinas de estado

### om-reviewer

```
nace ──> analyze-task ──> "consolidated" ──> espera "delegated, start"
delegated ──> start-task (lanza om-developer) ──> espera
espera ──(ronda N)──> review-task ──(issues)──> manda hallazgos ──> espera
                                  ──(sin issues)──> publish-task ──> in_review ──> espera
in_review ──(fase N mergeada, hay fase N+1)──> next-phase ──> start-task ──> espera
in_review ──(última fase mergeada, o sin fases)──> termina
```

### om-developer

```
nace ──> espera aviso del om-reviewer
aviso ──> execute-task (implementar) ──> "ronda 1" ──> espera
hallazgos ──> execute-task (corregir) ──> "ronda N" ──> espera
```

## Mapa de nombres

Nombres definitivos, que reemplazan a los usados provisionalmente en conversaciones anteriores:

| Provisional | Definitivo |
|---|---|
| `new-task` | `plan-task` + `create-task` |
| `implement-task` | `execute-task` |
| `retake-task` | `reiterate-task` |
| `summarize-task` | `publish-task` |
| `test-task` | `verify-task` |
| `reanalyze-task`, `reexecute-task` | eliminadas; el modo lo decide el input |

## Relación con las skills actuales

| Actual en `~/.claude/skills/` | Destino |
|---|---|
| `refine-us`, `us-to-tus`, `us-to-specs` | Se absorben en `plan-task` y `create-task`. |
| `implement-specs-{nestjs,nextjs,rails,react-native}` | Se absorben en `execute-task` con `references/{{stack}}.md`. |
| `pr-reviews` | Se absorbe en `review-task`. |
| `address-pr-comments-nestjs` | Se absorbe en `reiterate-task` + `execute-task` (modo corregir). |
| `ds-write-prd`, `ds-write-trd` | Reemplazadas por `write-prd`, `write-trd`, escritas de cero. |

## Pendientes derivados

- Escribir `~/.claude/agents/om-manager.md`, `om-reviewer.md` y `om-developer.md` con la lista de skills permitidas y la máquina de estado de cada rol.
- Definir el formato exacto de `verify.log`.
- Definir el formato del mensaje de hallazgos del om-reviewer al om-developer.

## Inventario (2026-08-29)

Todas escritas como borrador en `skills/`, pendientes de piloto.

| Rol | Skills |
|---|---|
| om-manager | `setup`, `write-prd`, `write-trd`, `write-ard`, `plan-task`, `create-task`, `consolidate-task`, `delegate-task`, `reiterate-task`, `check-task`, `check-work`, `clean-task`, `clean-work` |
| om-reviewer | `analyze-task`, `start-task`, `review-task`, `publish-task`, `next-phase` |
| om-developer | `execute-task`, `document-task` |
| Compartida | `verify-task` |
| Overmind | `add-project` (con `pause` y `remove`), `resume-project`, `check-portfolio`, `clean-portfolio`, `add-todo`, `complete-todo` |

Agentes en `agents/`: `overmind`, `om-manager`, `om-reviewer`, `om-developer`, `om-setup-worker` (subagente que `setup` usa para documentar un módulo con contexto limpio).

Pendiente de contenido: `references/{{stack}}.md` de `execute-task` y `verify-task` (nestjs, nextjs, rails, react-native).
Se escriben cuando el primer proyecto de cada stack pase por `setup`; el TRD del proyecto es la fuente y el reference solo el fallback.
Las skills previas de `~/.claude/skills/` fueron eliminadas por Sebastian el 2026-08-29 por no usarse.
