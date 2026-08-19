# Blog format spec

How the posts in this repo are built, so the next one looks and behaves the same.
Reference design: <https://infini-ai-lab.github.io/ai-environment-architect/>.

A working implementation of everything below is `index.html` plus `sections/`.
**The fastest way to start a new post is to copy those two things and replace the
text and the data.**

---

## 1. Standing instructions

The rules every post is held to, unless explicitly overridden.

1. **Match the reference design as closely as possible.** Same CDNs, same layout
   shell, same type scale, same card treatments. Do not redesign it.
2. **Body text lives in `sections/*.txt`, one file per section**, so any section can be
   edited independently without touching code or rebuilding.
3. **Numbers live in one `DATA` block** near the top of the page script. Words in
   `.txt`, data in the HTML — never the same fact in both places.
4. **Quick to read.** Tight writing; let the figures carry the argument. Target 6–8 min.
5. **Figures are animated and interactive** — they animate in on scroll, headline
   numbers count up, and switchers let the reader change which slice is shown.
6. **Every number comes from the source material**, and figures reproduce the
   source's own figures where one exists. Never invent a value to fill a cell;
   write `—` and say in the caption what is unreported.
7. **TL;DR is collapsible and starts collapsed.**
8. A figure with multiple panels defaults to the **all-panels** view.

---

## 2. Files

```
index.html          the post (self-contained page: shell + figures + parser)
main.html           byte-identical copy of index.html — keep them in sync
BLOG-FORMAT.md      this file
figures/            images the post embeds (cropped source figures, diagrams)
legacy/             previous designs, kept so nothing is lost
sections/
  manifest.txt      which section files render, and in what order
  00-meta.txt       title, subtitle, tags, byline, date, header links
  NN-name.txt       one section of the post each
  README.txt        the author-facing markup cheatsheet
```

`index.html` and `main.html` are duplicates because GitHub Pages serves the bare
URL from `index.html` while `main.html` is the linkable canonical post. After any
edit to one: `cp index.html main.html`.

Companion pages (a code-and-setup page, an appendix) sit beside them as their own
files and are reached from the header link row.

---

## 3. Page shell (copy verbatim)

### Head

```html
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://unpkg.com/react@18.2.0/umd/react.development.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@18.2.0/umd/react-dom.development.js" crossorigin></script>
<script src="https://unpkg.com/prop-types/prop-types.min.js"></script>   <!-- Recharts needs this -->
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script src="https://unpkg.com/recharts@2.12.7/umd/Recharts.js"></script>
```

Fonts: **Inter** for text, **JetBrains Mono** for code and identifiers. Body is
`bg-slate-50`. Ship the reference's spinner markup in `#root`, and wrap the whole
script in `try { … } catch` that prints a visible rendering error.

### Layout

| Element | Classes |
|---|---|
| page | `min-h-screen bg-slate-50 font-sans text-slate-800 leading-relaxed` |
| column | `max-w-3xl mx-auto bg-white min-h-screen shadow-lg border-x border-slate-100` |
| header | `pt-16 pb-8 px-8 md:px-12` |
| main | `px-8 md:px-12 pb-20` |
| tag pill | `px-3 py-1 bg-blue-100 text-blue-700 text-xs font-bold uppercase tracking-wide rounded-full` |
| h1 | `text-4xl md:text-5xl font-extrabold tracking-tight leading-tight` — last line `text-blue-600` |
| subtitle | `text-xl md:text-2xl font-semibold text-slate-700 tracking-tight leading-snug` |
| meta row | `flex items-center gap-6 text-sm text-slate-500 border-b border-slate-100 pb-2 mb-8` |
| section h2 | `text-2xl font-bold text-slate-900 tracking-tight`, in a `mb-6 mt-12` wrapper |
| body text | `text-lg text-slate-700 mb-6 leading-relaxed` |
| list | `text-lg list-disc pl-6 mb-6 space-y-2 text-slate-700` |
| figure card | `my-10 p-5 sm:p-6 bg-white border border-slate-200 rounded-xl shadow-sm` |
| figure title | `text-sm font-bold text-slate-500 uppercase tracking-wide` |
| caption | `text-xs text-slate-400` |
| callout | `my-8 p-6 bg-blue-50 border-l-4 border-blue-600 rounded-r-lg` |
| dark panel | `bg-slate-900 rounded-xl shadow-2xl ring-1 ring-slate-900/5` + fake title bar with three traffic-light dots |
| stat tile | `p-4 rounded-xl bg-white border border-slate-200 shadow-sm`, value `text-2xl font-extrabold text-blue-600` |

