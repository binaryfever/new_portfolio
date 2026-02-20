# Data Model: Homepage Hero Section

**Feature**: `001-homepage-hero`
**Date**: 2026-02-19

---

## Entities

### PortfolioOwner

Static content. Source of truth: props passed into `HeroSection.astro` from `src/pages/index.astro`.

| Field | Type | Validation | Notes |
|---|---|---|---|
| `fullName` | `string` | Non-empty; real name — no placeholder | Rendered as `<h1>`; dominant text element |
| `professionalTitle` | `string` | Non-empty; real title — no placeholder | Rendered as `<p>` near name; visually subordinate |

**Constraints**:
- Both fields MUST be real content (Principle IV — Content Authenticity)
- Both fields MUST appear in the initial HTML payload (FR-007 — no JS dependency for visibility)
- Neither field may be truncated at any supported viewport (320px – 2560px wide)

---

### SkillDomains

Conceptual entity only. Not represented as runtime data — the SVG animation carries this meaning
visually without rendering text labels.

| Domain | Role in composition |
|---|---|
| Design | One of three equidistant nodes in the SVG graph |
| Engineering | One of three equidistant nodes in the SVG graph |
| Leadership | One of three equidistant nodes in the SVG graph |

**Constraints**:
- Domain labels are NOT rendered as visible text in the animation (FR-010, spec Assumptions)
- The `<svg>` MUST include an accessible `<title>` for screen readers describing the
  interconnected-skills concept
- All three nodes MUST be visually equivalent in weight (equilateral triangle — no hierarchy)

---

### HeroAnimation

The animated SVG visual. Exists in two presentation states, determined entirely by CSS.

| Field | Type | Values | Notes |
|---|---|---|---|
| `motionState` | `enum` | `"animated"` \| `"static"` | Set by CSS `@media (prefers-reduced-motion: reduce)` — no JS |
| `isHydrated` | `boolean` | `false` → `true` | Pre-hydration: SVG backbone visible; post-hydration: CSS animation active |

**Animated state** (default; `prefers-reduced-motion` not set to `reduce`):
- Orbital particles traverse connecting edges via CSS `offset-path` / `offset-distance` keyframes
- Connecting edges display a traveling signal via `stroke-dashoffset` keyframes
- Central convergence node pulses via `opacity` keyframe

**Static state** (when `prefers-reduced-motion: reduce`):
- All `animation` properties set to `none` via `@media` block in the Svelte component `<style>`
- All SVG elements rendered at full opacity — readable as a static diagram
- Same DOM structure as animated state (no conditional rendering; no CLS risk)

**Pre-hydration state** (before Svelte island mounts):
- SVG backbone (nodes + edge outlines) is visible if the SVG element is SSR-rendered inline
- `.animation-container` has layout space pre-reserved via `aspect-ratio` on the wrapper
- No layout shift when island hydrates and animation initializes

**Constraints**:
- `motionState` transitions are managed entirely by CSS — no JS conditional rendering
- Animation MUST NOT obscure or reduce legibility of name/title (FR-003, FR-010)
- Animation MUST loop continuously without jarring restarts (FR-004)
- No CLS from any state transition or from island initialization

---

## Design Token Dependencies

The following token groups MUST exist in `src/design-tokens/tokens.css` before component
authoring begins. Actual values come from the Figma design file (Principle I gate).

| Token Group | Example Custom Property Names | Usage in Component |
|---|---|---|
| Motion — duration | `--duration-orbit-1`, `--duration-orbit-2`, `--duration-orbit-3`, `--duration-pulse` | `animation-duration` for particles, pulse, edge travel |
| Motion — easing | `--ease-standard`, `--ease-linear` | `animation-timing-function` |
| Color — animation | `--color-node-fill`, `--color-node-stroke`, `--color-edge-stroke`, `--color-particle`, `--color-center-node` | SVG element `fill` and `stroke` |
| Color — text | `--color-text-primary`, `--color-text-secondary` | `<h1>` and `<p>` color |
| Color — surface | `--color-hero-bg` | Hero `<section>` background |
| Typography | `--font-display`, `--font-size-hero-name`, `--font-size-hero-title`, `--line-height-heading` | Name and title |
| Spacing | `--space-4`, `--space-8`, `--space-16` | Hero padding, name/title gap |
| Layout | `--hero-animation-aspect-ratio` | `.animation-container` aspect-ratio |

*Specific values are design decisions determined by the Figma file — they MUST NOT be hardcoded
inline in components.*
