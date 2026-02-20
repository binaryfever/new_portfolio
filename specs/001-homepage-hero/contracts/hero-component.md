# Component Contract: Homepage Hero

**Feature**: `001-homepage-hero`
**Date**: 2026-02-19
**Type**: Component Interface Contract

> This is a fully static frontend feature. There are no REST endpoints, GraphQL operations,
> or server-side data fetching contracts. All content is authored statically and rendered at
> build time by Astro.

---

## HeroSection.astro

**File**: `src/components/sections/HeroSection.astro`
**Type**: Static Astro component — no JavaScript, no Svelte island at this level

### Props Interface

```typescript
interface Props {
  /** Portfolio owner's full name. Required. Must be real content (Principle IV). */
  fullName: string;
  /** Portfolio owner's professional title or role descriptor. Required. Must be real content. */
  professionalTitle: string;
}
```

### Rendered HTML Structure

```html
<section class="hero">
  <!-- Text content: initial HTML payload, no JS dependency -->
  <div class="hero__text">
    <h1 class="hero__name">{fullName}</h1>
    <p class="hero__title">{professionalTitle}</p>
  </div>

  <!-- Animation: space pre-reserved; island hydrates after text is painted -->
  <div class="hero__animation-container" aria-hidden="true">
    <SkillsAnimation client:visible />
  </div>
</section>
```

### CSS Contracts

| Selector | Required Properties | Source |
|---|---|---|
| `.hero` | `height: 100dvh`, `display: grid` or `flex` | Design tokens for layout |
| `.hero__name` | `font-size: var(--font-size-hero-name)`, `color: var(--color-text-primary)` | Token — no inline values |
| `.hero__title` | `font-size: var(--font-size-hero-title)`, `color: var(--color-text-secondary)` | Token — no inline values |
| `.hero__animation-container` | `aspect-ratio: var(--hero-animation-aspect-ratio)` | Token; declares layout space before hydration |

### Behavioral Contracts

| Requirement | Contract |
|---|---|
| FR-007 | `fullName` and `professionalTitle` MUST be in the rendered HTML — not injected by JS |
| FR-006 | `.hero` MUST occupy full viewport height on desktop (`height: 100dvh`) |
| FR-008 | `.hero__animation-container` MUST declare `aspect-ratio` before island hydrates — no CLS |
| FR-009 | Layout MUST reflow correctly at 320px, 768px, and 1200px+ breakpoints |
| FR-010 | No tagline, CTA, or text content beyond `fullName` and `professionalTitle` |
| WCAG AA | `<h1>` and `<p>` text MUST meet minimum 4.5:1 contrast ratio against `--color-hero-bg` |

---

## SkillsAnimation.svelte

**File**: `src/islands/SkillsAnimation.svelte`
**Type**: Svelte interactive island — no external props; all animation parameters from CSS tokens

### Props Interface

```typescript
// No external props.
// All animation timing and color parameters sourced from CSS custom properties.
// This component is self-contained and renders the same output regardless of parent context.
```

### SVG Structure Contract

```html
<svg
  viewBox="0 0 400 400"
  width="100%"
  height="100%"
  role="img"
  aria-labelledby="skills-anim-title"
>
  <title id="skills-anim-title">
    Abstract diagram of interconnected skill domains: Design, Engineering, and Leadership
  </title>

  <defs>
    <!-- Paths referenced by CSS offset-path for orbital particle animation -->
    <path id="edge-path-1" d="..." />
    <path id="edge-path-2" d="..." />
    <path id="edge-path-3" d="..." />
  </defs>

  <!-- Layer 1: Domain nodes (equilateral triangle formation) -->
  <g class="nodes">
    <circle class="node node--design"      cx="200" cy="80"  r="24" />
    <circle class="node node--engineering" cx="330" cy="310" r="24" />
    <circle class="node node--leadership"  cx="70"  cy="310" r="24" />
    <circle class="node node--center"      cx="200" cy="233" r="14" />
  </g>

  <!-- Layer 2: Connecting edges -->
  <g class="edges">
    <path class="edge edge--1" d="..." />
    <path class="edge edge--2" d="..." />
    <path class="edge edge--3" d="..." />
  </g>

  <!-- Layer 3: Orbital particles (one per edge; animated via offset-path) -->
  <g class="particles">
    <circle class="particle particle--1" r="5" />
    <circle class="particle particle--2" r="5" />
    <circle class="particle particle--3" r="5" />
  </g>
</svg>
```

> Exact `cx`/`cy`/`d` values are determined during implementation from the Figma design.
> The structure above is the contract — element count, class names, and layer order are fixed.

### Animation State Contract

| Element | Animated state | Reduced-motion state |
|---|---|---|
| `.edge` | `stroke-dasharray` + `stroke-dashoffset` `@keyframes` (traveling signal) | `stroke-dashoffset: 0; animation: none; opacity: 1` |
| `.node--center` | `opacity` pulse keyframe (compositor-promoted) | `opacity: var(--opacity-center-static); animation: none` |
| `.particle` | `offset-distance` `@keyframes` via `offset-path: url(#edge-path-n)` | `animation: none; offset-distance: 0%` |

State switching is managed exclusively by CSS:

```css
@media (prefers-reduced-motion: reduce) {
  .edge     { animation: none; stroke-dashoffset: 0; opacity: 1; }
  .node--center { animation: none; }
  .particle { animation: none; offset-distance: 0%; }
}
```

### Performance Contracts

| Constraint | Implementation |
|---|---|
| TBT < 200ms | No `requestAnimationFrame` loop; no main-thread animation; all motion via CSS keyframes and `offset-path` |
| CLS = 0 | SVG `width="100%" height="100%"` fills pre-sized container; no layout contribution of its own |
| No render blocking | Island uses `client:visible`; does not block initial paint of name and title |
| Zero animation library | Only native CSS and browser APIs; no npm animation dependency |

### Accessibility Contracts

| Requirement | Implementation |
|---|---|
| Screen reader description | `role="img"` + `aria-labelledby="skills-anim-title"` on `<svg>` |
| Container hidden from AT | `aria-hidden="true"` on `.hero__animation-container` in parent |
| Reduced motion | All continuous animation disabled via `@media (prefers-reduced-motion: reduce)` |
| No decorative text | No visible text labels inside the SVG (FR-010) |
