# Portfolio Plan 2 — Home Page Content Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the Plan-1 placeholder home with a real, content-rich single-page portfolio: animated hero, about section with stats, categorized tech stack, featured-projects card grid, OSS preview block, contact form with graceful Resend fallback, and a polished footer. All sections respect `prefers-reduced-motion`.

**Architecture:** Sections live as composable React components under `src/sections/`. Content (projects, stats, OSS highlights, tech stack) lives in pure TS data files under `src/content/` so it can be reused later by the OSS page (Plan 3) and project detail pages (Plan 3). Animation library mix per spec: Framer Motion for component motion, Lenis for smooth scroll, plus a reduced-motion utility that gates every animation. Contact form uses a server action with a graceful no-API-key fallback (renders a `mailto:` link) so the form is usable before `RESEND_API_KEY` is set.

**Tech Stack additions:** `framer-motion`, `lenis`, `react-hook-form`, `@hookform/resolvers`, `zod`, `resend`, `lucide-react` (icons), `react-intersection-observer` (count-up trigger).

**Output of this plan:** A real portfolio home page deployed to `https://portfolio-tawny-two-72.vercel.app/` that any visitor can read end-to-end. CI green. All tests pass.

---

## File Structure

Working from `C:/Users/Maruthan/Documents/github/portfolio`. New paths in **bold**.

```
src/
  app/
    page.tsx                          ← rewritten to compose section components
    actions/
      contact.ts                      ← (NEW) server action for contact form
  sections/                           ← (NEW)
    Hero.tsx
    About.tsx
    TechStack.tsx
    FeaturedProjects.tsx
    OSSPreview.tsx
    Contact.tsx
  content/                            ← (NEW)
    projects.ts
    stats.ts
    oss.ts
    techStack.ts
    contact.ts                        ← email + socials data
  design-system/
    components/
      Typewriter.tsx                  ← (NEW) reused in Hero
      StatCounter.tsx                 ← (NEW) reused in About
      ProjectCard.tsx                 ← (NEW) reused in FeaturedProjects
      TechChip.tsx                    ← (NEW) reused in TechStack
      Section.tsx                     ← (NEW) consistent section wrapper
      ContactForm.tsx                 ← (NEW)
      index.ts                        ← updated barrel
    layout/
      Footer.tsx                      ← updated content
    motion/                           ← (NEW)
      reducedMotion.ts                ← hook + utility
      variants.ts                     ← reusable Framer Motion variants
      LenisProvider.tsx               ← smooth-scroll provider
  lib/
    formatters.ts                     ← (NEW) e.g., compact number formatting
tests/
  reducedMotion.test.ts               ← (NEW)
  Typewriter.test.tsx                 ← (NEW)
  StatCounter.test.tsx                ← (NEW)
  ContactForm.test.tsx                ← (NEW)
  contact-action.test.ts              ← (NEW)
  e2e/
    home-content.spec.ts              ← (NEW) replaces theme-toggle.spec.ts as the main e2e
```

**Responsibility split:**
- `content/*` is pure data, no JSX, no hooks. Lets us migrate to MDX or a CMS later without changing components.
- `sections/*` compose design-system components with content. Server components by default; only mark `'use client'` where animation hooks require it.
- `motion/*` is the animation toolkit; sections import from here, never from `framer-motion` directly (one-place control if we ever swap libraries).
- `actions/contact.ts` is the only place the contact form talks to Resend.

---

## Conventions

- Package manager: **npm**.
- Commit style: Conventional Commits.
- Branch: continue on `main`, push at the end of each task (Plan 1 established the CI-green-on-main workflow).
- Every task ends with a commit. Push at the end of EACH task in this plan (Plan 1 batched pushes; this plan is past the bootstrap stage so we push continuously and let CI run).
- Animations gated by `useReducedMotion()` from the `motion/reducedMotion.ts` module — no exceptions.
- Sections render as **server components** unless they need hooks; mark `'use client'` only on the inner widget that needs it (e.g., `Typewriter`, `StatCounter`, `ContactForm`).

---

### Task 1: Install animation, form, and email libraries

**Files:**
- Modify: `portfolio/package.json`, `portfolio/package-lock.json`

- [ ] **Step 1: Install runtime deps**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm install framer-motion lenis react-hook-form @hookform/resolvers zod resend lucide-react react-intersection-observer
```

- [ ] **Step 2: Verify versions land**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm ls framer-motion lenis react-hook-form @hookform/resolvers zod resend lucide-react react-intersection-observer
```

Confirm each appears at top level with a resolved version. None should be marked invalid or missing peer.

- [ ] **Step 3: Verify nothing broke**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm test
npm run build
```

All exit 0. Tests still 22 passed. Build succeeds.

- [ ] **Step 4: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "chore(deps): add framer-motion, lenis, react-hook-form, zod, resend, lucide-react"
git push origin main
```

Wait for CI to go green: `gh run watch --repo maruthang/portfolio`.

---

### Task 2: Build the reduced-motion utility with TDD

**Files:**
- Create: `portfolio/src/design-system/motion/reducedMotion.ts`
- Create: `portfolio/tests/reducedMotion.test.ts`

- [ ] **Step 1: Write the failing test**

Create `portfolio/tests/reducedMotion.test.ts`:

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { renderHook } from '@testing-library/react';
import { useReducedMotion, getReducedMotion } from '@/design-system/motion/reducedMotion';

function setMatchMedia(matches: boolean) {
  Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: (query: string) => ({
      matches,
      media: query,
      onchange: null,
      addEventListener: () => {},
      removeEventListener: () => {},
      addListener: () => {},
      removeListener: () => {},
      dispatchEvent: () => false,
    }),
  });
}

describe('reducedMotion', () => {
  beforeEach(() => {
    setMatchMedia(false);
  });

  it('getReducedMotion returns false when prefers-reduced-motion: no-preference', () => {
    setMatchMedia(false);
    expect(getReducedMotion()).toBe(false);
  });

  it('getReducedMotion returns true when prefers-reduced-motion: reduce', () => {
    setMatchMedia(true);
    expect(getReducedMotion()).toBe(true);
  });

  it('useReducedMotion hook returns the current preference', () => {
    setMatchMedia(true);
    const { result } = renderHook(() => useReducedMotion());
    expect(result.current).toBe(true);
  });

  it('useReducedMotion returns false on first render when no media query support', () => {
    Object.defineProperty(window, 'matchMedia', { writable: true, value: undefined });
    const { result } = renderHook(() => useReducedMotion());
    expect(result.current).toBe(false);
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- reducedMotion
```

Expected: module not found.

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/motion/reducedMotion.ts`:

```typescript
'use client';

import { useEffect, useState } from 'react';

const QUERY = '(prefers-reduced-motion: reduce)';

export function getReducedMotion(): boolean {
  if (typeof window === 'undefined' || typeof window.matchMedia !== 'function') {
    return false;
  }
  return window.matchMedia(QUERY).matches;
}

export function useReducedMotion(): boolean {
  const [reduced, setReduced] = useState<boolean>(false);

  useEffect(() => {
    if (typeof window === 'undefined' || typeof window.matchMedia !== 'function') {
      return;
    }
    const mq = window.matchMedia(QUERY);
    setReduced(mq.matches);
    const handler = (e: MediaQueryListEvent) => setReduced(e.matches);
    mq.addEventListener('change', handler);
    return () => mq.removeEventListener('change', handler);
  }, []);

  return reduced;
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- reducedMotion
```

Expected: 4 passed.

- [ ] **Step 5: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(motion): add reduced-motion hook and utility"
git push origin main
```

---

### Task 3: Add reusable motion variants and LenisProvider

**Files:**
- Create: `portfolio/src/design-system/motion/variants.ts`
- Create: `portfolio/src/design-system/motion/LenisProvider.tsx`
- Modify: `portfolio/src/app/providers.tsx`

- [ ] **Step 1: Create variants library**

Create `portfolio/src/design-system/motion/variants.ts`:

```typescript
import type { Variants } from 'framer-motion';

export const fadeUp: Variants = {
  hidden: { opacity: 0, y: 12 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.4, ease: [0.4, 0, 0.2, 1] } },
};

export const fadeIn: Variants = {
  hidden: { opacity: 0 },
  visible: { opacity: 1, transition: { duration: 0.35 } },
};

export const stagger: Variants = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.06, delayChildren: 0.05 } },
};

