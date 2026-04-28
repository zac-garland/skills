# PPTX Slide Design System
**Dark-Theme Analytical Deck — pptxgenjs Implementation Guide**

---

## Philosophy

This system produces presentation-quality slides in a dark analytical aesthetic. The visual language prioritizes:
- **Information density without clutter** — every element earns its space
- **Typographic hierarchy** — the reader's eye should land on the right thing in the right order
- **Color as signal** — accent colors carry meaning, not just decoration
- **Consistency through tokens** — all sizing, color, and spacing decisions trace back to this document

Slides built with this system should feel like a well-designed Bloomberg terminal or research report, not a default PowerPoint template.

---

## Setup

```js
const pptxgen = require("pptxgenjs");
const pres = new pptxgen();
pres.layout = "LAYOUT_WIDE"; // 13.33" × 7.5" — standard widescreen
```

---

## Color Tokens

```js
const C = {
  // Backgrounds
  bg:      "07080D",   // slide background — near-black with slight blue cast
  surface: "0F1118",   // slightly elevated surface
  card:    "141720",   // card / panel background
  card2:   "1A1E2B",   // header row in tables, slightly elevated card
  border:  "1E2030",   // borders, dividers, grid lines

  // Accents — each carries a semantic role (see below)
  cyan:    "00E5FF",   // primary accent — key findings, links, headers
  coral:   "FF4F6D",   // negative / risk / warning
  lime:    "B8FF57",   // positive / confirmed / strong signal
  amber:   "FFB830",   // secondary / caution / subheads
  purple:  "9B6DFF",   // tertiary / supplemental data
  slate:   "78909C",   // neutral / muted data / de-emphasized rows

  // Text
  text:    "F0F2F8",   // primary text — near white
  muted:   "6B7A99",   // secondary labels, axis text, footnotes
  body:    "B0BBCE",   // body copy inside cards
};
```

### Semantic Color Roles
| Color | Use |
|---|---|
| `cyan` | Primary signal, key result, slide titles, table headers, top-bar accent |
| `lime` | Positive outcome, statistical significance, upward delta |
| `coral` | Negative outcome, risk, downward delta, warning |
| `amber` | Secondary emphasis, subheadings, section labels, caution |
| `purple` | Tertiary data series, supplemental or contextual values |
| `slate` | Neutral / not significant / de-emphasized comparison rows |
| `muted` | All secondary text: axis labels, footnotes, subtext |

---

## Typography

```js
// Display / slide titles
fontFace: "Georgia"          // serif — weight and authority
fontSize: 28–40              // varies by content density

// Section eyebrows (all-caps label above title)
fontFace: "Courier New"      // monospace — data/technical feel
fontSize: 8
charSpacing: 3

// Body copy inside cards
fontFace: "Calibri"          // clean, readable at small sizes
fontSize: 9–11

// Data values, code, table content
fontFace: "Courier New"      // monospace — aligns numbers, looks precise
fontSize: 8–11

// KPI values (large hero numbers)
fontFace: "Georgia"
fontSize: 24–36
bold: true
```

### Type Scale
| Element | Font | Size | Color |
|---|---|---|---|
| Slide title — primary word | Georgia, bold | 32–40 | `text` |
| Slide title — accent word | Georgia, bold | 32–40 | `cyan` |
| Section eyebrow | Courier New | 8 | `cyan`, `charSpacing: 3` |
| Headline banner body | Calibri | 10–11 | `body` |
| Headline banner label | Calibri, bold | 10–11 | `cyan` |
| Card section header | Calibri, bold | 11–13 | accent color |
| Card body text | Calibri | 9.5–10.5 | `body` |
| Table header cell | Calibri, bold | 9–10.5 | `cyan` |
| Table body cell | Calibri | 9–10 | `body` |
| Data / coefficient | Courier New | 8–10 | varies by sign |
| Footnote / footer | Courier New | 7–8 | `muted` |
| KPI value | Georgia, bold | 24–32 | accent color |
| KPI label | Courier New | 7 | `muted`, `charSpacing: 1.5` |

