# Research: Homepage Hero Section

**Feature**: `001-homepage-hero`
**Date**: 2026-02-19
**Status**: Complete — all NEEDS CLARIFICATION resolved

---

## Decision 1: Animation Technology

**Decision**: CSS keyframe animations on SVG elements. No third-party animation library. Native
Web Animations API (`element.animate()`) reserved for any one-time mount-time sequencing only.

**Rationale**:
- CSS keyframe animations run on the compositor thread for `transform` and `opacity` — the
  two properties browsers promote off the main thread. `stroke-dashoffset` via `@keyframes`
  is paint-thread only (not compositor), but does not block the main thread. TBT contribution
  for continuous looping animation: effectively zero.
- `requestAnimationFrame` executes JavaScript on the main thread every ~16ms. Under CPU
  throttling (Lighthouse's 4G measurement condition), any rAF callback risks accumulating into
  a long task (> 50ms) and adding directly to TBT.
- Svelte `tweened`/`spring` stores interpolate on the main thread via Svelte's scheduler,
  adding Svelte runtime overhead on top of rAF cost — worst option for TBT.
- Motion One (3.8kb gzipped) wraps the native Web Animations API and is runtime-efficient, but
  adding any dependency requires a documented performance impact assessment per Principle III.
  Native WAAPI covers the use case without the dependency. Rejected per Principle V.

**Alternatives considered**:
- Canvas 2D + rAF: higher expressiveness, zero DOM serialization, but all rendering on the main
  thread; no path to accessible `prefers-reduced-motion` static frame without extra logic.
  Rejected.
- WebGL / Three.js: far exceeds portfolio scope (Principle V). Rejected.
- Lottie: JSON + JS player; adds a dependency requiring documented performance impact. Rejected
  per Principle V.

---

## Decision 2: Svelte Island Loading Strategy

**Decision**: `client:visible` with a pre-sized container in the `.astro` parent, using
`aspect-ratio` to reserve layout space before island hydration.

**Rationale**:
- `client:visible` defers Svelte hydration until the element enters the viewport. For a hero
  (above the fold), hydration occurs quickly after first paint — but the name and title are
  already rendered from the initial HTML payload, satisfying FR-007 without any wait.
- The container's dimensions are declared with `aspect-ratio` in the Astro component's scoped
  CSS, reserving layout space before hydration. This eliminates CLS when the animation island
  initializes (FR-008, SC-003).
- `client:load` was rejected: hydrates synchronously in the critical path, eliminating the
  benefit of the islands architecture.
- `client:idle` was rejected: waits for main thread idle time, risking a visible empty
  container on slow devices. Pre-sized `client:visible` is the cleaner tradeoff.

---

## Decision 3: `prefers-reduced-motion` Pattern

**Decision**: CSS `@media (prefers-reduced-motion: reduce)` in the Svelte component's
`<style>` block as the primary mechanism. Single DOM structure in both states (no conditional
rendering). Optional `onMount` Svelte check only if any JS animation calls are added.

**Rationale**:
- CSS media query is zero-JS, enforced before Svelte hydration completes, and automatically
  responds to mid-session system preference changes without event listeners.
- Conditional Svelte rendering (different DOM for animated vs. static states) risks CLS if
  the two structures have different layout dimensions. A single DOM structure with CSS toggling
  animation properties eliminates this risk entirely.
- The `@media` block is co-located in the Svelte component's `<style>` — cleaner isolation
  than a global stylesheet.

---

## Decision 4: SVG Composition

**Decision**: Three-node radial graph in equilateral triangle formation, with curved connecting
edges, a central convergence node, and orbital particles traversing edges via CSS `offset-path`.

**Composition layers** (single `<svg viewBox="0 0 400 400">`):

1. **Static backbone** (SSR-visible before island hydration):
   Three `<circle>` nodes arranged in an equilateral triangle centered in the viewBox. Rendered
   in the initial HTML when the SVG is inlined; animation is additive via CSS.

2. **Connecting edges**:
   Three `<path>` elements with quadratic bezier curves (Q toward centroid) between each node
   pair. In animated state: `stroke-dasharray` + `stroke-dashoffset` `@keyframes` produce a
   traveling signal. In reduced-motion state: fully rendered, `stroke-dashoffset: 0`, static.

3. **Central convergence node**:
   A fourth smaller `<circle>` at the viewBox centroid, lower default opacity, pulsing via an
   `opacity` keyframe (compositor-promoted). Represents the intersection of the three domains.

4. **Orbital particles** (CSS `offset-path`):
   Three `<circle>` elements, one per edge, animated with `offset-distance` via
   `offset-path: url(#edge-path-n)` referencing paths in `<defs>`. Each particle at a different
   duration (e.g., 3s, 4s, 5s). `offset-path` animation is compositor-promoted — zero TBT
   contribution. In reduced-motion state: `animation: none`, particles static at origin.

**Key technical constraints**:
- `offset-path` requires the referenced path to be in `<defs>` with a stable `id`
- Compositor-promoted properties used throughout: `transform`, `opacity`, `offset-distance`
- `stroke-dashoffset` is paint-thread (not compositor), acceptable within the TBT budget for a
  static portfolio hero with no competing main-thread load
- All SVG elements in the initial DOM (no JS-injected elements); animation layer is pure CSS

**Alternatives considered**:
- Overlapping circles (Venn diagram): static and animated states visually identical if only
  opacity changes; insufficient visual interest. Rejected.
- Particle cloud (many nodes): less legible as "three connected domains." Rejected.

---

## Decision 5: Astro Project Structure and Configuration

**Decision**: Single Astro 5 SSG project. No adapter. `src/islands/` physically separated from
`src/components/` to enforce the Svelte-only-for-interactivity rule at the filesystem level.

**Key configuration decisions**:
- `output: 'static'` (explicit, though it is the Astro 5 default)
- No SSR adapter: Vercel, Netlify, and Cloudflare Pages all serve `dist/` directly for static
  output without an adapter
- Integrations at init: `@astrojs/svelte`, `@astrojs/sitemap`, `sharp` (image service)
- Design tokens as pure CSS custom properties — no Style Dictionary, no build step. Maps 1:1
  to Figma's "Copy as CSS" variable export
- TypeScript: extends `astro/tsconfigs/strict`, adds `noUncheckedIndexedAccess`,
  `exactOptionalPropertyTypes`, and `@` path alias mirrored in Vite

---

## Unresolved Items (Gates — Not Blockers for Planning)

- **Figma design file**: Not yet created. Required by Principle II before implementation tasks
  are marked complete.
- **Owner name and title**: Real content needed before layout tasks complete (Principle IV).
  Assumed available per spec Assumptions section.
- **Portfolio domain URL**: Must replace the placeholder in `astro.config.mjs` `site:` field
  before first deploy.
