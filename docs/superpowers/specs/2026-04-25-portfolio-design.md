# Portfolio Website Design Spec

**Author**: Maruthan G
**Date**: 2026-04-25
**Status**: Draft — pending user review
**Target launch**: v1 in ~3–4 weeks part-time

---

## 1. Overview

Build a personal portfolio at a custom domain (TBD — `maruthan.dev` recommended) that doubles as a resume, OSS contribution showcase, and writing platform. Audience is mixed: recruiters, peer developers, OSS maintainers, and future blog readers.

The portfolio must:
- Look and feel like it was built by someone who ships developer tooling (terminal/IDE-native aesthetic).
- Have a single-source-of-truth design system: change one token → propagates everywhere.
- Use animation deliberately (medium intensity) — earned, never decorative.
- Match or exceed the bar set by `yogeshwaran.com` (closest peer profile publicly).

The portfolio lives in a new repository (`maruthang/portfolio`), separate from the existing `MaruthanG/MaruthanG` profile-README repo.

---

## 2. Goals & Non-Goals

**Goals**
- Establish credibility for senior full-stack and OSS-aligned roles.
- Make the OSS contribution history (57 merged PRs, 103 open) immediately discoverable and verifiable.
- Provide a writing platform for OSS post-mortems, architecture notes, and tutorials.
- Enable client/recruiter contact via form, scheduled call, and direct channels.

**Non-Goals (v1)**
- E-commerce, payments, gated content.
- User accounts beyond optional GitHub OAuth (deferred to v2 guestbook).
- Multi-language i18n.
- Newsletter system (deferred to v2 — only if writing cadence justifies it).

---

## 3. Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 15 (App Router) + React 19 | User's day-job stack; SSR/ISR; OG image generation |
| Language | TypeScript (strict mode) | Type safety end-to-end |
| Styling | Tailwind CSS v4 (CSS-first config) | Token-driven utilities; CSS vars exposure |
| Component primitives | Radix UI via shadcn/ui base | Accessible primitives; copy-paste, no runtime dep |
| Animation | Framer Motion + GSAP ScrollTrigger + Lenis + View Transitions API | Component-level motion + scroll choreography + smooth scroll + native route transitions |
| Content | MDX via `next-mdx-remote` | File-based, git-tracked, no CMS overhead |
| Forms | React Hook Form + Zod + Resend | Validation + transactional email |
| Booking | Cal.com embed | Free; matches dev-portfolio convention |
| Analytics | Vercel Analytics + Plausible | Privacy-friendly; lightweight |
| Hosting | Vercel | Free tier covers all features; ISR; edge functions for OG |
| Domain | TBD (recommend `maruthan.dev`) | `.dev` matches identity |

---

## 4. Information Architecture

```
/                       Home: Hero, About, Stack, Featured Projects, OSS preview, Contact
/projects               All projects (filterable by tech)
/projects/[slug]        Per-project case study (problem → solution → architecture → outcomes)
/blog                   Blog index (categories, pagination)
/blog/[slug]            Post detail (TOC, reading time, syntax highlighting)
/oss                    Full OSS contributions table (sortable, links to PRs)
/uses                   Tools, editor, terminal, dotfiles, hardware
/now                    Currently working on / learning
/resume                 Web resume + PDF download
/404                    Terminal-themed easter egg page
```

All routes are statically rendered or ISR. No client-side-only routes.

---

## 5. Sections & Features

### 5.1 Home page — sections in order

1. **Hero**
   - Name, animated typewriter cycling: `Full Stack Developer` → `OSS Contributor` → `Bug Hunter` → `Tooling Builder`.
   - Blinking terminal cursor.
   - Tagline: one sentence summarizing identity.
   - CTAs: "View Work", "Resume", "Get in Touch".
   - Pills: location (Chennai), availability status (Open to opportunities / Heads down / Booked).
   - Subtle ambient background: dotted grid or noise texture.

2. **About**
   - Short narrative (3–4 paragraphs): journey, current focus, what drives the work.
   - Stats row with count-up animation on scroll-into-view: `57 PRs merged`, `10+ projects shipped`, `9+ major OSS projects contributed to`, `2+ years professional`.
   - Photo or stylized avatar.

3. **Tech Stack**
   - Categorized: Languages / Frontend / Backend / Databases / Cloud & DevOps / Data Engineering (Learning).
   - Filter chips that highlight matching items in other categories on hover.
   - Each item: icon + name + hover tooltip with short proficiency note.

4. **Featured Projects** (6–8 cards)
   - Card content: cover image, title, one-line summary, tech badges, status badge (Live / Archived / OSS), three CTAs: Live demo / GitHub / Case study.
   - Hover: cursor-following spotlight + subtle magnetic effect.
   - "View all projects →" link to `/projects`.

