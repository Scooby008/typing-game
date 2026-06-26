# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file browser game: "Type Racer GP," a typing-practice game styled as a racing game. The entire app — HTML, CSS, and JavaScript — lives in `index.html`. There is no build step, no package.json, and no dependencies.

## Running the game

Open `index.html` directly in a browser:

```bash
open index.html
```

There is no dev server, bundler, linter, or test suite. Edits to `index.html` take effect on next page load/refresh.

## Architecture

Everything is in one IIFE at the bottom of `index.html`. Key pieces:

- **Word/phrase banks** (`banks.easy/normal/hard`) — the content typed during races. Phrases (not single words) for race difficulty; `testPool` derives single words from these banks for the typing test.
- **Audio** — all sound effects are synthesized at runtime via Web Audio (`beep()` + the `sfx` object). No audio files.
- **Race state** — `racers[]` holds 4 cars (player at index 0, three AI rivals). Each has a `progress` value (0 to `RACE_DISTANCE`) driven by correct keystrokes (player) or `aiTick()` (AI, randomized per-frame increments). `requestAnimationFrame` loop (`raceLoop`) advances AI and re-renders car positions each frame.
- **Typing logic** — `handleKeydown()` is the global keydown handler for race mode; it diffs typed characters against the current phrase, updates `streak`/`nitro`/`progress`, and triggers screen-shake/sfx on mistakes. Completing a phrase calls `nextWord()` and applies a score bonus (boosted by `nitro >= 80`).
- **Typing test mode** — a parallel, separate state machine (`testActive`/`testRunning`, `handleTestKeydown()`, `finishTest()`) for the 30-second skill-check screen. It does not touch race state. The global keydown listener routes to either `handleTestKeydown` or `handleKeydown` based on `testRunning || testActive`.
- **Scoreboard** — `updateScoreboard()` ranks all 4 racers by `progress` and renders live WPM (real for the player via `updateWPM()`, simulated for AI via `racer.simWpm`) into the `#scoreboard` panel; a similar final ranking renders into `#finalScoreboard` at race end.
- **Screens** — plain `.overlay` divs toggled via the `hidden` class: start screen, 30s typing test screen, typing test results screen, and race results screen. No router/framework; visibility is managed directly through `classList`.

## Adding content

- New phrases/words: add to the appropriate array in `banks` (`easy`/`normal`/`hard`). `testPool` automatically picks up single-word entries from `easy`/`normal` (it splits phrases and filters to `^[a-z]+$` tokens), so no separate update needed there.
- New sound effects: add an entry to the `sfx` object using `beep(freq, dur, type, vol)` calls.
