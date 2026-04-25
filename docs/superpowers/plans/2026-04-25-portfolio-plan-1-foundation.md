# Portfolio Plan 1 — Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up a new Next.js 15 portfolio repo with a token-driven design system, dark/light theme toggle, base layout, three primitive components, smoke-tested with Vitest + Playwright, and deployed to Vercel.

**Architecture:** Single Next.js 15 App Router app. Design tokens defined as TypeScript constants → emitted to a generated `tokens.css` of CSS custom properties → consumed by Tailwind v4 utilities and component styles. Theme persisted via `next-themes` cookie strategy (no FOUC). Components built on Radix primitives where interactive, plain HTML otherwise. Animations and content are deferred to later plans.

**Tech Stack:** Next.js 15, React 19, TypeScript (strict), Tailwind CSS v4, `next-themes`, Vitest + React Testing Library, Playwright, Ladle (component preview), Vercel.

**Output of this plan:** A live URL at `https://<project>.vercel.app` showing a styled placeholder home page with working theme toggle, header, and footer. CI green. All tests pass.

---

## File Structure

New repo `portfolio` (sibling to `MaruthanG/MaruthanG`). Suggested location: `C:/Users/Maruthan/Documents/github/portfolio`.

```
portfolio/
├── package.json
├── next.config.ts
├── tsconfig.json
├── tailwind.config.ts
├── postcss.config.mjs
├── eslint.config.mjs
├── .prettierrc.json
├── .gitignore
├── vitest.config.ts
├── vitest.setup.ts
├── playwright.config.ts
├── .ladle/config.mjs
├── .github/workflows/ci.yml
├── public/
│   └── favicon.ico
├── scripts/
│   └── generate-tokens-css.ts
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── globals.css
│   │   └── providers.tsx
│   ├── design-system/
│   │   ├── tokens/
│   │   │   ├── colors.ts
│   │   │   ├── typography.ts
│   │   │   ├── spacing.ts
│   │   │   ├── radii.ts
│   │   │   ├── shadows.ts
│   │   │   ├── motion.ts
│   │   │   ├── breakpoints.ts
│   │   │   └── index.ts
│   │   ├── tokens.css                ← generated, gitignored
│   │   ├── utils/
│   │   │   └── cn.ts
│   │   ├── components/
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── ThemeToggle.tsx
│   │   │   └── index.ts
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   └── Footer.tsx
│   │   └── stories/
│   │       ├── Button.stories.tsx
│   │       ├── Card.stories.tsx
│   │       └── Badge.stories.tsx
│   └── lib/
│       └── (reserved for later plans)
└── tests/
    ├── tokens.test.ts
    ├── cn.test.ts
    ├── ThemeToggle.test.tsx
    ├── Button.test.tsx
    └── e2e/
        └── theme-toggle.spec.ts
```

**Responsibility split:**
- `tokens/*.ts` — single source of truth for design values.
- `scripts/generate-tokens-css.ts` — pure function that turns tokens into CSS variable declarations; runs at build and dev start.
- `tokens.css` — generated artifact; never edited by hand.
- `tailwind.config.ts` — re-exports tokens as Tailwind theme so utilities (`bg-brand-500`) resolve to CSS variables.
- `components/*` — visual primitives that consume tokens via Tailwind classes.
- `layout/*` — page chrome (Header, Footer).
- `app/providers.tsx` — wraps client-side providers (theme).

---

## Conventions

