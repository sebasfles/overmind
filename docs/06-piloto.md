# Punto 6: Piloto

Estado: pendiente.
Proyecto por elegir (recomendado: `auvral`, multirepo y personal).
Todo lo escrito en `01` a `05` es diseño; el piloto es donde se corrige.
Cada fallo se arregla en la skill o el agente con `update-method` antes de seguir al paso siguiente.

## Supuestos técnicos a validar

| Supuesto | Dónde se usa | Si falla |
|---|---|---|
| `claude --agent {{rol}}` carga agentes de `~/.claude/agents/` (symlinks) y de `.claude/agents/` del repo | todos los lanzamientos | mover los de la cabina a globales |
| `claude --bg` + `claude attach` en un pane de tmux da la misma experiencia que una sesión directa | consolidate, start-task, resume-overmind | forma alternativa documentada en el `02` |
| `SendMessage` entre sesiones locales lanzadas con `--bg` se entrega y se procesa como turno | toda la comunicación | sesiones directas; si tampoco, archivo de buzón en disco |
| `--allow-dangerously-skip-permissions` con `--bg` deja a om-reviewer y om-developer sin bloqueos del clasificador | consolidate, start-task | `permissionMode: auto` con allowlist en settings |
| El Workflow tool está disponible dentro de una sesión om-manager y acepta `model` por paso | setup (discovery y módulos) | fallback `Agent(om-setup-worker)` ya escrito |
| Un `initialPrompt` que invoca una skill (`/check-work`, `/analyze-task`, `/check-portfolio`) se ejecuta al arrancar | om-manager, om-reviewer, overmind | mandar el primer mensaje a mano desde quien lanza |
| `wt.exe -w 0 new-tab ... wsl.exe --cd ... tmux attach` abre la pestaña en la ventana actual | resume-project, resume-overmind | ajustar flags de Windows Terminal |

## Orden

1. `bin/install`: symlinks. Comprobar que `claude --agent om-manager` arranca en cualquier directorio y que `/plan-task` aparece en `/help`.
2. `bin/resume-overmind`: las tres sesiones. Comprobar que `overmind` abre `om-events` si falta.
3. `add-project {{proyecto}}` desde `overmind`: detección de layout, repo `{{name}}-docs` si es multirepo, registro.
4. `resume-project {{proyecto}}`: pestaña, tmux, om-manager en la ventana 0 con `check-work` diciendo "run setup first".
5. `setup`: discovery por componente (Workflow), confirmación de módulos, entrevista, documentación por módulo (Workflow), TRD, PRD, ARD, `CLAUDE.md`, inferidos, commit. Es la prueba más dura.
6. Una task chica y real: `plan-task` → `create-task` → `consolidate-task` (aquí se validan `--bg`, `attach`, `SendMessage` y el bootstrap del workspace) → `delegate-task` → `start-task` → una ronda → `publish-task` → merge → `clean-task`.
7. Una segunda task en paralelo con la primera, para ver dos workspaces y dos pares a la vez.
8. Un `reiterate-task` con un comentario tuyo en el PR.
9. Si el proyecto es multirepo, una task que toque dos repos: dos PRs de código y el PR del root con el resumen.

## Qué medir

- Cuántas veces tuviste que intervenir fuera de `plan-task`, la consolidación y el merge.
- Cuántos `[inferido]` de `setup` estaban mal.
- Cuántos hallazgos del om-reviewer fueron reales y cuántos ruido.
- Tokens por rol y por task, para revisar modelos y effort.

## Después del piloto

- Fusionar skills que nunca se invocaron solas (candidatas: `create-task` en `plan-task`, `start-task` en `analyze-task` y `next-phase`).
- Guardar el script de Workflow de `setup` en `skills/setup/references/`.
- Escribir `references/{{stack}}.md` de `execute-task` y `verify-task` con lo que el TRD del piloto declaró.
- Crear los symlinks definitivos y borrar `bin/install` si ya no hace falta, o dejarlo para la segunda máquina.
