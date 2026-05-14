# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The **static frontend** for Custimoo's shipment tracker. Two pages, no build step:

- `index.html` — phone-facing PWA for customers (single tracking-number lookup)
- `admin.html` — laptop dashboard for the CSM team (KPIs, tables, insights)

The **backend lives outside this repo** — a separate Google Apps Script project at `C:\Users\WaleedFarooq\Documents\Claude\Projects\Tracking Sheet\`, deployed at `https://script.google.com/macros/s/AKfycbwx9_BB153l0U57v2gKvWG2xe72B5eUxNIbU1lUPCgQJE-xK-7m-fBDDtgWMj2Gkb2Xew/exec`. Read from / write to the `Raw Data` Google Sheet.

This frontend talks to that backend over plain HTTPS with `?api=...` query params — no iframe, no `google.script.run`. That split is the whole point of the repo: it lets the customer page be installed as a PWA, which the original iframed Apps Script web app could not be.

## Architecture in one diagram

```
Phone (PWA install) ────► waleed325.github.io/Custimoo-Tracker/         (index.html)
Laptop (team)       ────► waleed325.github.io/Custimoo-Tracker/admin.html
                              │
                              │ fetch(?api=lookup|route|evidence|active|closed|remark|insights)
                              ▼
                    Apps Script  ── doGet ── dispatches on params.api ──► JSON
                          │                  (else falls through to _doGetHtml)
                          ▼
                    Raw Data + Delivered Archive sheets
```

`WebApp.gs` keeps the original HTML-rendering `doGet` logic intact as `_doGetHtml`, so the old iframe deployment continues to work in parallel during cutover.

## API contract

All endpoints are `GET` on the Apps Script URL with `?api=<name>` + extras. Defined in `Tracking Sheet/WebApp.gs:doGet`.

| `?api=` | Maps to backend fn | Extra params | Called from |
|---|---|---|---|
| `lookup` | `lookupShipment(tn, internal)` | `tn`, `internal` (0/1) | index, admin |
| `route` | `getRouteIntelligence(tn)` | `tn` | index |
| `evidence` | `getComparableShipments(tn, internal)` | `tn`, `internal` | index |
| `active` | `getInProcessOrders()` or `refreshProcessCache()` if `bust=1` | `bust` | admin |
| `closed` | `getClosedOrders()` or `refreshClosedCache()` if `bust=1` | `bust` | admin |
| `remark` | `updateRemark(tn, v)` (write) | `tn`, `v` | admin |
| `insights` | `getInsights()` | — | admin |

Frontend wrappers:
- `apiGet(name, params, onOk, onErr)` — flat callback style in `index.html`
- `apiCall(name, params).withSuccessHandler(fn).withFailureHandler(fn).run()` — chainable, deliberately mirrors the old `google.script.run` shape so the call sites in `admin.html` stayed structurally identical during the refactor

When adding a new API endpoint, both ends change: add a `case` in `WebApp.gs:doGet`, then call it via `apiGet`/`apiCall` from the relevant HTML.

## Commands

```powershell
# Local preview (static server — service worker requires http://localhost or https://)
python -m http.server 8766
# then open http://localhost:8766/index.html or /admin.html

# Push frontend changes
git add . ; git commit -m "..." ; git push   # GitHub Pages auto-deploys main

# Push backend changes (separate project)
cd "C:\Users\WaleedFarooq\Documents\Claude\Projects\Tracking Sheet"
clasp push
# Then in https://script.google.com → Deploy → Manage Deployments → New version
# The deployment URL stays the same; a New Version is what makes ?api=... pick up edits.
```

No build, no lint, no test runner. The HTML files are hand-written, plain JS, no framework. Verification = open the local server, click around, check devtools Network for `?api=...` calls returning JSON.

## PWA specifics

- `manifest.json` registers icons and `display: standalone`. Theme color `#33DCAE`.
- `sw.js` caches the app shell (`/`, `index.html`, manifest, PNG icons). API calls (anything to `script.google.com` / `googleusercontent.com`) skip the cache so live tracking data is never stale. Bump `CACHE_VERSION` when shell HTML/CSS changes so installed PWAs pick up fresh assets.
- `icon-512-maskable.png` has 20% safe-zone padding — required so Android adaptive icon shapes don't clip the "C" mark.
- Service worker fetch handler explicitly bails out for the Apps Script domains; do not add API URLs to `SHELL`.

## Brand (do not invent alternative styling)

Colors from `Desktop\Custimoo brand theme.md` and `Desktop\CLAUDE.md`:
- Brand green primary `#33DCAE`, dark `#1FB089`
- Grey dark `#5b6166`, grey `#9CA1A3`, light grey `#C5C9CB`
- Text `#1a1a1a`, background `#ffffff`
- Font stack: `-apple-system, "Segoe UI", "Helvetica Neue", Arial, sans-serif`

The header "C" mark on both pages is an inline SVG (`viewBox="0 0 100 100"`, black `#0f172a` C with a green `#33DCAE` leaf accent). `icon.svg` mirrors this exactly so favicon, PWA icon, and in-page logo all match.

## Deployment URLs

- **Public phone tracker (customers):** https://waleed325.github.io/Custimoo-Tracker/
- **Master dashboard (team):** https://waleed325.github.io/Custimoo-Tracker/admin.html
- **Repo:** https://github.com/Waleed325/Custimoo-Tracker
- **Apps Script backend:** the AKfycbwx9_BB153l0... URL above (one deployment, two consumers — this repo over `?api=`, and the legacy iframed `script.google.com` URL via `_doGetHtml`).

## When making changes

- Editing an HTML page? The two pages don't share JS — each is self-contained. Don't extract shared code into a separate file without also updating `SHELL` in `sw.js`.
- Adding/renaming an icon? Update `manifest.json`, `<link>` tags in both HTML heads, and `SHELL` in `sw.js`, then bump `CACHE_VERSION`.
- Touching the backend? Edit `WebApp.gs` over in `Tracking Sheet/`, then `clasp push` + create a New Version deployment. Existing version stays live until you flip the active deployment, so rollback is one click.
- Don't reintroduce `google.script.run` — the whole point of this repo is the JSON API split.
