# Hummingbird Dashboard — Design Principles (Clinical × Modern)

Companion to `design-system.md` (tokens + theme + StatCard). That file is *what to build*;
this file is *why, what to skip, and how to make it feel smooth and snappy*.

---

## Part 1 — Verdict on the research: keep / throw / append

### KEEP (earns its place in this project)

| Principle | Why it stays |
|---|---|
| **Strict color grammar** — ~90% grayscale; indigo on data/actions; magenta once; green/amber/red = status only, never decoration | Doubly justified: reads expensive (design) AND color is a safety channel in clinical UIs (a decorative red can be misread as an alert) |
| **5–9 core elements per view, progressive disclosure** | Your 5 stat cards already sit at the bottom of this range. Detail on demand (hover, drill-in), not everything at once |
| **Overview-first, drill down** | Cards → plots → All Records table is literally the evidence-backed pattern. Every future tab repeats this shape: summary strip on top, dense detail below |
| **Three-second rule** (Fuselab) | The acceptance test for the Overview tab: can a clinician glance at it and answer "is the trial working, and who's off-track?" within three seconds? If a headline answer needs scrolling, reading a legend, or hovering, the hierarchy failed. Rates up top, biggest type on the answer, everything else quieter |
| **Consolidation — answer on ONE screen** (Ron Design Lab EHR+ lesson) | Doctors in that study found it "faster to print records than interpret them digitally" because data was scattered across screens. The dashboard's whole value is cards + plots + table on one page; drill-down = state on the same page (hover, select, filter), never navigation away. A patient clicked in the swimmer plot should highlight in the table, not open a new route |
| **Density is audience-dependent** | Overview = minimal; plots + DataGrid = allowed to be dense. Both pull from the same tokens; don't "minimalize" the analyst views |
| **Denominators everywhere** | "DCR 62% · n=34/55". A rate without its denominator is the #1 credibility miss in clinical dashboards |
| **Data-freshness stamp** | "Data as of {date}" near the header. Near-zero cost, disproportionate trust gain |
| **Never color-alone encoding** | ~8% of men are red-green colorblind — the exact pair clinical status uses. Pill + label, ▲/▼ + number, marker shape + color |
| **Plots as siblings** | Waterfall / spider / swimmer share: card wrapper, gridlines in `--border`, one legend component, identical color-per-response-category. Consistency beats any single-chart improvement |
| **RECIST reference lines** | −30% / +20% dashed hairlines in `--border-strong` on waterfall + spider |
| **Gray-crowd spider plot** | All lines light gray; highlight the notable (or hovered) patient in indigo. The 90%-grayscale rule applied to data — the single move that separates it from a default Plotly chart |
| **Information order = clinical question order** | "Is it working, for whom, who's still on it?" → rates → waterfall (depth) → swimmer (duration) |

### THROW AWAY (in the research, not right for this project)

| Trend / advice | Why it's out |
|---|---|
| **AI-agent / conversational dashboard trend** | Enterprise trend-piece filler. An internal clinical dashboard needs legible data, not a chatbot |
| **Per-user dashboard customization** | Valid for hospital-wide platforms with 10 user roles; for a focused trial dashboard it's YAGNI — it adds settings surface and fragments the one carefully designed layout |
| **3D waterfall plots** | The oncology literature itself warns: z-axis is hard to read, legend burden up, insight marginal. Flat + sorted wins |
| **Real-time indicators / live-updating widgets** | Only honest if the data is actually live. Trial data is batch-loaded; a fake "live" pulse erodes the trust the freshness stamp builds |
| **Mobile-first layout** | Clinicians review response data on desktop/laptop. Make it *responsive* (below), but don't design for a phone first and strip density |
| **Glassmorphism / frosted-glass widgets** | THE 2026 inspiration-feed aesthetic (dark glassmorphism, frosted biometric widgets over gradient backgrounds) — and wrong for this project. Blur + translucency reduce legibility, which is the one thing clinical data can't trade. Verdict: never on surfaces holding data; at most a frosted modal backdrop (`backdrop-filter` on the overlay, solid white modal on top) if we want a taste of the trend on non-data chrome |
| **Dark mode (for now)** | A *sequencing* call, not a permanent one. Dark mode = a second value for every token (dark surface ramp, re-derived tints, re-checked contrast, second MUI palette, re-tested charts) — doubling the QA surface of a system that hasn't shipped v1. Because everything reads from tokens, adding it later is one `.dark { --bg: …; }` block + a MUI palette swap, not a rewrite. Ship light; add dark when a user actually asks |

