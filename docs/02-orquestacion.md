# Punto 2: Orquestación con manager, reviewer y developer

Estado: acordado el 2026-08-28.
Depende de: [01-documentacion.md](01-documentacion.md).

## Objetivo

Sebastian no corre skills una por una ni habla con quien escribe el código.
Habla con un solo agente por proyecto (el manager) y recibe PRs listos para aprobar.
Cada task la resuelve un par de sesiones (reviewer + developer) con contexto propio y acotado a esa task.

## Modelo de tres niveles

| Nivel | Sesión | Vida | Contexto que carga | Skills |
|---|---|---|---|---|
| Manager | Una por proyecto | Larga, se reanuda | `docs/` general, conversación con Sebastian, estado de las tasks | `plan-task`, `create-task`, `delegate-task`, `reiterate-task`, `check-*`, `clean-*` |
| Reviewer | Una por task | La task | Archivo de la task, `docs/` del módulo, diff y PR | `analyze-task`, `review-task`, `publish-task` |
| Developer | Una por task | La task | Archivo de la task, `docs/` del módulo, código | `execute-task`, `document-task` |

Los tres contextos son disjuntos a propósito.
El manager no ve código.
El reviewer no escribe código.
El developer no habla con Sebastian.

### Manager

Es la única sesión con la que habla Sebastian.
Con él se hace el planning y las decisiones de producto, arquitectura y técnicas.
Con `create-task` crea el archivo de la task.
Al crearla pregunta si se ejecuta ahora o queda en cola.
Con `consolidate-task` lanza al reviewer y media sus dudas con Sebastian; con `delegate-task` le entrega la task. Nunca lanza developers: eso lo hace el reviewer.
Nunca entra en el ciclo de revisión de una task.

### Reviewer

Recibe la task del manager con la instrucción: "lee la task, ¿tienes dudas?".
Las dudas se conversan entre reviewer, manager y Sebastian una sola vez, en la fase de consolidación.
Lo decidido en esa conversación lo escribe el reviewer en la sección `Context & decisions` del archivo de la task antes de que arranque el developer.
Después de la consolidación es autónomo: responde las dudas del developer y toma decisiones sin subirlas a nadie.
Sus decisiones quedan documentadas en la sección `Decisions` del comentario del PR.
Al terminar cada ciclo del developer corre `review-task`.
Cuando `review-task` pasa sin issues, corre `publish-task`: push, PR y comentario.
Nunca modifica código.
Es el único punto de publicación: pushear no es escribir código.

### Developer

Trabaja siempre en un worktree propio.
Corre `execute-task`, que incluye `verify-task` y al final `document-task`.
Sus dudas se las hace al reviewer, nunca al manager ni a Sebastian.
Nunca toca el remoto: su trabajo termina en un commit local y un mensaje "ronda N" al reviewer.
Aplasta su trabajo en un solo commit antes de cada ronda.
Las rondas de revisión ocurren en el worktree, antes de publicar; el PR nace con un solo commit.
Solo `reiterate-task` agrega commits nuevos a un PR existente.

## Roles como agentes definidos

Cada rol es un archivo `.claude/agents/{{rol}}.md` con frontmatter (nombre, descripción, herramientas permitidas, modelo) y el system prompt en el cuerpo.
Se lanza una sesión completa con ese rol usando `claude --agent {{rol}}`.
La task se pasa como puntero: `claude --agent developer "task: docs/tasks/142.md"`.
El rol vive versionado; el manager no redacta prompts largos cada vez.
Cada rol tiene su `--permission-mode`: el developer autónomo, el reviewer sin herramientas de escritura de código.

Estos agentes pueden vivir en `~/.claude/agents/` (globales, aplican a todos los proyectos) porque el flujo es la convención de Sebastian, no del proyecto.

## Mecánica de sesiones y tmux

