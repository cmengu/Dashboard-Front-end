# Hummingbird Clinical Dashboard — Design System

Brand-coded token layer + flattened MUI theme + full component/data-layer code.
Stack: **Vite · React (JSX) · Tailwind v3 · MUI (kept for DataGrid/forms)**.
Companion file: `design-principles.md` (the why: clinical × modern principles, keep/throw verdicts, motion budget).

Brand colors extracted from hummingbirdbioscience.com:
- **Indigo `#333674`** — workhorse accent. Allowed on: primary buttons, active nav, links, primary chart series, focus rings, selected rows, progress bars.
- **Magenta `#AF3791`** — the rare pop. Allowed on: ONE hero moment, one emphasized metric, second chart series. Nothing else.
- **Azure `#2D64A5`** — info / third chart series.
- Everything else is grayscale. Color only where it means something.

---

## 0. File scaffolding

```
src/
├── styles/
│   └── tokens.css            # ALL tokens: color + radius + spacing + motion
├── theme.js                  # MUI theme (reads tokens)
├── lib/
│   └── api.js                # fetch functions — components never call fetch directly
├── hooks/
│   └── useStats.js           # data-fetching hooks (loading/error/success state machine)
├── components/
│   ├── ui/                   # design-system primitives — dumb, NO data fetching
│   │   ├── StatCard.jsx
│   │   ├── StatusPill.jsx
│   │   ├── Button.jsx
│   │   ├── EmptyState.jsx    # designed empty state (icon + sentence + action)
│   │   └── ErrorState.jsx    # inline error + retry
│   ├── charts/               # the three plots share their skin here
│   │   ├── chartTheme.js     # ONE place: gridline color, font, tooltip style, category→color map + chart skin (principles §2.6)
│   │   ├── WaterfallPlot.jsx
│   │   ├── SpiderPlot.jsx
│   │   ├── SwimmerPlot.jsx
│   │   └── ChartTooltip.jsx  # shared token-styled tooltip
│   └── layout/
│       ├── Sidebar.jsx       # 240–260px / 56–72px rail; spec in principles §2.7
│       ├── FilterBar.jsx     # top-scope filters + active chips + result count; spec in principles §2.8
│       └── PageHeader.jsx    # title + "data as of" stamp lives here
└── pages/
    └── DashboardPage.jsx     # composes hooks + ui + charts; owns layout only
```

Two load-bearing rules:
1. **`components/ui/` never fetches data.** Primitives take props — that's what makes them reusable on any page with any data source.
2. **`charts/chartTheme.js` is "plots as siblings" as a file** — one category→color map imported by all three plots, so "PR is always the same green" is enforced by the import, not by discipline.

---

## 1. `src/styles/tokens.css`

Single source of truth. Global CSS reads it natively, Tailwind maps it in config,
MUI reads it via `var()` in style overrides.

