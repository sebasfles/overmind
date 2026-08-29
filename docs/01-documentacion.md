# Punto 1: Documentación autogenerada como base de contexto

Estado: acordado el 2026-08-28.

## Objetivo

Trabajar en muchos proyectos al mismo tiempo sin que el agente redescubra cada proyecto desde cero.
La documentación es el mecanismo de contexto: es input de cada tarea y también parte de su output.
La convención es de Sebastian, no del proyecto.
Todo proyecto (nuevo o existente) se adapta a la misma estructura, de modo que un único conjunto de skills funcione en todos sin adaptación.

## Principios

1. No documentar lo que el código ya dice; documentar el porqué.
   Lo derivable del código (columnas, tipos, request/response) se genera desde la fuente de verdad y no se copia a Markdown.
   Lo intencional (producto, decisiones, deuda, invariantes) lo escribe el agente al terminar cada tarea.
2. Progressive disclosure.
   El agente lee poco al inicio y profundiza solo en lo que necesita.
   Cargar toda la documentación al inicio degrada al agente tanto como no tener ninguna.
3. La documentación se mantiene fresca desde el flujo de trabajo, no con hooks.
   Todo cambio pasa por las skills, y las skills tienen el paso de documentar como obligatorio.
   No se usarán hooks de pre-commit ni bloqueos automáticos para esto (decisión explícita).
4. Lo inferido se marca.
   Cuando el agente deduce algo sin evidencia (por ejemplo, el porqué de una decisión en un proyecto viejo), lo marca con `[inferido]` para que Sebastian lo confirme o lo elimine.
   Sin esta marca, decisiones inventadas terminarían tratadas como verdad.

## Tipos de documento

### PRD (Product Requirements Document)

Describe qué hace el sistema y por qué, sin detalle de ingeniería.
Existe a nivel general y a nivel de módulo (acotado y más específico).
Se edita en el lugar: refleja el estado actual del producto, no acumula historia.

### TRD (Technical Requirements Document)

Describe cómo está construido el sistema.
Existe a nivel general y a nivel de módulo.
Se edita en el lugar: refleja el estado técnico actual.

El TRD general declara el stack y sus mecánicas.
Esto incluye cómo se generan los specs de API, cómo se corren tests, lint y migraciones.
Las skills leen esta sección para saber cómo operar en el proyecto, en vez de tener una skill por stack.

El TRD de módulo incluye la lista de endpoints que el módulo posee, una línea por endpoint con su propósito y un link al spec generado.
No se duplican request/response en Markdown.

### ARD (Architecture and Debt Record)

Registra decisiones de arquitectura y deuda técnica.
Existe a nivel general (decisiones globales e índice de deuda) y a nivel de módulo.

A diferencia del PRD y el TRD, el ARD es una bitácora.
Cada decisión es una entrada nueva y fechada.
Las entradas viejas no se borran; a lo sumo se marcan como superadas por una entrada posterior.
El valor del ARD está en el recorrido, no solo en el estado final.

Cada entrada del ARD contiene:

- Fecha.
- Decisión tomada.
- Alternativas descartadas.
- Razón de la elección.
- Deuda técnica que genera (si aplica).
- Disparador para revisitarla (por ejemplo, "cuando haya más de N tenants" o "cuando migremos a X").

### database.md (por módulo)

Describe el impacto del módulo en la base de datos.
Contiene:

- Tablas que el módulo posee.
- Tablas de otros módulos que referencia.
- Invariantes que viven solo en código y no están aplicadas por la base de datos.

No copia columnas ni tipos.
La fuente de verdad del schema sigue siendo Prisma, TypeORM, migraciones o el equivalente del stack.

### flows.md (por módulo, solo si aplica)

Diagramas de estado y de secuencia para flujos complejos.
Se escriben en Mermaid dentro del Markdown: es texto, se versiona con git, el agente lo lee y modifica como código, y se renderiza en GitHub.
Solo para flujos que lo ameriten; un diagrama para un CRUD es ruido.

### Especificación de API

Generada desde el código, no escrita a mano.
Ejemplos: `@nestjs/swagger` en NestJS, rswag o el schema GraphQL en Rails.
El TRD general indica cómo se genera en cada proyecto.
El agente lee el JSON o schema generado cuando necesita detalle de request/response.

## Estructura de archivos

Centralizada en `docs/` en vez de junto al código, para que funcione igual en NestJS, Next.js, Rails y React Native.
El agente siempre sabe dónde mirar sin importar el stack.

