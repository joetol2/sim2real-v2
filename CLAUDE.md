# Sim2Real — Project Notes for Claude

## What this repo is

This is the **production marketing/preview site** for **Sim2Real**, a robotics startup in stealth. The site is deployed to GitHub Pages at a custom domain via GitHub Actions.

There are two repos:
- `joetol2/sim2real` — **production repo, custom domain points here. This is the one to work in.**
- `joetol2/sim2real-v2` — the original development repo. No longer the primary. May be used for staging/preview.

Both repos are identical in code. A backup of the old site exists in branch `backup-pre-v2` on `joetol2/sim2real`.

---

## What Sim2Real does (company context)

Sim2Real is the missing link between simulation and real-world robot deployment. Robotics teams train policies in simulation but struggle to transfer them to physical hardware. Sim2Real provides the workflow layer that makes that transfer reliable.

**Key differentiator:** No human demonstrations required. Their method generates synthetic training data at scale from simulation, trains policies on only what a real robot can observe (camera, joint state, proprioception), and screens policies before expensive hardware rollout.

**Proven results:**
- 100% pick-and-place success — 800/800 trials, zero human demos, zero real-robot training steps
- 4,000 synthetic episodes outperform 22,000 human demonstration episodes (100% vs 94% success)
- 91% success on block stacking
- Works across manipulation, locomotion, navigation, tool use, human-robot interaction
- Compatible with Isaac Sim, MuJoCo, PyBullet, Gazebo, Webots, Genesis

**Tone:** Confident, technical, understated. Not hype-driven. Copy avoids em dashes. Headers use sentence case. The brand voice is "we built something that works and we can prove it."

---

## Tech Stack

- **React 18 + TypeScript**
- **Vite** with `@vitejs/plugin-react-swc`
- **Tailwind CSS** (utility classes throughout)
- **React Router v6** — `BrowserRouter` with `basename="/"`
- **@tanstack/react-query**
- **Bun** — package manager and build runner in CI
- **GitHub Actions** — deploys `dist/` to GitHub Pages on push to `main`

### Critical config notes
- `vite.config.ts` — `base: "/"` — must stay `/`. Changing to a subpath breaks the custom domain.
- `resolve.dedupe` in `vite.config.ts` — prevents duplicate React instances with certain deps. Do not remove.
- `public/404.html` — SPA routing hack for GitHub Pages (encodes path into query string on 404)
- `index.html` — decodes that redirect back into the real URL on load. Both files are required for deep-link routing to work on GitHub Pages.

---

## Deployment

1. Push to `main` triggers GitHub Actions
2. Workflow: checkout → setup Bun → `bun install` → `bun run build` → deploy `dist/` to GitHub Pages
3. The workflow file is at `.github/workflows/` — uses `actions/deploy-pages@v5`

**After using MCP `push_files` tool:** Always run `git checkout -- . && git pull origin main` locally to re-sync the working tree before making further edits. The MCP push does not update the local working copy.

---

## Navigation

### Top nav (`src/components/PageNav.tsx`)
Links (in order): **Product**, **Use Cases**, **About**, **Contact**

The `/see-it-in-action` page is **intentionally excluded** from nav. It is a secret/Easter egg page.

### Footer nav (`src/components/Footer.tsx`)
Same four links. Also contains social icons: Instagram, X (Twitter), YouTube, Substack, Discord.
The eyeball Easter egg sits **to the left** of the social icons, hidden until hover.

---

## Pages (`src/pages/`)

### `/` — `Index.tsx`
The homepage. Uses local `Section`, `Label`, `Heading`, `Body` helper components (defined at the top of the file, not imported). Also uses `HeroSection` component for the top hero.

