# What The Tech Podcast (AU) — Season Constellation

An interactive, astronomy-themed visualization of podcast episodes displayed as a constellation map. Each episode is a star, and shared themes draw connection lines between them.

**Live page:** Hosted on GitHub Pages and embedded as a navigation tab in the [What The Tech](https://www.whatthetech.com.au/) Substack publication.

## Features

- **Golden spiral layout** — episodes are positioned automatically using a Fibonacci spiral; no manual coordinates needed
- **Theme connections** — episodes sharing topics are linked by animated lines
- **Episode detail panel** — click any star to see the guest, key insights, standout moment, and listen links
- **Theme filtering** — click a theme chip to highlight only episodes covering that topic
- **Nebula backdrop** — animated color clouds drifting behind the constellation
- **Cursor trail** — faint stardust particles follow the mouse
- **Season orbit rings** — concentric dashed circles grouping episodes by season
- **Related episodes path** — opening an episode highlights connected stars and suggests a "listen next" episode
- **Search** — find episodes or guests by name with a spotlight animation
- **Parallax & shooting stars** — ambient depth effects responding to mouse movement
- **Responsive** — full mobile layout with bottom-sheet panel
- **Reduced motion** — all animations respect `prefers-reduced-motion`

## How it works

Everything lives in two files:

| File | Purpose |
|------|---------|
| `index.html` | The entire application — HTML, CSS, and JS in a single file |
| `episodes.json` | All podcast data — episodes, themes, platform URLs |

No build tools, no dependencies, no package manager. Edit `episodes.json` on GitHub, commit, and the live page updates within a minute.

## Adding a new episode

1. Open `episodes.json` in the GitHub web editor
2. Copy the last episode block and fill in: `id`, `season`, `number`, `title`, `guest`, `themes`, `keyInsights`, `standoutMoment`, `links`
3. Commit — the constellation re-layouts automatically

Episode IDs follow `s{season}e{number}` format (e.g. `s02e04`). Theme IDs are kebab-case and must match an entry in the `themes` array.

## Adding a new theme

Add an entry to the `themes` array in `episodes.json`:

```json
{ "id": "your-theme-id", "name": "Display Name", "color": "#HEXCODE" }
```

Pick jewel-tone colors (moderately saturated, medium-bright) that read well against the deep navy background.

## Deployment

See [DEPLOY.md](DEPLOY.md) for the full setup guide — GitHub Pages hosting, Substack nav tab integration, custom domain, and troubleshooting.

## Listen

- [Spotify](https://open.spotify.com/show/5IFOM3h1s161b362Q79uLR)
- [YouTube](https://www.youtube.com/@WhatTheTechPodcastAU)
- [Apple Podcasts](https://podcasts.apple.com/us/podcast/what-the-tech-podcast-au/id1866901744)
