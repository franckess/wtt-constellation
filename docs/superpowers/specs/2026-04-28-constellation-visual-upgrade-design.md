# Constellation Visual Upgrade — Design Spec

**Date:** 2026-04-28
**Scope:** 5 visual/interaction improvements to the WTT Constellation page
**Approach:** Pure CSS/SVG — zero dependencies, single-file architecture preserved

---

## Summary

Upgrade the constellation page with five features: vivid animated nebula backdrop, comet cursor trail, related-episodes path highlighting, season orbit rings, and search/guest spotlight. All implementations use vanilla CSS, SVG, and JS inside the existing `index.html` monolith.

---

## 1. Vivid Nebula Clouds

**Goal:** Transform the flat dark backdrop into a rich, James Webb-style deep space atmosphere with drifting color clouds.

**Current state:** `body::before` has 3 radial gradients at 0.04–0.08 opacity plus a `feTurbulence` noise overlay at 0.04 opacity.

**Changes:**

- Add a new fixed-position `<div class="nebula">` layer between `body::before` (z-index 0) and `.bg-stars` (z-index 2), at z-index 1. This avoids pseudo-element limits on `body`.
- The `.nebula` div contains 3 child divs (`.nebula-layer`), each with 2 radial gradients, for 6 total gradient blobs.
- Colors drawn from the theme palette:
  - Teal: `rgba(125, 211, 192, 0.18)`
  - Gold: `rgba(235, 179, 34, 0.20)`
  - Purple: `rgba(167, 139, 250, 0.14)`
  - Pink: `rgba(209, 131, 201, 0.10)`
  - Blue: `rgba(102, 197, 212, 0.12)`
  - Warm orange: `rgba(255, 142, 114, 0.10)`
- Each layer has its own `@keyframes nebulaDrift-N` animation:
  - Cycle durations: 35s, 45s, 55s (staggered so they never sync)
  - Movement: `transform: translate(Xpx, Ypx)` keyframes shifting ±30–50px
  - `ease-in-out` timing
- Layers use `mix-blend-mode: screen` so colors add rather than occlude
- The existing `body::after` noise overlay remains on top for grain texture
- The nebula layer also responds to parallax via `--px`/`--py` CSS variables (at 0.2x factor — slower than stars, creating depth)
- `prefers-reduced-motion`: all nebula animations frozen (set `animation: none`)

**Why a new div instead of more pseudo-elements:** `body` already uses `::before` and `::after`. Adding a real DOM element gives us 3 layers (each with its own `::before`/`::after` if needed) without any CSS hacks.

---

## 2. Comet Cursor Trail

**Goal:** A faint stardust trail following the mouse cursor across the starfield.

**Implementation:**

- `mousemove` listener on `document` (not just `.stage`, so it works across the full viewport)
- Throttled to fire every 40ms (~25 particles/second)
- Each tick creates an SVG `<circle>` appended to a dedicated `<svg class="cursor-trail">` overlay (fixed position, z-index 6 — above the `.stage` at z-index 5 so trail renders over stars, pointer-events: none)
  - Radius: 1.5px
  - Fill: `var(--star-warm)`
  - Initial opacity: 0.6
- Each particle has a CSS animation `.trail-particle`:
  - Duration: 500ms
  - Opacity: 0.6 → 0
  - Scale: 1 → 0.3
  - Transform: translateY(+8px) — slight downward drift
  - `forwards` fill mode
- Particle removes itself from DOM via `animationend` event listener
- Coordinate mapping: `mousemove` gives viewport coords; the SVG viewBox matches the viewport, so coords map 1:1
- **Disabled when:**
  - `reducedMotion` is true
  - Touch device detected (`'ontouchstart' in window`)
  - Document is hidden (`document.hidden`)

**Performance:** At 25 particles/sec × 500ms lifetime = max ~13 particles in DOM at once. Lightweight.

---

## 3. Related Episodes Path

**Goal:** When an episode panel is open, visually illuminate connected episodes and suggest a "listen next" path.

**Trigger:** Inside `openPanel(epId)`, after rendering the panel content.

**Star highlighting:**

