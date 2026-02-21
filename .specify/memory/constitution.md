<!--
SYNC IMPACT REPORT
==================
Version change: 1.0.0 → 1.1.0

Modified principles:
  - Principle II (Figma ↔ Code Parity): Added explicit support for code-first workflow.
    Removed the implicit requirement that a Figma file must exist before implementation
    begins. Parity is now required before merge to `main`, not before coding starts.
    Both design-first and code-first workflows are valid.

Removed sections: N/A
Added sections: N/A

Templates requiring updates:
  ✅ .specify/templates/plan-template.md  — Constitution Check section is generic; no
                                            structural change needed. Plan authors must
                                            note which workflow (design-first or code-first)
                                            is in use for Principle II.
  ✅ .specify/templates/spec-template.md  — No changes required.
  ✅ .specify/templates/tasks-template.md — No changes required.

Follow-up TODOs:
  - specs/001-homepage-hero/plan.md: Update Principle II gate note to reflect code-first
    approval (already done in this session).
  - specs/001-homepage-hero/tasks.md: T007 (Figma gate) removed from Phase 2; Figma sync
    moved to Phase 6 as a pre-merge task (already done in this session).
-->

<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0 (initial ratification)

Modified principles: N/A — initial authoring

Added sections:
  - Core Principles (I–V)
  - Technology Stack
  - Development Workflow
  - Governance

Removed sections: N/A

Templates requiring updates:
  ✅ .specify/templates/plan-template.md  — Constitution Check section is generic; no changes
                                            required at this time. Plan author must validate
                                            against these principles when filling the template.
  ✅ .specify/templates/spec-template.md  — No structural changes required; principles apply
                                            at spec authoring time.
  ✅ .specify/templates/tasks-template.md — No structural changes required; task categories
                                            align with portfolio workflow (no backend/testing
                                            mandate by default).

Follow-up TODOs:
  - None. All placeholders resolved.
-->

# Portfolio Constitution

## Core Principles

### I. Design-System First

Every visual element MUST originate from the design token system. This includes
colors, spacing, typography, shadows, border-radius, and motion values.

- All tokens MUST be defined in a single source (e.g., `src/design-tokens/`) and
  consumed by both Figma variables and CSS/Svelte components.
- One-off inline styles are PROHIBITED. If a value is not in the token system, the
  token system MUST be extended before the value is used.
- Component names in Figma MUST match component names in code (e.g., Figma
  `Card/Project` → `<ProjectCard />`). Deviations require documented justification.
- New components MUST be added to the design system before use in page layouts.

### II. Design Tooling (Owner-Managed)

Figma design files and Code Connect mappings are maintained by the portfolio owner
independently of the speckit workflow. They are not a gate for any feature branch merge.

- Speckit tasks MUST NOT include Figma file creation, Code Connect mapping, or design
  tool sync as acceptance criteria or blocking gates.
- The owner manages design↔code parity in Figma on their own schedule.

### III. Performance Budget (NON-NEGOTIABLE)

Every page MUST meet the following targets on a cold load (no cache, throttled 4G):

| Metric | Target |
|---|---|
| Lighthouse Performance | ≥ 95 |
| Largest Contentful Paint (LCP) | < 2.5 s |
| Cumulative Layout Shift (CLS) | < 0.1 |
| Total Blocking Time (TBT) | < 200 ms |
| First Contentful Paint (FCP) | < 1.8 s |

- All images MUST use Astro's `<Image />` component or equivalent with explicit
  `width`, `height`, and modern format (avif/webp with fallback).
- Video assets MUST be lazy-loaded and MUST NOT autoplay with audio.
- No third-party script may be added without a documented performance impact
  assessment showing the budget is maintained.
- Performance is validated via `astro build` + Lighthouse CI before any merge to
  `main`.

### IV. Content Authenticity

No placeholder content is permitted in any environment, including development
branches pushed for review.

- All layouts MUST be designed and validated against real project titles, real image
  dimensions, and real copy lengths before feature completion.
- Image and video assets MUST be sourced and sized before tasks that depend on them
  are marked complete.
- Media aspect ratios declared in code MUST match the actual asset's aspect ratio.

### V. Simplicity & Scope

This is a portfolio site. Complexity MUST be justified against that scope.

- No backend server, database, or authentication unless a specific, approved use
  case is documented in the feature spec.
- Dependencies MUST be evaluated at add-time: prefer Astro/Svelte built-ins and
  native browser APIs over third-party libraries.
- YAGNI applies: do not build for hypothetical future requirements.
- Component abstraction is only warranted when a pattern appears in 3 or more
  locations.

## Technology Stack

- **Framework**: Astro (latest stable) — static site generation with islands architecture
- **Component language**: Svelte — used for interactive islands only; static markup
  MUST use Astro `.astro` components
- **Language**: TypeScript — strict mode enabled throughout (`"strict": true`)
- **Styling**: CSS custom properties fed by design tokens; scoped component styles
  via Svelte `<style>` or Astro scoped CSS
- **Deploy target**: Static hosting (Vercel / Netlify / Cloudflare Pages) — no SSR
  unless explicitly approved
- **Image pipeline**: Astro built-in `<Image />` / `<Picture />`
- **Design tool**: Figma (owner-managed; not part of speckit workflow)

## Development Workflow

- All features follow the speckit workflow: `/speckit.specify` → `/speckit.plan` →
  `/speckit.tasks` → `/speckit.implement`
- Feature branches MUST be cut from `main` and merged via pull request
- The Performance Budget (Principle III) MUST be verified via Lighthouse CI on every
  PR that touches page layout, images, or third-party scripts
- No branch touching content or layout may merge while placeholder content is present
- Figma sync is owner-managed and is not a PR merge gate

## Governance

- This constitution supersedes all other development practices and informal agreements.
- All PRs and design reviews MUST verify compliance with Principles I–V.
- Violations of Principle III (Performance Budget) are blocking — they MUST be
  resolved or the exemption MUST be documented in a Complexity Tracking table in
  the feature's `plan.md`.
- Amendments require:
  1. A documented rationale
  2. A semantic version bump (see versioning rules in the speckit constitution command)
  3. An updated `Last Amended` date
  4. A Sync Impact Report prepended to this file
- Complexity beyond portfolio scope MUST be justified in the relevant feature spec
  under a dedicated "Complexity Justification" section.

**Version**: 1.2.0 | **Ratified**: 2026-02-19 | **Last Amended**: 2026-02-21
