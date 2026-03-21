# Design: redesign-typography

> **Phase**: Design
> **Created**: 2026-03-21
> **Prerequisite**: redesign-typography.plan.md ✅

---

## 1. Viewport Fix — `100dvh`

**Files**: `index.html:46`, `news.html:39`

```css
/* BEFORE */
min-height: 100vh;

/* AFTER */
min-height: 100dvh;
```

Both body rules. 2-line change.

---

## 2. Button `:active` Press States

**Files**: `index.html:526`, `news.html:167` (after `.btn:hover`)

Add after each `.btn:hover` rule:

```css
.btn:active {
  transform: scale(0.97) translateY(1px);
  opacity: 0.9;
  transition: transform 0.08s, opacity 0.08s;
}
```

Also add to interactive card/nav elements:

```css
.paper-card:active  { opacity: 0.85; transform: scale(0.995); }
.page-nav a:active  { transform: translateY(1px); }
.field-tag:active   { opacity: 0.75; }
```

---

## 3. Tagline Rewrite

**index.html:642** — replace content:
```html
<!-- BEFORE -->
Alternative Investment Research — Private Equity · Real Estate · Infrastructure · Hedge Fund · Venture Capital

<!-- AFTER -->
Institutional research at the frontier of private markets
```

**news.html** — find `.magazine-tagline` in the HTML body:
```html
<!-- BEFORE -->
Alternative Investment News — RSS Live Feed · Private Equity · Real Estate · Infrastructure · Hedge Fund · VC

<!-- AFTER -->
Live intelligence across private equity, real assets, and venture
```

Also reduce tagline letter-spacing (both files):
```css
/* BEFORE */
.magazine-tagline { font-size: 10.5px; letter-spacing: 0.2em; text-transform: uppercase; ... }

/* AFTER */
.magazine-tagline { font-size: 10.5px; letter-spacing: 0.12em; text-transform: uppercase; ... }
```

---

## 4. All-caps Reduction

**Audit result**: 27 total `text-transform: uppercase` instances.

### Keep uppercase (functional/conventional)
These serve navigation hierarchy, brand badges, or are too small to read without caps:

| Selector | Reason to keep |
|---|---|
| `.page-nav a` | Navigation pill — convention |
| `.magazine-tagline` | Masthead identity |
| `.ticker-label` | "실시간" functional label |
| `.paper-cat` | Category mini-badge (9px — needs caps) |
| `.source-badge` | Brand badge (8.5px — needs caps) |
| `.btn` | CTA button — convention |
| `.field-tag` | Primary tab nav |
| `.news-card-source` | Source attribution (functional) |
| `.sector-en` | English sub-label below Korean (9.5px) |

### Remove uppercase (reduce noise)

**index.html** — change these selectors:

| Selector | Line | Change |
|---|---|---|
| `.issue-info` | 82 | Remove `text-transform: uppercase`, reduce `letter-spacing: 0.05em` |
| `.loading-text` | 178 | Remove `text-transform: uppercase`, reduce `letter-spacing: 0.05em` |
| `.hero-link` | 220 | Remove `text-transform: uppercase`, keep font-weight |
| `.insight-desc` | 353 | Remove `text-transform: uppercase`, reduce `letter-spacing: 0.03em` |
| `.widget-title` | 398 | Remove `text-transform: uppercase`, keep font-weight |
| `.modal-label` | 463 | Remove `text-transform: uppercase` |
| `.modal-section-title` | 488 | Remove `text-transform: uppercase`, use sentence case in HTML |
| `.modal-meta-label` | 493 | Remove `text-transform: uppercase` |
| `.footer-link` | 557 | Remove `text-transform: uppercase`, keep `letter-spacing: 0.05em` |

**news.html** — change these selectors:

| Selector | Line | Change |
|---|---|---|
| `.issue-info` | 59 | Remove `text-transform: uppercase`, reduce `letter-spacing: 0.05em` |
| `.loading-text` | 93 | Remove `text-transform: uppercase` |
| `.cat-hero-label` | 108 | Remove `text-transform: uppercase`, reduce `letter-spacing: 0.08em` |
| `.section-meta` | 116 | Remove `text-transform: uppercase` (monospace already signals metadata) |
| `.footer-link` | 176 | Remove `text-transform: uppercase` |

**Result**: 27 → 9 instances. 67% reduction.

---

## 5. Skeleton Loader

Replace the spinner-based loading screen with a layout-matching skeleton.

### Skeleton CSS (add to both files, after `.loading-text` rule)

