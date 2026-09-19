# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package manager — just three files (`index.html`, `style.css`, `game.js`).

## Running the game

There is no build/test/lint tooling. To run it, just open `index.html` in a browser, or serve the directory statically:

```bash
python3 -m http.server 8000   # or: npx serve .
```

Then visit `http://localhost:8000`. Changes to `game.js`/`style.css`/`index.html` take effect on browser reload — no compilation step.

## Architecture

Everything lives in `game.js` as top-level state and functions (no classes, no modules, no bundler). The three files cooperate as follows:

- **`index.html`** — DOM structure: the main `<canvas id="board">` (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), a `<canvas id="next-canvas">` for the next-piece preview, HUD elements (`#score`, `#lines`, `#level`), and the pause/game-over `#overlay`.
- **`style.css`** — dark/retro-arcade visual theme only; no layout logic depends on it.
- **`game.js`** — all game logic, structured around this flow:

```
init() → createBoard(), spawn() first piece, requestAnimationFrame(loop)
loop(ts) → accumulate dt → if dt ≥ dropInterval, drop piece or lockPiece() → draw() → requestAnimationFrame(loop)
keydown handler → move / tryRotate() / softDrop() / hardDrop() / togglePause()
```

Key mechanics and where they live (all in `game.js`):

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index (1–8).
- **Pieces**: `PIECES` are square matrices; `randomPiece()` picks one of the 8 pieces (the 7 standard tetrominoes plus a 3x3 nut: a filled ring with a hollow centre). Rotation (`rotateCW`) transposes + reverses rows; `tryRotate()` applies basic wall-kick offsets (`[0, -1, 1, -2, 2]`) before giving up on a rotation.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and existing filled cells.
- **Locking/scoring**: `lockPiece()` → `merge()` writes the piece into `board`, then `clearLines()` removes full rows (scanning bottom-up) and awards points via `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`. Level increases every 10 lines cleared, and `dropInterval` speeds up accordingly (`max(100, 1000 - (level-1)*90)` ms).
- **Ghost piece**: `ghostY()` projects the current piece straight down; drawn via `draw()` with reduced alpha.
- **Rendering**: `draw()` (main board + grid + ghost + current piece) and `drawNext()` (next-piece preview) are plain Canvas 2D calls; there's no virtual DOM or diffing.
- Game over is triggered in `spawn()` when a freshly spawned piece immediately collides.

Tunable constants are all at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`. If you change `COLS`/`ROWS`/`BLOCK`, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

The README (`README.md`, in Spanish) has additional detail on controls and mechanics if needed.