export const reducedFade: Variants = {
  hidden: { opacity: 0 },
  visible: { opacity: 1, transition: { duration: 0 } },
};
```

- [ ] **Step 2: Create LenisProvider**

Create `portfolio/src/design-system/motion/LenisProvider.tsx`:

```typescript
'use client';

import { useEffect, type ReactNode } from 'react';
import Lenis from 'lenis';
import { useReducedMotion } from './reducedMotion';

export function LenisProvider({ children }: { children: ReactNode }) {
  const reduced = useReducedMotion();

  useEffect(() => {
    if (reduced) return;
    const lenis = new Lenis({ duration: 1.1, easing: (t) => 1 - Math.pow(1 - t, 3) });
    let raf = 0;
    const tick = (t: number) => {
      lenis.raf(t);
      raf = requestAnimationFrame(tick);
    };
    raf = requestAnimationFrame(tick);
    return () => {
      cancelAnimationFrame(raf);
      lenis.destroy();
    };
  }, [reduced]);

  return <>{children}</>;
}
```

- [ ] **Step 3: Wire LenisProvider into app providers**

Replace `portfolio/src/app/providers.tsx`:

```typescript
'use client';

import { ThemeProvider } from 'next-themes';
import type { ReactNode } from 'react';
import { LenisProvider } from '@/design-system/motion/LenisProvider';

export function Providers({ children }: { children: ReactNode }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem disableTransitionOnChange>
      <LenisProvider>{children}</LenisProvider>
    </ThemeProvider>
  );
}
```

- [ ] **Step 4: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm test
npm run build
```

All exit 0.

- [ ] **Step 5: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(motion): add motion variants and Lenis smooth-scroll provider"
git push origin main
```

---

### Task 4: Create content data files

**Files:**
- Create: `portfolio/src/content/contact.ts`
- Create: `portfolio/src/content/stats.ts`
- Create: `portfolio/src/content/techStack.ts`
- Create: `portfolio/src/content/projects.ts`
- Create: `portfolio/src/content/oss.ts`

- [ ] **Step 1: Create contact data**

Create `portfolio/src/content/contact.ts`:

```typescript
export const contact = {
  email: 'maruthangt@gmail.com',
  location: 'Chennai, India',
  availability: 'Open to opportunities',
  socials: [
    { label: 'GitHub', href: 'https://github.com/maruthang', handle: '@maruthang' },
    { label: 'LinkedIn', href: 'https://linkedin.com/in/maruthan-g-6a7415201', handle: 'maruthan-g' },
    { label: 'Telegram', href: 'https://t.me/Maruthang', handle: '@Maruthang' },
  ],
} as const;

export type Social = (typeof contact.socials)[number];
```

- [ ] **Step 2: Create stats data**

Create `portfolio/src/content/stats.ts`:

```typescript
export interface Stat {
  label: string;
  value: number;
  suffix?: string;
}

export const stats: Stat[] = [
  { label: 'Merged PRs', value: 57, suffix: '+' },
  { label: 'OSS projects', value: 9, suffix: '+' },
  { label: 'Projects shipped', value: 10, suffix: '+' },
  { label: 'Years professional', value: 2, suffix: '+' },
];
```

- [ ] **Step 3: Create tech stack data**

Create `portfolio/src/content/techStack.ts`:

```typescript
export type TechCategory =
  | 'Languages'
  | 'Frontend'
  | 'Backend'
  | 'Databases'
  | 'Cloud & DevOps'
  | 'Data Engineering';

export interface Tech {
  name: string;
  category: TechCategory;
  proficiency?: string;
  learning?: boolean;
}

export const techStack: Tech[] = [
  // Languages
  { name: 'TypeScript', category: 'Languages', proficiency: 'Daily driver' },
  { name: 'JavaScript', category: 'Languages' },
  { name: 'Python', category: 'Languages' },
  { name: 'PHP', category: 'Languages' },
  { name: 'SQL', category: 'Languages' },
  { name: 'Scala', category: 'Languages', learning: true },

  // Frontend
  { name: 'Next.js', category: 'Frontend', proficiency: 'Production' },
  { name: 'React', category: 'Frontend', proficiency: 'Production' },
  { name: 'Angular', category: 'Frontend', proficiency: 'Production' },
  { name: 'React Native', category: 'Frontend', proficiency: 'Production' },
  { name: 'Expo', category: 'Frontend' },
  { name: 'Ionic', category: 'Frontend' },
  { name: 'Tailwind CSS', category: 'Frontend' },
  { name: 'Redux Toolkit', category: 'Frontend' },
  { name: 'D3.js', category: 'Frontend' },

  // Backend
  { name: 'NestJS', category: 'Backend', proficiency: 'Production + OSS' },
  { name: 'Node.js', category: 'Backend', proficiency: 'Production + OSS' },
  { name: 'Express', category: 'Backend' },
  { name: 'BullMQ', category: 'Backend', proficiency: 'OSS contributor' },
  { name: 'Socket.io', category: 'Backend' },
  { name: 'TypeORM', category: 'Backend' },
  { name: 'WordPress / WooCommerce', category: 'Backend' },

  // Databases
  { name: 'PostgreSQL', category: 'Databases', proficiency: 'Production' },
  { name: 'MySQL', category: 'Databases' },
  { name: 'MariaDB', category: 'Databases' },
  { name: 'Redis', category: 'Databases' },
  { name: 'Delta Lake', category: 'Databases', learning: true },
  { name: 'Firebase', category: 'Databases' },

  // Cloud
  { name: 'AWS', category: 'Cloud & DevOps' },
  { name: 'Azure', category: 'Cloud & DevOps' },
  { name: 'Docker', category: 'Cloud & DevOps' },
  { name: 'GitLab CI', category: 'Cloud & DevOps' },
  { name: 'Vercel', category: 'Cloud & DevOps' },

  // Data Engineering (learning)
  { name: 'Databricks', category: 'Data Engineering', learning: true },
  { name: 'PySpark', category: 'Data Engineering', learning: true },
  { name: 'dbt', category: 'Data Engineering', learning: true },
  { name: 'Power BI', category: 'Data Engineering', learning: true },
];

export const techCategoriesInOrder: TechCategory[] = [
  'Languages',
  'Frontend',
  'Backend',
  'Databases',
  'Cloud & DevOps',
  'Data Engineering',
];
```

- [ ] **Step 4: Create projects data**

Create `portfolio/src/content/projects.ts`:

```typescript
export type ProjectStatus = 'live' | 'archived' | 'oss' | 'learning';

export interface Project {
  slug: string;
  title: string;
  summary: string;
  status: ProjectStatus;
  tech: string[];
  featured: boolean;
  links?: {
    live?: string;
    repo?: string;
    case?: string;
  };
}