```css
:root {
  /* ── Surfaces ─────────────────────────────────────────────
     App bg is the brand off-white (whisper of indigo), cards
     are pure white. That ~2% difference is what lets a 1px
     hairline border separate cards with NO shadow. */
  --bg:          #F9F9FD;
  --surface:     #FFFFFF;
  --surface-alt: #F1F2F8;   /* table headers, hover fills, icon chips */

  /* ── Hairline borders (indigo-tinted) ───────────────────── */
  --border:        #E5E6F0;
  --border-strong: #D2D4E2;  /* hover state for interactive cards */

  /* ── Text — near-black leaning indigo, never pure #000 ──── */
  --text-strong: #1C1E33;   /* headings, big stat numbers */
  --text:        #34374D;   /* body */
  --text-muted:  #696C84;   /* labels, secondary text */
  --text-subtle: #9B9DB2;   /* hints, placeholders, disabled */

  /* ── BRAND indigo — data + actions only ─────────────────── */
  --brand:       #333674;
  --brand-hover: #2A2C61;
  --brand-fg:    #FFFFFF;
  --brand-tint:  rgba(51, 54, 116, 0.10);   /* active nav bg, selected row */

  /* ── ACCENT magenta — the one hero pop ──────────────────── */
  --accent:       #AF3791;
  --accent-hover: #972F7D;
  --accent-fg:    #FFFFFF;
  --accent-tint:  rgba(175, 55, 145, 0.10);

  /* ── Azure — info / 3rd chart series ────────────────────── */
  --info:      #2D64A5;
  --info-tint: rgba(45, 100, 165, 0.10);

  /* ── Status — ONLY for status, never decoration ─────────── */
  --success: #12B76A; --success-text: #027A48; --success-tint: rgba(18, 183, 106, .12);
  --warning: #F79009; --warning-text: #B54708; --warning-tint: rgba(247, 144, 9, .12);
  --error:   #D92D20; --error-text:   #B42318; --error-tint:   rgba(217, 45, 32, .12);

  /* ── Radius scale ────────────────────────────────────────── */
  --radius-sm: 6px;    /* pills, small chips */
  --radius-md: 10px;   /* buttons, inputs, icon chips */
  --radius-lg: 14px;   /* cards */
  --radius-xl: 20px;   /* hero card, modals */

  /* ── Spacing (4px base) — pick from this, never random px ── */
  --space-1: 4px;  --space-2: 8px;  --space-3: 12px; --space-4: 16px;
  --space-5: 20px; --space-6: 24px; --space-8: 32px; --space-10: 40px;

  /* ── Motion — the app has exactly three speeds ────────────
     Err short: if an animation feels "nice", it's ~50ms too long. */
  --dur-fast:   120ms;  /* hovers, color/border changes, toggles */
  --dur-base:   200ms;  /* panel/tab transitions, card lift, sidebar collapse */
  --dur-slow:   400ms;  /* one-time entrances only (page load, chart/bar draw) */

  /* Easing — decelerate. Things ARRIVE fast and settle.
     Never linear, never bouncy. */
  --ease-out:    cubic-bezier(0.16, 1, 0.3, 1);   /* the "expensive" ease */
  --ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);  /* between two on-screen states */

  /* ── ONE shadow, floating things only (menus/modals/toasts), NEVER cards ── */
  --shadow-pop: 0 1px 2px rgba(28, 30, 51, .04), 0 4px 12px rgba(28, 30, 51, .08);
}

/* Base — same file or index.css after the :root block */
html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  scroll-behavior: smooth;
}
body {
  background: var(--bg);
  color: var(--text);
  font-family: Inter, system-ui, -apple-system, sans-serif;
}
/* Digits line up in columns — uneven number widths are a subtle
   "amateur" signal in a data UI */
.tabular { font-variant-numeric: tabular-nums; }

/* Content doesn't jump 15px when a scrollbar appears */
main { scrollbar-gutter: stable; }

/* Respect users who turn animation off (clinical = accessibility matters) */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

> **Dark mode note:** deliberately skipped for v1 (see principles file). Because
> everything reads from tokens, dark mode later = one `.dark { --bg: …; }` override
> block + a second MUI palette — not a rewrite. The architecture already paid for it.

---

## 2. `tailwind.config.js` (v3) — map tokens to utilities

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./index.html", "./src/**/*.{js,jsx}"],
  theme: {
    extend: {
      colors: {
        bg:            "var(--bg)",
        surface:       "var(--surface)",
        "surface-alt": "var(--surface-alt)",
        border:        "var(--border)",
        "border-strong": "var(--border-strong)",
        strong:  "var(--text-strong)",
        body:    "var(--text)",
        muted:   "var(--text-muted)",
        subtle:  "var(--text-subtle)",
        brand:   { DEFAULT: "var(--brand)", hover: "var(--brand-hover)" },
        accent:  { DEFAULT: "var(--accent)", hover: "var(--accent-hover)" },
        info:    "var(--info)",
      },
      borderRadius: {
        sm: "var(--radius-sm)", md: "var(--radius-md)",
        card: "var(--radius-lg)", xl: "var(--radius-xl)",
      },
      boxShadow: { pop: "var(--shadow-pop)" },
      fontFamily: { sans: ["Inter", "system-ui", "sans-serif"] },
      /* Motion tokens → real utility names: duration-fast/base/slow, ease-out-token */
      transitionDuration: {
        fast: "var(--dur-fast)",
        base: "var(--dur-base)",
        slow: "var(--dur-slow)",
      },
      transitionTimingFunction: {
        "out-token": "var(--ease-out)",
        "in-out-token": "var(--ease-in-out)",
      },
    },
  },
  plugins: [],
};
```

