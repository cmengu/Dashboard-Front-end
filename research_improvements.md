# Implementation Spec: Contrast + Tables + Loading Fixes (round 2)

**Audience of this document:** an AI coding agent. Follow instructions EXACTLY.
Every color is given as an exact hex value. Every change names the file it goes in.
Do not invent new colors, new animations, or new components beyond what is written here.
Where this document contradicts `design-system.md` or `feedback.md`, THIS DOCUMENT WINS.

**Background (context only, no action needed):** the client rejected the previous
design because it was too low-contrast (near-white background, near-invisible
borders) and the tables were plain. This spec raises contrast and adds
data-visuals inside table cells. It also removes the animated number count-up.

The project stack: Vite + React (JSX) + Tailwind v3 + MUI (DataGrid). Design tokens
live in `src/styles/tokens.css`. Tailwind maps tokens in `tailwind.config.js`. MUI
theme is `src/theme.js`.

---

## PART 1 — Color scheme changes

### 1.1 Replace token values in `src/styles/tokens.css`

Change ONLY the values listed below. Keep every other token exactly as it is.

| Token | OLD value | NEW value | Why |
|---|---|---|---|
| `--bg` | `#F9F9FD` | `#E9EBF3` | page background must visibly differ from white cards |
| `--surface-alt` | `#F1F2F8` | `#EEF0F6` | hover fills / icon chips, slightly deeper |
| `--border` | `#E5E6F0` | `#D5D7E5` | borders must be visible at arm's length |
| `--border-strong` | `#D2D4E2` | `#B9BCD0` | hover border state |
| `--text-strong` | `#1C1E33` | `#111327` | headings near-black |
| `--text` | `#34374D` | `#272A40` | body text darker |
| `--text-muted` | `#696C84` | `#565A74` | labels darker |
| `--text-subtle` | `#9B9DB2` | `#787B93` | old value failed WCAG AA contrast |

Then ADD these new tokens inside the same `:root` block (they do not exist yet):

```css
  /* ── Dark sidebar palette (Part 1.2) ─────────────────────── */
  --sidebar-bg:        #1B1D3E;   /* darkened brand indigo; NEVER pure #000 */
  --sidebar-text:      rgba(255, 255, 255, 0.72);
  --sidebar-text-hover:#FFFFFF;
  --sidebar-active-bg: rgba(255, 255, 255, 0.10);
  --sidebar-active-fg: #FFFFFF;
  --sidebar-border:    rgba(255, 255, 255, 0.08);

  /* ── Table header stripe (Part 1.3) ──────────────────────── */
  --table-header-bg: #333674;     /* same as --brand */
  --table-header-fg: #FFFFFF;

  /* ── In-cell data visuals (Part 2) ───────────────────────── */
  --cellbar-fill:  #5B5FA8;       /* lighter indigo so dark text stays readable on top */
  --cellbar-track: #EEF0F6;
  --heat-1: #EDEEF7;  /* heatmap ramp, low → high. Single indigo hue. */
  --heat-2: #D9DBEF;
  --heat-3: #BFC2E2;
  --heat-4: #A3A7D4;  /* highest tint that still passes AA with #111327 text */
```

### 1.2 Sidebar becomes dark (file: `src/components/layout/Sidebar.jsx`)

Rebuild the sidebar's colors using ONLY the `--sidebar-*` tokens above:

1. Sidebar container background: `var(--sidebar-bg)`. Remove any light background
   or border it currently has; its right edge needs no border (the dark/light
   split IS the edge).
2. Nav item default state: text color `var(--sidebar-text)`, transparent background.
3. Nav item hover: text `var(--sidebar-text-hover)`, background `var(--sidebar-active-bg)`.
4. Nav item ACTIVE: text `var(--sidebar-active-fg)`, background `var(--sidebar-active-bg)`,
   plus a 3px solid white vertical bar on the item's left edge
   (`box-shadow: inset 3px 0 0 #FFFFFF;` — do NOT use a border, it shifts layout).
