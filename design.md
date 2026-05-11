# Design

Reference doc for the **Beach Trip 2026** deck (`index.html` + `README.md`). Use this when editing to keep visual and structural consistency.

## Site overview

- **What:** Single-page Vietnamese itinerary deck for a 3-day team trip (16–18/5/2026, 6 people), comparing Quy Nhơn ⭐ (recommended) vs Phú Yên.
- **Where:** GitHub Pages from `master` root → `https://minhvuongrbs.github.io/may2026/`
- **Stack:** Static HTML + inline CSS + inline JS. No build step. Leaflet (~40 KB via unpkg CDN) for the interactive map.
- **Mirror:** `README.md` is a plain-markdown mirror so the repo is readable on GitHub.

## File structure

```
.
├── index.html          # the deck (single file, ~1500 lines)
├── README.md           # markdown mirror
├── design.md           # this file
└── images/
    ├── quy-nhon-hero.jpg     # hero background
    ├── quy-nhon-banner.jpg   # QN option banner
    ├── phu-yen.jpg           # PY option banner
    └── vinh-hy-banner.png    # legacy, kept for cleanup
```

## Information architecture

Top-down sections in `index.html`:

1. **Hero** — big "2026" year, title, lede, calendar+group meta pill
2. **Mục tiêu chuyến đi** — 3 emoji goal cards (🏖️🦐🚌)
3. **Phương án 01: Quy Nhơn** ⭐
   1. Header + meta + hero image
   2. 3 feature callouts (🤿🍤🚐)
   3. Why section
   4. **Lodging card** — Cá House (only on QN)
   5. Itinerary heading
   6. Day 0 travel ribbon (departure)
   7. **Interactive map section** — Day 1/2/3 tabs (only on QN)
   8. Day 1/2/3 cards (text view, mirrors map)
   9. Day 4 travel ribbon (return)
   10. **Food guide** — 5 cards (only on QN)
   11. **Add-on note** — Quang Trung sidebar (only on QN)
4. **Phương án 02: Phú Yên** — same outer skeleton, no lodging/map/food/add-on (kept lean as the alternative)
5. **Bảng đối chiếu** — 9-row comparison table, "Đề xuất" pick-badge on QN
6. **Chi tiết ngân sách** — 2 budget cards side by side
7. **Footer**

## Visual design

### Color palette (CSS variables on `:root`)

| Token | Value | Use |
|---|---|---|
| `--bg` | `#fafaf7` | Page background (warm cream) |
| `--surface` | `#ffffff` | Card backgrounds |
| `--ink` | `#1a1a1a` | Primary text |
| `--ink-soft` | `#555` | Secondary text |
| `--ink-faint` | `#888` | Tertiary text, labels |
| `--line` | `#ececec` | Borders, dashed dividers |
| `--accent` | `#e85a3c` | Coral — badge, pins, eyebrows, key highlights |
| `--accent-soft` | `#fdeae5` | Soft coral — `winner` cells, active stop card |
| `--shadow-sm` | `0 1px 3px rgba(0,0,0,0.04)` | Subtle card lift |
| `--shadow-md` | `0 4px 16px rgba(0,0,0,0.06)` | Hero image, hover state |

Two literal yellows used only in `.add-on-note`: bg `#fff8e6`, border `#f0e0a8`.

### Typography

- **Body:** Inter, 17 px base, line-height 1.6
- **Display / headings:** Fraunces serif, weight 500, letter-spacing −0.02 em
- **Mono:** SF Mono / Menlo for time stamps and footer
- **Eyebrows / period labels:** 11 px, weight 600, letter-spacing 0.12 em, uppercase, accent color

### Spacing

- `.wrap` — `max-width: 1080px; padding: 0 32px` (20 px on mobile)
- Section vertical padding: `72px 0`
- Card padding: 24–32 px
- Card radius: 12–16 px

## Component patterns

### Section header

```html
<div class="section-eyebrow">EYEBROW LABEL</div>
<h2 class="section-title">Big Fraunces title.</h2>
<p class="section-sub">Optional subtitle in soft ink, max-width 640.</p>
```

### Emoji feature card

Used in goals grid + per-option features. 3-col on desktop, 1-col below 800 px.

```html
<div class="expect-card">
  <div class="icon">🏖️</div>
  <h3>Title</h3>
  <p>Body in ink-soft.</p>
</div>
```

### Travel ribbon (Day 0 / Day 4)

Coral left-border + icon column + 2 legs (ĐN / HCM). Keeps daily itinerary cards focused on exploration days only.

### Day card (Day 1–3 in `.days` grid)

3-col grid. Each card has:
- `.day-num` — accent eyebrow ("Day 02 · Main biển ⭐⭐")
- `.day-date` — faint date ("Chủ Nhật · 17/5/2026")
- `<h4>` — Fraunces card title
- Stack of `.slot` rows:

```html
<div class="slot">
  <span class="when">5h–9h</span>
  <p>Slot description in ink-soft.</p>
</div>
```

Time range goes in `.when` (10–11 px uppercase accent). Body in `<p>` ink-soft.

### Lodging card

Used once (Cá House in QN). Two-column: emoji icon + info block with eyebrow, title, address, stats list. Coral left border to match other "key info" callouts.

### Comparison table (`.compare-table`)

- Col 1: tiêu chí (faint mono label, 22% width)
- Col 2: QN with `<span class="pick-badge">Đề xuất</span>` in header
- Col 3: PY (no badge)
- Cells with class `winner` get `--accent-soft` background — that's the visual cue for "this option wins this row"

