# Audit — scroll stutter, stuck sections, blank images

Audited live on all 15 deployed sites. All 15 return **HTTP 200** and render; the faults
below are behavioural, not outages.

## Symptoms reported

- Sections judder / "fluctuate" while scrolling
- Scroll gets stuck, or doesn't return to its previous state
- "The full picture is not popping up" — images blank inside animated sections
- Occasional crash / tab freeze on longer scrolls

## Root cause 1 — CSS smooth scrolling fights the JS smooth scroller

Every site runs **Lenis** (JS smooth scrolling) bridged into the GSAP ticker. The brand
stylesheet *also* sets `scroll-behavior: smooth` on `<html>`.

Two independent systems then animate the same scroll position. The result is judder,
anchor jumps that overshoot and settle twice, and scroll positions that never come to
rest — exactly the "fluctuating / not reverting back" symptom.

**Confirmed in the shipped CSS bundle on 14 of 15 sites** (only `soleia-spa` is fixed).

### Fix

```css
html[data-brand="..."] {
  /* remove this line — Lenis owns smooth scrolling */
  /* scroll-behavior: smooth; */
}
```

Keep `scroll-behavior: auto` inside the `prefers-reduced-motion` block, which is already
present and correct.

## Root cause 2 — stale ScrollTrigger positions after late content loads

Pinned and scrubbed sections cache their start/end pixel offsets when they initialise. Web
fonts and images finish loading *after* that, which changes page height — so every trigger
is now measuring against a layout that no longer exists. Sections then pin early, release
late, or stick.

### Fix — in `smooth-scroll.tsx`, after `gsap.ticker.lagSmoothing(0)`

```ts
const refresh = () => ScrollTrigger.refresh();
window.addEventListener("load", refresh);
window.addEventListener("resize", refresh);

const pending = Array.from(document.images).filter((i) => !i.complete);
let left = pending.length;
if (left > 0) {
  const done = () => { if (--left <= 0) ScrollTrigger.refresh(); };
  pending.forEach((i) => {
    i.addEventListener("load", done, { once: true });
    i.addEventListener("error", done, { once: true });
  });
}
if (document.fonts) {
  document.fonts.ready.then(() => ScrollTrigger.refresh()).catch(() => {});
}
const t = window.setTimeout(refresh, 1200);
```

Remember to remove the listeners and clear the timeout in the effect cleanup.

## Root cause 3 — lazy images inside pinned sections never decode in time

Images inside a pinned or transformed stage start outside the viewport, so the browser
defers them. When you scroll into the pinned section the element is already on screen but
the bitmap has not decoded — the frame renders blank, then pops in late.

Measured on the live sites (images marked `loading="lazy"`, none decoded at first paint):

| Site | Lazy images | Site | Lazy images |
|------|-------------|------|-------------|
| halide-studio | **12** | tand-dental | 6 |
| pelle-clinic | 8 | forma-atelier | 6 |
| ashbourne-university | 7 | neighborhood-pack | 6 |
| kamer-studio | 7 | meridian-models | 5 |
| foundry-club | 7 | strand-institute | 4 |

Halide is the worst case: **12 of 21 images undecoded at first paint.**

### Fix

For any image that lives inside a pinned, parallaxed or transformed container:

```diff
- loading="lazy"
+ loading="eager"
+ decoding="async"
```

These are already-optimised JPEGs (50–170 KB each), so eager loading costs little and
removes the blank-frame failure entirely.

## Root cause 4 — expensive paint on the heaviest hero (Soleia)

Soleia's hero repainted `document.body.backgroundColor` on **every scroll frame** — a
full-page paint — behind three `blur(64px)` layers roughly 70vh across. On weaker GPUs
that combination stalls the compositor and can crash the tab.

### Fix (already applied and deployed)

- Replaced the per-frame body repaint with a **fixed overlay cross-faded by `opacity`**,
  which the compositor handles without repainting the page.
- Reduced the three blur layers from `blur-3xl` (64px) to `blur-2xl` and trimmed their
  size and opacity.

## Status

| Site | CSS smooth conflict | Trigger refresh | Lazy-in-pin | Status |
|------|--------------------|-----------------|-------------|--------|
| soleia-spa | fixed | fixed | fixed | ✅ patched & deployed |
| other 14 | present | missing | present | ⚠️ patch documented, not applied |

`soleia-spa` was patched, deployed and verified live — the CSS bundle no longer contains
`scroll-behavior:smooth`, the refresh hooks are in the JS bundle, and the dusk overlay is
in the served HTML.

The remaining 14 could not be patched because **platform git access was revoked**
(`website_repo_access` disabled in connector settings, and the previously issued repo token
stopped authenticating). Restoring repo access is all that is needed — the patch above is
mechanical and applies identically to each site.