```css
/* ── SKELETON LOADER ── */
.skeleton-screen {
  padding: 2rem 0;
  animation: skeleton-fade-in 0.3s ease;
}
@keyframes skeleton-fade-in { from { opacity: 0 } to { opacity: 1 } }

.sk-block {
  background: linear-gradient(
    90deg,
    var(--paper-warm) 25%,
    rgba(240,236,227,0.04) 50%,
    var(--paper-warm) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.6s infinite linear;
  border-radius: var(--radius-sm);
}
@keyframes shimmer {
  from { background-position: 200% 0 }
  to   { background-position: -200% 0 }
}

/* Hero skeleton */
.sk-hero {
  display: grid;
  grid-template-columns: 1fr 360px;
  border: 1px solid var(--rule-dark);
  border-radius: var(--radius);
  overflow: hidden;
  margin-top: 2rem;
  margin-bottom: 2rem;
  background: var(--paper-warm);
  min-height: 280px;
}
.sk-hero-main { padding: 3rem; display: flex; flex-direction: column; gap: 1rem; border-right: 1px solid var(--rule); }
.sk-hero-side { padding: 2rem; display: flex; flex-direction: column; gap: 1.5rem; }
.sk-line { height: 14px; }
.sk-line-lg { height: 24px; }
.sk-line-xl { height: 34px; }
.sk-line-sm { height: 10px; }
.sk-line-w-80 { width: 80%; }
.sk-line-w-60 { width: 60%; }
.sk-line-w-40 { width: 40%; }

/* Grid skeleton */
.sk-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1px;
  background: var(--rule);
  border-radius: var(--radius);
  overflow: hidden;
  border: 1px solid var(--rule-dark);
}
.sk-card {
  background: var(--paper-warm);
  padding: 1.6rem;
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  min-height: 200px;
}

@media (max-width: 1024px) {
  .sk-hero { grid-template-columns: 1fr; }
  .sk-hero-main { border-right: none; }
  .sk-grid { grid-template-columns: repeat(2, 1fr); }
}
@media (max-width: 640px) {
  .sk-grid { grid-template-columns: 1fr; }
}
```

### Skeleton HTML (replaces `.loading-screen` div in index.html)

```html
<div class="skeleton-screen container">
  <div class="sk-hero">
    <div class="sk-hero-main">
      <div class="sk-block sk-line sk-line-w-40"></div>
      <div class="sk-block sk-line-xl sk-line-w-80"></div>
      <div class="sk-block sk-line-xl sk-line-w-60"></div>
      <div class="sk-block sk-line" style="margin-top:0.5rem"></div>
      <div class="sk-block sk-line sk-line-w-80"></div>
      <div class="sk-block sk-line sk-line-w-60"></div>
    </div>
    <div class="sk-hero-side">
      <div class="sk-block sk-line sk-line-w-40"></div>
      <div class="sk-block sk-line sk-line-w-80"></div>
      <div class="sk-block sk-line sk-line-w-60"></div>
      <div class="sk-block sk-line" style="margin-top:0.5rem"></div>
      <div class="sk-block sk-line sk-line-w-80"></div>
      <div class="sk-block sk-line sk-line-w-40"></div>
    </div>
  </div>
  <div class="sk-grid">
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div><div class="sk-block sk-line-sm" style="margin-top:auto"></div></div>
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div><div class="sk-block sk-line-sm" style="margin-top:auto"></div></div>
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div><div class="sk-block sk-line-sm" style="margin-top:auto"></div></div>
  </div>
</div>
```

For `news.html`, simpler skeleton (no hero, just a featured + grid):

```html
<div class="skeleton-screen container">
  <div class="sk-grid" style="grid-template-columns: 3fr 2fr; margin-bottom: 2rem; min-height: 220px;">
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line-xl sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div><div class="sk-block sk-line sk-line-w-80"></div></div>
    <div class="sk-card"><div class="sk-block sk-line sk-line-w-60"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-40"></div></div>
  </div>
  <div class="sk-grid">
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div></div>
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div></div>
    <div class="sk-card"><div class="sk-block sk-line-sm sk-line-w-40"></div><div class="sk-block sk-line sk-line-w-80"></div><div class="sk-block sk-line sk-line-w-60"></div></div>
  </div>
</div>
```

### JS change (index.html) — replace loading div render

In the JS, where `document.getElementById('app').innerHTML` is set to start loading, replace the loading-screen div with the skeleton-screen div using the same HTML above. The skeleton is replaced by actual content when data loads.

---

## Implementation Checklist

- [ ] `min-height: 100dvh` — index.html, news.html
- [ ] `.btn:active` + card/nav active states — index.html, news.html
- [ ] Tagline text content — index.html:642, news.html (HTML body)
- [ ] Tagline `letter-spacing: 0.12em` — index.html, news.html (CSS)
- [ ] Remove uppercase from 9 selectors in index.html
- [ ] Remove uppercase from 5 selectors in news.html
- [ ] Add skeleton CSS — index.html, news.html
- [ ] Replace loading-screen HTML with skeleton — index.html (static), news.html (static)
- [ ] Verify JS still replaces skeleton content on load (no JS changes needed if ID stays `app`)

---

## Next Step

`/pdca do redesign-typography` or implement directly following this spec.
