# Tasks: Homepage Hero Section

**Input**: Design documents from `/specs/001-homepage-hero/`
**Branch**: `001-homepage-hero`
**Stack**: Astro 5 SSG · Svelte islands · TypeScript strict · CSS custom properties

**Tests**: Not included — not requested in spec. Validation tasks (Lighthouse CI, type checks,
manual browser checks) are included in their respective phases and in Phase 6.

**Organization**: Tasks are grouped by user story to enable independent implementation and
testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel with other [P] tasks in the same phase (different files, no deps)
- **[Story]**: Which user story this task serves (US1, US2, US3 — maps to spec.md)

---

## Phase 1: Setup

**Purpose**: Bootstrap the Astro project from a blank repository. This is a net-new project —
no `src/` or config files exist yet.

- [x] T001 Bootstrap Astro 5 project from repo root: run `npm create astro@latest . -- --template minimal --typescript strict --no-install --no-git` then `npm install`
- [x] T002 Install integrations: run `npx astro add svelte`, `npx astro add sitemap`, `npm install sharp --save-dev`
- [x] T003 [P] Configure `astro.config.mjs`: set `site: 'https://your-portfolio-domain.com'` placeholder, `output: 'static'`, `integrations: [svelte(), sitemap()]`, `image.service.entrypoint: 'astro/assets/services/sharp'`, and `vite.resolve.alias: { '@': '/src' }`
- [x] T004 [P] Configure `tsconfig.json`: extend `astro/tsconfigs/strict`, add `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`, `verbatimModuleSyntax: true`, `paths: { "@/*": ["src/*"] }`, include `.astro/types.d.ts`
- [x] T005 Create source directory structure: `mkdir -p src/{components/sections,islands,design-tokens,layouts,pages,styles,assets} public`

**Checkpoint**: Astro dev server runs (`npm run dev`) at `http://localhost:4321` with no errors.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Design token system and base layout — must be complete before any user story work.

**⚠️ CRITICAL**: No user story implementation can begin until this phase is complete. T006 and
T007 are human-action gates that must be confirmed before T008 can proceed.

- [x] T006 Confirm Principle IV gate: owner full name "Fred McHale" and professional title "Design Leader" confirmed — real content, no placeholders
- [x] T008 Create `src/design-tokens/tokens.css` with initial token values defined directly in code (code-first workflow, Constitution v1.1.0): motion (`--duration-orbit-1: 3000ms`, `--duration-orbit-2: 4000ms`, `--duration-orbit-3: 5000ms`, `--duration-pulse: 2500ms`, `--duration-edge-travel: 3500ms`, `--ease-standard: cubic-bezier(0.4, 0, 0.2, 1)`, `--ease-linear: linear`), color (`--color-hero-bg`, `--color-text-primary`, `--color-text-secondary`, `--color-node-fill`, `--color-node-stroke`, `--color-edge-stroke`, `--color-particle`, `--color-center-node`, `--opacity-center-static: 0.6`), typography (`--font-display`, `--font-size-hero-name: clamp(2.5rem, 6vw, 5rem)`, `--font-size-hero-title: clamp(1rem, 2vw, 1.5rem)`, `--line-height-heading: 1.1`), spacing (`--space-2: 0.5rem`, `--space-4: 1rem`, `--space-8: 2rem`, `--space-16: 4rem`), layout (`--hero-animation-aspect-ratio: 1 / 1`) — color and font values are design decisions made in code; will be exported to Figma in Phase 6
- [x] T009 [P] Create `src/design-tokens/index.css` with a single line: `@import './tokens.css';`
- [x] T010 [P] Create `src/styles/global.css` with: CSS reset (`*, *::before, *::after { box-sizing: border-box }`, `body { margin: 0 }`), base font (`body { font-family: var(--font-display) }`), and `@layer reset, base, components` declaration
- [x] T011 Create `src/layouts/BaseLayout.astro`: import `@/design-tokens/index.css` and `@/styles/global.css` in the frontmatter, define `Props { title: string }`, render `<!doctype html>` shell with `lang="en"`, `<meta charset="UTF-8">`, `<meta name="viewport">`, `<title>{title}</title>`, and `<body><slot /></body>`

**Checkpoint**: `npm run dev` renders a blank page with no console errors. All token custom
properties are visible in browser DevTools under `:root`.

---

## Phase 3: User Story 1 — Immediate Identity Recognition (Priority: P1) 🎯 MVP

**Goal**: Owner's name and title are displayed prominently above the fold, rendered in the
initial HTML payload with no JavaScript dependency.

**Independent Test**: Open `http://localhost:4321` in a browser, disable JavaScript (DevTools
→ Settings → Debugger → Disable JavaScript), reload. Confirm the owner's full name is visible
as an `<h1>` and the professional title is visible as a `<p>`, both above the fold, at
1280×800 (desktop) and 375×812 (mobile portrait). No layout breakage.

### Implementation