---

## Layout Grid

```
Slide dimensions: 13.33" wide × 7.5" tall

Margins:
  Left/Right edge: 0.4"
  Top content start: ~0.55" (below eyebrow)
  Bottom content end: ~7.2" (above footer)
  Usable width: 12.5"
  Usable height: ~6.0" (between title and footer)

Common column splits:
  Two equal columns:  6.1" each, gap 0.1"
  Left 60 / Right 40: 7.5" / 4.85"
  Left 65 / Right 35: 8.5" / 4.1" (chart + sidebar)
  Full width panel:   12.5"

Row zones:
  Eyebrow:        y = 0.3,  h = 0.22
  Title:          y = 0.55, h = 0.85
  Accent rule:    y = 1.48, h = 0.03
  Headline box:   y = 1.57, h = 0.55
  Content zone:   y = 2.25 → 7.1
  Footer rule:    y = 7.22
  Footer text:    y = 7.27
```

---

## Core Component Library

### `bg(slide)` — Apply Background
```js
function bg(s) {
  s.background = { color: C.bg };
}
```

### `card(slide, x, y, w, h, opts)` — Dark Card / Panel
```js
function card(s, x, y, w, h, opts = {}) {
  s.addShape(pres.shapes.RECTANGLE, {
    x, y, w, h,
    fill: { color: opts.fill || C.card },
    line: { color: C.border, pt: 0.5 },
    shadow: { type: "outer", color: "000000", blur: 4, offset: 1, angle: 135, opacity: 0.25 }
  });
  // Optional top accent bar (color bar along top edge)
  if (opts.topColor) {
    s.addShape(pres.shapes.RECTANGLE, {
      x, y, w, h: 0.04,
      fill: { color: opts.topColor }, line: { type: "none" }
    });
  }
}
```

**Usage:** `card(s, 0.4, 2.25, 6.1, 3.5, { topColor: C.cyan })`

### `leftBar(slide, color, x, y, h)` — Vertical Accent Bar
```js
function leftBar(s, color, x, y, h) {
  s.addShape(pres.shapes.RECTANGLE, {
    x, y, w: 0.04, h,
    fill: { color }, line: { type: "none" }
  });
}
```
Used on the left edge of headline banners and callout boxes.

### `eyebrow(slide, text, x, y)` — Section Label
```js
function eyebrow(s, text, x, y) {
  s.addText(text.toUpperCase(), {
    x, y, w: 12, h: 0.22,
    fontSize: 8, fontFace: "Courier New",
    color: C.cyan, charSpacing: 3, margin: 0
  });
}
```
Always placed at `y = 0.3`. Format: `"SECTION N · TOPIC LABEL"`.

### `slideTitle(slide, parts, x, y)` — Two-Tone Title
```js
function slideTitle(s, parts, x, y) {
  s.addText(parts, {
    x, y, w: 12.5, h: 0.85,
    fontSize: 32, fontFace: "Georgia", bold: true,
    charSpacing: -0.5, margin: 0
  });
}
// parts example:
// [{ text: "The Signal ", options: { color: C.text } },
//  { text: "Works",       options: { color: C.cyan } }]
```

### `rule(slide, x, y)` — Cyan Accent Rule
```js
function rule(s, x, y) {
  s.addShape(pres.shapes.RECTANGLE, {
    x, y, w: 0.55, h: 0.03,
    fill: { color: C.cyan }, line: { type: "none" }
  });
}
```
Always placed at `y = 1.48`, immediately below the title.

