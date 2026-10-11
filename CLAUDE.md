# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, no build, no tests, no linter. README and UI text are in Spanish.

## Running

Open `index.html` in a browser, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files: `index.html` (DOM + two canvases), `style.css`, `game.js` (all logic, single global script with `'use strict'`, no modules).

`game.js` details that span the file:
- Board is a `ROWS x COLS` matrix; `0` = empty, `1–7` = piece type index into both `COLORS` and `PIECES` (same index used for cell values, so cell value = color).
- Game state lives in module-level `let` variables declared on one line (`board, current, next, score, ...`) and reset in `init()`. `init()` also serves as restart handler.
- Piece flow: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current`, rolls new `next`, and calls `endGame()` if the new piece collides.
- Loop: `loop(ts)` via `requestAnimationFrame` accumulates `dropAccum`; gravity interval is `dropInterval` (recomputed in `clearLines`: `max(100, 1000 - (level-1)*90)`). Pause cancels the animation frame and restarts `loop` on resume; `endGame` also cancels it.
- Rotation: `rotateCW` + `tryRotate` wall kicks of `[0,-1,1,-2,2]` columns (not SRS).
- Scoring: `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/row.
- Changing `COLS`/`ROWS`/`BLOCK` requires updating `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` x `ROWS*BLOCK`). Next-piece canvas is a fixed 120x120 (4x4 grid at 30px).