5. Logo / product name at top: white text.
6. Any divider inside the sidebar: `var(--sidebar-border)`.
7. Icons inside nav items inherit the text color (`currentColor`).

RULES: nothing else in the app becomes dark. Content area, cards, tables, filter
bar all stay light. Do not add a dark-mode toggle.

### 1.3 Indigo table header stripe (file: `src/theme.js`)

In the `MuiDataGrid` styleOverrides, replace the current `columnHeaders` block with:

```js
columnHeaders: {
  backgroundColor: "var(--table-header-bg)",
  color: "var(--table-header-fg)",
},
columnHeaderTitle: { fontWeight: 600 },
sortIcon: { color: "var(--table-header-fg)" },
menuIconButton: { color: "var(--table-header-fg)" },
```

RULES:
- This applies to ALL DataGrids automatically via the theme. Do not style headers
  per-table.
- Header text stays `textTransform: none` (no UPPERCASE).
- Everything below the header row stays light: white rows, `var(--border)` row
  separators.

### 1.4 Hero metric (file: `src/pages/DashboardPage.jsx` + `src/components/ui/StatCard.jsx`)

Exactly ONE stat card is visually promoted:

1. In `CARD_DEFS`, add `hero: true` to the `orr` entry ONLY.
2. In `StatCard.jsx`, when `hero` is true, render the value at `text-[44px]`
   instead of `text-[28px]` (keep `font-semibold`, `tracking-[-0.02em]`, `tabular`).
3. Keep the existing hero behaviors (magenta bar + tinted chip) unchanged.
4. In the grid, the hero card spans 2 columns at the `xl` breakpoint:
   give it `xl:col-span-2` and change the grid to `xl:grid-cols-6` so the row
   still fills (4 normal cards × 1 col + 1 hero × 2 cols = 6).

RULES: never more than one `hero` card. Do not enlarge any other number.

### 1.5 What is FORBIDDEN in Part 1

- Pure black `#000000` anywhere.
- Dark backgrounds anywhere except the sidebar.
- New accent colors. The palette is: indigo `#333674`, magenta `#AF3791`, azure
  `#2D64A5`, the status green/amber/red already in tokens, grayscale. Nothing else.
- Shadows on cards (shadow stays only on menus/modals/toasts).
- Magenta on more than ONE element per screen.

---

## PART 2 — Table upgrades

Build order below = priority order. If time-boxed, stop after 2.2 and the client
still sees a transformed table.

### 2.1 In-cell bars for every `n (%)` column

Create a new file `src/components/ui/CellBar.jsx`:

```jsx
/**
 * CellBar — renders "n (x%)" with a proportional bar underneath.
 * Used inside DataGrid cells via renderCell. Dumb component, props only.
 *
 * props:
 *  n    number   count, e.g. 12
 *  pct  number   0–100, e.g. 40.0
 */
export function CellBar({ n, pct }) {
  const clamped = Math.min(100, Math.max(0, pct ?? 0));
  return (
    <div className="flex w-full flex-col justify-center gap-1 py-1">
      <span className="tabular text-right text-[13px] leading-none text-body">
        {n} ({clamped.toFixed(1)}%)
      </span>
      <div className="h-[4px] w-full rounded-full"
           style={{ background: "var(--cellbar-track)" }}>
        <div
          className="h-full rounded-full"
          style={{ width: `${clamped}%`, background: "var(--cellbar-fill)" }}
        />
      </div>
    </div>
  );
}
```

Wire it into every DataGrid column whose data is a count-with-percentage
(TEAE table, TRAE table, summary tables). Column definition pattern:

