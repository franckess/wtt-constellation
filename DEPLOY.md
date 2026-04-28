# Deploying Constellation as a Substack nav tab

This guide gives you a permanent **"Constellation"** tab in your Substack navigation bar — exactly the way Lenny's Newsletter has a "Lennybot" tab linking out to `lennybot.com`. Click the tab → readers land on your interactive constellation page.

**The setup**: you host the constellation as a free public webpage (GitHub Pages), then add a Substack nav tab pointing to it. Both Substack and GitHub Pages free tiers cover this entirely. Adding new episodes is one file edit; the live page updates within a minute.

---

## How Lenny's Newsletter does it (the pattern)

Lenny's Substack nav bar shows: `Home · Newsletter · Podcast · Investing · Lennybot · Contact me`. Each one is just a Substack nav-bar entry. "Lennybot" is configured as an external URL pointing at `lennybot.com` — a completely separate page hosted elsewhere. From the reader's perspective, it feels like a native Substack tab. From a technical perspective, it's a regular hyperlink.

That's what we're building for you, except instead of a chatbot it's the constellation, and instead of a custom domain (you can add that later, free) it'll initially live at a `github.io` URL.

---

## What's in this folder

| File                  | Purpose                                                          |
| --------------------- | ---------------------------------------------------------------- |
| `index.html`          | The interactive constellation page. Goes on GitHub Pages.        |
| `episodes.json`       | All episode data. **The only file you'll edit going forward.**   |
| `og-preview.png`      | 1200×630 social card image. Shown when the URL is shared.        |
| `substack-hero.jpg`   | Inline image you can also drop into individual posts.            |
| `DEPLOY.md`           | This file.                                                       |

---

## Part 1 — Host the constellation page (~15 min, one time)

### 1. Create a GitHub repository

