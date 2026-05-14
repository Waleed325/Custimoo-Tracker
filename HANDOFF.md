# Handoff — what's done and what's left for you

## What I did (local, no live systems touched)

1. **Added JSON API to `WebApp.gs`** (additive, non-breaking).
   - Renamed the existing HTML-rendering `doGet(e)` to `_doGetHtml(e)`.
   - Added a new `doGet(e)` that dispatches `?api=lookup|route|evidence|active|closed|remark|insights` to JSON and falls through to `_doGetHtml(e)` when no `api` parameter is set.
   - The current deployment behavior (HTML output, iframed dashboard, public tracker) is preserved exactly.
   - File: `C:\Users\WaleedFarooq\Documents\Claude\Projects\Tracking Sheet\WebApp.gs`

2. **Built a static site** at `C:\Users\WaleedFarooq\Documents\Claude\Projects\custimoo-tracker\`.
   - `index.html` — Track.html refactored: static title, `fetch` instead of `google.script.run`, PWA `<link rel="manifest">` + service worker registration.
   - `admin.html` — Master.html refactored: static title, `fetch`-based `apiCall` chainable mirroring the old `google.script.run` API so internal call sites are unchanged.
   - `manifest.json`, `sw.js`, `icon.svg`, `README.md`.

## What you need to do — in order

### Step 1 — Generate PNG icons from `icon.svg` (~2 min)

GitHub Pages and Android PWAs need real PNGs. Easiest path:

- Open https://realfavicongenerator.net or https://maskable.app/editor in your browser
- Upload `icon.svg` → export 192x192, 512x512, and a 512x512 maskable PNG
- Save them next to `icon.svg` as `icon-192.png`, `icon-512.png`, `icon-512-maskable.png`

(Skip this for the first dry run if you want — the SVG works on iOS, just not on older Android.)

### Step 2 — Redeploy Apps Script as a new version (~3 min)

The new `?api=...` JSON endpoint only exists once you push and redeploy.

```powershell
cd "C:\Users\WaleedFarooq\Documents\Claude\Projects\Tracking Sheet"
clasp push
```

Then in https://script.google.com → open the project → **Deploy → Manage Deployments → ✏️ edit the existing one → New version → Deploy**. The URL stays the same.

**Smoke test the API** in your browser:

```
https://script.google.com/macros/s/AKfycbwx9_BB153l0U57v2gKvWG2xe72B5eUxNIbU1lUPCgQJE-xK-7m-fBDDtgWMj2Gkb2Xew/exec?api=lookup&tn=<a-real-tracking-number>
```

You should see JSON, not HTML. If you see the dashboard HTML, the new version isn't deployed yet.

### Step 3 — Test locally (~2 min)

```powershell
cd "C:\Users\WaleedFarooq\Documents\Claude\Projects\custimoo-tracker"
npx serve .
```

Open `http://localhost:3000/index.html` — search a real tracking number; should work end-to-end.
Open `http://localhost:3000/admin.html` — dashboard should load rows.

### Step 4 — Push to GitHub + enable Pages (~5 min)

```powershell
cd "C:\Users\WaleedFarooq\Documents\Claude\Projects\custimoo-tracker"
gh repo create custimoo-tracker --public --source=. --remote=origin --push
gh repo edit --enable-pages --pages-branch=main --pages-path=/
```

(Or via the GitHub web UI: New repo → push → Settings → Pages → Source = main branch / root.)

Wait ~1 min, then open `https://<your-github-username>.github.io/custimoo-tracker/`.

### Step 5 — Install on phone (~2 min)

- **iPhone**: open the URL in Safari → tap Share → Add to Home Screen → tap the new "Custimoo" icon → opens full-screen.
- **Android**: open in Chrome → see "Install app" prompt (or menu → Install app) → tap icon from app drawer.

### Step 6 — Custom domain (optional, ~15 min + DNS propagation)

If you want `track.custimoo.com` instead of the `*.github.io` URL:

1. In your DNS provider, add a CNAME record: `track` → `<your-github-username>.github.io`
2. In the repo, create a file named `CNAME` (no extension) containing `track.custimoo.com`
3. In Settings → Pages, enter `track.custimoo.com` as the custom domain and tick "Enforce HTTPS"

## Rollback

If anything goes wrong with Apps Script:

- In Apps Script editor → Deploy → Manage Deployments → click the gear → "Manage versions" → roll back to the pre-change version.
- The static site (GitHub Pages) is independent — just delete the repo or unpublish Pages.

## Acceptance tests

- [ ] `?api=lookup&tn=...` returns JSON in browser address bar
- [ ] `index.html` on laptop finds a real TN and shows full result + route intelligence
- [ ] `admin.html` on laptop loads rows, KPI cards reflect data, clicking a row expands journey
- [ ] Edit a remark on admin.html → reloads as saved
- [ ] iPhone Safari: "Add to Home Screen" → tapping icon opens full-screen with green theme bar
- [ ] Android Chrome: "Install app" → launches from app drawer
- [ ] Both phone and laptop URL show real-time data from the same Google Sheet