Cada proyecto tiene una sesión de tmux.
El manager corre en la primera ventana.
Cada task ejecutándose es una ventana `task-{{id}}` con dos panes: izquierda reviewer, derecha developer (el de la fase actual, si hay fases).
La ventana la abre el manager en `consolidate-task` con solo el pane del reviewer; el pane del developer lo abre el reviewer al delegarse.

### Identidad de las sesiones

Cada `claude --agent {{rol}}` es un proceso independiente con su propia sesión, historial y contexto.
Dos tasks en paralelo son cuatro procesos que no comparten nada.
Se distinguen por tres identificadores, todos fijados por el manager al lanzarlos:

| Identificador | Ejemplo | Uso |
|---|---|---|
| Nombre de sesión (`-n`) | `task-0142-reviewer`, `task-0142-developer`, o `task-0142-developer-phase-2` si hay fases | Dirección para `SendMessage`. |
| Directorio de trabajo (worktree) | `{{repo}}/.claude/worktrees/0142_badge_wall/` | Aísla el código y agrupa el historial de sesiones de la task en su propia carpeta. |
| ID de sesión | UUID | Para `claude attach {{id}}` o `claude -r {{id}}`. |

La convención `task-{{id}}-{{rol}}` hace el direccionamiento determinista: el reviewer sabe a quién hablarle porque conoce su propio id de task.

### Lanzamiento (forma principal, a validar en el piloto): `--bg` + `attach`

1. El manager lanza ambas sesiones en background:
   Primero crea el worktree y la rama; luego, con cwd en el worktree, `claude --bg --agent reviewer -n task-{{id}}-reviewer "task: docs/tasks/{{id}}_{{title}}/"` y el equivalente con `--agent developer`.
   Ambas comparten el mismo worktree.
2. El manager crea la ventana de tmux `task-{{id}}` con dos panes y en cada uno corre `claude attach {{id}}`.
   `attach` abre la sesión interactiva completa en ese pane: se ve el progreso en vivo, mitad reviewer y mitad developer.
   Con Ctrl+Z se vuelve a la shell y la sesión sigue corriendo.
3. El reviewer trabaja en el mismo worktree que el developer porque necesita correr lint y tests sobre la rama.
   Alternan; nunca escriben a la vez porque el reviewer no escribe código.

Ventajas: `claude agents` lista las sesiones vivas, `claude stop {{id}}` pausa, `claude rm {{id}}` borra la sesión y su worktree, `claude attach {{id}}` reabre.
El proceso sobrevive si se cierra el pane.

### Lanzamiento (forma alternativa, si `--bg` + `attach` no convence)

En cada pane, con cwd en el worktree, se corre directamente `claude --agent {{rol}} -n task-{{id}}-{{rol}} "task: docs/tasks/{{id}}_{{title}}/"`.
Funcionalmente es lo mismo; se pierde la administración con `claude agents / stop / rm` y el proceso muere con el pane.
El resto del diseño no cambia.

### Dónde viven los historiales

Claude Code guarda cada sesión en `~/.claude/projects/{{slug-del-directorio}}/{{session-id}}.jsonl`, agrupada por el directorio donde corrió.
Como reviewer y developer corren dentro del worktree, sus sesiones caen en una carpeta propia, por ejemplo `~/.claude/projects/-home-fless-dev-designli-projects-diy-diy-platform--claude-worktrees-0142_badge_wall/`.
Por eso no aparecen en el `claude -r` del repo principal: el picker filtra por directorio.

### Comunicación entre sesiones

`SendMessage` a la sesión por nombre.
El mensaje entra en la cola de la otra sesión y se procesa en su siguiente turno.
Los mensajes son punteros (carpeta de task, branch, número de PR, ronda, fase), nunca diffs ni contenido.
No se usa `tmux send-keys` para comunicar sesiones.

## Carpeta de task

Ruta: `docs/tasks/{{id}}_{{title}}/`, por ejemplo `docs/tasks/0142_badge_wall/`.
`{{id}}` es secuencial por proyecto con cuatro dígitos; `{{title}}` es snake_case corto.
La carpeta es la fuente de verdad de la task.
Cualquier sesión nueva puede retomarla leyéndola; las sesiones vivas son un acelerador, no una dependencia.