```jsx
{
  field: "anyTeae",
  headerName: "Any TEAE",
  flex: 1,
  align: "right",
  headerAlign: "right",
  renderCell: (params) => (
    <CellBar n={params.row.anyTeae.n} pct={params.row.anyTeae.pct} />
  ),
  sortComparator: (a, b) => a.pct - b.pct,   // sort by percentage, not object
}
```

RULES:
- The bar has NO animation. It renders at final width immediately.
- Bar color is always `var(--cellbar-fill)`. Never green/amber/red bars — color
  in this app means status, and incidence is not a status.
- If `pct` is missing/undefined, render just the number, no bar.
- Zero values render `0 (0.0%)` with an empty track (no fill), not a blank cell.

### 2.2 Grouped rows: SOC group headers with PT rows beneath

The AE tables currently show System Organ Class (SOC) as a wrapping text column.
Replace with full-width group header rows. MUI DataGrid Community edition has NO
row grouping feature — implement it in the data layer:

1. Pre-process rows before passing to DataGrid: sort by SOC, then insert a
   synthetic row before each SOC's first PT row:
   `{ id: "soc-<socName>", isGroupHeader: true, socName, socN, socPct }`.
2. Add `getRowClassName={(p) => p.row.isGroupHeader ? "soc-group-row" : ""}`.
3. Use `colSpan` via `renderCell` on the FIRST column only: when
   `isGroupHeader`, render `<span>{socName} — {socN} ({socPct}%)</span>` styled
   `font-semibold text-strong`; all OTHER columns return `null` for group rows
   (define each column's `renderCell` to check `params.row.isGroupHeader` first).
4. CSS for the group row (global stylesheet):

```css
.soc-group-row {
  background: var(--brand-tint);
  pointer-events: none;           /* not selectable, not hoverable */
}
```

5. Remove the SOC column entirely from the columns array (the group row now
   carries it). This kills the 4-line row-height wrapping problem.
6. Sorting: when the user sorts by any column, sort PT rows WITHIN each SOC
   group; group header rows keep their position. If that is too complex, disable
   column sorting on grouped tables (`sortable: false` on all columns) — a stable
   grouped read is worth more than sort.

### 2.3 Heatmap tint on grade/severity columns

For columns representing Grade 3 / Grade ≥3 / Grade 4–5 counts, tint the CELL
background by value. Exact mapping (percentage of subjects):

| pct value | background token |
|---|---|
| 0 | none (white) |
| >0 and <5 | `--heat-1` |
| ≥5 and <10 | `--heat-2` |
| ≥10 and <20 | `--heat-3` |
| ≥20 | `--heat-4` |

Implement with `cellClassName: (params) => heatClass(params.row.grade3.pct)` and
four CSS classes `.heat-1 { background: var(--heat-1); }` etc. Text in heat cells
stays `var(--text-strong)` and right-aligned.

RULES: heat tint is the indigo ramp above — NOT red/orange. Only grade/severity
columns get heat. Never combine CellBar and heat tint in the same cell.

### 2.4 Density toggle

Add a three-option segmented control (labels exactly: `Compact`, `Comfortable`,
`Spacious`) in each table's toolbar, top-right, left of Export:

- Maps to DataGrid `density` prop values `"compact" | "standard" | "comfortable"`.
- Default: `Comfortable` (= MUI `"standard"`).
- Persist choice in `localStorage` key `hb-table-density`; read it on mount; one
  shared value for all tables (not per-table).

### 2.5 Sticky first column

On horizontally scrollable tables, the first column (Subject ID or PT) must stay
visible during scroll. Community edition has no column pinning. Use CSS:

```css
.MuiDataGrid-cell[data-field="subjectId"],
.MuiDataGrid-columnHeader[data-field="subjectId"] {
  position: sticky;
  left: 0;
  z-index: 2;
  background: var(--surface);
  box-shadow: 2px 0 4px rgba(28, 30, 51, 0.06); /* edge reads as a layer */
}
```

Replace `subjectId` with the actual first field name per table. The sticky header
cell keeps the indigo header background (`var(--table-header-bg)`), not white.
If a table does not scroll horizontally, skip this — no shadow on non-scrolling
tables.

### 2.6 Keep from the previous spec (still valid, unchanged)

- Row hover: `--surface-alt` fill + `inset 2px 0 0 var(--brand)` left bar.
- Numeric columns right-aligned, `tabular-nums`.
- "Ongoing" pill only when value is Yes; em-dash `—` otherwise.
- One date format everywhere: `02 Jul 2026`. One glyph: `≥`.
- Zero-row TRAE table renders `EmptyState`, never a grid of zeros.
- `columnSeparator: none`, fixed-height grid container (never `autoHeight`).

### 2.7 FORBIDDEN in Part 2

- Zebra striping (alternating row colors) — row separators + group rows carry
  the structure.
- Sparklines: SKIP for now (no per-row time-series data in the current API).
  Do not add chart libraries to cells.
- Icons or emoji inside data cells.
- Any cell animation (bars/heat render instantly).
- Red or orange as bar fills or heat ramps.

---

## PART 3 — Remove number loading animations

Client request. Remove ALL animated counting/growing on load:

1. **StatCard bar animation** (`src/components/ui/StatCard.jsx`): delete the
   `barWidth` state, the `useEffect`, and the `requestAnimationFrame` import
   usage. The bar div gets `style={{ width: `${clamped}%` }}` directly and NO
   `transition-[width]` class. It renders at final width on first paint.
2. **Any number count-up** (0→128 ticking animation) anywhere in the codebase:
   delete it. Numbers render at final value immediately. Search for
   `requestAnimationFrame`, `CountUp`, `animate`, `interval` near stat values.
3. **Chart draw-in animations** (bars growing on mount): also remove. For
   Recharts set `isAnimationActive={false}`; for MUI charts set
   `skipAnimation`. Charts render complete on first paint.
4. **KEEP** (these are not number loading): skeleton→content swap, hover
   transitions, filter panel expand/collapse, chip enter/exit.

---

## Acceptance checklist (verify each before calling it done)

- [ ] Page background is visibly gray next to white cards from 1 m away.
- [ ] Sidebar is dark indigo `#1B1D3E`; content area fully light; no other dark surfaces.
- [ ] Every DataGrid header row is solid indigo with white 600-weight text, no UPPERCASE.
- [ ] Exactly one hero stat card (ORR) at 44px; all others 28px.
- [ ] All `n (%)` cells show number + static proportional indigo bar; zeros show empty track.
- [ ] AE tables show indigo-tinted SOC group rows; SOC column deleted; no 4-line wrapped rows.
- [ ] Grade ≥3 columns show the 4-step indigo heat ramp; no red/orange fills.
- [ ] Density control present, persists across reload via `hb-table-density`.
- [ ] No number counts up, no bar grows, no chart draws in — everything at final state on first paint.
- [ ] Text contrast: run every text tier through a WCAG checker; all ≥ 4.5:1 on their background.
- [ ] Magenta appears at most once per screen.

---

## Sources (for the human, not the agent)

- https://www.925studios.co/blog/saas-dashboard-design-examples-2026 — dark-sidebar fintech trust pattern (Mercury/Ramp/Brex), single-accent discipline, size-as-hierarchy
- https://www.setproduct.com/blog/data-table-ui-design — density modes, frozen-edge shadow, priority+ columns
- https://data.europa.eu/apps/data-visualisation-guide/table-design-integrating-visualisations — in-cell bars, heatmap cells
- https://blog.datawrapper.de/new-table-tool-barcharts-fixed-rows-responsive-2/ — bars/sparklines in real tables
- https://www.aydesign.ai/blog/dark-mode-dashboard-design-patterns-2026 — never pure #000
- https://www.pencilandpaper.io/articles/ux-pattern-analysis-enterprise-data-tables — enterprise table UX
