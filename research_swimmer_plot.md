# Research: Swimmer Plot UI/UX

Context: the dashboard has a `SwimmerPlot.jsx` (see `design-system.md` §charts).
This doc gathers the conventions a swimmer plot MUST follow to read as clinically
correct, plus the interactive features that make a *web* version better than a
static journal figure. Applies the house rules from `design-system.md` (brand
color grammar, one chart skin in `chartTheme.js`).

---

## What a swimmer plot is (so choices make sense)

One horizontal bar per patient = their time on study. Symbols on the bar mark
clinical events (response onset, progression, death). It exists for early-phase
oncology where aggregate stats hide individual response patterns — the reader
scans one patient at a time. Every design decision below serves "can I read one
patient's whole story along one line."

---

## Non-negotiable conventions (get these wrong = looks amateur to a clinician)

1. **One bar per patient, horizontal.** Y-axis = subjects, X-axis = time
   (days or months from treatment start — pick one unit and label it).
2. **Sort by bar length (treatment duration), longest at top → shortest at
   bottom.** This is THE convention. The descending-staircase shape is what makes
   the plot instantly legible. Do not sort by subject ID.
3. **Ongoing treatment = right-pointing arrow (▶) at the bar's end.** Censoring at
   data cutoff. A patient still on treatment gets an arrowhead, not a flat cap.
   This is the single most recognized swimmer-plot signal — never omit it.
4. **Response type = distinct marker shape AND color, placed at the event's
   time.** The de-facto standard shape set:
   - CR (complete response) = filled circle
   - PR (partial response) = filled triangle
   - SD (stable disease) = filled diamond
   - PD (progressive disease) = filled square
   Shape carries meaning even in grayscale/print — color alone is not enough
   (colorblind reviewers).
5. **A legend is mandatory** and must decode both the bar fill (what the bar color
   means) and every marker shape. A swimmer plot without a legend is unreadable.
6. **Bar fill encodes ONE categorical dimension** — usually disease stage at
   baseline or treatment arm. Do not encode two things in the bar.

---

## Applying the Hummingbird color grammar (house rule)

The clinical convention says "distinct color per response," but our system says
~90% grayscale, indigo on data, status colors only for status. Reconcile like
this, and put the map in `chartTheme.js` so all three plots share it:

- **Bar fill = arm/stage** → the three brand series colors: indigo `#333674`,
  magenta `#AF3791`, azure `#2D64A5`. Same arm→color map the pie/waterfall use.
- **Response markers** → lean on SHAPE for the four categories (circle/triangle/
  diamond/square), and keep marker color restrained:
  - CR/PR (the good outcomes) → indigo family / filled dark.
  - SD → neutral gray.
  - PD → the reserved muted red `#D92D20` — this is the ONE place a status color
    on a marker is legitimate, because progression IS a clinical status.
  - Death → an ✕ or open marker in `--text-strong`.
- Ongoing arrow → same color as its bar.

Net: shape does the heavy lifting, color stays on-brand, and PD's red is earned.

---

## Web interactivity — what makes it beat a static figure

From Boehringer's `dv.swimmerplot` module (a production clinical dashboard
component) — these are the features worth building, in priority order:

1. **Hover tooltip per element.** Two tooltip types:
   - Hover a bar → Subject ID, arm/stage, treatment start day, end day, duration,
     ongoing (Y/N).
   - Hover a marker → Subject ID, event type (CR/PR/SD/PD/death), study day.
   This replaces cramming text onto the plot. Keep it the shared token-styled
   `ChartTooltip`.
2. **Sort control.** Let the user re-sort: default = duration (longest→shortest),
   plus options to sort by arm, by baseline stage, or by time-to-first-response.
   Direction toggle (asc/desc). Default MUST be duration.
3. **Filter by arm / response category.** Dropdown or chips to show only Arm A, or
   only patients who ever had PR+. Ties into the app's existing FilterBar pattern.
4. **Reference line at data-cutoff date** (vertical dashed line) — orients "how
   much of this is censored."
5. **Click a bar → open that patient's detail** (if a patient page exists).

