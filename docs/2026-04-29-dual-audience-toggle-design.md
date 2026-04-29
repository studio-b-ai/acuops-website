# Dual-Audience Toggle — Design Doc

**Date:** 2026-04-29  
**PR:** 11 (phase 1 design only — no HTML yet)  
**Status:** Awaiting Kevin sign-off  
**Scope:** Awareness pivot only. Same product, same SKU, same $300/mo pricing. Copy adjustment only — no feature or pricing differences between audiences.

---

## 1. Audience Naming

**Chosen labels: `Reseller` / `In-house`**

Toggle pill: `[ Reseller | In-house ]`

Alternates considered and rejected:

| Candidate | Reason rejected |
|-----------|----------------|
| VAR / Direct | "VAR" is accurate but "Direct" is ambiguous — sounds like a purchase channel, not a buyer type |
| VAR / End User | "End User" reads as the person at the keyboard, not the IT team running Acumatica |
| Channel / In-House | "Channel" is jargon for non-channel buyers |
| Reseller / Enterprise | "Enterprise" implies large company; this is about internal ops, not company size |
| Partner / Self-managed | Two words on one side is awkward in a pill |

**Why `Reseller / In-house`:**
- "Reseller" is plain. A VAR will immediately recognize it as themselves. A direct buyer will immediately recognize they are *not* it.
- "In-house" is plain ops language — the IT team that runs their own Acumatica. No jargon. No aspirational framing.
- Both fit in the pill at one word each. Clean toggle ergonomics.

Default on first visit: **Reseller** (canonical product positioning; most traffic from VAR channel).

---

## 2. Toggle Placement + UX

**Chosen placement: sticky nav bar, right of the nav links, left of the CTA button.**

Rationale: The audience choice is global — it affects every section. Anchoring it in the sticky nav means the pill stays visible as the visitor scrolls, so they can flip it mid-read without hunting for a control. Alternatives considered:

| Option | Verdict |
|--------|---------|
| Sticky nav (chosen) | Persistent, always accessible, reads as a global view switch |
| Below dispatch strip | Gets scrolled away quickly; visitor loses the control |
| Hero section only | Visitor can't re-toggle after scrolling past hero |
| Floating corner button | Obscures content; bad on mobile |
| Below KPI ticker | Too far from nav context; looks orphaned |

### Wireframe (desktop, 1440px)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ DISPATCH STRIP: ● STATUS ACTIVE · 12 PROBES · 146 POST-PUBLISH TESTS · 90s  │
├──────────────────────────────────────────────────────────────────────────────┤
│ <·> AcuOps   A Studio B product   Features  Pricing  FAQ  Bolt  License      │
│                                   ┌─────────────────┐  ┌─────────────────┐  │
│                                   │ Reseller In-house│  │  REQUEST DEMO → │  │
│                                   └─────────────────┘  └─────────────────┘  │
│ ──────────────────────── terminal-green hairline rule ──────────────────────  │
├──────────────────────────────────────────────────────────────────────────────┤
│ Hero content                                                                  │
```

**Toggle visual spec:**
- Pill container: `border: 1px solid var(--rule)`, `background: var(--surface)`, `border-radius: 0` (hard-edge, matches Mission Control register)
- Active segment: `background: var(--signal)` (terminal green `#10B981`), `color: var(--bg)` (dark slate)
- Inactive segment: `color: var(--text-secondary)`, transparent background
- Font: `Geist Mono`, `0.68rem`, `letter-spacing: 0.08em`, `text-transform: uppercase`
- Labels: `RESELLER` / `IN-HOUSE`
- Transition: 200ms ease, background slides with CSS transition on the active segment

### Wireframe (mobile, 390px)

On screens ≤640px the nav links are hidden (hamburger). The toggle moves to just below the nav bar, full-width, above the hero eyebrow. Same pill styling, slightly larger tap target (min-height 36px).

```
┌────────────────────────┐
│ DISPATCH STRIP         │
├────────────────────────┤
│ <·> AcuOps  ≡ (menu)  │
├────────────────────────┤
│ ┌──────────────────┐   │
│ │ RESELLER│IN-HOUSE│   │
│ └──────────────────┘   │
├────────────────────────┤
│ Hero                   │
```

---

## 3. State Persistence

