# Constellation Visual Upgrade Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add 5 visual/interaction features to the WTT Constellation page: vivid nebula clouds, comet cursor trail, related episodes path, season orbit rings, and search/guest spotlight.

**Architecture:** All changes go into a single file — `index.html`. New CSS is inserted into the existing `<style>` block. New HTML elements are added to the `<body>`. New JS functions are added to the existing `<script>` block and wired into the boot sequence. Zero external dependencies.

**Tech Stack:** Vanilla HTML5, CSS3 (keyframes, custom properties, mix-blend-mode), SVG (namespace-aware element creation), ES6+ JavaScript.

**Spec:** `docs/superpowers/specs/2026-04-28-constellation-visual-upgrade-design.md`

---

## File Structure

All changes are in a single file:

- **Modify:** `index.html`
  - CSS additions: after line 826 (before `</style>`)
  - HTML additions: after line 843 (after `<svg class="bg-stars" ...>`) and after line 861 (after `</aside>` for panel)
  - JS additions: after line 1263 (after `formatDate`) and modifications to `render()`, `openPanel()`, `closePanel()`, and the boot sequence

---

### Task 1: Vivid Nebula Clouds — CSS

**Files:**
- Modify: `index.html` (CSS block, lines 816–827)

- [ ] **Step 1: Add nebula CSS after the reduced-motion block**

Insert the following CSS immediately before the closing `</style>` tag (line 827):

```css
  /* ============ NEBULA CLOUDS ============ */
  .nebula {
    position: fixed;
    inset: -60px;
    z-index: 1;
    pointer-events: none;
    transform: translate(calc(var(--px) * 0.2), calc(var(--py) * 0.2));
    transition: transform 1.6s cubic-bezier(0.22, 1, 0.36, 1);
  }

  .nebula-layer {
    position: absolute;
    inset: 0;
    mix-blend-mode: screen;
  }

  .nebula-layer:nth-child(1) {
    background:
      radial-gradient(ellipse 55% 45% at 18% 25%, rgba(125, 211, 192, 0.18), transparent 65%),
      radial-gradient(ellipse 45% 55% at 82% 70%, rgba(235, 179, 34, 0.20), transparent 60%);
    animation: nebulaDrift1 35s ease-in-out infinite;
  }

  .nebula-layer:nth-child(2) {
    background:
      radial-gradient(ellipse 50% 40% at 65% 20%, rgba(167, 139, 250, 0.14), transparent 65%),
      radial-gradient(ellipse 40% 50% at 30% 75%, rgba(209, 131, 201, 0.10), transparent 60%);
    animation: nebulaDrift2 45s ease-in-out infinite;
  }

  .nebula-layer:nth-child(3) {
    background:
      radial-gradient(ellipse 60% 35% at 50% 50%, rgba(102, 197, 212, 0.12), transparent 65%),
      radial-gradient(ellipse 35% 55% at 75% 40%, rgba(255, 142, 114, 0.10), transparent 60%);
    animation: nebulaDrift3 55s ease-in-out infinite;
  }

  @keyframes nebulaDrift1 {
    0%, 100% { transform: translate(0, 0); }
    25%      { transform: translate(30px, -20px); }
    50%      { transform: translate(-15px, 35px); }
    75%      { transform: translate(-30px, -10px); }
  }

  @keyframes nebulaDrift2 {
    0%, 100% { transform: translate(0, 0); }
    25%      { transform: translate(-25px, 40px); }
    50%      { transform: translate(35px, 15px); }
    75%      { transform: translate(20px, -35px); }
  }

  @keyframes nebulaDrift3 {
    0%, 100% { transform: translate(0, 0); }
    25%      { transform: translate(40px, 25px); }
    50%      { transform: translate(-30px, -40px); }
    75%      { transform: translate(-15px, 50px); }
  }
```

- [ ] **Step 2: Update the `body::after` noise overlay z-index**

The noise overlay at line 112 currently has `z-index: 1`. Change it to `z-index: 2` so it sits above the nebula and `.bg-stars` gets `z-index: 3`. Find this CSS:

