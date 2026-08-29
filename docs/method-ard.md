# ARD del método

Bitácora de decisiones sobre cómo trabajan los agentes.
Se apila; nunca se reescribe.
Las decisiones de diseño originales están en `01` a `05`; aquí van los cambios posteriores y su razón.

## 2026-08-29: Roles con prefijo `om-`

- Decision: los agentes se llaman `overmind`, `om-manager`, `om-reviewer`, `om-developer`, `om-setup-worker`; las sesiones `om-{{proyecto}}-manager`, `om-{{id}}-reviewer`, `om-{{id}}-developer[-phase-n]`.
- Alternatives rejected: sufijo `-agent`; nombres sin prefijo escribiendo siempre "the {{rol}} agent" en la prosa.
- Reason: sin prefijo, los tres nombres de rol se leen como personas en la prosa de 27 skills y en las listas de sesiones.
- Debt created: none.
- Revisit when: nunca, salvo que aparezca otro sistema de agentes en la misma máquina con el mismo prefijo.
- Files: agents/, skills/, docs/, bin/resume-project.

## 2026-08-29: Standalone, no plugin

- Decision: skills y agentes viven en este repo y se enlazan con symlinks desde `~/.claude/`; no se empaqueta como plugin.
- Alternatives rejected: plugin en un marketplace personal.
- Reason: una persona, una máquina; el plugin cobra un prefijo en cada invocación y un paso de recarga sin dar nada que un repo con symlinks no dé.
- Debt created: la conversión a plugin queda pendiente si aparece una segunda máquina o persona.
- Revisit when: dos máquinas, dos personas, o colisiones de nombres con plugins de proyectos.
- Files: docs/04-operacion.md.

## 2026-08-29: Skills de proyecto para mantener el método

- Decision: `update-method` (en `.claude/skills/` de este repo) aplica cambios transversales, y `bin/lint-method` verifica lo mecánico.
- Alternatives rejected: `update-skill` y `update-agent` separadas; seguir haciendo los pases a mano.
- Reason: cada cambio del método tocó entre 20 y 44 archivos; sin un procedimiento y un lint, algo queda a medias.
- Debt created: none.
- Revisit when: el lint empiece a dar falsos positivos que cueste más mantener que lo que atrapa.
- Files: .claude/skills/update-method/, bin/lint-method, docs/method-ard.md.

## 2026-08-29: `setup` extrae, no espera la convención; módulos como Workflow

- Decision: `setup` trata la convención como salida, nunca como precondición: inventaría documentación en cualquier lugar y formato, la cruza con el código y con Sebastian, y produce `docs/`. La documentación por módulo corre como Workflow (paralelo con tope 4, resultados con schema, resume), con `Agent(om-setup-worker)` como fallback.
- Alternatives rejected: exigir la estructura `docs/` antes de correr; hacer toda la skill un Workflow (las partes interactivas no pueden correr en segundo plano); seguir solo con subagentes sueltos.
- Reason: los proyectos existentes tienen docs dispersas y ninguno tiene la convención; y el fan-out por módulo es el único trabajo del sistema con N jobs independientes donde el resume y el schema pagan.
- Debt created: la skill instruye escribir el script del Workflow en tiempo de ejecución; conviene guardar un script de referencia en `skills/setup/references/` tras el piloto.
- Revisit when: el piloto muestre que el Workflow agrega fricción frente a los subagentes.
- Files: skills/setup/SKILL.md, docs/03-skills.md.

## 2026-08-29: `setup` con dos fan-outs: discovery por componente y documentación por módulo

- Decision: el discovery corre en paralelo por componente (repo o app) con `om-setup-worker` en modo `discover` (solo lectura, salida con schema); la sesión principal fusiona, propone el mapa carpetas → módulos y lo confirma con Sebastian; luego la documentación corre en paralelo por módulo con el mismo agente en modo `document`. Son dos Workflows separados.
- Alternatives rejected: un solo Workflow (no puede parar a preguntar); discovery secuencial en la sesión principal (en `hux` son 10 repos y llena el contexto antes de escribir nada).
- Reason: los módulos no existen hasta que termina el discovery y Sebastian los confirma; y los módulos que cruzan componentes solo aparecen en la fusión, que es trabajo humano más sesión principal, no de un worker.
- Debt created: none.
- Revisit when: el piloto en un multirepo muestre que la fusión de candidatos necesita más estructura en el schema.
- Files: skills/setup/SKILL.md, agents/om-setup-worker.md, docs/03-skills.md.

