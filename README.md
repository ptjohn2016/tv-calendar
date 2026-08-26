# 📺 Screen Time

A vibrant, iPhone-first TV calendar. One shared watchlist, Google sign-in, episode
countdowns, and a confetti celebration when a show you follow gets renewed.

Everything lives in a single `index.html` — no build step, no bundler, no backend.

---

## What it does

- **Week strip calendar** — swipe left/right (or ← →) between weeks. Each day pill shows
  coloured pips for what's airing.
- **This week** button — jumps straight back to the current week from anywhere.
- **Countdown panel** — a right-hand rail listing every tracked show by how many
  **days / weeks** until its next episode. On a phone it's a slide-in drawer behind the
  clock icon; on desktop it's always visible.
- **New-season celebration** — full-screen confetti when a tracked show gets renewed.
- **Shared list** — everyone signed in sees and edits the same watchlist.

Air times are converted from the network's timezone into **your** local clock, so an
American show that airs Tuesday 10pm ET appears on the right day for you.

---

## Where the data comes from

**[TVmaze](https://www.tvmaze.com/api)** — free, no API key, no signup, CORS-enabled,
which is exactly what a static page on GitHub Pages needs.

| Endpoint | Used for |
|---|---|
| `/search/shows?q=` | the Add-show search |
| `/shows/:id/episodes` | every episode + air date → fills the calendar |

Episode lists are cached in `localStorage` for 12 hours and requests run three-at-a-time,
to stay well inside TVmaze's ~20 calls / 10 seconds limit.

### How the renewal celebration works

No TV API exposes a "renewed!" flag. So the app derives it: it remembers the highest
season number it has ever seen for each show (`maxSeason`, seeded the moment you add the
show so there's never a false alarm). When a later fetch turns up a **higher** season
number, that's a renewal — confetti fires, and the announcement is written to Firestore so
everyone on the list sees it. Each person celebrates once, on their own device.

### Not included

The calendar tells you **when** an episode airs and marks it as aired. It does not search,
link to, or scrape torrent indexers — sourcing stays on your side.

---

## Setup

### 1. Try it immediately

The app works out of the box in **local mode** — your list is saved in that browser only.
Because it uses ES modules, it must be served over HTTP; double-clicking the file won't work.
There's a dependency-free PowerShell server included:

```bash
powershell -ExecutionPolicy Bypass -File serve.ps1
```

Then open <http://localhost:8765/>. (If you have Python or Node instead,
`python -m http.server 8765` or `npx serve` do the same job.)

### 2. Firebase (Google login + shared list)

1. Create a project at <https://console.firebase.google.com>.
2. **Build → Authentication → Get started → Sign-in method → Google → Enable.**
3. **Build → Firestore Database → Create database → Production mode.**
4. **Project settings → Your apps → Web (`</>`)** → register the app → copy the config object.
5. Paste it into the `CONFIG.firebase` block near the top of the `<script>` in `index.html`.
6. Put your Google addresses in `CONFIG.allowlist`, e.g.
   ```js
   allowlist: ["you@gmail.com", "them@gmail.com"],
   ```
7. Copy `firestore.rules` into **Firestore → Rules**, put the same addresses in the list
   there, and **Publish**.

> The `apiKey` in `CONFIG` is public by design — every Firebase web app ships one. It is
> an identifier, not a secret. `firestore.rules` is what actually protects your data, which
> is why the allowlist has to be maintained in **both** places.

### 3. GitHub Pages

No command line needed:

1. <https://github.com/new> → name it `tv-calendar` → **Public** → **Create repository**.
2. **uploading an existing file** → drag `index.html` in (plus the other files if you
   want them) → **Commit changes**.
3. **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)` → Save.**
   First build takes 1–3 minutes.

Live at `https://YOU.github.io/tv-calendar/`.

> The file must be named `index.html` in lowercase. GitHub Pages runs on a
> case-sensitive filesystem and only serves lowercase `index.html` as a folder's
> default page — `INDEX.html` gives you a 404 at the bare URL.

Last step: back in Firebase, **Authentication → Settings → Authorised domains** → add
`YOU.github.io` (hostname only, no repo name, no `https://`), or Google sign-in will be
rejected on the live site with `auth/unauthorized-domain`.

### 4. Add to your iPhone home screen

Open the Pages URL in Safari → Share → **Add to Home Screen**. It launches full-screen
with no browser chrome, respects the notch and home indicator, and remembers your session.

---

## Keyboard shortcuts

| Key | Action |
|---|---|
| `←` `→` | Previous / next week |
| `T` | This week |
| `Esc` | Close panels |

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire app — markup, styles, logic |
| `firestore.rules` | Security rules to paste into the Firebase console |
| `serve.ps1` | Dependency-free local preview server (Windows) |
| `README.md` | This file |