1. Sign in at [github.com](https://github.com).
2. Top-right `+` → **New repository**.
3. Name it `wtt-constellation` (this becomes part of your URL).
4. Set **Public** (free Pages requires public).
5. Don't tick "Add a README". Click **Create repository**.

### 2. Upload the deploy files

1. On the empty repo page, click **uploading an existing file** (the link mid-page).
2. Drag in all four files: `index.html`, `episodes.json`, `og-preview.png`, `substack-hero.jpg`.
3. Scroll, click **Commit changes**.

### 3. Turn on GitHub Pages

1. In the repo: **Settings** (top tab) → **Pages** (left sidebar).
2. **Source**: Deploy from a branch.
3. **Branch**: `main` and `/ (root)`. Click **Save**.
4. Wait ~1 minute. GitHub will display a green box with your URL — like:
   ```
   https://YOUR-USERNAME.github.io/wtt-constellation/
   ```
5. Open it. The constellation should render with all 11 episodes. Save this URL — you'll need it for Part 2.

---

## Part 2 — Add the Constellation tab to your Substack nav (~2 min)

This is the bit that makes it feel like a native part of your publication.

1. Sign into your Substack publication's dashboard.
2. Click **Settings** (bottom-left of the dashboard).
3. In the left sidebar, click **Website**.
4. Scroll to the **Navigation bar** section.
5. Click **Add item**.
6. Fill in:
   - **Title**: `Constellation` (or `Episode Map`, `Universe`, `The Map` — whatever fits your voice)
   - **URL**: paste your GitHub Pages URL from Part 1
7. Click **Add**.
8. (Optional) Drag the new item to your preferred position in the nav order — it usually looks best between "Archive" and "About".

That's it. Refresh your publication's homepage and you'll see the new tab. Click it and your readers land on the interactive constellation.

### A note on tab placement

Three placement patterns work well, in order of how prominent the tab is:

- **First (most prominent)** — `Constellation · Home · Archive · About`
- **Middle (the Lenny pattern)** — `Home · Episodes · Constellation · About`
- **Last (subtle)** — `Home · Archive · About · Constellation`

If your podcast is the centerpiece of your Substack, middle placement is usually the sweet spot — it gives the tab a "supporting feature" feel rather than a "main destination" feel.

---

## Part 3 — Adding new episodes (the only ongoing task)

When a new episode goes live, you edit one file in the GitHub web UI. No HTML edits, no rebuild, no local tools.

1. Go to your GitHub repo, click `episodes.json`.
2. Click the pencil icon (top-right of the file view) to edit in browser.
3. Find the `"episodes"` array. Copy the last episode block, paste a new one at the bottom, and fill in:
   - `id`: e.g. `"s02e04"`
   - `season`, `number`, `title`, `releaseDate` (YYYY-MM-DD)
   - `guest`: `name`, `role`, `company`, two-letter `initials` for the avatar
   - `themes`: pick from the IDs in the `"themes"` array at the top of the file
   - `keyInsights`: 3–5 short paraphrases, in your own voice
   - `standoutMoment`: one memorable quote, exchange, or idea
   - `links`: Spotify / YouTube / Apple URLs once you have them
4. Scroll to the bottom of the page, write a commit message like `"Add S2E4: [Guest Name]"`, click **Commit changes**.
5. Within ~1 minute, the live page picks up the new episode. The constellation re-layouts itself automatically — no positioning by hand.

If you want a fresh social-card preview after adding an episode, message me and I'll regenerate `og-preview.png` and `substack-hero.jpg` for you in seconds.

---

## Part 4 — Use the constellation as a hero image too (optional)

The nav tab is the main path, but `substack-hero.jpg` (84KB) is also handy to drop at the top of any related post — for example, a season recap, a "year in review", or a guest-introduction post. Substack supports JPEG/PNG drag-and-drop in the editor.

When the post is read on Substack with the URL pasted (or shared on Twitter/LinkedIn), the OG preview image kicks in automatically — readers see the constellation thumbnail with the title and tagline.

---

## Part 5 — Polishing (optional but worth it)

### Custom domain (free)

The `github.io` URL works fine but feels less branded than something like `constellation.whatthetech.au`. If you have a domain:

1. In your DNS provider (Squarespace, GoDaddy, Cloudflare, etc.), add a CNAME record from `constellation` (or whatever subdomain) pointing to `YOUR-USERNAME.github.io`.
2. In your GitHub repo: **Settings → Pages → Custom domain**, enter `constellation.whatthetech.au`, save.
3. Tick **Enforce HTTPS** once GitHub finishes its check (~10 minutes).
4. Update the URL in your Substack nav tab to use the custom domain.

### Wire in your platform pills

The top-right Spotify / YouTube / Apple buttons currently show as muted placeholders. To make them live:

1. Open `episodes.json` on GitHub, click the pencil to edit.
2. At the top, find:
   ```json
   "platforms": { "spotify": "", "youtube": "", "apple": "" },
   "substackUrl": ""
   ```
3. Paste your show URLs into the platform empty strings, and your Substack URL (e.g. `https://whatthetech.substack.com`) into `substackUrl`.
4. Commit. The pills become tappable and brighten on hover. A subtle `← Substack` link will also appear, giving readers a clear path back to your publication when they arrive via the nav tab.

You can fill in per-episode `links` the same way for the panel buttons inside each episode card.

### Add a new theme

In `episodes.json`, add an entry to the `"themes"` array:

```json
{ "id": "your-theme-id", "name": "Display Name", "color": "#HEXCODE" }
```

Then reference the `id` from any episode's `themes` array. Pick colors moderately saturated, medium-bright (jewel tones, not pastels), so they read well against deep navy.

---

## What it costs

**$0/month indefinitely.** GitHub free tier covers Pages with no traffic limits at podcast scale. Substack's free tier covers everything we need. A custom domain is optional and only costs whatever you'd pay your domain registrar (~$15/year if you don't already own one).

---

## If something breaks

- **"Could not load episodes" message on the page** — `episodes.json` isn't valid JSON. Most likely a missing comma or bracket. Paste the file content into [jsonlint.com](https://jsonlint.com) to find the line.
- **Constellation looks empty** — check the `"episodes"` array isn't empty, and that the JSON parses cleanly.
- **Substack nav tab doesn't appear** — refresh the page; nav changes can take ~30 seconds to propagate. Mobile app may take longer than the web view.
- **Stars overlap or look cramped** — unlikely under 30 episodes. If it ever happens, reduce the `0.11` multiplier in `index.html`'s `render()` function (search for `Math.min(w, h) * 0.11`).
- **Social preview shows the wrong image after an update** — some platforms cache OG images aggressively. Use [opengraph.xyz](https://www.opengraph.xyz) to force a re-fetch, or [LinkedIn's post inspector](https://www.linkedin.com/post-inspector/).

---

## Why this setup vs alternatives

- **Why not embed the live page directly inside a Substack post?** Substack's editor only allows their approved embed list (YouTube, Spotify, Datawrapper). Custom HTML and arbitrary iframes aren't supported, so a separate page linked from the nav is the cleanest pattern.
- **Why GitHub Pages and not Netlify Drop?** Both work, but GitHub gives you a commit history (great for tracking when each episode was added), and the in-browser editor means you never need local tools.
- **Why JSON-driven instead of one bundled HTML file?** Editing one focused data file in the browser is far less error-prone than editing inside a 1,500-line HTML file. Same `episodes.json` could later drive social cards, a public API, or a full site.
