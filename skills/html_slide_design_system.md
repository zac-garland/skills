# Presentation Design System
## Single-File HTML Slide Deck — Reference Guide

This document captures every design and implementation decision made in `thesis_defense.html` so the same aesthetic and structure can be reproduced quickly for future decks.

---

## File Choice & Format

**Format:** Single `.html` file  
**Dependencies:** Google Fonts (CDN), no JS libraries, no build step  
**Compatibility:** Any modern browser; open via `file://` in Chrome/Firefox  
**Rendering:** Firefox handles local `file://` without same-origin issues; Chrome requires `--allow-file-access-from-files` flag for iframes

**Why single HTML over other formats:**
- Zero setup — send one file, open anywhere
- Full CSS animation and JS interactivity
- No Keynote/PowerPoint rendering inconsistencies
- Version-controllable plaintext
- Can embed iframes, interactive charts, code blocks natively

---

## Color System

All colors defined as CSS custom properties on `:root`. Import this block into any new deck.

```css
:root {
  /* Backgrounds */
  --bg:      #07080D;   /* near-black — slide background */
  --surface: #0F1118;   /* slightly lighter — used sparingly */
  --card:    #141720;   /* card background */
  --card2:   #1A1E2B;   /* table header / secondary card */
  --border:  rgba(255,255,255,0.07); /* subtle card borders */

  /* Accent Palette — each has a semantic role */
  --cyan:   #00E5FF;   /* primary accent — titles, active elements */
  --coral:  #FF4F6D;   /* alert / challenges / criticism */
  --lime:   #B8FF57;   /* confirmation / positive results */
  --amber:  #FFB830;   /* secondary emphasis / warnings */
  --purple: #9B6DFF;   /* tertiary / code / pipeline stages */
  --slate:  #78909C;   /* muted / supporting text */

  /* Text */
  --text:   #F0F2F8;   /* primary text */
  --muted:  #6B7A99;   /* secondary / labels */
}
```

**Color assignment rules:**
- Use `--cyan` for the primary slide title emphasis (`<em>` tags)
- Assign one accent per major section: H1 → amber, challenges → coral, confirmation → lime
- Cards get a 2px top border in their section's accent color
- Pill badges use 10% opacity background of their accent color

---

## Typography

```css
/* Import */
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@400;500&family=DM+Sans:wght@300;400;500&display=swap');

--font-h: 'Syne', sans-serif;    /* headings, titles, numbers */
--font-b: 'DM Sans', sans-serif; /* body text, cards */
--font-m: 'DM Mono', monospace;  /* code, numbers, labels, eyebrows */
```

**Type scale:**
| Element | Font | Size | Weight |
|---|---|---|---|
| Slide title | Syne | 2.6rem | 800 |
| Title slide hero | Syne | 3.6rem | 800 |
| KPI values | Syne | 2.8rem | 800 |
| Section title in card | Syne | 0.85–0.9rem | 700 |
| Eyebrow label | DM Mono | 0.68rem | 400 |
| Body text | DM Sans | 0.78–0.82rem | 400 |
| Table cells | DM Sans | 0.78rem | 400 |
| Code blocks | DM Mono | 0.72rem | 400 |
| Slide footer | DM Mono | 0.62rem | 400 |

**Title pattern:** every slide title uses `<em>` with `font-style:normal; color:var(--cyan)` on the second part:
```html
<div class="slide-title">Portfolio Construction <em>Methodology</em></div>
```

**Eyebrow pattern:**
```html
<div class="eyebrow">Section 3 · Signal Construction</div>
```
Eyebrow → Rule (2px cyan bar) → Title → Content

---

## Layout System

### Slide structure (every slide follows this):

```html
<div class="slide" id="sN">
  <div class="eyebrow">Section X · Label</div>
  <div class="slide-title">Main Title <em>Accented Part</em></div>
  <div class="rule"></div>

  <!-- CONTENT AREA — flex:1 to fill remaining space -->
  <div class="g2" style="flex:1;">
    ...
  </div>

  <div class="slide-footer">
    <span>Section Name</span>
    <span>Slide N / Total</span>
  </div>
</div>
```

### Grid utilities:

```css
.g2  { display:grid; grid-template-columns: 1fr 1fr; gap:1rem; }
.g3  { display:grid; grid-template-columns: 1fr 1fr 1fr; gap:1rem; }
.g4  { display:grid; grid-template-columns: repeat(4,1fr); gap:1rem; }
.g5  { display:grid; grid-template-columns: repeat(5,1fr); gap:0.85rem; }
.g32 { display:grid; grid-template-columns: 1.1fr 0.9fr; gap:1.1rem; }
.col { display:flex; flex-direction:column; gap:0.9rem; }
```

