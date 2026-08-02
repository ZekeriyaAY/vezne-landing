---
name: update-landing-screenshots
description: "Refresh the product screenshots on the vezne-landing homepage (src/pages/index.astro \"See it in action\" section) by spinning up vezne-api + vezne-web locally, seeding a demo account, and recapturing Dashboard/Rules/Import with agent-browser. Use whenever vezne-web's UI changes enough that the landing page's screenshots look stale."
---

# update-landing-screenshots

Regenerates the three product screenshots shown on the `vezne-landing`
homepage under "See it in action"
(`src/pages/index.astro` → `src/assets/screenshots/{dashboard,rules,import}.png`).
Run this after a `vezne-web` UI change makes the current screenshots look
out of date. This skill only ever writes inside `vezne-landing`
(`src/assets/screenshots/*.png`, and `src/pages/index.astro` only if the
page structure genuinely needs to change) — it must never modify
`vezne-api` or `vezne-web` source.

Assumes the standard workspace layout: `vezne-landing`, `vezne-api`, and
`vezne-web` as sibling directories under the same parent, per the
workspace-root `AGENTS.md`.

## 1. Start the two apps

```bash
cd ../vezne-api
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
curl -s http://localhost:8000/health   # expect {"status":"healthy",...}
```

`vezne-api/.env` should already have `JWT_SECRET` etc. set from a
previous run (see `.env.example` if not). This reuses the existing
`vezne-api_pgdata` Docker volume, so a demo account created by a
previous run of this skill (and its seeded data) normally survives —
try logging in before registering a new one.

```bash
cd ../vezne-web
npm run dev -- --port 5173 &
```

`vezne-web/.env` should already point `VITE_API_URL` at
`http://localhost:8000` with CORS on the api side allowing
`http://localhost:5173` (dev compose default). Wait for
`http://localhost:5173/` to respond before continuing.

## 2. Get a demo account with realistic data

Use `agent-browser` (see the `agent-browser` skill for tool basics — load
it with `agent-browser skills get core` if unfamiliar).

1. Open `http://localhost:5173/login` and try signing in with
   `demo@vezne.app`. If you don't have the password handy and login
   fails, just register it fresh (`/register`) — **pick any throwaway
   password at registration time and don't write it down anywhere
   that gets committed**; this is a disposable local dev database, not
   real data, and `vezne-landing` is a public repo. Never hardcode a
   password in this skill file or in any committed file.