### `headline(slide, text, x, y, w)` — One-Sentence Takeaway Banner
```js
function headline(s, text, x, y, w) {
  // Translucent cyan background
  s.addShape(pres.shapes.RECTANGLE, {
    x, y, w, h: 0.55,
    fill: { color: C.cyan, transparency: 94 },
    line: { color: C.cyan, pt: 0.5, transparency: 78 }
  });
  leftBar(s, C.cyan, x, y, 0.55);
  s.addText([
    { text: "📌 ", options: { fontSize: 10 } },
    { text: "The idea in one sentence: ", options: { fontSize: 10, color: C.cyan, bold: true } },
    { text, options: { fontSize: 10, color: C.body } }
  ], { x: x + 0.1, y: y + 0.08, w: w - 0.15, h: 0.42, fontFace: "Calibri", margin: 0 });
}
```
Always placed at `y = 1.57`. Full width: `w = 12.5`.

### `kpi(slide, value, label, sub, x, y, color)` — KPI Card
```js
function kpi(s, val, label, sub, x, y, color) {
  card(s, x, y, 2.35, 1.0, { topColor: color });
  s.addText(val,   { x: x+0.15, y: y+0.1,  w: 2.05, h: 0.5,  fontSize: 28, fontFace: "Georgia", bold: true, color, margin: 0 });
  s.addText(label.toUpperCase(), { x: x+0.15, y: y+0.58, w: 2.05, h: 0.2, fontSize: 7, fontFace: "Courier New", color: C.muted, charSpacing: 1.5, margin: 0 });
  if (sub) s.addText(sub, { x: x+0.15, y: y+0.76, w: 2.05, h: 0.18, fontSize: 7, fontFace: "Courier New", color: C.slate, margin: 0 });
}
```
Five KPIs fit across full width at `x = [0.4, 2.9, 5.4, 7.9, 10.4]`, `y = 2.3`.

### `pill(slide, text, x, y, color)` — Tag / Badge
```js
function pill(s, text, x, y, color) {
  const w = text.length * 0.072 + 0.25;
  s.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x, y, w, h: 0.22,
    fill: { color, transparency: 90 },
    line: { color, pt: 0.5, transparency: 55 },
    rectRadius: 0.05
  });
  s.addText(text, { x: x+0.08, y: y+0.02, w: w-0.1, h: 0.18, fontSize: 7.5, fontFace: "Courier New", color, bold: true, margin: 0 });
}
```

### `footer(slide, leftText, rightText)` — Slide Footer
```js
function footer(s, left, right) {
  s.addShape(pres.shapes.RECTANGLE, { x: 0.4, y: 7.22, w: 12.5, h: 0.02, fill: { color: C.border }, line: { type: "none" } });
  s.addText(left,  { x: 0.4,  y: 7.27, w: 7,   h: 0.22, fontSize: 8, fontFace: "Courier New", color: C.muted, margin: 0 });
  s.addText(right, { x: 7.5,  y: 7.27, w: 5.3, h: 0.22, fontSize: 8, fontFace: "Courier New", color: C.muted, align: "right", margin: 0 });
}
```
Right text is typically `"Slide N / TOTAL"`.

---

## Table Styling

### Standard Dark Table
```js
s.addTable(rows, {
  x, y, w, h,
  fontSize: 10, fontFace: "Calibri",
  border: { pt: 0.3, color: C.border },
  fill: { color: C.card },
  color: C.body,
});
```

### Header Row Pattern
```js
// First row of `rows` array:
[
  { text: "Column A", options: { bold: true, color: C.cyan, fill: { color: C.card2 } } },
  { text: "Column B", options: { bold: true, color: C.cyan, fill: { color: C.card2 } } },
]
```

### Column Width Array
Pass `colW: [w1, w2, ...]` to `addTable()` when columns need unequal widths. Sum must equal `w`.

### Conditional Cell Colors
```js
// Positive value → lime, negative → coral, neutral → body
const cellColor = val > 0 ? C.lime : val < 0 ? C.coral : C.body;
{ text: val.toFixed(2), options: { color: cellColor, bold: Math.abs(val) > threshold } }
```

### Significance Stars Convention
```js
function stars(p) {
  if (p < 0.01) return "***";
  if (p < 0.05) return "**";
  if (p < 0.10) return "*";
  return "";
}
```

---

## Chart Styling

