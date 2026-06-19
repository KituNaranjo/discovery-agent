---
description: Convierte los supuestos riesgosos del MVP de un discovery en hipótesis falsables y diseña el experimento más barato para cada una. Sujeto al gate de hipótesis.
argument-hint: "<carpeta del discovery, p. ej. discoveries/bazarpapeleria>"
---

# Generar hipótesis y experimentos de un discovery

Usa la skill `discovery` (sección "Hipótesis y experimentos").

**Discovery:** `$ARGUMENTS`
Si está vacío, pregunta cuál discovery antes de continuar.

Prerrequisito: deben existir `$ARGUMENTS/outputs/mvp-canvas.md` y
`$ARGUMENTS/outputs/evidence-map.json`. Si faltan, corre antes los comandos previos.

Idea rectora (diapositiva 15): el puente entre output e impact es una **hipótesis**,
y las hipótesis se comprueban. Idea rectora (diapositiva 32): cada experimento es
**comprar información barata** sobre el riesgo más grande antes de construir.

## Pasos

1. Lee `mvp-canvas.md` (sobre todo "Riesgos / supuestos" y la métrica de éxito),
   `requisitos.md` y `evidence-map.json` de `$ARGUMENTS/outputs/`.

2. Identifica los **supuestos más riesgosos** (saltos de fe de los que depende el
   MVP y que aún no están validados). Prioriza por riesgo (impacto de equivocarse
   × incertidumbre), del más peligroso al menos.

3. Para cada supuesto prioritario escribe una **hipótesis falsable** y su
   **experimento**, siguiendo la skill. Cada una debe tener: el supuesto, el
   enunciado "Creemos que… si… porque…", una **señal medible** (de negocio, no de
   vanidad), un **criterio de éxito concreto** (con número/porcentaje), el
   **experimento más barato** que responda (con caja de tiempo) y una **regla de
   decisión** que diga qué hacer si pasa **y si falla**.

4. Escribe primero `$ARGUMENTS/outputs/experiment-board.json` (el tablero
   machine-readable que audita el gate) y luego `$ARGUMENTS/outputs/hypotheses.md`.

## Formato OBLIGATORIO de hypotheses.md (estructura visual completa)

El `.md` debe ser visualmente rico y autoexplicativo. Genera EXACTAMENTE estas
secciones, en este orden, todo derivado de los artefactos (cero invención):

1. **Encabezado:** título "# Hipótesis y Experimentos — <nombre del producto>",
   seguido de una línea de cita con la fecha actual y las fuentes
   (`mvp-canvas.md`, `requisitos.md`, `evidence-map.json`).

2. **Sección "## Cadena output → outcome → impact":** un diagrama Mermaid
   `flowchart LR` con tres nodos encadenados (Output → Outcome → Impact), tomando
   el output de las funcionalidades mínimas del Canvas, el outcome del comportamiento
   que debe cambiar y el impact de la métrica de éxito con su meta. Colores:
   output `#1A4E8A` (azul), outcome `#3E6FA6` (azul medio), impact `#E89B0C` (ámbar).
   Cierra con una frase: las hipótesis son los puentes; si un supuesto falla, la
   cadena se corta antes de producir impacto.

3. **Sección "## Orden de prioridad por riesgo":** una tabla Markdown con columnas
   `# | Hipótesis | Riesgo | Experimento | Caja de tiempo`, una fila por hipótesis,
   ordenadas de mayor a menor riesgo. En la columna Riesgo usa el emoji de color:
   🔴 Alto, 🟡 Medio, 🟢 Bajo. La columna Hipótesis lleva un título corto; el
   Experimento, su tipo resumido; la Caja de tiempo, el plazo · costo.

4. **Una test card por hipótesis** (`### [H-NN] <título> — riesgo: alto|medio|bajo`),
   y dentro de cada una, EN ESTE ORDEN:
   - Primero un diagrama Mermaid `flowchart TD` de **árbol de decisión**: el nodo
     del experimento, una rama "pasa el umbral" → Perseverar (verde, `ok`) y una
     rama "falla" → Pivotar/descartar (rojo, `no`). El color del nodo del test
     refleja el riesgo (alto/medio = rojo/ámbar). Usa los `classDef` de la skill.
   - Luego las viñetas: **Supuesto a probar**, **Hipótesis**, **Señal medible**,
     **Criterio de éxito**, **Experimento**, **Caja de tiempo/costo**,
     **Regla de decisión** (con el "si pasa →" y el "si falla →").

5. **Sección final "## Secuencia de aprendizaje recomendada":** un diagrama Mermaid
   que muestre en qué orden/paralelo se corren los experimentos (qué va en el mismo
   periodo y qué depende de qué), más una o dos frases que lo expliquen. Colores por
   riesgo: alto rojo, medio ámbar, bajo verde; nodo final verde "Listo para construir".

Respeta las convenciones de color de la skill (sección 8). Separa cada sección con
una línea `---`.

5'. Cierra el turno con un resumen breve: cuántas hipótesis, cuál es la #1 por
riesgo y qué experimento la ataca primero.

## El gate de hipótesis

Al escribir estos artefactos, el hook `hypothesis-gate.py` exige que cada
hipótesis sea comprobable (señal medible, umbral concreto, regla de decisión que
contemple el fallo, métrica no de vanidad). Si bloquea, **no lo esquives**:
reescribe la hipótesis en forma falsable. Una hipótesis que no se puede refutar no
es una hipótesis, es un deseo.