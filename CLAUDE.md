# Sim2Real v2 — Project Notes for Claude

## Overview

This is the marketing/preview site for **Sim2Real**, a robotics sim-to-real transfer startup currently in stealth. It is a React + TypeScript + Vite SPA deployed to GitHub Pages via GitHub Actions (Bun-based build).

**Two repos exist:**
- `joetol2/sim2real-v2` — the working development repo (this one)
- `joetol2/sim2real` — the production repo where the custom domain points

Changes are developed here in `sim2real-v2`, then synced to `sim2real` (either via the MCP `push_files` tool if both repos are in scope, or by the user running `git push` from their Mac terminal). Both repos have identical code and the same GitHub Actions deploy workflow.

The custom domain is configured on `joetol2/sim2real`. GitHub Pages serves from the `gh-pages` environment artifact built by the workflow.

---

## Tech Stack

- **React 18 + TypeScript**
- **Vite** (with `@vitejs/plugin-react-swc`)
- **Tailwind CSS** (utility classes throughout)
- **React Router v6** (`BrowserRouter` with `basename="/"`)
- **@tanstack/react-query**
- **Bun** (package manager and build runner in CI)
- **GitHub Actions** → GitHub Pages deployment on push to `main`

Key config files:
- `vite.config.ts` — `base: "/"` (important: must stay `/` for custom domain; was `/sim2real-v2/` before migration)
- `public/404.html` — SPA redirect hack for GitHub Pages (encodes path into query string)
- `index.html` — decodes the 404 redirect back into real URL on load

---

## Site Structure

### Pages (`src/pages/`)

| Route | File | Notes |
|---|---|---|
| `/` | `Index.tsx` | Homepage with hero, feature sections, Easter egg link |
| `/product` | `Product.tsx` | Product detail with 2x2 positioning card grid |
| `/use-cases` | `UseCases.tsx` | Use case sections |
| `/about` | `About.tsx` | Team + company info |
| `/contact` | `Contact.tsx` | Contact info page |
| `/demos` | `Demos.tsx` | Demo videos page |
| `/physics` | `Physics.tsx` | Physics simulation detail |
| `/models` | `Models.tsx` | Model detail |
| `/press` | `Press.tsx` | Press/media |
| `/see-it-in-action` | `SeeItInAction.tsx` | **Hidden page** — password gated, not in nav |

`/see-it-in-action` is intentionally excluded from the main navigation. It is a secret page accessible only via:
1. A hidden eyeball icon in the footer (left of social icons, `opacity-0 hover:opacity-100`)
2. A hidden link on "Results, not promises" heading on the homepage — eye icon appears on hover

### Key Components (`src/components/`)

- **`PasswordGate.tsx`** — Wraps `SeeItInAction`. Password: `s2rVIP`. Uses `sessionStorage` key `sim2real_gate_ok`. Renders blurred children behind a frosted glass overlay modal. Animates out on correct password.
- **`Footer.tsx`** — Contains the animated eyeball Easter egg (`Eye` SVG component) and `SocialIcons`. The eyeball triggers a pop animation (scales to 2x then springs back) followed by a slow realistic blink (10s cycle, ~300ms blink duration).
- **`PageNav.tsx`** — Top navigation bar used on all pages.
- **`ScrollToTop.tsx`** — Scrolls to top on route change.
- **`Section.tsx`** (if present) — Wrapper component used on Contact and other pages for consistent section styling.

---

## Layout Width Conventions

Established and applied consistently across all pages:

- **`max-w-4xl`** — all text-only content sections (heroes, paragraphs, CTAs, intro text)
- **`max-w-6xl`** — card grids, team grids, multi-column layouts

**Important pattern:** Section outer divs always span full width (for dark/colored backgrounds to bleed edge-to-edge). The `max-w` constraint is applied on an **inner wrapper div** inside the section, not on the section itself. This prevents background clipping.

Example:
```tsx
<section className="py-24 bg-white/5">
  <div className="max-w-4xl mx-auto px-6">
    {/* content */}
  </div>
</section>
```

---

## Easter Egg — "See It In Action"

Two hidden entry points, both invisible until hover:

### 1. Footer Eyeball (`src/components/Footer.tsx`)
- `Eye` SVG component with three separate path elements:
  - Bottom arc (static, `fill="none"`)
  - Top lid (`className="eye-lid"`, animated with `scaleY`)
  - Pupil (`className="eye-pupil"`, fades during blink)
- All paths have explicit `fill="none"` to prevent white corner fill artifacts
- Wrapped in a `Link` to `/see-it-in-action` with `opacity-0 hover:opacity-100`
- Animation: on hover → `eye-pop` class (scale 2x, spring back) → after 500ms → `eye-blink` class (10s cycle blink)
- Uses `animState: 'idle' | 'pop' | 'blink'` state + `timerRef` to chain animations

### 2. Homepage Heading Link (`src/pages/Index.tsx`)
- `ResultsLink` component wraps the "Results, not promises" h2
- Same `animState` + `timerRef` pattern as footer eyeball
- Eye icon only appears on hover (hidden at rest)
- Links to `/see-it-in-action`

---

## CSS Animations (`src/index.css`)

Key custom keyframes for the eyeball Easter egg:

```css
/* Pop: scale up to 2x then spring back */
@keyframes eye-pop { ... }
.animate-eye-pop { animation: eye-pop 0.5s ease forwards; }

/* Blink: top lid collapses down (scaleY 0) for ~300ms every 10s */
@keyframes eye-lid-close {
  0%, 3%, 100% { transform: scaleY(1); }
  1%, 2%       { transform: scaleY(0); }
}
.animate-eye-blink .eye-lid {
  transform-origin: 12px 12px;  /* center of 24x24 SVG */
  animation: eye-lid-close 10s linear infinite;
}
.animate-eye-blink .eye-pupil {
  animation: eye-pupil-hide 10s linear infinite;
}
```

Password gate CSS classes: `.gate-card`, `.gate-input-wrap`, `.gate-btn`, `.gate-error`, `@keyframes gate-in`, `@keyframes gate-shake`

---

## Demo Videos (`src/assets/videos/`)

Used on the `/see-it-in-action` page (3 demo sections):
- `cube_stack.mp4` — stacking cubes demo
- `dice_sort.mp4` — dice sorting demo
- `dice_throw_clean.mp4` — dice throwing demo

Other videos exist in this folder (used on other pages):
- `header_01.mp4`, `0.mp4`, `6qNvYpRr.mp4`, `Multi-Shot.mp4`, `sim4real_Dan_Video_resized_480.mov`, `tycho_jr_pit_in_cup.mp4`, `tycho_jr_socks_v01.mp4`

---

## Design System

- **Background:** Fixed gradient `linear-gradient(to top right, #016fb5, #012b62)` — deep navy/blue
- **Fonts:** Space Grotesk (headings), Inter (body) — loaded from Google Fonts
- **Color tokens** (CSS vars in `src/index.css`): `--background: 212 70% 18%`, `--primary: 210 100% 60%`
- **Card style:** `rounded-2xl border border-white/10 bg-white/5` with optional `backdrop-blur`
- **Text hierarchy:** white for headings, `white/80` or `white/60` for body, `white/40` for muted

---

## Deployment Workflow

1. Push to `main` branch of either repo triggers GitHub Actions
2. Workflow: checkout → setup Bun → `bun install` → `bun run build` → deploy `dist/` to GitHub Pages
3. `sim2real-v2` deploys to `joetol2.github.io/sim2real-v2` (preview)
4. `sim2real` deploys to the custom domain (production)

**MCP tool note:** In Claude Code web sessions, the `mcp__github__push_files` tool triggers the deploy workflow automatically. After using it, always run `git checkout -- . && git pull origin main` locally to re-sync the working tree before making further edits.

---

## Known Constraints

- **GitHub Pages = static only** — no server-side logic. The password gate (`PasswordGate.tsx`) is client-side only. The password (`s2rVIP`) is visible in the source bundle. This is acceptable as a soft internal gate, not a security boundary.
- **SPA routing on GitHub Pages** — handled via `public/404.html` redirect hack + decoder script in `index.html`. Do not remove these.
- **`base: "/"`** in `vite.config.ts` — must remain `/` for the custom domain deployment. Changing it to a subpath would break `sim2real` production.
- **React deduplication** — `resolve.dedupe` in `vite.config.ts` prevents duplicate React instances from causing hook errors with certain dependencies.