**When to use which:**
- `g2`: two-column content split (most common)
- `g3`: three equal hypothesis cards
- `g4`: KPI stat row (4 numbers)
- `g5`: pipeline steps (5 equal columns)
- `g32`: code + examples side by side (slightly wider left)
- `col`: stacked cards in a column

---

## Component Library

### Card
```html
<div class="card">
  <div class="card-title">Title Text</div>
  <div class="card-body">Body text here.</div>
</div>

<!-- With top accent border -->
<div class="card" style="border-top:2px solid var(--cyan);">
```

### KPI Block
```html
<div class="card card-accent" style="text-align:center;padding:1rem;">
  <div class="kpi-val">181,305</div>        <!-- 2.8rem Syne 800 cyan -->
  <div class="kpi-label">Earnings Calls</div>
  <div class="kpi-sub">Apr 2010 – Dec 2025</div>
</div>
```

### Step Card (pipeline stages)
```html
<div class="step">
  <div style="position:absolute;top:0;left:0;right:0;height:2px;background:var(--cyan);..."></div>
  <div class="step-num" style="color:var(--cyan);">01</div>
  <div class="step-title">Stage Name</div>
  <div class="step-body">Description text.</div>
</div>
```

### Hypothesis Card
```html
<div class="hyp" style="border-top:2px solid var(--cyan);">
  <div class="hyp-label" style="color:var(--cyan);">H1 · Label</div>
  <div class="hyp-title">Short Title</div>
  <div class="hyp-body">Explanation...</div>
</div>
```

### Gate Row (checklist items)
```html
<div class="gate">
  <div class="gate-icon" style="background:rgba(0,229,255,0.15);color:var(--cyan);">✓</div>
  <div>
    <div class="gate-title">Condition Name</div>
    <div class="gate-body">Explanation of why this gate exists.</div>
  </div>
</div>
```

### Formula Display
```html
<div class="formula">
  AdjNov = novelty × (⅓·section + ⅓·attention + ⅓·depth) × penalty(f)
</div>
```
Monospace, lime text, dark background, cyan left border.

### Pill Badge
```html
<span class="pill pill-cyan">Label</span>   <!-- cyan -->
<span class="pill pill-amber">Label</span>  <!-- amber -->
<span class="pill pill-coral">Label</span>  <!-- coral -->
<span class="pill pill-lime">Label</span>   <!-- lime -->
```

### Code Block
```html
<div class="code-block">
  <span class="code-k">keyword</span> = (
    <span class="code-s">"string text"</span>
    <span class="code-c"># comment</span>
    <span class="code-v">variable</span>
  )
</div>
```
Color tokens: `.code-k` purple, `.code-s` lime, `.code-c` slate, `.code-v` amber.

### Mechanism Banner (highlighted callout)
```html
<div class="mechanism">
  <strong>Label:</strong> Body text that explains the core mechanism or key point.
</div>
```
Subtle cyan tint, used for RQ statements, key findings, key caveats.

### Data Table
```html
<table class="tbl">
  <tr><th>Variable</th><th>N</th><th>Mean</th></tr>
  <tr><td>ret_t=0</td><td>185,622</td><td>0.0004</td></tr>
</table>
```
Dark header (`var(--card2)`), cyan header text, alternating row tint.

---

## Slide Transitions

```css
.slide {
  opacity: 0;
  pointer-events: none;
  transform: translateX(40px);
  transition: opacity .45s cubic-bezier(.4,0,.2,1),
              transform .45s cubic-bezier(.4,0,.2,1);
}
.slide.active {
  opacity: 1;
  pointer-events: auto;
  transform: translateX(0);
}
.slide.exit {
  opacity: 0;
  transform: translateX(-40px);
  transition: opacity .3s ease, transform .3s ease;
}
```

Slides enter from the right (positive translateX → 0) and exit to the left (0 → negative translateX). The exit class is added momentarily and removed after 350ms.

---

## Navigation System