Section breakdown:
1. **"The problem"** — stat callout (3-5x), explains sim-to-real deployment cost problem
2. **"What Sim2Real does"** (dark) — bullet list of 4 workflow benefits
3. **"The differentiator"** — "No humans in the loop", CTA button → `/product`
4. **"What we have proven"** (dark) — `ResultsLink` Easter egg heading + 3 stat cards (100%, 4k>22k, Any task)
5. **"The bottleneck"** — bullet list of 3 sim workflow failure modes
6. **"How it works"** (dark) — 4-step 2x2 card grid (01-04)
7. **"Infrastructure"** — "Built on sims, not against them", lists compatible sim platforms
8. **"Who this is for"** (dark) — target audience description
9. **Final CTA** — "Talk to us" button → `/contact`

### `/product` — `Product.tsx`
Product detail page. Contains a **2x2 positioning card grid** comparing "Simulators" vs "Sim2Real" across 4 dimensions. Each card has a label, a "Simulators" row, and a "Sim2Real" row.

### `/use-cases` — `UseCases.tsx`
Multiple use case sections. Each uses max-w-4xl inner wrapper.

### `/about` — `About.tsx`
Team + company info. Team grid uses max-w-6xl.

### `/contact` — `Contact.tsx`
Contact page styled with hero label "Get in touch". Uses `Section` component. max-w-4xl throughout.

### `/demos` — `Demos.tsx`
Demo videos page.

### `/physics` — `Physics.tsx`
Physics simulation detail page.

### `/models` — `Models.tsx`
Model detail page.

### `/press` — `Press.tsx`
Press/media page.

### `/see-it-in-action` — `SeeItInAction.tsx` ⚠️ SECRET PAGE
Password-gated. Not in nav. Accessible only via Easter egg entry points (see below).

Hero: "See it in action" — "Three tasks. One method. No human demonstrations."

Three demo sections (alternating dark/light, 2-column grid — video left/right alternates):
- **Task 01 — Block stacking**: `cube_stack.mp4`, stat: 91% success rate
- **Task 02 — Color-conditioned sorting**: `dice_sort.mp4`, stat: 100% pick-and-place success (800/800 trials)
- **Task 03 — Dynamic throwing**: `dice_throw_clean.mp4`, stat: 4k > 22k (synthetic beats human data)

---

## Layout Width Conventions

Applied consistently across all pages:

- **`max-w-4xl`** — all text-only sections (heroes, paragraphs, CTAs, intro text)
- **`max-w-6xl`** — card grids, team grids, multi-column layouts, Section outer containers

**Critical pattern:** Section outer `<section>` elements always span **full width** so background colors bleed edge-to-edge. The `max-w` constraint goes on an **inner wrapper `<div>`** inside the section, never on the section itself.

```tsx
// CORRECT
<section className="py-24 bg-white/5">
  <div className="max-w-4xl mx-auto px-6">
    {/* content */}
  </div>
</section>

// WRONG — clips the background
<section className="py-24 bg-white/5 max-w-4xl mx-auto">
  {/* content */}
</section>
```

---

## Design System

- **Background:** `linear-gradient(to top right, #016fb5, #012b62)` — fixed deep navy/blue. Set on `body` with `background-attachment: fixed`.
- **Dark sections:** `backgroundColor: '#012b62'` (inline style, not Tailwind — avoids Tailwind purge issues)
- **Fonts:** Space Grotesk (headings, `font-heading`), Inter (body, `font-body`) — Google Fonts
- **CSS vars** (`src/index.css`): `--background: 212 70% 18%`, `--primary: 210 100% 60%`, `--border: 212 30% 30%`
- **Card style:** `rounded-2xl border border-white/10 bg-white/5` with optional `backdrop-blur`
- **Text hierarchy:**
  - Headings: `text-foreground` (white)
  - Body: `text-white/70` or `text-white/80`
  - Muted: `text-white/60` or `text-white/40`
  - Labels/eyebrow text: `text-muted-foreground`, `tracking-[0.3em] uppercase text-sm font-heading`
