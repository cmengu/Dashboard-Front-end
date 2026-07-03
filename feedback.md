# Dashboard Feedback — Screenshot Review vs. Design System

Review of the current build (Patient Status Overview screenshots, 3 Jul 2026)
against `design-principles.md` + `design-system.md`.

**Verdict:** the dashboard doesn't feel "too simple" — simple is the goal. It feels
**unstyled**, which is different. Simple-and-designed reads expensive; the current
build reads like library defaults with the decoration removed. Ordered by impact:

---

## 1. Charts — the loudest problem (violates the color grammar)

The #1 rule — ~90% grayscale, indigo on data, status colors never decorative — is
broken everywhere below the fold:

- **"Active Patients by ARM" pie** uses dark gray / magenta / blue / teal — a random
  default palette. Arms A/B/C aren't statuses; per the system they should be the
  three brand series colors — indigo `#333674`, magenta `#AF3791`, azure `#2D64A5` —
  consistently mapped in `chartTheme.js`.
- **Response charts** use orange for SD and red for PD as *fill*, at full saturation,
  dominating the page. Red/orange should exist only as status accents; here SD (the
  biggest bar) screams alarm-orange. Map CR/PR/SD/PD once in `chartTheme.js`
  (indigo family for CR/PR, gray for SD, muted red reserved for PD) and import it
  everywhere — the "plots as siblings" rule.
- **Donut: "Inactive 63.3%" rendered in dark navy-gray** — the *inactive* category is
  visually heavier than active. Gray-out the uninteresting slice; indigo goes on the
  slice that answers the clinical question.
- Default legends, default tick styling, vertical axis boxes — the §2.6 checklist
  (no chart frame, horizontal-only 1px gridlines, no tick marks, direct labels,
  rounded bar caps, token-styled tooltip) is the single biggest "modern" lever and
  none of it is applied yet.
- **Reconsider the two pies.** The docs already favor bars over pies for comparison:
  "Active by ARM" with 3 slices is clearer as a horizontal bar with direct labels —
  and it kills one legend.

## 2. Stat cards — no structure

- Cards sit gray-on-gray with no visible hairline border and no white surface, so
  there's no figure/ground contrast. The tokens exist exactly for this:
  `--bg: #F9F9FD` page, pure-white `--surface` cards, 1px `--border`. That ~2%
  contrast is what makes cards float without shadows.
- Cards are **different widths and misaligned** ("11 Currently Active" is nearly
  twice as wide as its neighbors; rate cards sit on a staggered second row). Use the
  2→3→5 grid from `DashboardPage.jsx` so the five cards read as one deliberate strip.
- Huge dead space *inside* each card — number, label, and icon are far apart.
  Tighten to the 4px-scale proximity rule: label 4–6px from value, icon chip in a
  `--surface-alt` rounded square. Dense within, generous between.
- DCR/ORR bars are full-width dark lines with **no track**. Give them the rounded
  track from `StatCard.jsx` (`--surface-alt` background, indigo fill) — a bar with
  no track looks like a stray divider.

## 3. Trust / clinical details missing

- **No "Data as of {date}" stamp** — the cheapest trust win, absent.
- Denominators inconsistent: DCR shows `n=29/30`, but "11 Currently Active" and
  "6 Ever Had PR+" have no "of 30". Add hints everywhere ("of 30 enrolled").
- **Filter bar:** three dropdowns + a Reset button, but no active-filter chips,
  no "Showing X of 30 patients" count, and Reset is visible even when nothing is
  filtered. §2.8 covers all three.

## 4. Typography & hierarchy

- Big numbers are fine, but everything else is one shade of mid-gray at similar
  sizes — section titles, subtitles, and axis labels blur together. Apply the
  weight+gray hierarchy: titles 600 in `--text-strong`, subtitles 13px
  `--text-muted`, axis labels 12px `--text-subtle`.
- Center-aligned chart titles read dated — left-align them with the card padding.

---

## One-sentence answer

Don't add more stuff to fix "too simple" — apply the system already written:
white cards with hairline borders on the tinted background, the three-brand-color
chart map in `chartTheme.js` replacing the default palette, denominators +
freshness stamp, and the §2.6 chart de-defaulting. That's build-order steps 3–5
in `design-principles.md`, and it's the difference between "empty" and "restrained."

---
---

# Part 2 — Safety Summary & AE Parsed Table pages

Different failure mode from the Overview: less "unstyled defaults," more
**untamed layout + MUI default blue leaking everywhere**. Ordered by impact:

## 1. Page header is a landing-page hero on an internal tool

Giant centered magenta "Safety Summary" + centered gray subtitle. Three violations:

- **Magenta is the one-hero-moment color** — spending it on every page title (and
  again on "TEAE"/"TRAE" table titles) makes it decoration. Four magenta headings
  on one page.