- [x] T012 [US1] Create `src/components/sections/HeroSection.astro` with `Props { fullName: string; professionalTitle: string }` interface, semantic HTML: `<section class="hero">` containing `<div class="hero__text">` with `<h1 class="hero__name">{fullName}</h1>` and `<p class="hero__title">{professionalTitle}</p>`, and an empty `<div class="hero__animation-container" aria-hidden="true">` placeholder for the island
- [x] T013 [US1] Add scoped `<style>` to `src/components/sections/HeroSection.astro`: `.hero { height: 100dvh; display: grid; align-items: center; background-color: var(--color-hero-bg) }`, `.hero__name { font-size: var(--font-size-hero-name); line-height: var(--line-height-heading); color: var(--color-text-primary); margin: 0 }`, `.hero__title { font-size: var(--font-size-hero-title); color: var(--color-text-secondary); margin: var(--space-2) 0 0 }`, `.hero__animation-container { aspect-ratio: var(--hero-animation-aspect-ratio); width: 100% }` — no hardcoded values, all from tokens
- [x] T014 [P] [US1] Create `src/pages/index.astro`: import `BaseLayout` and `HeroSection`, render `<BaseLayout title="[Owner Name] — Portfolio"><HeroSection fullName="[real name]" professionalTitle="[real title]" /></BaseLayout>` with the confirmed real content from T006
- [x] T015 [US1] Verify US1 independent test manually: disable JavaScript, load the page at 1280×800 and 375×812, confirm name `<h1>` and title `<p>` are visible above the fold with no layout breakage; confirm neither is truncated

**Checkpoint**: User Story 1 is fully functional and independently testable. Name and title are
visible with JS disabled on desktop and mobile. This is the MVP.

---

## ~~Phase 4: User Story 2 — Skill Interconnection via Animation~~ DROPPED

> **Design pivot (2026-02-19)**: SVG animation dropped in favour of a pure typographic hero
> (Kanso-inspired). `src/islands/SkillsAnimation.svelte` was created but is no longer imported.
> US2 (FR-003, FR-004, FR-008) and US3 (FR-005) are superseded. The file can be deleted before
> merge.

---

## ~~Phase 5: User Story 3 — Motion-Safe Experience~~ DROPPED

> Superseded by the Phase 4 design pivot — no animation means no reduced-motion handling needed.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Responsive layout validation, accessibility verification, type-checking, and
Lighthouse CI gate (Constitution Principle III).

- [ ] T024 Verify responsive layout in `src/components/sections/HeroSection.astro` at 320px, 768px, 1280px, and 2560px: hero name stays on one line at all breakpoints, title remains legible, no horizontal overflow; adjust `--font-size-hero-name` or padding tokens if any breakpoint breaks
- [ ] T025 [P] Verify WCAG AA contrast: confirm `--color-text-primary` (#0a0a0a) vs `--color-hero-bg` (#ffffff) ≥ 4.5:1 and `--color-text-secondary` (#757575) vs `--color-hero-bg` (#ffffff) ≥ 4.5:1; update token values in `src/design-tokens/tokens.css` if below threshold
- [ ] T026 [P] Run `npx astro check` from repository root; resolve all TypeScript errors to zero errors
- [ ] T027 [P] Delete unused `src/islands/SkillsAnimation.svelte` (dropped in Phase 4 pivot); confirm `npm run build` still succeeds with zero errors
- [ ] T028 Run Lighthouse CI gate: `npm run build && npm run preview`, then `npx lighthouse http://localhost:4321 --preset=desktop --throttling-method=simulate --output=html --output-path=lighthouse-report.html`; confirm Performance ≥ 95, LCP < 2.5s, CLS < 0.1, TBT < 200ms, FCP < 1.8s (Constitution Principle III — blocking before merge)
- [ ] T029 Create Figma design file and confirm Principle II gate (code-first sync, Constitution v1.1.0): design the typographic hero layout in Figma to match the implemented code; create Figma Variables matching all token values from `src/design-tokens/tokens.css`; name the component `HeroSection` to match the codebase; publish Code Connect mapping; confirm in PR description that Figma and code agree on layout, spacing, typography, and naming with no divergence

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Setup)**: No dependencies — start immediately
- **Phase 2 (Foundational)**: Depends on Phase 1 completion — **BLOCKS all user stories**
  - T006 is confirmed (real content locked in)
  - T008 can begin immediately (token values defined in code, no Figma dependency)
  - T009, T010 can run in parallel after Phase 1 (different files, no Phase 2 dependency)
  - T011 depends on T009 + T010
- **Phase 3 (US1)**: Depends on Phase 2 completion
- **Phase 6 (Polish)**: Depends on Phase 3 completion

### Within Each Phase

- T003 [P] and T004 [P] run in parallel after T002
- T009 [P] and T010 [P] run in parallel (different files)
- T013 and T014 [P] run in parallel after T012
- T025 [P], T026 [P], T027 [P] run in parallel after T024

---

## Parallel Opportunities

### Phase 1
```
T001 → T002 → [T003 [P] + T004 [P]] → T005
```

### Phase 2
```
T006 (human gate, confirm)
  ↓
T008
  ↓
[T009 [P] + T010 [P]]
  ↓
T011
```

### Phase 3 (US1)
```
T012
  ↓
[T013 + T014 [P]]
  ↓
T015 (manual verification)
```

### Phase 6 (Polish)
```
T024
  ↓
[T025 [P] + T026 [P] + T027 [P]]
  ↓
T028 → T029
```

---

## Implementation Strategy

### Completed

1. ✅ Phase 1: Setup — Astro project bootstrapped
2. ✅ Phase 2: Foundational — tokens, base layout, real content confirmed
3. ✅ Phase 3: User Story 1 — name + title rendered in initial HTML, no JS dependency

### Remaining

4. Phase 6: Polish — responsive validation, contrast check, type check, Lighthouse CI, Figma sync

---

## Notes

- Phases 4 and 5 (animation + reduced-motion) were dropped on 2026-02-19 in favour of a pure typographic hero
- `src/islands/SkillsAnimation.svelte` exists but is unused — delete it in T027 before merge
- Token values in `tokens.css` were authored code-first; Figma sync is the T029 pre-merge gate (Constitution v1.1.0 Principle II)
- Lighthouse CI (T028) is a blocking gate — the PR cannot merge with Performance < 95
- Figma Code Connect (T029) is a blocking gate — the PR cannot merge without parity confirmation