Now `bg-surface`, `border-border`, `text-strong`, `bg-brand`, `rounded-card`,
`shadow-pop`, `duration-base`, `ease-out-token` all resolve to the same tokens
MUI uses. Retune the whole app's motion by editing one CSS line.

---

## 3. `src/theme.js` — flattened MUI theme (kills the Material look)

De-Materials every existing MUI component (DataGrid included) with zero rewrites:
no elevation shadows, no UPPERCASE buttons, no Roboto, 10px radius, hairline borders.

```js
import { createTheme } from "@mui/material/styles";

// MUI needs literal hex for its contrast-text math (it can't resolve a
// var() string in palette). Keep these 4 in sync with tokens.css.
// Everything in `components` below uses var() and stays live-linked.
export const theme = createTheme({
  palette: {
    primary:    { main: "#333674" },   // --brand
    secondary:  { main: "#AF3791" },   // --accent
    info:       { main: "#2D64A5" },   // --info
    error:      { main: "#D92D20" },
    background: { default: "#F9F9FD", paper: "#FFFFFF" },
    text:       { primary: "#1C1E33", secondary: "#696C84" },
    divider:    "#E5E6F0",
  },
  shape: { borderRadius: 10 },
  typography: {
    fontFamily: "Inter, system-ui, sans-serif",
    button: { textTransform: "none", fontWeight: 600 },  // kill UPPERCASE
  },
  components: {
    MuiPaper: {
      defaultProps: { elevation: 0 },
      styleOverrides: {
        root: {
          backgroundImage: "none",
          border: "1px solid var(--border)",
          boxShadow: "none",
        },
      },
    },
    MuiCard: {
      defaultProps: { elevation: 0 },
      styleOverrides: {
        root: { border: "1px solid var(--border)", borderRadius: 14 },
      },
    },
    MuiButton: {
      defaultProps: { disableElevation: true },
    },
    MuiSelect: {
      styleOverrides: { root: { borderRadius: 10 } },
    },
    MuiAutocomplete: {
      styleOverrides: {
        paper: { boxShadow: "var(--shadow-pop)", border: "1px solid var(--border)" },
      },
    },
    // DataGrid — flatten border, quiet the header row
    MuiDataGrid: {
      styleOverrides: {
        root: { border: "1px solid var(--border)", borderRadius: 14 },
        columnHeaders: { backgroundColor: "var(--surface-alt)" },
      },
    },
  },
});
```

Wire-up in `main.jsx`:

```jsx
import { ThemeProvider, CssBaseline } from "@mui/material";
import { theme } from "./theme";
import "./styles/tokens.css";

<ThemeProvider theme={theme}>
  <CssBaseline />
  <App />
</ThemeProvider>
```

**Rule that keeps the codebase sane:** one element, one system. Never MUI `sx`
AND Tailwind classes on the same element — they fight over specificity and CSS
injection order. Tokens are shared; styling authority per element is not.

**DataGrid virtualization gotcha:** virtualization needs a fixed-height viewport
to know which rows to render. `autoHeight` grows the grid to fit ALL rows → no
viewport → every row becomes real DOM → jank that scales with data.

```jsx
// ❌ defeats virtualization — fine at 20 rows, dies at 1,000
<DataGrid autoHeight rows={patients} columns={cols} />

// ✅ fixed container height — grid virtualizes internally.
//    Bonus: the card occupies 560px in ALL states (skeleton/3 rows/3,000 rows)
//    → zero layout shift.
<div style={{ height: 560 }}>
  <DataGrid rows={patients} columns={cols} />
</div>
```

---

## 4. `src/components/ui/StatCard.jsx`

