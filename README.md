# HD2 Phonebox Widgets

A collection of **standalone, single-file HTML widgets for [Helldivers 2](https://store.steampowered.com/app/553850/Helldivers_2/)**, built to run inside **[PhoneBox](https://github.com/ersinkoc/phonebox)** as custom HTML widgets — or in any modern browser.

Each widget is a self-contained `.html` file with inline CSS and vanilla JavaScript: **no build step, no framework, no backend, no API keys**. Widgets pull live data from the Helldivers 2 community API and refresh automatically.

- **PhoneBox project:** <https://github.com/ersinkoc/phonebox>
- **This project:** <https://github.com/TheDeviceNull/HD2-Phonebox-Widgets>

---

## What is PhoneBox?

[PhoneBox](https://github.com/ersinkoc/phonebox) turns an old Android phone into a customizable desk panel / touch dashboard for Windows, Linux and macOS. It supports a **Custom HTML** widget type (sandboxed), and these files are designed to be added to a PhoneBox page one widget per tile. Everything also works in a plain browser if you just open the file.

## Requirements

- A modern browser (or a PhoneBox Custom HTML widget host).
- Internet access to `api.helldivers2.dev` from the device that renders the widget.
- Nothing else: no server, no build tools, no accounts, no API keys.

## Usage

1. Copy the `.html` files (or clone this repository).
2. In PhoneBox, add a **Custom HTML** widget to a page and point it at the file (or paste its contents), sizing it as you like. One file = one widget.
3. Or simply open any file in a browser. All layouts are responsive; pick small tiles for compact views or large ones for full detail.

Recommended combinations:

- **One phone page** with `hd2-1`, `hd2-2`, `hd2-3`, `hd2-4` as separate tiles.
- **Or** a single full-page widget using `hd2-landscape.html` (designed for landscape screens, works in portrait too).

## Widgets

### `hd2-1-stato-guerra.html` — War status (*Stato guerra*)

Global galactic-war snapshot.

- **Data:** `GET /api/v1/war`
- **Shows:** Helldivers online (player count); missions won / lost; total missions; mission success rate; accuracy; deaths; revives; bullets fired; kills split by faction — **Terminids, Automatons, Illuminate**.
- **Responsiveness:** extra-tiny sizes show just the player count; stat tiles are added as space grows; kill breakdown on medium/large; "Last updated HH:MM:SS" footer on large.
- **Refresh:** every 60 s + random jitter (0–8 s).

### `hd2-2-ordine-maggiore.html` — Major Order (*Ordine Maggiore*)

Active Major Order(s) with live progress.

- **Data:** `GET /api/v1/assignments`, plus `GET /api/v1/planets/{id}` to resolve planet names (fetches at most 4 per cycle; results cached in `localStorage` key `hd2:planets`).
- **Shows:** order title/description and overview; expiration **countdown** ("Ends in"); **reward** (Warbond Medal / medals amount); per-task checklist with progress bars and completion checks.
- **Task types:** Kill enemies (faction, plus specific enemy names via a built-in `ENEMIES` map keyed `"4:<id>"`; unknown enemies shown as `Faction [type:id]`); **Liberate** / **Hold** a planet; **Defend** planets. Task types map to API fields `valueTypes`/`values` (faction type 1, goal type 3, enemy type 4, planet type 12; task types 3 kill, 11 liberate, 12 defend, 13 hold).
- **Known enemies** (name table included): Radicals, Troopers, Shredder Tanks, Agitators, Vox Engines, Chargers, Dragon Roaches, Bile Spewers, Bile Spitters, Voteless, Obtruders, Gatekeepers, Wretches, Crushers, Factory Striders. Unmapped enemies fall back to faction + ID; new entries can be added to the `ENEMIES` object.
- **Smart projection:** for numeric goals it computes the measured progress rate from local history (last hour, up to 80 samples in `localStorage`, key `hd2:hist:<id>`, needs a sample at least ~2 min old) and shows **projected final result %** ("Result at expiry") and **time to completion** ("Complete in").
- **Responsiveness:** smallest sizes show only the first order and compact bars; larger sizes show all active orders, progress detail, projections and reward.
- **Refresh:** every 60 s + jitter; countdowns re-render every 30 s.

### `hd2-3-campagne-attive.html` — Active campaigns (*Campagne attive*)

Frontline overview of every active campaign.

- **Data:** `GET /api/v1/campaigns`
- **Shows:** planet name; sector; campaign type — **Liberation** vs **Defense**; attacking/defending faction with color — **Super Terra/Humans, Terminids, Automatons, Illuminate**; progress bar (% liberated or % defended); **Helldivers on the planet**; extra info per type — defense **ends-in** countdown, or liberation **regen %/h**; faction colors.
- **Sorting:** by number of Helldivers (descending), then progress (descending).
- **Responsiveness:** number shown by size — xs 1, s 2, m 3, l 5; smaller sizes shorten labels and hide secondary rows.
- **Refresh:** every 60 s + jitter.

### `hd2-4-dispacci.html` — Dispatches (*Dispacci*)

Official in-game news feed.

- **Data:** `GET /api/v1/dispatches`
- **Shows:** latest dispatches, **newest first**, with publication timestamp; message text with **HTML tags stripped** and safe escaping.
- **Responsiveness:** number shown by size — xs 1, s 2, m 3, l 5; line clamping adapts to size.
- **Refresh:** every 60 s + jitter.

### `hd2-landscape.html` — Galactic War dashboard (*Guerra Galattica*)

All-in-one combined dashboard in a **2×2 grid** (single column stacked in portrait).

- **Sections:** War status · Major Order · Active campaigns (top 5) · Dispatches (last 6).
- **Layout:** optimized for landscape phones/screens; orientation-aware CSS; auto-fitting (font scaling down to a floor; extra dispatches dropped first) so each section fits its card; per-section status dot + live countdown to next refresh.
- **Scheduler:** independent per-section refresh — war / major order / campaigns every **120 s**, dispatches every **600 s**, plus jitter; requests serialized with a **minimum 2.5 s gap**; localStorage cache with max age (20 min / 12 h); stale cached data shown while fresh; **exponential backoff** on failure (30 s → 5 min max); **HTTP 429** honors `Retry-After` and pauses **all** sections; 20 s per-request timeout, 60 s per-section timeout; on network errors a no-CORS diagnosis probe runs (at most once a minute) and reports why (offline, DNS/VPN/adblock, provider block, server reachable but CORS/429).
- **Debug log panel:** tap **3 times quickly in the bottom-right corner** to toggle (or set `DEBUG_OPEN=true`); buttons **Close / Clear / Copy / Test network**; persistent log in `localStorage` key `hd2:log:land`; countdowns; rate-limit header (`x-ratelimit-*`, `retry-after`) logging; JS error / rejected-promise capture; **network test** checking CORS with `X-Super-*`, `X-Super-Client` only, no-CORS probe, and generic CORS. **Runtime flags at the top of the script:** `EVERY`, `MAXAGE`, `GAP_MS`, `DEBUG_OPEN`, `ENEMIES`, `BASE`, `HEADERS`.

---

## Data source

All widgets read the **Helldivers 2 community API** (no API key required):

- Base URL: `https://api.helldivers2.dev/api/v1`
- Endpoints used: `GET /war`, `GET /assignments`, `GET /campaigns`, `GET /dispatches`, `GET /planets/{id}`
- Required headers sent on every request: `Accept: application/json`, **`X-Super-Client: phonebox-hd2-dashboard`**, **`X-Super-Contact: phonebox-user`**
- **Rate limiting:** the API allows ~5 requests / 10 s; widgets refresh slowly, add random jitter, stagger requests between widgets, honor `Retry-After`, retry with backoff, and (single-widget variants) retry once without custom headers on CORS/network `TypeError`.
- On CORS or network failure a widget shows the error and keeps retrying; it never blocks.

## Common behavior (all widgets)

- **Single file, zero dependencies:** inline CSS + vanilla JS (ES5-style), no modules, no external assets, no CDN.
- **Dark theme:** Helldivers-style palette, yellow accent `#ffe710`, faction colors, subtle borders; fluid typography (`clamp`, `vmin`); `box-sizing: border-box`; no scrollbars on small tiles; touch-friendly viewport.
- **Responsive sizing:** size class auto-detected (`xs` < 150 px height or < 170 px width; `s` < 300; `m` < 560; `l` otherwise) and re-applied on `resize` / `orientationchange`; layout and amount of detail adapt; landscape variant adds orientation media queries.
- **Status indicator:** small dot, bottom-right — gray = loading, green = last refresh OK, red = error.
- **Caching:** `localStorage` (with in-memory fallback) — last good data shown while fresh; keys `hd2:cache:<page>`, `hd2:u:<key>`, `hd2:hist:<id>`, `hd2:planets`, `hd2:log:land`.
- **Safe rendering:** all dynamic strings HTML-escaped; HTML tags stripped from dispatch text.
- **Robustness:** loading and error states, `null`/`NaN`-safe formatting ("–"), numbers formatted for Italian locale (`it-IT`, e.g. `1.234.567`), compact short format (`k` / `M` / `Mld`), time rendering (`h m`, `m s`), retry scheduling, and render-time try/catch.
- **Number/date localization:** `Intl.NumberFormat("it-IT")`, `toLocaleString/TimeString("it-IT")`.

## Language

Widget interface strings (labels, status messages, countdowns) are in **Italian**. Data from the API (planet names, dispatch text, order text) appears as provided. This README is in English.

## License

Distributed under the **GNU General Public License v3.0** — see [LICENSE](LICENSE).

## Credits & thanks

- Built for **[PhoneBox](https://github.com/ersinkoc/phonebox)**. **Thanks to its developer, [ersinkoc](https://github.com/ersinkoc), for the great work** on PhoneBox. 🙏
- Live data by the **Helldivers 2 community API** — <https://api.helldivers2.dev> (unofficial, no API key). Helldivers 2 is © Arrowhead Game Studios / Sony Interactive Entertainment; this project is not affiliated with or endorsed by them.