```css
  body::after {
    content: '';
    position: fixed; inset: 0;
    background-image: url("data:image/svg+xml,...");
    opacity: 0.04;
    pointer-events: none;
    z-index: 1;
    mix-blend-mode: overlay;
  }
```

Change `z-index: 1;` to `z-index: 2;`.

- [ ] **Step 3: Bump z-index values for layers above the nebula**

Update the following z-index values to make room for the nebula at z-index 1 and noise at z-index 2:

| Selector | Old z-index | New z-index | Line |
|---|---|---|---|
| `.bg-stars` | 2 | 3 | ~119 |
| `.shooting-star` | 3 | 4 | ~422 |
| `.hint` | 4 | 5 | ~771 |
| `.stage` | 5 | 6 | ~275 |
| `svg.cursor-trail` (new, Task 3) | — | 7 | (new) |

- [ ] **Step 4: Add reduced-motion override for nebula**

In the existing `@media (prefers-reduced-motion: reduce)` block (line 817), add:

```css
    .nebula-layer { animation: none !important; }
```

Insert it after `.shooting-star { display: none; }` (line 825).

- [ ] **Step 5: Verify — serve locally and check nebula renders**

Run: `python3 -m http.server 8000` from the project root, open `http://localhost:8000`.

Expected: Softly drifting color clouds visible behind the constellation stars. Six gradient blobs in teal, gold, purple, pink, blue, and warm orange slowly shift position on independent cycles.

---

### Task 2: Vivid Nebula Clouds — HTML

**Files:**
- Modify: `index.html` (HTML body, after line 843)

- [ ] **Step 1: Add the nebula div after the bg-stars SVG**

Insert immediately after line 843 (`<svg class="bg-stars" ...>`):

```html
<div class="nebula">
  <div class="nebula-layer"></div>
  <div class="nebula-layer"></div>
  <div class="nebula-layer"></div>
</div>
```

- [ ] **Step 2: Verify — reload and confirm nebula visible**

Reload `http://localhost:8000`.

Expected: Vivid nebula clouds drifting behind the stars. The parallax effect should move the nebula at ~0.2x the mouse movement speed (slower than stars at 0.4x).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add vivid nebula cloud backdrop with animated gradient layers"
```

---

### Task 3: Comet Cursor Trail — CSS

**Files:**
- Modify: `index.html` (CSS block)

- [ ] **Step 1: Add cursor trail CSS after the nebula CSS**

Append after the nebula CSS (added in Task 1):

```css
  /* ============ CURSOR TRAIL ============ */
  .cursor-trail {
    position: fixed;
    inset: 0;
    z-index: 7;
    pointer-events: none;
  }

  .trail-particle {
    fill: var(--star-warm);
    animation: trailFade 500ms ease-out forwards;
  }

  @keyframes trailFade {
    0%   { opacity: 0.6; transform: scale(1) translateY(0); }
    100% { opacity: 0;   transform: scale(0.3) translateY(8px); }
  }
```

- [ ] **Step 2: Add reduced-motion override for cursor trail**

In the `@media (prefers-reduced-motion: reduce)` block, add:

```css
    .cursor-trail { display: none; }
