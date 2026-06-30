# Discovery Agent

Agente de Claude Code que convierte **conocimiento de negocio crudo** (entrevistas y
notas de campo) en **artefactos de producto** —personas, stakeholders, requisitos,
user stories y MVP Canvas— e **hipótesis con experimentos** para validarlos. Es el
producto de la Unidad 1 de *Ingeniería de Software* (Maestría en Software, UPS).

El objetivo no es escribir el código de la aplicación, sino **entender el problema
antes de construir**: partir de evidencia real y llegar, de forma trazable, hasta un
MVP y los experimentos baratos que lo ponen a prueba.

## Qué es y para qué sirve

El agente es **genérico**: no está atado a ningún caso. Cada caso de negocio es un
**discovery** independiente bajo `discoveries/`. El agente lee la evidencia cruda de
ese discovery y produce sus artefactos, respetando cuatro reglas no negociables:

- **Cero invención.** No se afirma ningún dolor, persona o requisito que no esté
  respaldado por una entrevista real del discovery. Si la evidencia no alcanza, se
  dice, en lugar de rellenar con suposiciones.
- **Trazabilidad.** Cada persona, dolor y requisito cita el archivo de entrevista del
  que proviene.
- **Personas de primera mano.** Una persona solo se considera respaldada si existe una
  entrevista *en primera persona* de ese rol; que la mencionen otros no basta.
- **Aislamiento entre discoveries.** El agente trabaja solo dentro del discovery
  indicado y nunca mezcla evidencia ni artefactos de uno con otro.

Los artefactos se escriben en español (salvo términos técnicos como *user story*,
*MVP* o *stakeholder*) y siguen siempre el formato definido por la skill `discovery`.

## Estructura del repositorio

- `discoveries/<nombre>/` — un caso de descubrimiento. Contiene:
  - `interviews/` — evidencia cruda: entrevistas y notas en Markdown, cada una con
    frontmatter (`rol_entrevistado`, `primera_persona`, `anonimizada`). Es la **única
    fuente de verdad** sobre el negocio de ese discovery.
  - `outputs/` — todo artefacto generado de ese discovery (personas, requisitos, user
    stories, MVP Canvas, hipótesis, mapa de evidencia y reporte).
  - `discoveries/bazarpapeleria/` — caso de estudio real desarrollado (ver más abajo).
  - `discoveries/_template/` — discovery en blanco para copiar al empezar uno nuevo.
- `.claude/` — el agente en sí (genérico):
  - `skills/discovery/` — la skill `discovery`: el estándar de formato de cada
    artefacto.
  - `commands/discovery/` — los comandos del flujo de trabajo.
  - `hooks/` — los gates (`readiness-gate.py` y `hypothesis-gate.py`).
  - `scripts/` — el generador del reporte visual.
- `CLAUDE.md` — instrucciones del proyecto (reglas, flujo y descripción de los gates).
- `docs/` — documentación complementaria.

## Flujo de trabajo

Cada comando recibe la carpeta del discovery como argumento (por ejemplo
`/discovery:analyze discoveries/bazarpapeleria`) y escribe únicamente dentro de su
carpeta `outputs/`:

1. **`/discovery:analyze <discovery>`** — lee toda la evidencia de `interviews/`,
   extrae personas, stakeholders, dolores y requisitos, y produce `personas.md`,
   `requisitos.md` y `evidence-map.json` (el mapa de evidencia que auditan los gates).

2. **`/discovery:generate-mvp <discovery>`** — a partir de esos artefactos genera
   `user-stories.md` (en formato INVEST) y `mvp-canvas.md`. Sujeto al **gate de
   readiness**.

3. **`/discovery:experiments <discovery>`** — convierte los supuestos riesgosos del
   MVP en hipótesis falsables y diseña el experimento más barato para cada una,
   produciendo `hypotheses.md` y `experiment-board.json`. Sujeto al **gate de
   hipótesis**.

Además, **`/discovery:report <discovery>`** genera `report.html`, un reporte visual y
a color del discovery (personas, cadena de valor y tablero de experimentos) a partir
de los artefactos estructurados.

## Los gates

Dos hooks imponen reglas duras antes de dejar escribir ciertos artefactos. Ambos se
auto-ubican: deducen a qué discovery pertenece la escritura desde la ruta del archivo,
así que funcionan para cualquier discovery.

- **Gate de readiness** — antes de escribir `mvp-canvas.md` o `user-stories.md`,
  valida que la evidencia sea suficiente: existe el mapa de evidencia, hay un mínimo de
  entrevistas, cada persona primaria tiene respaldo de primera mano y no hay dolores
  huérfanos. Si no, **bloquea** la escritura y explica qué falta.

- **Gate de hipótesis** — antes de escribir `hypotheses.md` o `experiment-board.json`,
  valida que cada hipótesis sea **comprobable**: señal medible, criterio de éxito
  concreto (con número), regla de decisión que contemple el fallo y una métrica que no
  sea de vanidad. Una hipótesis que no se puede refutar no pasa.

La validación es independiente (no confía en lo que el modelo declara): se audita
contra los archivos en disco. En
[docs/demo-gate.md](docs/demo-gate.md) se documenta una demostración del gate de
readiness —un bloqueo real con sus motivos y su resolución consiguiendo más
evidencia— como prueba de que los gates efectivamente funcionan.

## Caso de estudio: `bazarpapeleria`

Ejemplo real desarrollado de principio a fin. Es una papelería con servicio de
impresiones atendida por una sola persona. El dolor central, identificado en la
evidencia, es que **se imprime antes de cotizar**: la propietaria ejecuta el trabajo y
recién después informa el precio; cuando el cliente lo rechaza, el papel y la tinta ya
se gastaron sin posibilidad de recuperarlos.

El discovery parte de cuatro piezas de evidencia en `interviews/`: dos entrevistas de
primera mano de la propietaria (`propietaria.md`, `propietaria_seguimiento.md`), una
entrevista de primera mano de un cliente (`cliente.md`) y una nota de observación
directa del proceso de venta (`observador.md`).

A partir de ahí, los artefactos en `outputs/` recogen 2 personas y sus dolores, los
requisitos candidatos, las user stories en formato INVEST y un MVP Canvas centrado en
la **cotización previa de impresiones** (mostrar el precio y confirmar con el cliente
*antes* de imprimir). La fase de experimentos plantea las hipótesis falsables sobre
los supuestos más riesgosos del MVP —empezando por la de mayor riesgo: que cotizar
antes realmente *reduzca* el insumo perdido y no solo adelante el rechazo— con su
experimento más barato. Todo el contenido es trazable a las entrevistas, y el
`report.html` del caso resume el descubrimiento de forma visual.

## Documentación

- [Demostración del readiness-gate (bloqueo y resolución)](docs/demo-gate.md)
- [CLAUDE.md](CLAUDE.md) — reglas de trabajo, flujo y detalle de los gates.
