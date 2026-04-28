# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Interactive constellation visualization for the "What The Tech Podcast (AU)". Podcast episodes are rendered as stars on a golden-spiral layout, connected by shared theme lines. Built as a zero-dependency, static site for GitHub Pages, embedded as a Substack nav tab.

## Architecture

**Single-page monolith by design** — all HTML, CSS, and JS live in `index.html` (~1300 lines). This is intentional: zero build steps, editable via GitHub's web UI, instant GitHub Pages deploys.

- `index.html` — the entire application (styles, SVG rendering engine, interaction logic)
- `episodes.json` — all podcast data (episodes, themes, platform URLs); the only file edited for content updates
- `og-preview.png` / `substack-hero.jpg` — social preview images

### Rendering Pipeline (in `index.html` `<script>` block)

1. **Boot**: `loadData()` fetches `episodes.json` with cache-bust `?v=${Date.now()}`
2. **Layout**: `computePositions()` places stars on a Fibonacci golden spiral — no manual coordinates
3. **Render**: `render()` builds SVG stars + connection lines; `renderBgStars()` adds ambient field; `renderPlatforms()`, `renderLegend()`, `renderStats()` fill UI chrome
4. **Interaction**: `openPanel(epId)` slides in episode detail drawer; `setThemeFilter(themeId)` dims non-matching stars; `highlightConnections(epId)` ripples lines on hover
5. **Effects**: parallax on mouse, shooting stars every 8-22s, click burst particles

### Key Design Decisions

- **Golden spiral layout** auto-scales with episode count — never manually position stars
- **Theme connections**: episodes sharing a `themes[]` ID get an SVG line between them
- **CSS variables** on `:root` control the color scheme (deep navy + gold brand palette)
- **Reduced motion**: `prefers-reduced-motion` check disables all animations
- **XSS protection**: `escapeHtml()` used on all user-facing data from JSON

## Development

No build tools, package manager, or test runner. To work on this locally:

```bash
# Serve locally (any static server works)
python3 -m http.server 8000
# Then open http://localhost:8000
```

## Data Model (`episodes.json`)

```
podcast.platforms  — top-level Spotify/YouTube/Apple show URLs (empty string = muted placeholder)
podcast.substackUrl — enables "← Substack" back-link when set
themes[]           — { id, name, color } — referenced by episodes via id
episodes[]         — { id (s01e03), season, number, title, releaseDate, guest, themes[], keyInsights[], standoutMoment, links }
```

Episode IDs follow `s{season:02d}e{number:02d}` format. Theme IDs are kebab-case. Theme colors should be jewel tones (moderately saturated, medium-bright) to read against `--bg-deep: #020a1a`.

## Conventions

- **CSS classes**: `.star-*` (constellation elements), `.panel-*` (detail drawer), `.theme-chip`, `.platform-pill`
- **Data attributes**: `data-id`, `data-from`, `data-to`, `data-themes`, `data-empty` for JS hooks
- **SVG elements**: created with proper `http://www.w3.org/2000/svg` namespace
- **Fonts**: Instrument Serif (display), Plus Jakarta Sans (body), JetBrains Mono (labels) via Google Fonts CDN
- **Responsive**: single breakpoint at `768px` — panel becomes bottom sheet on mobile

## Troubleshooting

- **"Could not load episodes"** — invalid JSON in `episodes.json`; validate at jsonlint.com
- **Stars overlap/cramped** — reduce `0.11` multiplier in `computePositions` call (`Math.min(w, h) * 0.11`)
- **Shooting stars / parallax not working** — check `prefers-reduced-motion` isn't set
