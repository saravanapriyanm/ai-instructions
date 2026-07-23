---
name: to-astro
description: Converts a Vite + React project (using Radix UI, MUI, Tailwind v4, shadcn-style components) into an AstroJS project. Handles component migration, dependency updates, config files, and React island patterns.
allowed-tools: Read, Edit, Write, Glob, Grep, Bash
---

You are migrating a Vite + React project into an AstroJS project. The source project uses:
- Vite as bundler
- React 18 with Radix UI, MUI, Tailwind CSS v4
- `next-themes` for theming
- Framer Motion (`motion`), react-hook-form, recharts, embla-carousel, react-dnd, react-resizable-panels
- pnpm as package manager

## Migration Steps

### Step 1 — Understand the source project
Read the project's `package.json`, `vite.config.*`, `tailwind.config.*`, and `src/` directory structure. Identify:
- Entry point (usually `src/main.tsx` or `src/index.tsx`)
- Pages vs. reusable components
- Any routing setup (react-router-dom etc.)
- Global styles and CSS imports

### Step 2 — Scaffold the Astro project
Run inside the target directory:
```bash
pnpm create astro@latest . --template minimal --no-git --install --typescript strict
```
Then add the React integration:
```bash
pnpm astro add react tailwind --yes
```

### Step 3 — Update `package.json` dependencies
Add all original dependencies to the new Astro `package.json`. Key replacements:
- Keep all Radix UI, MUI, Emotion, lucide-react, clsx, tailwind-merge, CVA, cmdk, sonner, vaul, recharts, embla-carousel-react, react-hook-form, react-dnd, react-resizable-panels, react-day-picker, input-otp, react-slick, react-responsive-masonry, date-fns, react-popper
- Replace `next-themes` with `astro-themes` or use a custom ThemeProvider React component wrapped in a `client:only="react"` island
- Keep `motion` (Framer Motion) — wrap animated components as React islands with `client:load`
- Remove: `vite`, `@vitejs/plugin-react` (Astro handles this internally)
- Add: `@astrojs/react`, `@astrojs/tailwind`

### Step 4 — Configure Astro (`astro.config.mjs`)
```js
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [
    react(),
    tailwind({ applyBaseStyles: false }),
  ],
});
```

### Step 5 — Migrate Tailwind config
Copy `tailwind.config.*` and global CSS. For Tailwind v4:
- The `@import "tailwindcss"` directive approach is used instead of `@tailwind base/components/utilities`
- Place global styles in `src/styles/global.css` and import in your Astro layout

### Step 6 — Create Astro layout and pages
- Create `src/layouts/Layout.astro` as the base layout (replaces your React root/App component)
- Create `src/pages/index.astro` as the home page
- Move page-level React components into `.astro` files using React islands

### Step 7 — Migrate React components
For each component in `src/components/`:
1. **Pure display components** (no interactivity, no hooks) → Convert to `.astro` components
2. **Interactive components** (useState, useEffect, event handlers, Radix UI, MUI, react-hook-form, recharts, react-dnd, embla-carousel, motion) → Keep as `.tsx` and use as React islands

In `.astro` files, use React components like:
```astro
---
import MyForm from '../components/MyForm.tsx';
import MyChart from '../components/MyChart.tsx';
---
<MyForm client:load />
<MyChart client:visible />
```

### Step 8 — Handle `next-themes`
`next-themes` is Next.js-specific. Replace with:
```tsx
// src/components/ThemeProvider.tsx
'use client'; // not needed in Astro, but keep for clarity
import { ThemeProvider as NextThemesProvider } from 'next-themes'; // still works if wrapped as island!
// OR use a simple script-based approach in your Layout.astro:
```
In `Layout.astro`, add inline script for theme toggling:
```astro
<script is:inline>
  const theme = localStorage.getItem('theme') ?? 'light';
  document.documentElement.classList.toggle('dark', theme === 'dark');
</script>
```
If keeping `next-themes`, wrap the entire app in a `client:only="react"` ThemeProvider island.

### Step 9 — Handle routing
If the source uses React Router:
- Replace with Astro's file-based routing under `src/pages/`
- Each route becomes a `.astro` file
- Dynamic routes use `src/pages/[slug].astro`

### Step 10 — Verify and fix
```bash
pnpm install
pnpm dev
```
Check for:
- Missing `client:*` directives on interactive components (will be hydration errors)
- CSS not loading (check global CSS import in Layout.astro)
- `window`/`document` access outside browser context — guard with `if (typeof window !== 'undefined')`
- MUI / Emotion SSR — wrap MUI components as `client:only="react"` to avoid SSR style issues

## Key Astro Island Directives
| Directive | When to use |
|---|---|
| `client:load` | Hydrate immediately on page load (forms, interactive widgets) |
| `client:idle` | Hydrate when browser is idle (less critical UI) |
| `client:visible` | Hydrate when component enters viewport (charts, carousels) |
| `client:only="react"` | Skip SSR entirely (MUI, next-themes, react-dnd, anything with window access) |

## Components that MUST use `client:only="react"`
- MUI components (Emotion SSR requires special setup)
- `next-themes` ThemeProvider
- `react-dnd` (DndProvider)
- Any component using `window`, `document`, or browser-only APIs on init

## Output
After migration, confirm:
- [ ] `pnpm dev` starts without errors
- [ ] All pages render correctly
- [ ] Interactive components hydrate properly
- [ ] Dark/light theme works
- [ ] Tailwind styles apply
