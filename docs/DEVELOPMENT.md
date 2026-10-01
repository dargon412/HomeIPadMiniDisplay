# Development guide

For working on the app on a computer.

## Running locally

No build step, no dependencies. Just serve the folder over HTTP (needed so `fetch` and
`localStorage` behave like they will on the iPad):

```bash
cd ipad-dashboard
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Stop with `Ctrl + C`.

> Opening `index.html` via `file://` mostly works but can block `fetch`, so prefer the server.

Editing tip: hard-reload after changing CSS/JS to beat the cache — `⌘ + Shift + R`
(Chrome/Firefox) or `⌘ + ⌥ + R` (Safari).

## Testing at the real screen size

The iPad mini 4 is **1024 × 768 landscape** (and 1536 physical px wide @2x). Use the browser
responsive mode set to that size. The layout has a **portrait fallback** under
`@media (orientation: portrait)`.

Check every change at 1024×768, because the header is dense and the calendar grid has exactly
enough room for 6 week-rows.

## Browser compatibility rules

The only real target is **iPadOS 15 Safari**. Keep the JavaScript conservative:

**Available:** `var`/`let`/`const`, arrow functions, template literals, `Promise`, `fetch`,
`AbortController`, `classList`, `URL`/`Blob`, `navigator.clipboard` (guarded), `Object.assign`.

**Avoid:**

| Feature | Use instead |
|---|---|
| `?.` optional chaining | `a && a.b` |
| `??` nullish coalescing | `a != null ? a : b` |
| `Array.prototype.at()` | `arr[arr.length - 1]` |
| Top-level `await` | `.then()` or an IIFE |
| Class fields / `#private` | plain constructor assignments |
| `structuredClone` | `JSON.parse(JSON.stringify(x))` |

The codebase deliberately reads a little old-fashioned. That is intentional.

## Code map (`app.js`)

Sections are marked by banner comments:

| Section | Key functions |
|---|---|
| Helpers | `$`, `el`, `pad2`, `ymd`, `parseYmd`, `fetchJson` |
| Config | `normalizeConfig`, `CFG` |
| Clock | `tickClock` |
| Weather | `iconEmoji`, `pickTemperature`, `nearestStation`, `detectAndApply`, `renderForecast`, `loadWeather`, `openWeatherDetail` |
| Bus | `etaUrl`, `minutesUntil`, `buildBusSkeleton`, `renderRoute`, `loadBus` |
| Calendar | `rosterLetter`, `renderCalendar`, `openModal`, `addEvent` |
| Wiring | `bindUi`, `reloadDashboard`, `isInteracting`, `scheduleRefresh`, `start` |

`app.js` runs `start()` on `DOMContentLoaded` (or immediately if already loaded).

## Code map (`settings.js`)

Wrapped in an IIFE that exposes two globals:

- `window.DashboardConfig = { load, save, reset, defaults }`
- `window.openSettings()`

Internally it keeps a `working` copy of the config while you edit, and only writes it when you
press **儲存並套用**.

## Common change recipes

### Add a field to settings

1. Add the default in `config.js` and in `settings.js → defaults()`.
2. Add it to `load()` in `settings.js` so saved copies merge it.
3. Add a control in the relevant `sectionX()` function. Use `makeField()` / `textInput()` /
   `selectInput()`.
4. Normalize/consume it in `app.js` (`normalizeConfig()` or wherever it's used).
5. Update `docs/DATA-STORAGE.md`.

### Add a new data refresh

1. Write `loadSomething()` returning a Promise, using `fetchJson`.
2. Add it to `scheduleRefresh()`:
   ```js
   refreshTimers.push(setInterval(function () {
     if (!isInteracting()) loadSomething();
   }, ms));
   ```
3. Call it once in `reloadDashboard()` and `start()`.

### Change the header height

The header's height is used to size `main`. Update **both**:

- `style.css` → `#topbar { height: ... }`
- `style.css` → `main { height: calc(100% - ...) }`

## Debugging

The app is plain browser code, so the browser's own tools are best:

- **Safari on the Mac:** with the iPad connected, *Develop → [iPad] → [page]*.
- **Console:** `DashboardConfig.load()` prints the effective config; `localStorage.getItem('homeDashboard.config.v1')` prints the raw saved copy.
- A syntax error in `config.js` blanks the page. Check the console first.

### Testing without hitting the network

You can stub `fetch` before the app scripts run to inject fixed JSON for the weather or bus APIs.
This is how the features were verified during development, and it's a good pattern to reuse.

## What a good change looks like

- No new dependencies.
- Degrades gracefully offline (individual `.catch()` per request).
- Doesn't break the 1024×768 layout.
- Auto-refresh still pauses during interaction.
- `CHANGELOG.md` updated.
