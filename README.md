# Custimoo Tracker — Static Frontend

Customer-facing shipment tracker (`index.html`) and team-facing master dashboard (`admin.html`), hosted as a static site so the customer tracker can be installed as a PWA on phones.

Backend stays in Google Apps Script (`Tracking Sheet` project) and is reached via plain HTTPS with `?api=...` JSON endpoints.

## Files

| File | Role |
|---|---|
| `index.html` | Phone-friendly customer tracker. PWA-enabled (manifest + service worker). |
| `admin.html` | Laptop master dashboard for the team. KPI cards, table, insights tab. |
| `manifest.json` | PWA manifest (name, icons, theme color `#33DCAE`). |
| `sw.js` | Service worker — caches the app shell, never caches Apps Script API calls. |
| `icon-source.png` | Original 512x512 brand logo provided by Waleed. |
| `icon-192.png`, `icon-512.png`, `icon-512-maskable.png` | PWA icons referenced by `manifest.json`. |
| `icon-180.png` | Apple touch icon (iOS home-screen size). |
| `favicon-32.png` | Browser tab favicon. |
| `icon.svg` | Vector version of the C-with-leaf mark (matches the inline header SVGs). |

## Backend endpoint

Both pages call:

```
https://script.google.com/macros/s/AKfycbwx9_BB153l0U57v2gKvWG2xe72B5eUxNIbU1lUPCgQJE-xK-7m-fBDDtgWMj2Gkb2Xew/exec
```

with one of:

| `?api=` | Maps to | Used by |
|---|---|---|
| `lookup&tn=<TN>&internal=0\|1` | `lookupShipment(tn, internal)` | index, admin |
| `route&tn=<TN>` | `getRouteIntelligence(tn)` | index |
| `evidence&tn=<TN>&internal=0\|1` | `getComparableShipments(tn, internal)` | index |
| `active&bust=0\|1` | `getInProcessOrders()` / `refreshProcessCache()` | admin |
| `closed&bust=0\|1` | `getClosedOrders()` / `refreshClosedCache()` | admin |
| `remark&tn=<TN>&v=<text>` | `updateRemark(tn, v)` | admin |
| `insights` | `getInsights()` | admin |

The dispatcher lives in `WebApp.gs` (`doGet`) — the existing HTML rendering is preserved at `_doGetHtml`, so the old iframe deployment keeps working in parallel during cutover.

## Local dev

```
npx serve .
# then open http://localhost:3000
```

Service workers need HTTPS or `localhost` — both work. Use a real phone on the same Wi-Fi or ngrok to test the PWA install flow.

## Branding

Colors follow `Custimoo brand theme.md` on the Desktop:
- Brand green `#33DCAE` (theme color, hero)
- Brand green dark `#1FB089` (text, borders)
- Grey dark `#5b6166`, grey `#9CA1A3`, light grey `#C5C9CB`
- White background, `-apple-system, "Segoe UI", ...` font stack
