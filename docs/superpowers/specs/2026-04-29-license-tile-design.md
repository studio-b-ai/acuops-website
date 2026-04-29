# AcuOps License Status Tile — Design Spec

**Date:** 2026-04-29  
**PRs:** 5a (functionality) + 5b (Comic Book DNA visual pass)  
**Repos:** studio-b-ai/acuops-website, studio-b-ai/studiob (CORS fix)

---

## Context

Phase 1 of the AcuOps License Gate is live — Heritage Fabrics key `ak_live_974175107494cc7cad6b`
validates successfully. VARs need a self-service surface to check their license status before
automated reminders fire. This is a minimal read-only tile, not a management portal.

---

## Architecture

**Static-site-friendly.** `acuops-website` is plain HTML. The tile is vanilla JS `fetch()` —
no framework, no build step. Ships as a new `license.html` page.

**Separate from `index.html`.** The `/validate` endpoint hardcodes `renew_url: "https://acuops.com/license"`.
The tile must live at that URL. Using a `license.html` page avoids the circular-link problem: the
renew CTA on the tile links to `mailto:sales@acuops.com`, not back to the same page.

**CORS fix (companion PR on studiob).** The `studiob-api` `/api/v1/acuops/licenses/validate` endpoint
currently returns no `Access-Control-Allow-Origin` header. Browser `fetch()` from `acuops.com` will be
blocked. Fix: add Hono built-in `cors()` middleware scoped to the license route, permitting
`https://acuops.com` and `https://acuops.com/*`.

---

## Design Decisions

| Question | Decision | Rationale |
|----------|----------|-----------|
| URL shape | `license.html` at `acuops.com/license` | Matches `RENEW_URL` in API; avoids circular link |
| Key entry | Input box + `?key=` query param pre-fill + `localStorage` persistence | VARs bookmark their status page |
| Grace display | Expiry date ("Expires 2026-05-15") | Clearer than countdown for multi-client VARs |
| Renew CTA | `mailto:sales@acuops.com` | Phase 1 is manual billing — no Stripe yet |
| Instance count | `1 / 3` with mini progress bar | Surfaces tier capacity before it's a problem |

---

## States and Copy

### Empty (no key)
> Paste your AcuOps license key to check status.  
> [input] `ak_live_...`  
> [Check Status →]

### Active
```
● ACTIVE — Heritage Fabrics
Tier: legacy-founder
Instances: 1 / 3 [▓░░]
Expires: —
```

### Grace
```
⚠ GRACE PERIOD — Heritage Fabrics
License expires 2026-05-15 — 7 days remaining.
Instances: 1 / 3
[Renew → sales@acuops.com]
```

### Expired / Revoked
```
✗ EXPIRED — License has expired. Renew to restore access.
[Contact sales@acuops.com]
```

### Unknown (404)
```
? KEY NOT FOUND
This key is not in the AcuOps system.
Email sales@acuops.com to purchase.
```

### Rate limited (429)
```
Rate limit reached. Try again in N seconds.
```

---

## PR 5a — Functionality

Files changed:
1. `license.html` — new page, all tile logic self-contained  
2. `index.html` — add `<a href="/license">License status</a>` to nav (one line)

`license.html` inherits the Mission Control design tokens (same Google Fonts import, same CSS vars
from inline `<style>`). Plain styling — legible, on-register, not polished.

OG tags required:
```html
<meta property="og:title" content="AcuOps License Status">
<meta property="og:description" content="Check your AcuOps license key status.">
<meta property="og:url" content="https://acuops.com/license">
<meta property="og:type" content="website">
<meta name="twitter:card" content="summary">
```

Verification:
- Empty state renders
- Heritage key (`ak_live_974175107494cc7cad6b`) → ACTIVE with correct tier/count
- Unknown key (`ak_test_invalid_404`) → KEY NOT FOUND
- `?key=ak_live_974175107494cc7cad6b` pre-fills and auto-submits

---

## PR 5b — Comic Book DNA Visual Pass

Applied to `license.html` on top of 5a's structure. Follows existing Mission Control register:
- Half-tone dot pattern in status header (SVG background, same density as existing dispatch strip)
- Caution-stripe corners on the status card (CSS `repeating-linear-gradient`, `--signal` + dark)
- Hand-lettered accent via Caveat font on status label ("ACTIVE", "GRACE PERIOD", etc.)
- Asymmetric panel: status left-column + instance bar right-column at desktop width
- CRT grain overlay (already on `body::before` — inherits automatically from shared CSS)

Does NOT change `license.html` HTML structure — only CSS additions in the `<style>` block.

---

## CORS Fix (studiob PR 5a.5)

File: `packages/api/src/app.ts`

Add before route mounts:
```ts
import { cors } from 'hono/cors';
// ...
app.use('/api/v1/acuops/licenses/*', cors({
  origin: ['https://acuops.com', 'https://www.acuops.com'],
  allowMethods: ['POST', 'OPTIONS'],
  allowHeaders: ['Content-Type'],
}));
```

This is the minimal surface — only the license endpoint, only acuops.com. No wildcard origins.

---

## Testing

1. `curl -X POST ... -H "Origin: https://acuops.com"` → response includes `Access-Control-Allow-Origin: https://acuops.com`
2. Open `license.html` via Playwright, enter Heritage key, assert active state
3. Enter `ak_test_invalid_404`, assert unknown state
4. Load `?key=ak_live_974175107494cc7cad6b`, assert auto-submit
5. Desktop screenshot (1280px) + mobile screenshot (375px) for 5b PR description