```

---

### Task 4: Comet Cursor Trail — HTML + JS

**Files:**
- Modify: `index.html` (HTML body + JS block)

- [ ] **Step 1: Add the cursor-trail SVG element to the body**

Insert after the nebula div (added in Task 2), before `<div class="stage">`:

```html
<svg class="cursor-trail" id="cursorTrail" preserveAspectRatio="xMidYMid slice"></svg>
```

- [ ] **Step 2: Add the setupCursorTrail JS function**

Insert after the `setupParallax()` function (after line ~1103):

```javascript
/* ---- Comet cursor trail: stardust particles follow the mouse ---- */
function setupCursorTrail() {
  if (reducedMotion || 'ontouchstart' in window) return;
  const trailSvg = document.getElementById('cursorTrail');
  let lastEmit = 0;

  function resizeTrailSvg() {
    trailSvg.setAttribute('viewBox', `0 0 ${window.innerWidth} ${window.innerHeight}`);
  }
  resizeTrailSvg();
  window.addEventListener('resize', resizeTrailSvg);

  document.addEventListener('mousemove', e => {
    const now = Date.now();
    if (now - lastEmit < 40) return;
    lastEmit = now;
    if (document.hidden) return;

    const c = document.createElementNS(SVG_NS, 'circle');
    c.setAttribute('cx', e.clientX);
    c.setAttribute('cy', e.clientY);
    c.setAttribute('r', 1.5);
    c.setAttribute('class', 'trail-particle');
    trailSvg.appendChild(c);
    c.addEventListener('animationend', () => c.remove());
  });
}
```

- [ ] **Step 3: Wire setupCursorTrail into the boot sequence**

In the `.then()` boot block (after `startShootingStars();` at line ~1280), add:

```javascript
    setupCursorTrail();
```

- [ ] **Step 4: Verify — move mouse across the constellation**

Reload `http://localhost:8000`.

Expected: Small golden particles appear at the cursor position as you move the mouse, fading and shrinking over ~500ms with a slight downward drift. No particles on touch devices or with reduced motion.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add comet cursor trail with fading stardust particles"
```

---

### Task 5: Season Orbit Rings — CSS

**Files:**
- Modify: `index.html` (CSS block)

- [ ] **Step 1: Add season ring CSS after the cursor trail CSS**

```css
  /* ============ SEASON ORBIT RINGS ============ */
  .season-ring {
    fill: none;
    stroke: rgba(255, 247, 230, 0.04);
    stroke-width: 1;
    stroke-dasharray: 4 8;
    opacity: 0;
    animation: fadeRing 1s ease forwards;
    animation-delay: 2.5s;
  }

  .season-ring-label {
    font-family: 'JetBrains Mono', monospace;
    font-size: 8px;
    letter-spacing: 0.15em;
    fill: var(--ink-faint);
    opacity: 0.5;
    text-anchor: middle;
  }

  @keyframes fadeRing {
    to { opacity: 1; }
  }
```

---

### Task 6: Season Orbit Rings — JS

**Files:**
- Modify: `index.html` (JS `render()` function)

- [ ] **Step 1: Add season ring rendering inside render()**

In the `render()` function, after `svg.innerHTML = '';` (line ~955) and before the connections `linesGroup` creation (line ~958), add:

```javascript
  // season orbit rings
  const ringsGroup = document.createElementNS(SVG_NS, 'g');
  ringsGroup.setAttribute('class', 'season-rings');
  const seasonMap = {};
  placed.forEach(ep => {
    const s = ep.season;
    if (!seasonMap[s]) seasonMap[s] = [];
    const dx = ep.x - cx;
    const dy = ep.y - cy;
    seasonMap[s].push(Math.hypot(dx, dy));
  });
  Object.entries(seasonMap).forEach(([season, radii]) => {
    const avgRadius = radii.reduce((a, b) => a + b, 0) / radii.length;
    const ring = document.createElementNS(SVG_NS, 'circle');
    ring.setAttribute('cx', cx);
    ring.setAttribute('cy', cy);
    ring.setAttribute('r', avgRadius);
    ring.setAttribute('class', 'season-ring');
    ringsGroup.appendChild(ring);

    const label = document.createElementNS(SVG_NS, 'text');
    label.setAttribute('x', cx);
    label.setAttribute('y', cy - avgRadius - 8);
    label.setAttribute('class', 'season-ring-label');
    label.textContent = `S${season}`;
    ringsGroup.appendChild(label);
  });
  svg.appendChild(ringsGroup);
```

- [ ] **Step 2: Verify — reload and check rings**

Reload `http://localhost:8000`.