### Common Options (apply to all chart types)
```js
{
  chartArea: { fill: { color: C.card } },
  plotArea:  { fill: { color: C.card } },
  catAxisLabelColor: C.muted,
  valAxisLabelColor: C.muted,
  catAxisLineColor:  C.border,
  valAxisLineColor:  C.border,
  valGridLine: { color: "1E2030", size: 0.5 },
  catGridLine: { style: "none" },
  showLegend:  true,
  legendPos:   "b",
  legendFontSize: 9,
  legendColor: C.muted,
  valAxisTitleColor: C.muted,
  catAxisTitleColor: C.muted,
}
```

### Chart Color Sequences
```js
// Ordered palette for multi-series charts
chartColors: [C.cyan, C.amber, C.lime, C.coral, C.purple, C.slate]

// Diverging (positive/negative) — single series, conditional
const barColors = data.map(v => v >= 0 ? C.cyan : C.coral);
```

### Line Chart
```js
s.addChart(pres.charts.LINE, seriesData, {
  x, y, w, h,
  lineSize: 2.5,
  lineSmooth: false,
  ...commonOptions,
});
```

### Bar / Column Chart
```js
s.addChart(pres.charts.BAR, seriesData, {
  x, y, w, h,
  barDir: "col",  // "col" = vertical bars, "bar" = horizontal
  showValue: true,
  dataLabelFontSize: 9,
  dataLabelColor: C.text,
  dataLabelPosition: "outEnd",
  ...commonOptions,
});
```

### Series Data Format
```js
const seriesData = [
  {
    name: "Series Label",
    labels: ["A", "B", "C", "D"],   // x-axis categories
    values: [1.2, -0.5, 0.8, 1.4],  // y-axis values
  }
];
```

---

## Slide Templates

### Template A — Title Slide
```
Components:
  Decorative oval (bg accent, x:8 y:-1.5 w:6 h:6, fill cyan transparency:95)
  Ghost watermark text (large, transparency:97, background decoration)
  eyebrow()         y=0.9
  Two-tone title    y=1.3, fontSize=52
  Headline banner   y=3.35
  Pills row         y=4.35
  Author name       y=4.8
  footer()
```

### Template B — Results / Data Slide (KPIs + Table)
```
Components:
  eyebrow()         y=0.3
  slideTitle()      y=0.55
  rule()            y=1.48
  headline()        y=1.57
  5× kpi()          y=2.3
  card() left half  y=3.5, w=6.1, h=2.8
  card() right half y=3.5, w=6.1, h=2.8
  footer()
```

### Template C — Comparison Table (Two Clustered Blocks)
```
Components:
  eyebrow()
  slideTitle()
  rule()
  Headline banner (key finding)
  Column header bar     (full width, C.card2)
  Two color-blocked sub-headers (amber block / cyan block)
  Sub-label row         (SE / t-stat / p for each side)
  Section divider rows  (colored, per model)
  Data rows             (alternating fill: C.card / "0A0B10")
  Footnote bar          (y≈7.05, Courier New 7pt)
  footer()
```

### Template D — 2×2 Model Cards
```
Components:
  eyebrow()
  slideTitle()
  rule()
  headline()
  4× card()         2 columns × 2 rows, each with:
    - Color top bar
    - Model label (bold, accent color)
    - Result box (translucent bg, Courier New coefficients)
    - Plain-English interpretation (Calibri, C.body)
  Controls bar      (full width, cyan tint, y≈6.45)
  footer()
```

### Template E — Waterfall / Funnel
```
Components:
  eyebrow()
  slideTitle()
  rule()
  headline()
  Horizontal bars   (proportional width to value, each a distinct color)
  Stage labels      (right of each bar, with Δ% reduction)
  Right panel card  (breakdown table)
  footer()
```

---

## Decorative Elements

### Background Oval (Title / Hero Slides)
```js
s.addShape(pres.shapes.OVAL, {
  x: 8, y: -1.5, w: 6, h: 6,
  fill: { color: C.cyan, transparency: 95 },
  line: { type: "none" }
});
```

