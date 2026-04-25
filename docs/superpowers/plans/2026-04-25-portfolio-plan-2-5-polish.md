# Portfolio Plan 2.5 — Visual Polish & Motion Richness Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Close the polish + motion gap with reference portfolios `yogeshwaran.com` (heavy GSAP, custom cursor, boot loader, horizontal-scroll projects) and `portfolio-frontend-ruby-delta-31.vercel.app` (theme palette picker, mobile drawer, fluid type, animated status ping). After this plan ships, the live site visibly competes with Yogeshwaran's at first glance.

**Architecture:** Add GSAP alongside the existing Framer Motion (Framer for component motion, GSAP for scroll choreography that pins/scrubs). Introduce a small `motion/` toolkit of reusable hooks (`useMagneticHover`, scroll variants) and visual primitives (BootLoader, CustomCursor, HeroBackground, TechMarquee). All animations gated by `useReducedMotion()` — no exceptions. Touch devices skip cursor + heavy effects via `(hover: none)` media query. Lighthouse target: stay ≥95 on Performance, Accessibility, Best Practices, SEO.

**Tech additions:** `gsap` (3.x — already loaded as plugin in spec but not installed yet).

**Output:** A live site at `https://portfolio-tawny-two-72.vercel.app/` with a boot loader, custom cursor, animated hero background, line-by-line hero name reveal, marquee tech bar, scroll-revealing sections, magnetic project cards, horizontal-scroll featured projects (desktop), theme palette picker, mobile drawer, animated availability ping, back-to-top, skip-to-content, and tightened focus rings — without breaking any existing test or regressing Lighthouse.

**Order rationale:** Tasks 1–8 are low-risk polish (a11y + simple CSS). Tasks 9–13 add visual richness with libraries we already have. Tasks 14–17 introduce the higher-risk effects (cursor, loader, palette dialog, GSAP horizontal scroll). Task 18 is the regression sweep.

---

## File Structure (additions on top of Plan 2 state)

```
src/
  app/
    layout.tsx                                ← add skip-link, BootLoader, CustomCursor
    globals.css                               ← add @keyframes (marquee, twinkle, fluid type, focus-visible)
  design-system/
    components/
      Monogram.tsx                            ← (NEW) "M" mark
      MobileDrawer.tsx                        ← (NEW) replaces inline mobile nav
      BackToTop.tsx                           ← (NEW)
      AvailabilityPing.tsx                    ← (NEW) green pulse + status text
      BootLoader.tsx                          ← (NEW) counter + bar
      CustomCursor.tsx                        ← (NEW) blend-mode cursor + dot
      ThemePalette.tsx                        ← (NEW) accent color picker
    motion/
      useMagneticHover.ts                     ← (NEW) cursor-pull hook for cards
      useGsap.ts                              ← (NEW) GSAP context wrapper for cleanup
      variants.ts                             ← extend with sectionReveal
    sections/
      Hero.tsx                                ← (MODIFY) line reveals + bg + ping
      TechStack.tsx                           ← (MODIFY) prepend marquee
      FeaturedProjects.tsx                    ← (MODIFY) wrap with HorizontalScroll
    visuals/                                  ← (NEW)
      HeroBackground.tsx                      ← grid + glows + stars
      TechMarquee.tsx                         ← auto-scrolling tech list
      HorizontalScrollProjects.tsx            ← GSAP-pinned projects (desktop), grid (mobile)
    layout/
      Header.tsx                              ← (MODIFY) Monogram + ThemePalette + drawer trigger
      Footer.tsx                              ← (MODIFY) BackToTop + Monogram
tests/
  Monogram.test.tsx                           ← (NEW)
  MobileDrawer.test.tsx                       ← (NEW)
  AvailabilityPing.test.tsx                   ← (NEW)
  ThemePalette.test.tsx                       ← (NEW)
  useMagneticHover.test.ts                    ← (NEW)
  e2e/
    home.spec.ts                              ← (MODIFY) add tests for drawer, palette, back-to-top
```

---

## Conventions

- Package manager: **npm**.
- Commit style: Conventional Commits.
- Branch: `main`. Push at end of each task.
- **No `Co-Authored-By: Claude` trailer in commit messages** (user preference, durable rule).
- Run all 4 CI commands locally before push: `npm run typecheck && npm run format:check && npm test && npm run build`.
- Watch CI green: `gh run watch --repo maruthang/portfolio --exit-status`.
- Reduced-motion: every animation hook MUST short-circuit when `useReducedMotion()` returns true.
- Touch devices: cursor + magnetic hover MUST short-circuit when `(hover: none)` matches.

---

### Task 1: Install GSAP

**Files:**
- Modify: `portfolio/package.json`, `portfolio/package-lock.json`

- [ ] **Step 1: Install**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm install gsap
```

- [ ] **Step 2: Verify version landed**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm ls gsap
```

GSAP 3.x expected.

- [ ] **Step 3: Verify nothing broke**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck && npm run format:check && npm test && npm run build
```

All exit 0. Tests still 41 unit / 4 e2e passing.

- [ ] **Step 4: Commit and push**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
git add -A
git commit -m "chore(deps): add gsap for scroll-driven animations"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 2: Add skip-to-content link

**Files:**
- Modify: `portfolio/src/app/layout.tsx`
- Modify: `portfolio/src/app/globals.css`

- [ ] **Step 1: Add skip-link CSS**

Append to `portfolio/src/app/globals.css`:

```css
/* Skip-to-content link — visible only on keyboard focus */
.skip-link {
  position: absolute;
  top: -100px;
  left: 0.5rem;
  z-index: 100;
  padding: 0.5rem 1rem;
  background: var(--color-brand-500);
  color: white;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  font-weight: 500;
  transition: top 150ms cubic-bezier(0.4, 0, 0.2, 1);
}
.skip-link:focus-visible {
  top: 0.5rem;
  outline: 2px solid var(--color-brand-500);
  outline-offset: 2px;
}
```

- [ ] **Step 2: Wire into root layout**

Modify `portfolio/src/app/layout.tsx` — inside `<body>`, before `<Providers>`, add:

```tsx
<a href="#main-content" className="skip-link">
  Skip to main content
</a>
```

And wrap the existing `<main>` element body in a wrapper with `id="main-content"` (or add the id to `<main>` directly if it doesn't already have one).

- [ ] **Step 3: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck && npm run format:check && npm test && npm run build
```

All exit 0.

- [ ] **Step 4: Commit and push**

```bash
git add -A
git commit -m "feat(a11y): add skip-to-content link"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 3: Brand monogram component

**Files:**
- Create: `portfolio/src/design-system/components/Monogram.tsx`
- Create: `portfolio/tests/Monogram.test.tsx`

- [ ] **Step 1: Write failing test**

Create `portfolio/tests/Monogram.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { Monogram } from '@/design-system/components/Monogram';