- Centered display type belongs on marketing pages. Internal tools: **left-aligned
  ~20px/600 title in `--text-strong`** inside `PageHeader.jsx`, freshness stamp on
  the right — currently "Updated: 2026-07-02" is buried in a side panel, twice,
  in two different formats.
- Subtitle "Clinical Safety Overview & Analysis" adds nothing. Cut it.

## 2. Default MUI blue has taken over the accent role

"Safety Summary Controls & Filters", "Column Filter", "Min % threshold", the
`COLUMNS FILTERS DENSITY EXPORT` toolbars, `EXPORT TEAE CSV`, and the **solid
saturated-blue DataGrid header row** — all default MUI `#1976d2`, all UPPERCASE.
Clearest sign `theme.js` isn't wired (or these pages sit outside the
ThemeProvider). One fix cascades: apply the flattened theme — indigo primary,
`textTransform: none`, DataGrid header `--surface-alt` with `--text-muted`
labels instead of a blue banner.

## 3. The filter panel wastes half the viewport

The four-column controls block is ~600px tall: five dropdowns stack vertically in
column one while columns two/three hold one control each above empty space. §2.8
is the fix:

- **One horizontal filter bar**, all controls in a row, sticky on scroll.
- Active filters as **chips with ×** + "Showing 54 of 54 records" (the AE table
  already has this line — promote it into the bar).
- "Data Statistics & Export" isn't a filter — Export moves to the table toolbar
  top-right, record counts into the results line. Deletes the fourth column.
- Gray-filled dropdowns read as **disabled** (gray fill + gray text = the disabled
  treatment). Set filters ("A, B, C", "1800, 3000") should look active: white
  field, `--border`, dark text.

## 4. The TRAE table is an EmptyState pretending to be data

A full grid of `0 (0.0%)` with its own toolbar and search box. When TRAE count is
zero, render the designed `EmptyState`: "No treatment-related adverse events
recorded · 0 of 30 subjects." A table of zeros makes the reader scan every cell
to learn "nothing."

## 5. Table craft (TEAE + summary table + AE Parsed)

- **Truncation everywhere**: "Patients with ≥ 1...", "30 (100....", "Gra..."
  headers. If side-by-side TEAE/TRAE forces columns this narrow, stack the tables
  or drop redundant columns — a clinical table you can't read has failed its only job.
- **"Subjects with at least 1" summary table** stretches label and value ~800px
  apart across full page width. Proximity rule: cap the card ~560px, right-align
  values next to labels, `tabular-nums`.
- **n (%) columns right-aligned** in every table — currently left-aligned, digits
  don't line up.
- **SOC column** wraps to 4 lines → row heights lurch 40px→90px. Truncate with
  tooltip, or abbreviate.
- **"Ongoing: No" pill on every row** = 54 repetitions of nothing. Pill only for
  "Yes" (the notable state), em-dash otherwise.
- **Inconsistent notation/dates**: "Grade >= 3" vs "Grade ≥ 3"; `02/07/2026` vs
  `2026-07-02 23:58:11` vs `2024-04-01`. One date format (`02 Jul 2026`), one
  glyph (≥). In a clinical UI ambiguous DD/MM vs MM/DD is a genuine safety issue.
- Default gray horizontal scrollbars cutting through both tables — §2.5.2 fix.

## Priority order for these pages

1. Wire `theme.js` so the default blue disappears globally.
2. Collapse the filter panel into the §2.8 horizontal bar.
3. `PageHeader` with left-aligned title + ONE freshness stamp.
4. `EmptyState` for the zero-row TRAE table.
5. Table alignment / truncation / date-format pass.

**The pattern across both reviews:** the Overview needed its chart skin; these
pages need the theme actually applied plus layout discipline.

---
---

# Part 3 — Finalized action list (modern · clean · micro-interactions)

Constraint: MUI X **Community** edition (no column pinning, no row grouping —
flagged below where it matters).

## Filter architecture decision: two-tier, expand BELOW the bar (not a modal)

Filtering is a **feedback loop, not a form** — nudge filter → watch numbers →
nudge again. A modal covers the data being filtered (instant-apply becomes
pointless behind an overlay), demands open/close per tweak, and occludes the
chips + result count exactly when scope is being changed. Popups are right only
for single rich controls (date-range calendar) or true query builders with
batch-apply. Four dropdowns + a threshold row = inline panel wins.

- Tier 1 (always visible, 56px): Arm · Relatedness · Cut-off date ·
  "⚙ More filters (n)" with count badge in brand-tint · "Showing X of Y" · Reset
  (only when non-default).
- Tier 2: expands below, pushes content (user-initiated = sanctioned layout
  animation). Smooth expand = `grid-template-rows: 0fr → 1fr` transition,
  `--dur-base` `--ease-out`; contents fade+4px rise with ~50ms delay; chevron
  rotates 180° same duration; persist open state in localStorage.
- Active filters from EITHER tier render as chips with × under the bar.
- Instant apply, no Apply button (dataset is small; <300ms rule).

## A. Modern & sleek (in build order)