### JavaScript pattern (copy this):
```javascript
const slides = document.querySelectorAll('.slide');
let cur = 0;

function go(n) {
  if (n === cur) return;
  slides[cur].classList.remove('active');
  slides[cur].classList.add('exit');
  setTimeout(() => slides[Math.min(cur,n)].classList.remove('exit'), 350);
  cur = ((n % slides.length) + slides.length) % slides.length;
  slides[cur].classList.add('active');
  // update dots and counter
}

// Keyboard
document.addEventListener('keydown', e => {
  if (e.key === 'ArrowRight' || e.key === ' ') go(cur + 1);
  if (e.key === 'ArrowLeft') go(cur - 1);
  if (e.key === 'f') document.documentElement.requestFullscreen?.();
  const n = parseInt(e.key);
  if (n >= 1 && n <= slides.length) go(n - 1);
});

// Touch swipe
let tx = 0;
document.addEventListener('touchstart', e => { tx = e.touches[0].clientX; });
document.addEventListener('touchend', e => {
  const dx = e.changedTouches[0].clientX - tx;
  if (Math.abs(dx) > 50) go(dx < 0 ? cur + 1 : cur - 1);
});
```

### Navigation dot pattern:
```html
<div class="nav">
  <button class="nav-btn" id="prev">‹</button>
  <div id="dots"></div>
  <div class="nav-counter" id="counter">1 / N</div>
  <button class="nav-btn" id="next">›</button>
</div>
```
Dots auto-generated in JS. Active dot: wider (20px), cyan, box-shadow glow. Inactive: 6px circle, muted.

---

## Slide ID Convention

Slides are numbered `s0` through `sN`. First slide is `s0` (title), last is the summary.

Per-slide background overrides go on the ID selector:
```css
#s0  { background: radial-gradient(ellipse at 70% 30%, rgba(0,229,255,0.06), transparent 60%), var(--bg); }
#s11 { background: radial-gradient(ellipse at 30% 70%, rgba(184,255,87,0.05), transparent 55%), var(--bg); }
```

Title and summary slides get subtle radial gradient tints. All other slides use flat `var(--bg)`.

---

## Padding & Spacing

| Context | Value |
|---|---|
| Slide padding | `3rem 4rem` |
| Card padding | `1.25rem 1.4rem` |
| Grid gap (standard) | `1rem` |
| Grid gap (tight) | `0.85rem` |
| Rule margin | `1rem 0 1.4rem` |
| Footer padding-top | `1rem` |
| Border radius (card) | `12px` |
| Border radius (small card) | `10px` |
| Border radius (pill) | `20px` |

---

## Eyebrow → Rule → Title Pattern

Every content slide follows this exact header sequence:

```
SECTION N · LABEL        ← DM Mono, 0.68rem, cyan, uppercase, 0.18em spacing
Main Title Accented Part  ← Syne, 2.6rem, 800 — plain white + cyan <em>
────                      ← 3rem wide, 2px, cyan, 1rem margin above/below
```

The rule (`.rule`) is a landmark — it visually separates the slide header from content and creates consistent vertical rhythm across all slides.

---

## Defense-Specific Conventions

When building a **defense deck** (as opposed to a results/pitch deck), follow these structural rules:

1. **State criticism before defense** — dedicate a full slide to the opposing view, stated fairly
2. **Separate signal from complexity** — show the raw signal first, add pipeline layers second
3. **Every claim needs a test** — if you assert X, name the robustness check that would falsify it
4. **Placeholder slides for Q&A** — anticipate 3–5 likely questions, draft one-paragraph answers
5. **Timeline slide is mandatory** — if results are pending, explain the constraint, the plan, and what you need
6. **Narrow the hypothesis** — if under pressure, defend H1 only. H2/H3 are extensions.

**Card accent color by content type:**
- `--cyan`: primary methodology / your position
- `--amber`: hypothesis / what you expect to find
- `--coral`: criticism / challenges / what could go wrong
- `--lime`: confirmation / what success looks like
- `--slate`: pending / future work / caveats

---

## Reuse Checklist

When starting a new deck from this template:

- [ ] Update `:root` color semantics if the topic changes
- [ ] Replace font imports if a different aesthetic is needed
- [ ] Assign one accent per major section (not per slide)
- [ ] Write eyebrow labels as `Section N · Topic`
- [ ] Add `style="flex:1;"` to the main content div so it fills the slide
- [ ] Set slide footer: left = topic, right = `Slide N / Total`
- [ ] Keep slides under ~7 content elements — if a slide needs more, split it
- [ ] Add `id="sN"` to each slide for background overrides
- [ ] Total slide count: 10–14 is the sweet spot for a 20-minute defense