- Find all episodes sharing at least one theme with the active episode (using `DATA.episodes` and the active episode's `themes[]`)
- Add CSS class `.related` to their `.star-group` elements
- `.related` styles:
  - `.star-halo`: opacity 0.25, scale 1.15 (brighter than default idle, dimmer than hover)
  - `.star-core`: filter with slightly stronger glow
  - `.star-label`: fill `var(--ink)` (full brightness)
- Stars not related and not active get `.dimmed` (same as existing theme filter dimming)

**Connection line animation:**

- Lines connecting the active star to related stars get class `.path-glow`
- `.path-glow` styles:
  - `stroke: var(--line-active)` (gold at 0.35 opacity)
  - `stroke-width: 1.5`
  - `stroke-dasharray: 6 4`
  - `animation: pathFlow 2s linear infinite` — cycles `stroke-dashoffset` from 0 to -20 (traveling light effect)
- All other connection lines get `.dimmed`

**"Listen next" suggestion in panel:**

- After the "Listen" links section, add a new section: `<div class="panel-section-title">Explore next</div>`
- Pick the related episode with the most shared themes (tie-break: most total connections across all themes)
- Render as a clickable mini-card:
  ```html
  <button class="panel-next" onclick="openPanel('s01e07')">
    <div class="guest-avatar">HM</div>
    <div>
      <div class="guest-name">Leadership Mindset in the AI Era</div>
      <div class="guest-role">Hannah Maude</div>
    </div>
  </button>
  ```
- `.panel-next` styled as a horizontal flex row with hover glow, matching panel aesthetic
- Clicking it calls `openPanel()` for that episode (panel re-renders, path updates)

**Cleanup:** `closePanel()` removes `.related`, `.path-glow`, and `.dimmed` from all stars and lines (same cleanup pattern as existing theme filter).

**Interaction with theme filter:** If a theme filter is active when a panel opens, the related-episodes path is NOT applied (theme filter takes precedence, matching existing hover behavior).

---

## 4. Season Orbit Rings

**Goal:** Faint concentric rings visually grouping episodes by season.

**Computation (inside `render()`):**

- After `computePositions()`, group placed episodes by `season`
- For each season, calculate: average radius from center `(cx, cy)` across all episodes in that season
- Store as `seasonRings = [{ season: 1, radius: 142 }, { season: 2, radius: 98 }]` (example values)

**Rendering:**

- Create an SVG `<g class="season-rings">` group, inserted before the connections group (so rings are behind everything)
- For each season ring:
  - SVG `<circle>` centered at `(cx, cy)` with `r = averageRadius`
  - `stroke: rgba(255, 247, 230, 0.04)`
  - `stroke-dasharray: 4 8`
  - `fill: none`
  - `stroke-width: 1`
  - CSS class `.season-ring`
- Season label: SVG `<text>` positioned at top of ring (cx, cy - radius - 8):
  - Text: `S1`, `S2`, etc.
  - Font: JetBrains Mono, 8px
  - Fill: `var(--ink-faint)` at 0.5 opacity
  - `text-anchor: middle`

**Entry animation:**

- `.season-ring` starts with `opacity: 0`
- Fades in via `@keyframes fadeRing` with a 2.5s delay (after stars have loaded)

**Edge case:** If a season has only 1 episode, the ring radius equals that episode's distance from center — still valid as a ring.

---

## 5. Search / Guest Spotlight

**Goal:** Let visitors find a specific episode or guest by name, with a spotlight animation on the matching star.

**UI placement:**

- A search container `<div class="search">` positioned fixed, bottom-left, above the legend. The legend sits at `bottom: 24px` and is ~80px tall, so search sits at `bottom: 120px`, `left: 40px`, z-index 15
- Contains an `<input class="search-input">` styled as:
  - Pill shape (border-radius: 100px)
  - Background: `rgba(14, 16, 48, 0.6)` with `backdrop-filter: blur(6px)`
  - Border: `1px solid var(--panel-edge)`, gold on focus
  - Font: JetBrains Mono, 11px
  - Placeholder: `"Find episode or guest..."`
  - Width: 220px (expands to 280px on focus via transition)
- Below the input: `<div class="search-results">` dropdown, hidden by default

**Search logic (JS):**

- On `input` event, run a case-insensitive substring match against:
  - `episode.title`
  - `episode.guest.name`
- Filter to episodes where either matches
- Show top 5 results in `.search-results` as `<button class="search-result">` items:
  - Each shows: guest initials avatar (small, 28px) + episode title + guest name
  - Styled like mini panel-guest cards

**Spotlight animation on selection:**

1. Close any open panel
2. Clear any active theme filter
3. Add `.spotlight` class to the matched `.star-group`:
   - `.spotlight .star-halo`: animated scale from 1 to 2.5 and back, opacity peak at 0.5, duration 1.5s — a radial gold "sonar ping"
   - `.spotlight .star-core`: scale 1.6 briefly
4. After 400ms delay, call `openPanel(epId)` for the matched episode
5. `.spotlight` class auto-removed after 1.5s (via `setTimeout`)

**Dismiss:** Escape key or blur on input hides the dropdown and clears the search.

**Mobile (≤768px):** Search input moves to top of page below the header, full width with padding. Dropdown overlays the constellation.

---

## Shared Concerns

**Performance:**
- Nebula: 3 divs with gradients + CSS transforms = lightweight. No JS per-frame rendering.
- Cursor trail: max ~13 SVG circles in DOM. Negligible.
- All new animations respect `prefers-reduced-motion: reduce` — frozen or hidden.

**Accessibility:**
- Search input has `aria-label="Search episodes"`
- Search results are `role="listbox"` with `role="option"` items
- Spotlight animation is decorative (no information conveyed solely through it)
- Season ring labels are presentational SVG text (not interactive)

**Mobile:**
- Comet cursor trail: disabled on touch devices (no mousemove)
- Search: repositioned to top, full width
- Season rings: unchanged (they scale with the SVG viewBox)
- Related episodes path: works the same (panel is bottom-sheet on mobile)

**File changes:** All changes are in `index.html` only. No changes to `episodes.json` or any other file.