### Budget card

`.budget-card` with `.row` items, `.row.total` for the bold final line. Numbers right-aligned via flex justify-between, monospace tabular numerals.

### Map section (QN only)

- Day tabs row at top — pill buttons (`Day 1 · T7`, `Day 2 · CN`, `Day 3 · T2`), active = ink bg
- 2-col body: Leaflet canvas (left, ~60%) + scrollable panel (right, ~40%)
- Pins: 30 px coral circle + 3 px white border + bold number, rendered via Leaflet `divIcon`
- Route: dashed coral polyline `#e85a3c`, weight 3, opacity 0.7
- Panel: eyebrow + title + sub + "Open route in Google Maps" button + period-grouped stop cards
- Lazy init via IntersectionObserver; `map.invalidateSize()` runs 80 ms after first paint to fix Leaflet's known display-grid sizing issue
- Mobile (≤ 800 px): stacks map (360 px) above panel (max 520 px)

Data lives in the `QN_DAYS` JS object:
```js
QN_DAYS[dayKey] = {
  eyebrow, title, subtitle,
  stops: [{ num, time, period, name, desc, lat, lng }, ...]
}
```

Tile layer: CartoDB Voyager (light, free, no API key). Attribution to OSM + CARTO is required.

### Food guide (QN only)

`.food-grid` of `.food-card` (3-col → 1-col below 800 px). Each card:
```html
<div class="food-card">
  <div class="icon">🍜</div>
  <h4>Category</h4>
  <ul>
    <li><b>Quán bold</b> — address · note · <a href="maps-link">📍</a></li>
  </ul>
</div>
```
Dashed dividers between `<li>`.

### Add-on note

`.add-on-note` — soft yellow callout (`#fff8e6` bg, `#f0e0a8` border) with 💡 icon. Used as a lightweight signal: this is optional / secondary info, not part of the main plan.

## Conventions

### Language

- All copy in **Vietnamese**.
- Day-of-week mostly in full: Thứ Sáu / Thứ Bảy / Chủ Nhật / Thứ Hai / Thứ Ba. Short forms (T6, T7, CN, T2, T3) appear in day-num eyebrows where space matters.
- Dates: `DD/M/YYYY` (Vietnamese norm) — e.g. `16/5/2026`.
- Times: `Xh` or `Xh30`, ranges with en-dash `5h–9h`. No AM/PM.

### Iconography

Keep emoji usage consistent so readers learn the vocabulary:
- 🚌 departure ribbon · 🏠 return ribbon · 🏡 lodging
- 🏖️ beach · 🦐 seafood · 🚌 transport — goals
- 🤿 snorkel · 🍤 seafood · 🚐 transport — QN features
- 🪨 geology · 🌅 scenery · 🐟 specialty — PY features
- 🍜 bún · 🥞 đặc sản trưa · 🦐 hải sản · ☕ cafe · 🎁 đặc sản đem về — food guide
- ⭐ priority day · ⭐⭐ main day · 💡 sidebar note
- 📍 inline Google Maps link

### Star markers

`⭐` after day-num = priority day. `⭐⭐` = main / peak day. Used on Day 1 + Day 2 of each option's itinerary.

### "Đề xuất" badge

Pick-badge appears only on Quy Nhơn (option header + comparison column header). PY has none. To flip the recommendation, move the badge and re-mark `winner` cells.

## Responsive

Single major breakpoint at **800 px** (a few places use 700 px). Below it:
- All multi-col grids collapse to 1-col
- Map stacks: map (360 px) on top, panel below
- Compare table cells get smaller padding/font; pick-badge wraps to its own line
- Travel ribbon stacks tag above legs

## Lazy init + animation

- `.reveal` class + IntersectionObserver fades cards in at scroll (rootMargin −40 px, fallback `setTimeout` 1.5 s shows everything if observer fails)
- Map initializes only when the map element enters viewport (rootMargin 120 px)
- After first map paint, `map.invalidateSize()` runs to recompute size inside the grid container

## Deploy

1. Edit `index.html` + `README.md` (keep them in sync — the README mirror is the canonical plain-text view)
2. Commit + push to `master`
3. GitHub Pages auto-rebuilds (~30 s) → live URL
4. Verify with:
   ```bash
   gh api /repos/minhvuongrbs/may2026/pages/builds/latest \
     --jq '{status, error: .error.message, commit: .commit[:7]}'
   ```

No GitHub Action workflow — Pages uses legacy build_type with branch `master`, path `/`.

## Future-proofing notes

- **New option (3rd destination):** copy the QN section structure; decide whether to give it lodging/map/food/add-on or keep it lean like PY.
- **Flipping the recommendation:** move the `pick-badge` between option headers, then re-mark `winner` cells across the comparison table.
- **Coordinate accuracy:** for new pins, either query Nominatim (`https://nominatim.openstreetmap.org/search?format=json&q=...`) or paste a user-provided Google Maps short link (e.g. `maps.app.goo.gl/...`) and resolve with `curl -sIL` — the `Location` header redirects to a full URL whose `!3dLAT!4dLNG` segment is the exact place coordinate.
- **New food entry:** add `<li>` in the appropriate food-card; if the place has a Google Maps link, append `· <a href="..." target="_blank" rel="noopener">📍</a>`.
- **Adding a map for PY:** mirror the QN map section structure. Reuse `.map-section` CSS as-is; create a parallel `PY_DAYS` object and a second `(function initPYMap(){ ... })()` block. The Leaflet library is already loaded.