Expected: Faint dashed concentric rings appear after ~2.5s, one per season, with "S1" and "S2" labels at the top of each ring.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add season orbit rings grouping episodes by season"
```

---

### Task 7: Related Episodes Path — CSS

**Files:**
- Modify: `index.html` (CSS block)

- [ ] **Step 1: Add related-path and panel-next CSS**

Append after the season ring CSS:

```css
  /* ============ RELATED EPISODES PATH ============ */
  .star-group.related .star-halo {
    opacity: 0.25 !important;
    transform: scale(1.15) !important;
    animation: none;
  }

  .star-group.related .star-core {
    filter: drop-shadow(0 0 16px rgba(255, 247, 230, 0.9))
            drop-shadow(0 0 28px rgba(255, 217, 168, 0.5));
  }

  .star-group.related .star-label {
    fill: var(--ink);
  }

  .connection.path-glow {
    stroke: var(--line-active);
    stroke-width: 1.5;
    stroke-dasharray: 6 4;
    opacity: 1;
    animation: pathFlow 2s linear infinite;
  }

  @keyframes pathFlow {
    to { stroke-dashoffset: -20; }
  }

  /* ---- "Explore next" card in panel ---- */
  .panel-next {
    display: flex;
    align-items: center;
    gap: 12px;
    width: 100%;
    padding: 12px 14px;
    margin-top: 6px;
    background: rgba(14, 16, 48, 0.4);
    border: 1px solid var(--panel-edge);
    border-radius: 10px;
    color: var(--ink);
    cursor: pointer;
    font-family: 'Plus Jakarta Sans', system-ui, sans-serif;
    text-align: left;
    transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
  }

  .panel-next:hover {
    border-color: var(--star-warm);
    background: rgba(255, 217, 168, 0.08);
    transform: translateY(-2px);
    box-shadow: 0 6px 16px -8px rgba(255, 217, 168, 0.4);
  }

  .panel-next .guest-avatar {
    width: 36px;
    height: 36px;
    font-size: 15px;
    flex-shrink: 0;
    animation: none;
  }

  .panel-next .guest-name {
    font-size: 13px;
    font-weight: 600;
  }

  .panel-next .guest-role {
    font-size: 11px;
    color: var(--ink-soft);
  }
```

---

### Task 8: Related Episodes Path — JS

**Files:**
- Modify: `index.html` (JS: `openPanel()`, `closePanel()`, new helper)

- [ ] **Step 1: Add `findBestNextEpisode` helper function**

Insert after the `clearConnectionRipple()` function (after line ~1039):

```javascript
/* ---- Find the most-connected related episode for "Explore next" ---- */
function findBestNextEpisode(currentEp) {
  const currentThemes = new Set(currentEp.themes);
  let best = null;
  let bestShared = 0;
  let bestTotal = 0;

  DATA.episodes.forEach(ep => {
    if (ep.id === currentEp.id) return;
    const shared = ep.themes.filter(t => currentThemes.has(t)).length;
    if (shared === 0) return;
    const total = ep.themes.length;
    if (shared > bestShared || (shared === bestShared && total > bestTotal)) {
      best = ep;
      bestShared = shared;
      bestTotal = total;
    }
  });
  return best;
}
```

- [ ] **Step 2: Add `highlightRelatedEpisodes` function**

Insert after `findBestNextEpisode`:

```javascript
/* ---- Highlight stars and lines related to active episode ---- */
function highlightRelatedEpisodes(epId) {
  if (activeTheme) return; // theme filter takes precedence
  const ep = DATA.episodes.find(e => e.id === epId);
  if (!ep) return;
  const epThemes = new Set(ep.themes);

  const relatedIds = new Set();
  DATA.episodes.forEach(other => {
    if (other.id === epId) return;
    if (other.themes.some(t => epThemes.has(t))) relatedIds.add(other.id);
  });

  document.querySelectorAll('.star-group').forEach(g => {
    const id = g.dataset.id;
    if (id === epId) return; // active star already styled
    if (relatedIds.has(id)) g.classList.add('related');
    else g.classList.add('dimmed');
  });

  document.querySelectorAll('.connection').forEach(line => {
    const from = line.dataset.from;
    const to = line.dataset.to;
    const touchesActive = from === epId || to === epId;
    const touchesRelated = relatedIds.has(from) || relatedIds.has(to);
    if (touchesActive && touchesRelated) {
      line.classList.add('path-glow');
      line.classList.remove('dimmed');
    } else {
      line.classList.add('dimmed');
      line.classList.remove('path-glow');
    }
  });
}
```

- [ ] **Step 3: Add `clearRelatedHighlights` function**

Insert after `highlightRelatedEpisodes`:

```javascript
function clearRelatedHighlights() {
  document.querySelectorAll('.star-group').forEach(g => {
    g.classList.remove('related', 'dimmed');
  });
  document.querySelectorAll('.connection').forEach(line => {
    line.classList.remove('path-glow', 'dimmed');
  });
}
```

- [ ] **Step 4: Modify `openPanel()` to add "Explore next" section and highlight related**

In `openPanel()`, find the `content.innerHTML` template string (lines ~1140–1160). Replace the closing part:

Find:
```javascript
    <div class="panel-section-title">Listen</div>
    <div class="panel-links">${links}</div>
  `;
```