### APPEND (missing from the clinical research — the "modern feel" layer)

The clinical literature is silent on *feel*. Everything below is the append.

---

## Part 2 — The modern feel: smooth, snappy, sleek

"Snappy" is engineerable. It decomposes into four things: **latency, motion, interaction
states, and layout stability.** Each has concrete numbers and tokens.

### 2.1 Latency budgets (what "snappy" actually is)

Perceived-speed thresholds (Nielsen's numbers, still the standard):

- **< 100 ms** → feels instant. Everything local must land here: tab switch, hover, filter toggle, sort.
- **100–300 ms** → perceptible lag. Acceptable only for server round-trips.
- **> 300 ms** → needs feedback (skeleton, progress), or it feels broken.

Rules that follow:

1. **Never block the UI on data.** Render the shell + skeletons immediately; let numbers arrive. (StatCard already has the skeleton state.)
2. **Optimistic UI for local interactions.** Filter click updates the UI instantly; fetch reconciles after. Don't spinner a checkbox.
3. **Stale-while-revalidate.** Show last-known data with the freshness stamp while refetching (TanStack Query default behavior — use it). An instant "slightly old + timestamped" beats a spinner every time, *especially* in clinical context where the stamp makes staleness honest.
4. **Tab switches are instant.** Keep tab panels mounted (hide, don't unmount) or cache their queries so returning to a tab never re-loads.
5. **Virtualize the big table.** MUI DataGrid virtualizes rows by default — don't defeat it with `autoHeight` on long lists. (Why: virtualization needs a fixed-height viewport to know which rows to render; `autoHeight` renders ALL rows as real DOM. Code fix in `design-system.md` §3.)

> The code for §2.1's data-state pattern (the `useStats` hook, `ErrorState`,
> `EmptyState`, and the page-level branch) now lives in `design-system.md` §5.

### 2.2 Motion system (smooth ≠ more animation)

The counterintuitive finding from every polished product: **sleek = fewer, faster,
more purposeful animations.** Slow/springy animation reads consumer-toy; snappy
= short durations + decelerating easing.

Motion tokens (`--dur-fast/base/slow`, `--ease-out`, `--ease-in-out`) are now IN
`tokens.css` — see `design-system.md` §1 — and mapped to Tailwind utilities
(`duration-fast/base/slow`, `ease-out-token`) in §2. How they work: a CSS variable
holds any value including a time, so `transition: width var(--dur-base) var(--ease-out)`
works anywhere, and the Tailwind mapping gives them utility names. The payoff:
**the app has exactly three speeds** — all motion feels designed by one hand, and
retuning the whole app = editing one line.

The motion budget — where animation is ALLOWED:

| Moment | Treatment |
|---|---|
| Hover states | `--dur-fast`, color/border only (cheap to render) |
| Clickable-card lift | `translateY(-2px)` + `--shadow-pop`, `--dur-base` — affordance, so clickable elements only |
| Tab/panel switch | Fade + 4–8px slide, `--dur-base` `--ease-out`. Never a full slide-across |
| Rate-bar / chart draw | Once, on first mount, `--dur-slow`. Never re-animate on tab return or refetch |
| Sidebar collapse | Width transition `--dur-base`; icons stay fixed, labels fade |
| Dropdowns/menus | Scale from 0.97 + fade, `--dur-fast` |

Where animation is BANNED: layout reflow (nothing pushes other content while
animating — transform/opacity only, they're GPU-composited and stay 60fps),
looping/pulsing attention-grabbers (alarm-fatigue in a clinical UI), number
count-ups on every refetch (once on load at most), skeleton shimmer slower
than 1.5s, and anything that replays when data merely revalidates.

Implementation: **CSS transitions for 90% of this.** Reach for Framer Motion only
for enter/exit of mounted/unmounted elements (modals, toasts). Every animation
respects the `prefers-reduced-motion` kill-switch already in `tokens.css`.

### 2.3 Interaction states (where "sleek" actually lives)

Users judge polish by the states designers forget. Every interactive element
gets all five — this is the checklist:

| State | Treatment (from tokens) |
|---|---|
| **Hover** | Fill: `--surface-alt`; or border: `--border-strong`. Subtle — visible, not loud |
| **Active/pressed** | `scale(0.98)` or a shade darker, `--dur-fast` — the "click felt" signal |
| **Focus-visible** | 2px ring `--brand` at 40%, offset 2px. Keyboard-only (`:focus-visible`), on EVERYTHING interactive |
| **Disabled** | `--text-subtle` + `cursor-not-allowed`; never just lower opacity on white (fails contrast) |
| **Selected** | `--brand-tint` fill + `--brand` text/border (nav items, table rows, filter chips) |

Plus the three data states every view needs designed, not defaulted:

- **Loading** → skeletons matching final layout exactly (no shift).
- **Empty** → a designed empty state: one muted icon, one sentence, one action. Never a blank white card.
- **Error** → inline, human, with a retry — not a toast that vanishes.

### 2.4 Layout stability (the invisible half of "smooth")

Jank = things moving that shouldn't. The rules:

1. **Reserve space for everything async.** Skeletons at exact final dimensions; fixed heights for chart containers (`aspect-ratio` or explicit height) so plots don't reflow the page when they render.
2. **`tabular-nums` everywhere numbers change** (already in tokens) — refetches don't wiggle the layout.
3. **Scrollbar-safe**: `scrollbar-gutter: stable` on the main scroll container so content doesn't jump 15px when a scrollbar appears.
4. **Images/illustrations get explicit width/height** (or aspect-ratio) — zero CLS.
5. **Responsive without breakpoint whiplash**: the stat grid's 2→3→5 column skip-4 pattern; sidebar collapses to icon-rail at `lg`, drawer below `md`; plots shrink gracefully (min-width + horizontal scroll inside their card, never squashed axes).

### 2.5 Small sleek details (cheap, high-yield, in priority order)

1. **`scroll-behavior: smooth` + sticky table header** in All Records — the header staying put while rows scroll reads instantly professional.
2. **Custom scrollbar** (thin, `--border-strong` thumb, transparent track) inside cards/tables. Default chrome scrollbars are the loudest "unstyled" tell on Windows.
3. **Toasts bottom-right, 4s, one at a time**, `--shadow-pop` — the only floating shadow in the app besides menus.
4. **Chart tooltips restyled to the token system** (white card, `--border` hairline, `--shadow-pop`, `tabular-nums`). Default Plotly/Recharts tooltips break the design language harder than anything else on the page.
5. **Keyboard affordances**: `/` focuses search; `Esc` closes anything floating.
6. **Transition tokens on the sidebar active-pill** so the `--brand-tint` highlight slides between nav items (`--dur-base`, `--ease-out`) instead of teleporting — one detail, reads extremely modern, costs ~10 lines.

### 2.6 Chart skin — what makes a plot read "modern" (goes in `chartTheme.js`)

Default Plotly/Recharts styling is the loudest "unstyled" tell after scrollbars.
The modern chart look decomposes into five removals and two additions:

**Remove** (the defaults that read dated):
1. Chart border / axis box — no frame around the plot area; the card IS the frame.
2. Heavy gridlines — horizontal only, 1px `--border`, no vertical gridlines at all.
3. Axis tick marks — labels alone, in `--text-subtle` at 12px, are enough.
4. Legend boxes with borders — plain inline swatch + label, or better, direct labeling on the data.
5. Default tooltip — replace with the token-styled `ChartTooltip` (§2.5.4).

**Add:**
1. **Rounded bar caps** on the waterfall + swimmer bars (2–3px radius on the outer end only) — the single cheapest "2026" signal on a bar chart.
2. **Gradient fill under line series** (spider plot patient lines when highlighted): the series color at ~15% opacity fading to transparent at the baseline. Only on the ONE highlighted line — a gradient under every gray-crowd line is noise.

What stays clinical and does NOT get the treatment: RECIST reference lines remain
dashed `--border-strong` hairlines (they're thresholds, not decoration), and bars
never get gradients — a gradient-filled waterfall bar makes the value ambiguous.

### 2.7 Sidebar — the numbers and the rules (`Sidebar.jsx`)

The sidebar is an *orientation* device: it answers "where am I" at a glance.
Research-backed spec (NN/g + consensus dimensions):

**Dimensions**
- Expanded: **240–260px**. Collapsed icon rail: **56–72px**.
- Item height: **40–44px** (touch-target floor). Icons: **20px**.
- Group labels: 11–12px uppercase, letter-spaced, `--text-subtle`, with 16–24px
  extra space above — the gap does as much grouping work as the label.

**Structure**
- **5–7 primary items max** (we need ~4–5 — comfortably inside). Beyond that,
  scanning slows and hierarchy collapses.
- Three visually distinct levels: primary (14–15px medium) → secondary children
  (13–14px lighter, indented 16–24px, shown only under the active parent) →
  utility (settings/help/account) **pinned at the bottom**, smaller and grayed,
  separated by whitespace. Equal visual weight on every item is the #1 cited
  sidebar mistake.
- **Labels always visible when expanded.** NN/g: icon-only nav as the default
  state raises cognitive load — "a word is worth a thousand pictures." Icon-only
  is legal ONLY in the collapsed rail, and then every item gets a hover tooltip.

**States** (the quality lives here)
- Active: `--brand-tint` fill + `--brand` text/icon — the sliding active pill
  (§2.5.6) is the premium version. Optional: filled icon variant on active,
  outline elsewhere.
- Hover: `--surface-alt` fill. Focus-visible: the standard 2px brand ring —
  nav is the most keyboard-traversed element in the app.

**Behavior**
- Collapse animates width `--dur-base` `--ease-out`; **icons stay fixed, labels
  fade** (already in the motion budget) — labels sliding or icons jumping during
  collapse is the jank tell.
- **Persist collapse preference in `localStorage`** — resetting on every load is
  a repeatedly-cited failure.
- Never behind a hamburger on desktop; drawer overlay below `md` (matches §2.4.5).

### 2.8 Filter bar — scope, apply/reset, and honest chips

The filter bar answers "what am I looking at." In a clinical dashboard this is
a trust surface, not a convenience: an unnoticed stale filter silently
misrepresents the data.

**Placement & scope**
- **Top bar, above the content** — right for few, high-level, frequently-toggled
  criteria (cohort, response category, active status, date range). Sidebar filter
  panels are for faceted search with dozens of values, not this.
- **A top filter bar must affect EVERYTHING below it.** A filter that applies to
  only one chart lives on that chart's card, never in the page bar — mixed scopes
  are the biggest source of filter confusion.
- Sticky at top on scroll, next to the sticky table header (§2.5.1).

**Apply/reset**
- **Instant apply** when filtering is local/fast (<~300ms) — best feel, no button.
  **Batch apply** (explicit button) only if queries are slow or users set several
  filters before looking; the button says **"Apply"** — never "Close"/"Done" —
  and shows a staged-changes signal (enabled state or count).
- **"Reset" returns to defaults in ONE click and is always visible** whenever any
  filter is non-default. Users explore more freely when the escape hatch is obvious.

**Active-filter visibility** (the part most dashboards get wrong)
- Every active filter renders as a **chip with its own ×** in a row visible
  without opening any dropdown — the user must see *why the numbers look like
  this* at a glance. Chips: `--brand-tint` bg, `--brand` text, `--radius-sm`.
- **"Clear all"** appears at 2+ chips.
- **Result count next to the chips** — "Showing 34 of 128 patients" — confirms
  the filter acted AND keeps the denominator honest (the denominators-everywhere
  rule applied to filtering).
- **Empty-result state offers recovery inline**: "No patients match — clear
  filters?" with the action right there, never just a blank table.

**Styling** — same card language as everything else: white surface, hairline
bottom border, 44px controls at `--radius-md`.

## Part 3 — The merge: one sentence each

- **Clinical** gives the *what*: strict color semantics, denominators, freshness, overview→drill, plots-as-siblings, accessibility as table stakes.
- **Modern** gives the *feel*: <100ms local interactions, short decelerating motion spent only where it means something, all five interaction states, zero layout shift.
- They don't trade off — both converge on **restraint with intention**: the clinical side because attention and color are safety channels; the modern side because restraint is what reads expensive.

## Part 4 — The pure-CSS distillation: ten moves that make a frontend read "good"

Everything above, compressed to what it means in raw CSS. Ordered by impact.
Use as the review checklist when eyeballing any screen.

1. **Background/card contrast, not shadows.** Tinted-gray page (`--bg`), pure-white cards, 1px hairline border. The ~2% brightness gap makes cards float. Card drop-shadows = the #1 amateur tell; the one shadow (`--shadow-pop`) is reserved for things that genuinely float (menus, modals, toasts).
2. **One accent, everything else grayscale.** ~90% grays; indigo only where it means something (primary action, active nav, data). Restraint is what reads expensive.
3. **Type hierarchy from weight + gray, not size.** One sans-serif (Inter); hierarchy via 600-vs-400 and the four text-gray steps, not ten font sizes. Big numbers: `-0.02em` tracking + `tabular-nums`. Never pure #000 on pure #fff.
4. **A 4px spacing scale you never break.** Generous between groups (24px), tight within them (label 4–6px from its value). Proximity does more grouping work than any border.
5. **One radius scale everywhere.** 6 / 10 / 14 / 20px. Mixed radii read chaotic; a scale reads like one hand.
6. **All five interaction states on everything interactive.** Hover, pressed, focus-visible, disabled, selected (§2.3). Users judge polish by the states designers forget; focus rings are a11y and polish in one move.
7. **Motion: few, fast, decelerating.** Three durations, one ease-out, transform/opacity only. Nothing loops, bounces, or replays on refetch (§2.2).
8. **Zero layout shift.** Skeletons at exact final size, fixed chart/table heights, `scrollbar-gutter: stable`, explicit media dimensions (§2.4). Invisible when right — which is why it reads "smooth."
9. **Style the library defaults.** Scrollbars, chart tooltips, focus outlines, Material shadows — defaults break the design language louder than anything you built. Cheap to fix, high yield (§2.5).
10. **Tokens make it enforceable.** Every value above lives once in `:root`. Consistency isn't discipline, it's architecture — you can't drift when there's only one `--border` to use.

> One sentence: one accent on a calm gray-scale, hairlines instead of shadows,
> weight-driven type on a 4px grid, all five states, fast decelerating motion,
> nothing ever shifts — all enforced through tokens rather than memory.

### Build order (updated)

1. Tokens incl. motion tokens (`design-system.md` §1–2) — done on paper
2. Data layer: `useStats` hook + Error/Empty states + DashboardPage branch (`design-system.md` §5)
3. StatCards with n= denominators + freshness stamp in header
4. Interaction-state pass over existing components (§2.3 checklist)
5. Plot unification: tokens for gridlines/legends/tooltips, gray-crowd spider, RECIST lines
6. Layout stability pass (§2.4) + sticky header + fixed DataGrid height + scrollbars
7. Sidebar polish (sliding active pill) + the one magenta hero moment — last, as always