### Ghost Watermark Text
```js
s.addText("Q5", {
  x: 7.5, y: 1.5, w: 5, h: 3.5,
  fontSize: 200, fontFace: "Georgia", bold: true,
  color: "FFFFFF", transparency: 97,
  align: "center", margin: 0
});
```

### Section Divider Row (inside tables / comparison views)
```js
s.addShape(pres.shapes.RECTANGLE, { x, y, w: 12.5, h: 0.24, fill: { color: accentColor, transparency: 91 }, line: { color: accentColor, pt: 0.5, transparency: 75 } });
leftBar(s, accentColor, x, y, 0.24);
s.addText(sectionLabel, { x: x+0.12, y: y+0.03, w: 12.2, h: 0.18, fontSize: 8.5, fontFace: "Courier New", color: accentColor, bold: true, margin: 0 });
```

### Full-Width Callout Box (controls / methods note)
```js
s.addShape(pres.shapes.RECTANGLE, { x: 0.4, y, w: 12.5, h: 0.48, fill: { color: C.cyan, transparency: 94 }, line: { color: C.cyan, pt: 0.5, transparency: 78 } });
leftBar(s, C.cyan, 0.4, y, 0.48);
s.addText([
  { text: "Label: ", options: { bold: true, color: C.cyan } },
  { text: "body text here.", options: { color: C.body } }
], { x: 0.58, y: y+0.06, w: 12.2, h: 0.38, fontSize: 9.5, fontFace: "Calibri", margin: 0 });
```

---

## Numbered Step Cards

Used for pipeline / methodology / process slides:

```js
const steps = [
  { n: "01", name: "Step Name", desc: "Description text.", color: C.cyan   },
  { n: "02", name: "Step Name", desc: "Description text.", color: C.amber  },
  { n: "03", name: "Step Name", desc: "Description text.", color: C.purple },
  { n: "04", name: "Step Name", desc: "Description text.", color: C.lime   },
  { n: "05", name: "Step Name", desc: "Description text.", color: C.coral  },
];

const bw = 2.38, bh = 3.2, gap = 0.1, bx = 0.4, by = 2.25;
steps.forEach((st, i) => {
  const x = bx + i * (bw + gap);
  card(s, x, by, bw, bh, { topColor: st.color });
  s.addText(st.n,   { x: x+0.15, y: by+0.15, w: bw-0.2, h: 0.55, fontSize: 26, fontFace: "Georgia", bold: true, color: st.color, margin: 0 });
  s.addText(st.name,{ x: x+0.15, y: by+0.72, w: bw-0.2, h: 0.35, fontSize: 12, fontFace: "Calibri", bold: true, color: C.text, margin: 0 });
  s.addText(st.desc,{ x: x+0.15, y: by+1.1,  w: bw-0.2, h: 2.0,  fontSize: 9.5,fontFace: "Calibri", color: C.body, margin: 0 });
});
```

---

## Two-Column Arm Cards

Used for hypothesis / methodology slides with two parallel tracks:

```js
const arms = [
  { color: C.amber, title: "Arm A · Label",  subtitle: "Framing question?", items: ["Item 1", "Item 2", "Item 3"] },
  { color: C.cyan,  title: "Arm B · Label",  subtitle: "Framing question?", items: ["Item 1", "Item 2", "Item 3"] },
];

arms.forEach((arm, i) => {
  const x = 0.4 + i * 6.3;
  card(s, x, 2.45, 6.1, 4.4, { topColor: arm.color });
  s.addText(arm.title,    { x: x+0.15, y: 2.55, w: 5.8, h: 0.32, fontSize: 13, fontFace: "Calibri", bold: true, color: arm.color, margin: 0 });
  s.addText(arm.subtitle, { x: x+0.15, y: 2.87, w: 5.8, h: 0.25, fontSize: 10, fontFace: "Calibri", color: C.muted, italic: true, margin: 0 });
  arm.items.forEach((item, j) => {
    s.addShape(pres.shapes.RECTANGLE, { x: x+0.2, y: 3.25+j*1.05, w: 0.28, h: 0.28, fill: { color: arm.color, transparency: 85 }, line: { type: "none" } });
    s.addText(`0${j+1}`, { x: x+0.21, y: 3.27+j*1.05, w: 0.26, h: 0.24, fontSize: 9, fontFace: "Georgia", bold: true, color: arm.color, align: "center", margin: 0 });
    s.addText(item, { x: x+0.55, y: 3.24+j*1.05, w: 5.4, h: 0.9, fontSize: 10, fontFace: "Calibri", color: C.body, margin: 0 });
  });
});
```