The page is **light-mode only**, like the reference. Do not add a theme toggle.

---

## 4. Section markup

Parsed by `parseSection()` / `inline()` in the page. The author-facing copy of this
lives in `sections/README.txt` — keep the two in step.

### Blocks

```
## Heading                   section heading
### Subheading               smaller heading
plain text                   paragraph (consecutive lines join)
- item                       bulleted list
1. item                      numbered list
> text                       highlighted callout block
---                          divider
# text                       comment, never rendered

[[TLDR]]                     collapsible TL;DR, starts open
[[TLDR:closed]]              collapsible TL;DR, starts collapsed   ← the default choice
[[FIG:name]]                 drops in a figure component by name
[[STATS]]                    stat grid; one "value | label | note" per line
[[REF]] authors | title | venue | url
```

### Inline

```
**bold**   *italic*   `code`   [label](url)   10^4   --
```

Inline forms **nest** — `**10^8 items**` superscripts correctly inside bold. Only
`` `code` `` is a leaf. If you add a new inline form, recurse into bold/italic/link
contents or nesting silently breaks.

### `00-meta.txt`

`key: value` per line. `tag`, `link` and `title` may repeat; the rest are single.

```
tag: First Topic
tag: Second Topic
title: First line of the headline,
title_accent: Second line          ← rendered in blue on its own line
subtitle: One sentence saying what the post shows.
byline: Author · Affiliation
readtime: 7 min read
date: Jan 1, 2026
link: Code & setup | setup.html | Code
link: Paper | https://… | FileText
```

Link icons available: `Code`, `FileText`, `Layers`, `Cpu`.

---

## 5. Figures

Each figure is a React component registered in the `FIGURES` map and placed from a
section file with `[[FIG:name]]`. Name them after what they show, not their chart
type. An unknown name renders a visible amber warning listing the valid ones rather
than failing silently.

Every figure:

- sits in a `FigureCard` (title, optional switcher, legend, caption);
- is wrapped in `<Reveal>`, which uses an `IntersectionObserver` to play its
  animation the first time it scrolls into view;
- ships a **table view** behind a "Show data" toggle, so no value is reachable
  only by hovering;
- carries a caption naming the source table or figure it came from.

### Kinds that have earned their place

| Kind | Use it for |
|---|---|
| glyph grid | pass/fail per item across a few conditions (crossed cell = failed) |
| cropped source figure + switcher | reproducing a real figure at readable size |
| dark schematic panel | algorithm, architecture or pipeline walkthroughs |
| segmented bar + chips | a budget split or a proportion |
| grouped horizontal bars | per-item comparison across 2–3 alternatives |
| single-series area chart + metric switcher | one sweep viewed through several metrics |
| stat tiles / count-ups | the headline numbers |

### Chart rules

Non-negotiable, learned the hard way:

- **Never a dual-axis chart.** A source figure may print one — two measures of
  different scale sharing a plot, one on a right-hand axis. Split it: bars for the
  measure, the second quantity as stat tiles above the chart.
- **Colour follows the entity, not the rank.** A given series keeps its hue in every
  chart on the page, so filtering or reordering never repaints it. Slot order:

  | slot | hex | role |
  |---|---|---|
  | 1 | `#2563eb` | blue — the subject of the post / "this work" |
  | 2 | `#ea580c` | orange — first comparison |
  | 3 | `#0d9488` | teal — second comparison |
  | 4 | `#eda100` | yellow — third; sub-3:1 on white, so needs visible labels or the table view |
  | — | `#64748b` | slate — an un-tuned or "do nothing" reference, not a real series |
  | — | `#d03b3b` | status red — failure only, always paired with a `✕` glyph, never colour alone |

  Slots 1–3 pass CVD separation, the normal-vision floor and 3:1 contrast on a white
  surface across **all pairs**. A fourth hue breaks all-pairs, which is why a fourth
  series is usually better off in the table view. Re-validate before changing any of
  these.
