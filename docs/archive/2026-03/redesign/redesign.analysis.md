# Gap Analysis: Visual Redesign

> **Date**: 2026-03-21
> **Scope**: index.html, news.html — CSS variables, colors, layout, texture
> **Final Match Rate: 100%** (after iteration)

---

## Summary

The redesign eliminated the "AI product" visual fingerprint (blue/teal/purple multi-gradient palette) and replaced it with a premium finance publication aesthetic — single amber-gold accent (#c9a84c), warm dark background, cream ink, and grain texture.

---

## ✅ Implemented (9/9 goals)

1. **Multi-accent gradient removed** — all `rgba(79,142,247,...)` blue values replaced with amber equivalents across all CSS hover/border states in both files
2. **Single amber-gold accent (#c9a84c)** — `--accent`, `--navy`, `--teal`, `--gold` all unified to `#c9a84c`
3. **Warm dark background (#0d0d0b)** — replaces cold blue-tinted `#07090f`
4. **Warm cream ink (#f0ece3)** — replaces cold slate `#f1f5f9`
5. **Masthead bar: solid amber** — replaces rainbow gradient (blue→teal→purple)
6. **Gradient text eliminated** — magazine name, insight numbers, footer logo now use solid amber color
7. **Grain/noise texture overlay** — `body::after` with SVG feTurbulence noise, `mix-blend-mode: overlay`
8. **Warm amber radial glows** — body background uses `rgba(201,168,76,...)` amber gradients
9. **Featured first paper card** — `.paper-grid .paper-card:first-child { grid-column: span 2 }` with amber left border

### Category palette (JS metadata) updated
| Category | Before | After |
|---|---|---|
| 사모펀드 (PE) | `#d97706` (bright amber) | `#c9a84c` (accent gold) |
| 부동산 (RE) | `#10b981` (saturated green) | `#7aaa8a` (muted sage) |
| 인프라 (INF) | `#4f8ef7` (bright blue) | `#7a96b0` (desaturated slate) |
| 헤지펀드 (HF) | `#a855f7` (bright purple) | `#9e84a0` (muted mauve) |
| 벤처캐피탈 (VC) | `#ef4444` (bright red) | `#b07878` (muted terracotta) |

---

## Backlog (acknowledged, not in scope for this iteration)

| Issue | Priority |
|---|---|
| All-caps labels with heavy letter-spacing (24 instances) | Medium |
| Magazine tagline is a feature spec list, not editorial copy | Low |
| Loading spinner → skeleton loader | Low |
| No `:active` pressed state on buttons (`scale(0.98)`) | Low |
| `min-height: 100vh` → `min-height: 100dvh` | Low |