5. **Open Source preview**
   - Aggregate stats card: `57 merged · 103 open · 9 projects`.
   - Top 3 most-impactful PRs with project + title + merge date.
   - "View full contribution history →" link to `/oss`.

6. **Contact**
   - Form (name, email, message) with React Hook Form + Zod + Resend delivery.
   - Cal.com booking embed for scheduled calls.
   - Direct channels: LinkedIn, GitHub, Telegram, email.

7. **Footer**
   - Sitemap, "Built with [stack]", last-deployed timestamp (from Vercel build env), View source link, copyright.

### 5.2 Project detail pages (`/projects/[slug]`)

Each case study includes:
- Hero with project name, role, dates, tech badges, live/repo links.
- Problem statement.
- Solution overview.
- Architecture diagram (Mermaid or static SVG).
- Key technical decisions with reasoning.
- Screenshots / demo GIFs.
- Outcomes / metrics where available.
- Related projects.

Source: MDX file per project under `content/projects/`.

### 5.3 OSS page (`/oss`) — the differentiator

- Live PR counter (server-fetched daily via ISR or weekly cron-backed revalidation).
- Sortable, filterable table: project | PR # | title | status (merged/open) | merged-on date | link.
- Top contributions cards (manually curated highlights with longer descriptions).
- "Currently contributing to" badge row.
- Link to GitHub PR feed.

Data source: GitHub GraphQL API at build time (cached) + manual curation file for highlights.

### 5.4 Blog (`/blog` and `/blog/[slug]`)

- File-based MDX under `content/blog/`.
- Frontmatter: title, slug, date, category, tags, description, cover.
- Categories: OSS / Architecture / Tutorials / Notes.
- Index: card grid with cover, title, description, reading time, category pill.
- Detail: TOC sidebar (sticky), reading-time, syntax-highlighted code blocks (Shiki) with copy button, view counter (Vercel KV or Plausible-derived), prev/next post navigation.
- RSS feed at `/rss.xml`.

### 5.5 Power-user features

- **Command palette** (`⌘K` / `Ctrl+K`): kbar or cmdk library. Actions: navigate to any section/page, search blog posts, jump to projects, toggle theme, open social links, copy email.
- **Theme toggle**: light / dark / system. Persisted via cookie (no flash on hydration). Sun/moon morph animation.
- **Live GitHub stats widget** on home: contribution heatmap (current year), top languages.
- **Dynamic OG images**: per-project, per-blog-post via `next/og` edge runtime.
- **`/uses` page**: editor, terminal, fonts, dotfiles repo link, hardware.
- **`/now` page**: text + timestamped updates section.
- **Resume**: web view at `/resume` + downloadable PDF generated from same data source via `react-pdf` or static checked-in PDF.

### 5.6 SEO & Performance

- Sitemap auto-generated, submitted to Google Search Console.
- `robots.txt` allowing all.
- Structured data: JSON-LD `Person` on home, `BlogPosting` per post, `BreadcrumbList` per detail page.
- Per-page metadata: title, description, OG image, Twitter card.
- Lighthouse target: ≥95 on Performance, Accessibility, Best Practices, SEO.
- Core Web Vitals monitored via Vercel Speed Insights.

---

## 6. Design System

**Principle**: change one token → propagates to all components.

### Architecture

```
src/design-system/
  tokens/
    colors.ts          brand, neutral scale, accent, semantic (success/warn/error/info)
    typography.ts      font families, sizes, weights, line-heights, tracking
    spacing.ts         4px-base scale (xs=4, sm=8, md=12, lg=16, xl=24, 2xl=32, 3xl=48, 4xl=64)
    radii.ts           sm=4, md=8, lg=12, xl=16, full=9999
    shadows.ts         elevation scale (sm, md, lg, xl)
    motion.ts          durations (fast=150, base=250, slow=400), easings (standard, spring, decelerate)
    breakpoints.ts     sm=640, md=768, lg=1024, xl=1280, 2xl=1536
    index.ts           barrel export
  primitives/          unstyled wrappers around Radix UI
  components/          Button, Card, Badge, Input, Textarea, Dialog, Tooltip, CodeBlock, Tabs, Skeleton...
  patterns/            Hero, ProjectCard, TechStackChip, Timeline, BlogPostCard, StatCounter, SectionHeader...
```

### Token flow

1. Tokens defined as TypeScript constants in `tokens/`.
2. Tokens imported into `tailwind.config.ts` — generates utilities.
3. Tokens emitted as CSS custom properties via a generated `tokens.css` (built at compile time) — consumed by raw CSS and dynamic component props.
4. Components consume via Tailwind classes OR `var(--color-brand-500)` — never hardcoded.

### Typography