`docs/tasks/_drafts/{{title}}.md` guarda planes en conversación que aún no son task; `plan-task` los escribe cuando la conversación crece y `create-task` los borra al crear la carpeta.
`check-work` ignora `_drafts/`.

Decidido el 2026-08-28: `docs/tasks/` está trackeada por git; solo `docs/tasks/_drafts/` va en `.gitignore`.
La carpeta de una task se escribe en un solo lugar en cada momento, y el punto de traspaso es la delegación:

- Antes de `delegate-task`, en el checkout principal (rama base).
  Ahí la escriben `create-task` (plan) y el reviewer durante la consolidación (`Context & decisions`).
- `consolidate-task` termina con el único commit de docs de toda la task, en la rama base y con aprobación de Sebastian: `docs(tasks): {{id}}_{{title}} planned`.
  Contiene el plan y `Context & decisions`: exactamente lo que Sebastian aprobó.
- `delegate-task` empieza con `git rebase origin/{{base}}` en el worktree; la rama recibe la carpeta completa.
- Después de `delegate-task`, solo en la copia del worktree: `Developer notes`, `verify.log`, `Result`, `retakes.md`.
  Viajan en el PR y llegan a la rama base con el merge.
  Nadie vuelve a escribir la copia del checkout principal; se actualiza sola con `git pull`.
- `clean-task` no commitea nada.
- El checkout principal vive siempre en la rama base; cada skill del manager lo sincroniza con `origin/{{base}}` antes de actuar.
  Si un proyecto prohíbe pushes directos a la rama base, el commit `planned` queda local y se decide por proyecto cómo publicarlo.

### task.md

```yaml
---
id: 0142              # secuencial por proyecto; el manager toma el mayor en docs/tasks/ y suma uno
title: badge_wall
type: feature         # feature | bug | docs | chore | refactor
ticket: DIY-231       # opcional: Jira, Linear, issue de GitHub
branch: feat/0142_badge_wall      # vacío si la task tiene fases; cada fase tiene la suya
modules: [billing, notifications]   # primario primero; una task puede tocar varios
phases: 0             # 0 si no hay fases
depends_on: []        # opcional; solo si hay tasks apiladas o paralelas que se tocan
updated: 2026-08-28
---
```

Secciones del cuerpo:

- `Goal`: qué se quiere lograr y por qué.
- `Scope`: qué entra y qué no.
- `Acceptance`: criterios verificables.
- `Context & decisions`: lo conversado en la consolidación; lo escribe el reviewer antes de que exista un developer.
- `Developer notes`: nota de cierre del developer por ronda: qué hizo, qué dejó pendiente.

### Estado derivado

El frontmatter no tiene campo de estado; ningún commit existe para cambiar de estado.
`check-task` deduce todo de disco, git y `gh`:

| Evidencia | Estado |
|---|---|
| Carpeta en la rama base, sin worktree | `planned` |
| Worktree existe, `Context & decisions` vacío | `consolidating` |
| Worktree existe, `Context & decisions` escrito, sin commits por delante de `origin/{{base}}`, sin sesión de developer | `consolidated` (lista para delegar) |
| Rama con commits por delante de `origin/{{base}}`, o existe sesión `task-{{id}}-developer*` | `in_progress` |
| `gh pr list --head {{rama}}` devuelve un PR abierto | `in_review` |
| PR mergeado y el worktree aún existe | `merged` (pendiente de `clean-task`) |
| PR mergeado y sin worktree | `done` |

Con fases, la misma tabla se aplica por rama de fase.
No existe el campo `pr:`; `gh` lo sabe por la rama.

### Tipos

El `type` determina el prefijo de la rama y parte del comportamiento:

