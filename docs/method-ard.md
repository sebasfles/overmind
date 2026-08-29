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
