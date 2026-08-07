# AGENTS.md

Vanilla HTML5 Canvas game. No build toolchain, no dependencies, no test suite.

## Running

- Open `index.html` directly in a browser, or serve the repo root static: `npx serve .` then visit `http://localhost:3000`.
- There is **no `package.json`**. Do not run `npm install`, `npm test`, lint, or typecheck — none exist. "Verifying a change" means reloading the page in a browser.

## Architecture

- `game.js` (single file) is loaded by `index.html` via a plain `<script src="game.js">` — **not** `type="module"`. All classes (`Ship`, `Asteroid`, `Bullet`, `Particle`) and top-level constants (`W`, `H`, `keys`, `ctx`, …) live on the global scope. Do not add `import`/`export` without also switching the script tag to `type="module"`.
- Entry flow is at the bottom of `game.js`: `initGame()` → `requestAnimationFrame(loop)`. `loop` derives a `dt` (clamped to 0.05s to avoid the spiral-of-death after tab switches) and calls `update(dt)` then `draw()`.
- State machine: `state ∈ {'playing', 'dead', 'gameover'}`. `'dead'` is the per-life respawn delay (`deadTimer`, ~2s) — **distinct** from `'gameover'`, which is end-of-game awaiting `Space` to restart. Don't conflate them.

## Conventions worth preserving

- **Toroidal space**: positions wrap edges via `wrap(v, max)`. Every `update(dt)` wraps `x` against `W` and `y` against `H`. Keep this pattern when adding any moving entity.
- **Two input channels**: `keys[code]` is held-state (drives rotation/thrust); `justPressed[code]` is edge-triggered and read exactly once per frame via `pressed(code)` (drives shooting and restart). Do not read `justPressed` directly — `pressed()` consumes it.
- **Asteroid sizes** index 1/2/3 (3=large → 1=small). The `RADII` / `SPEEDS` / `POINTS` arrays use a `0` placeholder at index 0. `spawnAsteroids()` always emits size 3; `Asteroid.split()` decrements by 1 and returns `[]` at size 1.
- **Canvas is fixed 800×600 in two places**: `index.html` (`<canvas width="800" height="600">`) and `game.js` (`const W = 800; const H = 600;`). Change both together, or wrap-around math silently breaks.