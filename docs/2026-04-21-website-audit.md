# acuops.com — Website Audit

**Date:** 2026-04-21
**Author:** Kevin Bibelhausen (via audit session w/ Claude)
**Status:** Approved — implementation scheduled as PR 5b in the AcuOps License Gate arc ([docs/plans/2026-04-21-acuops-license-gate-architecture.md](https://github.com/studio-b-ai/studiob/blob/main/docs/plans/2026-04-21-acuops-license-gate-architecture.md) on the `studiob` repo)
**Site state audited:** https://acuops.com as-of 2026-04-21 (Mission Control reskin live per PR [#2](https://github.com/studio-b-ai/acuops-website/pull/2) + Comic Book DNA pass 2026-04-20)

This is the durable record of the acuops.com website audit. The fresh-session agent implementing PR 5b (visual uplift) should read this in full before touching code. PR 5a (license-boundary copy + license-status tile) is scoped separately and lands first.

## Buyer we're designing for

**The one buyer:** Acumatica VAR principal.

- 5–15 years in the channel
- 20–60 customers on their book
- Lives in: Acumatica's own UI all day, Slack DMs with customers, GitHub/ADO for their team's Customization repos, the Acumatica partner portal
- Skeptical of vendor claims by default. Can smell a Figma mockup from across the room. Has sat through too many "AI-powered ERP DevOps" pitches at Summit.

### Primary anxieties that MUST be disarmed

1. **"My customer's prod breaks after a publish and it's Thursday afternoon."** Core AcuOps job. Silent alerts are the real disqualifier.
2. **"Rival VAR down the road offers DevOps + poaches my best account."** Competitive pressure is the purchase driver, not the feature list.
3. **"24.1 → 24.2 becomes a six-week hotfix parade."** Upgrade-cycle pain. Anything pre-flighting the upgrade is disproportionately valuable.
4. **"Another vendor shows up with a $300/mo line item on my customer's invoice."** Disqualifying. Must be baked into the VAR's margin, invisible to the end customer.
5. **"I don't have a pager-duty culture. My junior consultant isn't on-call at 2 AM."** AcuOps must not impose on-call discipline the VAR doesn't already run.

### Budget unlock

**$300/mo out of a $3–8k/mo implementation engagement is invisible noise.** The real sell is "baked into your margin, customer never sees it." Price framing must lean on this, not on ROI math.

### Trust signal ladder (descending importance)

- Seeing a real CI run with a real commit SHA from a real Acumatica customization repo (blinded as `[client-1]`)
- A named 30-day receipt (real counts of caught failures, real upgrades pre-flighted, real zero-outage quarters)
- Pricing disclosed with no "call us"
- Named competitors — or the deliberate absence where there are none (AcuOps competes against "nothing" in most VAR minds, which is its own trust signal)
- Blinded named references (partner VARs running AcuOps in production)

## What's working — do not touch

- **Hero line.** *"Acumatica's a great ERP. The operations layer is an exercise left to the reader."* This is the correct Basecamp-style anti-hype frame. It states an opinion, names the gap, and doesn't oversell. Do not rewrite for cleverness.
- **First FAQ answer.** *"Do you compete with my implementation practice? — No. We don't take Acumatica implementation work. Per-instance flat pricing means we have zero incentive to push hours your way. You keep the implementation margin; we take the ops layer nobody wanted to own."* This is the highest-stakes sentence on the page for a VAR buyer. Never rewrite.
- **§04 Slack positioning.** *"Your workspace. Your alerts. Your clients' workspaces stay separate."* The "your workspace / your clients' workspaces" framing is the exact mental model a VAR needs. Expand the section's visual anchor (see fix item 5) but do not touch these three sentences.
- **Pricing disclosure + flat axis.** $300/mo per Acumatica instance, no tiers, no seats, no end-customer meter. The pricing printout ivory inversion is the right visual move — it's the one bright object on the dark console, which is exactly the weight it deserves.
- **`01 / 07` section numbering.** Mission Control register convention. Keep it.
- **Ivory pricing printout.** The "inverted ivory object pinned to dark console" signature move. Do not remove, do not dim.
- **The `/api/ci-metrics` "Last publish" panel.** Real fetch, real commit SHA, real relative timestamp. Already live. Its role should expand (fix item 1) but the concept and the endpoint are correct.
- **Terminal green as single accent.** `#10B981` per design-system doc. Do not migrate to burnt orange (`#FF7A1A` is retired per [memory/context/b-studio-design-system.md](https://github.com/studio-b-ai/acuops-website/blob/main/docs/2026-04-21-website-audit.md) anti-patterns list).
- **Comic Book DNA applied.** Hard edges, `border-radius: 0`, already shipped in [#2](https://github.com/studio-b-ai/acuops-website/pull/2). Do not retrofit — already done.

## What's broken — fix in this order of impact-per-effort

### 1. Live dogfood in the hero — the single highest-impact change

The dispatch strip at the top of the page is static (`STATUS ACTIVE` · `UPTIME 99.97` · hardcoded metrics). The "Last publish" panel further down DOES fetch real data from `/api/ci-metrics`. The imbalance is the problem — a VAR reads the dispatch strip first and prices it as marketing chrome. Make the dispatch strip tell the truth.

**Build:** Promote the dispatch strip to a real-fetching operations panel.

- **Left slot:** Last CI run across AcuOps-protected instances. Blinded client identifier (`[client-1]` or `[partner-1]`), real commit SHA (short form), real relative timestamp (*"4 min ago"*). Pulled from the `studio-b-ai/acuops-pipeline` GitHub Actions API, OR from a thin Railway proxy for private-repo Actions runs (`studio-b-ai/client-asthetik`). Pick the stack that's cheapest to ship; blinded real data beats comprehensive real data.
- **Middle slot:** 7-day probe-success sparkline. SVG, no JS chart lib, 20–30 data points at most. Pulled from the `webhook-router /api/ci-metrics` endpoint (already live, already CORS-enabled — extend its response shape if needed).
- **Right slot:** Relative timestamp of the most recent post-publish test run, with the test name (*"TestPCCLeadTimeTab · 14 min ago"*). Visible proof the test matrix is running, not aspirational.

Fetch cadence: 60s on page load, 5-min CDN cache. Server-side blinding (never trust client-side redaction — same rule as Amplify's audit ledger). No fake data under any circumstances; if the endpoint 500s, render the last-known-good snapshot with a muted *"cached"* label, not a placeholder.

**Why this is #1:** A VAR who watches the dispatch strip update while they scroll has already been sold. Everything else on the page supports this moment.

### 2. Capability numerals at 6rem+ in terminal green

Section §02 (Capabilities) currently describes *"12 probes, 146 tests"* in body-copy-weight text. These are the pitch. They should be the hero of the section.

**Build:** Two oversized numerals, display-weight Geist, 6–7rem, `var(--signal)` terminal green. Under each, one concrete artifact in JetBrains Mono:

```
           12                           146
      probes                     post-publish tests
   probe-so-lookup          TestPCCLeadTimeTab
```

CSS-only change; no new data wiring. Worst-effort, highest-visual-return swap on the page.

### 3. 30-day receipts tile

Section §01 opens with *"The problem"* but doesn't yet carry the receipts that make the problem concrete. Add one tile near the top of §01 or the head of §02:

> **Past 30 days across all AcuOps-protected instances:**
> 4 mid-publish failures caught · 2 schema drifts detected · 1 Acumatica minor-version break pre-caught · 0 customer-visible incidents.

Real or blinded-real numbers only. The `0 customer-visible incidents` line is the closer — VARs don't buy probes, they buy the absence of 2 AM Slack pings from their customer.

**Source of truth:** a small rollup endpoint on `webhook-router` that counts `probe_failure`, `schema_drift`, and `release_pre_flight_fail` events from the last 30 days. Cache at the edge (5-min TTL). If the endpoint isn't up by PR 5b time, ship with hardcoded numbers for the current 30-day window and add the wire in a follow-up chip.

### 4. Scale the §03 terminal block 2×

VARs who identify as technical (most do) want to see stdout, not prose. §03 *"How it works"* has a terminal-looking block today, but it's small and visually dominated by the surrounding copy.

**Build:** Scale the block 2× on desktop. Real-looking publish logs, real SHA, a real-shape `Customization.publishEnd` response with the `isCompleted: true · isFailed: false` trio plus a few `Sql#N(skipped, already applied)` lines (those are the signature marker of a real Acumatica publish — anyone who's watched one recognizes them instantly).

Use a single monospace stack (`var(--mono-font)` = JetBrains Mono). Keep the terminal chrome minimal: one label strip (`publishEnd ▸ 2026-04-19 19:57:06 UTC`), the log body, done. No traffic-light window controls (that's macOS Terminal grammar, not NOC grammar).

### 5. Partner-branded Slack example in §04

§04 says *"Your workspace, your alerts"* but the Slack card mockup is generic — no VAR logo slot, no VAR handle, no customizable alert template. A VAR reading the section can't see themselves in it.

**Build:** Replace the generic card with a three-part mockup:
- **Top strip:** mocked VAR workspace name + avatar slot (use a placeholder like `@partner-var` with a neutral mark the VAR can imagine as their own).
- **Message body:** a real-shape AcuOps alert (`probe_failure` on `[client-1]` · container publish `c7f22e1` · SOP `ensure_gi_role_access` failed at step 3 · Slack CTA: "Open dashboard").
- **Bottom caption:** *"Customized alert template · renames and routing under your control"* — tells the VAR they can make it their thing.

The point is less the literal mockup and more that the VAR's imagined brand name fits on top of AcuOps' plumbing without visual friction.

### 6. Four VAR-specific FAQ additions

The current six FAQs are strong but miss four anxieties the buyer profile demands:

- **"What happens on 24.2 → 24.3?"** — Named release numbers matter to VARs who are mid-upgrade-cycle. Answer: the 146-test suite runs against the upgrade candidate in a sandbox before it hits their client's production; diff in Slack before Acumatica's release notes land.
- **"Can my junior consultant run this without pager duty?"** — Disarms the on-call-culture anxiety (#5 in the buyer profile). Answer: no. Alerts route to a VAR-controlled Slack channel during business hours only by default. Escalation policy is opt-in. AcuOps doesn't impose pager rotation.
- **"Do you charge for my sandbox tenant?"** — Answer: no. The meter is Acumatica *instances* (production URLs), not tenants. Sandboxes ride free under the same license.
- **"If my customer leaves, what happens to the data?"** — Every VAR has an account that's churned or will. Answer: probe history + test runs are retained for 90 days post-deprovision for audit; then hard-deleted. Export available on request. You keep your customer's data; we don't.

Add these in order — each answers one buyer anxiety in under three sentences. Rule #7 of the voice contract applies: opinionated, direct, no "please contact sales."

### 7. Bright committed demo CTA panel

The current demo CTA section (*"Request demo"* eyebrow, HubSpot form) is another dark section — visually indistinguishable from the surrounding console. A CTA that looks like every other panel doesn't commit.

**Build:** One bright visual surface on the page. Two options, pick the one that rhymes with the ivory pricing printout:

- **Option A (ivory printout):** a second ivory inversion, narrower than the pricing card, with the HubSpot form rendered in dark-on-ivory. Reads as *"the other committed object"* — pricing and demo are the two moments that demand a pause.
- **Option B (terminal-green commitment panel):** full-width `var(--signal-dim)` panel (`rgba(16,185,129,0.15)`) with a terminal-green border top + bottom, dark text inside. Reads as *"the one active moment."*

Both work for the register. Option A honors the existing family grammar (two ivory objects, bracketing the dark middle). Option B is more assertive. Default if undecided: A.

Either way, the form itself must not change. The HubSpot embed continues to use the shared Studio B form ID (`ea2c2f4a-4e71-4dd5-8747-814c87634f5c`) with per-register styling (`.hs-form-console`).

### 8. Fade-in safety net

If the page uses `.fade-in` classes with `opacity: 0` (it does — 7 sections are wrapped in `<section class="fade-in">`), implement the three-fallback pattern from [bolt-website/index.html](https://github.com/studio-b-ai/bolt-website/blob/main/index.html):

1. `if (!('IntersectionObserver' in window))` → reveal immediately
2. `setTimeout(reveal-all, 1500)` belt-and-suspenders
3. `@media (prefers-reduced-motion: reduce)` CSS override
4. `.no-js` selector on `<html>` (toggled by a `<script>` at document head) for JS-disabled browsers

**Why this is on the list:** link-preview bots (LinkedIn, Slack, Twitter), full-page screenshot tools, slow scrollers (iPad Safari on 3G), and accessibility users with `prefers-reduced-motion: reduce` — all currently see a blank page for the fold or longer. This is the one item that's invisible when correct and embarrassing when broken. Cheap insurance.

## Scope handoff — PR 5a vs PR 5b

The AcuOps License Gate arc splits this audit's execution into two PRs so the commercial boundary lands first and the visual uplift stays focused.

**PR 5a — License-boundary copy + license-status tile** (ships after PR 1 lands)

- Homepage hero/pricing frames free vs. paid explicitly: *"Self-install the pipeline today — free. Add Mission Control for $300/mo/instance."*
- New FAQ entry: *"Is it really BSL-licensed? How do you enforce payment?"* — direct honest answer, no legal posturing. Wording target: *"BSL-1.1 for the pipeline — you can read, fork, and run it. Mission Control (probes, alerts, tests, dashboard) is gated by a license key validated against studiob-api. No key, no probes. We enforce by plumbing, not by paperwork."*
- VAR-margin reframing on the pricing section: *"$300/mo out of a $5k/mo engagement is 6% — and it's the reason your customer doesn't churn. Bake it into your retainer. AcuOps never appears on their invoice."*
- New hero-adjacent tile: **License status lookup**. VAR pastes an `ACUOPS_LICENSE_KEY`, client-side `fetch()` to `/api/v1/acuops/licenses/validate`, renders roster + expiry countdown. CORS on `studiob-api` enabled for `https://acuops.com` origin as part of this PR.

**PR 5b — Mission Control visual uplift** (ships after PR 5a stable for 24h)

All 8 items above. Ship as one PR unless any single item grows beyond ~150 LOC and 2 hours — then split the offending item into its own follow-up.

## The "wow that's too much" bar for THIS register

Unlike b.studio (Billboard — loud through color and yolk-yellow hero) and Bolt (Cut Sheet — loud through warehouse yellow and industrial signage), Mission Control achieves *"too much"* through **DENSITY of live truth**.

Concretely: every square inch should carry a real artifact a VAR can verify.

- Real commit SHA in the dispatch strip (not a fake one)
- Real test name in the capability numerals (not "146 tests")
- Real `Sql#N(skipped, already applied)` lines in the §03 terminal (not `$ npm install`)
- Real blinded-client name in the alert mockup (not "Acme Corp")
- Real 30-day counts in the receipts tile (not "dozens of incidents caught")

If the page feels restrained compared to Bolt's navy-yellow stripes or b.studio's yolk hero, that's the register doing its job. Terminal green is the accent; live truth is the amplifier. The VAR who reads every real artifact on the page and can't find the one fake number is the VAR who converts.

## Ship-to-acuops.com verification recipe (post-PR-5b)

Do not mark PR 5b complete until this full recipe runs green. Merged PR ≠ shipped site. Every step documented here was decided in the 2026-04-21 scoping session.

1. **Verify Railway source.** Query the Railway GraphQL API or `railway` CLI to confirm the `acuops-website` service is deploying from `studio-b-ai/acuops-website:main`. Do not assume. The pattern was bitten by `bolt-website` for months before it was caught.
2. **Repoint if wrong.** If the service is pointed at a stale repo or branch, repoint via the GraphQL mutation pattern (documented in `bolt-website` PR #2 workflow).
3. **Trigger a fresh deploy.** Use the MCP `railway_trigger_deploy` + `railway_poll_deploy` tools (or `railway up --detach`). Do not claim success until the deployment status reads `SUCCESS`.
4. **Playwright-verify against the live domain.** Hit `https://acuops.com/`. Confirm (a) the hero renders, (b) the dispatch strip is fetching real data (run ID differs from the static placeholder), (c) the license-status tile renders even with an empty key, (d) the 30-day receipts tile shows non-placeholder numbers. Screenshot for the PR description.
5. **"Tests pass" is not verification.** [CLAUDE.md Rule 4](https://github.com/studio-b-ai/studiob/blob/main/memory/CLAUDE.md) on production sacredness applies — Playwright against the live domain is the verification bar.
6. **Add footer cross-links.** Every Studio B product site footers back to the firm (`bolt.b.studio` → `b.studio`; `amplify.b.studio` → `b.studio`). Match the pattern. Consider a *"By Studio B"* mark in the nav modeled on bolt's *"B O L T · by Studio B"* anchor. Cross-link Bolt, Amplify, and Relay in the footer too — all four products link to each other from every footer. Canonical footer block: [bolt-website/index.html](https://github.com/studio-b-ai/bolt-website/blob/main/index.html).
7. **Ship the chip once, not twice.** Commit the PR 5b completion chip only after acuops.com serves the new content AND the footer links land on b.studio / bolt.b.studio with one click. Anything less is a false claim.

## References

- [AcuOps License Gate architecture brief](https://github.com/studio-b-ai/studiob/blob/main/docs/plans/2026-04-21-acuops-license-gate-architecture.md) — parent plan this audit feeds into
- [memory/context/b-studio-design-system.md](https://github.com/studio-b-ai/acuops-website) — five-register family, Mission Control palette + mark + anti-patterns (user-side memory file; reference copy in the `studiob` repo under `memory/context/` for reviewers)
- [memory/feedback_studio-b-voice.md](https://github.com/studio-b-ai/acuops-website) — 8 voice rules that apply to every word on the page
- [bolt-website PR #1](https://github.com/studio-b-ai/bolt-website/pull/1) — compare-don't-copy reference for *"uplifting an existing register without swapping its palette"* (Cut Sheet, not Mission Control)
- [b-studio-website PR #17](https://github.com/studio-b-ai/b-studio-website/pull/17) — compare-don't-copy reference for complete section architecture at the Billboard register
- [amplify/docs/plans/2026-04-21-amplify-website-audit.md](https://github.com/studio-b-ai/amplify/blob/main/docs/plans/2026-04-21-amplify-website-audit.md) — parallel audit for the Prospectus register; this doc mirrors its structure
- [acuops-website PR #2](https://github.com/studio-b-ai/acuops-website/pull/2) — Mission Control reskin that this audit builds on
