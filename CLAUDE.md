# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

A collection of self-contained browser games. Every game is a **single HTML file** — no build step, no dependencies, no server required. Open the file directly in a browser to play.

## Running Games

```
# Open any game directly in the default browser (Windows)
Start-Process shooter.html
Start-Process tictactoe.html
Start-Process Gemini_game.html
```

There are no build, lint, or test commands — the files run as-is in the browser. Use browser DevTools (F12 → Console) to check for JS errors after changes.

## Git Workflow

All changes are committed and pushed to GitHub. Commit messages follow this pattern:
- `feat: <description>` — new feature or game
- `fix: <description>` — bug fix
- `refactor: <description>` — code restructure without behavior change
- `style: <description>` — visual/UI changes

Always push after committing:
```
git push
```

## Architecture

### Game File Pattern

Each game is a single HTML file with three sections in order: `<style>`, `<body>` (canvas element only), `<script>`. All game logic lives in the script block.

### `shooter.html` — Primary Game (800×600 canvas)

The script is divided into 10 labeled sections (marked with `// ===` block comments):

| Section | Contents |
|---|---|
| 1 | `PAL` color palette object, button rect constants (`BTN_START`, `BTN_RETRY`, `BTN_AGAIN`) |
| 2 | Global state variables: `gameState`, `score`, `currentLevel`, entity arrays, screen shake vars |
| 3 | Input: `keys` dict (keydown/keyup), `mouse` object, `handleClick()` dispatcher |
| 4 | `levelConfig` object — one entry per level with `enemySpeed`, `targetKills`, `spawnRate`, `enemyTypes`, `bossSpawn`, `name` |
| 5 | Entity classes: `Player`, `Bullet`, `EnemyBullet`, `EnemyBasic`, `EnemyFast`, `EnemyTank`, `EnemyShooter`, `EnemyBoss`, `Particle` |
| 6 | Effect helpers: `triggerShake()`, `spawnDeathExplosion()`, `spawnHitParticles()`, `spawnMuzzleFlash()` |
| 7 | Game logic: `updateCollisions()`, `updateSpawning()`, `checkLevelComplete()`, `startGame()`, `nextLevel()` |
| 8 | UI renderers: `drawBackground()`, `drawScanlines()`, `drawVignette()`, `drawMenuScreen()`, `drawHUD()`, `drawGameOverScreen()`, `drawLevelTransitionScreen()`, `drawVictoryScreen()` |
| 9 | `loop()` — `requestAnimationFrame` loop, state-gated update+draw, screen shake translate |
| 10 | Boot: calls `loop()` |

**State machine:** `MENU → PLAYING → LEVEL_TRANSITION → PLAYING` (repeats) or `→ GAMEOVER` / `→ VICTORY`

**Rendering order inside `PLAYING`:** background → smoke particles → enemy bullets → player bullets → enemies → player → non-smoke particles → scanlines → vignette → HUD. Screen shake is applied via `ctx.save()/translate(shakeX, shakeY)/restore()` wrapping steps 1–8.

**All collision detection** uses `circleCollide(a, b)` = `Math.hypot(dx, dy) < a.radius + b.radius`. Arrays are iterated in reverse when splicing to avoid index-skip bugs.

**Every entity class** implements `update()` and `draw()`. Entities with multi-hit health (`EnemyTank`, `EnemyShooter`, `EnemyBoss`) also implement `hit()` for flash effects.

**Particle system:** single `Particle` class with a `type` field (`'square'`, `'spark'`, `'muzzle'`, `'smoke'`) that controls decay rate, gravity, and draw style. All particles live in one global `particles[]` array; capped at 300.

**Adding a new level:** add an entry to `levelConfig` and increment `MAX_LEVELS`. The `enemyTypes` array controls which enemy classes can spawn (random selection each spawn tick).

**Adding a new enemy type:** create a class with `update()`/`draw()`/`hit()`, add a case in `spawnEnemy()`'s switch, and add the type string to the relevant `levelConfig[n].enemyTypes` arrays.

### `Gemini_game.html` — Prototype (800×600 canvas)

Earlier, simpler version of the shooter. Uses HTML div overlays (`.screen` class with `display:none` toggling) for menus instead of canvas-drawn UI. Single `Enemy` class, single `Bullet` class, no ammo/reload system. Useful as a reference for the simpler baseline pattern.

### `tictactoe.html` — DOM-based (no canvas)

CSS Grid layout, event-driven (no game loop). Not relevant to canvas game work.