**Skip for v1** (documented as not-worth-it here so nobody adds them): zoom/pan,
draggable reordering, animated bar-draw on load (the app removed number/chart
load animations per client request — swimmer bars render at final state too).

---

## Layout / readability specifics

- **Remove chart frame and vertical gridlines.** Keep faint horizontal alignment
  only if needed; the bars themselves are the rows.
- **Hide Y-axis tick marks**; the subject ID can live in the hover tooltip or as a
  small left-margin label, not as heavy axis furniture.
- **X-axis:** labeled with unit ("Months from first dose"), light 1px gridlines
  at sensible intervals, no tick marks — matches the §2.6 chart skin.
- **Fixed-height, scrollable container** if there are many patients (30 patients =
  ~30 rows; give each row a fixed height ~20–24px and let the card scroll rather
  than shrinking bars to illegibility).
- **Row hover highlight** — faint `--surface-alt` band across the hovered lane so
  the eye tracks one patient across the full width.
- **Combined waterfall + swimmer** is a known "advanced" move (waterfall of tumor
  % change beside the swimmer). Powerful but only if the client asks — it doubles
  the legend load. Note it as a possible upgrade, not v1.

---

## Recommendation for the build

For `SwimmerPlot.jsx`, v1 scope:
- Bars sorted by duration desc, fill = arm (3 brand colors from `chartTheme.js`).
- Four response markers by shape; PD in reserved red; ongoing = right arrow.
- Shared hover tooltip (bar + marker variants), legend decoding fill + shapes.
- Sort control (default duration) + arm/response filter wired to FilterBar.
- Cutoff reference line. No load animation. De-defaulted chart skin per §2.6.

That set is clinically correct, on-brand, and clearly a step past a static figure
without tipping into gimmick territory.

---

## Making it visually appealing (the aesthetic layer)

"Make it make sense" is the floor. This section is the ceiling — how to make the
same correct plot look *designed*. The principle: a swimmer plot gets its beauty
from **rhythm, restraint, and one moment of color** — never from decoration added
on top. Everything below earns its pixels.

### A. The bars themselves — this is 80% of the look
- **Rounded bar caps (pill-shaped lanes).** Square-ended bars read as a spreadsheet;
  fully rounded end-caps (`border-radius: 999px` / `linecap: round`) instantly read
  as modern. This one change does more than any color choice.
- **Consistent lane height with a real gap between lanes.** Aim bar height ≈ 55–65%
  of the row height so there's a clear channel of background between patients. Bars
  that touch look like a heat grid; bars floating in whitespace look intentional.
  Peltier's rule of thumb: tune the Y-scale so top/bottom margins are tight but
  lanes never crowd.
- **Thin, not chunky.** ~14–20px bars with generous gaps read more elegant than
  fat bars packed edge to edge. Density comes from the count of lanes, not their
  thickness.
- **Subtle depth, not shadow.** If a bar needs presence, a 1px inner border one
  shade darker than its fill beats any drop shadow. Keep the app's "hairlines not
  shadows" rule.

### B. Color — one saturated moment on a calm field
The trap: giving every response category a loud color turns the plot into
confetti. The elegant move (matches the house grammar):
- **Bars = one calm hue per arm**, but pulled toward the *muted/desaturated* end
  (e.g. indigo at ~70% saturation, not full strength). Muted bars let the markers
  pop.
- **Markers = the saturated accents.** Because bars are calm, a single filled PR
  triangle or a red PD square becomes the eye magnet — exactly the clinical event
  you want seen first. Contrast of saturation (calm bar, vivid marker) is what
  makes it feel crafted.
- **Grayscale/mono fallback works.** A genuinely well-designed swimmer plot still
  reads in grayscale because shape + value carry it. Test it: desaturate the whole
  thing — if it's still legible, the design is sound (this is also the print/
  colorblind-safe test).
- **Never RAG (red/amber/green) the bars.** That's Gantt-tool convention and reads
  generic/corporate; our red is reserved for PD only.

### C. Markers — the jewelry
- **Size markers deliberately larger than the bar is thick** so they sit *on* the
  lane like beads on a string, not buried in it. ~1.3–1.5× the bar height.