| type | Rama | Comportamiento |
|---|---|---|
| feature | `feat/{{id}}_{{title}}` | Flujo normal. |
| bug | `bugfix/{{id}}_{{title}}` | La carpeta incluye `replication.md`. El developer reproduce el bug siguiéndolo antes de tocar código; si no reproduce, se detiene y avisa al reviewer. El reviewer verifica el fix contra los mismos pasos. |
| docs | `docs/{{id}}_{{title}}` | `verify-task` no corre tests, solo lint de Markdown si existe. |
| chore | `chore/{{id}}_{{title}}` | Flujo normal. |
| refactor | `refactor/{{id}}_{{title}}` | Flujo normal; `Acceptance` exige comportamiento idéntico. |

### Fases

Casi ninguna task tiene fases; el default es un solo PR.
Solo una feature evidentemente grande se divide en fases dentro de la misma task, y lo decide Sebastian.
Cada `phase_N.md` tiene su propio frontmatter (`branch`) y su `Scope` y `Acceptance`.

Reglas:

- Un worktree por task, una rama por fase: `feat/0142_badge_wall-phase-1`, `-phase-2`.
  Con guion y no con `/` porque git no permite que `feat/x` y `feat/x/phase-1` coexistan.
- Un PR por fase.
- Un reviewer para toda la task (`task-0142-reviewer`), que vela por que el plan completo funcione.
- Un developer nuevo por fase (`task-0142-developer-phase-1`, `-phase-2`), con contexto limpio.
- La fase N+1 arranca solo cuando el PR de la fase N está mergeado.
  Cuando Sebastian mergea, se lo dice al manager y el manager le avisa al reviewer que continúe.
  No se usan PRs apilados: son más rápidos pero traen rebases en cascada.
- Al terminar su trabajo el developer deja una nota de cierre en `phase_N.md` (qué hizo, qué dejó pendiente); el reviewer escribe `Result` al saber del merge, en la copia del worktree, y viaja en el PR de la fase siguiente.
  Para la última fase no hay PR siguiente: su resultado va en el comentario del PR.
  Así un reviewer de reemplazo puede retomar desde disco.
- Al pasar a la fase N+1 el reviewer mata la sesión del developer de la fase N y levanta uno nuevo sin contexto.

### Worktree

`{{repo}}/.claude/worktrees/{{id}}_{{title}}/`.
Lo crea el manager en `consolidate-task` con `git worktree add -b {{rama}} {{ruta}} origin/{{base}}` para controlar el nombre de la rama, y lanza al reviewer con ese cwd.
La rama nace sin la carpeta de la task; la recibe en el rebase de `delegate-task`, después del push `planned`.
Se crea en la consolidación y no en la delegación porque una sesión de Claude Code tiene cwd fijo y el reviewer necesita vivir dentro del worktree para correr lint y tests.
Si Sebastian decide no delegar todavía, el worktree y la rama se quedan; solo se detiene la sesión del reviewer (`claude stop`, que conserva la conversación) y se cierra la ventana de tmux.
`delegate-task` reabre la misma sesión (`claude attach`, o `claude -r` si no fue `--bg`) con su contexto intacto.
Reviewer y developer comparten el worktree.

## Ciclo de una task

1. Sebastian al manager: "quiero X".
2. `plan-task` (manager + Sebastian): lee `docs/` siguiendo la ruta de lectura del punto 1 y planea.
   Output: la idea cerrada, y `docs/tasks/_drafts/{{title}}.md` si la conversación creció.
3. `create-task` (manager + Sebastian): escribe la carpeta `docs/tasks/{{id}}_{{title}}/` en el checkout principal con `task.md`, `replication.md` si es bug y `phase_N.md` si hay fases.
   Nada se commitea todavía.