- Package manager: **npm** (use `npm install`, `npm run`).
- Commit style: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`).
- Branch: work on `main` directly for v1; protect later.
- Every task ends with a commit. Small commits.

---

### Task 1: Initialize new Next.js 15 repo

**Files:**
- Create: `portfolio/` (entire directory)
- Create: `portfolio/package.json`, `portfolio/tsconfig.json`, `portfolio/next.config.ts`, `portfolio/.gitignore`, `portfolio/src/app/layout.tsx`, `portfolio/src/app/page.tsx`

- [ ] **Step 1: Scaffold Next.js app**

Run from `C:/Users/Maruthan/Documents/github/`:

```bash
npx create-next-app@latest portfolio --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --no-turbopack --use-npm
```

Answer prompts:
- Would you like to use Tailwind? → Yes
- Would you like your code inside a `src/` directory? → Yes
- Would you like to use App Router? → Yes
- Would you like to customize the import alias? → Yes (`@/*`)

Expected output: `Success! Created portfolio at ...`

- [ ] **Step 2: Verify dev server boots**

```bash
cd portfolio
npm run dev
```

Open `http://localhost:3000` — confirm Next.js welcome page renders. Stop server with Ctrl+C.

- [ ] **Step 3: Initialize git and make first commit**

```bash
cd portfolio
git init
git add -A
git commit -m "chore: scaffold Next.js 15 app with TypeScript and Tailwind"
```

- [ ] **Step 4: Create GitHub repo and push**

```bash
gh repo create maruthang/portfolio --public --source=. --remote=origin --push
```

Expected: repo visible at `https://github.com/maruthang/portfolio`.

---

### Task 2: Enforce TypeScript strict mode and add Prettier

**Files:**
- Modify: `portfolio/tsconfig.json`
- Create: `portfolio/.prettierrc.json`
- Create: `portfolio/.prettierignore`
- Modify: `portfolio/package.json` (add scripts and devDeps)

- [ ] **Step 1: Enable strict TypeScript**

Replace `compilerOptions` in `portfolio/tsconfig.json` so it includes:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

- [ ] **Step 2: Install Prettier**

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

- [ ] **Step 3: Create `.prettierrc.json`**

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

- [ ] **Step 4: Create `.prettierignore`**

```
.next
node_modules
src/design-system/tokens.css
public
```

- [ ] **Step 5: Add scripts to `package.json`**

In `package.json` `scripts`, add:

```json
"typecheck": "tsc --noEmit",
"format": "prettier --write .",
"format:check": "prettier --check ."
```

- [ ] **Step 6: Verify**

```bash
npm run typecheck
npm run format
```

Both must exit 0.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "chore: enable strict TypeScript and add Prettier"
```

---

### Task 3: Create design tokens (TypeScript source of truth)

**Files:**
- Create: `portfolio/src/design-system/tokens/colors.ts`
- Create: `portfolio/src/design-system/tokens/typography.ts`
- Create: `portfolio/src/design-system/tokens/spacing.ts`
- Create: `portfolio/src/design-system/tokens/radii.ts`
- Create: `portfolio/src/design-system/tokens/shadows.ts`
- Create: `portfolio/src/design-system/tokens/motion.ts`
- Create: `portfolio/src/design-system/tokens/breakpoints.ts`
- Create: `portfolio/src/design-system/tokens/index.ts`

- [ ] **Step 1: Create `colors.ts`**

```typescript
export const colors = {
  brand: {
    50: '#e6f1ff',
    100: '#cce3ff',
    200: '#99c7ff',
    300: '#66abff',
    400: '#338fff',
    500: '#58a6ff',
    600: '#1f7ae0',
    700: '#155bb0',
    800: '#0c3d80',
    900: '#061f50',
    950: '#031028',
  },
  neutral: {
    50: '#f7f8f9',
    100: '#eff1f3',
    200: '#dde1e6',
    300: '#c1c8d0',
    400: '#8b949e',
    500: '#6e7681',
    600: '#484f58',
    700: '#30363d',
    800: '#21262d',
    900: '#161b22',
    950: '#0d1117',
  },
  semantic: {
    success: '#3fb950',
    warning: '#d29922',
    error: '#f85149',
    info: '#58a6ff',
  },
} as const;

export type ColorTokens = typeof colors;
```

- [ ] **Step 2: Create `typography.ts`**

```typescript
export const typography = {
  fontFamily: {
    sans: ['var(--font-geist-sans)', 'ui-sans-serif', 'system-ui', 'sans-serif'],
    mono: ['var(--font-geist-mono)', 'ui-monospace', 'SFMono-Regular', 'monospace'],
  },
  fontSize: {
    xs: '0.75rem',
    sm: '0.875rem',
    base: '1rem',
    lg: '1.125rem',
    xl: '1.25rem',
    '2xl': '1.5rem',
    '3xl': '1.875rem',
    '4xl': '2.25rem',
    '5xl': '3rem',
    '6xl': '3.75rem',
    '7xl': '4.5rem',
  },
  fontWeight: {
    regular: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
  },
  lineHeight: {
    tight: '1.1',
    snug: '1.25',
    normal: '1.5',
    relaxed: '1.7',
  },
  letterSpacing: {
    tight: '-0.02em',
    normal: '0',
    wide: '0.02em',
  },
} as const;

export type TypographyTokens = typeof typography;
```

- [ ] **Step 3: Create `spacing.ts`**

```typescript
export const spacing = {
  '0': '0',
  px: '1px',
  '1': '0.25rem',
  '2': '0.5rem',
  '3': '0.75rem',
  '4': '1rem',
  '5': '1.25rem',
  '6': '1.5rem',
  '8': '2rem',
  '10': '2.5rem',
  '12': '3rem',
  '16': '4rem',
  '20': '5rem',
  '24': '6rem',
  '32': '8rem',
  '40': '10rem',
  '48': '12rem',
} as const;

export type SpacingTokens = typeof spacing;
```

- [ ] **Step 4: Create `radii.ts`**

```typescript
export const radii = {
  none: '0',
  sm: '0.25rem',
  md: '0.5rem',
  lg: '0.75rem',
  xl: '1rem',
  '2xl': '1.5rem',
  full: '9999px',
} as const;

export type RadiiTokens = typeof radii;
```

- [ ] **Step 5: Create `shadows.ts`**

```typescript
export const shadows = {
  none: 'none',
  sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
  md: '0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)',
  lg: '0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)',
  xl: '0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)',
} as const;

export type ShadowTokens = typeof shadows;
```

- [ ] **Step 6: Create `motion.ts`**

```typescript
export const motion = {
  duration: {
    fast: '150ms',
    base: '250ms',
    slow: '400ms',
  },
  easing: {
    standard: 'cubic-bezier(0.4, 0, 0.2, 1)',
    decelerate: 'cubic-bezier(0, 0, 0.2, 1)',
    accelerate: 'cubic-bezier(0.4, 0, 1, 1)',
  },
} as const;

export type MotionTokens = typeof motion;
```

- [ ] **Step 7: Create `breakpoints.ts`**

```typescript
export const breakpoints = {
  sm: '640px',
  md: '768px',
  lg: '1024px',
  xl: '1280px',
  '2xl': '1536px',
} as const;

export type BreakpointTokens = typeof breakpoints;
```

- [ ] **Step 8: Create `index.ts` barrel**

```typescript
export { colors } from './colors';
export { typography } from './typography';
export { spacing } from './spacing';
export { radii } from './radii';
export { shadows } from './shadows';
export { motion } from './motion';
export { breakpoints } from './breakpoints';

export type { ColorTokens } from './colors';
export type { TypographyTokens } from './typography';
export type { SpacingTokens } from './spacing';
export type { RadiiTokens } from './radii';
export type { ShadowTokens } from './shadows';
export type { MotionTokens } from './motion';
export type { BreakpointTokens } from './breakpoints';
```

- [ ] **Step 9: Verify TypeScript compiles**

```bash
npm run typecheck
```

Expected: exit 0.

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "feat(design-system): add design tokens for colors, typography, spacing, radii, shadows, motion, breakpoints"
```

---

### Task 4: Install Vitest and write the first failing token test

**Files:**
- Create: `portfolio/vitest.config.ts`
- Create: `portfolio/vitest.setup.ts`
- Create: `portfolio/tests/tokens.test.ts`
- Modify: `portfolio/package.json` (scripts + devDeps)

- [ ] **Step 1: Install Vitest and helpers**

```bash
npm install -D vitest @vitest/ui jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event @vitejs/plugin-react
```

- [ ] **Step 2: Create `vitest.config.ts`**

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'node:path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    globals: true,
    css: true,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

- [ ] **Step 3: Create `vitest.setup.ts`**

```typescript
import '@testing-library/jest-dom/vitest';
```

- [ ] **Step 4: Add test scripts**

In `package.json` `scripts` add:

```json
"test": "vitest run",
"test:watch": "vitest",
"test:ui": "vitest --ui"
```

- [ ] **Step 5: Write the failing test for token shape**

Create `portfolio/tests/tokens.test.ts`:

```typescript
import { describe, it, expect } from 'vitest';
import { colors, spacing, typography } from '@/design-system/tokens';

describe('design tokens', () => {
  it('exposes the brand color scale 50-950', () => {
    const expectedKeys = ['50', '100', '200', '300', '400', '500', '600', '700', '800', '900', '950'];
    expect(Object.keys(colors.brand)).toEqual(expectedKeys);
  });

  it('uses VSCode-blue as the brand 500 accent', () => {
    expect(colors.brand[500]).toBe('#58a6ff');
  });

  it('uses a 4px spacing base scale', () => {
    expect(spacing['1']).toBe('0.25rem');
    expect(spacing['4']).toBe('1rem');
  });

  it('declares CSS-variable-based font families', () => {
    expect(typography.fontFamily.sans[0]).toBe('var(--font-geist-sans)');
    expect(typography.fontFamily.mono[0]).toBe('var(--font-geist-mono)');
  });
});
```

- [ ] **Step 6: Run the test — expect PASS**

```bash
npm test
```

Expected: 4 passed (the tokens already exist from Task 3).

> Note: This is a structural assertion test, not TDD-from-red. The tokens existed first; the test pins their shape so any change is intentional.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "test(design-system): pin token shape and key values"
```

---

### Task 5: Build the CSS-variable generator with TDD

**Files:**
- Create: `portfolio/scripts/generate-tokens-css.ts`
- Create: `portfolio/tests/generate-tokens-css.test.ts`
- Modify: `portfolio/.gitignore`
- Modify: `portfolio/package.json`

- [ ] **Step 1: Append to `.gitignore`**

```
# generated tokens
src/design-system/tokens.css
```

- [ ] **Step 2: Write the failing test**

Create `portfolio/tests/generate-tokens-css.test.ts`:

```typescript
import { describe, it, expect } from 'vitest';
import { tokensToCss } from '@/../scripts/generate-tokens-css';

describe('tokensToCss', () => {
  it('emits a :root block', () => {
    const css = tokensToCss();
    expect(css).toMatch(/^:root\s*\{/m);
  });

  it('emits brand color custom properties', () => {
    const css = tokensToCss();
    expect(css).toContain('--color-brand-500: #58a6ff;');
    expect(css).toContain('--color-brand-50: #e6f1ff;');
  });

  it('emits spacing custom properties', () => {
    const css = tokensToCss();
    expect(css).toContain('--spacing-4: 1rem;');
  });

  it('emits radius custom properties', () => {
    const css = tokensToCss();
    expect(css).toContain('--radius-md: 0.5rem;');
  });

  it('emits motion custom properties', () => {
    const css = tokensToCss();
    expect(css).toContain('--duration-base: 250ms;');
    expect(css).toContain('--easing-standard: cubic-bezier(0.4, 0, 0.2, 1);');
  });

  it('emits semantic color properties', () => {
    const css = tokensToCss();
    expect(css).toContain('--color-semantic-success: #3fb950;');
  });
});
```

- [ ] **Step 3: Run the test — expect FAIL**

```bash
npm test -- generate-tokens-css
```

Expected: FAIL — module not found.

- [ ] **Step 4: Implement `generate-tokens-css.ts`**

Create `portfolio/scripts/generate-tokens-css.ts`:

```typescript
import { writeFileSync } from 'node:fs';
import { resolve } from 'node:path';
import { colors, spacing, radii, shadows, motion } from '../src/design-system/tokens';

function flatten(obj: Record<string, unknown>, prefix: string): string[] {
  const lines: string[] = [];
  for (const [key, value] of Object.entries(obj)) {
    if (typeof value === 'string') {
      lines.push(`  --${prefix}-${key}: ${value};`);
    } else if (typeof value === 'object' && value !== null) {
      for (const [subKey, subValue] of Object.entries(value as Record<string, string>)) {
        lines.push(`  --${prefix}-${key}-${subKey}: ${subValue};`);
      }
    }
  }
  return lines;
}

export function tokensToCss(): string {
  const lines: string[] = [':root {'];
  lines.push('  /* colors */');
  lines.push(...flatten(colors as unknown as Record<string, unknown>, 'color'));
  lines.push('  /* spacing */');
  lines.push(...flatten(spacing as unknown as Record<string, unknown>, 'spacing'));
  lines.push('  /* radius */');
  lines.push(...flatten(radii as unknown as Record<string, unknown>, 'radius'));
  lines.push('  /* shadow */');
  lines.push(...flatten(shadows as unknown as Record<string, unknown>, 'shadow'));
  lines.push('  /* motion */');
  for (const [k, v] of Object.entries(motion.duration)) {
    lines.push(`  --duration-${k}: ${v};`);
  }
  for (const [k, v] of Object.entries(motion.easing)) {
    lines.push(`  --easing-${k}: ${v};`);
  }
  lines.push('}');
  return lines.join('\n') + '\n';
}