- **Display / headings**: Geist Sans (or Inter Display).
- **Body**: Geist Sans.
- **Mono / code**: JetBrains Mono or Geist Mono.
- Type scale: 12, 14, 16, 18, 20, 24, 30, 36, 48, 60, 72.

### Color

- **Dark mode primary** (matches developer aesthetic). Light mode is supported but secondary.
- Neutral scale: 50–950 (slate or zinc base).
- Accent: a single brand accent (proposed: VSCode-blue `#58a6ff` to mirror your README).
- Semantic: success (green), warning (amber), error (red), info (blue).

### Component preview

- Ladle for isolated component preview (lighter than Storybook, no plugin ecosystem needed). Runs on a separate dev port.

---

## 7. Animation Strategy

**Library mix**:
- **Framer Motion** — component-level animations, gestures, layout animations.
- **GSAP + ScrollTrigger** — complex scroll choreography (hero, section reveals).
- **Lenis** — smooth scroll baseline.
- **View Transitions API** — native cross-fade for route changes (with Framer Motion fallback).

**Where animations live**:

| Location | Effect | Library |
|---|---|---|
| Hero | Typewriter role-cycler, blinking cursor, staggered word reveal | Framer Motion |
| Section headers | Fade + 12px slide-up on scroll into view | Framer Motion `whileInView` |
| Project cards | Cursor-following spotlight, magnetic hover | Framer Motion + custom hook |
| Stat counters | Count-up on scroll-into-view | Framer Motion `useMotionValue` |
| Tech stack chips | Stagger-in on first reveal | Framer Motion `staggerChildren` |
| Theme toggle | Sun/moon morph | Framer Motion SVG path morph |
| Page transitions | Cross-fade via View Transitions API | Native + fallback |
| Command palette | Spring-in modal | Framer Motion |
| Loading states | Shimmer skeletons | Tailwind animation |
| 404 page | Terminal boot sequence | Framer Motion timed sequence |

**Reduced-motion**: All animations respect `prefers-reduced-motion`. Degrades to instant transitions and removes parallax. Implemented via a global `useReducedMotion` hook and CSS media query.

**Performance budget**: No animation may cause layout thrash. Use `transform` and `opacity` only. GSAP ScrollTrigger pinned sections must release on unmount.

---

## 8. Content Strategy

- **Projects**: 6–8 in featured set. Source from existing README (B2B Marketplace, Conversational Commerce Bot, Sales Analytics Platform, Fitness Ecosystem, Health & Wellness App, Data Warehouse). Each gets a case study MDX file.
- **OSS highlights**: 5–8 most-impactful merged PRs with longer narrative.
- **Blog**: launch with 2–3 seed posts (e.g., "What I learned shipping PRs to VS Code", "NestJS CLI watch mode internals"). Cadence target: 1 post / 6 weeks.
- **`/now`**: updated monthly.
- **`/uses`**: updated as setup changes.

All content is checked into the repo as MDX or JSON. No external CMS.

---

## 9. Repository & Deployment

- **New repo**: `maruthang/portfolio` (separate from `MaruthanG/MaruthanG` which keeps the profile README).
- **Branching**: `main` auto-deploys to production. PR previews via Vercel.
- **Env vars** (in Vercel): `RESEND_API_KEY`, `GITHUB_TOKEN` (for OSS data fetch), `CONTACT_EMAIL_TO`.
- **CI**: GitHub Actions for lint + typecheck + build on PR. Vercel handles deploy.
- **Domain**: To be acquired. Recommendation `maruthan.dev`. Configured via Vercel DNS.

---

## 10. Out of Scope (v2 backlog)

- Guestbook with GitHub OAuth.
- Newsletter system (Resend Audiences / Buttondown).
- Spotify "now playing" widget.
- Hidden easter eggs (Konami code, terminal commands beyond /404).
- i18n.
- Comments on blog posts.

---

## 11. Success Criteria

- Lighthouse ≥95 across all four categories on home, blog index, blog post, project detail.
- All sections render correctly across viewports 360px → 2560px.
- Theme toggle persists with no FOUC.
- Command palette opens in <100ms after `⌘K`.
- OSS table loads with current PR counts within 2s on 3G fast.
- Contact form delivers email reliably; success/failure states visible.
- All MDX code blocks have working copy buttons and correct syntax highlighting.
- Reduced-motion users see no animations beyond fade.
- Verifiable links: every project has a working live demo OR repo link OR is marked "Archived".
- Spec-to-launch: ≤4 weeks part-time.

---

## 12. Open Questions

- Final domain name: `maruthan.dev` vs `maruthan.com` vs other.
- Brand accent color: stick with VSCode blue `#58a6ff` or pick a different signature color.
- Avatar: real photo, illustrated avatar, or initials-only.
- Cal.com account: existing or create new.
- Resume PDF: maintain a checked-in PDF or generate from data via `react-pdf`.