Replace with:
```javascript
    <div class="panel-section-title">Listen</div>
    <div class="panel-links">${links}</div>
    ${(() => {
      const next = findBestNextEpisode(ep);
      if (!next) return '';
      return `
        <div class="panel-section-title">Explore next</div>
        <button class="panel-next" data-next-id="${next.id}">
          <div class="guest-avatar">${escapeHtml(next.guest.initials || '·')}</div>
          <div>
            <div class="guest-name">${escapeHtml(next.title)}</div>
            <div class="guest-role">${escapeHtml(next.guest.name)}</div>
          </div>
        </button>
      `;
    })()}
  `;
```

- [ ] **Step 5: Add click handler for "Explore next" button and trigger highlights**

In `openPanel()`, after the line `panel.classList.add('open');` (line ~1165), add:

```javascript

  // highlight related episodes on the constellation
  clearRelatedHighlights();
  highlightRelatedEpisodes(epId);

  // wire up "explore next" button
  const nextBtn = content.querySelector('.panel-next');
  if (nextBtn) {
    nextBtn.addEventListener('click', () => openPanel(nextBtn.dataset.nextId));
  }
```

- [ ] **Step 6: Modify `closePanel()` to clear related highlights**

Find `closePanel()` (line ~1168). Replace it:

Find:
```javascript
function closePanel() {
  document.getElementById('panel').classList.remove('open');
  document.querySelectorAll('.star-group').forEach(g => g.classList.remove('active'));
  activeEpisode = null;
}
```

Replace with:
```javascript
function closePanel() {
  document.getElementById('panel').classList.remove('open');
  document.querySelectorAll('.star-group').forEach(g => g.classList.remove('active'));
  clearRelatedHighlights();
  activeEpisode = null;
}
```

- [ ] **Step 7: Verify — click a star and check related path**

Reload `http://localhost:8000`. Click any star.