function main() {
  const css = tokensToCss();
  const outPath = resolve(__dirname, '../src/design-system/tokens.css');
  writeFileSync(outPath, css, 'utf8');
  console.log(`wrote ${outPath} (${css.length} bytes)`);
}

if (require.main === module) {
  main();
}
```

- [ ] **Step 5: Run the test — expect PASS**

```bash
npm test -- generate-tokens-css
```

Expected: 6 passed.

- [ ] **Step 6: Wire into npm scripts**

Add to `package.json` `scripts`:

```json
"tokens:build": "tsx scripts/generate-tokens-css.ts",
"prebuild": "npm run tokens:build",
"predev": "npm run tokens:build"
```

- [ ] **Step 7: Install `tsx`**

```bash
npm install -D tsx
```

- [ ] **Step 8: Generate the CSS once**

```bash
npm run tokens:build
```

Expected: `wrote .../src/design-system/tokens.css (XXXX bytes)`. File exists but is gitignored.

- [ ] **Step 9: Commit**

```bash
git add -A
git commit -m "feat(design-system): generate tokens.css from TypeScript token source"
```

---

### Task 6: Wire tokens into Tailwind v4 and globals

**Files:**
- Modify: `portfolio/src/app/globals.css`
- Modify: `portfolio/tailwind.config.ts` (or create if absent)

- [ ] **Step 1: Replace `globals.css`**

Overwrite `portfolio/src/app/globals.css`:

```css
@import 'tailwindcss';
@import '../design-system/tokens.css';