4. `consolidate-task` (manager): sincroniza la rama base, crea worktree y rama según `type` desde `origin/{{base}}`, abre la ventana de tmux `task-{{id}}` y lanza `task-{{id}}-reviewer` dentro del worktree, sin developer.
   El reviewer corre `analyze-task`: lee la carpeta (en el checkout principal) y los docs de los módulos, y le cuenta sus dudas al manager; el manager y Sebastian las resuelven; el reviewer escribe `Context & decisions` en la copia del checkout principal.
   Cuando el reviewer avisa "consolidated", el manager le pide a Sebastian aprobación y commitea y pushea `docs(tasks): {{id}}_{{title}} planned` en la rama base: plan más decisiones, el único commit de docs de la task.
   Luego pregunta: ¿delegar ahora?
   Si no: detiene la sesión del reviewer (conservando su conversación) y cierra la ventana de tmux. Worktree y rama se quedan. La task aparece `consolidated` en `check-work`.
5. `delegate-task` (manager): verifica `depends_on`.
   Si el reviewer está detenido, reabre la ventana de tmux y la misma sesión (`claude attach` o `claude -r`); solo si la sesión se perdió lanza un reviewer nuevo, cuyo `analyze-task` es idempotente.
   Hace `git fetch` y `git rebase origin/{{base}}` en el worktree: la rama recibe la carpeta con las decisiones.
   Manda "delegated, start" al reviewer.
   Desde aquí todo lo que se escribe en la carpeta va a la copia del worktree, y el manager no interviene hasta que el reviewer publique.
6. El reviewer lanza `task-{{id}}-developer` (o `-developer-phase-1`) en el pane derecho, con cwd en el worktree, y le manda "context ready, start".
7. Developer: `execute-task` → rebase desde `origin/{{base}}` → (bug: reproducir con `replication.md`) → implementa → `verify-task` → `document-task` → aplasta en un commit → `Developer notes` → `SendMessage` al reviewer: "round 1 ready".
8. Reviewer corre `review-task` (ver Pipeline) sobre el worktree.
   Si hay hallazgos, se los manda al developer; el developer corrige, re-corre `verify-task`, aplasta, avisa "round 2".
   Se repite hasta que no haya hallazgos.
   Sin hallazgos, el reviewer corre `publish-task`: push, abre el PR, escribe el comentario, avisa al manager "PR #{{n}} ready".
9. Manager avisa a Sebastian, en una línea, que el PR está listo.
10. Sebastian lee el comentario del PR y decide:
    - Aprueba y hace el merge (siempre lo hace él, por ahora).
    - O le dice al manager `reiterate-task` con sus comentarios.
11. Con el merge, Sebastian le pide al manager `clean-task`; ver "Limpieza".
    Si la task tiene fases y no era la última, en vez de limpiar, el manager avisa al reviewer "phase N merged, continue"; el reviewer corre `next-phase` (mata al developer, crea la rama de la fase N+1, lanza un developer nuevo) y el ciclo vuelve al paso 7.
    La limpieza ocurre al mergear la última fase.

### Limpieza de una task mergeada

Con `--bg`:

```
claude rm {{id-reviewer}}                                  # sesión (y worktree cuando es seguro)
claude rm {{id-developer}}                                  # uno por fase si las hubo
git worktree remove {{repo}}/.claude/worktrees/0142_badge_wall    # si claude rm no lo eliminó
git branch -d feat/0142_badge_wall
tmux kill-window -t task-0142
```

Con la forma alternativa (sesiones directas):

```
claude project purge {{repo}}/.claude/worktrees/0142_badge_wall   # transcripts, tasks, historial y config de ese directorio
git worktree remove {{repo}}/.claude/worktrees/0142_badge_wall
git branch -d feat/0142_badge_wall
tmux kill-window -t task-0142
```

En ambos casos el manager no commitea nada: la carpeta final llegó a la rama base con el merge del PR, y sin worktree la task se deriva como `done`.
Antes de limpiar, el manager hace `git pull` en el checkout principal para traer ese estado final.
Sebastian también puede pedirle al manager que limpie todas las ramas ya mergeadas de una vez.

### reiterate-task