export const projects: Project[] = [
  {
    slug: 'b2b-marketplace',
    title: 'B2B Multi-Vendor Marketplace',
    summary:
      'B2B marketplace with reverse-auction bidding, KYC/seller verification, AI-powered product creation (AWS Lambda), live chat, and dispute management. 17+ custom WordPress plugins, full Docker Compose infra, and a backup → deploy → rollback CI/CD pipeline.',
    status: 'live',
    tech: ['WordPress', 'WooCommerce', 'PHP', 'MariaDB', 'Redis', 'Docker', 'GitLab CI', 'AWS Lambda'],
    featured: true,
  },
  {
    slug: 'conversational-commerce-bot',
    title: 'Conversational Commerce Bot',
    summary:
      'Production WhatsApp ↔ WooCommerce bridge. Customers browse, manage carts, place orders, and resolve disputes via WhatsApp. HMAC-SHA256 webhook verification, idempotency via SQLite, multi-step conversation state.',
    status: 'live',
    tech: ['NestJS 11', 'TypeScript', 'SQLite', 'Express 5', 'Meta WhatsApp Cloud API'],
    featured: true,
  },
  {
    slug: 'sales-analytics-platform',
    title: 'Enterprise Sales & Commerce Analytics',
    summary:
      'Multi-channel analytics platform aggregating 8+ e-commerce and quick-commerce channels. Real-time dashboards, dynamic report builder, cohort analysis, 2FA/MFA (TOTP), CASL RBAC, and 192+ API endpoints.',
    status: 'live',
    tech: ['NestJS 10', 'Next.js 15', 'React 18', 'PostgreSQL', 'Redis', 'BullMQ', 'Socket.io', 'D3.js'],
    featured: true,
  },
  {
    slug: 'fitness-ecosystem',
    title: 'Cross-Platform Fitness Ecosystem',
    summary:
      'Mobile app, admin dashboard, and marketing site for a fitness platform. Trainer bookings, workout planning, AI coaching (GPT-4o), social fitness, real-time messaging, payment processing, push notifications.',
    status: 'live',
    tech: ['NestJS 10', 'React Native + Expo 51', 'Next.js 15', 'PostgreSQL', 'Socket.io', 'OpenAI GPT-4o'],
    featured: true,
  },
  {
    slug: 'health-wellness-app',
    title: 'Health & Wellness Mobile App',
    summary:
      'Hybrid mobile app for hydration tracking, step counting, and health goal management with Keycloak-based SSO and Firebase push notifications.',
    status: 'live',
    tech: ['NestJS 10', 'Angular 18', 'Ionic 8', 'Capacitor 6', 'MySQL', 'Keycloak', 'Firebase'],
    featured: true,
  },
  {
    slug: 'enterprise-data-warehouse',
    title: 'Enterprise Data Warehouse',
    summary:
      'Medallion architecture (Bronze → Silver → Gold) powering executive Power BI dashboards in the insurance domain. SCD Type 1/2, Delta Lake optimizations, and Databricks Asset Bundles for IaC.',
    status: 'learning',
    tech: ['Databricks', 'PySpark', 'Scala', 'Delta Lake', 'dbt', 'Azure ADLS', 'Power BI'],
    featured: true,
  },
];

export const featuredProjects = projects.filter((p) => p.featured);
```

- [ ] **Step 5: Create OSS data**

Create `portfolio/src/content/oss.ts`:

```typescript
export interface OssProject {
  name: string;
  href: string;
  stars: string;
  merged: number;
  open: number;
  focus: string;
}

export interface OssHighlight {
  project: string;
  pr: string;
  title: string;
  href: string;
  mergedOn?: string;
}

export const ossStats = {
  totalMerged: 57,
  totalOpen: 103,
  projectCount: 9,
} as const;

export const ossProjects: OssProject[] = [
  {
    name: 'nestjs/nest-cli',
    href: 'https://github.com/nestjs/nest-cli',
    stars: '2.1k+',
    merged: 15,
    open: 2,
    focus: 'SWC compiler, watch mode, build system, async shutdown hooks, signal forwarding',
  },
  {
    name: 'nestjs/swagger',
    href: 'https://github.com/nestjs/swagger',
    stars: '1.4k+',
    merged: 14,
    open: 0,
    focus: 'Schema handling, plugin fixes, enum mutation, TS project references, SWC metadata',
  },
  {
    name: 'microsoft/vscode',
    href: 'https://github.com/microsoft/vscode',
    stars: '183k+',
    merged: 12,
    open: 46,
    focus: 'ANSI escape handling, chat/terminal tooling, editor UI, context key services',
  },
  {
    name: 'nestjs/graphql',
    href: 'https://github.com/nestjs/graphql',
    stars: '1.5k+',
    merged: 6,
    open: 0,
    focus: 'Apollo subscriptions, abstract directive inheritance, global prefix handling',
  },
  {
    name: 'nodejs/undici',
    href: 'https://github.com/nodejs/undici',
    stars: '7k+',
    merged: 4,
    open: 2,
    focus: 'HTTP interceptors, redirect options, type definitions',
  },
  {
    name: 'taskforcesh/bullmq',
    href: 'https://github.com/taskforcesh/bullmq',
    stars: '7k+',
    merged: 3,
    open: 13,
    focus: 'Worker scheduler registry, repeatable jobs, queue internals',
  },
  {
    name: 'angular/angular-cli',
    href: 'https://github.com/angular/angular-cli',
    stars: '27k+',
    merged: 2,
    open: 3,
    focus: 'Build system, error stack traces, styleUrl validation',
  },
];

export const ossHighlights: OssHighlight[] = [
  {
    project: 'nodejs/undici',
    pr: '#5066',
    title: 'fix(interceptor): add throwOnMaxRedirect to types and interceptor opts',
    href: 'https://github.com/nodejs/undici/pull/5066',
  },
  {
    project: 'microsoft/vscode',
    pr: '#307960',
    title: 'fix: handle heredoc/multiline commands in terminal tool execution',
    href: 'https://github.com/microsoft/vscode/pull/307960',
  },
  {
    project: 'nestjs/graphql',
    pr: '#3937',
    title: 'fix(apollo): respect useGlobalPrefix on custom subscription path',
    href: 'https://github.com/nestjs/graphql/pull/3937',
  },
];
```

- [ ] **Step 6: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm test
```

Both exit 0. Tests still 22 passed (no test changes yet).

- [ ] **Step 7: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(content): add projects, OSS, tech stack, stats, contact data"
git push origin main
```

---

### Task 5: Section wrapper component

**Files:**
- Create: `portfolio/src/design-system/components/Section.tsx`

- [ ] **Step 1: Implement Section**

Create `portfolio/src/design-system/components/Section.tsx`:

```typescript
import { forwardRef, type HTMLAttributes } from 'react';
import { cn } from '@/design-system/utils/cn';

export interface SectionProps extends HTMLAttributes<HTMLElement> {
  eyebrow?: string;
  title?: string;
  description?: string;
}

export const Section = forwardRef<HTMLElement, SectionProps>(
  ({ eyebrow, title, description, className, children, ...props }, ref) => (
    <section ref={ref} className={cn('scroll-mt-20 py-20', className)} {...props}>
      {(eyebrow || title || description) && (
        <div className="mb-10 max-w-2xl">
          {eyebrow && (
            <p className="font-mono text-sm uppercase tracking-wide text-[var(--color-brand-500)]">
              {eyebrow}
            </p>
          )}
          {title && (
            <h2 className="mt-2 font-mono text-3xl font-bold sm:text-4xl">{title}</h2>
          )}
          {description && (
            <p className="mt-3 text-[var(--muted)]">{description}</p>
          )}
        </div>
      )}
      {children}
    </section>
  ),
);
Section.displayName = 'Section';
```

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 3: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(design-system): add Section wrapper component"
git push origin main
```

---

### Task 6: Typewriter component with TDD

**Files:**
- Create: `portfolio/src/design-system/components/Typewriter.tsx`
- Create: `portfolio/tests/Typewriter.test.tsx`

- [ ] **Step 1: Write the failing test**

Create `portfolio/tests/Typewriter.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen, act } from '@testing-library/react';
import { Typewriter } from '@/design-system/components/Typewriter';

describe('Typewriter', () => {
  it('renders the first word immediately, statically, with reduced motion', () => {
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: () => ({
        matches: true,
        media: '(prefers-reduced-motion: reduce)',
        onchange: null,
        addEventListener: () => {},
        removeEventListener: () => {},
        addListener: () => {},
        removeListener: () => {},
        dispatchEvent: () => false,
      }),
    });
    render(<Typewriter words={['Full Stack', 'OSS']} />);
    expect(screen.getByText('Full Stack')).toBeInTheDocument();
  });

  it('renders the typed prefix while typing', async () => {
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: () => ({
        matches: false,
        media: '(prefers-reduced-motion: reduce)',
        onchange: null,
        addEventListener: () => {},
        removeEventListener: () => {},
        addListener: () => {},
        removeListener: () => {},
        dispatchEvent: () => false,
      }),
    });
    vi.useFakeTimers();
    render(<Typewriter words={['Hello']} typingSpeedMs={10} pauseMs={50} />);
    await act(async () => {
      vi.advanceTimersByTime(40);
    });
    const node = document.querySelector('[data-testid="typewriter"]');
    expect(node?.textContent ?? '').toContain('Hell');
    vi.useRealTimers();
  });

  it('renders an aria-live polite region for accessibility', () => {
    render(<Typewriter words={['x']} />);
    const node = document.querySelector('[data-testid="typewriter"]');
    expect(node?.getAttribute('aria-live')).toBe('polite');
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- Typewriter
```

