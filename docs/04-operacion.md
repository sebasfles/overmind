# Punto 4: Operación diaria, cabina y portafolio

Estado: parcialmente acordado el 2026-08-28.
Las secciones marcadas "Por definir" aún no se han conversado.
Depende de: [02-orquestacion.md](02-orquestacion.md), [03-skills.md](03-skills.md).

## Cabina (cockpit)

Un solo lugar donde Sebastian ve todos los proyectos.
Es una sesión de tmux `overmind` con una sesión de Claude corriendo en la raíz del repo `~/dev/personal/projects/overmind` con el rol `overmind`.
El nombre es deliberadamente distinto de "om-manager" para que nunca se confunda con el de un proyecto.

### Regla

La cabina observa, agrega, notifica y lleva a Sebastian al lugar correcto.
Nunca planea ni decide sobre un proyecto.
Las conversaciones de producto, arquitectura y técnicas siguen siendo con el om-manager de cada proyecto, en su sesión de tmux.

Razón: el om-manager vale por su contexto profundo de un proyecto.
Un overmind con poder de decisión tendría contexto superficial de todos, decidiría peor y agregaría un salto donde la información se pierde.
El planning con el om-manager es la conversación de mayor valor del flujo y no se intermedia.

### Qué hace

- Estado agregado: `check-work` de cada proyecto registrado, en una sola tabla.
  Una línea por ítem, sin explicaciones; el detalle se ve en la sesión del proyecto o en el PR.
  Sale de disco (`docs/tasks/*.md`), git y `gh`; no le pregunta a los om-managers.
- Eventos: los om-managers le avisan por `SendMessage` cuando algo requiere de Sebastian (PR listo, task trabada en delegación).
- Limpieza transversal: `clean-work` en todos los proyectos.
- Deriva de documentación: compara el `updated` de los docs de cada módulo con los últimos commits que tocaron ese módulo.
  Es el chequeo que se decidió no hacer con hooks; aquí es una lectura, no un bloqueo.
- Mantener el método: skills, agentes y `CLAUDE.md` global evolucionan desde aquí.

Más adelante puede vivir aquí la priorización entre proyectos ("¿qué ataco hoy?"), como decisión de Sebastian informada por datos, no del agente.

No se crea `audit-portfolio`: la deriva de docs y la basura acumulada son columnas de `check-portfolio`.
Se separaría solo si el reporte se vuelve demasiado largo para el tablero diario.

## Registro de proyectos

La cabina no adivina recorriendo `~/dev`; solo considera los proyectos registrados.
Archivo `portfolio/projects.yaml` en el repo `overmind`:

```yaml
projects:
  - name: diy
    root: /home/fless/dev/designli/projects/diy
    tmux: diy
    status: active        # active | paused
    repos:
      - name: diy-platform
        base_branch: develop
      - name: diy-infra
        base_branch: main
```

Ver [05-layouts.md](05-layouts.md) para `root` y `repos` en single repo, monorepo y multirepo.

## Skills del overmind

| Skill | Qué hace |
|---|---|
| `add-project` | Registra un proyecto (`root`, `repos`) en `projects.yaml`; en multirepo convierte la carpeta en repo de docs (`{{name}}-docs`). Verifica la convención del punto 1 y, si no, ofrece correr `setup` desde su om-manager. |
| `pause-project` / `remove-project` | Lo saca del tablero sin borrar nada. |
| `resume-project` | Abre una pestaña nueva de Windows Terminal, en WSL, en la raíz del proyecto, dentro de su sesión de tmux, con el om-manager corriendo. |
| `check-portfolio` | Tabla por proyecto: tasks en delegación, en progreso, en PR esperando a Sebastian, ejecutores ociosos, deriva de docs, basura acumulada. |
| `clean-portfolio` | `clean-work` en todos los proyectos activos. |
| `add-todo` | Captura rápida de algo para no olvidar, opcionalmente ligado a un proyecto. |
| `complete-todo` | Tacha un todo. |

### resume-project

Es un script de shell determinista (`bin/resume-project`); la skill es un envoltorio fino que lo llama con el nombre del proyecto.
Funciona también como comando directo, sin la cabina.