@theme {
  --color-brand-50: var(--color-brand-50);
  --color-brand-100: var(--color-brand-100);
  --color-brand-200: var(--color-brand-200);
  --color-brand-300: var(--color-brand-300);
  --color-brand-400: var(--color-brand-400);
  --color-brand-500: var(--color-brand-500);
  --color-brand-600: var(--color-brand-600);
  --color-brand-700: var(--color-brand-700);
  --color-brand-800: var(--color-brand-800);
  --color-brand-900: var(--color-brand-900);
  --color-brand-950: var(--color-brand-950);

  --color-neutral-50: var(--color-neutral-50);
  --color-neutral-100: var(--color-neutral-100);
  --color-neutral-200: var(--color-neutral-200);
  --color-neutral-300: var(--color-neutral-300);
  --color-neutral-400: var(--color-neutral-400);
  --color-neutral-500: var(--color-neutral-500);
  --color-neutral-600: var(--color-neutral-600);
  --color-neutral-700: var(--color-neutral-700);
  --color-neutral-800: var(--color-neutral-800);
  --color-neutral-900: var(--color-neutral-900);
  --color-neutral-950: var(--color-neutral-950);

  --color-success: var(--color-semantic-success);
  --color-warning: var(--color-semantic-warning);
  --color-error: var(--color-semantic-error);
}

:root {
  --bg: var(--color-neutral-50);
  --fg: var(--color-neutral-900);
  --muted: var(--color-neutral-500);
  --border: var(--color-neutral-200);
  --surface: #ffffff;
}

.dark {
  --bg: var(--color-neutral-950);
  --fg: var(--color-neutral-100);
  --muted: var(--color-neutral-400);
  --border: var(--color-neutral-800);
  --surface: var(--color-neutral-900);
}

html, body {
  background: var(--bg);
  color: var(--fg);
  font-family: var(--font-geist-sans), ui-sans-serif, system-ui, sans-serif;
}
```

- [ ] **Step 2: Verify dev server picks up changes**

```bash
npm run dev
```

Open `http://localhost:3000`. Page should show with neutral-50 background. Stop server.

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "feat(design-system): wire CSS variable tokens into Tailwind v4 theme"
```

---

### Task 7: Add `cn` utility with TDD

**Files:**
- Create: `portfolio/src/design-system/utils/cn.ts`
- Create: `portfolio/tests/cn.test.ts`

- [ ] **Step 1: Install dependencies**

```bash
npm install clsx tailwind-merge
```

- [ ] **Step 2: Write the failing test**

Create `portfolio/tests/cn.test.ts`:

```typescript
import { describe, it, expect } from 'vitest';
import { cn } from '@/design-system/utils/cn';

describe('cn', () => {
  it('joins truthy class names', () => {
    expect(cn('a', 'b')).toBe('a b');
  });

  it('drops falsy values', () => {
    expect(cn('a', false, undefined, null, '', 'b')).toBe('a b');
  });

  it('merges conflicting Tailwind classes — last wins', () => {
    expect(cn('p-2', 'p-4')).toBe('p-4');
  });

  it('honors conditional class objects', () => {
    expect(cn('base', { active: true, inactive: false })).toBe('base active');
  });
});
```

- [ ] **Step 3: Run — expect FAIL**

```bash
npm test -- cn
```

Expected: module not found.

- [ ] **Step 4: Implement `cn`**

Create `portfolio/src/design-system/utils/cn.ts`:

```typescript
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]): string {
  return twMerge(clsx(inputs));
}
```

- [ ] **Step 5: Run — expect PASS**

```bash
npm test -- cn
```

Expected: 4 passed.

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat(design-system): add cn utility for class name merging"
```

---

### Task 8: Add theme toggle with `next-themes`

**Files:**
- Create: `portfolio/src/app/providers.tsx`
- Create: `portfolio/src/design-system/components/ThemeToggle.tsx`
- Create: `portfolio/tests/ThemeToggle.test.tsx`
- Modify: `portfolio/src/app/layout.tsx`

- [ ] **Step 1: Install `next-themes`**

```bash
npm install next-themes
```

- [ ] **Step 2: Create providers wrapper**

Create `portfolio/src/app/providers.tsx`:

```typescript
'use client';

import { ThemeProvider } from 'next-themes';
import type { ReactNode } from 'react';

export function Providers({ children }: { children: ReactNode }) {
  return (
    <ThemeProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </ThemeProvider>
  );
}
```