Expected: module not found.

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/components/Typewriter.tsx`:

```typescript
'use client';

import { useEffect, useState } from 'react';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';
import { cn } from '@/design-system/utils/cn';

export interface TypewriterProps {
  words: string[];
  typingSpeedMs?: number;
  deletingSpeedMs?: number;
  pauseMs?: number;
  className?: string;
}

export function Typewriter({
  words,
  typingSpeedMs = 80,
  deletingSpeedMs = 40,
  pauseMs = 1500,
  className,
}: TypewriterProps) {
  const reduced = useReducedMotion();
  const [index, setIndex] = useState(0);
  const [text, setText] = useState('');
  const [phase, setPhase] = useState<'typing' | 'pausing' | 'deleting'>('typing');

  useEffect(() => {
    if (reduced) return;
    const current = words[index] ?? '';

    if (phase === 'typing') {
      if (text.length < current.length) {
        const t = setTimeout(() => setText(current.slice(0, text.length + 1)), typingSpeedMs);
        return () => clearTimeout(t);
      }
      const t = setTimeout(() => setPhase('deleting'), pauseMs);
      return () => clearTimeout(t);
    }

    if (phase === 'deleting') {
      if (text.length > 0) {
        const t = setTimeout(() => setText(text.slice(0, -1)), deletingSpeedMs);
        return () => clearTimeout(t);
      }
      setPhase('typing');
      setIndex((i) => (i + 1) % words.length);
    }
  }, [text, phase, index, words, reduced, typingSpeedMs, deletingSpeedMs, pauseMs]);

  if (reduced) {
    return (
      <span data-testid="typewriter" aria-live="polite" className={className}>
        {words[0] ?? ''}
      </span>
    );
  }

  return (
    <span data-testid="typewriter" aria-live="polite" className={cn('inline-block', className)}>
      {text}
      <span className="ml-0.5 inline-block h-[1em] w-[2px] -translate-y-[0.05em] animate-pulse bg-[var(--color-brand-500)] align-middle" />
    </span>
  );
}
```

- [ ] **Step 4: Add `vi` global import to test if needed**

If the test fails because `vi` is undefined, prepend `import { vi } from 'vitest';` to `tests/Typewriter.test.tsx`.

- [ ] **Step 5: Run — expect PASS**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- Typewriter
```

Expected: 3 passed.

- [ ] **Step 6: Run full suite**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test
```

Expected: 25 passed (22 + 3 new).

- [ ] **Step 7: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(design-system): add Typewriter component with reduced-motion fallback"
git push origin main
```

---

### Task 7: Hero section

**Files:**
- Create: `portfolio/src/sections/Hero.tsx`

- [ ] **Step 1: Implement**

Create `portfolio/src/sections/Hero.tsx`:

```typescript
import { Button } from '@/design-system/components/Button';
import { Badge } from '@/design-system/components/Badge';
import { Typewriter } from '@/design-system/components/Typewriter';
import { contact } from '@/content/contact';

const ROLES = ['Full Stack Developer', 'OSS Contributor', 'Bug Hunter', 'Tooling Builder'];

export function Hero() {
  return (
    <section className="py-20 sm:py-32">
      <div className="flex flex-wrap items-center gap-2 text-xs">
        <Badge variant="success">{contact.availability}</Badge>
        <Badge>{contact.location}</Badge>
      </div>

      <h1 className="mt-6 font-mono text-4xl font-bold leading-tight sm:text-6xl">
        Hi, I&apos;m <span className="text-[var(--color-brand-500)]">Maruthan</span>
        <span className="block text-[var(--muted)]">
          <Typewriter words={ROLES} />
        </span>
      </h1>

      <p className="mt-6 max-w-2xl text-lg text-[var(--muted)]">
        Full-stack developer at Finstein shipping production web and mobile apps across NestJS,
        Next.js, Angular, and React Native. Active OSS contributor — 57 merged PRs across VS Code,
        NestJS, Node.js undici, BullMQ, and more.
      </p>

      <div className="mt-8 flex flex-wrap gap-3">
        <Button asChild={false} onClick={undefined}>
          <a href="#projects">View work</a>
        </Button>
        <Button variant="outline">
          <a href="#contact">Get in touch</a>
        </Button>
        <Button variant="ghost">
          <a href={`mailto:${contact.email}`}>Email</a>
        </Button>
      </div>
    </section>
  );
}
```

NOTE on `<Button>` containing `<a>`: this is fine for visual styling but produces invalid HTML (button > a). For Plan 2 v1 we'll accept this; Plan 6 polish can introduce a polymorphic `Button` component (`asChild`) that renders the underlying tag correctly.

Better quick fix: render anchor-styled-as-button. Apply the Button's class directly to anchor tags. Refactor the implementation to:

```typescript
import { Typewriter } from '@/design-system/components/Typewriter';
import { Badge } from '@/design-system/components/Badge';
import { contact } from '@/content/contact';

const ROLES = ['Full Stack Developer', 'OSS Contributor', 'Bug Hunter', 'Tooling Builder'];

const buttonBase =
  'inline-flex h-10 items-center justify-center rounded-md px-4 text-base font-medium transition-colors';
const primaryStyles = `${buttonBase} bg-[var(--color-brand-500)] text-white hover:bg-[var(--color-brand-600)]`;
const outlineStyles = `${buttonBase} border border-[var(--border)] text-[var(--fg)] hover:bg-[var(--surface)]`;
const ghostStyles = `${buttonBase} text-[var(--fg)] hover:bg-[var(--surface)]`;

export function Hero() {
  return (
    <section className="py-20 sm:py-32">
      <div className="flex flex-wrap items-center gap-2 text-xs">
        <Badge variant="success">{contact.availability}</Badge>
        <Badge>{contact.location}</Badge>
      </div>

      <h1 className="mt-6 font-mono text-4xl font-bold leading-tight sm:text-6xl">
        Hi, I&apos;m <span className="text-[var(--color-brand-500)]">Maruthan</span>
        <span className="block text-[var(--muted)]">
          <Typewriter words={ROLES} />
        </span>
      </h1>

      <p className="mt-6 max-w-2xl text-lg text-[var(--muted)]">
        Full-stack developer at Finstein shipping production web and mobile apps across NestJS,
        Next.js, Angular, and React Native. Active OSS contributor — 57 merged PRs across VS Code,
        NestJS, Node.js undici, BullMQ, and more.
      </p>

      <div className="mt-8 flex flex-wrap gap-3">
        <a href="#projects" className={primaryStyles}>
          View work
        </a>
        <a href="#contact" className={outlineStyles}>
          Get in touch
        </a>
        <a href={`mailto:${contact.email}`} className={ghostStyles}>
          Email
        </a>
      </div>
    </section>
  );
}
```

Use the second implementation (anchor-styled-as-button) — that's the version to ship.

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 3: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(sections): add Hero with typewriter, badges, and CTA links"
git push origin main
```

---

### Task 8: StatCounter component with TDD

**Files:**
- Create: `portfolio/src/design-system/components/StatCounter.tsx`
- Create: `portfolio/tests/StatCounter.test.tsx`

- [ ] **Step 1: Write the failing test**

Create `portfolio/tests/StatCounter.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { StatCounter } from '@/design-system/components/StatCounter';