One shell for all five cards: counts are pure grayscale; rates get a thin indigo
bar (color only on data). Durations now come from the motion tokens.

```jsx
import { useEffect, useState } from "react";

/**
 * StatCard — clinical stat tile. Dumb component: no fetching, props only.
 *
 * props:
 *  label    string            "Total Patients"
 *  value    string | number   128 | "62%"
 *  icon     Lucide component  optional
 *  percent  number            0–100 → renders the indigo bar (rate cards)
 *  hint     string            small context line, e.g. "n=34/55"
 *  hero     boolean           ONE card may set this → magenta bar + tinted chip
 *  loading  boolean           skeleton state (owned by the page, not the card)
 *  onClick  function          optional → card becomes a real <button>
 */
export function StatCard({
  label, value, icon: Icon, percent, hint,
  hero = false, loading = false, onClick, className = "",
}) {
  const hasBar = typeof percent === "number";
  const clamped = hasBar ? Math.min(100, Math.max(0, percent)) : 0;

  // Bar animates 0 → value on mount (width transition needs a start frame)
  const [barWidth, setBarWidth] = useState(0);
  useEffect(() => {
    if (!hasBar) return;
    const raf = requestAnimationFrame(() => setBarWidth(clamped));
    return () => cancelAnimationFrame(raf);
  }, [hasBar, clamped]);

  const barColor = hero ? "bg-accent" : "bg-brand";
  const chip = hero
    ? "bg-accent/10 text-accent"
    : "bg-surface-alt text-muted";

  const Tag = onClick ? "button" : "div";

  if (loading) {
    // Skeleton matches final layout dimensions exactly → no shift when data lands
    return (
      <div className={`rounded-card border border-border bg-surface p-5 ${className}`}>
        <div className="mb-4 h-9 w-9 animate-pulse rounded-md bg-surface-alt" />
        <div className="h-7 w-16 animate-pulse rounded bg-surface-alt" />
        <div className="mt-2 h-3.5 w-24 animate-pulse rounded bg-surface-alt" />
      </div>
    );
  }

  return (
    <Tag
      onClick={onClick}
      className={[
        "group flex flex-col rounded-card border border-border bg-surface p-5 text-left",
        "transition-all duration-base ease-out-token",
        onClick
          // Motion = affordance: lift + shadow ONLY when clickable
          ? "cursor-pointer hover:-translate-y-0.5 hover:border-border-strong hover:shadow-pop " +
            "focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-brand/40 focus-visible:ring-offset-2"
          : "hover:border-border-strong",
        className,
      ].join(" ")}
    >
      {Icon && (
        <span
          className={`mb-4 inline-flex h-9 w-9 items-center justify-center rounded-md ${chip}
                      transition-colors duration-fast group-hover:text-strong`}
        >
          <Icon size={18} strokeWidth={2} aria-hidden />
        </span>
      )}

      {/* Big numerals need slight negative tracking or they look loose */}
      <span className="tabular text-[28px] font-semibold leading-none tracking-[-0.02em] text-strong">
        {value}
      </span>

      <span className="mt-1.5 text-[13px] font-medium leading-snug text-muted">
        {label}
      </span>

      {hasBar && (
        <div
          className="mt-4 h-1.5 w-full overflow-hidden rounded-full bg-surface-alt"
          role="progressbar"
          aria-valuenow={clamped}
          aria-valuemin={0}
          aria-valuemax={100}
          aria-label={label}
        >
          <div
            className={`h-full rounded-full ${barColor} transition-[width] duration-slow ease-out-token`}
            style={{ width: `${barWidth}%` }}
          />
        </div>
      )}

      {hint && (
        <span className="mt-2 text-[12px] leading-snug text-subtle">{hint}</span>
      )}
    </Tag>
  );
}
```

---

## 5. The data layer — loading / empty / error as a state machine

The pattern: one fetch produces a small state machine (`useState` under the hood),
and the JSX renders one branch per state. Split of responsibilities:

> **The hook owns WHEN** (the state machine) · **the page owns WHICH** (the branch)
> · **the primitives own HOW IT LOOKS** (skeleton/empty/error skins).