1. **Wire `theme.js` + ThemeProvider on every page** — 80% of "modern" in one
   commit: kills default blue, Roboto→Inter, UPPERCASE, elevation, blue DataGrid
   header. Nothing else pays off until this lands.
2. **One depth model**: hairlines not shadows. Page #F9F9FD, white cards, 1px
   #E5E6F0. Shadow only on floating things (menus/toasts).
3. **One accent**: indigo on actions/nav/focus/series-1. Magenta ≤ once per
   screen (currently 4× as headings). Everything else grayscale.
4. **Type = weight + gray, not size**: page title 20/600 left; card titles
   14/600; labels 12 muted; stat numbers 28/600, -0.02em, tabular-nums. No
   centered display type on internal tools.
5. **Motion: 3 speeds, 1 ease, transform/opacity only** (120/200/400ms,
   cubic-bezier(.16,1,.3,1)). Nothing loops/bounces/replays on refetch.
6. **De-default charts** (§2.6): no plot frame, horizontal-only gridlines, no
   tick marks, direct labels, rounded bar caps, token tooltip.

## B. Clean pages (ruthless deletion + alignment)

1. Hero headers → `PageHeader` (title left, "Data as of 02 Jul 2026" right).
   Delete subtitle lines.
2. Filter block → two-tier bar. Deletes ~500px dead space per page — biggest win.
3. **Delete every duplicate**: one Updated stamp (header), one record-count line
   (bar), one Export (table toolbar).
4. **One date format: `02 Jul 2026`** everywhere. Month-as-word kills DD/MM
   ambiguity — correctness, not style. One glyph: ≥.
5. TRAE zero-grid → `EmptyState` ("No treatment-related adverse events · 0 of
   30 subjects"), delete its toolbar+search.
6. Summary table: max-width ~560px, values right-aligned, tabular-nums, 40px rows.
7. **DataGrid discipline**: custom slim toolbar (search left, icon Export right,
   never default GridToolbar); numeric columns right-aligned; SOC one line +
   tooltip; Ongoing pill only for "Yes", em-dash otherwise;
   `columnSeparator: none`; compact density; fixed-height container.
   **Community constraints**: no pinning → cut redundant columns instead
   (Grade ≥ 3 derivable; MedDRA PT ≈ AE Term — show one, tooltip other); no row
   grouping → pre-group in data if ever needed.
8. Alignment audit: 20px card padding, 24px gutters, top-aligned card rows.

## C. Micro-interactions that work (by yield-per-effort)

1. **Elevate-on-scroll** sticky bar/header: flush at top; gains bottom hairline +
   faint shadow once scrolled (1px sentinel + IntersectionObserver, ~10 lines).
2. **Scroll-edge fades** on horizontally scrollable tables: 24px right-edge
   gradient that disappears at scroll end. Pure CSS.
3. **Row hover**: `--surface-alt` fill + `inset 2px 0 0 var(--brand)` left bar —
   zero layout shift, quietly premium.
4. **Custom scrollbars** (thin, `--border-strong` thumb) + `scrollbar-gutter:
   stable`.
5. **Stat number count-up ONCE on first load** (~600ms) — never on refetch/tab
   return.
6. **Chip enter/exit**: scale .95→1 + fade, 120ms.
7. **Export success state**: "✓ Exported" for 1.5s.
8. **Click-to-copy Subject IDs** with momentary ✓.
9. **Keyboard**: focus-visible rings everywhere; `/` focuses search; `Esc`
   closes advanced panel.
10. **Chart draw-in once on mount** (400ms) + bar hover darken. Never re-animate
    on filter change.

**Refuse** (consumer-toy in a clinical tool): parallax, tilt cards,
glassmorphism on data, pulsing "live" dots on batch data, springy overshoot,
count-up on refetch, slow skeleton shimmer.

---

## Open questions (answers may reshuffle priorities)

1. **Which filters do users actually touch?** Tier-1 pick (Arm · Relatedness ·
   Cut-off) is an educated guess; if cohort is filtered constantly, it moves up.
2. **Row counts at scale** — 54 AEs now; if this grows to thousands, revisit
   instant-apply (may need debounce/batch) and pagination vs. virtualized scroll.

## Sources (filter pattern + DataGrid styling)

- Pencil & Paper — Enterprise Filtering UX:
  https://www.pencilandpaper.io/articles/ux-pattern-analysis-enterprise-filtering
- LogRocket — Filtering UX patterns:
  https://blog.logrocket.com/ux-design/filtering-ux-ui-design-patterns-best-practices/
- Eleken — Filter UI for SaaS: https://www.eleken.co/blog-posts/filter-ux-and-ui-for-saas
- UXPin — Filter UI & UX: https://www.uxpin.com/studio/blog/filter-ui-and-ux/
- MUI X DataGrid Styling: https://mui.com/x/react-data-grid/style/
- MUI X Styling Recipes: https://mui.com/x/react-data-grid/style-recipes/