describe('StatCounter', () => {
  it('renders the final value text immediately when reduced motion', () => {
    Object.defineProperty(window, 'matchMedia', {
      writable: true,
      value: () => ({
        matches: true,
        media: '(prefers-reduced-motion: reduce)',
        onchange: null,
        addEventListener: () => {},
        removeEventListener: () => {},
      }),
    });
    render(<StatCounter value={57} suffix="+" label="Merged PRs" />);
    expect(screen.getByText('57+')).toBeInTheDocument();
    expect(screen.getByText('Merged PRs')).toBeInTheDocument();
  });

  it('exposes value and label as accessible text', () => {
    render(<StatCounter value={9} label="OSS projects" />);
    expect(screen.getByText('OSS projects')).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- StatCounter
```

Expected: module not found.

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/components/StatCounter.tsx`:

```typescript
'use client';

import { motion, useMotionValue, useSpring, useTransform } from 'framer-motion';
import { useInView } from 'react-intersection-observer';
import { useEffect } from 'react';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';

export interface StatCounterProps {
  value: number;
  label: string;
  suffix?: string;
}

export function StatCounter({ value, label, suffix = '' }: StatCounterProps) {
  const reduced = useReducedMotion();
  const { ref, inView } = useInView({ threshold: 0.4, triggerOnce: true });
  const motionValue = useMotionValue(0);
  const spring = useSpring(motionValue, { damping: 24, stiffness: 80 });
  const display = useTransform(spring, (v) => `${Math.round(v)}${suffix}`);

  useEffect(() => {
    if (reduced) return;
    if (inView) motionValue.set(value);
  }, [inView, value, motionValue, reduced]);

  return (
    <div ref={ref} className="space-y-1">
      <p className="font-mono text-3xl font-bold text-[var(--fg)] sm:text-4xl">
        {reduced ? `${value}${suffix}` : <motion.span>{display}</motion.span>}
      </p>
      <p className="text-sm text-[var(--muted)]">{label}</p>
    </div>
  );
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- StatCounter
```

Expected: 2 passed.

- [ ] **Step 5: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(design-system): add StatCounter with count-up and reduced-motion fallback"
git push origin main
```

---

### Task 9: About section

**Files:**
- Create: `portfolio/src/sections/About.tsx`

- [ ] **Step 1: Implement**

Create `portfolio/src/sections/About.tsx`:

```typescript
import { Section } from '@/design-system/components/Section';
import { StatCounter } from '@/design-system/components/StatCounter';
import { stats } from '@/content/stats';

export function About() {
  return (
    <Section id="about" eyebrow="01 / About" title="A few things about me">
      <div className="grid gap-12 lg:grid-cols-[1.5fr_1fr]">
        <div className="space-y-4 text-[var(--muted)]">
          <p>
            I'm a full-stack developer at <span className="text-[var(--fg)]">Finstein</span>, where
            I ship production web and mobile applications across NestJS, Next.js, Angular, and
            React Native (Expo). Before that I taught programming and robotics as a STEM
            instructor at LMES Academy.
          </p>
          <p>
            What I love most is improving the tools developers depend on. I&apos;m an active
            contributor to <span className="text-[var(--fg)]">VS Code</span>, the{' '}
            <span className="text-[var(--fg)]">NestJS ecosystem</span> (CLI, Swagger, GraphQL),{' '}
            <span className="text-[var(--fg)]">Node.js undici</span>,{' '}
            <span className="text-[var(--fg)]">BullMQ</span>, and a handful of other open source
            projects. 57 merged PRs and counting.
          </p>
          <p>
            Currently upskilling in <span className="text-[var(--fg)]">Data Engineering</span> —
            Databricks, PySpark, Delta Lake, dbt, and Power BI. Based in Chennai, India.
          </p>
        </div>

        <div className="grid grid-cols-2 gap-6 self-start">
          {stats.map((s) => (
            <StatCounter key={s.label} value={s.value} suffix={s.suffix} label={s.label} />
          ))}
        </div>
      </div>
    </Section>
  );
}
```

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 3: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(sections): add About with narrative and stat counters"
git push origin main
```

---

### Task 10: TechChip + TechStack section

**Files:**
- Create: `portfolio/src/design-system/components/TechChip.tsx`
- Create: `portfolio/src/sections/TechStack.tsx`

- [ ] **Step 1: Implement TechChip**

Create `portfolio/src/design-system/components/TechChip.tsx`:

```typescript
import { cn } from '@/design-system/utils/cn';
import type { Tech } from '@/content/techStack';

export function TechChip({ tech }: { tech: Tech }) {
  return (
    <span
      className={cn(
        'inline-flex items-center gap-1 rounded-md border px-2.5 py-1 font-mono text-xs',
        tech.learning
          ? 'border-[var(--color-warning)]/40 text-[var(--color-warning)]'
          : 'border-[var(--border)] text-[var(--fg)] hover:border-[var(--color-brand-500)]/60',
      )}
      title={tech.proficiency ?? (tech.learning ? 'Currently learning' : undefined)}
    >
      {tech.name}
      {tech.learning && <span aria-hidden> ◌</span>}
    </span>
  );
}
```

- [ ] **Step 2: Implement TechStack**

Create `portfolio/src/sections/TechStack.tsx`:

```typescript
import { Section } from '@/design-system/components/Section';
import { TechChip } from '@/design-system/components/TechChip';
import { techStack, techCategoriesInOrder } from '@/content/techStack';

export function TechStack() {
  return (
    <Section
      id="stack"
      eyebrow="02 / Stack"
      title="What I build with"
      description="Production-tested across multiple roles. Items marked ◌ are areas I'm currently learning."
    >
      <div className="space-y-8">
        {techCategoriesInOrder.map((cat) => {
          const items = techStack.filter((t) => t.category === cat);
          if (items.length === 0) return null;
          return (
            <div key={cat}>
              <h3 className="mb-3 font-mono text-sm uppercase tracking-wide text-[var(--muted)]">
                {cat}
              </h3>
              <div className="flex flex-wrap gap-2">
                {items.map((t) => (
                  <TechChip key={t.name} tech={t} />
                ))}
              </div>
            </div>
          );
        })}
      </div>
    </Section>
  );
}
```

- [ ] **Step 3: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 4: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(sections): add TechStack with categorized chips"
git push origin main
```

---

### Task 11: ProjectCard component

**Files:**
- Create: `portfolio/src/design-system/components/ProjectCard.tsx`

- [ ] **Step 1: Implement**

Create `portfolio/src/design-system/components/ProjectCard.tsx`:

```typescript
import { ArrowUpRight } from 'lucide-react';
import { Card } from '@/design-system/components/Card';
import { Badge } from '@/design-system/components/Badge';
import type { Project } from '@/content/projects';

const statusVariant: Record<Project['status'], 'default' | 'success' | 'warning' | 'error'> = {
  live: 'success',
  archived: 'default',
  oss: 'default',
  learning: 'warning',
};

const statusLabel: Record<Project['status'], string> = {
  live: 'Live',
  archived: 'Archived',
  oss: 'OSS',
  learning: 'Learning',
};

export function ProjectCard({ project }: { project: Project }) {
  return (
    <Card className="flex flex-col gap-4 transition-colors hover:border-[var(--color-brand-500)]/60">
      <div className="flex items-start justify-between gap-3">
        <h3 className="font-mono text-lg font-semibold leading-tight text-[var(--fg)]">
          {project.title}
        </h3>
        <Badge variant={statusVariant[project.status]}>{statusLabel[project.status]}</Badge>
      </div>

      <p className="text-sm text-[var(--muted)]">{project.summary}</p>

      <div className="flex flex-wrap gap-1.5">
        {project.tech.map((t) => (
          <span
            key={t}
            className="rounded-md border border-[var(--border)] px-2 py-0.5 font-mono text-[11px] text-[var(--muted)]"
          >
            {t}
          </span>
        ))}
      </div>

      {project.links && (
        <div className="mt-auto flex flex-wrap gap-3 text-sm">
          {project.links.live && (
            <a
              href={project.links.live}
              target="_blank"
              rel="noreferrer"
              className="inline-flex items-center gap-1 text-[var(--color-brand-500)] hover:underline"
            >
              Live <ArrowUpRight className="h-3.5 w-3.5" />
            </a>
          )}
          {project.links.repo && (
            <a
              href={project.links.repo}
              target="_blank"
              rel="noreferrer"
              className="inline-flex items-center gap-1 text-[var(--muted)] hover:text-[var(--fg)]"
            >
              GitHub <ArrowUpRight className="h-3.5 w-3.5" />
            </a>
          )}
          {project.links.case && (
            <a
              href={project.links.case}
              className="inline-flex items-center gap-1 text-[var(--muted)] hover:text-[var(--fg)]"
            >
              Case study →
            </a>
          )}
        </div>
      )}
    </Card>
  );
}
```

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 3: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(design-system): add ProjectCard component"
git push origin main
```

---

### Task 12: Featured Projects section

**Files:**
- Create: `portfolio/src/sections/FeaturedProjects.tsx`

- [ ] **Step 1: Implement**

Create `portfolio/src/sections/FeaturedProjects.tsx`:

```typescript
import { Section } from '@/design-system/components/Section';
import { ProjectCard } from '@/design-system/components/ProjectCard';
import { featuredProjects } from '@/content/projects';

export function FeaturedProjects() {
  return (
    <Section
      id="projects"
      eyebrow="03 / Work"
      title="Selected projects"
      description="A subset of what I've shipped. Detailed case studies with architecture notes will live at /projects."
    >
      <div className="grid gap-6 md:grid-cols-2">
        {featuredProjects.map((p) => (
          <ProjectCard key={p.slug} project={p} />
        ))}
      </div>
    </Section>
  );
}
```

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 3: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(sections): add Featured Projects grid"
git push origin main
```

---

### Task 13: OSS Preview section

**Files:**
- Create: `portfolio/src/sections/OSSPreview.tsx`

- [ ] **Step 1: Implement**

Create `portfolio/src/sections/OSSPreview.tsx`:

```typescript
import { ArrowUpRight } from 'lucide-react';
import { Section } from '@/design-system/components/Section';
import { Card } from '@/design-system/components/Card';
import { Badge } from '@/design-system/components/Badge';
import { ossStats, ossProjects, ossHighlights } from '@/content/oss';

export function OSSPreview() {
  return (
    <Section
      id="oss"
      eyebrow="04 / Open Source"
      title="Open source contributions"
      description="The work I'm proudest of. Full table with every PR will live at /oss."
    >
      <div className="grid gap-6 md:grid-cols-3">
        <Card>
          <p className="font-mono text-4xl font-bold text-[var(--color-brand-500)]">
            {ossStats.totalMerged}
          </p>
          <p className="text-sm text-[var(--muted)]">Merged PRs</p>
        </Card>
        <Card>
          <p className="font-mono text-4xl font-bold text-[var(--fg)]">{ossStats.totalOpen}</p>
          <p className="text-sm text-[var(--muted)]">Open PRs in flight</p>
        </Card>
        <Card>
          <p className="font-mono text-4xl font-bold text-[var(--fg)]">{ossStats.projectCount}</p>
          <p className="text-sm text-[var(--muted)]">OSS projects</p>
        </Card>
      </div>

      <div className="mt-10 grid gap-3">
        <h3 className="font-mono text-sm uppercase tracking-wide text-[var(--muted)]">
          Recent merged highlights
        </h3>
        {ossHighlights.map((h) => (
          <a
            key={h.href}
            href={h.href}
            target="_blank"
            rel="noreferrer"
            className="flex items-start justify-between gap-4 rounded-md border border-[var(--border)] bg-[var(--surface)] p-4 transition-colors hover:border-[var(--color-brand-500)]/60"
          >
            <div className="space-y-1">
              <div className="flex items-center gap-2 text-xs">
                <span className="font-mono text-[var(--color-brand-500)]">{h.project}</span>
                <Badge>{h.pr}</Badge>
              </div>
              <p className="text-sm text-[var(--fg)]">{h.title}</p>
            </div>
            <ArrowUpRight className="mt-1 h-4 w-4 shrink-0 text-[var(--muted)]" />
          </a>
        ))}
      </div>

      <div className="mt-10">
        <h3 className="mb-3 font-mono text-sm uppercase tracking-wide text-[var(--muted)]">
          Currently contributing to
        </h3>
        <div className="flex flex-wrap gap-2">
          {ossProjects.map((p) => (
            <a
              key={p.name}
              href={p.href}
              target="_blank"
              rel="noreferrer"
              className="rounded-md border border-[var(--border)] px-3 py-1.5 font-mono text-xs text-[var(--fg)] transition-colors hover:border-[var(--color-brand-500)]/60"
              title={p.focus}
            >
              {p.name}{' '}
              <span className="text-[var(--muted)]">
                ({p.merged} merged · {p.open} open)
              </span>
            </a>
          ))}
        </div>
      </div>
    </Section>
  );
}
```

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm run build
```

Both exit 0.

- [ ] **Step 3: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(sections): add OSS Preview with stats, highlights, and project list"
git push origin main
```

---

### Task 14: Contact server action with TDD

**Files:**
- Create: `portfolio/src/app/actions/contact.ts`
- Create: `portfolio/tests/contact-action.test.ts`

- [ ] **Step 1: Write the failing test**

Create `portfolio/tests/contact-action.test.ts`:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { sendContactMessage, contactSchema } from '@/app/actions/contact';

vi.mock('resend', () => ({
  Resend: class {
    emails = {
      send: vi.fn().mockResolvedValue({ data: { id: 'test' }, error: null }),
    };
  },
}));

describe('contactSchema', () => {
  it('accepts valid input', () => {
    const result = contactSchema.safeParse({
      name: 'Jane',
      email: 'jane@example.com',
      message: 'Hello there, this is a longer message body.',
    });
    expect(result.success).toBe(true);
  });

  it('rejects short message', () => {
    const result = contactSchema.safeParse({
      name: 'Jane',
      email: 'jane@example.com',
      message: 'short',
    });
    expect(result.success).toBe(false);
  });

  it('rejects invalid email', () => {
    const result = contactSchema.safeParse({
      name: 'Jane',
      email: 'not-an-email',
      message: 'A perfectly fine length message.',
    });
    expect(result.success).toBe(false);
  });
});

describe('sendContactMessage', () => {
  beforeEach(() => {
    delete process.env.RESEND_API_KEY;
  });

  it('returns a fallback status when RESEND_API_KEY is absent', async () => {
    const result = await sendContactMessage({
      name: 'Jane',
      email: 'jane@example.com',
      message: 'A perfectly fine length message.',
    });
    expect(result.status).toBe('fallback');
    expect(result.mailtoHref).toMatch(/^mailto:/);
  });

  it('returns ok status when RESEND_API_KEY is present', async () => {
    process.env.RESEND_API_KEY = 'test-key';
    const result = await sendContactMessage({
      name: 'Jane',
      email: 'jane@example.com',
      message: 'A perfectly fine length message.',
    });
    expect(result.status).toBe('ok');
  });

  it('returns error status when input is invalid', async () => {
    const result = await sendContactMessage({
      name: '',
      email: 'bad',
      message: 'x',
    });
    expect(result.status).toBe('error');
    expect(result.fieldErrors).toBeDefined();
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- contact-action
```

Expected: module not found.

- [ ] **Step 3: Implement**

Create `portfolio/src/app/actions/contact.ts`:

```typescript
'use server';

import { z } from 'zod';
import { Resend } from 'resend';
import { contact } from '@/content/contact';

export const contactSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Valid email required'),
  message: z.string().min(10, 'Message must be at least 10 characters'),
});

export type ContactInput = z.infer<typeof contactSchema>;

export type ContactResult =
  | { status: 'ok' }
  | { status: 'fallback'; mailtoHref: string }
  | { status: 'error'; fieldErrors?: Record<string, string[]>; message?: string };

export async function sendContactMessage(input: ContactInput): Promise<ContactResult> {
  const parsed = contactSchema.safeParse(input);
  if (!parsed.success) {
    return {
      status: 'error',
      fieldErrors: parsed.error.flatten().fieldErrors,
    };
  }

  const apiKey = process.env.RESEND_API_KEY;
  if (!apiKey) {
    const subject = encodeURIComponent(`Hello from ${parsed.data.name}`);
    const body = encodeURIComponent(parsed.data.message);
    return {
      status: 'fallback',
      mailtoHref: `mailto:${contact.email}?subject=${subject}&body=${body}`,
    };
  }

  try {
    const resend = new Resend(apiKey);
    await resend.emails.send({
      from: 'Portfolio Contact <onboarding@resend.dev>',
      to: contact.email,
      replyTo: parsed.data.email,
      subject: `[portfolio] ${parsed.data.name}`,
      text: `From: ${parsed.data.name} <${parsed.data.email}>\n\n${parsed.data.message}`,
    });
    return { status: 'ok' };
  } catch (err) {
    return {
      status: 'error',
      message: err instanceof Error ? err.message : 'Failed to send message',
    };
  }
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- contact-action
```

Expected: 6 passed.

- [ ] **Step 5: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(actions): add contact server action with Resend + mailto fallback"
git push origin main
```

---

### Task 15: ContactForm component with TDD + Contact section

**Files:**
- Create: `portfolio/src/design-system/components/ContactForm.tsx`
- Create: `portfolio/tests/ContactForm.test.tsx`
- Create: `portfolio/src/sections/Contact.tsx`

- [ ] **Step 1: Write the failing test**

Create `portfolio/tests/ContactForm.test.tsx`:

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ContactForm } from '@/design-system/components/ContactForm';

describe('ContactForm', () => {
  it('renders name, email, message fields and submit button', () => {
    render(<ContactForm action={vi.fn()} />);
    expect(screen.getByLabelText(/name/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/message/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /send message/i })).toBeInTheDocument();
  });

  it('shows validation error on short message', async () => {
    const user = userEvent.setup();
    render(<ContactForm action={vi.fn()} />);
    await user.type(screen.getByLabelText(/name/i), 'Jane');
    await user.type(screen.getByLabelText(/email/i), 'jane@example.com');
    await user.type(screen.getByLabelText(/message/i), 'short');
    await user.click(screen.getByRole('button', { name: /send message/i }));
    expect(await screen.findByText(/at least 10 characters/i)).toBeInTheDocument();
  });

  it('calls action with valid input and shows success state', async () => {
    const action = vi.fn().mockResolvedValue({ status: 'ok' });
    const user = userEvent.setup();
    render(<ContactForm action={action} />);
    await user.type(screen.getByLabelText(/name/i), 'Jane');
    await user.type(screen.getByLabelText(/email/i), 'jane@example.com');
    await user.type(screen.getByLabelText(/message/i), 'Hello there, this is plenty long enough.');
    await user.click(screen.getByRole('button', { name: /send message/i }));
    expect(await screen.findByText(/thanks/i)).toBeInTheDocument();
    expect(action).toHaveBeenCalledWith({
      name: 'Jane',
      email: 'jane@example.com',
      message: 'Hello there, this is plenty long enough.',
    });
  });

  it('shows mailto fallback link when action returns fallback', async () => {
    const action = vi.fn().mockResolvedValue({
      status: 'fallback',
      mailtoHref: 'mailto:test@example.com',
    });
    const user = userEvent.setup();
    render(<ContactForm action={action} />);
    await user.type(screen.getByLabelText(/name/i), 'Jane');
    await user.type(screen.getByLabelText(/email/i), 'jane@example.com');
    await user.type(screen.getByLabelText(/message/i), 'Hello there, this is plenty long enough.');
    await user.click(screen.getByRole('button', { name: /send message/i }));
    const link = await screen.findByRole('link', { name: /open email client/i });
    expect(link).toHaveAttribute('href', 'mailto:test@example.com');
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- ContactForm
```

Expected: module not found.

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/components/ContactForm.tsx`:

```typescript
'use client';

import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { cn } from '@/design-system/utils/cn';

const schema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Valid email required'),
  message: z.string().min(10, 'Message must be at least 10 characters'),
});

type FormValues = z.infer<typeof schema>;

type ActionResult =
  | { status: 'ok' }
  | { status: 'fallback'; mailtoHref: string }
  | { status: 'error'; message?: string };

interface ContactFormProps {
  action: (input: FormValues) => Promise<ActionResult>;
}

const fieldClass =
  'w-full rounded-md border border-[var(--border)] bg-[var(--surface)] px-3 py-2 text-sm text-[var(--fg)] placeholder:text-[var(--muted)] focus:border-[var(--color-brand-500)] focus:outline-none';

export function ContactForm({ action }: ContactFormProps) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
  } = useForm<FormValues>({ resolver: zodResolver(schema) });

  const [result, setResult] = useState<ActionResult | null>(null);

  const onSubmit = handleSubmit(async (values) => {
    const r = await action(values);
    setResult(r);
    if (r.status === 'ok') reset();
  });

  if (result?.status === 'ok') {
    return (
      <p className="rounded-md border border-[var(--color-success)]/30 bg-[var(--color-success)]/10 p-4 text-sm text-[var(--color-success)]">
        Thanks — your message is on its way.
      </p>
    );
  }

  return (
    <form onSubmit={onSubmit} className="space-y-4" noValidate>
      <div className="space-y-1">
        <label htmlFor="name" className="text-sm font-medium text-[var(--fg)]">
          Name
        </label>
        <input id="name" type="text" autoComplete="name" className={fieldClass} {...register('name')} />
        {errors.name && <p className="text-xs text-[var(--color-error)]">{errors.name.message}</p>}
      </div>

      <div className="space-y-1">
        <label htmlFor="email" className="text-sm font-medium text-[var(--fg)]">
          Email
        </label>
        <input id="email" type="email" autoComplete="email" className={fieldClass} {...register('email')} />
        {errors.email && <p className="text-xs text-[var(--color-error)]">{errors.email.message}</p>}
      </div>

      <div className="space-y-1">
        <label htmlFor="message" className="text-sm font-medium text-[var(--fg)]">
          Message
        </label>
        <textarea
          id="message"
          rows={5}
          className={cn(fieldClass, 'resize-y')}
          {...register('message')}
        />
        {errors.message && <p className="text-xs text-[var(--color-error)]">{errors.message.message}</p>}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="inline-flex h-10 items-center justify-center rounded-md bg-[var(--color-brand-500)] px-4 text-base font-medium text-white transition-colors hover:bg-[var(--color-brand-600)] disabled:opacity-50"
      >
        {isSubmitting ? 'Sending…' : 'Send message'}
      </button>

      {result?.status === 'fallback' && (
        <p className="text-xs text-[var(--muted)]">
          Email isn&apos;t configured here yet —{' '}
          <a href={result.mailtoHref} className="text-[var(--color-brand-500)] hover:underline">
            open email client
          </a>{' '}
          to send directly.
        </p>
      )}

      {result?.status === 'error' && result.message && (
        <p className="text-xs text-[var(--color-error)]">{result.message}</p>
      )}
    </form>
  );
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- ContactForm
```

Expected: 4 passed.

- [ ] **Step 5: Implement Contact section**

Create `portfolio/src/sections/Contact.tsx`:

```typescript
import { Section } from '@/design-system/components/Section';
import { ContactForm } from '@/design-system/components/ContactForm';
import { sendContactMessage } from '@/app/actions/contact';
import { contact } from '@/content/contact';

export function Contact() {
  return (
    <Section
      id="contact"
      eyebrow="05 / Contact"
      title="Let's talk"
      description="Hiring, collaboration, or just saying hi — all welcome."
    >
      <div className="grid gap-12 lg:grid-cols-[1fr_1.2fr]">
        <div className="space-y-6">
          <p className="text-[var(--muted)]">
            Direct line:{' '}
            <a
              href={`mailto:${contact.email}`}
              className="text-[var(--color-brand-500)] hover:underline"
            >
              {contact.email}
            </a>
          </p>
          <div className="space-y-2">
            <h3 className="font-mono text-sm uppercase tracking-wide text-[var(--muted)]">
              Find me elsewhere
            </h3>
            <ul className="space-y-1 text-sm">
              {contact.socials.map((s) => (
                <li key={s.label}>
                  <a
                    href={s.href}
                    target="_blank"
                    rel="noreferrer"
                    className="text-[var(--fg)] hover:text-[var(--color-brand-500)]"
                  >
                    {s.label} <span className="text-[var(--muted)]">— {s.handle}</span>
                  </a>
                </li>
              ))}
            </ul>
          </div>
        </div>

        <ContactForm action={sendContactMessage} />
      </div>
    </Section>
  );
}
```

- [ ] **Step 6: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm test
npm run build
```

All exit 0. Tests should now show 39 passed (25 + 4 ContactForm + 6 contact-action + 4 prior new from earlier tasks). Adjust if your count differs — the important thing is no regressions.

- [ ] **Step 7: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(sections): add Contact section with form, validation, and mailto fallback"
git push origin main
```

---

### Task 16: Compose home page from sections

**Files:**
- Modify: `portfolio/src/app/page.tsx`

- [ ] **Step 1: Replace `page.tsx`**

Overwrite `portfolio/src/app/page.tsx`:

```typescript
import { Hero } from '@/sections/Hero';
import { About } from '@/sections/About';
import { TechStack } from '@/sections/TechStack';
import { FeaturedProjects } from '@/sections/FeaturedProjects';
import { OSSPreview } from '@/sections/OSSPreview';
import { Contact } from '@/sections/Contact';

export default function HomePage() {
  return (
    <>
      <Hero />
      <About />
      <TechStack />
      <FeaturedProjects />
      <OSSPreview />
      <Contact />
    </>
  );
}
```

- [ ] **Step 2: Verify dev server**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run dev
```

In background. Then `curl -s http://localhost:3000 | grep -E "Hi, I|Stack|Selected projects|Open source|Let's talk"` — should return at least 5 distinct hits. Kill server.

- [ ] **Step 3: Verify build**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run build
```

Exit 0.

- [ ] **Step 4: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "feat(home): compose home page from Hero, About, TechStack, Projects, OSS, Contact"
git push origin main
```

---

### Task 17: Polish footer + update e2e + final verify

**Files:**
- Modify: `portfolio/src/design-system/layout/Footer.tsx`
- Replace: `portfolio/tests/e2e/theme-toggle.spec.ts` → `portfolio/tests/e2e/home.spec.ts`

- [ ] **Step 1: Update Footer**

Replace `portfolio/src/design-system/layout/Footer.tsx`:

```typescript
import { contact } from '@/content/contact';

export function Footer() {
  const year = new Date().getFullYear();
  const built = process.env.NEXT_PUBLIC_BUILD_TIME ?? new Date().toISOString().slice(0, 10);

  return (
    <footer className="border-t border-[var(--border)] py-10 text-sm text-[var(--muted)]">
      <div className="mx-auto grid max-w-5xl gap-6 px-4 sm:grid-cols-3">
        <div>
          <p className="font-mono text-[var(--fg)]">
            maruthan<span className="text-[var(--color-brand-500)]">.dev</span>
          </p>
          <p className="mt-2">© {year} Maruthan G</p>
        </div>
        <div>
          <p className="font-mono text-xs uppercase tracking-wide">Connect</p>
          <ul className="mt-2 space-y-1">
            {contact.socials.map((s) => (
              <li key={s.label}>
                <a href={s.href} target="_blank" rel="noreferrer" className="hover:text-[var(--fg)]">
                  {s.label}
                </a>
              </li>
            ))}
          </ul>
        </div>
        <div>
          <p className="font-mono text-xs uppercase tracking-wide">Built with</p>
          <p className="mt-2">Next.js · TypeScript · Tailwind v4</p>
          <p>
            <a
              href="https://github.com/maruthang/portfolio"
              target="_blank"
              rel="noreferrer"
              className="hover:text-[var(--fg)]"
            >
              View source
            </a>{' '}
            · Last build {built}
          </p>
        </div>
      </div>
    </footer>
  );
}
```

- [ ] **Step 2: Replace e2e test**

Delete `portfolio/tests/e2e/theme-toggle.spec.ts` and create `portfolio/tests/e2e/home.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test('home page renders all major sections', async ({ page }) => {
  await page.goto('/');
  await expect(page.getByRole('heading', { level: 1 })).toContainText('Maruthan');
  await expect(page.getByRole('heading', { name: /a few things about me/i })).toBeVisible();
  await expect(page.getByRole('heading', { name: /what i build with/i })).toBeVisible();
  await expect(page.getByRole('heading', { name: /selected projects/i })).toBeVisible();
  await expect(page.getByRole('heading', { name: /open source contributions/i })).toBeVisible();
  await expect(page.getByRole('heading', { name: /let's talk/i })).toBeVisible();
});

test('theme toggle flips html class', async ({ page }) => {
  await page.goto('/');
  const toggle = page.getByRole('button', { name: /toggle theme/i });
  await toggle.click();
  const htmlClass = await page.locator('html').getAttribute('class');
  expect(htmlClass).toMatch(/dark|light/);
});

test('contact form shows validation error on short message', async ({ page }) => {
  await page.goto('/#contact');
  await page.getByLabel(/name/i).fill('Jane');
  await page.getByLabel(/email/i).fill('jane@example.com');
  await page.getByLabel(/message/i).fill('short');
  await page.getByRole('button', { name: /send message/i }).click();
  await expect(page.getByText(/at least 10 characters/i)).toBeVisible();
});

test('contact form fallback shows mailto link without RESEND_API_KEY', async ({ page }) => {
  await page.goto('/#contact');
  await page.getByLabel(/name/i).fill('Jane');
  await page.getByLabel(/email/i).fill('jane@example.com');
  await page.getByLabel(/message/i).fill('A perfectly fine length message body.');
  await page.getByRole('button', { name: /send message/i }).click();
  await expect(page.getByRole('link', { name: /open email client/i })).toBeVisible();
});
```

- [ ] **Step 3: Run all tests**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck
npm test
npm run build
npm run test:e2e
```

All exit 0.

- [ ] **Step 4: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "test(e2e): cover all home sections, contact form validation, and mailto fallback"
git push origin main
```

- [ ] **Step 5: Verify Vercel deploy**

After CI goes green, Vercel auto-deploys. Open `https://portfolio-tawny-two-72.vercel.app/` and confirm:
- Hero with typewriter (cycles roles)
- About with stat counters (animate on scroll into view)
- Tech Stack chips by category
- Featured projects grid (6 cards)
- OSS preview with stats and PR highlights
- Contact form (submitting without env var shows mailto fallback link)
- Footer with three columns

If anything is broken in production but works locally, debug via Vercel build logs.

---

## Self-Review

**Spec coverage** — checked against `docs/superpowers/specs/2026-04-25-portfolio-design.md`:

| Spec section | Covered by |
|---|---|
| §5.1.1 Hero (typewriter, badges, CTAs) | Tasks 6, 7 |
| §5.1.2 About (narrative + stat counters) | Tasks 8, 9 |
| §5.1.3 Tech Stack (categorized chips) | Task 10 |
| §5.1.4 Featured Projects (cards) | Tasks 11, 12 |
| §5.1.5 OSS preview (stats + highlights) | Task 13 |
| §5.1.6 Contact (form + Resend) | Tasks 14, 15 |
| §5.1.7 Footer (sitemap, built-with) | Task 17 |
| §7 Animation library mix (Framer Motion + Lenis) | Tasks 1, 3 |
| §7 Reduced-motion respected globally | Tasks 2, 6, 8 |

**Deferred to later plans (intentional):**
- Project detail pages (case studies) → Plan 3
- Full OSS table at `/oss` → Plan 3
- Blog system → Plan 4
- Command palette, /uses, /now, /resume, OG images → Plan 5
- Custom domain, SEO, analytics → Plan 6

**Placeholder scan:** None. Every step contains runnable code.

**Type consistency:** `Project`, `Tech`, `OssProject`, `OssHighlight`, `Stat`, `Social`, `ContactInput`, `ContactResult` defined once in their respective `content/*.ts` or `actions/contact.ts` files; consumed by section/component files via imports.

**Open clarifications before execution:**
- Avatar/photo: not included in this plan (the About section is text-only). If you want a photo, drop the file at `public/avatar.jpg` and we can add it in a single follow-up task.
- The Hero "View work" / "Get in touch" anchors target `#projects` and `#contact` — the Section components have matching `id` props.

---

## Definition of Done for Plan 2

- ✅ All 6 home sections render in production at `https://portfolio-tawny-two-72.vercel.app/`.
- ✅ Typewriter cycles through 4 roles in the hero.
- ✅ Stat counters animate from 0 → final value on scroll into view (and skip the animation under `prefers-reduced-motion`).
- ✅ Tech stack chips render in 6 categories.
- ✅ 6 featured project cards render with status badges and tech badges.
- ✅ OSS preview shows the 57/103/9 stats card, 3 PR highlights, and the "Currently contributing to" project list.
- ✅ Contact form validates client-side, calls the server action, and shows a mailto fallback when `RESEND_API_KEY` is absent.
- ✅ Footer shows the three-column layout (brand, connect, built-with).
- ✅ All unit tests pass (~40 tests expected).
- ✅ Playwright e2e covers home rendering, theme toggle, contact form validation, and mailto fallback (4 tests).
- ✅ TypeScript strict, Prettier clean, CI green on `main`.
- ✅ No regressions in Plan 1 functionality (header still sticky, theme toggle still works).