Entorno verificado el 2026-08-28: Windows Terminal (`WT_SESSION` definido), `wt.exe` invocable desde WSL, distro `Ubuntu`, tmux 3.4.
Sebastian ya trabaja con una sesión de tmux por proyecto (`diy`, `auvral`, `drive-now`).

Comportamiento:

1. Lee `projects.yaml` para obtener `root` y `tmux`.
2. Si la sesión de tmux no existe, la crea con cwd en `root` y la primera ventana `om-manager` corriendo `claude --agent om-manager -n om-{{name}}-manager`.
3. Abre la pestaña en la ventana actual de Windows Terminal y se engancha a la sesión.

```bash
tmux has-session -t "$name" 2>/dev/null || \
  tmux new-session -d -s "$name" -c "$path" -n om-manager "claude --agent om-manager -n om-${name}-manager"

wt.exe -w 0 new-tab --title "$name" \
  wsl.exe -d Ubuntu --cd "$path" \
  -- tmux attach -t "$name"
```

`-w 0` abre la pestaña en la ventana actual de Windows Terminal.
Engancharse a una sesión ya enganchada desde otra pestaña es válido; ambas la reflejan.

## Todos

Un todo es una captura rápida: algo que Sebastian anota para no olvidarlo y sigue con lo que estaba haciendo.
Puede estar relacionado a un proyecto o no.
Puede terminar siendo una task, una decisión, una conversación con alguien, o nada.
No se clasifica al anotarlo; la fricción de captura debe ser mínima.

Reglas:

- Un todo puede ligarse a un proyecto de forma opcional.
  `check-portfolio` lo muestra en la fila de ese proyecto; los globales van en una sección aparte.
- Lo que ya muestra el estado (por ejemplo un PR en `in_review`) no hace falta anotarlo, pero tampoco está prohibido.
- Cuando un todo se convierte en trabajo para agentes, Sebastian salta al om-manager del proyecto, lo planea con `plan-task` y lo cierra con `complete-todo`.

Skills: `add-todo` para anotar y `complete-todo` para tacharlo.
Listar no necesita skill: lo hace `check-portfolio`.

Almacenamiento: `portfolio/todos.md`, Markdown plano, legible y editable sin el agente.

## Día típico

1. Sebastian abre la cabina y pide `check-portfolio`.
2. Ve qué espera por él: PRs en `in_review`, delegaciones trabadas, todos pendientes.
3. `resume-project {{name}}` lo lleva al om-manager de ese proyecto.
4. Habla con el om-manager: aprueba PRs, planea tasks nuevas, delega.
5. Vuelve a la cabina o salta a otro proyecto.

## Por definir

### Contenido de los agentes

`agents/om-manager.md`, `om-reviewer.md`, `om-developer.md`, `overmind.md` y `om-setup-worker.md`, en el repo `overmind`, con symlinks desde `~/.claude/agents/`.
Escritos como borrador el 2026-08-29; ver `03-skills.md`.
Cada uno con: descripción del rol, skills permitidas, herramientas permitidas, modo de permisos, máquina de estado (para om-reviewer y om-developer), y reglas de qué nunca hace.

### `~/OPINIONS.md`: descartado

Decidido el 2026-08-28: no se crea.
Las opiniones que importan a los agentes de Sebastian son estrechas (código, arquitectura, proceso) y ya tienen tres casas mejores:

| Tipo de opinión | Dónde vive |
|---|---|
| Regla corta, siempre aplica | `~/.claude/CLAUDE.md` |
| Criterio de un rol | `~/.claude/agents/{{rol}}.md` |
| Decisión de un proyecto | `ARD.md` del proyecto |

Un cuarto lugar se desincronizaría de los otros tres.
La línea del `CLAUDE.md` global que mandaba leer `~/OPINIONS.md` ya fue eliminada.

Se revisita solo si los agentes toman decisiones de criterio equivocadas de forma repetida y la corrección no cabe en una línea de `CLAUDE.md` ni pertenece a un rol.
En ese caso el archivo nace con contenido real, no inferido.

### Piloto

Proyecto donde se valida el flujo de punta a punta.
Debe validar: `--bg` + `attach` en panes de tmux, entrega de `SendMessage` entre sesiones, `setup` sobre un repo existente, y una task completa hasta el merge.
Proyecto por elegir.
