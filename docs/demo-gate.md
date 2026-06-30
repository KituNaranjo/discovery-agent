# Demostración del readiness-gate — bloqueo real y resolución

Este documento registra una demostración del gate de readiness
(`.claude/hooks/readiness-gate.py`): cómo **bloquea** la generación del MVP cuando la
evidencia de un discovery es insuficiente, y cómo **deja pasar** una vez que la
evidencia se completa. Es la evidencia de que el gate no es decorativo: efectivamente
impide escribir `mvp-canvas.md` / `user-stories.md` sin respaldo.

Fecha de la demostración: 2026-06-30.

---

## Cómo se invoca el gate

El gate es un hook `PreToolUse`: antes de cada escritura de un artefacto del MVP,
Claude Code le entrega por entrada estándar un payload que describe qué archivo se
va a escribir y en qué discovery. Para la demostración replicamos ese payload:

```json
{ "tool_input": { "file_path": "<ruta>/outputs/mvp-canvas.md" }, "cwd": "<discovery>" }
```

El gate **devuelve un código de salida** que Claude Code interpreta así:

- `exit = 2` → **bloqueo**: la escritura no se realiza, y el gate explica por qué.
- `exit = 0` → **permitido**: la evidencia es suficiente y la escritura procede.

---

## El arco demostrado: bloqueo → resolución

Se preparó un discovery de prueba con evidencia **deliberadamente insuficiente** y se
intentó "escribir" su `mvp-canvas.md` (mediante el payload del hook). Luego se
completó la evidencia y se reintentó.

### Estado inicial (insuficiente)

- `interviews/` contiene **una sola** nota, `observador.md`, y es de observación
  (`primera_persona: false`).
- `outputs/evidence-map.json` declara una persona **primaria** `Propietaria`
  (rol `propietaria`) y un dolor `imprime-antes-de-cotizar` cuya fuente es
  `propietaria.md`.
- Pero **no existe** ninguna entrevista de primera mano de la propietaria en disco:
  la persona está "construida de oídas" y el dolor cita un archivo que no está.

### ETAPA 1 — el gate BLOQUEA

Con ese payload apuntando a `outputs/mvp-canvas.md`, el gate respondió:

```text
────────────────────────────────────────────────────────────────
✗ HOOK readiness-gate: evidencia insuficiente para generar el MVP
────────────────────────────────────────────────────────────────
  • Solo hay 1 entrevista(s) en interviews; se requieren al menos 2.
  • Persona «Propietaria» (rol: propietaria) no tiene una entrevista en primera persona en disco. Está construida de oídas.
  • Dolor «imprime-antes-de-cotizar» cita «propietaria.md», que no existe en interviews/.
────────────────────────────────────────────────────────────────
Acción: levanta más evidencia (agrega/entrevista) y reintenta.
────────────────────────────────────────────────────────────────

exit = 2
```

`exit = 2` es el **bloqueo**: la escritura de `mvp-canvas.md` no se realiza. El gate
señala con precisión las tres carencias: pocas entrevistas, una persona primaria sin
respaldo de primera mano, y un dolor huérfano que cita una fuente inexistente.

### Resolución — conseguir la evidencia que faltaba

La respuesta correcta **no** es sortear el gate, sino levantar evidencia real. Se
agregan las dos entrevistas de primera mano de la propietaria que faltaban:

- `interviews/propietaria.md` — `rol_entrevistado: propietaria`, `primera_persona: true`
- `interviews/propietaria_seguimiento.md` — `rol_entrevistado: propietaria`, `primera_persona: true`

Con esto se corrigen las tres causas a la vez: ya hay ≥2 entrevistas, la persona
primaria `Propietaria` tiene respaldo de primera mano en disco, y el dolor que citaba
`propietaria.md` ahora sí encuentra su fuente.

### ETAPA 2 — el gate PERMITE

Con la evidencia ya completa, el mismo payload sobre el mismo `mvp-canvas.md` dio:

```text
exit = 0
```

`exit = 0`, sin mensajes de error: el gate **deja pasar** la escritura del MVP. El
bloqueo se resolvió por la vía legítima —más evidencia—, no relajando la regla.

---

## Verificación sobre el discovery real (`bazarpapeleria`)

El mismo gate, apuntado al discovery real ya completo
(`discoveries/bazarpapeleria/outputs/mvp-canvas.md`), permite sin objeciones:

```text
exit = 0
```

Esto confirma que el gate distingue correctamente entre evidencia insuficiente
(bloquea) y un discovery con respaldo de primera mano completo (permite).

---

## Cómo reproducir esta demostración

1. Crear una carpeta de discovery de prueba con subcarpetas `interviews/` y `outputs/`.
2. Poner en `interviews/` solo `observador.md` (con `primera_persona: false`).
3. Escribir `outputs/evidence-map.json` con una persona primaria `propietaria` y un
   dolor que cite `propietaria.md` (archivo aún inexistente).
4. Construir un payload con
   `{"tool_input":{"file_path":"<ruta>/outputs/mvp-canvas.md"},"cwd":"<discovery>"}`
   y entregarlo al gate → **exit 2** con los tres motivos.
5. Agregar `propietaria.md` y `propietaria_seguimiento.md` (ambos
   `primera_persona: true`, `rol_entrevistado: propietaria`).
6. Reintentar con el mismo payload → **exit 0**.

---

## Conclusión

El readiness-gate cumple su función: **impide generar el MVP cuando la evidencia es
insuficiente** (entrevistas por debajo del mínimo, persona primaria sin respaldo de
primera mano, dolores huérfanos) y **lo permite una vez subsanada la evidencia**. La
validación la hace de forma independiente contra el disco, no confiando en lo que el
modelo declara: la única manera de pasar el gate es consiguiendo evidencia real.
