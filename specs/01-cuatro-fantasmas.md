# SPEC 01 — Cuatro fantasmas con personalidades distintas

> **Estado:** Approved
> **Depende de:** —
> **Fecha:** 2026-09-05
> **Objetivo:** Implementar 4 fantasmas, cada uno con un comportamiento único — uno de ellos persigue agresivamente a Pac-Man — con salida escalonada de la pen y alternancia dispersión/caza.

## Scope

**In:**

- 4 fantasmas con kinds distintos: `chaser`, `predictor`, `pincer`, `shy`.
- Renombrado de kinds existentes: `hunter` → `chaser`, `random` → `shy` (en `maze.js`, `game.js`).
- Salida escalonada de la pen con retrasos configurados por fantasma.
- Oscilación (bobbing) de los fantasmas que esperan dentro de la pen.
- Alternancia de modos dispersión/caza en bucle de 7s/20s.
- Tabla `GHOST_CONFIG` con esquina de dispersión y retraso de salida por kind.

**Out of scope (futuros specs):**

- Píldoras de poder y modo asustado (frightened). No existen píldoras aún.
- Fruta, bonús y niveles.
- Velocidades por fase (elroy) o por fantasma.
- Colisiones fantasma-fantasma.

## Data model

```js
// maze.js
const GHOST_CONFIG = {
  chaser:    { releaseDelay: 0,   home: { x: 27, y: 0 }  },
  predictor: { releaseDelay: 120, home: { x: 0, y: 0 }   },
  pincer:    { releaseDelay: 300, home: { x: 27, y: 30 } },
  shy:       { releaseDelay: 540, home: { x: 0, y: 30 }  },
};

const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'chaser' },
  { x: 14, y: 14, kind: 'predictor' },
  { x: 13, y: 15, kind: 'pincer' },
  { x: 14, y: 15, kind: 'shy' },
];
```

```js
// game.js — cada fantasma gana dos campos
ghost = { x, y, dir, speed, kind, released: false, releaseAt: 0 }
```

- `game.tick` cuenta frames; `releaseAt = game.tick + GHOST_CONFIG[kind].releaseDelay` al crear/resetear.
- Constantes de fase: `SCATTER_FRAMES = 420`, `CHASE_FRAMES = 1200`; la fase se calcula de `game.tick`.
- Colores por índice en `render.js` (rojo, cian, rosa, naranja): sin cambios, ligados al orden de `GHOST_STARTS`.
- Convención: ~60fps vía `requestAnimationFrame`; 120 frames ≈ 2s.

## Implementation plan

1. `maze.js`: expandir `GHOST_STARTS` a 4 kinds y añadir `GHOST_CONFIG`. Temporalmente todos actúan random (solo `hunter` tenía lógica). Verificar: 4 fantasmas de 4 colores en pantalla.
2. `game.js`: reemplazar el branch de kinds por `ghostTarget()` — `chaser`→celda de Pac-Man, `predictor`→4 celdas por delante, `pincer`→fórmula Inky usando a `chaser` como referencia, `shy`→random; añadir `game.tick` y la fase dispersión/caza (en dispersión todos persiguen su `home`). Verificar: 4 comportamientos distintos + alternancia visible.
3. `game.js`: lógica de pen — `released`/`releaseAt`, bobbing mientras esperan, liberación cuando `tick >= releaseAt`; `resetPositions` reinicia la secuencia al perder una vida. Verificar: uno sale al instante, tres oscilan y salen escalonados.
4. `game.js`: eliminar referencias antiguas (`hunter`/`random`) y validar el bucle completo. Verificar: estados ganar/perder alcanzables sin errores de consola.

## Acceptance criteria

- [ ] Hay exactamente 4 fantasmas, uno por kind: `chaser`, `predictor`, `pincer`, `shy`.
- [ ] Cada fantasma se dibuja con su color clásico (rojo, cian, rosa, naranja).
- [ ] `chaser` reduce su distancia Manhattan a Pac-Man durante la fase de caza.
- [ ] `predictor` apunta a la celda 4 tiles por delante de la dirección de Pac-Man.
- [ ] El objetivo de `pincer` deriva de la posición de `chaser` (fórmula Inky).
- [ ] `shy` no persigue un objetivo fijo; sus decisiones varían (random).
- [ ] En fase de dispersión, los 4 dirigen hacia su `home` configurado.
- [ ] `chaser` sale en el tick 0; los otros tres esperan en la pen y salen en su `releaseAt`.
- [ ] Los que esperan oscilan sin salir de su celda.
- [ ] No existen referencias a los kinds antiguos `hunter` y `random`.
- [ ] El juego mantiene los estados ganar/perder sin errores de consola.

## Decisions

- **Sí:** nombres descriptivos para los kinds. **No:** nombres clásicos (`blinky`/`pinky`/`inky`/`clyde`).
- **Sí:** cuarteto clásico simplificado. **No:** comportamientos a medida.
- **Sí:** salida escalonada con retrasos fijos en frames. **No:** liberación por puntos.
- **Sí:** alternancia fija 7s/20s en bucle. **No:** plan clásico completo ni caza permanente.
- **Sí:** bobbing mientras esperan. **No:** quietos hasta salir.
- **No:** velocidad por fantasma/fase — todos usan `GHOST_SPEED` actual.
- **Sí:** timing por frames (~60fps), no por tiempo real.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| Un fantasma se queda atascado dentro/fuera de la pen | La puerta (3) es transitable para ghosts; la decisión se toma entre celdas vecinas; el target solo orienta |
| El bobbing cruza el límite de la celda | Offset ±0.25 y clamp a la celda |
| El timing depende de 60fps | Frames como base; se documenta la asunción en el código |

## What is **not** in this spec

- Píldoras de poder y modo asustado.
- Fruta, bonús y niveles.
- Velocidades elroy por fase.
- Colisiones fantasma-fantasma.

Cada una de esas, si llega, va en su propio spec.