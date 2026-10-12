# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no build step, no tests, no linter. README and code comments are in Spanish.

## Running

Open `index.html` in a browser, or serve locally with `npx serve .` (http://localhost:3000).

## Architecture

All game logic lives in `game.js` (~420 lines, `'use strict'`, global scope, loaded by `index.html`). It is organized in sections:

- **Input**: `keys` (held state) and `justPressed` (edge-triggered). `pressed(code)` consumes the edge, so call it once per frame per key (used for Space to shoot / restart).
- **Entities**: `Bullet`, `Asteroid`, `Ship`, `Particle` classes, each with `update(dt)` and `draw()`; they draw directly to the global `ctx`. Entities flag `dead = true` and are removed by array filtering in `update()`.
- **Game state**: module-level globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) and a `state` string: `'playing' | 'dead' | 'gameover'`. `'dead'` is a 2s respawn pause (`deadTimer`) where asteroids keep moving; `'gameover'` waits for Space to call `initGame()`.
- **Loop**: `loop(ts)` → `update(dt)` → `draw()`, with `dt` in seconds clamped to 0.05.

Key conventions:
- Canvas is fixed at `W=800` × `H=600`; space is toroidal, so all positions use `wrap(v, max)`.
- Asteroid tuning is in the `RADII`/`SPEEDS`/`POINTS` arrays, indexed by size (1–3, index 0 unused). `split()` produces two asteroids of `size - 1`.
- Collision is circle-based using `dist()` and `radius` (ship vs asteroid uses `a.radius * 0.82`). Ship is ignored while `invincible > 0`.
- Clearing all asteroids calls `nextLevel()`, which spawns `3 + level` asteroids outside a safe radius around screen center.