### 5.1 `src/hooks/useStats.js`

```jsx
import { useState, useEffect, useCallback } from "react";
import { fetchStats } from "../lib/api";

export function useStats() {
  // ONE status string, not separate isLoading/isError booleans:
  // two booleans allow the impossible state `loading && error`.
  // One string = always exactly one state. This IS the state machine.
  const [status, setStatus] = useState("loading"); // 'loading' | 'error' | 'success'
  const [data, setData] = useState(null);

  // useCallback memoizes `load` so the effect below doesn't re-fire every
  // render (an inline function is a NEW function each render → infinite loop).
  const load = useCallback(async () => {
    setStatus("loading");            // retry() also flips UI back to skeletons
    try {
      const result = await fetchStats();   // the await = the hundreds-of-ms gap
      setData(result);
      setStatus("success");
    } catch {
      setStatus("error");
    }
  }, []);

  useEffect(() => { load(); }, [load]);    // run once on mount

  return { status, data, retry: load };
}
```

Why the shell renders immediately (mechanically): the component function runs
**synchronously** and returns JSX first — with `status === "loading"` — and only
*then* does the effect fire the fetch. React literally cannot block on it. The
only decision you own is what the loading branch returns: the REAL layout with
skeletons in the data slots, never a blank page + spinner.

**`empty` is NOT a status.** Empty = successful fetch, zero rows — it's *derived*
(`status === "success" && data.total.n === 0`), never stored. Derive what you
can, store only what you must — this kills more React bugs than any other rule.

> Scaling note: when you have many endpoints, TanStack Query replaces this hook's
> internals (caching, stale-while-revalidate, dedup, refetch-on-focus) and returns
> the same `status`/`data` shape — components stay identical. Learn it this way
> first; swap the engine later.

### 5.2 `src/components/ui/ErrorState.jsx` — inline, human, with a retry

```jsx
export function ErrorState({ message, onRetry }) {
  return (
    <div className="flex flex-col items-center gap-3 rounded-card border border-border bg-surface p-10 text-center">
      <p className="text-[14px] text-muted">{message}</p>
      <button
        onClick={onRetry}
        className="rounded-md bg-brand px-4 py-2 text-[13px] font-semibold text-white
                   transition-colors duration-fast hover:bg-brand-hover
                   focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-brand/40 focus-visible:ring-offset-2"
      >
        Try again
      </button>
    </div>
  );
}
```

### 5.3 `src/components/ui/EmptyState.jsx` — never a blank white card

```jsx
export function EmptyState({ icon: Icon, title, hint }) {
  return (
    <div className="flex flex-col items-center gap-2 rounded-card border border-border bg-surface p-10 text-center">
      {Icon && <Icon size={28} strokeWidth={1.5} className="text-subtle" aria-hidden />}
      <p className="text-[14px] font-medium text-body">{title}</p>
      {hint && <p className="text-[13px] text-subtle">{hint}</p>}
    </div>
  );
}
```

Both use the same card shell (`rounded-card border-border bg-surface`) as
everything else — even the error screen belongs to the family.

### 5.4 `src/pages/DashboardPage.jsx` — composing it all

