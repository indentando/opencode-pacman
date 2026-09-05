# SPEC 02 — Salida de los fantasmas de la pen

> **Estado:** Approved
> **Depende de:** SPEC 01
> **Fecha:** 2026-09-05
> **Objetivo:** Corregir la salida de la pen: al liberarse, cada fantasma navega hasta la puerta y alcanza el mapa, y la puerta se vuelve de salida única para que ningún fantasma quede atrapado ni reingrese.

## Scope

**In:**

- Navegación hacia la puerta mientras el fantasma está dentro de la pen: ignora scatter/chase y sube en línea recta hasta estar fuera (`y <= 11`).
- Flag persistente `exited` por fantasma; la puerta (celda `3`) se vuelve de salida única: bloqueada para fantasmas con `exited === true`.
- Retrasos escalonados, `released`/`releaseAt` y bobbing de SPEC 01 intactos.
- Misma lógica de salida tras perder una vida (ruta de `resetPositions`).

**Out of scope (futuros specs):**

- Colisiones/solape fantasma-fantasma (comparten la puerta de 2 celdas al salir).
- Cambios a `GHOST_CONFIG`, kinds, scatter/chase, retrasos, bobbing o render.
- Comportamiento clásico de "un fantasma se queda en la pen".

## Data model

```js
// game.js
const PEN_EXIT_ROW = 11; // y <= 11 => el fantasma ya dejó la pen (puerta en y=12)

// cada fantasma gana un campo
ghost = { x, y, dir, speed, kind, released, releaseAt, exited: false }
```

- `isWall`/`canMove` reciben el **objeto fantasma** como `actor` (en vez del string `'ghost'`): `if ( v === 3 ) return actor === 'pacman' || actor.exited === true;`. `pacman` sigue bloqueado por la puerta; un fantasma en tránsito la cruza; uno ya `exited` no reingresa.

## Implementation plan

1. `game.js`: añadir `exited: false` en `createGame` y `resetPositions`; branch de tránsito en `moveGhost` (`released && !exited`): en cada celda alineada `dir = 'up'` (la puerta está arriba en ambas columnas de la pen) y marcar `exited = true` al cumplir `y <= PEN_EXIT_ROW`. Verificar: los 4 salen en su orden aunque aún sin protección de reingreso.
2. `game.js`: `isWall`/`canMove`/`decideGhost` pasan el objeto fantasma como actor; la puerta bloquea a los `exited`. Verificar: un fantasma fuera (p. ej. `pincer`/`shy` en scatter persiguiendo esquina inferior) cruza sobre la puerta sin reingresar.
3. `game.js`+`render.js`: sin cambios de render; validar transición pen→mapa y el bucle completo. Verificar: estados ganar/perder sin errores de consola.

## Acceptance criteria

- [ ] Los 4 fantasmas abandonan la pen en el frame de su `releaseAt` y alcanzan el mapa (`y ≤ 11`).
- [ ] Mientras un fantasma está dentro de la pen, su dirección es siempre `'up'` (nunca `down`, nunca target de scatter/chase).
- [ ] Ningún fantasma con `exited === true` vuelve a cruzar la puerta hacia la pen (incluso en scatter persiguiendo esquina inferior).
- [ ] Al perder una vida, los 4 regresan a la pen (bobbing) y vuelven a salir con su `releaseAt` usando la misma navegación.
- [ ] `GHOST_CONFIG`, `releaseDelay` y bobbing de SPEC 01 quedan intactos.
- [ ] Estados ganar/perder alcanzables sin errores de consola.

## Decisions

- **Sí:** navegación a la puerta dentro de la pen (fuerza `'up'`). **No:** variante "solo excluir down" ni "corredor clásico" — más complejas sin beneficio.
- **Sí:** puerta de salida única (one-way) por fantasma. **No:** geometría re-evaluada cada frame / reingreso permitido — reintroduce el atasco.
- **Sí:** mantener `released`/`releaseAt` y bobbing de SPEC 01. **No:** refactor a un único estado combinado.
- **Sí:** misma lógica tras perder vida. **No:** limitar el fix a la partida inicial.
- **No:** colisiones/solape fantasma-fantasma en la puerta — spec futuro.
- **Sí:** `exited` como flag persistente. **No:** marcar salida por geometría cada frame.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| Fantasma fuera vuelve a atascarse | Puerta one-way + tránsito nunca elige `down` |
| Solape en la puerta al salir en fila | Aceptado: colisiones fantasma-fantasma fuera de scope |
| Actor mixto (string vs objeto) en `isWall`/`canMove` rompe a pacman | `'pacman'` mantiene su bloqueo de puerta; cubierto por criterios |