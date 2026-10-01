# External APIs

All data comes from Hong Kong Government open data. No API key, no rate limit, free.
All three hosts send `access-control-allow-origin: *`.

## 1. KMB / Long Win bus ETA

Host: `https://data.etabus.gov.hk` · version path `/v1/transport/kmb/`

### Get real-time ETA for one stop + one route

```
GET /v1/transport/kmb/eta/{stopId}/{route}/{serviceType}
```

Example: `https://data.etabus.gov.hk/v1/transport/kmb/eta/DCFF4041D0C0ACF8/1A/1`

```json
{
  "type": "ETA", "version": "1.0",
  "generated_timestamp": "2026-10-02T00:26:43+08:00",
  "data": [
    {
      "co": "KMB",
      "route": "1A",
      "dir": "O",
      "service_type": 1,
      "seq": 35,
      "dest_tc": "尖沙咀碼頭",
      "dest_en": "STAR FERRY",
      "eta_seq": 1,
      "eta": "2026-10-02T00:25:17+08:00",
      "rmk_tc": "最後班次",
      "data_timestamp": "2026-10-02T00:26:19+08:00"
    }
  ]
}
```

- `data` is already ordered (next bus first). We use `data[0..2]`.
- `eta` can be `null` or in the past. `minutesUntil()` handles this.
- `rmk_tc` may be `"最後班次"` (final bus) — shown as a small note.

### Direction values

The documented direction codes are `O` (outbound) and `I` (inbound), **but the live public API
rejected them** during development. What actually works is the words:

```
GET /v1/transport/kmb/route-stop/{route}/outbound/{serviceType}
GET /v1/transport/kmb/route-stop/{route}/inbound/{serviceType}
```

Use `outbound` / `inbound` in code. This was verified against the live API.

### Other endpoints used by `tools.html` / settings

| Purpose | Endpoint |
|---|---|
| All routes | `/v1/transport/kmb/route/` |
| One route detail | `/v1/transport/kmb/route/{route}/{dir}/{serviceType}` |
| Stops on a route | `/v1/transport/kmb/route-stop/{route}/{dir}/{serviceType}` |
| One stop detail | `/v1/transport/kmb/stop/{stopId}` |
| All stops (big, ~5 MB) | `/v1/transport/kmb/stop/` |

KMB stop IDs are 16-character uppercase hex, e.g. `DCFF4041D0C0ACF8`.

### service_type

A number distinguishing variants of the same route number. `1` is the normal service and is
correct for the large majority of routes. 181 of 796 routes have more than one type. If a route
shows no buses when you expect some, try `2` or `3`. LWB uses the same endpoint.

## 2. Citybus ETA

Host: `https://rt.data.gov.hk` · version path `/v2/transport/citybus/`

### Get ETA

```
GET /v2/transport/citybus/eta/{company}/{stopId}/{route}
```

`company` is `CTB` (Citybus). Example:
`https://rt.data.gov.hk/v2/transport/citybus/eta/CTB/002410/5B`

Response is the same shape as KMB, except:

- `service_type` is absent — Citybus doesn't use it.
- Stop IDs are 6-digit strings, e.g. `002410`.

### Other endpoints

| Purpose | Endpoint |
|---|---|
| One route detail | `/v2/transport/citybus/route/{company}/{route}` |
| Stops on a route | `/v2/transport/citybus/route-stop/{company}/{route}/{dir}` |
| One stop detail | `/v2/transport/citybus/stop/{stopId}` |

`dir` is `outbound` or `inbound` (same words as KMB).

## 3. Hong Kong Observatory weather

Host: `https://data.weather.gov.hk` · path `/weatherAPI/opendata/weather.php`

Common query: `?lang={tc|sc|en}&dataType={type}`

### `rhrread` — current readings

```json
{
  "temperature": { "data": [ { "place": "沙田", "value": 28, "unit": "C" } ] },
  "humidity":    { "data": [ { "value": 80, "unit": "percent" } ] },
  "uvindex":     { "data": [ { "value": 3, "desc": "中" } ] },
  "icon":        [ 63 ],
  "warningMessage": "",
  "updateTime": "2026-10-02T00:45:00+08:00"
}
```

`temperature.data` is a list of 27 districts. `pickTemperature()` prefers `CFG.weather.district`,
then falls back to 香港天文台, then the first entry.

### `flw` — short-term forecast (the paragraph)

```json
{
  "forecastDesc": "大致多雲，有幾陣驟雨及狂風雷暴，部分地區雨勢較大，氣溫介乎27至31度。吹微風…",
  "forecastPeriod": "本港地區天氣預報",
  "updateTime": "2026-10-02T00:45:00+08:00",
  "warningMessage": ""
}
```

Typically ~50 characters — too long for the header, so it lives in the popup.

### `fnd` — 9-day forecast

```json
{
  "weatherForecast": [
    {
      "forecastDate": "20261002",
      "week": "星期五",
      "forecastWeather": "大致多雲，有幾陣驟雨及狂風雷暴，部分地區雨勢較大。",
      "forecastMaxtemp": { "value": 31, "unit": "C" },
      "forecastMintemp": { "value": 27, "unit": "C" },
      "ForecastIcon": 63,
      "PSR": "高"
    }
  ]
}
```

Notes:

- `forecastDate` has **no dashes** (`YYYYMMDD`). `fndKey()` converts it to `YYYY-MM-DD`.
- The icon field is **`ForecastIcon`** with a capital F (unlike `rhrread`'s `icon`).
- Daily summaries average ~15 characters, which is why the header uses this field.
- There are 9 days; we display 7.

### Weather icons

HKO icon codes are mapped to emoji in `app.js` → `HKO_ICONS` / `iconEmoji()`.
Codes are grouped by tens: 50s sunny, 60s rain, 70s fog, 80s windy, 90s extreme.
`iconEmoji` falls back to the nearest decade so unknown codes still render something sensible.

## Error handling rules

- Every fetch goes through `fetchJson(url, timeoutMs)` which uses `AbortController` for a timeout
  and rejects on non-2xx.
- Each call site `.catch()`-es individually so one failed request can't blank the screen.
- A failed bus row shows **載入失敗**; it will retry on the next cycle.
- `HTTP 422` from KMB/Citybus usually means a bad stop ID, route, or service type.