---

## Summary / Findings Slide

```js
const items = [
  { n: "①", color: C.cyan,   head: "Finding one headline.", body: "Supporting detail." },
  { n: "②", color: C.lime,   head: "Finding two headline.", body: "Supporting detail." },
  { n: "③", color: C.amber,  head: "Finding three headline.", body: "Supporting detail." },
  { n: "④", color: C.purple, head: "Finding four headline.", body: "Supporting detail." },
];

items.forEach((it, i) => {
  const y = 1.75 + i * 1.28;
  s.addShape(pres.shapes.RECTANGLE, { x: 0.4, y, w: 0.35, h: 0.35, fill: { color: it.color, transparency: 85 }, line: { type: "none" } });
  s.addText(it.n, { x: 0.42, y: y+0.03, w: 0.32, h: 0.3, fontSize: 12, fontFace: "Georgia", bold: true, color: it.color, align: "center", margin: 0 });
  s.addText([
    { text: it.head + "  ", options: { bold: true, color: C.text } },
    { text: it.body,         options: { color: C.body } }
  ], { x: 0.88, y: y+0.02, w: 12.0, h: 1.18, fontSize: 10, fontFace: "Calibri", margin: 0 });
});
```

---

## Slide Anatomy Checklist

Every slide should have all of these:

- [ ] `bg(s)` — background applied
- [ ] `eyebrow(s, "SECTION N · LABEL", 0.4, 0.3)` — section context
- [ ] `slideTitle(s, [...], 0.4, 0.55)` — two-tone title
- [ ] `rule(s, 0.4, 1.48)` — cyan rule below title
- [ ] `headline(s, "...", 0.4, 1.57, 12.5)` — one-sentence takeaway
- [ ] Content zone (`y = 2.25` onward)
- [ ] `footer(s, "Context string", "Slide N / TOTAL")`

---

## Spacing Reference

| Spacing | Value |
|---|---|
| Card internal padding | `x+0.15` from card edge |
| Card top-bar height | `0.04"` |
| Gap between two-column cards | `0.1"` |
| Gap between KPI cards | `2.5" per card` (5 fit in 12.5") |
| Section divider height | `0.24"` |
| Headline banner height | `0.55"` |
| Footer rule y | `7.22"` |
| Footer text y | `7.27"` |
| Left accent bar width | `0.04"` |

---

## Output

```js
pres.writeFile({ fileName: "./output.pptx" })
  .then(() => console.log("Done"))
  .catch(console.error);
```

---

## Quick-Start Skeleton

```js
const pptxgen = require("pptxgenjs");
const pres = new pptxgen();
pres.layout = "LAYOUT_WIDE";

// Paste C, makeShadow, bg, card, leftBar, eyebrow,
// slideTitle, rule, headline, kpi, pill, footer here

const s = pres.addSlide();
bg(s);
eyebrow(s, "Section 1 · Overview", 0.4, 0.3);
slideTitle(s, [
  { text: "Your Slide ",  options: { color: C.text } },
  { text: "Title Here",  options: { color: C.cyan } }
], 0.4, 0.55);
rule(s, 0.4, 1.48);
headline(s, "One sentence that captures exactly what this slide is saying.", 0.4, 1.57, 12.5);

// ... add your content here ...

footer(s, "Deck Title · Section Name", "Slide 1 / N");

pres.writeFile({ fileName: "./output.pptx" }).then(() => console.log("Done"));
```
