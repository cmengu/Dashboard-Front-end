# Feedback — Patient Tracker & PFS Module (table-heavy pages)

Screenshot review, 3 Jul 2026. Companion to `feedback.md` (Parts 1–3) and the
design docs. These two pages share one diagnosis: **the "2005 admin panel" look
has a precise anatomy**, and every element of it is fixable on MUI DataGrid
Community — the primitive feel is unthemed-DataGrid + pill-everything, not the
library.

---

## 1. Why it reads "2005", by signal (loudest first)

### 1.1 The saturated blue header band
A solid `#1976d2` bar, white text, vertical column separators = the default look
of 2000s enterprise grids. Modern tables (Linear, Stripe, Notion) invert it: the
**header is quieter than the data** — white or `--surface-alt` background,
12px/600 `--text-muted` labels, zero vertical separators, one hairline below,
sticky. The data is the star; the header whispers.

### 1.2 Pill soup
Six-plus chips per row: ARM badge, "No" pill, "Triplet" pill, Y/N circles,
response pill, PD pill. **When everything is a pill, nothing is.** A chip is a
highlighting device — its value comes from scarcity. Rule: chips on at most 1–2
truly stateful columns (Best Response; PFS Event/Censored). Plain text
everywhere else.

### 1.3 ⚠ Semantic color BUG (correctness, not style)
- **"Progressive Disease: Y" renders as a GREEN badge.** Green = good;
  progression = the worst outcome on the page. Must be red/error.
- SD is alarm-orange (the most common, neutral outcome) while PD — the actually
  bad one — sits gray. Inverted salience.
- Fix: one clinical color map in `chartTheme.js` used by charts AND table cells:
  PD = `--error`, PR/CR = positive, SD = neutral gray. Y/N flags colored by
  whether Y is good or bad *for that column*, never blanket green-Y.

### 1.4 Repetition noise
"Demo Center Sir…" (truncated!) ×12 rows, "B" ×12, "Triplet" ×12, "No" ×12.
A column that is ~100% one value carries no per-row information — it's context,
not data. Fixes, in preference order:
1. Lift site/arm out of the table — that's what the filter bar is for; when
   filtered to one site, repeating its name every row is pure noise.
2. Render the default/majority value as muted text or em-dash; visual weight
   only on exceptions.

### 1.5 Decorated numbers + gradient hero
- Stat strips color each number differently (blue/orange/red/purple) — color as
  decoration, violating the color grammar. All stat numbers `--text-strong`;
  underline-style cards → proper StatCards.
- PFS page shows the same pair twice (19 / 24.7, 19 / 24.7) as four cards —
  collapse or differentiate (the "(With Scans)" distinction needs to be the
  headline, not a suffix).
- The indigo **gradient hero banner** with centered italic white subtitle is the
  most "Bootstrap template" element in all screenshots. Both pages →
  `PageHeader` (20px/600 left title + freshness stamp), magenta cap = once.
- Floating circular filter FAB overlapping the banner → filters belong in the
  §2.8 two-tier bar, not a FAB (FABs are mobile-app affordances).

---

## 2. The 2026 table spec (all achievable on DataGrid Community)

| Element | Current (2005) | Target (2026) |
|---|---|---|
| Header | Blue band, white text, separators | `--surface-alt`, 12px/600 muted, no separators, hairline below, sticky |
| Rows | ~38px, zebra-ish | 44px, no zebra, hairline dividers, hover = `--surface-alt` + `inset 2px 0 0 var(--brand)` |
| Cells | Pill per cell | Plain 13px text; Subject ID 500-weight link; chips ONLY Response + Event |
| Numbers | Left-aligned, "27.00" | Right-aligned, `tabular-nums`, "27" / "27.0" |
| Y/N flags | Colored circles everywhere | "Yes" / muted "—"; color only when clinically notable |
| Dates | "Sun, 01 Dec 2…" truncated | "01 Dec 2024" — no weekday, one format app-wide |
| Overflow | Columns amputated ("First D…", "PD Cl…") | 7–8 columns max + right-edge scroll fade; rest → row-click detail panel |

**The structural fix — row-click detail panel.** Both tables carry 12+ columns
and amputate the rightmost. Move secondary fields (prior PD1/CTLA4 flags, PD
dates, death, first-dose date) into a slide-in side panel (or expandable row)
opened by clicking the row. Table keeps only what a scanning clinician needs.
This is "overview first, detail on demand" applied to tables. Community edition:
use `onRowClick` + own Drawer/panel component (built-in master-detail is Pro;
the DIY version is ~50 lines and looks better anyway).

Implementation notes (per-column, on top of the Part-3 theme overrides):
- `align: "right", headerAlign: "right"` + valueFormatter on all numerics.
- `renderCell` → `StatusPill` for the two chip columns only; everything else
  default text rendering.
- `columnSeparator: { display: "none" }`, `disableColumnMenu` where unused.
- Fixed-height container (virtualization + zero layout shift).
- Search stays; wire `/` to focus it. Export → quiet icon button, "✓ Exported"
  success state.

---

## 3. Keep DataGrid or switch?

**Keep — for now.** Everything in §2 is theme + colDef work: hours, not days.
The primitive look is not the library's fault.

Decision rule for v2: if after the restyle you still want Linear-grade control
(bespoke row-expand animations, fully custom cells, zero fighting component
opinions), the 2026 answer is **TanStack Table** — headless, free, your markup
styled 100% by your tokens, TanStack Virtual for scale. Evaluate only AFTER
theming lands, so you're comparing against a fair baseline.
- **Skip AG Grid**: heavier, enterprise aesthetic, wrong direction.
- **Skip MUI Pro**: pinning/master-detail don't fix anything on this list that
  the DIY versions don't.

## Priority order for these two pages

1. Semantic color fix (green PD badge) — correctness first.
2. Theme overrides land (header de-blue, separators gone, row hover) — shared
   with Part 3 item A1.
3. De-pill: chips only on Response + Event; Y/N → text/em-dash.
4. Column diet + row-click detail panel.
5. Gradient hero + FAB → PageHeader + filter bar; stat strips → StatCards,
   numbers uncolored.
6. Date/number formatting pass (right-align, tabular-nums, "01 Dec 2024").