2. Once on the Dashboard, seed data:
   - Settings → **Create default categories**
   - Settings → **Create default tags**
   - Settings → **Generate sample data (90 days)**
     (these three buttons also appear as a one-time "Get started in
     seconds" panel on the Dashboard for a brand-new account)
3. The seed data does **not** create Rules — add 2-3 illustrative ones
   by hand via Rules → New rule, e.g.:
   - Name "Netflix", contains "NETFLIX" → category *Streaming Services*
   - Name "Coffee shops", contains "STARBUCKS" → category *Coffee & Snacks*
   - Name "Salary", contains "SALARY" → any income category
   Pick categories that exist after step 2; the category picker is a
   searchable combobox grouped by parent category.

## 3. Known rendering quirks to work around

- **Dashboard charts only render correctly on a fresh page load.**
  Clicking a date-range preset (This year / Last 6 months / etc.)
  after the initial mount reliably leaves the chart panels blank
  (reproduced repeatedly — looks like a chart-library remount bug in
  `vezne-web`, not an agent-browser issue). If you need a specific
  range, navigate directly to `/dashboard`, **reload once**, and only
  then click a preset — if it still comes up blank, reload again and
  just use the default "This month" range instead of chasing it.
- **`agent-browser screenshot --full` does not capture past one
  viewport height on these app pages** — the app shell likely scrolls
  an inner container rather than `<body>`, so full-page stitching
  doesn't extend. Work around it by setting a generous fixed viewport
  height instead and screenshotting normally (not `--full`):
  ```bash
  agent-browser set viewport 1440 1500   # tall enough for the whole Dashboard
  ```
  Check the resulting image isn't clipped; adjust the height and retake
  if it is.
- **Hide the TanStack Query devtools icon before every screenshot** —
  it's a dev-only floating button that must not appear in marketing
  screenshots:
  ```bash
  agent-browser eval "document.querySelectorAll('.tsqd-parent-container').forEach(el => el.style.display='none'); 'hidden'"
  ```
- The native file-input "Choose File" button on `/import` renders in
  whatever language the OS is set to (e.g. Turkish "Dosya Seç" on this
  machine) — that's a real (tracked) product quirk, not a screenshot
  artifact. See `vezne-web/PLAN.md` §1.2. Leave it as-is unless that's
  been fixed since.

## 4. Capture each screenshot

For each target page, navigate, wait for network idle + a short extra
wait for chart animations, hide devtools, then screenshot at a tight
viewport height (avoids a lot of empty black space below the content —
check the actual content height in a first pass and adjust):

```bash
agent-browser open http://localhost:5173/dashboard
agent-browser set viewport 1440 1500
agent-browser wait --load networkidle
agent-browser wait 1500
agent-browser eval "document.querySelectorAll('.tsqd-parent-container').forEach(el => el.style.display='none'); 'hidden'"
agent-browser screenshot /tmp/dashboard-raw.png

agent-browser open http://localhost:5173/rules
agent-browser set viewport 1440 340
agent-browser wait --load networkidle
agent-browser eval "document.querySelectorAll('.tsqd-parent-container').forEach(el => el.style.display='none'); 'hidden'"
agent-browser screenshot /tmp/rules-raw.png

agent-browser open http://localhost:5173/import
agent-browser set viewport 1440 480
agent-browser wait --load networkidle
agent-browser eval "document.querySelectorAll('.tsqd-parent-container').forEach(el => el.style.display='none'); 'hidden'"
agent-browser screenshot /tmp/import-raw.png
```

Optionally downscale to ~1200px wide with `sips` before handing off —
not required (Astro re-encodes to WebP on build regardless, see step
5), but keeps the PNGs checked into git smaller:

```bash
sips -s formatOptions best --resampleWidth 1200 /tmp/dashboard-raw.png --out /tmp/dashboard.png
```

## 5. Drop the images into vezne-landing

`src/pages/index.astro` imports these three files by fixed path via
`astro:assets`, so a same-name overwrite needs no code change:

```bash
cp /tmp/dashboard.png ../vezne-landing/src/assets/screenshots/dashboard.png
cp /tmp/rules.png     ../vezne-landing/src/assets/screenshots/rules.png
cp /tmp/import.png    ../vezne-landing/src/assets/screenshots/import.png
```

If a page's layout changed enough that the old crop/viewport no longer
makes sense (new sidebar item, taller card, etc.), it's fine to also
touch the `previews` array or `.preview-frame` CSS in
`src/pages/index.astro` — just keep the rest of the page's structure
and copy intact.

## 6. Verify and hand off

```bash
cd ../vezne-landing
npm run build          # confirms Astro can still process/optimize the images
npx astro preview --port 4323 &
agent-browser open http://localhost:4323/
agent-browser screenshot --full /tmp/landing-check.png
```

Look at `/tmp/landing-check.png` yourself before calling it done —
check the new screenshots actually reflect the UI change that
prompted this refresh, and that nothing looks clipped or misaligned.

Per this repo's normal workflow: don't commit unless the user asks;
when they do, a short commit message like "Refresh landing
screenshots" is enough (see recent git log for tone).

## 7. Clean up

```bash
pkill -f "vite --port 5173"
cd ../vezne-api && docker compose -f docker-compose.yml -f docker-compose.dev.yml down
```

(`down` without `-v` — keep the `pgdata` volume so the demo account and
seed data are still there next time this skill runs.)