- **White halo/stroke around each marker** (1–2px white outline). When a marker
  sits on a colored bar, a thin white ring separates it cleanly — the single most
  effective "this looks professional" trick for overlaid markers.
- **Join related events with a connecting segment** in the event's color (response
  onset → response end). ggswim calls this out: linking start/end lets the eye read
  each response episode *and its duration* as one object. Turns scattered dots into
  a readable story.
- Keep the four shapes (circle/triangle/diamond/square) — shape variety is itself
  visually interesting AND functional.

### D. Layout rhythm & framing
- **Group the lanes by arm with a hair of extra gap between groups** (a slightly
  bigger channel every time the arm changes, if sorted by arm). Micro-grouping
  creates visual rhythm without lines.
- **Kill the chart frame and all vertical gridlines.** A few very light horizontal
  guide lines at x-axis intervals only. The plot should look like it floats on the
  card, not boxed in.
- **Generous left margin** for subject labels in a small, muted, tabular font —
  aligned, never wrapping. Alignment is 90% of "clean."
- **A single vertical accent line at data cutoff**, dashed, in a muted tone — it
  frames the censoring story and adds one intentional vertical element to an
  otherwise horizontal composition.
- **Legend as a tidy horizontal strip** under the title, not a boxed panel floating
  in the corner. Merge the bar-fill legend and the marker legend into one clean row
  (ggswim's merged-legend technique) so the reader learns the whole language in one
  glance.

### E. Motion (within the client's "no load animation" constraint)
- No bar-draw-in on load (bars render final — consistent with the app rule).
- Allowed, because it's interaction not decoration: on hover, the whole lane
  brightens slightly and its markers scale ~1.1× (120ms). That hover life is what
  separates a web plot from a screenshot, and it costs nothing on load.

### F. The upscale option (only if the client wants "wow")
**Paired waterfall + swimmer**, sharing the Y-axis: a horizontal tumor-% -change
waterfall on the left, the swimmer timeline on the right, one row per patient.
It's the most impressive oncology figure there is — but it doubles the legend and
data load. Offer as a v2 showpiece, not the default.

### Quick "is it beautiful yet" checklist
- [ ] Bars are pill-shaped with visible background channels between them.
- [ ] Bar colors are muted; markers are the saturated pop.
- [ ] Every marker has a thin white halo and sits proud of its lane.
- [ ] Response episodes are joined by a colored connecting segment.
- [ ] No chart frame, no vertical gridlines, one dashed cutoff line.
- [ ] Subject labels are a small aligned tabular column; legend is one merged strip.
- [ ] Desaturating the whole plot to grayscale still reads correctly.
- [ ] Hover brings the lane to life; nothing animates on load.

---

## Sources

- https://chop-cgtinformatics.github.io/ggswim/ — modern swimmer aesthetics: merged legends, event-joining segments, marker-as-separate-aes, theme_ggswim polish
- https://peltiertech.com/swimmer-plots-excel/ — concrete craft: lane height/spacing, no end caps, hidden-element legend tricks, tight Y-scale margins
- https://www.themillerlab.io/posts/swimmer_plots/ — clinical swimmer plot conventions (sorting, arrows, response markers, coord_flip layout)
- https://boehringer-ingelheim.github.io/dv.swimmerplot/articles/dv-swimmerplot.html — production interactive module: tooltip vars, sort direction, group/filter, CR/PR/SD/PD shape map, ongoing arrows
- https://www.lexjansen.com/phuse-us/2020/dv/DV07.pdf — "Visualizing Clinical Trial Design in 20 Graphs" (swimmer among them)
- https://pharmasug.org/proceedings/2014/DG/PharmaSUG-2014-DG07.pdf — original "Swimmer Plot: Tell a Graphical Story of Time to Response"
- https://pharmasug.org/proceedings/2025/DV/PharmaSUG-2025-DV-337.pdf — combined waterfall + swimmer plot
- https://cran.r-project.org/web/packages/swimplot/vignettes/Introduction.to.swimplot.html — swimplot R package reference
