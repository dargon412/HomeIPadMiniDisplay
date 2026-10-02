# Changelog

All notable changes to this project are recorded here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Changed
- **Target device is now a Samsung Galaxy Tab A9** (1340 × 800, 5:3, Chrome) instead of an
  iPad mini 4. The layout was already relative, so the change is tuning rather than a rewrite:
  a `--topbar-h` variable ties the header height to the `main` calc, and a wide-landscape media
  query (`min-width: 1100px`) enlarges the header, forecast strip, bus chips and calendar text,
  and shifts the split to 40% / 60%. Verified with live API data at 1340 × 800.
- Repository renamed to `dargon412/HomeMiniDisplay`; the Pages URL is now
  <https://dargon412.github.io/HomeMiniDisplay/>.

### Fixed
- **Taps unreliable on iPadOS.** The ⚙️ gear, calendar day taps and the weather box did not
  respond on the iPad, while the month buttons did. `addEventListener('click')` is not dependable
  on non-`<button>` elements in iPadOS Safari — a tap that moves even slightly is treated as a
  scroll and `click` never fires.
  - Added an `onTap()` helper: handles `touchend` when the finger barely moved, with a guard so a
    following synthetic `click` does not double-fire. Falls back to `click` for mouse input.
  - Calendar day taps now land anywhere in the cell (day number, badge, weather icon, event text)
    via `findUp()`, replacing `closest()`.
  - All interactive elements (month nav, day cells, gear, weather box, modal buttons) use `onTap`.

### Added
- `APP_VERSION` constant, printed in the ⚙️ settings footer, so you can tell which build a device
  is running.
- `?v=N` query strings on `style.css` / `config.js` / `settings.js` / `app.js` to defeat Safari's
  aggressive caching. **Bump `N` in `index.html` on every release.**

- **7-day forecast strip** in the header (icon + high/low per day), from the HKO 9-day forecast.
- **Weather icon on the calendar** for the next 7 days, beside the shift badge.
- **Tap the weather box** to open a detail popup: full short-term forecast paragraph, update time,
  active warning, and a 7-day text forecast list.
- **Auto-refresh interval** setting (10 s – 10 min), replacing the fixed 30 s.
- **Auto-refresh pauses while interacting** — settings open, typing, stop search, or within 4 s of
  a tap — and resumes automatically.
- **GPS location detection** (`📍 使用我的位置`) picks the nearest of 27 HKO weather stations;
  runs automatically on first visit.
- Developer documentation under `docs/` (`ARCHITECTURE.md`, `API.md`, `DATA-STORAGE.md`,
  `DEVELOPMENT.md`) and this changelog.

### Changed
- Header shows the concise **daily** weather summary instead of the long short-term paragraph;
  the full text moved to the popup. Widened the summary area.
- Roster badges now show only the label (`早` / `夜` / `休`) — the `M` / `N` / `L` letters are no
  longer printed. The legend is generated from the configured cycle.

### Fixed
- **Settings page would not open.** An infinite recursion (`sectionStops` → `refreshStopList` →
  `rerender` → `render` → `sectionStops`) aborted the open. A `fillStopList(container)` helper now
  does the initial fill so no lookup-on-missing-node occurs.

## [0.1.0] — initial build

### Added
- Always-on dashboard web app: live clock and date.
- Bus ETA panel: next 3 arrivals for configured stops/routes, KMB/LWB and Citybus.
- Calendar with 4-day roster (M → N → L → L) computed from an anchor date.
- Per-day events stored in `localStorage`.
- Current weather from the Hong Kong Observatory.
- On-screen settings page with an integrated bus-stop finder.
- Export/import of settings as `config.js` or JSON.
- `tools.html` standalone stop finder.
- README with setup, GitHub, GitHub Pages and iPad kiosk instructions.