- [ ] **Step 3: Wire providers into root layout**

Replace `portfolio/src/app/layout.tsx` body with:

```typescript
import './globals.css';
import type { Metadata } from 'next';
import { Geist, Geist_Mono } from 'next/font/google';
import { Providers } from './providers';

const geistSans = Geist({ variable: '--font-geist-sans', subsets: ['latin'] });
const geistMono = Geist_Mono({ variable: '--font-geist-mono', subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Maruthan G — Portfolio',
  description: 'Full-stack developer and OSS contributor',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={`${geistSans.variable} ${geistMono.variable} antialiased`}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

- [ ] **Step 4: Write the failing component test**

Create `portfolio/tests/ThemeToggle.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ThemeProvider } from 'next-themes';
import { ThemeToggle } from '@/design-system/components/ThemeToggle';

function renderWithTheme(ui: React.ReactElement) {
  return render(
    <ThemeProvider attribute="class" defaultTheme="light" enableSystem={false}>
      {ui}
    </ThemeProvider>,
  );
}

describe('ThemeToggle', () => {
  it('renders an accessible toggle button', () => {
    renderWithTheme(<ThemeToggle />);
    expect(screen.getByRole('button', { name: /toggle theme/i })).toBeInTheDocument();
  });

  it('switches html class from light to dark on click', async () => {
    const user = userEvent.setup();
    renderWithTheme(<ThemeToggle />);
    const button = screen.getByRole('button', { name: /toggle theme/i });
    await user.click(button);
    expect(document.documentElement.classList.contains('dark')).toBe(true);
  });
});
```

- [ ] **Step 5: Run — expect FAIL**

```bash
npm test -- ThemeToggle
```

Expected: module not found.

- [ ] **Step 6: Implement `ThemeToggle`**

Create `portfolio/src/design-system/components/ThemeToggle.tsx`:

```typescript
'use client';

import { useTheme } from 'next-themes';
import { useEffect, useState } from 'react';
import { cn } from '@/design-system/utils/cn';

export function ThemeToggle({ className }: { className?: string }) {
  const { resolvedTheme, setTheme } = useTheme();
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  const isDark = mounted && resolvedTheme === 'dark';
  const next = isDark ? 'light' : 'dark';

  return (
    <button
      type="button"
      aria-label="Toggle theme"
      onClick={() => setTheme(next)}
      className={cn(
        'inline-flex h-9 w-9 items-center justify-center rounded-md border border-[var(--border)] text-[var(--fg)] hover:bg-[var(--surface)]',
        className,
      )}
    >
      <span aria-hidden>{mounted ? (isDark ? '☀' : '☾') : '·'}</span>
    </button>
  );
}
```

- [ ] **Step 7: Run — expect PASS**

```bash
npm test -- ThemeToggle
```

Expected: 2 passed.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "feat(theme): add ThemeToggle with next-themes provider"
```

---

### Task 9: Build Button primitive with TDD

**Files:**
- Create: `portfolio/src/design-system/components/Button.tsx`
- Create: `portfolio/tests/Button.test.tsx`

- [ ] **Step 1: Write the failing test**

Create `portfolio/tests/Button.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '@/design-system/components/Button';

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });

  it('fires onClick', async () => {
    const user = userEvent.setup();
    let clicked = false;
    render(<Button onClick={() => (clicked = true)}>Go</Button>);
    await user.click(screen.getByRole('button'));
    expect(clicked).toBe(true);
  });

  it('applies primary variant class by default', () => {
    render(<Button>X</Button>);
    expect(screen.getByRole('button').className).toMatch(/bg-\[var\(--color-brand-500\)\]/);
  });

  it('applies ghost variant when specified', () => {
    render(<Button variant="ghost">X</Button>);
    const cls = screen.getByRole('button').className;
    expect(cls).toMatch(/bg-transparent/);
    expect(cls).not.toMatch(/bg-\[var\(--color-brand-500\)\]/);
  });

  it('disables when disabled prop set', () => {
    render(<Button disabled>X</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('forwards type attribute', () => {
    render(<Button type="submit">X</Button>);
    expect(screen.getByRole('button')).toHaveAttribute('type', 'submit');
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
npm test -- Button
```

Expected: module not found.

- [ ] **Step 3: Implement `Button`**

Create `portfolio/src/design-system/components/Button.tsx`:

```typescript
import { forwardRef, type ButtonHTMLAttributes } from 'react';
import { cn } from '@/design-system/utils/cn';

export type ButtonVariant = 'primary' | 'ghost' | 'outline';
export type ButtonSize = 'sm' | 'md' | 'lg';

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: ButtonVariant;
  size?: ButtonSize;
}

const base =
  'inline-flex items-center justify-center font-medium rounded-md transition-colors disabled:opacity-50 disabled:pointer-events-none focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[var(--color-brand-500)] focus-visible:ring-offset-2 focus-visible:ring-offset-[var(--bg)]';

const variants: Record<ButtonVariant, string> = {
  primary: 'bg-[var(--color-brand-500)] text-white hover:bg-[var(--color-brand-600)]',
  ghost: 'bg-transparent text-[var(--fg)] hover:bg-[var(--surface)]',
  outline: 'bg-transparent border border-[var(--border)] text-[var(--fg)] hover:bg-[var(--surface)]',
};

const sizes: Record<ButtonSize, string> = {
  sm: 'h-8 px-3 text-sm',
  md: 'h-10 px-4 text-base',
  lg: 'h-12 px-6 text-lg',
};

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', className, type = 'button', ...props }, ref) => {
    return (
      <button
        ref={ref}
        type={type}
        className={cn(base, variants[variant], sizes[size], className)}
        {...props}
      />
    );
  },
);
Button.displayName = 'Button';
```

- [ ] **Step 4: Run — expect PASS**

