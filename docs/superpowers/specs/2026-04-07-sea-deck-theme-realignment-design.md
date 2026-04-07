# SEA Deck Theme Realignment — Design

**Date:** 2026-04-07
**Scope:** CSS-only theme adjustments to `sea-market-appendix.html` / `index.html`
**Reference:** [Sivugo/lanmea-presentation](https://github.com/Sivugo/lanmea-presentation) (cloned to `/tmp/lanmea-ref` during brainstorming)

## Goal

Bring the SEA deck's typography closer to the `Sivugo/lanmea-presentation` reference (lighter weights, airier editorial feel) while preserving yesterday's text-contrast gain and changing zero HTML structure or content.

## Background

During brainstorming, comparison of the two repos revealed:

- **Already shared:** identical `:root` token names, identical color hex values, same fonts (Gelasio + Inter from Google Fonts), shared chrome selectors (`.nav-bar`, `.slide-header`, `.accent-bar`, `.animate-in`, `.stagger-1..8`, `.deco-circle-dashed`). The SEA deck inherited the design system from this exact reference.
- **Drifted:** SEA's body text is currently heavier and darker than the reference, and a handful of stat numbers are inflated.
  - SEA `--text-secondary: 0.92` vs reference `0.65` (bumped 2026-04-06)
  - SEA `--text-muted: 0.70` vs reference `0.4` (bumped 2026-04-06)
  - SEA uses `font-weight: 600` 63 times and `400` 30 times. Reference uses `300` 44 times and `500` 42 times.
  - SEA has 4 × `font-size: 32px` and 2 × `font-size: 38px`. Reference's biggest recurring stat number is `26px`.
- **Wrinkle (out of scope here):** SEA cards and charts are built with per-slide custom selectors (`.s2v-econ-card`, `.s3v-stat-card`, `.s4v-country-card`, `.s5v-callout`, etc.) rather than the reference's reusable component classes (`.card`, `.pipeline`, `.belief-item`). Lifting reference component CSS will not auto-restyle SEA. We accept this and aim for cascade-friendly base rules instead.

## Principles

1. **CSS-only.** Never touch slide HTML markup or content text.
2. **Preserve yesterday's contrast gain** for `--text-secondary` and `--text-muted`, but tag them clearly so a one-line edit reverts to reference values.
3. **Prefer base rules over per-slide rewrites.** Smaller blast radius, easier to revert.
4. **Keep `index.html` and `sea-market-appendix.html` byte-identical** after the change, per project rule.

## Concrete Changes

All edits are inside the `<style>` block of `sea-market-appendix.html`.

### Change 1 — Annotate text-contrast tokens

Add a comment above `--text-secondary` and `--text-muted` showing the reference values, so reverting is a 30-second find-and-replace on two numbers. No value change.

```css
:root {
  /* SEA bumped these for readability on 2026-04-06.
     To match Sivugo/lanmea-presentation reference style,
     set --text-secondary: rgba(36, 24, 24, 0.65)
     and --text-muted:    rgba(36, 24, 24, 0.4) */
  --text-secondary: rgba(36, 24, 24, 0.92);
  --text-muted:     rgba(36, 24, 24, 0.70);
  ...
}
```

### Change 2 — Add base body weight rule

Right after the `html, body { ... }` block, add:

```css
.slide { font-weight: 300; }
```

This drops the default text weight inside every slide from `400` (Inter regular) to `300` (Inter light) — the reference's airy editorial weight. Selectors that explicitly set `500`, `600`, or `700` (slide titles, big numbers, labels, pill badges, h2 spans) still win via specificity. Body paragraphs and card descriptions inherit the lighter weight.

### Change 3 — Tone down oversized stat numbers

The SEA deck has these inflated sizes (counts from grep):
- `font-size: 32px` × 4
- `font-size: 38px` × 2

The reference uses `26px` as the recurring big-stat size, with one-off `36px` and `44px`. For each of the 6 instances:
1. Read the surrounding selector and what it labels
2. If it's a deliberate hero/cover stat (e.g. cover slide title or closing slide), leave it
3. Otherwise reduce to `28px` (matching the reference's `slide-header h2` and similar prominent type)

Expected outcome: 4–6 single-line edits. Worst case, all 6 stay because they're all hero stats — that's also a valid result.

### Change 4 — Sync both files

After all edits land in `sea-market-appendix.html`, copy the file to `index.html` so they stay byte-identical (per project rule). Verify with `diff sea-market-appendix.html index.html` (must exit 0).

## Out of Scope

- Per-slide `.s{N}v-*` selector audit (would be Option B, possible follow-up)
- Color value changes (already aligned)
- Layout, spacing, padding changes
- Any HTML/content changes
- The 63 explicit `font-weight: 600` instances — most are on labels/badges where the reference also uses 600
- Refactoring SEA's per-slide CSS into reference component classes (would change structure)

## Verification

After all edits:
1. `diff sea-market-appendix.html index.html` exits 0
2. Open `sea-market-appendix.html` in a browser, walk all 7 slides
3. Specifically check slide 6 for overflow (it was tightened aggressively yesterday — lighter weight may pull text up slightly but shouldn't cause overflow)
4. If anything looks off, narrow the change rather than abandoning the approach

## Easy Revert Plan

The whole change is designed to be easy to roll back, in three independent levers:

- **Revert text contrast to reference:** edit two numbers in `:root` (the comment block tells you which)
- **Revert body weight:** remove the single line `.slide { font-weight: 300; }`
- **Revert stat sizes:** `git diff` will show the 4–6 single-line edits; revert any individually

## Risks

- **Slide 6 overflow.** Already tight after yesterday's compression. Lighter weight typically *helps* fit more content, but worth confirming.
- **Any inline `font-weight: 400` on slide content** would still apply at 400 (more specific than `.slide`). Acceptable — these are intentional.
- **Cover and closing slides** use larger type; if their stats are in the 32px/38px set, they may need to stay where they are. Decided per-instance during Change 3.