Two mechanisms; no auto-detection from referrer:

1. **URL parameter:** `?audience=in-house` or `?audience=reseller`. On first paint, JS reads `URLSearchParams` before `DOMContentLoaded` and applies the correct audience immediately — no flash of the wrong copy. Kevin can link `acuops.com/?audience=in-house` in outbound emails to direct buyers and they land in the right view.

2. **localStorage:** Key `acuops_audience`, value `"reseller"` or `"in-house"`. Written on every toggle flip. Read on page load (after URL param check — URL param wins if both are present). A returning visitor lands on their last choice.

**Priority chain:**
```
URL param present? → use it (and write to localStorage)
URL param absent?  → localStorage present? → use it
Otherwise          → default: "reseller"
```

No referrer auto-detection. Not worth the maintenance surface for an awareness toggle.

---

## 4. Accessibility + Keyboard

Two `<button>` elements inside a `role="group"` container, using `aria-pressed`:

```html
<div class="audience-toggle" role="group" aria-label="Viewing as">
  <button class="aud-btn aud-btn--active" data-aud="reseller"
          aria-pressed="true">RESELLER</button>
  <button class="aud-btn" data-aud="in-house"
          aria-pressed="false">IN-HOUSE</button>
</div>
```

- **Tab:** focus moves to each button in order
- **Space / Enter on a button:** activates that audience; `aria-pressed` updates on both buttons; copy crossfades
- **Focus ring:** `outline: 2px solid var(--signal)` with `outline-offset: 2px` — terminal-green, visible on both dark nav and active green segment

---

## 5. Section-by-Section Copy Map

### Dispatch strip

**Current:** `● STATUS ACTIVE · 12 PROBES · 146 POST-PUBLISH TESTS · ~90s ALERT LATENCY · ACUMATICA 24.200.001`

**Differs between audiences?** No. Operational stats are identical. Unchanged.

---

### Hero eyebrow + h1 + margin annotation

**Current:**
- Eyebrow: `AcuOps · Acumatica DevOps Platform`
- H1: `Acumatica's a great ERP. The operations layer is an exercise left to the reader.`
- Annotation: `← this is the part where you eat the cost.`

**Differs?** No. "exercise left to the reader" is a programmer/ERP joke that works for both audiences. The margin annotation ("you eat the cost") works for both — a VAR eats it for the client, an in-house IT team eats it for the company. Unchanged.

---

### Hero body paragraph (`.hero-body`)

**Current (reseller):**
> When a minor release drifts a DAC and the customization breaks at 6pm Friday, it's not Acumatica's phone that rings — it's yours. AcuOps is the team you weren't hired to be. Twelve probes, 146 post-publish tests, a CI/CD pipeline through the Customization API. Read-only API, session-gated. Credentials stay in your vault, not ours.