```bash
npm test -- Button
```

Expected: 6 passed.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat(design-system): add Button primitive with primary/ghost/outline variants"
```

---

### Task 10: Add Card and Badge primitives

**Files:**
- Create: `portfolio/src/design-system/components/Card.tsx`
- Create: `portfolio/src/design-system/components/Badge.tsx`
- Create: `portfolio/src/design-system/components/index.ts`

- [ ] **Step 1: Implement `Card`**

Create `portfolio/src/design-system/components/Card.tsx`:

```typescript
import { forwardRef, type HTMLAttributes } from 'react';
import { cn } from '@/design-system/utils/cn';

export const Card = forwardRef<HTMLDivElement, HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div
      ref={ref}
      className={cn(
        'rounded-lg border border-[var(--border)] bg-[var(--surface)] p-6 shadow-sm',
        className,
      )}
      {...props}
    />
  ),
);
Card.displayName = 'Card';

export const CardHeader = forwardRef<HTMLDivElement, HTMLAttributes<HTMLDivElement>>(
  ({ className, ...props }, ref) => (
    <div ref={ref} className={cn('mb-4 flex flex-col space-y-1.5', className)} {...props} />
  ),
);
CardHeader.displayName = 'CardHeader';

export const CardTitle = forwardRef<HTMLHeadingElement, HTMLAttributes<HTMLHeadingElement>>(
  ({ className, ...props }, ref) => (
    <h3 ref={ref} className={cn('text-xl font-semibold leading-tight', className)} {...props} />
  ),
);
CardTitle.displayName = 'CardTitle';

export const CardDescription = forwardRef<HTMLParagraphElement, HTMLAttributes<HTMLParagraphElement>>(
  ({ className, ...props }, ref) => (
    <p ref={ref} className={cn('text-sm text-[var(--muted)]', className)} {...props} />
  ),
);
CardDescription.displayName = 'CardDescription';
```

- [ ] **Step 2: Implement `Badge`**

Create `portfolio/src/design-system/components/Badge.tsx`:

```typescript
import { forwardRef, type HTMLAttributes } from 'react';
import { cn } from '@/design-system/utils/cn';

export type BadgeVariant = 'default' | 'success' | 'warning' | 'error';

const variants: Record<BadgeVariant, string> = {
  default: 'bg-[var(--surface)] text-[var(--fg)] border-[var(--border)]',
  success: 'bg-[var(--color-success)]/10 text-[var(--color-success)] border-[var(--color-success)]/20',
  warning: 'bg-[var(--color-warning)]/10 text-[var(--color-warning)] border-[var(--color-warning)]/20',
  error: 'bg-[var(--color-error)]/10 text-[var(--color-error)] border-[var(--color-error)]/20',
};

export interface BadgeProps extends HTMLAttributes<HTMLSpanElement> {
  variant?: BadgeVariant;
}

export const Badge = forwardRef<HTMLSpanElement, BadgeProps>(
  ({ variant = 'default', className, ...props }, ref) => (
    <span
      ref={ref}
      className={cn(
        'inline-flex items-center rounded-md border px-2 py-0.5 text-xs font-medium',
        variants[variant],
        className,
      )}
      {...props}
    />
  ),
);
Badge.displayName = 'Badge';
```

- [ ] **Step 3: Create barrel `index.ts`**

Create `portfolio/src/design-system/components/index.ts`:

```typescript
export { Button } from './Button';
export type { ButtonProps, ButtonVariant, ButtonSize } from './Button';
export { Card, CardHeader, CardTitle, CardDescription } from './Card';
export { Badge } from './Badge';
export type { BadgeProps, BadgeVariant } from './Badge';
export { ThemeToggle } from './ThemeToggle';
```

- [ ] **Step 4: Verify build**

```bash
npm run typecheck
npm run build
```

Both must succeed.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat(design-system): add Card and Badge primitives plus components barrel"
```

---

### Task 11: Add Header and Footer layout

**Files:**
- Create: `portfolio/src/design-system/layout/Header.tsx`
- Create: `portfolio/src/design-system/layout/Footer.tsx`

- [ ] **Step 1: Implement `Header`**

Create `portfolio/src/design-system/layout/Header.tsx`:

```typescript
import Link from 'next/link';
import { ThemeToggle } from '@/design-system/components/ThemeToggle';

export function Header() {
  return (
    <header className="sticky top-0 z-40 border-b border-[var(--border)] bg-[var(--bg)]/80 backdrop-blur">
      <div className="mx-auto flex h-14 max-w-5xl items-center justify-between px-4">
        <Link href="/" className="font-mono text-sm font-semibold">
          maruthan<span className="text-[var(--color-brand-500)]">.dev</span>
        </Link>
        <nav className="flex items-center gap-6 text-sm">
          <Link href="/projects" className="text-[var(--muted)] hover:text-[var(--fg)]">
            Projects
          </Link>
          <Link href="/blog" className="text-[var(--muted)] hover:text-[var(--fg)]">
            Blog
          </Link>
          <Link href="/oss" className="text-[var(--muted)] hover:text-[var(--fg)]">
            OSS
          </Link>
          <ThemeToggle />
        </nav>
      </div>
    </header>
  );
}
```

- [ ] **Step 2: Implement `Footer`**

Create `portfolio/src/design-system/layout/Footer.tsx`:

```typescript
export function Footer() {
  const year = new Date().getFullYear();
  return (
    <footer className="border-t border-[var(--border)] py-8 text-sm text-[var(--muted)]">
      <div className="mx-auto flex max-w-5xl flex-col items-center justify-between gap-4 px-4 sm:flex-row">
        <p>© {year} Maruthan G. Built with Next.js + Tailwind.</p>
        <a
          href="https://github.com/maruthang/portfolio"
          className="hover:text-[var(--fg)]"
          target="_blank"
          rel="noreferrer"
        >
          View source
        </a>
      </div>
    </footer>
  );
}
```

