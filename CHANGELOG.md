# Changelog

All notable changes to this project are recorded here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Fixed
- **Tapping a calendar day while the add-event popup was open switched the popup to that day.**
  The device log showed the tap target was `DIV#modal` (the overlay), **not** a calendar cell, yet
  the day cell handler still ran. The earlier guard only acted when the target was inside the
  calendar, so it bailed out and did nothing. The capture-phase guard now blocks by **exclusion**:
  while the popup is open it swallows every `touchstart`/`touchend`/`click` except those on the
  popup's own controls, and closes the popup once per gesture. `openModal()` also refuses to switch
  the date of an already-open popup as a safety net.
- Import failed with `JSON.parse: unexpected character at line 1 column 1` when fed the exported
  `config.js` (a comment plus `const CONFIG = ...;`). `parseBackup()` now unwraps both formats.
- **Taps unreliable on iPadOS.** The ⚙️ gear, calendar day taps and the weather box did not
  respond on the iPad, while the month buttons did. `addEventListener('click')` is not dependable
  on non-`<button>` elements in iPadOS Safari — a tap that moves even slightly is treated as a
  scroll and `click` never fires. Added an `onTap()` helper (handles `touchend` when the finger
  barely moved, guards against the touch+click double-fire, falls back to `click` for mouse), and
  all interactive elements now use it.
- **Settings page would not open.** An infinite recursion (`sectionStops` → `refreshStopList` →
  `rerender` → `render` → `sectionStops`) aborted the open. A `fillStopList(container)` helper now
  does the initial fill so no lookup-on-missing-node occurs.

### Added
- **Single combined backup button (⬇️ 匯出備份 Backup).** Replaces the separate `匯出 config.js` /
  `匯出 JSON 備份` / `複製設定 JSON` buttons. It downloads one `home-dashboard-backup.json`
  containing the config **and all calendar events**, and the matching 匯入 button restores both.
  Import still accepts the older plain-JSON and `config.js` backups (config only).
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
- **Target device is now a Samsung Galaxy Tab A9** (1340 × 800, 5:3, Chrome) instead of an
  iPad mini 4. The layout was already relative, so the change is tuning rather than a rewrite:
  a `--topbar-h` variable ties the header height to the `main` calc, and a wide-landscape media
  query (`min-width: 1100px`) enlarges the header, forecast strip, bus chips and calendar text,
  and shifts the split to 40% / 60%. Verified with live API data at 1340 × 800.
- Repository renamed to `dargon412/HomeMiniDisplay`; the Pages URL is now
  <https://dargon412.github.io/HomeMiniDisplay/>.
- Header shows the concise **daily** weather summary instead of the long short-term paragraph;
  the full text moved to the popup. Widened the summary area.
- Roster badges now show only the label (`早` / `夜` / `休`) — the `M` / `N` / `L` letters are no
  longer printed. The legend is generated from the configured cycle.

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
