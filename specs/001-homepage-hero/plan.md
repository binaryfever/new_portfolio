# Implementation Plan: Homepage Hero Section

**Branch**: `001-homepage-hero` | **Date**: 2026-02-19 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-homepage-hero/spec.md`

## Summary

Build a full-viewport hero section for the portfolio homepage that prominently displays the
owner's name and professional title (rendered in initial HTML, no JS dependency) alongside a
looping abstract SVG animation representing the interconnection of three skill domains —
Design, Engineering, and Leadership. The animation uses CSS keyframe animations on SVG
elements (compositor-driven, zero TBT contribution), respects `prefers-reduced-motion` via a
CSS media query, and is delivered as a Svelte island with `client:visible` to keep text
rendering on the critical path. The Astro project is bootstrapped from scratch as part of
this feature.

## Technical Context

**Language/Version**: TypeScript 5.x (strict) — Astro 5 (latest stable)
**Primary Dependencies**: Astro 5 (SSG), @astrojs/svelte, @astrojs/sitemap, sharp (image service)
**Storage**: N/A
**Testing**: Lighthouse CI (performance budget per Principle III); `svelte-check` for type checking; no E2E framework mandated for this feature
**Target Platform**: Static hosting (Vercel / Netlify / Cloudflare Pages); modern browsers (ES2020+)
**Project Type**: Web (Astro SSG with Svelte islands)
**Performance Goals**: Lighthouse ≥ 95, LCP < 2.5s, CLS < 0.1, TBT < 200ms, FCP < 1.8s (Constitution Principle III + spec SC-004)
**Constraints**: Text must render without JS (FR-007); animation container pre-sized to prevent CLS (FR-008, SC-003); `prefers-reduced-motion` via CSS media query (FR-005); no external animation library
**Scale/Scope**: Single portfolio homepage; one hero section component

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|---|---|---|
| I. Design-System First | ✅ PASS | Animation colors, timing, spacing, and motion values defined as CSS custom properties in `src/design-tokens/tokens.css` before any component is authored. No inline styles. |
| II. Figma ↔ Code Parity | ✅ CODE-FIRST (v1.1.0) | Code-first workflow approved per Constitution v1.1.0. Figma design file will be created after implementation to match the code. Code Connect mapping for `<HeroSection />` and `<SkillsAnimation />` required before PR merge to `main`. |
| III. Performance Budget | ✅ PASS (pending validation) | Animation uses CSS keyframes on SVG elements (compositor thread, zero TBT). Text in initial HTML covers FCP/LCP (FR-007). CLS prevented by pre-sized island container. Validated by Lighthouse CI at build. |
| IV. Content Authenticity | ⚠️ GATE | Owner's real name and professional title must be confirmed and supplied before any layout task is marked complete. No placeholder text may be committed to any pushed branch. |
| V. Simplicity & Scope | ✅ PASS | No backend, no DB, no auth. Animation uses only native CSS and WAAPI. No external animation library. YAGNI applied throughout. |

**Post-design re-check**: Required after Phase 1 confirms component names and token definitions
align between Figma and code.

## Project Structure

### Documentation (this feature)

```text
specs/001-homepage-hero/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   └── hero-component.md  # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit.tasks — not created here)
```

### Source Code (repository root)

```text
src/
├── components/
│   └── sections/
│       └── HeroSection.astro       # Static shell: name, title, animation container
├── islands/
│   └── SkillsAnimation.svelte      # Svelte island: SVG animation (client:visible)
├── design-tokens/
│   ├── tokens.css                  # CSS custom properties: color, spacing, type, motion
│   └── index.css                   # Barrel import
├── layouts/
│   └── BaseLayout.astro            # Page shell: <head>, token import, global CSS
├── pages/
│   └── index.astro                 # Homepage: composes HeroSection
├── styles/
│   └── global.css                  # Reset, base typography, @layer order
└── assets/                         # Fonts, images (Astro <Image /> pipeline)

public/                             # Unprocessed: favicon.ico, robots.txt, og-image.png
```

**Structure Decision**: Astro SSG web application. No backend. `src/islands/` physically
separated from `src/components/` to enforce the Svelte-only-for-interactivity rule at the
filesystem level and in code review. Design tokens in `src/design-tokens/` as the single
source of truth per Principle I.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

*No active violations. Principle II and IV are pre-merge gates, not violations.*