- [ ] **Step 3: Wire into root layout**

Replace `portfolio/src/app/layout.tsx`:

```typescript
import './globals.css';
import type { Metadata } from 'next';
import { Geist, Geist_Mono } from 'next/font/google';
import { Providers } from './providers';
import { Header } from '@/design-system/layout/Header';
import { Footer } from '@/design-system/layout/Footer';

const geistSans = Geist({ variable: '--font-geist-sans', subsets: ['latin'] });
const geistMono = Geist_Mono({ variable: '--font-geist-mono', subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Maruthan G — Portfolio',
  description: 'Full-stack developer and OSS contributor',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className={`${geistSans.variable} ${geistMono.variable} antialiased`}>
        <Providers>
          <Header />
          <main className="mx-auto min-h-[calc(100vh-3.5rem-6rem)] max-w-5xl px-4 py-12">
            {children}
          </main>
          <Footer />
        </Providers>
      </body>
    </html>
  );
}
```

- [ ] **Step 4: Replace placeholder home page**

Replace `portfolio/src/app/page.tsx`:

```typescript
import { Button, Card, CardHeader, CardTitle, CardDescription, Badge } from '@/design-system/components';

export default function HomePage() {
  return (
    <div className="space-y-12">
      <section className="space-y-4">
        <h1 className="font-mono text-4xl font-bold sm:text-6xl">
          Hi, I&apos;m <span className="text-[var(--color-brand-500)]">Maruthan</span>
        </h1>
        <p className="text-lg text-[var(--muted)]">
          Full-stack developer and OSS contributor. Foundation in place; content arrives in Plan 2.
        </p>
        <div className="flex gap-3">
          <Button>View work</Button>
          <Button variant="outline">Resume</Button>
        </div>
      </section>

      <section>
        <Card>
          <CardHeader>
            <CardTitle>Design system smoke test</CardTitle>
            <CardDescription>If this card is styled and the badges below render, tokens are wired correctly.</CardDescription>
          </CardHeader>
          <div className="flex flex-wrap gap-2">
            <Badge>default</Badge>
            <Badge variant="success">success</Badge>
            <Badge variant="warning">warning</Badge>
            <Badge variant="error">error</Badge>
          </div>
        </Card>
      </section>
    </div>
  );
}
```

- [ ] **Step 5: Verify in browser**

```bash
npm run dev
```

Open `http://localhost:3000`. Confirm:
- Header sticky, theme toggle works (☀ ↔ ☾)
- Hero text styled
- Card with badges renders
- Footer at bottom
- Toggling theme changes background and surface colors

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat(layout): add Header, Footer, and styled placeholder home"
```

---

### Task 12: Set up Ladle for component preview

**Files:**
- Create: `portfolio/.ladle/config.mjs`
- Create: `portfolio/src/design-system/stories/Button.stories.tsx`
- Create: `portfolio/src/design-system/stories/Card.stories.tsx`
- Create: `portfolio/src/design-system/stories/Badge.stories.tsx`
- Modify: `portfolio/package.json`

- [ ] **Step 1: Install Ladle**

```bash
npm install -D @ladle/react
```

- [ ] **Step 2: Add scripts**

Add to `package.json`:

```json
"ladle": "ladle serve",
"ladle:build": "ladle build"
```

- [ ] **Step 3: Configure Ladle**

Create `portfolio/.ladle/config.mjs`:

```javascript
export default {
  stories: 'src/**/*.stories.{ts,tsx}',
  defaultStory: 'button--primary',
};
```

- [ ] **Step 4: Create Button story**

Create `portfolio/src/design-system/stories/Button.stories.tsx`:

```typescript
import type { Story } from '@ladle/react';
import { Button, type ButtonProps } from '@/design-system/components/Button';
import '@/app/globals.css';

export const Primary: Story<ButtonProps> = (args) => <Button {...args}>Primary</Button>;
Primary.args = { variant: 'primary', size: 'md' };

export const Ghost: Story<ButtonProps> = (args) => <Button {...args}>Ghost</Button>;
Ghost.args = { variant: 'ghost' };

export const Outline: Story<ButtonProps> = (args) => <Button {...args}>Outline</Button>;
Outline.args = { variant: 'outline' };

export const Disabled: Story<ButtonProps> = (args) => <Button {...args}>Disabled</Button>;
Disabled.args = { disabled: true };
```

- [ ] **Step 5: Create Card story**

Create `portfolio/src/design-system/stories/Card.stories.tsx`:

```typescript
import type { Story } from '@ladle/react';
import { Card, CardHeader, CardTitle, CardDescription } from '@/design-system/components/Card';
import '@/app/globals.css';

export const Basic: Story = () => (
  <Card>
    <CardHeader>
      <CardTitle>Card title</CardTitle>
      <CardDescription>Some description text.</CardDescription>
    </CardHeader>
    <p>Body content goes here.</p>
  </Card>
);
```

- [ ] **Step 6: Create Badge story**

Create `portfolio/src/design-system/stories/Badge.stories.tsx`:

```typescript
import type { Story } from '@ladle/react';
import { Badge, type BadgeProps } from '@/design-system/components/Badge';
import '@/app/globals.css';

