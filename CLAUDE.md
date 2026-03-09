# Huisie - Claude Code Guide

> **Note for Claude**: Keep this file up to date. If you make any major changes to the codebase (new features, architectural shifts, renamed functions, changed state shape, etc.), update the relevant sections of this file before finishing.

## Project Overview

Huisie is a single-file, browser-based home cleaning tracker. Everything lives in `index.html` — HTML, CSS, and JavaScript together. No build step, no dependencies, no framework.

## Architecture

- **Single file**: `index.html` contains all styles, markup, and logic
- **No build tools**: Open directly in a browser or serve statically
- **Persistence**: `localStorage` key `huisie` stores full app state as JSON
- **No server**: Pure client-side; deploy via GitHub Pages (root, `main` branch)
- **PWA**: `manifest.json` + `sw.js` (cache-first service worker) + `icon.svg` enable install-to-homescreen and offline use

## Key Concepts

### State Shape
```js
{
  topTasks: [{ id, name, minutes, done, timerRunning, timerSeconds, timerStartedAt }],
  rooms: [{ id, name, open, tasks: [...] }]
}
```

- `timerSeconds` — seconds remaining as of last pause/start; not decremented live
- `timerStartedAt` — Unix ms timestamp when current run began; `null` when stopped

### Shareable Setup Shape (clipboard export)
```js
{
  topTasks: [{ name, minutes }],
  rooms: [{ room, tasks: [{ name, minutes }] }]
}
```

### Important Functions
- `render()` — full re-render of both the top tasks card and room list
- `bindEvents()` — attaches all event listeners after render (called by `render`)
- `bindTask(li, task, rid)` — binds events for a single newly-inserted task `<li>`; `rid` is room id or `'top'`
- `startTimer(taskId)` / `pauseTimer(taskId)` — manage countdown intervals stored in `timers` object; timestamp-based (sets `timerStartedAt`, does not decrement live)
- `computeRemaining(task)` — computes real remaining seconds from `timerStartedAt` + `timerSeconds`; falls back to `timerSeconds` when not running
- `acquireWakeLock()` / `releaseWakeLockIfIdle()` — manage Screen Wake Lock API to keep screen on while timers are active; no-op if API unavailable
- `saveState()` / `loadState()` — localStorage serialization
- `stateToSetup()` / `setupToState()` — convert between runtime state and shareable setup JSON
- `esc(s)` — HTML escape helper used before inserting user content into innerHTML

### Factory Defaults
The `FACTORY` array at the top of the script defines the default rooms and tasks loaded when localStorage is empty.

## Styling

- CSS custom properties defined in `:root` — warm linen palette
- Accent color: sage green (`--accent: #5C9474`)
- Font: Nunito (Google Fonts)
- Fully responsive with a `@media (max-width: 540px)` breakpoint

## UI Structure

- **Header**: brand + "Reset day" and "Copy setup" buttons
- **Top tasks card** (`#topSection`): general tasks not tied to a room
- **Room cards** (`#roomList`): collapsible, each with task list and add-task input
- **Add room section**: dashed card at bottom of room list
- **Bottom bar**: destructive "Reset" button (clears localStorage)

## Common Patterns

- New tasks added inline (no full re-render) using `bindTask()` after DOM insertion
- Progress badges update in-place via `refreshProgress(rid)`
- Timers stored in module-level `timers` object (`taskId -> intervalId`); timestamp-based to avoid drift
- Audio via Web Audio API (`playTick` on completion, `playBeep` on timer expiry)
- Screen Wake Lock acquired when any timer starts, released when all timers stop; re-acquired on `visibilitychange`
