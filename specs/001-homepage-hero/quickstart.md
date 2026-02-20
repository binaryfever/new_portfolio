# Quickstart: Homepage Hero Section

**Feature**: `001-homepage-hero`
**Date**: 2026-02-19

---

## Prerequisites

- Node.js ≥ 20 (LTS)
- npm ≥ 10
- Git

---

## 1. Bootstrap the Astro Project

From the repository root:

```bash
npm create astro@latest . -- \
  --template minimal \
  --typescript strict \
  --no-install \
  --no-git

npm install
```

`--template minimal` gives a blank Astro project without example pages. TypeScript is
configured via `--typescript strict`, generating a `tsconfig.json` that extends
`astro/tsconfigs/strict`.

---

## 2. Install Integrations

```bash
npx astro add svelte
npx astro add sitemap
npm install sharp --save-dev
```

`npx astro add` handles integration installation and auto-updates `astro.config.mjs`.
`sharp` is required explicitly so the image pipeline uses the fast Node.js image processor
rather than the WASM Squoosh fallback.

---

## 3. Configure `astro.config.mjs`

Replace the generated config with:

```js
import { defineConfig } from 'astro/config';
import svelte from '@astrojs/svelte';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  // Required for sitemap generation and canonical URLs in <meta> tags
  // Replace with the real domain before first deploy
  site: 'https://your-portfolio-domain.com',

  // Explicit static output — no SSR, no adapter needed
  output: 'static',

  integrations: [
    svelte(),
    sitemap(),
  ],

  image: {
    service: {
      entrypoint: 'astro/assets/services/sharp',
    },
  },

  vite: {
    resolve: {
      alias: {
        // Must mirror tsconfig paths so Vite and TypeScript agree
        '@': '/src',
      },
    },
  },
});
```

---

## 4. Configure `tsconfig.json`

Replace the generated file with:

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "verbatimModuleSyntax": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  },
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist", "node_modules", ".astro"]
}
```

---

## 5. Create Directory Structure

```bash
mkdir -p src/{components/sections,islands,design-tokens,layouts,pages,styles,assets}
mkdir -p public
```

---

## 6. Create Design Token File

**`src/design-tokens/tokens.css`**

Populate token values from the Figma file (Principle I + II gate). Skeleton:

```css
:root {
  /* Motion */
  --duration-orbit-1: 3000ms;
  --duration-orbit-2: 4000ms;
  --duration-orbit-3: 5000ms;
  --duration-pulse: 2500ms;
  --duration-edge-travel: 3500ms;
  --ease-standard: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-linear: linear;

  /* Color — sourced from Figma token export */
  --color-hero-bg:         /* TBD */;
  --color-text-primary:    /* TBD */;
  --color-text-secondary:  /* TBD */;
  --color-node-fill:       /* TBD */;
  --color-node-stroke:     /* TBD */;
  --color-edge-stroke:     /* TBD */;
  --color-particle:        /* TBD */;
  --color-center-node:     /* TBD */;
  --opacity-center-static: 0.6;

  /* Typography */
  --font-display:          /* TBD from Figma */;
  --font-size-hero-name:   clamp(2.5rem, 6vw, 5rem);
  --font-size-hero-title:  clamp(1rem, 2vw, 1.5rem);
  --line-height-heading:   1.1;

  /* Spacing */
  --space-1:  0.25rem;
  --space-2:  0.5rem;
  --space-4:  1rem;
  --space-8:  2rem;
  --space-16: 4rem;

  /* Layout */
  --hero-animation-aspect-ratio: 1 / 1;
}
```

**`src/design-tokens/index.css`**:

```css
@import './tokens.css';
```

> ⚠️ **Principle IV gate**: Replace all `/* TBD */` values with real Figma token values
> before any layout task is marked complete.

---

## 7. Create Base Layout

**`src/layouts/BaseLayout.astro`**:

```astro
---
import '@/design-tokens/index.css';
import '@/styles/global.css';

interface Props {
  title: string;
}

const { title } = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

---

## 8. Develop Locally

```bash
npm run dev
```

Serves at `http://localhost:4321` with hot module reload.

---

## 9. Build and Preview

```bash
npm run build     # Generates dist/
npm run preview   # Serves dist/ locally for Lighthouse testing
```

---

## 10. Lighthouse Validation (Principle III Gate)

After `npm run preview`:

```bash
npx lighthouse http://localhost:4321 \
  --preset=desktop \
  --throttling-method=simulate \
  --output=html \
  --output-path=lighthouse-report.html \
  --view
```

Required targets before any PR merge touching layout:

| Metric | Target |
|---|---|
| Performance | ≥ 95 |
| LCP | < 2.5s |
| CLS | < 0.1 |
| TBT | < 200ms |
| FCP | < 1.8s |

---

## 11. Type Checking

```bash
npx astro check      # Astro + TypeScript
npx svelte-check     # Svelte component types
```

Both MUST pass with zero errors before tasks are marked complete.

---

## Key File Locations

| Purpose | Path |
|---|---|
| Design tokens | `src/design-tokens/tokens.css` |
| Base layout | `src/layouts/BaseLayout.astro` |
| Homepage | `src/pages/index.astro` |
| Hero section component | `src/components/sections/HeroSection.astro` |
| Skills animation island | `src/islands/SkillsAnimation.svelte` |
| Global styles | `src/styles/global.css` |
| Astro config | `astro.config.mjs` |
| TypeScript config | `tsconfig.json` |