## 2026-08-29: El om-manager reenvía eventos al overmind, no mensajes

- Decision: el om-manager reenvía al `overmind`, en una línea y solo si está corriendo, todo mensaje entre sesiones que cambie el estado de una task o la bloquee (consolidated, PRs ready, phase merged, retake sent, cleaned, errores). No reenvía el relevo de preguntas de la consolidación ni contenido.
- Alternatives rejected: reenviar todo mensaje recibido (ruido en la cabina durante la consolidación); reenviar solo "PR ready" (los bloqueos quedaban invisibles hasta que Sebastian entrara al proyecto).
- Reason: la cabina existe para decir dónde hace falta mirar; los bloqueos son justamente eso.
- Debt created: none.
- Revisit when: la lista de eventos crezca y convenga un formato estructurado en vez de una línea.
- Files: agents/om-manager.md, agents/overmind.md.

## 2026-08-29: Tres sesiones en el overmind, eventos tipados, escalada única

- Decision: la cabina se divide en `overmind` (estado y proyectos), `om-events` (buzón, agente `om-events` en haiku) y `om-config` (método, agente `om-config`). Los eventos que mandan los om-managers son tipados (`blocker`, `action`, `info`) y se archivan en `portfolio/events/`, apilándose con contadores. El único mensaje que sube toda la cadena es un bloqueo por permisos, credenciales, entorno o herramientas; om-reviewer y om-developer corren con `bypassPermissions`. Los tres agentes de la cabina son de ámbito de proyecto (`.claude/agents/`); los cuatro roles de proyecto siguen globales.
- Alternatives rejected: una sola sesión overmind con los tres trabajos (mezcla contexto y hace ruido con los eventos); agentes de cabina globales (aparecerían como subagentes delegables en cada om-manager); eliminar `update-method` al existir `om-config` (el agente es quién, la skill es el procedimiento invocable).
- Reason: cada sesión long-lived necesita un solo motivo para vivir; y el buzón separado hace que los eventos se vean sin interrumpir la conversación con la cabina.
- Debt created: `bypassPermissions` quita la red del clasificador a om-reviewer y om-developer; la mitigación es el aislamiento del workspace y las reglas duras de cada agente.
- Revisit when: un om-developer haga algo destructivo fuera de su workspace, o el buzón necesite más que tres tipos.
- Files: .claude/agents/, agents/om-manager.md, agents/om-reviewer.md, agents/om-developer.md, bin/resume-overmind, bin/lint-method, portfolio/events/, docs/04-operacion.md, docs/03-skills.md, .claude/skills/update-method/SKILL.md, CLAUDE.md.

## 2026-08-29: Sesiones singleton con el nombre del agente; modelos y effort por rol

- Decision: las sesiones de la cabina se llaman como su agente (`overmind`, `om-events`, `om-config`) porque son singletons; los roles con varias instancias siguen con nombres de instancia (`om-diy-manager`, `om-0142-reviewer`). `overmind` baja a opus/medium (agrega y enruta); `om-setup-worker` en modo discover corre en sonnet dentro del Workflow; las skills llevan `effort` según su naturaleza: xhigh en análisis y revisión, high en planificación y escritura, medium en traspasos, low en las mecánicas.
- Alternatives rejected: nombres de sesión distintos del agente para todo (`overmind-events`), un solo effort para todas las skills.
- Reason: agente y sesión son cosas distintas y eso confunde; igualarlos donde hay una sola instancia quita un nombre que recordar. El modelo va con el costo del error del rol y el effort con el razonamiento que pide la tarea.
- Debt created: none.
- Revisit when: el piloto muestre una skill mecánica que necesite más razonamiento, o una de análisis que no lo aproveche.
- Files: .claude/agents/, skills/*/SKILL.md, docs/04-operacion.md, bin/resume-overmind.