- **Buttons:** `bg-white text-[#012b62] font-heading font-semibold tracking-wide uppercase rounded`
- **Scroll reveal:** `useScrollReveal` hook — fades + translates up on enter. Used on all section inner wrappers.

---

## Easter Egg — "See It In Action"

Two hidden entry points. Both are `opacity-0` at rest and `opacity-100` on hover. Both animate identically.

### Entry point 1 — Footer eyeball (`src/components/Footer.tsx`)

The `Eye` SVG component has **three separate path elements** (this is important — merging them causes white corner fill artifacts):
```tsx
const Eye = ({ className = "" }: { className?: string }) => (
  <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="1.5" strokeLinecap="round" strokeLinejoin="round" className={...}>
    <path fill="none" d="M2 12 C5.5 19 18.5 19 22 12" />        {/* bottom arc — static */}
    <path fill="none" className="eye-lid" d="M2 12 C5.5 5 18.5 5 22 12" />  {/* top lid — animates scaleY */}
    <circle fill="none" className="eye-pupil" cx="12" cy="12" r="3" />       {/* pupil — fades on blink */}
  </svg>
);
```

All paths need explicit `fill="none"` — without it the browser fills enclosed shapes white.

Animation state machine in `SocialIcons` component:
```tsx
const [eyeAnim, setEyeAnim] = useState<'idle' | 'pop' | 'blink'>('idle');
const timerRef = React.useRef<...>(null);

// On hover: pop → wait 500ms → blink
setEyeAnim('pop');
timerRef.current = setTimeout(() => setEyeAnim('blink'), 500);
```

The `Link` wrapping the eye has `opacity-0 hover:opacity-100 transition-opacity duration-300`.

### Entry point 2 — "Results, not promises" heading (`src/pages/Index.tsx`)

`ResultsLink` component — wraps the heading in a `Link to="/see-it-in-action"`. Identical `animState` + `timerRef` pattern. The eye SVG is inline (not the `Eye` component) and uses `opacity-0 group-hover:opacity-100`.

---

## CSS Animations (`src/index.css`)

### Eye pop (on hover)
```css
@keyframes eye-pop {
  0%   { transform: scale(1); }
  40%  { transform: scale(2); }
  70%  { transform: scale(0.9); }
  100% { transform: scale(1); }
}
.animate-eye-pop {
  animation: eye-pop 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}
```

### Eye blink (after pop, loops every 10s)
```css
@keyframes eye-lid-close {
  0%, 3%, 100% { transform: scaleY(1); }
  1%, 2%       { transform: scaleY(0); }   /* ~300ms closed at 10s cycle */
}
@keyframes eye-pupil-hide {
  0%, 3%, 100% { opacity: 1; }
  1%, 2%       { opacity: 0; }
}
.animate-eye-blink .eye-lid {
  transform-origin: 12px 12px;  /* center of 24x24 viewBox — lid folds down to bottom arc */
  animation: eye-lid-close 10s linear infinite;
}
.animate-eye-blink .eye-pupil {
  animation: eye-pupil-hide 10s linear infinite;
}
```

### Hero fade-up animations
```css
@keyframes fade-up { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
.animate-fade-up          { animation: fade-up 0.8s ease-out both; }
.animate-fade-up-delay-1  { animation: fade-up 0.8s ease-out 0.15s both; }
.animate-fade-up-delay-2  { animation: fade-up 0.8s ease-out 0.3s both; }
.animate-fade-up-delay-3  { animation: fade-up 0.8s ease-out 0.45s both; }
```

### Password gate
```css
.gate-card        { frosted glass card, max-width 400px, animates in with gate-in keyframe }
.gate-input-wrap  { flex row with input + arrow button, error state turns border red }
.gate-input-wrap.shake { gate-shake keyframe — horizontal jitter on wrong password }
.gate-btn         { arrow submit button, right side of input }
.gate-error       { red error text, opacity-0 by default, .visible shows it }
```

---

## Password Gate (`src/components/PasswordGate.tsx`)

