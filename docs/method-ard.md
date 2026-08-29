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
