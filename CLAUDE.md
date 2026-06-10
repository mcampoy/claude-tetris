# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step required. Open `index.html` directly in a browser, or serve locally:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

On Windows: `start index.html`

## Architecture

Three files, no dependencies, no bundler:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600px) for the main board, `<canvas id="next-canvas">` (120×120px) for the next-piece preview, a side panel with score/lines/level, and an overlay div for pause/game-over states.
- **`style.css`** — Dark/retro arcade theme using CSS variables, flexbox layout, and `backdrop-filter` on overlays.
- **`game.js`** — All game logic (~300 lines, `'use strict'`, no classes).

### game.js internals

- **Board state**: `board` is a `ROWS × COLS` matrix; `0` = empty, `1–7` = piece color index.
- **Piece objects**: `{ type, shape, x, y }` where `shape` is a 2D matrix copy from `PIECES`.
- **Rotation**: `rotateCW(shape)` transposes + reverses rows. `tryRotate()` applies wall kicks with offsets `[0, -1, 1, -2, 2]`.
- **Game loop**: `requestAnimationFrame`-based. `dropAccum` accumulates `dt`; when it exceeds `dropInterval`, the piece drops one row or locks.
- **Lock sequence**: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. If `spawn()` collides immediately, `endGame()` is called.
- **Speed formula**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms. Level increases every 10 lines.

### Key tunable constants in game.js

| Constant | Default | Notes |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | If changed, update canvas `width`/`height` in `index.html` to `COLS×BLOCK` / `ROWS×BLOCK` |
| `BLOCK` | 30px | Cell size in pixels |
| `COLORS` | 7 colors | Index 1–7 maps to pieces I, O, T, S, Z, J, L |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