```jsx
import { Users, Activity, CheckCircle2, Gauge, Target, FolderSearch } from "lucide-react";
import { useStats } from "../hooks/useStats";
import { StatCard } from "../components/ui/StatCard";
import { ErrorState } from "../components/ui/ErrorState";
import { EmptyState } from "../components/ui/EmptyState";

// Card SHAPE is config, not state — lives outside the component, created once.
// Adding a 6th card = one line. Same array drives skeleton grid AND loaded grid,
// which is what guarantees zero layout shift by construction.
const CARD_DEFS = [
  { key: "total",  label: "Total Patients",   icon: Users },
  { key: "active", label: "Currently Active", icon: Activity },
  { key: "pr",     label: "Ever Had PR+",     icon: CheckCircle2 },
  { key: "dcr",    label: "Overall DCR",      icon: Gauge,  isRate: true },
  { key: "orr",    label: "Overall ORR",      icon: Target, isRate: true },
];

export function DashboardPage() {
  const { status, data, retry } = useStats();

  const isEmpty = status === "success" && data.total.n === 0;   // derived, not stored

  return (
    <div className="space-y-6 p-6">
      {/* SHELL — renders in EVERY state, no conditions. User sees the page
          frame at t=0; data fills in rather than "arrives with a shift". */}
      <header className="flex items-baseline justify-between">
        <h1 className="text-xl font-semibold text-strong">Trial Overview</h1>
        {data?.asOf && (
          <span className="text-[12px] text-subtle">Data as of {data.asOf}</span>
        )}
      </header>

      {/* THE BRANCH — one state, one render.
          Order: error first (must win), then empty (only meaningful post-
          success), then loading+success SHARING one branch — same grid, same
          five StatCards, only the `loading` prop flips. */}
      {status === "error" ? (
        <ErrorState message="Couldn't load trial data." onRetry={retry} />
      ) : isEmpty ? (
        <EmptyState
          icon={FolderSearch}
          title="No patients enrolled yet"
          hint="Stats will appear once the first patient is registered."
        />
      ) : (
        <div className="grid grid-cols-2 gap-4 sm:grid-cols-3 xl:grid-cols-5">
          {CARD_DEFS.map((def) => {
            const stat = data?.[def.key];    // undefined while loading — safe,
            return (                          // skeleton branch never reads it
              <StatCard
                key={def.key}
                label={def.label}
                icon={def.icon}
                loading={status === "loading"}
                value={def.isRate ? `${stat?.pct}%` : stat?.n}
                percent={def.isRate ? stat?.pct : undefined}
                hint={stat?.hint}             // e.g. "n=34/55" from the API
              />
            );
          })}
        </div>
      )}
    </div>
  );
}
```

Expected API shape (`lib/api.js` → `fetchStats()`):

```js
{
  asOf: "28 Jun 2026",
  total:  { n: 128 },
  active: { n: 47,  hint: "of 128 enrolled" },
  pr:     { n: 34,  hint: "≥ partial response" },
  dcr:    { pct: 62, hint: "n=34/55 · disease control rate" },
  orr:    { pct: 27, hint: "n=15/55 · objective response rate" },
}
```

---

## 6. Design details recap (what each refinement is doing)

| Detail | Why it reads "sleek" |
|---|---|
| `tracking-[-0.02em]` on the number | Big numerals need slight negative letter-spacing or they look loose |
| `tabular-nums` | Digits equal-width → values don't jitter or misalign across the row |
| Bar animates 0→value on mount, `duration-slow` | The one moment of motion; never re-animates on tab return/refetch |
| Lift + `shadow-pop` **only when clickable** | Motion = affordance; static cards get border-darken only |
| `focus-visible:ring-brand/40` | Keyboard focus ring in brand indigo — a11y and polish in one move |
| Icon chip: neutral gray, brightens on hover | Icons aid scanning but never carry per-card color |
| Skeleton `loading` state at exact dimensions | No layout shift when data arrives |
| `role="progressbar"` + aria values | Screen reader announces "Overall DCR, 62%" — clinical tools get audited |
| `prefers-reduced-motion` kill-switch | Animations collapse to instant for users who opt out |
| `Tag = button` when `onClick` | A clickable div is an a11y bug; a real `<button>` gets keyboard/SR free |
| Grid `2 → 3 → 5` columns (skip 4) | 5 cards at 4-wide leaves a lonely orphan on row two |
| One `status` string, not booleans | Impossible states are unrepresentable; empty is derived, never stored |
| Fixed heights on DataGrid + charts | Virtualization works AND nothing reflows when data renders |

### Rules recap

1. All five cards share one grayscale shell — variety comes from what the metric *is* (count vs rate), not decoration.
2. Indigo appears only on the rate bars. Magenta only if you promote ONE card with `hero` — never more than one.
3. Green/amber/red are reserved for actual status, never card decoration.
4. `components/ui/` never fetches; pages own state; hooks own the machine.