describe('Monogram', () => {
  it('renders the M letter', () => {
    render(<Monogram />);
    expect(screen.getByText('M')).toBeInTheDocument();
  });

  it('marks itself decorative for screen readers', () => {
    render(<Monogram />);
    const node = screen.getByText('M').parentElement;
    expect(node?.getAttribute('aria-hidden')).toBe('true');
  });

  it('forwards a className when provided', () => {
    const { container } = render(<Monogram className="custom-class" />);
    expect(container.firstChild).toHaveClass('custom-class');
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm test -- Monogram
```

Module not found.

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/components/Monogram.tsx`:

```typescript
import { cn } from '@/design-system/utils/cn';

export function Monogram({ className }: { className?: string }) {
  return (
    <span
      aria-hidden="true"
      className={cn(
        'inline-flex h-9 w-9 items-center justify-center rounded-md bg-[var(--color-brand-500)] font-mono text-base font-bold text-white',
        className,
      )}
    >
      <span>M</span>
    </span>
  );
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
npm test -- Monogram
```

3 passed.

- [ ] **Step 5: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

All exit 0. Tests now 44 unit (41 + 3 new).

- [ ] **Step 6: Commit and push**

```bash
git add -A
git commit -m "feat(design-system): add Monogram brand mark"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 4: Fluid typography utilities

**Files:**
- Modify: `portfolio/src/app/globals.css`

- [ ] **Step 1: Extend the `@theme` block in globals.css**

In `portfolio/src/app/globals.css`, inside the existing `@theme { ... }` block, add fluid type scale entries:

```css
  /* Fluid font sizes via clamp(min, preferred, max) */
  --text-fluid-sm: clamp(0.875rem, 0.85rem + 0.125vw, 1rem);
  --text-fluid-base: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  --text-fluid-lg: clamp(1.125rem, 1rem + 0.625vw, 1.375rem);
  --text-fluid-xl: clamp(1.25rem, 1.1rem + 0.75vw, 1.625rem);
  --text-fluid-2xl: clamp(1.5rem, 1.3rem + 1vw, 2rem);
  --text-fluid-3xl: clamp(1.875rem, 1.5rem + 1.875vw, 3rem);
  --text-fluid-4xl: clamp(2.25rem, 1.75rem + 2.5vw, 3.75rem);
  --text-fluid-5xl: clamp(3rem, 2rem + 5vw, 5rem);
  --text-fluid-6xl: clamp(3.75rem, 2.5rem + 6.25vw, 6rem);
```

These auto-generate Tailwind utilities `text-fluid-sm` through `text-fluid-6xl` in v4.

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck && npm run format:check && npm test && npm run build
```

All exit 0.

- [ ] **Step 3: Commit and push**

```bash
git add -A
git commit -m "feat(design-system): add fluid typography clamp() utilities"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 5: Tighter focus-visible styling

**Files:**
- Modify: `portfolio/src/app/globals.css`

- [ ] **Step 1: Append focus-visible base**

Append to `portfolio/src/app/globals.css`:

```css
/* Universal focus-visible — replaces default browser outline */
*:focus-visible {
  outline: 2px solid var(--color-brand-500);
  outline-offset: 2px;
  border-radius: 0.25rem;
}

/* Remove default outline for elements that have their own focus treatment */
button:focus-visible,
a:focus-visible,
input:focus-visible,
textarea:focus-visible {
  outline: 2px solid var(--color-brand-500);
  outline-offset: 2px;
}
```

- [ ] **Step 2: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 3: Commit and push**

```bash
git add -A
git commit -m "feat(a11y): unify focus-visible ring across all interactive elements"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 6: Back-to-top button

**Files:**
- Create: `portfolio/src/design-system/components/BackToTop.tsx`
- Modify: `portfolio/src/design-system/layout/Footer.tsx`

- [ ] **Step 1: Implement BackToTop**

Create `portfolio/src/design-system/components/BackToTop.tsx`:

```typescript
'use client';

import { ArrowUp } from 'lucide-react';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';
import { cn } from '@/design-system/utils/cn';

export function BackToTop({ className }: { className?: string }) {
  const reduced = useReducedMotion();

  const onClick = () => {
    window.scrollTo({ top: 0, behavior: reduced ? 'auto' : 'smooth' });
  };

  return (
    <button
      type="button"
      aria-label="Back to top"
      onClick={onClick}
      className={cn(
        'inline-flex h-9 w-9 items-center justify-center rounded-md border border-[var(--border)] bg-[var(--surface)] text-[var(--fg)] transition-colors hover:border-[var(--color-brand-500)]/60 hover:text-[var(--color-brand-500)]',
        className,
      )}
    >
      <ArrowUp className="h-4 w-4" />
    </button>
  );
}
```

- [ ] **Step 2: Wire into Footer**

Modify `portfolio/src/design-system/layout/Footer.tsx` — replace the rightmost column ("Built with") to include the BackToTop button beside the "View source" link. Read the current file first, then add `<BackToTop />` at an appropriate position.

- [ ] **Step 3: Verify**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 4: Commit and push**

```bash
git add -A
git commit -m "feat(design-system): add BackToTop button in Footer"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 7: Animated availability ping

**Files:**
- Create: `portfolio/src/design-system/components/AvailabilityPing.tsx`
- Create: `portfolio/tests/AvailabilityPing.test.tsx`
- Modify: `portfolio/src/sections/Hero.tsx`

- [ ] **Step 1: Write failing test**

Create `portfolio/tests/AvailabilityPing.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { AvailabilityPing } from '@/design-system/components/AvailabilityPing';

describe('AvailabilityPing', () => {
  it('renders the status text', () => {
    render(<AvailabilityPing>Open to opportunities</AvailabilityPing>);
    expect(screen.getByText('Open to opportunities')).toBeInTheDocument();
  });

  it('renders both ping halo and solid dot for live status', () => {
    const { container } = render(<AvailabilityPing>Live</AvailabilityPing>);
    const ping = container.querySelector('[data-testid="ping-halo"]');
    const dot = container.querySelector('[data-testid="ping-dot"]');
    expect(ping).toBeInTheDocument();
    expect(dot).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
npm test -- AvailabilityPing
```

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/components/AvailabilityPing.tsx`:

```typescript
import type { ReactNode } from 'react';
import { cn } from '@/design-system/utils/cn';

export function AvailabilityPing({
  children,
  className,
}: {
  children: ReactNode;
  className?: string;
}) {
  return (
    <span
      className={cn(
        'inline-flex items-center gap-2 rounded-full border border-[var(--color-success)]/30 bg-[var(--color-success)]/10 px-3 py-1 text-xs font-medium text-[var(--color-success)]',
        className,
      )}
    >
      <span className="relative flex h-2.5 w-2.5">
        <span
          data-testid="ping-halo"
          aria-hidden="true"
          className="absolute inline-flex h-full w-full animate-ping rounded-full bg-[var(--color-success)] opacity-75"
        />
        <span
          data-testid="ping-dot"
          aria-hidden="true"
          className="relative inline-flex h-2.5 w-2.5 rounded-full bg-[var(--color-success)]"
        />
      </span>
      {children}
    </span>
  );
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
npm test -- AvailabilityPing
```

2 passed.

- [ ] **Step 5: Use in Hero**

Modify `portfolio/src/sections/Hero.tsx` — replace the `<Badge variant="success">{contact.availability}</Badge>` with `<AvailabilityPing>{contact.availability}</AvailabilityPing>`. Keep the location Badge as is.

- [ ] **Step 6: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

Tests: 46 unit (44 + 2 new).

- [ ] **Step 7: Commit and push**

```bash
git add -A
git commit -m "feat(design-system): add AvailabilityPing with animated halo, use in Hero"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 8: Mobile drawer dialog

**Files:**
- Create: `portfolio/src/design-system/components/MobileDrawer.tsx`
- Create: `portfolio/tests/MobileDrawer.test.tsx`
- Modify: `portfolio/src/design-system/layout/Header.tsx`

- [ ] **Step 1: Write failing test**

Create `portfolio/tests/MobileDrawer.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MobileDrawer } from '@/design-system/components/MobileDrawer';

const links = [
  { href: '#projects', label: 'Projects' },
  { href: '#blog', label: 'Blog' },
];

describe('MobileDrawer', () => {
  it('is closed by default', () => {
    render(<MobileDrawer links={links} />);
    expect(screen.queryByRole('dialog')).not.toBeInTheDocument();
  });

  it('opens when the toggle button is clicked', async () => {
    const user = userEvent.setup();
    render(<MobileDrawer links={links} />);
    await user.click(screen.getByRole('button', { name: /open menu/i }));
    expect(screen.getByRole('dialog', { name: /mobile navigation/i })).toBeInTheDocument();
    expect(screen.getByRole('link', { name: 'Projects' })).toBeInTheDocument();
  });

  it('closes when the close button is clicked', async () => {
    const user = userEvent.setup();
    render(<MobileDrawer links={links} />);
    await user.click(screen.getByRole('button', { name: /open menu/i }));
    await user.click(screen.getByRole('button', { name: /close menu/i }));
    expect(screen.queryByRole('dialog')).not.toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
npm test -- MobileDrawer
```

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/components/MobileDrawer.tsx`:

```typescript
'use client';

import { useEffect, useState } from 'react';
import { Menu, X } from 'lucide-react';
import { cn } from '@/design-system/utils/cn';

interface DrawerLink {
  href: string;
  label: string;
}

export function MobileDrawer({ links }: { links: DrawerLink[] }) {
  const [open, setOpen] = useState(false);

  useEffect(() => {
    if (!open) return;
    const onKey = (e: KeyboardEvent) => {
      if (e.key === 'Escape') setOpen(false);
    };
    window.addEventListener('keydown', onKey);
    document.body.style.overflow = 'hidden';
    return () => {
      window.removeEventListener('keydown', onKey);
      document.body.style.overflow = '';
    };
  }, [open]);

  return (
    <>
      <button
        type="button"
        aria-label="Open menu"
        aria-expanded={open}
        aria-controls="mobile-drawer"
        onClick={() => setOpen(true)}
        className="inline-flex h-9 w-9 items-center justify-center rounded-md text-[var(--fg)] hover:bg-[var(--surface)] md:hidden"
      >
        <Menu className="h-5 w-5" />
      </button>

      {open && (
        <>
          <div
            aria-hidden="true"
            onClick={() => setOpen(false)}
            className="fixed inset-0 z-40 bg-black/40 backdrop-blur-sm md:hidden"
          />
          <div
            id="mobile-drawer"
            role="dialog"
            aria-modal="true"
            aria-label="Mobile navigation"
            className={cn(
              'fixed right-0 top-0 z-50 flex h-full w-full max-w-xs flex-col border-l border-[var(--border)] bg-[var(--bg)] p-6 shadow-xl md:hidden',
            )}
          >
            <div className="flex items-center justify-between">
              <span className="font-mono text-sm uppercase tracking-wide text-[var(--muted)]">
                Menu
              </span>
              <button
                type="button"
                aria-label="Close menu"
                onClick={() => setOpen(false)}
                className="inline-flex h-9 w-9 items-center justify-center rounded-md text-[var(--fg)] hover:bg-[var(--surface)]"
              >
                <X className="h-5 w-5" />
              </button>
            </div>
            <nav className="mt-8 flex flex-col gap-1">
              {links.map((l) => (
                <a
                  key={l.href}
                  href={l.href}
                  onClick={() => setOpen(false)}
                  className="rounded-md px-3 py-2 text-base text-[var(--fg)] hover:bg-[var(--surface)]"
                >
                  {l.label}
                </a>
              ))}
            </nav>
          </div>
        </>
      )}
    </>
  );
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
npm test -- MobileDrawer
```

3 passed.

- [ ] **Step 5: Wire into Header**

Modify `portfolio/src/design-system/layout/Header.tsx`:
- Import `MobileDrawer`.
- Hide the existing inline `<nav>` link list on mobile (`hidden md:flex`).
- Render `<MobileDrawer links={...} />` next to the ThemeToggle.

The link list to pass: `[{href:'#projects',label:'Projects'},{href:'#blog',label:'Blog'},{href:'#oss',label:'OSS'},{href:'#about',label:'About'},{href:'#contact',label:'Contact'}]`.

- [ ] **Step 6: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

49 unit tests (46 + 3).

- [ ] **Step 7: Commit and push**

```bash
git add -A
git commit -m "feat(layout): replace inline mobile nav with accessible drawer dialog"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 9: Hero background richness (grid + glows + stars)

**Files:**
- Create: `portfolio/src/design-system/visuals/HeroBackground.tsx`
- Modify: `portfolio/src/app/globals.css` (add @keyframes for star twinkle)
- Modify: `portfolio/src/sections/Hero.tsx` (mount HeroBackground)

- [ ] **Step 1: Add twinkle keyframes**

Append to `portfolio/src/app/globals.css`:

```css
/* Hero star twinkle */
@keyframes twinkle {
  0%, 100% { opacity: 0.2; transform: scale(0.8); }
  50% { opacity: 0.9; transform: scale(1.1); }
}
@media (prefers-reduced-motion: reduce) {
  .hero-star { animation: none !important; }
}
```

- [ ] **Step 2: Implement HeroBackground**

Create `portfolio/src/design-system/visuals/HeroBackground.tsx`:

```typescript
const STAR_POSITIONS = [
  { top: '15%', left: '20%', delay: '0s', size: '2px' },
  { top: '30%', left: '70%', delay: '0.6s', size: '1px' },
  { top: '50%', left: '10%', delay: '1.2s', size: '2px' },
  { top: '65%', left: '85%', delay: '0.3s', size: '1px' },
  { top: '20%', left: '90%', delay: '1.5s', size: '2px' },
  { top: '80%', left: '30%', delay: '0.9s', size: '1px' },
  { top: '40%', left: '50%', delay: '0.4s', size: '1px' },
  { top: '70%', left: '60%', delay: '1.8s', size: '2px' },
];

export function HeroBackground() {
  return (
    <div aria-hidden="true" className="pointer-events-none absolute inset-0 -z-10 overflow-hidden">
      {/* Dotted grid */}
      <div
        className="absolute inset-0 opacity-[0.06]"
        style={{
          backgroundImage:
            'radial-gradient(circle, var(--color-brand-500) 1px, transparent 1px)',
          backgroundSize: '24px 24px',
          maskImage: 'radial-gradient(ellipse at center, black 30%, transparent 75%)',
          WebkitMaskImage: 'radial-gradient(ellipse at center, black 30%, transparent 75%)',
        }}
      />
      {/* Top-left brand glow */}
      <div
        className="absolute -left-32 -top-32 h-96 w-96 rounded-full opacity-30 blur-3xl"
        style={{ background: 'radial-gradient(circle, var(--color-brand-500), transparent 70%)' }}
      />
      {/* Bottom-right warm glow */}
      <div
        className="absolute -bottom-32 -right-32 h-96 w-96 rounded-full opacity-20 blur-3xl"
        style={{ background: 'radial-gradient(circle, var(--color-brand-300), transparent 70%)' }}
      />
      {/* Twinkling stars */}
      {STAR_POSITIONS.map((s, i) => (
        <span
          key={i}
          className="hero-star absolute rounded-full bg-white/70"
          style={{
            top: s.top,
            left: s.left,
            width: s.size,
            height: s.size,
            animation: `twinkle 3s ease-in-out ${s.delay} infinite`,
          }}
        />
      ))}
    </div>
  );
}
```

- [ ] **Step 3: Mount in Hero**

Modify `portfolio/src/sections/Hero.tsx`:
- Import `HeroBackground`.
- Change the `<section>` to `relative`: add `className="relative py-20 sm:py-32"`.
- Render `<HeroBackground />` as the first child of the `<section>`.

- [ ] **Step 4: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 5: Commit and push**

```bash
git add -A
git commit -m "feat(hero): add ambient background — dotted grid, dual glows, twinkling stars"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 10: Hero name line-by-line reveal

**Files:**
- Modify: `portfolio/src/sections/Hero.tsx`

- [ ] **Step 1: Convert Hero to use Framer Motion line reveals**

Replace the Hero `<h1>` block with a stagger animation. Read the current Hero, then:

1. Mark the file `'use client'` at the top (currently a server component).
2. Import `motion` from `framer-motion` and `useReducedMotion` from `@/design-system/motion/reducedMotion`.
3. Wrap each top-level line of the `<h1>` in a `motion.span` with `display: block`. The first line is "Hi, I'm Maruthan", second line is the Typewriter span.
4. Apply this stagger config:

```tsx
const containerVariants = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.12, delayChildren: 0.1 } },
};
const lineVariants = {
  hidden: { opacity: 0, y: 24, clipPath: 'inset(100% 0 0 0)' },
  visible: {
    opacity: 1,
    y: 0,
    clipPath: 'inset(0 0 0 0)',
    transition: { duration: 0.6, ease: [0.4, 0, 0.2, 1] },
  },
};
```

5. Use them as: `<motion.h1 ... initial="hidden" animate="visible" variants={containerVariants}>` then `<motion.span variants={lineVariants} className="block">...</motion.span>` per line.
6. If `useReducedMotion()` returns true, skip the variants and render the h1 plain.

- [ ] **Step 2: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 3: Commit and push**

```bash
git add -A
git commit -m "feat(hero): line-by-line reveal animation on hero heading"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 11: Tech marquee

**Files:**
- Create: `portfolio/src/design-system/visuals/TechMarquee.tsx`
- Modify: `portfolio/src/app/globals.css` (add @keyframes for marquee)
- Modify: `portfolio/src/sections/TechStack.tsx` (prepend marquee)

- [ ] **Step 1: Add marquee keyframes**

Append to `portfolio/src/app/globals.css`:

```css
/* Tech marquee — auto-scrolling row */
@keyframes marquee {
  from { transform: translateX(0); }
  to { transform: translateX(-50%); }
}
.marquee-track {
  animation: marquee 40s linear infinite;
  will-change: transform;
}
@media (prefers-reduced-motion: reduce) {
  .marquee-track { animation: none; }
}
```

- [ ] **Step 2: Implement TechMarquee**

Create `portfolio/src/design-system/visuals/TechMarquee.tsx`:

```typescript
import { techStack } from '@/content/techStack';

export function TechMarquee() {
  // Repeat the list twice so the loop is seamless
  const items = [...techStack, ...techStack];
  return (
    <div
      aria-hidden="true"
      className="relative -mx-4 overflow-hidden border-y border-[var(--border)] bg-[var(--surface)] py-3"
      style={{
        maskImage: 'linear-gradient(to right, transparent, black 10%, black 90%, transparent)',
        WebkitMaskImage:
          'linear-gradient(to right, transparent, black 10%, black 90%, transparent)',
      }}
    >
      <div className="marquee-track flex w-max items-center gap-8 whitespace-nowrap text-sm">
        {items.map((t, i) => (
          <span key={i} className="flex items-center gap-8 font-mono text-[var(--muted)]">
            <span>{t.name}</span>
            <span className="text-[var(--color-brand-500)]/40">/</span>
          </span>
        ))}
      </div>
    </div>
  );
}
```

- [ ] **Step 3: Mount above TechStack**

Modify `portfolio/src/sections/TechStack.tsx`:
- Import `TechMarquee`.
- Render `<TechMarquee />` as the first child returned from the section, before the `<Section ... />` block. Or wrap both in a fragment and place marquee before the heading.

- [ ] **Step 4: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 5: Commit and push**

```bash
git add -A
git commit -m "feat(tech): add auto-scrolling tech marquee above TechStack section"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 12: Section reveal animations

**Files:**
- Modify: `portfolio/src/design-system/motion/variants.ts`
- Modify: `portfolio/src/design-system/components/Section.tsx`

- [ ] **Step 1: Extend variants**

Modify `portfolio/src/design-system/motion/variants.ts` — append:

```typescript
export const sectionReveal: Variants = {
  hidden: { opacity: 0, y: 32 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.6, ease: [0.4, 0, 0.2, 1] },
  },
};
```

- [ ] **Step 2: Convert Section to use motion**

Replace `portfolio/src/design-system/components/Section.tsx` so the wrapper animates on scroll into view. The component becomes a client component.

```typescript
'use client';

import { forwardRef, type HTMLAttributes } from 'react';
import { motion } from 'framer-motion';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';
import { sectionReveal } from '@/design-system/motion/variants';
import { cn } from '@/design-system/utils/cn';

export interface SectionProps extends HTMLAttributes<HTMLElement> {
  eyebrow?: string;
  title?: string;
  description?: string;
}

export const Section = forwardRef<HTMLElement, SectionProps>(
  ({ eyebrow, title, description, className, children, ...props }, ref) => {
    const reduced = useReducedMotion();
    const variants = reduced ? undefined : sectionReveal;

    return (
      <motion.section
        ref={ref}
        className={cn('scroll-mt-20 py-20', className)}
        initial={reduced ? false : 'hidden'}
        whileInView={reduced ? undefined : 'visible'}
        viewport={{ once: true, amount: 0.2 }}
        variants={variants}
        {...(props as Record<string, unknown>)}
      >
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
      </motion.section>
    );
  },
);
Section.displayName = 'Section';
```

NOTE: The `{...(props as Record<string, unknown>)}` cast is necessary because Framer's `MotionProps` and React's `HTMLAttributes` overlap on event handlers. It's an acceptable cast for this wrapper.

- [ ] **Step 3: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

If any test breaks because Section is now `'use client'`, check that no test directly mounts Section in an SSR-only context. If a test fails, surface it in the report rather than skip.

- [ ] **Step 4: Commit and push**

```bash
git add -A
git commit -m "feat(motion): scroll-reveal animation on every Section"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 13: Magnetic hover hook + apply to project cards

**Files:**
- Create: `portfolio/src/design-system/motion/useMagneticHover.ts`
- Create: `portfolio/tests/useMagneticHover.test.ts`
- Modify: `portfolio/src/design-system/components/ProjectCard.tsx`

- [ ] **Step 1: Write failing test**

Create `portfolio/tests/useMagneticHover.test.ts`:

```typescript
import { describe, it, expect } from 'vitest';
import { renderHook } from '@testing-library/react';
import { useMagneticHover } from '@/design-system/motion/useMagneticHover';

describe('useMagneticHover', () => {
  it('returns a ref object', () => {
    const { result } = renderHook(() => useMagneticHover());
    expect(result.current).toHaveProperty('current');
  });

  it('does not throw when reduced motion is preferred', () => {
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
    const { result } = renderHook(() => useMagneticHover());
    expect(result.current).toHaveProperty('current');
  });
});
```

- [ ] **Step 2: Run — expect FAIL**

```bash
npm test -- useMagneticHover
```

- [ ] **Step 3: Implement**

Create `portfolio/src/design-system/motion/useMagneticHover.ts`:

```typescript
'use client';

import { useEffect, useRef } from 'react';
import { useReducedMotion } from './reducedMotion';

export function useMagneticHover<T extends HTMLElement = HTMLDivElement>(strength = 0.25) {
  const ref = useRef<T>(null);
  const reduced = useReducedMotion();

  useEffect(() => {
    if (reduced) return;
    if (typeof window === 'undefined') return;
    if (window.matchMedia('(hover: none)').matches) return;
    const node = ref.current;
    if (!node) return;

    const onMove = (e: MouseEvent) => {
      const rect = node.getBoundingClientRect();
      const cx = rect.left + rect.width / 2;
      const cy = rect.top + rect.height / 2;
      const dx = (e.clientX - cx) * strength;
      const dy = (e.clientY - cy) * strength;
      node.style.transform = `translate3d(${dx}px, ${dy}px, 0)`;
    };
    const onLeave = () => {
      node.style.transform = '';
    };

    node.addEventListener('mousemove', onMove);
    node.addEventListener('mouseleave', onLeave);
    return () => {
      node.removeEventListener('mousemove', onMove);
      node.removeEventListener('mouseleave', onLeave);
      node.style.transform = '';
    };
  }, [reduced, strength]);

  return ref;
}
```

- [ ] **Step 4: Run — expect PASS**

```bash
npm test -- useMagneticHover
```

2 passed.

- [ ] **Step 5: Apply to ProjectCard**

Modify `portfolio/src/design-system/components/ProjectCard.tsx`:
- Add `'use client';` at the top.
- Import `useMagneticHover`.
- Call `const ref = useMagneticHover<HTMLDivElement>(0.08);` (low strength — subtle pull).
- Pass `ref` to the outer `<Card ref={ref} ...>`. If `Card` doesn't already accept ref forwarding, this will error — Card uses `forwardRef` already (Plan 1), so it should work.
- Add CSS transition on transform for smooth return: append `transition-transform duration-300 ease-out` to the className.

- [ ] **Step 6: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

51 unit tests (49 + 2 new).

- [ ] **Step 7: Commit and push**

```bash
git add -A
git commit -m "feat(motion): magnetic hover effect on ProjectCard"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 14: Custom cursor

**Files:**
- Create: `portfolio/src/design-system/components/CustomCursor.tsx`
- Modify: `portfolio/src/app/globals.css` (cursor base + interactive selectors)
- Modify: `portfolio/src/app/layout.tsx` (mount CustomCursor)

- [ ] **Step 1: Add cursor base CSS**

Append to `portfolio/src/app/globals.css`:

```css
/* Custom cursor — only on hover-capable, non-reduced-motion devices */
@media (hover: hover) and (prefers-reduced-motion: no-preference) {
  body.has-custom-cursor { cursor: none; }
  body.has-custom-cursor a,
  body.has-custom-cursor button,
  body.has-custom-cursor [data-cursor-hover],
  body.has-custom-cursor input,
  body.has-custom-cursor textarea,
  body.has-custom-cursor label { cursor: none; }
}
.cc-ring {
  position: fixed;
  top: 0;
  left: 0;
  width: 28px;
  height: 28px;
  margin-left: -14px;
  margin-top: -14px;
  border: 1.5px solid var(--color-brand-500);
  border-radius: 9999px;
  pointer-events: none;
  z-index: 9999;
  mix-blend-mode: difference;
  transition: width 200ms ease-out, height 200ms ease-out, margin 200ms ease-out, opacity 150ms ease-out;
  opacity: 0;
}
.cc-ring.cc-ring--visible { opacity: 1; }
.cc-ring.cc-ring--hover { width: 64px; height: 64px; margin-left: -32px; margin-top: -32px; }
.cc-ring.cc-ring--click { width: 18px; height: 18px; margin-left: -9px; margin-top: -9px; }
.cc-dot {
  position: fixed;
  top: 0;
  left: 0;
  width: 5px;
  height: 5px;
  margin-left: -2.5px;
  margin-top: -2.5px;
  background: var(--color-brand-500);
  border-radius: 9999px;
  pointer-events: none;
  z-index: 10000;
  opacity: 0;
}
.cc-dot.cc-dot--visible { opacity: 1; }
```

- [ ] **Step 2: Implement CustomCursor**

Create `portfolio/src/design-system/components/CustomCursor.tsx`:

```typescript
'use client';

import { useEffect, useRef } from 'react';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';

export function CustomCursor() {
  const ringRef = useRef<HTMLDivElement>(null);
  const dotRef = useRef<HTMLDivElement>(null);
  const reduced = useReducedMotion();

  useEffect(() => {
    if (reduced) return;
    if (typeof window === 'undefined') return;
    const isTouch = window.matchMedia('(hover: none)').matches;
    if (isTouch) return;

    document.body.classList.add('has-custom-cursor');

    const ring = ringRef.current;
    const dot = dotRef.current;
    if (!ring || !dot) return;

    let raf = 0;
    let mx = -100;
    let my = -100;
    let dx = -100;
    let dy = -100;

    const tick = () => {
      // Smooth follow for the ring (lerp), dot is direct
      dx += (mx - dx) * 0.18;
      dy += (my - dy) * 0.18;
      ring.style.transform = `translate(${dx}px, ${dy}px)`;
      dot.style.transform = `translate(${mx}px, ${my}px)`;
      raf = requestAnimationFrame(tick);
    };

    const onMove = (e: MouseEvent) => {
      mx = e.clientX;
      my = e.clientY;
      ring.classList.add('cc-ring--visible');
      dot.classList.add('cc-dot--visible');
    };

    const isHoverTarget = (el: Element | null): boolean => {
      while (el) {
        const tag = el.tagName;
        if (tag === 'A' || tag === 'BUTTON' || tag === 'INPUT' || tag === 'TEXTAREA') return true;
        if (el instanceof HTMLElement && el.dataset.cursorHover) return true;
        el = el.parentElement;
      }
      return false;
    };

    const onOver = (e: MouseEvent) => {
      if (isHoverTarget(e.target as Element)) ring.classList.add('cc-ring--hover');
      else ring.classList.remove('cc-ring--hover');
    };
    const onDown = () => ring.classList.add('cc-ring--click');
    const onUp = () => ring.classList.remove('cc-ring--click');
    const onLeave = () => {
      ring.classList.remove('cc-ring--visible');
      dot.classList.remove('cc-dot--visible');
    };

    window.addEventListener('mousemove', onMove);
    window.addEventListener('mouseover', onOver);
    window.addEventListener('mousedown', onDown);
    window.addEventListener('mouseup', onUp);
    document.addEventListener('mouseleave', onLeave);
    raf = requestAnimationFrame(tick);

    return () => {
      cancelAnimationFrame(raf);
      window.removeEventListener('mousemove', onMove);
      window.removeEventListener('mouseover', onOver);
      window.removeEventListener('mousedown', onDown);
      window.removeEventListener('mouseup', onUp);
      document.removeEventListener('mouseleave', onLeave);
      document.body.classList.remove('has-custom-cursor');
    };
  }, [reduced]);

  if (reduced) return null;

  return (
    <>
      <div ref={ringRef} className="cc-ring" aria-hidden="true" />
      <div ref={dotRef} className="cc-dot" aria-hidden="true" />
    </>
  );
}
```

- [ ] **Step 3: Mount in root layout**

Modify `portfolio/src/app/layout.tsx`:
- Import `CustomCursor`.
- Inside `<Providers>`, before `<Header />`, render `<CustomCursor />`.

- [ ] **Step 4: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 5: Commit and push**

```bash
git add -A
git commit -m "feat(motion): add custom cursor with ring + dot, reduced-motion + touch fallbacks"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 15: Boot loader

**Files:**
- Create: `portfolio/src/design-system/components/BootLoader.tsx`
- Modify: `portfolio/src/app/globals.css`
- Modify: `portfolio/src/app/layout.tsx`

- [ ] **Step 1: Add loader CSS**

Append to `portfolio/src/app/globals.css`:

```css
/* Boot loader */
.boot-loader {
  position: fixed;
  inset: 0;
  z-index: 100000;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;
  background: var(--bg);
  transition: opacity 350ms ease-out, visibility 0s linear 350ms;
}
.boot-loader.boot-loader--gone {
  opacity: 0;
  visibility: hidden;
}
.boot-counter {
  font-family: var(--font-geist-mono);
  font-size: clamp(3rem, 8vw, 6rem);
  font-weight: 800;
  letter-spacing: -0.04em;
  color: var(--color-brand-500);
  line-height: 1;
}
.boot-bar {
  width: 200px;
  height: 2px;
  background: var(--border);
  border-radius: 2px;
  overflow: hidden;
}
.boot-fill {
  width: 0%;
  height: 100%;
  background: linear-gradient(90deg, var(--color-brand-500), var(--color-brand-300));
  border-radius: 2px;
  transition: width 80ms linear;
}
.boot-text {
  font-family: var(--font-geist-mono);
  font-size: 0.7rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  color: var(--muted);
}
@media (prefers-reduced-motion: reduce) {
  .boot-loader { display: none !important; }
}
```

- [ ] **Step 2: Implement BootLoader**

Create `portfolio/src/design-system/components/BootLoader.tsx`:

```typescript
'use client';

import { useEffect, useState } from 'react';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';

export function BootLoader() {
  const reduced = useReducedMotion();
  const [count, setCount] = useState(0);
  const [gone, setGone] = useState(false);

  useEffect(() => {
    if (reduced) {
      setGone(true);
      return;
    }
    let active = true;
    const start = performance.now();
    const duration = 800; // ms total

    const tick = () => {
      if (!active) return;
      const elapsed = performance.now() - start;
      const pct = Math.min(100, Math.round((elapsed / duration) * 100));
      setCount(pct);
      if (pct < 100) {
        requestAnimationFrame(tick);
      } else {
        setTimeout(() => setGone(true), 200);
      }
    };
    requestAnimationFrame(tick);

    return () => {
      active = false;
    };
  }, [reduced]);

  if (reduced || gone) return null;

  return (
    <div className="boot-loader" role="status" aria-live="polite" aria-label="Loading portfolio">
      <div className="boot-counter">{count}</div>
      <div className="boot-bar" role="progressbar" aria-valuenow={count} aria-valuemin={0} aria-valuemax={100}>
        <div className="boot-fill" style={{ width: `${count}%` }} />
      </div>
      <div className="boot-text">Maruthan G</div>
    </div>
  );
}
```

- [ ] **Step 3: Mount in root layout**

Modify `portfolio/src/app/layout.tsx`:
- Import `BootLoader`.
- Inside `<body>`, as the FIRST child (before skip link), render `<BootLoader />`.

This ensures the loader covers the whole page until React mounts and the loader removes itself.

- [ ] **Step 4: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

If `npm test` fails because the loader briefly mounts in tests rendering the layout, that's expected — the layout itself isn't tested directly, only sections. If a section test breaks, surface it.

If `npm run test:e2e` fails because Playwright now sees the boot loader, update the e2e test to `page.waitForSelector('.boot-loader--gone, [aria-label="Loading portfolio"]:not(.boot-loader)', { state: 'attached' })` OR `await page.waitForLoadState('networkidle')` before assertions. If you need to update e2e, do it in this same task.

- [ ] **Step 5: Commit and push**

```bash
git add -A
git commit -m "feat(motion): add boot loader with 0-100 counter and progress bar"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 16: Theme palette picker

**Files:**
- Create: `portfolio/src/design-system/components/ThemePalette.tsx`
- Create: `portfolio/tests/ThemePalette.test.tsx`
- Modify: `portfolio/src/app/globals.css` (accent variant `data-accent` selectors)
- Modify: `portfolio/src/app/providers.tsx` (read accent from cookie/localStorage)
- Modify: `portfolio/src/design-system/layout/Header.tsx` (mount ThemePalette next to ThemeToggle)

- [ ] **Step 1: Define accent variants in CSS**

Append to `portfolio/src/app/globals.css`:

```css
/* Accent palette variants — switch by setting data-accent on <html> */
[data-accent='blue'] {
  --color-brand-500: #58a6ff;
  --color-brand-300: #66abff;
  --color-brand-600: #1f7ae0;
}
[data-accent='cyan'] {
  --color-brand-500: #00e5c8;
  --color-brand-300: #5ef2dd;
  --color-brand-600: #00b39d;
}
[data-accent='orange'] {
  --color-brand-500: #ff7a45;
  --color-brand-300: #ffa07a;
  --color-brand-600: #e0561c;
}
[data-accent='magenta'] {
  --color-brand-500: #ff3da6;
  --color-brand-300: #ff7ec0;
  --color-brand-600: #d01885;
}
```

- [ ] **Step 2: Write failing test**

Create `portfolio/tests/ThemePalette.test.tsx`:

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ThemePalette } from '@/design-system/components/ThemePalette';

describe('ThemePalette', () => {
  it('renders a trigger button', () => {
    render(<ThemePalette />);
    expect(screen.getByRole('button', { name: /change accent color/i })).toBeInTheDocument();
  });

  it('opens a popover with 4 swatches when clicked', async () => {
    const user = userEvent.setup();
    render(<ThemePalette />);
    await user.click(screen.getByRole('button', { name: /change accent color/i }));
    expect(screen.getByRole('button', { name: /accent: blue/i })).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /accent: cyan/i })).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /accent: orange/i })).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /accent: magenta/i })).toBeInTheDocument();
  });

  it('sets data-accent on html when a swatch is selected', async () => {
    const user = userEvent.setup();
    render(<ThemePalette />);
    await user.click(screen.getByRole('button', { name: /change accent color/i }));
    await user.click(screen.getByRole('button', { name: /accent: cyan/i }));
    expect(document.documentElement.getAttribute('data-accent')).toBe('cyan');
  });
});
```

- [ ] **Step 3: Run — expect FAIL**

```bash
npm test -- ThemePalette
```

- [ ] **Step 4: Implement**

Create `portfolio/src/design-system/components/ThemePalette.tsx`:

```typescript
'use client';

import { Palette } from 'lucide-react';
import { useEffect, useState } from 'react';
import { cn } from '@/design-system/utils/cn';

const ACCENTS = [
  { id: 'blue', label: 'Blue', hex: '#58a6ff' },
  { id: 'cyan', label: 'Cyan', hex: '#00e5c8' },
  { id: 'orange', label: 'Orange', hex: '#ff7a45' },
  { id: 'magenta', label: 'Magenta', hex: '#ff3da6' },
] as const;

type AccentId = (typeof ACCENTS)[number]['id'];
const STORAGE_KEY = 'portfolio-accent';
const DEFAULT_ACCENT: AccentId = 'blue';

export function ThemePalette({ className }: { className?: string }) {
  const [open, setOpen] = useState(false);
  const [current, setCurrent] = useState<AccentId>(DEFAULT_ACCENT);

  useEffect(() => {
    const saved =
      (typeof localStorage !== 'undefined'
        ? (localStorage.getItem(STORAGE_KEY) as AccentId | null)
        : null) ?? DEFAULT_ACCENT;
    setCurrent(saved);
    document.documentElement.setAttribute('data-accent', saved);
  }, []);

  const choose = (id: AccentId) => {
    setCurrent(id);
    document.documentElement.setAttribute('data-accent', id);
    if (typeof localStorage !== 'undefined') localStorage.setItem(STORAGE_KEY, id);
    setOpen(false);
  };

  const currentHex = ACCENTS.find((a) => a.id === current)?.hex ?? ACCENTS[0]!.hex;

  return (
    <div className={cn('relative', className)}>
      <button
        type="button"
        aria-label="Change accent color"
        aria-haspopup="dialog"
        aria-expanded={open}
        onClick={() => setOpen((s) => !s)}
        className="relative inline-flex h-9 w-9 items-center justify-center rounded-md border border-[var(--border)] text-[var(--fg)] hover:bg-[var(--surface)]"
      >
        <Palette className="h-4 w-4" />
        <span
          aria-hidden="true"
          className="absolute bottom-1 right-1 h-2 w-2 rounded-full border border-[var(--bg)]"
          style={{ background: currentHex }}
        />
      </button>
      {open && (
        <div
          role="dialog"
          aria-label="Accent color"
          className="absolute right-0 top-full z-30 mt-2 flex gap-2 rounded-md border border-[var(--border)] bg-[var(--surface)] p-2 shadow-lg"
        >
          {ACCENTS.map((a) => (
            <button
              key={a.id}
              type="button"
              aria-label={`Accent: ${a.label}`}
              onClick={() => choose(a.id)}
              className={cn(
                'h-7 w-7 rounded-full border transition-transform hover:scale-110',
                current === a.id
                  ? 'border-[var(--fg)] ring-2 ring-[var(--fg)] ring-offset-2 ring-offset-[var(--surface)]'
                  : 'border-[var(--border)]',
              )}
              style={{ background: a.hex }}
            />
          ))}
        </div>
      )}
    </div>
  );
}
```

- [ ] **Step 5: Run — expect PASS**

```bash
npm test -- ThemePalette
```

3 passed.

- [ ] **Step 6: Mount in Header**

Modify `portfolio/src/design-system/layout/Header.tsx`:
- Import `ThemePalette`.
- Render it next to `ThemeToggle` in the right cluster.

- [ ] **Step 7: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

54 unit tests (51 + 3 new).

- [ ] **Step 8: Commit and push**

```bash
git add -A
git commit -m "feat(theme): add accent color palette picker (blue/cyan/orange/magenta)"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 17: Horizontal-scroll Featured Projects (GSAP pin)

**Files:**
- Create: `portfolio/src/design-system/visuals/HorizontalScrollProjects.tsx`
- Modify: `portfolio/src/sections/FeaturedProjects.tsx`

- [ ] **Step 1: Implement HorizontalScrollProjects**

Create `portfolio/src/design-system/visuals/HorizontalScrollProjects.tsx`:

```typescript
'use client';

import { useEffect, useRef } from 'react';
import { useReducedMotion } from '@/design-system/motion/reducedMotion';
import { ProjectCard } from '@/design-system/components/ProjectCard';
import type { Project } from '@/content/projects';

export function HorizontalScrollProjects({ projects }: { projects: Project[] }) {
  const sectionRef = useRef<HTMLDivElement>(null);
  const trackRef = useRef<HTMLDivElement>(null);
  const reduced = useReducedMotion();

  useEffect(() => {
    if (reduced) return;
    if (typeof window === 'undefined') return;
    if (window.matchMedia('(max-width: 1023px)').matches) return; // mobile/tablet: skip pinning

    let cleanup: (() => void) | undefined;

    (async () => {
      const { gsap } = await import('gsap');
      const { ScrollTrigger } = await import('gsap/ScrollTrigger');
      gsap.registerPlugin(ScrollTrigger);

      const section = sectionRef.current;
      const track = trackRef.current;
      if (!section || !track) return;

      const distance = track.scrollWidth - section.clientWidth;
      const tween = gsap.to(track, {
        x: -distance,
        ease: 'none',
        scrollTrigger: {
          trigger: section,
          start: 'top top',
          end: () => `+=${distance}`,
          pin: true,
          scrub: 1,
          invalidateOnRefresh: true,
        },
      });

      cleanup = () => {
        tween.kill();
        ScrollTrigger.getAll().forEach((t) => t.kill());
      };
    })();

    return () => cleanup?.();
  }, [reduced]);

  // Mobile/tablet/reduced-motion fallback: vertical grid
  return (
    <>
      {/* Desktop horizontal scroll */}
      <div ref={sectionRef} className="relative hidden lg:block">
        <div className="overflow-hidden">
          <div
            ref={trackRef}
            className="flex gap-6 py-8"
            style={{ width: 'max-content' }}
          >
            {projects.map((p) => (
              <div key={p.slug} className="w-[420px] shrink-0">
                <ProjectCard project={p} />
              </div>
            ))}
          </div>
        </div>
      </div>
      {/* Mobile/tablet grid */}
      <div className="grid gap-6 md:grid-cols-2 lg:hidden">
        {projects.map((p) => (
          <ProjectCard key={p.slug} project={p} />
        ))}
      </div>
    </>
  );
}
```

- [ ] **Step 2: Use in FeaturedProjects**

Modify `portfolio/src/sections/FeaturedProjects.tsx`:
- Import `HorizontalScrollProjects` instead of `ProjectCard`.
- Replace the existing `<div className="grid gap-6 md:grid-cols-2">{...map(ProjectCard)...}</div>` with `<HorizontalScrollProjects projects={featuredProjects} />`.

- [ ] **Step 3: Verify**

```bash
npm run typecheck && npm run format:check && npm test && npm run build
```

- [ ] **Step 4: Verify e2e still passes**

```bash
npm run test:e2e
```

The "Selected projects" heading must still be findable. Project cards still need to render at viewport widths e2e uses (default Playwright is 1280×720 — that's `lg`, so horizontal scroll branch will mount). The cards should still be visible because GSAP will start with the track at x=0 — the FIRST card is in view at scroll 0.

If the e2e test fails because the second card is no longer visible without scroll, the fix is either:
1. Scroll the page in the test before asserting on later cards.
2. Loosen the assertion to just check that "Selected projects" heading exists (which is what current home.spec.ts does).

The current home.spec.ts only checks the section heading exists, so this should pass. Verify and report.

- [ ] **Step 5: Commit and push**

```bash
git add -A
git commit -m "feat(projects): pin section + horizontal scroll on desktop, grid on mobile"
git push origin main
gh run watch --repo maruthang/portfolio --exit-status
```

---

### Task 18: Final regression sweep + production verify

**Files:**
- (verification only — no code changes expected)

- [ ] **Step 1: Run full test suite locally**

```bash
cd C:/Users/Maruthan/Documents/github/portfolio
npm run typecheck && npm run format:check && npm test && npm run build && npm run test:e2e
```

All exit 0. Report counts.

- [ ] **Step 2: Verify CI green**

```bash
gh run list --repo maruthang/portfolio --limit 1
```

Latest run status: success.

- [ ] **Step 3: Verify production**

After Vercel auto-deploys (~60 sec after CI green), probe:

```bash
curl -s https://portfolio-tawny-two-72.vercel.app/ | grep -E "Hi, I|few things|build with|Selected projects|Open source|Let.{0,3}s talk|skip-link|cc-ring|boot-counter|marquee-track" | head -20
```

Expected hits include the existing 6 section markers PLUS several new ones from this plan: `skip-link`, `cc-ring` (custom cursor), `boot-counter` (loader), `marquee-track` (tech marquee). Report what's found.

- [ ] **Step 4: Smoke check the new behaviors visually**

Open the production URL in a browser. Confirm visually (you can't be a subagent for this — it's the controller's responsibility):
- Boot loader flashes briefly on first load
- Custom cursor follows mouse, expands on hover over links/buttons
- Hero has dotted-grid + glows + twinkling stars
- Hero name reveals line by line on first paint
- Availability badge has green pulsing halo
- Tech marquee scrolls horizontally above the stack section
- Sections fade up + slide in as scrolled into view
- Project cards subtly pull toward cursor on hover
- Featured Projects section pins and scrolls horizontally on desktop
- Theme palette picker shows 4 swatches; each swatch flips the accent everywhere
- Mobile drawer opens/closes (test at <768px)
- Back-to-top button works

If anything's broken in production but worked locally, debug via Vercel logs.

- [ ] **Step 5: Update memory**

Append a brief Plan 2.5 completion note in the controller-side memory at `C:/Users/Maruthan/.claude/projects/C--Users-Maruthan-Documents-github-MaruthanG/memory/project_portfolio_site.md`. (The controller does this; subagent is not responsible.)

---

## Self-Review

**Spec coverage** — checked against `docs/superpowers/specs/2026-04-25-portfolio-design.md`:

| Spec section | Covered by |
|---|---|
| §6 Animation strategy: GSAP for scroll choreography | Task 1, 17 |
| §6 Reduced-motion respected globally | Tasks 2, 7, 9, 10, 11, 12, 13, 14, 15, 17 (every animation) |
| §6 Component preview (Ladle) | Not affected — existing stories still work |
| §5.5 Theme toggle | Already in Plan 1; Plan 2.5 adds accent palette on top |
| §5.5 Loading states (shimmer skeletons) | Boot loader replaces "shimmer" with counter (better) |
| §5.5 Easter egg: terminal boot sequence | Boot loader fulfills this |
| §11 Lighthouse ≥95 | Task 18 verifies |

**Placeholder scan:** None. Every task contains runnable code or commands.

**Type consistency:** Cross-task type names (`AccentId`, `Project`, `DrawerLink`) defined once in their owning files; consumers import.

**Open clarifications:**
- The hero name reveal animation (Task 10) requires the Hero component to become a client component. Currently it's a server component. The change is benign but worth noting because Tasks 9, 10, 11 all touch Hero.tsx; later tasks should read the file fresh before editing.
- Task 17's GSAP horizontal scroll defers loading GSAP via dynamic `import()`. This keeps GSAP out of the initial JS bundle for users who never reach the projects section on desktop.

---

## Definition of Done for Plan 2.5

- ✅ All 18 tasks committed and pushed; CI green on each.
- ✅ Live production site at `https://portfolio-tawny-two-72.vercel.app/` shows: boot loader, custom cursor, animated hero background + name reveal, availability ping, tech marquee, scroll-revealed sections, magnetic project cards, horizontal-scroll featured projects (desktop), theme palette picker, mobile drawer, back-to-top, skip link.
- ✅ Reduced-motion (System Settings → Reduce Motion) cleanly disables every animation; no broken layouts.
- ✅ Touch device (devtools mobile emulation) hides custom cursor, falls back to vertical grid for projects.
- ✅ Lighthouse Performance ≥95, Accessibility ≥95, Best Practices ≥95, SEO ≥95 (mobile + desktop).
- ✅ All unit tests pass (~60 expected: 41 from Plan 2 + ~19 new).
- ✅ Playwright e2e suite still 4 tests passing.
- ✅ TypeScript strict, Prettier clean, no Claude co-author trailers in commits.
- ✅ No regressions in Plan 2 functionality.