- **One series → one colour, no legend** (the title names it). Two or more → a
  legend is always present, so identity is never colour-alone.
- Hairline solid gridlines (`#f1f5f9`), never dashed. Thin bars (`barSize` 9–11),
  `radius={[0,3,3,0]}`, `minPointSize={2}` so a tiny value stays visible.
- No number on every mark. Direct-label selectively; the table view carries the rest.
- Wide tables scroll inside their own `overflow-x-auto` container — the page body
  never scrolls sideways.

### Motion

```js
const STATIC = /[?&]static=1/.test(location.search) ||
               matchMedia('(prefers-reduced-motion: reduce)').matches;
const ANIM = !STATIC;
```

Pass `isAnimationActive={ANIM}` to every Recharts series and gate CSS animations on
the same flag. This respects the reader's motion preference *and* makes the page
screenshot-able.

---

## 6. Preview and verify

The page fetches `sections/*.txt` at runtime, so **it must be served over HTTP** —
a `file://` URL fails on CORS. The page detects this and prints instructions.

```bash
cd <repo>
python3 -m http.server 8000 --bind 127.0.0.1
# open http://127.0.0.1:8000/  and hard-refresh with Ctrl+Shift+R
```

Fetches are cache-busted (`?v=<timestamp>`, `cache: 'no-store'`) so an edited
`.txt` always shows up, but hard-refresh anyway after changing `index.html`.

### URL flags

| flag | effect |
|---|---|
| `?static=1` | freeze every animation and count-up at its final value |
| `?tables=1` | open every figure's data table |

### Checking a render without a browser

```bash
google-chrome --headless=new --no-sandbox --disable-gpu --hide-scrollbars \
  --force-prefers-reduced-motion --virtual-time-budget=25000 \
  --window-size=1100,22000 --screenshot=out.png \
  "http://127.0.0.1:8000/index.html?static=1&tables=1"
```

- `?static=1` / `--force-prefers-reduced-motion` are **required**. Recharts' entry
  animation never completes under `--virtual-time-budget`, so without them every bar
  screenshots at zero width and the chart looks broken when it is fine.
- A tall `--window-size` renders the whole page in one shot *and* trips every
  `IntersectionObserver`, so all figures mount.
- To find where content ends, measure dark pixels **inside the card** (x ≈ 230–870
  at 1100 px wide). The card's `border-x` makes every row of the full-width image
  non-uniform, so a whole-row uniformity test always says "content".
- Also check 390 px wide for mobile.
- Open the screenshots and read them. A chart that renders is not a chart that is
  correct; check the numbers against the source.

---

## 7. Traps

- **Recharts + log axis + bars.** `scale="log"` with scalar values does work; bars
  clamp to the domain minimum. Set `domain` and `ticks` to explicit decades and
  format them as `10ⁿ`, or the formatter labels non-decade values nonsensically.
- **Recharts needs `prop-types`** loaded before it, or the UMD build throws.
- **Babel `text/babel`**: no `data-presets` attribute — the reference omits it, and
  adding it changes which transforms run.
- **A custom `Icon` component must forward `style`**, or per-icon colours silently
  do nothing.
- **Don't number references with a module-level counter** that increments during
  render; derive the number from the array index.
- **Back up before overwriting.** Check `git status` first — pages carrying
  uncommitted edits go to `legacy/` before being replaced.

---

## 8. Starting a new post

1. Copy `index.html`, `sections/` and `figures/` into the new post's directory;
   `cp index.html main.html`.
2. Rewrite `sections/00-meta.txt`, delete the section files you do not want, and
   update `sections/manifest.txt` to match.
3. Replace the `DATA` block in `index.html` with the new post's numbers, keeping the
   `C` colour tokens as they are.
4. Delete the figure components that do not apply, add new ones in the same shape
   (`FigureCard` + `Reveal` + table view), and register them in `FIGURES`.
5. Write the sections. One idea per section file; a figure every 2–3 paragraphs.
6. Serve, screenshot at 1100 px and 390 px with `?static=1&tables=1`, and read the
   images before calling it done.
