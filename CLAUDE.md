# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS (ES6+). No dependencies, no bundler, no tests, no linter, and not a git repo. UI text and code comments are in Spanish.

## Running

Open `index.html` directly in a browser, or serve it with `npx serve .` (http://localhost:3000). There is no build step; edit and reload.

## Architecture

All game logic lives in `game.js`, loaded as a classic (non-module) script by `index.html`, which only provides a `<canvas id="canvas" width="800" height="600">`. `game.js` hardcodes `W = 800` and `H = 600` — keep them in sync with the canvas attributes.

- **Loop**: `loop(ts)` → `update(dt)` → `draw()` via `requestAnimationFrame`. `dt` is in seconds and clamped to 0.05 to avoid big jumps after tab switches. All motion is in px/s or px/s².
- **State machine**: global `state` is `'playing' | 'dead' | 'gameover'`. `update()` branches on it up front. `'dead'` freezes the ship, keeps asteroids and particles moving, and respawns after `deadTimer` (2s). `'gameover'` waits for Space to call `initGame()`.
- **Entities**: classes `Ship`, `Bullet`, `Asteroid`, `Particle`, each with `update(dt)` and `draw()`. Entities are kept in module-level arrays (`bullets`, `asteroids`, `particles`) and removed by setting `dead = true` and then reassigning the array with `.filter(...)`.
- **World is toroidal**: positions use `wrap(v, max)`. Particles are the exception and do not wrap.
- **Asteroid sizes** are 1/2/3 and index the parallel arrays `RADII`, `SPEEDS`, `POINTS` (index 0 is unused padding). `split()` produces two asteroids of `size - 1`. The README's score table is Large 20 / Medium 50 / Small 100, which matches `POINTS`.
- **Input**: `keys[e.code]` holds held state. `pressed(code)` is an edge-triggered, consume-once check backed by `justPressed`. Space fires through `pressed('Space')` (one shot per press, plus a 0.2s cooldown). Rotation and thrust read `keys` directly.
- **Collisions** are circle-based using `dist()` and `radius`. Ship vs asteroid uses `ship.radius + a.radius * 0.82` and is skipped while `ship.invincible > 0`. That timer also drives the blink in `Ship.draw()`.
- **Levels**: when `asteroids.length === 0`, `nextLevel()` runs. It clears bullets and particles, resets the ship, and spawns `3 + level` large asteroids, kept at least `SAFE_DIST` (130px) from the center.

## Notes

- The README mentions power-ups and a "shooting star" asteroid type. Neither is implemented in `game.js`.
