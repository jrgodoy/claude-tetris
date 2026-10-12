# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, no build, no tests, no linter. README and UI text are in Spanish.

## Running

Open `index.html` in a browser, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files: `index.html` (DOM + two canvases), `style.css`, `game.js` (all logic, single global script with `'use strict'`, no modules).

`game.js` details that span the file:
- Board is a `ROWS x COLS` matrix; `0` = empty, `1–8` = piece type index into both `COLORS` and `PIECES` (same index used for cell values, so cell value = color).
- Game state lives in module-level `let` variables declared on one line (`board, current, next, score, ...`) and reset in `init()`. `init()` also serves as restart handler.
- Piece flow: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`, rolls new `next`, and calls `endGame()` if the new piece collides.
- Loop: `loop(ts)` via `requestAnimationFrame` accumulates `dropAccum`; gravity interval is `dropInterval` (via `levelInterval(level)` = `max(100, 1000 - (level-1)*90)`, recomputed in `clearLines`). `level = startLevel + floor(lines/10)`; `startLevel` (1–10, chosen in the pause menu) is applied by `init()`.
- Pause: `P`/`Escape` call `togglePause()`, which cancels the animation frame and shows `#pause-menu` (Reanudar, Reiniciar, Ver controles, Nivel inicial). Game keys are ignored while `paused`; resume resets `lastTime`/`dropAccum` and restarts `loop`. `#overlay` is only for GAME OVER. Menu buttons `blur()` after click so Space doesn't activate them.
- Rotation: `rotateCW` + `tryRotate` wall kicks of `[0,-1,1,-2,2]` columns (not SRS).
- Scoring: `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/row.
- Theming: CSS variables in `:root` (dark) and `:root[data-theme="light"]`. `setTheme(theme)` sets `data-theme`, swaps `palette` (`COLORS`/`LIGHT_COLORS`), reads `--grid` into `gridColor` for the canvas, and redraws (`draw()` + `drawNext()`, since the loop is cancelled when paused/game over). Not persisted; always starts dark.
- Changing `COLS`/`ROWS`/`BLOCK` requires updating `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` x `ROWS*BLOCK`). Next-piece canvas is a fixed 120x120 (4x4 grid at 30px).