**Differs?** Yes. "it's yours" and "the team you weren't hired to be" are VAR-framed (the VAR fields external client calls; the in-house IT team fields internal calls but wasn't hired to be a DevOps team).

**In-house variant — Option A (recommended):**
> When a minor release drifts a DAC and the customization breaks at 6pm Friday, it's not Acumatica's phone that rings — it's the operations floor's. AcuOps is the DevOps layer your ERP budget didn't include. Twelve probes, 146 post-publish tests, a CI/CD pipeline through the Customization API. Read-only API, session-gated. Credentials stay in your vault, not ours.

**In-house variant — Option B:**
> When a minor release drifts a DAC and the customization breaks at 6pm Friday, the floor calls IT. AcuOps is the ops layer your ERP implementation didn't leave time to build. Twelve probes, 146 post-publish tests, a CI/CD pipeline through the Customization API. Read-only API, session-gated. Credentials stay in your vault, not ours.

**Kevin's call on:** Option A vs B. I lean A — preserves the original sentence rhythm and "not Acumatica's phone that rings" setup lands cleanly.

**Resolved 2026-04-29 by Kevin:** Option A — `"it's not Acumatica's phone that rings — it's the operations floor's"`

---

### Hero panel (`.hero-panel` KPI readout)

**Current:**
```
Audience:  Acumatica VARs
Platform:  Monitor · CI/CD · Validate
Access:    Read-only API, session-gated
Meter:     $300 / mo · per instance · flat
```

**Differs?** One row only.

**In-house variant:**
```
Audience:  In-house Acumatica teams
Platform:  Monitor · CI/CD · Validate
Access:    Read-only API, session-gated
Meter:     $300 / mo · per instance · flat
```

---

### 01 / 07 — The problem

**Current (reseller):**
> **Title:** You've been stitching this together *per-client, for a decade.*
>
> **Body:** You've tried internal scripts. You've SSH'd into the instance when the client complains. Maybe a NetSuite-style monitoring add-on that doesn't know what a `PXException` is. None of them survive an Acumatica minor release.
>
> Infor CloudSuite ships built-in environment monitoring. Sage Intacct has runbook integrations. Microsoft Dynamics 365 Business Central has AppSource partners that watch tenants. Acumatica has VARs stitching it together.
>
> AcuOps is the thing that should have come in the box.

**Differs?** Yes. "per-client," "when the client complains," and "Acumatica has VARs stitching it together" are all VAR-framed.

**In-house variant:**
> **Title:** You've been stitching this together *for the team, for years.*
>
> **Body:** You've tried internal scripts. You've SSH'd into the instance when operations calls. Maybe a monitoring add-on that doesn't know what a `PXException` is. None of them survive an Acumatica minor release.
>
> Infor CloudSuite ships built-in environment monitoring. Sage Intacct has runbook integrations. Microsoft Dynamics 365 Business Central has AppSource partners that watch tenants. Acumatica leaves the ops layer to you.
>
> AcuOps is the thing that should have come in the box.

*Changes:* "per-client, for a decade" → "for the team, for years." "when the client complains" → "when operations calls." "Acumatica has VARs stitching it together" → "Acumatica leaves the ops layer to you." Last line unchanged.

---

### 02 / 07 — Capabilities

**Current intro (reseller):**
> Twelve health probes. 146 post-publish tests. A CI/CD pipeline that pushes customizations through the Customization API. All three read-only against your clients' instances.

**Differs?** One phrase.

**In-house variant (intro only):**
> Twelve health probes. 146 post-publish tests. A CI/CD pipeline that pushes customizations through the Customization API. All three read-only against your instance.

Feature cards: unchanged (describe technical capabilities, not audience).

---

### 03 / 07 — How it works

**Current (reseller):**
> **Title:** Point AcuOps at the instance. Get Slack alerts *before your client calls.*
>
> **Step 01:** Point AcuOps at your client's Acumatica instance. Read-only API access, session-gated. Credentials live in your vault — we don't hold keys, we don't hold sessions between runs.
>
> **Step 02:** Twelve health probes run continuously. The CI/CD pipeline deploys customizations when you push to GitHub. 146 tests validate every publish against the live screen. *(unchanged)*
>
> **Step 03:** Slack notifications fire before your client does. You see every alert across every client. Your client sees only their own instance. Drill into dashboards for the stack trace.

**Differs?** Title, Step 01, Step 03.

**In-house variant:**
> **Title:** Point AcuOps at the instance. Get Slack alerts *before the floor calls.*
>
> **Step 01:** Point AcuOps at your Acumatica instance. Read-only API access, session-gated. Credentials live in your vault — we don't hold keys, we don't hold sessions between runs.
>
> **Step 03:** Slack notifications fire before operations does. You see every alert across every probe. Drill into dashboards for the stack trace.

---

### 04 / 07 — Slack integration

This is the most VAR-specific section. The two-workspace story (client workspace + operator workspace) collapses for in-house.

**Current (reseller):**
> **Title:** Your workspace. Your alerts. Your clients' workspaces *stay separate.*
>
> **Intro:** AcuOps installs directly into your client's Slack workspace. They see their own alerts. You see everyone's. One event, two audiences, zero manual forwarding.
>
> **Left panel — label:** Your client sees | **h3:** Their own workspace.
> **Body:** AcuOps drops into your client's existing Slack. No shared channels. No cross-workspace noise. Two dedicated channels, scoped to their instance: `#acuops-alerts` (Deploys, health warnings, sync errors) · `#acuops-intake` (Requests that route to HubSpot tickets). **Footnote:** Each client's data stays in their workspace. They never see your other clients or your internal channels.
>
> **Right panel — label:** You see | **h3:** All clients, one view.
> **Body:** Every notification also posts to your operator workspace. All clients, all alerts, one place: `#deployments` · `#system-health` · `#integration-issues`. **Footnote:** One event, two audiences. Zero manual forwarding.
>
> **Setup steps:** 01. We send your client a one-click install link during onboarding. 02. They approve AcuOps in their Slack workspace. Ten seconds, no meeting required. 03. Channels are created automatically. Alerts start flowing immediately.

**In-house variant:**
> **Title:** Your workspace. Your alerts. *Routed to the team that owns the fix.*
>
> **Intro:** AcuOps installs into your Slack workspace. Two dedicated channels: one for alerts, one for intake. Alerts route automatically to the right team.
>
> **Left panel — label:** Operations sees | **h3:** Their channels.
> **Body:** Two dedicated channels in your existing workspace: `#acuops-alerts` (Deploys, health warnings, schema drift) · `#acuops-intake` (Requests that route to tickets). **Footnote:** No new workspace to manage. Alerts go to the right channel automatically.
>
> **Right panel — label:** IT sees | **h3:** Everything, one view.
> **Body:** Every alert also posts to your ops channel. All probes, all deploys, one place: `#deployments` (Deploy results + rollback signals) · `#system-health` (Health probe alerts) · `#integration-issues` (Sync and integration failures). **Footnote:** One alert, right channel, right team.
>
> **Setup steps:** 01. Connect AcuOps to your Slack workspace during onboarding. 02. Approve the bot. Ten seconds, no meeting required. 03. Channels are created automatically. Alerts start flowing immediately.

**Kevin's call on:** "Operations sees / IT sees" panel labels — do those match how in-house Acumatica teams are actually organized? If the two intra-company audiences are better named differently, adjust.

**Resolved 2026-04-29 by Kevin:** `Operations sees / IT sees` — labels approved as written.

---

### 05 / 07 — Pricing

**Current:**
> **Title:** $300 per month. Per Acumatica instance. *Flat.*
> **Intro:** Not per seat. Not per transaction. Not per alert. The meter is the instance — one number, one invoice. Every capability included. Volume discounts at 10+ and 25+ instances.
> **Volume line:** Volume · 15% off at 10+ instances · 25% off at 25+

**Differs?** Volume tier line only. Pricing card, feature list, and total are identical.

**In-house variant (intro only):**
> **Intro:** Not per seat. Not per transaction. Not per alert. The meter is the instance — one number, one invoice. Every capability included.

Volume tier line hidden on in-house view (an in-house team managing 25 Acumatica instances is extremely rare; showing it implies a portfolio they don't have).

---

### 06 / 07 — Why this, not that

This section is fully VAR-framed. All four decisions need rewrites.

**Current (reseller):**
> **Intro:** We've priced this product three different ways. This is the one that survived first contact with our own client portfolio.
>
> **Decision 01 — Per instance, not per seat:** Your client's AP clerk and your VAR's senior architect are both running drift checks against the same DAC. Per-seat billing doesn't reflect the work. Instance-count does. One Acumatica tenant = $300/mo, regardless of who logs in or how many clerks open SO301000.
>
> **Decision 02 — Flat, not metered:** We thought about $0.10 per test run. We thought about a tier-bump at 50 deploys per month. Then we ran the numbers on our own portfolio. The cost-to-serve variance between a quiet month and a wild one is close to nothing. A flat meter means you know the invoice before the month starts.
>
> **Decision 03 — $300 is what the work costs:** Twelve probes running every few minutes. A 146-test Playwright suite against every publish. A CI/CD pipeline that handles Import, publishBegin, co-publish, publishEnd polling, and rollback signals. An on-call rotation that catches the indexer when it times out. Priced at what it takes to actually run the thing — which is the only price that survives a year.
>
> **Decision 04 — Resell it, or don't:** Some VARs bill it straight through at cost. Some mark it up to $500 and make it a managed-service line item. We don't care. The meter is your Acumatica instance count — not seats, not clients, not end-customer invoices. What you do with the margin is yours.

**In-house variant:**
> **Intro:** We've priced this product three different ways. This is the one that survived first contact with our own production environment.
>
> **Decision 01 — Per instance, not per seat:** Your AP clerk and your IT admin are both covered. Per-seat billing would make you think twice about who gets access to the monitoring tools. Instance-count doesn't — one Acumatica tenant = $300/mo, regardless of who logs in or how many people open SO301000.
>
> **Decision 02 — Flat, not metered:** We thought about $0.10 per test run. We thought about a tier-bump at 50 deploys per month. Then we ran the numbers. The cost-to-serve variance between a quiet month and a wild one is close to nothing. A flat meter means you know the line item before the quarter starts.
>
> **Decision 03 — $300 vs. the alternative:** The alternative is a DevOps engineer who also knows Acumatica well enough to write Playwright tests against the Customization API. That person exists, they cost $140k/year, and they have other things to do. Twelve probes, 146 tests, a CI/CD pipeline, and an on-call rotation that catches the indexer when it times out — at $300/mo. The math isn't close.
>
> **Decision 04 — No IT headcount required:** This isn't a tool that needs someone to watch a dashboard. It watches itself and posts to Slack when something needs a human. Your IT team handles the fix — AcuOps handles the detection, the deploy pipeline, and the post-publish validation. The ops layer runs whether or not someone has a monitoring window open.

---

### 07 / 07 — FAQ

**Current (reseller) — 6 questions:**
1. Do you compete with my implementation practice?
2. Why not bill my hourly rate and handle it myself?
3. What happens on an Acumatica minor release?
4. Do you need access to my clients' environments?
5. Can I resell this to my clients?
6. Do I need to be a Studio B client to buy AcuOps?

**Shared questions (keep for both audiences, minor reword on Q4):**
- Q3: What happens on an Acumatica minor release? → unchanged
- Q4: Do you need access to *my clients'* environments? → "Do you need access to our Acumatica environment?" (same answer, one-word change)
- Q6: Do I need to be a Studio B client to buy AcuOps? → unchanged

**VAR-only (hide on in-house):**
- Q1: "Do you compete with my implementation practice?" — irrelevant for in-house
- Q5: "Can I resell this to my clients?" — irrelevant for in-house

**Replace with in-house-specific:**

> **Q: What if our Acumatica partner already handles this?**
> A (softer, recommended): AcuOps supplements rather than replaces — it gives your team visibility into the same environment your partner manages, without waiting on a project to be open.
>
> A (sharper, alternate): If they do, great — ask them what's in the test suite and when the last probe ran. Most implementation partners don't run continuous monitoring between projects. AcuOps runs 24/7 regardless of whether a project is active.

> **Q: Why not handle this ourselves?**
> A: You can. A 146-test Playwright suite against the Customization API takes a few weeks to build and someone to maintain. At $300/mo, AcuOps costs less than two hours of a mid-level developer per month. The suite runs whether or not your team has capacity.

**In-house FAQ order:**
1. What if our Acumatica partner already handles this? *(new)*
2. Why not handle this ourselves? *(new)*
3. What happens on an Acumatica minor release? *(shared)*
4. Do you need access to our Acumatica environment? *(shared, minor reword)*
5. Do I need to be a Studio B client to buy AcuOps? *(shared)*

**Kevin's call on:** Softer vs. sharper answer for Q1. I recommend softer — the in-house buyer may have a good relationship with their partner.

**Resolved 2026-04-29 by Kevin:** Softer variant — "AcuOps supplements rather than replaces — gives your team visibility without waiting on a project to be open."

---

### Demo CTA

**Current:**
> **Title:** See what your clients' environments *look like right now.*
> **Intro:** 30-minute demo. We point AcuOps at a sandbox of your choosing and walk through what the probes, pipeline, and test suite catch. No slide deck.

**In-house variant:**
> **Title:** See what your environment *looks like right now.*
> **Intro:** 30-minute demo. We point AcuOps at a sandbox of your choosing and walk through what the probes, pipeline, and test suite catch. No slide deck.

CI metrics tile (right column): unchanged — same live data, same blinded `[client-1]` instance name, no copy change.

---

### Footer

**Current:** `AcuOps — software we run on our own portfolio, billed per instance, not per seat.`

**Differs?** No. "our own portfolio" is Studio B's, not the visitor's. Works for both. Unchanged.

---

## 6. Implementation Approach

**Recommended: paired-span pattern.**

`<body>` carries a class: `aud-reseller` (default) or `aud-in-house`. CSS hides the non-active variant:

```css
.aud-reseller .aud-ih  { display: none; }
.aud-in-house  .aud-res { display: none; }
```

Differing blocks ship in both versions:

```html
<p class="hero-body aud-res">
  …it's not Acumatica's phone that rings — it's yours…
</p>
<p class="hero-body aud-ih">
  …it's not Acumatica's phone that rings — it's the operations floor's…
</p>
```

**Pros of paired-span:**
- No FOUC — correct copy is in DOM on first paint. JS sets `body` class from URL param in a synchronous `<script>` in `<head>`, before any CSS paint.
- Readable diff — PR review sees both versions side by side.
- Search engines index both. (Minor benefit; SEO targets the default/reseller view.)
- Graceful degradation — if JS fails, reseller copy renders (correct default).
- No hydration step. No JS framework dependency.

**Alternate considered: data-attribute + JS hydration.** Content in JS objects, injected on toggle. Rejected: FOUC risk, harder to review, search engines miss the alternate.

**Affected block count:** ~12 paired spans across 8 sections. Estimated page HTML growth: ~180 lines.

---

## 7. Edge Cases

**Print styles:** Print whichever audience is currently active. `body.aud-reseller / body.aud-in-house` is already set; `@media print` inherits it. No special handling.

**SEO / OG tags:** Stay as-is — default (reseller) view. `?audience=in-house` is a UI state, not a separate page. No `hreflang`, no alternate canonical. The existing `og:description` ("AcuOps watches the Acumatica instances you run for clients") stays; it reflects canonical positioning.

**CI metrics tile + dispatch strip:** Live data from `/api/ci-metrics`. Identical for both audiences. No branching.

**No-FOUC guarantee:** A `<script>` in `<head>` (before stylesheets paint) reads URL param and sets `document.body.className`. Falls back to `aud-reseller` if param is absent and localStorage is empty.

**`?audience=` param cleanup:** After reading the param and applying it, use `history.replaceState` to strip it from the URL bar (keeps the URL clean; state is now in localStorage). Kevin can still share the link — the param is read before replaceState fires.

---

## 8. Open Questions for Kevin — ALL RESOLVED 2026-04-29

1. **Hero body — Option A vs B** (§5, hero body). I recommend A.
   **Resolved 2026-04-29 by Kevin:** Option A — `"it's not Acumatica's phone that rings — it's the operations floor's"`

2. **Slack panel roles — "Operations sees / IT sees"** — do those labels match how in-house Acumatica teams are organized at a company like Heritage Fabrics? If the actual split is different (e.g., Warehouse / Finance, or Operations / Accounting), rename.
   **Resolved 2026-04-29 by Kevin:** `Operations sees / IT sees` approved as written.

3. **FAQ partner-answer tone** — softer ("supplements rather than replaces") vs. sharper ("ask them what's in the test suite"). I recommend softer.
   **Resolved 2026-04-29 by Kevin:** Softer variant — "AcuOps supplements rather than replaces — gives your team visibility without waiting on a project to be open."

4. **Volume tier on pricing** — hide on in-house view? I recommend yes.
   **Resolved 2026-04-29 by Kevin:** Hidden on in-house view using `data-audience-only="reseller"` paired-span pattern.

5. **Toggle label capitalization** — `RESELLER / IN-HOUSE` (uppercase, Geist Mono) or `Reseller / In-house` (title case)? Uppercase matches the Mission Control register. Confirm.
   **Resolved 2026-04-29 by Kevin:** `RESELLER / IN-HOUSE` — uppercase, Geist Mono, Mission Control register.

---

## Phase 2 Estimate

- **Additive HTML:** ~180 lines (paired copy spans for ~12 blocks)
- **CSS:** ~60 lines (toggle styles + `.aud-res` / `.aud-ih` visibility rules)
- **JS:** ~50 lines (toggle init, URL param, localStorage, `aria-pressed` management)
- **Total additive:** ~290 LOC in `index.html`. No new files. No new dependencies.
- **Playwright screenshots:** 16 at 1440px desktop (8 sections × 2 audiences) + 8 at 390px mobile = 24 total.
- **Build time estimate:** 2–3 hours implementation + verification.