```
CLAUDE.md                  # corto: stack, comandos, y "la doc vive en docs/"
docs/
  PRD.md                   # producto general
  TRD.md                   # técnico general, declara stack y mecánicas
  ARD.md                   # decisiones globales + índice de deuda
  modules/
    {{modulo}}/
      README.md            # 20-40 líneas: qué hace, límites, links a los demás
      prd.md
      trd.md               # incluye endpoints que posee
      ard.md               # bitácora del módulo
      database.md
      flows.md             # mermaid: estados y secuencias (solo si aplica)
```

## Versionamiento

Git es el sistema de versiones; no se construye un changelog manual dentro de cada archivo.

Como el autor en git siempre es Sebastian aunque escriba un agente, cada documento lleva un frontmatter mínimo:

```yaml
---
updated: 2026-08-28
source: 0142_badge_wall   # la task que provocó el cambio
---
```

Esto resuelve trazabilidad (qué tarea cambió esta decisión) y detección de documentos viejos (un `updated` antiguo en un módulo activo es una señal de deriva).

## Ruta de lectura del agente

1. `CLAUDE.md`: corto, indica la estructura de la documentación del proyecto.
2. `docs/PRD.md` y `docs/TRD.md`: contexto general del producto y el sistema, y lista de módulos.
3. Decide en qué módulo hace sentido el cambio.
4. Lee a fondo solo ese módulo: `README.md` y luego los archivos que necesite.

Así el agente no se llena de toda la información, solo de la necesaria.

## Skills

Un solo conjunto de skills, genérico en su flujo.
El detalle por stack no vive en skills separadas, sino en dos lugares:

- El TRD general del proyecto, que declara stack y mecánicas.
- Archivos `references/{{stack}}.md` dentro de la skill, que se cargan solo si hace falta detalle.

Agregar un stack nuevo es agregar un archivo de referencia, no una skill.
Esto reemplaza las cuatro copias actuales de `implement-specs-*` (nestjs, nextjs, rails, react-native) por una sola `execute-task`.

| Skill | Rol |
|---|---|
| `setup` | Crea o reconcilia la convención de documentación en un proyecto. Llama a `write-prd`, `write-trd` y `write-ard`. |
| `plan-task` + `create-task` | Fase de planning. Lee la documentación siguiendo la ruta de lectura y produce la carpeta de la task en disco. |
| `execute-task` | Implementa la task. Opera según el stack declarado en el TRD. Al terminar llama a `document-task`. |
| `document-task` | Actualiza PRD, TRD, ARD, database y flows del módulo tocado, con `updated` y `source`. |
| `review-task` | Verifica que la task tocó lo necesario, incluida la documentación. |

La lista completa de skills por rol está en [03-skills.md](03-skills.md).

### setup

Es única e idempotente.
Sirve para repos vacíos, repos viejos sin documentación y repos a medias.
Conceptualmente hace siempre lo mismo:

1. Detecta el estado: stack, módulos existentes, qué documentación existe y cuál falta o está vieja (según `updated`).
2. Produce un reporte de brechas: estado ideal vs estado actual.
3. Rellena las brechas llamando a `write-prd`, `write-trd` y `write-ard`, a nivel general y por módulo.
   Lo inferido se marca con `[inferido]`.
4. Escribe o ajusta el `CLAUDE.md` corto.

En un proyecto viejo el paso 3 es pesado y se paraleliza por módulo con subagentes de contexto limpio.
En un proyecto nuevo es casi instantáneo.

### document-task

Debe ser un paso nombrado y obligatorio al final de `execute-task`.
Si no es explícito, el agente lo omite cuando el contexto se pone largo.
Actualiza:

- `prd.md` y `trd.md` del módulo: edición en el lugar.
- `ard.md` del módulo: entrada nueva con decisiones tomadas y deuda generada.
- `database.md` y `flows.md` si el cambio los afecta.
- Frontmatter `updated` y `source` en cada archivo tocado.

### review-task

Como segunda línea de frescura, incluye el ítem: ¿el módulo tocado tiene su documentación actualizada?

## Pendientes derivados

- `ds-write-trd` está roto: hoy es una copia del PRD (frontmatter y cuerpo). Hay que reescribirlo.
- No existe `write-ard`. Hay que crearlo.
- `~/OPINIONS.md` se descartó (ver [04-operacion.md](04-operacion.md)); el criterio transversal vive en `~/.claude/CLAUDE.md` y en los agentes por rol.
- Ningún proyecto en `~/dev` tiene `CLAUDE.md` ni `.claude/` propio. Serán los primeros candidatos para `setup`.