- **Password:** `s2rVIP`
- **sessionStorage key:** `sim2real_gate_ok` (value `"1"`)
- Once unlocked, stays unlocked for the browser session (survives page refresh, lost on tab close)
- Status states: `'locked'` | `'unlocking'` | `'unlocked'`
- On correct password: sets `unlocking` → fades gate out (0.55s) → sets `unlocked` → renders children only
- On wrong password: sets `shake` → shakes input → clears input
- Children render **behind** the gate (blurred) while locked — unblur animates on unlock
- **Security note:** Password is visible in the JS bundle. This is intentional — it's a soft internal gate for sharing with VIPs, not a real security boundary.

---

## Key Components

### `src/components/HeroSection.tsx`
Used only on the homepage (`Index.tsx`). Contains the main hero video/content at the top of the page.

### `src/components/PageNav.tsx`
Top nav bar. Used on all pages except `Index.tsx` (which uses `HeroSection` which has its own nav). Links: Product, Use Cases, About, Contact. Has mobile hamburger menu.

### `src/components/Footer.tsx`
Used on all pages. Contains nav links + social icons + hidden eyeball Easter egg. Social icons: Instagram, X, YouTube, Substack, Discord (all currently link to `#`).

### `src/components/ScrollToTop.tsx`
Scrolls to top on route change. Rendered once in `App.tsx`.

### `src/components/PasswordGate.tsx`
Wraps `SeeItInAction` page. See above.

### `src/hooks/useScrollReveal.ts`
Returns `{ ref, isVisible }`. Attach `ref` to any element — it becomes visible (fade + slide up) when scrolled into view. Uses `IntersectionObserver`.

### Other components
`FocusCards`, `HeroSection`, `DemosSection`, `PositioningSection`, `VideoShowcase`, `SimulationSection`, `StealthStatement`, `ContactCTA`, `NavLink`, `RobotViewer` — used on various inner pages.

---

## Videos (`src/assets/videos/`)

Used on `/see-it-in-action`:
- `cube_stack.mp4` — block stacking demo
- `dice_sort.mp4` — color-conditioned sorting demo
- `dice_throw_clean.mp4` — dynamic throwing demo

Used on other pages:
- `header_01.mp4`, `0.mp4`, `6qNvYpRr.mp4`, `Multi-Shot.mp4`
- `sim4real_Dan_Video_resized_480.mov`
- `tycho_jr_pit_in_cup.mp4`, `tycho_jr_socks_v01.mp4`

---

## App.tsx Routes

```tsx
<Route path="/" element={<Index />} />
<Route path="/product" element={<Product />} />
<Route path="/use-cases" element={<UseCases />} />
<Route path="/about" element={<About />} />
<Route path="/contact" element={<Contact />} />
<Route path="/demos" element={<Demos />} />
<Route path="/physics" element={<Physics />} />
<Route path="/models" element={<Models />} />
<Route path="/press" element={<Press />} />
<Route path="/see-it-in-action" element={<SeeItInAction />} />
<Route path="*" element={<NotFound />} />
```

---

## Known Constraints & Gotchas

- **GitHub Pages = static only.** No server-side logic. Password gate is client-side only.
- **SPA routing hack:** `public/404.html` and the decoder script in `index.html` are required for deep links to work. Do not remove either file.
- **`base: "/"`** in `vite.config.ts` must stay `/`. It was previously `/sim2real-v2/` — changing it back breaks the custom domain.
- **React deduplication:** `resolve.dedupe` in `vite.config.ts` prevents hook errors. Keep it.
- **Dark sections use inline style** `backgroundColor: '#012b62'` not a Tailwind class — safer against purge.
- **Eye SVG fill:** All three eye path elements need explicit `fill="none"`. Without it, the browser treats intersecting arcs as an enclosed region and fills it white.
- **After MCP push:** Run `git checkout -- . && git pull origin main` to sync local tree.