Expected: 
- The panel shows an "Explore next" card at the bottom with the most-related episode
- Stars sharing themes with the active episode glow brighter (`.related`)
- Connection lines between active and related stars pulse with a traveling light
- Unrelated stars are dimmed
- Clicking the "Explore next" card opens that episode's panel and updates the path
- Closing the panel clears all highlights

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: add related episodes path highlighting and explore-next suggestion"
```

---

### Task 9: Search / Guest Spotlight — CSS

**Files:**
- Modify: `index.html` (CSS block)

- [ ] **Step 1: Add search and spotlight CSS**

Append after the related-path CSS:

```css
  /* ============ SEARCH / GUEST SPOTLIGHT ============ */
  .search {
    position: fixed;
    bottom: 120px;
    left: 40px;
    z-index: 15;
    opacity: 0;
    animation: fadeUp 0.9s cubic-bezier(0.22, 1, 0.36, 1) 1.8s forwards;
  }

  .search-input {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.08em;
    color: var(--ink);
    background: rgba(14, 16, 48, 0.6);
    border: 1px solid var(--panel-edge);
    border-radius: 100px;
    padding: 9px 16px;
    width: 220px;
    outline: none;
    backdrop-filter: blur(6px);
    -webkit-backdrop-filter: blur(6px);
    transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
  }

  .search-input::placeholder {
    color: var(--ink-faint);
  }

  .search-input:focus {
    width: 280px;
    border-color: var(--star-warm);
    box-shadow: 0 0 16px rgba(235, 179, 34, 0.15);
  }

  .search-results {
    position: absolute;
    bottom: calc(100% + 6px);
    left: 0;
    width: 300px;
    background: var(--panel-bg);
    border: 1px solid var(--panel-edge);
    border-radius: 12px;
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    overflow: hidden;
    display: none;
  }

  .search-results.visible {
    display: block;
  }

  .search-result {
    display: flex;
    align-items: center;
    gap: 10px;
    width: 100%;
    padding: 10px 14px;
    background: transparent;
    border: none;
    border-bottom: 1px solid rgba(235, 179, 34, 0.08);
    color: var(--ink);
    cursor: pointer;
    font-family: 'Plus Jakarta Sans', system-ui, sans-serif;
    text-align: left;
    transition: background 0.2s ease;
  }

  .search-result:last-child { border-bottom: none; }

  .search-result:hover {
    background: rgba(255, 217, 168, 0.08);
  }

  .search-result .guest-avatar {
    width: 28px;
    height: 28px;
    font-size: 11px;
    flex-shrink: 0;
    animation: none;
  }

  .search-result-title {
    font-size: 12px;
    font-weight: 600;
  }

  .search-result-guest {
    font-size: 11px;
    color: var(--ink-soft);
  }

  /* Spotlight sonar ping */
  .star-group.spotlight .star-halo {
    animation: spotlightPing 1.5s ease-out forwards !important;
  }

  .star-group.spotlight .star-core {
    transform: scale(1.6);
    transition: transform 0.3s ease;
  }

  @keyframes spotlightPing {
    0%   { opacity: 0.15; transform: scale(1); }
    40%  { opacity: 0.5;  transform: scale(2.5); }
    100% { opacity: 0.04; transform: scale(1); }
  }
```

- [ ] **Step 2: Add mobile responsive overrides for search**

In the existing `@media (max-width: 768px)` block (line ~798), add:

```css
    .search {
      bottom: auto;
      top: 140px;
      left: 16px;
      right: 16px;
    }
    .search-input { width: 100%; }
    .search-input:focus { width: 100%; }
    .search-results {
      width: 100%;
      bottom: auto;
      top: calc(100% + 6px);
    }
```

---

### Task 10: Search / Guest Spotlight — HTML + JS

**Files:**
- Modify: `index.html` (HTML body + JS block)

- [ ] **Step 1: Add the search HTML element**

Insert after the `<div class="stats" ...>` element (after line ~856), before the panel `<aside>`:

```html
<div class="search">
  <input class="search-input" id="searchInput" type="text" placeholder="Find episode or guest..." aria-label="Search episodes">
  <div class="search-results" id="searchResults" role="listbox"></div>
