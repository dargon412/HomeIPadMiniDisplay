# Changelog

All notable changes to this project are recorded here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added
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
