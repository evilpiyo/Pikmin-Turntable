# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, static HTML page (`index.html`) — no build step, no package manager, no dependencies. It's a timing tool for the "轉盤" (turntable) minigame in the mobile game Pikmin Bloom: six pots arranged in a hexagon each need to be tapped at a specific offset (in 0.5s steps) after the round starts, and this page plays a stopwatch that highlights each pot at its target time so a player can rehearse the click timing.

`_Ref/規則心得.png` is a screenshot of the community post (Chinese, Pikmin Bloom Taiwan discussion group) documenting the observed per-pot timing pattern this tool implements — background/domain reference only, not something to edit or treat as a spec to keep in sync with code.

## Running it

Open `index.html` directly in a browser (double-click, or serve the directory with any static file server). There is no build, lint, or test tooling in this repo.

## Architecture

Everything — markup, CSS, and JS — lives in `index.html`. Key pieces:

- **Hexagon layout**: the six `.pot` elements are positioned via `data-order="0"`–`"5"` selectors in CSS (`left`/`top` percentages), going clockwise from the bottom. The order value also drives timing: `potSeconds()` = base seconds input + `order * 0.5`.
- **Pot type/icon**: each pot has `data-type` (`pot` / `sprout` / `other`), cycled by clicking a pot (only when not running). This only changes the emoji shown (`ICONS` map) — it does not affect timing, which is purely a function of `data-order`.
- **Timer loop**: `startRun()`/`stopRun()`/`tick()` implement the stopwatch using `requestAnimationFrame` and `performance.now()`. On each frame, every pot's `.active` class is toggled based on whether elapsed time falls within `HIGHLIGHT_WINDOW` (0.4s) of its target `potSeconds()`. The run auto-stops once elapsed time passes the last pot's target + the highlight window.
- **Base seconds input** (`#baseSeconds`): sets the bottom pot's (order 0) target seconds; the other five pots are derived by adding `0.5s` per step clockwise. Editing it live-updates all pot labels via `refreshLabels()`. `#decBtn`/`#incBtn` are touch-sized +/- buttons that nudge the same input by `STEP` via `adjustBase()`; both get disabled together with the input during a run.
- **Reset button** (`#resetBtn`, inside `.timer`, below the `00.00` readout): intentionally just calls `location.reload()` — a full page refresh — rather than reusing `stopRun()`. It's meant to be a hard reset of all state (including any pot type the player cycled), not a soft stop.
- **Help modal** (`#helpOverlay`/`#helpBtn`/`#helpClose`): a simple show/hide overlay (`.open` class toggled by `openHelp()`/`closeHelp()`) triggered from the `?` button at the right end of `.controls`. Closes on the close button or on clicking the backdrop. Content is static instructional text, not derived from app state.
- **Decorative-only**: the starfield (`#stars`, `initStars()`), the flower/grass/Pikmin field, and the `.copyright` line are pure visual flourish with no interaction or state.

When changing pot timing or layout, keep `data-order` (CSS position), `potSeconds()` (timing math), and the initial `.pot-sec` label text in `index.html` consistent with each other — there's no single source of truth to derive them from automatically.

## Changing the pot-type emojis (🔮/👑/🥙)

The three pot-type icons are duplicated in four places in `index.html` — all four must be updated together or the initial page state, the hint text, and the help modal will drift from what clicking actually produces:

1. **`ICONS` map** (JS) — `const ICONS = { pot: '👑', sprout: '🥙', other: '🔮' };`. This is the actual source of truth used when a pot is clicked and cycles type (`pot.querySelector('.pot-icon').textContent = ICONS[nextType]`).
2. **Initial `.pot-icon` spans** (HTML, the six `.pot` elements in `.stage`) — all six pots start as `data-type="other"` and hardcode `🔮` as their initial glyph. This must match `ICONS.other`.
3. **Hint text** (HTML, `.hint`) — `點擊卡片可切換 🔮其他 ／ 👑金盆 ／ 🥙大花苗` spells out all three emojis for the player; must match `ICONS` in the same order (other → pot → sprout as cycled by `TYPE_CYCLE`).
4. **Help modal body** (HTML, inside `#helpOverlay .modal ol`) — repeats the same `🔮其他／👑金盆／🥙大花苗` list in its own `<li>`.

(Line numbers are omitted here deliberately since they drift as the file is edited — search for the strings above instead.)

`💠` (flowers) and the CSS-drawn Pikmin field are unrelated decoration, not part of the pot-type icon set.