</div>
```

- [ ] **Step 2: Add the setupSearch JS function**

Insert after the `setupCursorTrail()` function (added in Task 4):

```javascript
/* ---- Search / Guest Spotlight ---- */
function setupSearch() {
  const input = document.getElementById('searchInput');
  const results = document.getElementById('searchResults');

  input.addEventListener('input', () => {
    const q = input.value.trim().toLowerCase();
    if (q.length < 2) {
      results.classList.remove('visible');
      results.innerHTML = '';
      return;
    }

    const matches = DATA.episodes.filter(ep =>
      ep.title.toLowerCase().includes(q) ||
      ep.guest.name.toLowerCase().includes(q)
    ).slice(0, 5);

    if (matches.length === 0) {
      results.classList.remove('visible');
      results.innerHTML = '';
      return;
    }

    results.innerHTML = matches.map(ep => `
      <button class="search-result" role="option" data-id="${ep.id}">
        <div class="guest-avatar">${escapeHtml(ep.guest.initials || '·')}</div>
        <div>
          <div class="search-result-title">${escapeHtml(ep.title)}</div>
          <div class="search-result-guest">${escapeHtml(ep.guest.name)}</div>
        </div>
      </button>
    `).join('');
    results.classList.add('visible');

    results.querySelectorAll('.search-result').forEach(btn => {
      btn.addEventListener('click', () => {
        const epId = btn.dataset.id;
        input.value = '';
        results.classList.remove('visible');
        results.innerHTML = '';
        spotlightEpisode(epId);
      });
    });
  });

  input.addEventListener('blur', () => {
    // Delay so click on result registers before hide
    setTimeout(() => {
      results.classList.remove('visible');
    }, 200);
  });

  // Escape dismisses search dropdown
  input.addEventListener('keydown', e => {
    if (e.key === 'Escape') {
      input.value = '';
      input.blur();
      results.classList.remove('visible');
      results.innerHTML = '';
    }
  });
}

function spotlightEpisode(epId) {
  // Close existing panel and clear filters
  closePanel();
  if (activeTheme) setThemeFilter(activeTheme); // toggles off

  // Add spotlight class
  const starGroup = document.querySelector(`.star-group[data-id="${epId}"]`);
  if (starGroup) {
    starGroup.classList.add('spotlight');
    setTimeout(() => starGroup.classList.remove('spotlight'), 1500);
  }

  // Open panel after a short delay so spotlight lands first
  setTimeout(() => openPanel(epId), 400);
}
```

- [ ] **Step 3: Wire setupSearch into the boot sequence**

In the `.then()` boot block, after `setupCursorTrail();`, add:

```javascript
    setupSearch();
```

- [ ] **Step 4: Verify — type a guest name in the search**

Reload `http://localhost:8000`. Click the search input, type "Dave".

Expected:
- A dropdown appears with "Building Australian AI from Scratch" / "Dave Lemphers"
- Clicking it triggers a gold sonar-ping animation on the corresponding star
- After 400ms, the episode panel slides open
- The search input clears and dropdown hides

- [ ] **Step 5: Verify edge cases**

Test:
- Type fewer than 2 characters — no dropdown
- Type a string matching no episodes — no dropdown
- Press Escape — dropdown hides, input clears
- On mobile viewport (resize to <768px) — search appears at top

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add search with guest spotlight animation"
```

---

### Task 11: Final Integration Check

**Files:**
- Modify: `index.html` (no code changes expected — verification only)

- [ ] **Step 1: Full feature walkthrough**

Reload `http://localhost:8000` and verify all 5 features work together:

1. **Nebula:** Vivid color clouds drift behind the constellation
2. **Cursor trail:** Golden particles follow the mouse
3. **Season rings:** Faint dashed circles with S1/S2 labels
4. **Click a star:** Panel opens, related stars glow, connection lines pulse, "Explore next" card appears
5. **Click "Explore next":** New panel opens, path updates
6. **Close panel:** All highlights clear
7. **Search:** Type a name, select result, spotlight pings, panel opens
8. **Theme filter:** Click a theme chip — path highlighting defers to the filter
9. **Reduced motion:** Enable `prefers-reduced-motion` in browser dev tools — nebula frozen, cursor trail hidden, shooting stars hidden

- [ ] **Step 2: Check interaction conflicts**

Test these sequences:
- Search → spotlight → while panel open, click a different star → related path updates
- Theme filter active → click star → no related path (theme filter takes precedence) → close panel → theme filter still active
- Resize window → rings, nebula, cursor trail all adapt

- [ ] **Step 3: Final commit**

If any integration fixes were needed:

```bash
git add index.html
git commit -m "fix: integration polish for visual upgrade features"
```

---

Plan complete and saved to `docs/superpowers/plans/2026-04-28-constellation-visual-upgrade.md`. Two execution options:

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints

Which approach?
