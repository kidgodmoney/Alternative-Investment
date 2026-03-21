# Plan: redesign-typography

> **Phase**: Plan
> **Created**: 2026-03-21
> **Feature**: Typography, UX polish, and mobile viewport fixes
> **Parent**: redesign (archived 2026-03-21, 100% match)

---

## Background

The first redesign cycle (color palette + grain texture) achieved 100% match rate and is archived. This plan addresses the 5 backlog items identified during that cycle's gap analysis.

---

## Goals

### 1. Reduce all-caps label overuse
**Problem**: 24 instances of `text-transform: uppercase` with heavy letter-spacing (≥0.1em) across both files. Every label, tag, badge, and navigation item screams in capitals — the design lacks visual hierarchy variation.

**Target**: Reduce to uppercase only where it genuinely serves branding (e.g., the masthead tagline, section-level labels). Convert body-level labels to sentence case.

**Affected elements**: `.field-tag`, `.paper-cat`, `.hero-label`, `.issue-info`, `.insight-desc`, `.loading-text`, `.footer-link`, badge text in modals.

---

### 2. Shorten and rewrite the magazine tagline
**Problem**: Current taglines read like feature specifications:
- `index.html`: `"Alternative Investment Research — Private Equity · Real Estate · Infrastructure · Hedge Fund · Venture Capital"`
- `news.html`: `"Alternative Investment News — RSS Live Feed · Private Equity · Real Estate · Infrastructure · Hedge Fund · VC"`

This is a list, not a tagline. It communicates nothing about voice or value.

**Target**: One concise editorial line per page. Examples:
- Research page: `"Institutional research at the frontier of private markets"`
- News page: `"Live intelligence across private equity, real assets, and venture"`

---

### 3. Skeleton loader instead of circular spinner
**Problem**: `.loading-spinner` is a basic CSS border-spinner (`border-radius: 50%; animation: spin`). Spinners communicate waiting without context — skeleton loaders communicate the shape of what's coming.

**Target**: Replace spinner with a skeleton that matches the hero + paper grid layout shape. Static skeleton cards using CSS `background: linear-gradient` shimmer animation.

---

### 4. Button `:active` press state
**Problem**: No `:active` pseudo-class defined on `.btn` or any interactive element. Buttons feel weightless — no physical feedback.

**Target**: Add `transform: scale(0.97) translateY(1px)` and slight opacity reduction on `:active` for `.btn`, `.page-nav a`, `.field-tag`, `.paper-card`.

---

### 5. Mobile viewport fix (`100dvh`)
**Problem**: `min-height: 100vh` on `body` causes layout jumping on iOS Safari, which adds/removes browser chrome as the user scrolls.

**Target**: Replace with `min-height: 100dvh` (dynamic viewport height) on body in both files.

---

## Scope

**Files**: `index.html`, `news.html`
**Type**: CSS + HTML content (no JS logic changes)
**Risk**: Low — all changes are additive CSS rules or text content edits

---

## Implementation Order

1. `min-height: 100dvh` — 2-line change, zero visual risk
2. Button `:active` states — additive CSS only
3. Tagline rewrite — HTML text content only
4. All-caps reduction — targeted CSS selector changes (audit first)
5. Skeleton loader — replaces `.loading-screen` HTML+CSS block

---

## Success Criteria

- `text-transform: uppercase` instances reduced by at least 60% (from 24 → under 10)
- Taglines are 1 sentence each, no bullet lists
- Loading state shows skeleton cards matching grid shape
- All `.btn` elements have visible press feedback on `:active`
- Body uses `min-height: 100dvh` on both pages
- No regressions in existing layout or functionality

---

## Next Step

`/pdca design redesign-typography`