export const All: Story<BadgeProps> = () => (
  <div style={{ display: 'flex', gap: '0.5rem' }}>
    <Badge>default</Badge>
    <Badge variant="success">success</Badge>
    <Badge variant="warning">warning</Badge>
    <Badge variant="error">error</Badge>
  </div>
);
```

- [ ] **Step 7: Verify Ladle**

```bash
npm run tokens:build
npm run ladle
```

Open the URL printed (default `http://localhost:61000`). Confirm Button, Card, Badge stories all render. Stop server.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "feat(stories): add Ladle config and stories for Button, Card, Badge"
```

---

### Task 13: Add Playwright smoke test for theme toggle

**Files:**
- Create: `portfolio/playwright.config.ts`
- Create: `portfolio/tests/e2e/theme-toggle.spec.ts`
- Modify: `portfolio/package.json`
- Modify: `portfolio/.gitignore`

- [ ] **Step 1: Install Playwright**

```bash
npm init playwright@latest -- --quiet --browser=chromium --gha=false --install-deps=false
```

When prompted, accept defaults; choose TypeScript and `tests/e2e` as the test folder.

- [ ] **Step 2: Replace `playwright.config.ts`**

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: 'list',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  webServer: {
    command: 'npm run build && npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

- [ ] **Step 3: Append to `.gitignore`**

```
playwright-report
test-results
```

- [ ] **Step 4: Replace generated example test**

Delete any auto-generated `tests/e2e/example.spec.ts` if present, then create `portfolio/tests/e2e/theme-toggle.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test('home page renders and theme toggle flips the html class', async ({ page }) => {
  await page.goto('/');
  await expect(page.getByRole('heading', { level: 1 })).toContainText('Maruthan');

  const toggle = page.getByRole('button', { name: /toggle theme/i });
  await toggle.click();

  const htmlClass = await page.locator('html').getAttribute('class');
  expect(htmlClass).toMatch(/dark|light/);
});
```

- [ ] **Step 5: Add e2e script**

In `package.json`:

```json
"test:e2e": "playwright test"
```

- [ ] **Step 6: Run the e2e test**

```bash
npm run test:e2e
```

Expected: 1 passed.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "test(e2e): add Playwright smoke test for home page and theme toggle"
```

---

### Task 14: Add GitHub Actions CI

**Files:**
- Create: `portfolio/.github/workflows/ci.yml`

- [ ] **Step 1: Create CI workflow**

Create `portfolio/.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run typecheck
      - run: npm run format:check
      - run: npm test
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: npm run test:e2e
```

- [ ] **Step 2: Commit and push**

```bash
git add -A
git commit -m "ci: add GitHub Actions workflow for typecheck, format, unit, build, e2e"
git push origin main
```

- [ ] **Step 3: Verify CI green**

Visit `https://github.com/maruthang/portfolio/actions`. Confirm the workflow run succeeds. If it fails, fix the failing step and push again.

---

### Task 15: Deploy to Vercel

**Files:**
- (none — Vercel config via dashboard)

- [ ] **Step 1: Install Vercel CLI**

```bash
npm install -g vercel
```

- [ ] **Step 2: Link and deploy**

```bash
cd portfolio
vercel
```

Answer prompts:
- Set up and deploy? → Y
- Which scope? → personal account
- Link to existing project? → N
- What's your project's name? → portfolio
- In which directory is your code located? → ./
- Want to override settings? → N

Wait for deploy. Note the preview URL.

- [ ] **Step 3: Promote to production**

```bash
vercel --prod
```

Note the production URL (e.g., `https://portfolio-xxxx.vercel.app`).

- [ ] **Step 4: Verify in browser**

Open the production URL. Confirm:
- Page loads
- Theme toggle works
- No console errors
- Lighthouse > 90 on Performance, Accessibility, Best Practices, SEO

- [ ] **Step 5: Document the URL**

Append to `portfolio/README.md` (create if absent):

```markdown
# portfolio

Personal portfolio for Maruthan G.

- Live: https://<your-vercel-url>
- Stack: Next.js 15, React 19, Tailwind v4, TypeScript
- See `docs/` in the parent planning repo for design and plans.
```

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "docs: add README with live URL"
git push origin main
```

---

## Self-Review

**Spec coverage** — checked against `docs/superpowers/specs/2026-04-25-portfolio-design.md`:

| Spec section | Covered by |
|---|---|
| §3 Tech Stack: Next.js 15, TS strict, Tailwind v4 | Tasks 1, 2, 6 |
| §6 Design System tokens architecture | Tasks 3, 5, 6 |
| §6 Token-driven flow (TS → tokens.css → Tailwind) | Tasks 5, 6 |
| §6 Component preview (Ladle) | Task 12 |
| §5.5 Theme toggle (no FOUC, persisted) | Task 8 |
| §11 Lighthouse ≥95 success criterion | Task 15 step 4 |
| §9 Vercel deployment, GitHub Actions | Tasks 14, 15 |
| §1 Single source of truth design system | Task 5 (generator) + Task 6 (Tailwind wiring) |

**Deferred to later plans (intentional):**
- Hero animations, About, Stack, Projects sections → Plan 2
- Project detail pages, OSS page → Plan 3
- Blog → Plan 4
- Command palette, /uses, /now, OG images → Plan 5
- Analytics, custom domain, structured data → Plan 6

**Placeholder scan:** None found. All steps contain runnable code or commands.

**Type consistency:** `ButtonProps`, `BadgeProps`, `cn` signature, and `tokensToCss` signature consistent across tasks.

**Open clarification before execution:**
- GitHub username — confirm `maruthang` (used in Task 1 step 4 and Task 15 step 5). If different, update before running.
- Repo name — `portfolio` is used. Change in Task 1 if you prefer another name.

---

## Definition of Done for Plan 1

- ✅ Repo `maruthang/portfolio` exists and is public.
- ✅ Live URL on Vercel renders home page.
- ✅ Dark/light theme toggle works without flash on load.
- ✅ All unit tests pass (`npm test`).
- ✅ Playwright smoke test passes (`npm run test:e2e`).
- ✅ TypeScript strict mode, no errors (`npm run typecheck`).
- ✅ Prettier clean (`npm run format:check`).
- ✅ CI green on `main`.
- ✅ Tokens regenerate via `npm run tokens:build`; changing a token value visibly updates the live page after rebuild.