El manager anota los comentarios de Sebastian, fechados, en `retakes.md` de la copia del worktree (la task ya está delegada).
Vuelve a levantar el par reviewer + developer sobre el mismo branch, worktree y PR.
Si el par sigue vivo, les avisa; si no, los relanza.
El reviewer corre `analyze-task` incorporando los retakes (es lo único que vuelve a preguntar) y el ciclo continúa desde el paso 7.
`publish-task` actualiza el PR existente y agrega el commit de la nueva ronda.
`retakes.md` viaja en el commit de la siguiente ronda del developer y llega con el PR.

### Si algo muere

El estado está en el archivo de la task, en git y en el PR.
Si la sesión existe: `claude attach {{id}}` o `claude -r`.
Si no: se lanza un par nuevo con el mismo puntero y retoma desde el estado del archivo.

## review-task: el Pipeline

Orden: intent → rebase → lint → test → review → documentation → push.

Lint y test van antes de la revisión profunda porque un test rojo cambia qué se revisa.

| Paso | Qué verifica el reviewer |
|---|---|
| intent | Que los cambios del código corresponden al `Goal` y `Scope` de la task, sin desviaciones ni extras. |
| rebase | Que el developer rebaseó desde `origin/{{base}}`. Lo pide si no está hecho. |
| lint | Re-corre lint sobre el commit final, vía `verify-task`. |
| test | Re-corre typecheck y tests sobre el commit final, vía `verify-task` (uno a uno, `--runInBand`). |
| review | Encuentra issues en el código creado para la task. Documenta el error y cómo se arregló, no cómo se resolvió la task. |
| documentation | Que el developer corrió `document-task`: docs del módulo actualizados con `updated` y `source`. |
| push | Lo hace el propio reviewer en `publish-task` una vez que todo lo anterior pasó. |

Re-correr lint y tests sobre el commit final es lo que hace irrelevante el orden en que el developer los corrió: prueba que el último estado es verde.
Además el reviewer comprueba en `docs/tasks/{{id}}.log` (registro de `verify-task`) que lint y test se corrieron después del último fix.

## publish-task: push, PR y comentario

El reviewer hace el push, abre el PR si no existe, y escribe un solo comentario que edita en cada `reiterate-task` en vez de apilar uno nuevo.
No escribe ningún estado; `in_review` se deriva del PR abierto.

```
## Intent
Un párrafo: qué quería lograr la task y qué se acordó en la delegación.

## What changed
Máximo 10 bullets concisos.

## Decisions
Decisiones que tomó el reviewer de forma autónoma durante la task, con su razón.
Es lo que Sebastian lee para decidir entre aprobar y retake.

## Risk assessment
Nivel (Low / Medium / High) y una frase de justificación.

## Pipeline
- [x] intent
- [x] rebase
- [x] lint
- [x] test
- [x] review: N issues → fixed (detalle por issue: archivo:línea, error, fix, re-check)
- [x] documentation
- [x] push
```

## Escalamiento

Solo existe en la fase de consolidación.
Después, el reviewer decide todo dentro del alcance de la task y lo documenta en `Decisions`.
El único camino de vuelta es `reiterate-task` desde Sebastian a través del manager.

## Paralelismo

Varias tasks en paralelo en el mismo proyecto: una ventana de tmux y un worktree por task.
Los conflictos entre tasks se resuelven en el rebase.
`depends_on` se usa cuando hay tasks apiladas o paralelas que tocan lo mismo; el manager no lanza una task hasta que sus dependencias estén en `done`.

## Skills

La lista completa de skills por rol, sus eventos y las máquinas de estado están en [03-skills.md](03-skills.md).

## Pendientes derivados

- Validar en un piloto que `claude --bg` + `claude attach` dentro de panes de tmux da la misma experiencia visual que una sesión directa, y que `SendMessage` entre sesiones lanzadas así se entrega de forma confiable.
  Si no, pasar a la forma alternativa documentada arriba.
- Definir los archivos `~/.claude/agents/manager.md`, `reviewer.md` y `developer.md`.
- Decidir la plantilla exacta de `docs/tasks/{{id}}_{{title}}/task.md` (secciones arriba) dentro de `create-task`.
