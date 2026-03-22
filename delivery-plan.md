# HGS Delivery Plan — Epics, Stories, Tasks, and Research Notes

## Purpose

This document translates `product.md` into an execution-ready backlog structure with:
- recommended delivery phases,
- Epics → Stories → Tasks,
- cross-cutting dependencies,
- major risks,
- targeted research notes that materially affect scope.

It is intentionally more operational than the original product spec.

---

## 1. What the current repository implies

The current repository appears to contain the 26 playable HTML5 games plus a security review, but not yet the actual marketplace platform (storefront, backend, auth system, payments, admin tooling, analytics pipeline, etc.).

That means the MVP scope in `product.md` is not just “feature work”; it is primarily **new platform construction** around an existing catalog.

### Practical consequence

Before user-facing marketplace features, HGS needs a **Phase 0 foundation** covering:
1. application architecture,
2. game catalog ingestion/metadata normalization,
3. hosting/runtime model for embedded games,
4. auth and payments foundations,
5. compliance/security decisions that affect information architecture and UX.

---

## 2. Recommended delivery phases

## Phase 0 — Foundation and Risk Reduction

Goal: create the platform skeleton and resolve blockers that would otherwise cause major rework.

**Exit criteria**
- Framework, backend, database, and deployment path selected.
- Canonical game metadata model exists for all 26 titles.
- Embedded game communication contract defined.
- Compliance decisions recorded for casino-category access, age gating, and target geographies.
- Security baseline defined (CSP, iframe isolation, webhook verification, secrets handling, audit logging).

## Phase 1 — MVP Commerce and Core Play Experience

Goal: users can browse catalog, create accounts, buy credits, unlock games, and play them.

**Exit criteria**
- Public storefront live.
- User registration and login live.
- Credits can be purchased via Stripe.
- Game unlock flow works.
- Published games can be launched in HGS player shell.
- Admin can manage catalog and prices.

## Phase 2 — Retention and Trust

Goal: make users return, compete, and contribute trustworthy content.

**Exit criteria**
- Leaderboards, ratings/reviews, achievements, and notifications are live.
- Score submission path has meaningful validation/anti-abuse controls.
- Social and profile depth is sufficient to support repeat usage.

## Phase 3 — Marketplace Expansion and Optimization

Goal: improve discovery, analytics, monetization leverage, and publisher scalability.

**Exit criteria**
- Third-party submission pipeline exists.
- Advanced recommendations/discovery are live.
- Events/tournaments and deeper analytics support growth loops.

---

## 3. Prioritized Epic list

Recommended implementation order:

1. **Epic A — Product/Compliance Foundation**
2. **Epic B — Platform Architecture & DevOps**
3. **Epic C — Catalog & Content Model**
4. **Epic D — Storefront & Discovery MVP**
5. **Epic E — Accounts & Identity**
6. **Epic F — Wallet/Credits & Payments**
7. **Epic G — Game Player Shell & Entitlement Enforcement**
8. **Epic H — Admin Console**
9. **Epic I — Leaderboards & Score Integrity**
10. **Epic J — Reviews, Ratings, and Social Proof**
11. **Epic K — Achievements, Notifications, and Retention Loops**
12. **Epic L — Publisher Portal & Growth Systems**

---

## 4. Epic → Story → Task breakdown

## Epic A — Product/Compliance Foundation

**Why first**
Several open questions in `product.md` affect IA, UX, payments, legal review, and even audience targeting. This should be resolved before implementation gets too far.

### Story A1 — Define monetization model for MVP
**Outcome:** clear decision between credits-only, subscription-only, or hybrid MVP.

**Tasks**
- Compare credits-only vs subscription-only vs hybrid against implementation complexity.
- Model expected ARPU/LTV assumptions for 26-title catalog.
- Decide whether free trials are catalog-wide, category-based, or title-based.
- Decide whether casino titles use different pricing tiers.
- Document MVP monetization decision and non-goals.

### Story A2 — Define age-gating and casino access policy
**Outcome:** approved policy for accessing casino-themed games.

**Tasks**
- Define target launch markets and excluded markets.
- Classify games that require age/casino gating.
- Define neutral DOB collection flow and storage policy.
- Define whether under-threshold users are blocked from account creation, casino category only, or all purchases.
- Add legal review checklist and sign-off owner.

### Story A3 — Define audience positioning and COPPA posture
**Outcome:** product explicitly targets general audience / teen+ audience rather than accidentally drifting into child-directed territory.

**Tasks**
- Review art, copy, and marketing language for child-directed signals.
- Define minimum supported age policy in Terms/UX.
- Decide how profile/community features behave for minors.
- Create privacy requirements for age-related data.

### Story A4 — Establish ratings/compliance packaging plan
**Outcome:** plan for ratings and disclosure where relevant.

**Tasks**
- Determine whether HGS needs storefront-level age/rating labels and/or per-game labels.
- Decide whether any launch channels require IARC/ESRB-style disclosures.
- Add metadata fields for content descriptors (e.g. simulated gambling).
- Capture compliance review workflow for new games.

---

## Epic B — Platform Architecture & DevOps

### Story B1 — Select and bootstrap application stack
**Outcome:** one agreed frontend/backend/data stack.

**Tasks**
- Confirm Next.js vs React SPA decision.
- Confirm backend choice (FastAPI vs Node/Fastify/Express).
- Create mono-repo or poly-repo structure.
- Add environment management strategy.
- Add local development scripts and CI baseline.

### Story B2 — Provision infrastructure foundations
**Outcome:** deployable environments for dev/staging/prod.

**Tasks**
- Set up hosting for app frontend.
- Set up backend hosting.
- Provision PostgreSQL.
- Provision object storage/CDN for assets.
- Configure domain, TLS, and environment secrets.
- Configure preview/staging deployment flow.

### Story B3 — Establish security baseline
**Outcome:** platform has baseline controls before launch features are layered on.

**Tasks**
- Define CSP for host app and for game iframe origins.
- Define frame embedding restrictions (`frame-ancestors` / allowed embed contexts).
- Define secure cookie/token strategy.
- Add rate limiting and bot protection requirements.
- Add webhook signature verification pattern.
- Add audit log requirements for purchases/admin actions.

### Story B4 — Define observability and supportability
**Outcome:** errors, purchase failures, and game load failures are diagnosable.

**Tasks**
- Add structured logging requirements.
- Add metrics/event taxonomy.
- Define error monitoring tooling.
- Define operational dashboards for signups, checkout conversion, and game-load failure rate.

---

## Epic C — Catalog & Content Model

### Story C1 — Normalize game metadata for all 26 titles
**Outcome:** all games exist as first-class catalog records.

**Tasks**
- Create canonical game schema.
- Assign slugs, categories, pricing defaults, device support flags, and visibility status.
- Create short and long descriptions for each game.
- Tag casino-themed games distinctly.
- Record asset paths and launch URLs for every title.

### Story C2 — Create media asset pipeline
**Outcome:** catalog cards and detail pages can render consistent media.

**Tasks**
- Generate/collect thumbnails for all titles.
- Generate screenshot set and optional preview GIF/video for priority titles.
- Create featured-game metadata model.
- Define aspect-ratio and image optimization rules.

### Story C3 — Define catalog governance
**Outcome:** admins can publish safely and consistently.

**Tasks**
- Define draft/published/archive states.
- Define required metadata fields for publishing.
- Create validation rules for category, price, descriptions, and media.
- Define “similar games” recommendation seed data for MVP.

---

## Epic D — Storefront & Discovery MVP

### Story D1 — Landing page and featured content
**Outcome:** users can land on HGS and immediately understand the value proposition.

**Tasks**
- Design hero section.
- Build featured/promoted game module.
- Add responsive page layout.
- Add footer and legal links.
- Add empty/loading states.

### Story D2 — Catalog browsing and filtering
**Outcome:** users can find games quickly.

**Tasks**
- Build game card component.
- Implement category filters.
- Implement title search with instant filtering.
- Add free vs premium visual states.
- Add pagination or infinite scroll decision.

### Story D3 — Game detail pages
**Outcome:** each title has a conversion-oriented detail page.

**Tasks**
- Build game detail template.
- Add screenshots/GIF carousel.
- Add compatibility badges.
- Add pricing and entitlement state.
- Add similar-game recommendations.
- Add aggregate rating display placeholder if reviews ship later.

---

## Epic E — Accounts & Identity

### Story E1 — Email/password authentication
**Outcome:** users can create and access accounts reliably.

**Tasks**
- Implement signup, login, logout.
- Implement email verification flow.
- Implement password reset flow.
- Add session persistence and device sign-out strategy.

### Story E2 — OAuth sign-in
**Outcome:** lower-friction acquisition.

**Tasks**
- Add Google OAuth.
- Add GitHub OAuth.
- Define account linking and duplicate-email handling.
- Define account recovery edge cases.

### Story E3 — User profile and library
**Outcome:** users can manage identity and owned games.

**Tasks**
- Create profile page.
- Add avatar upload policy.
- Show owned games.
- Show play history and transaction history.
- Add privacy controls for profile visibility if social features are planned.

---

## Epic F — Wallet/Credits & Payments

### Story F1 — Credits wallet model
**Outcome:** credits are represented safely and consistently.

**Tasks**
- Define ledger-based credits model.
- Define transaction types (purchase, unlock, refund, promo, adjustment).
- Define idempotency rules.
- Add balance calculation strategy.
- Define refund/reversal behavior.

### Story F2 — Stripe checkout for credit packs
**Outcome:** users can purchase credits with cards and wallet methods supported by Stripe.

**Tasks**
- Create credit pack catalog.
- Implement Checkout Session creation.
- Configure hosted vs embedded Checkout.
- Add Apple Pay / Google Pay acceptance through Stripe-supported checkout path.
- Implement post-payment success/cancel UX.

### Story F3 — Payment fulfillment and receipts
**Outcome:** successful payments reliably add credits.

**Tasks**
- Implement webhook endpoint.
- Verify Stripe webhook signatures.
- Add idempotent fulfillment job.
- Persist transaction records.
- Trigger receipt email.
- Add reconciliation/admin support workflow.

### Story F4 — Entitlement purchase flow
**Outcome:** users can unlock a game with credits and retain access.

**Tasks**
- Implement unlock confirmation flow.
- Check sufficient balance.
- Create purchase record and debit credits atomically.
- Add owned/unlocked state to catalog and detail pages.
- Handle refund and support edge cases.

### Story F5 — Optional subscription discovery spike
**Outcome:** subscription remains a researched but intentionally deferred or approved scope item.

**Tasks**
- Compare subscription implementation cost to credits-only MVP.
- Determine whether subscription interacts with individual unlocks or bypasses them.
- Recommend defer/keep decision.

---

## Epic G — Game Player Shell & Entitlement Enforcement

### Story G1 — Embedded game runtime shell
**Outcome:** games play within HGS chrome.

**Tasks**
- Build iframe player shell.
- Add loading spinner/progress state.
- Add fullscreen toggle.
- Add quick exit / back to catalog.
- Add mobile orientation messaging handoff.

### Story G2 — Secure game-to-host integration contract
**Outcome:** score, session, and state events have a forward-compatible integration layer.

**Tasks**
- Define typed message schema.
- Add origin allowlist and strict `targetOrigin` policy.
- Define handshake/nonce strategy.
- Implement wrapper bridge around legacy games where needed.
- Decide fallback behavior for games that cannot yet emit trusted events.

### Story G3 — Access control and launch gating
**Outcome:** only entitled and allowed users can launch gated content.

**Tasks**
- Enforce login requirement where needed.
- Enforce age/casino restrictions before launch.
- Enforce ownership/unlock checks.
- Create trial mode gating behavior if enabled.
- Handle unauthorized/deep-link access gracefully.

### Story G4 — Save/progress capability assessment
**Outcome:** realistic understanding of which games can support cloud saves.

**Tasks**
- Audit each game for existing save/progress behavior.
- Separate “easy to integrate” vs “requires source-level game changes.”
- Define a minimal save-state interface for future work.

---

## Epic H — Admin Console

### Story H1 — Catalog administration
**Outcome:** internal team can manage listings without direct database edits.

**Tasks**
- Build game list admin table.
- Add publish/unpublish controls.
- Add metadata edit form.
- Add price editing.
- Add media management.

### Story H2 — Commerce and user operations
**Outcome:** support team can answer purchase and account issues.

**Tasks**
- Add user lookup.
- Add transaction history view.
- Add credit adjustment workflow with audit log.
- Add refund/support notes model.

### Story H3 — Basic business metrics dashboard
**Outcome:** founders can monitor MVP health.

**Tasks**
- Show users, revenue, purchases, active players.
- Add date filters.
- Add top games by views/plays/revenue.
- Add export requirement definition.

---

## Epic I — Leaderboards & Score Integrity

### Story I1 — Score submission pipeline
**Outcome:** games can submit scores to backend.

**Tasks**
- Define per-game score event contract.
- Add authenticated score submission API.
- Add replay and tamper-resistance checks where possible.
- Store raw submission payloads for audit.

### Story I2 — Leaderboard views
**Outcome:** users can view per-game and global rankings.

**Tasks**
- Build per-game leaderboard UI.
- Build global leaderboard UI.
- Add weekly/monthly/all-time filters.
- Add pagination and anti-spam display rules.

### Story I3 — Anti-cheat strategy for legacy games
**Outcome:** realistic trust model for scores.

**Tasks**
- Tier games by score trustworthiness.
- Define server-validation feasibility for each title.
- Add anomaly detection rules for impossible scores.
- Add moderator tools to hide invalid scores.
- Explicitly gate casino-related competition until validation is acceptable.

---

## Epic J — Reviews, Ratings, and Social Proof

### Story J1 — Ratings
**Outcome:** users can leave one rating per game.

**Tasks**
- Add rating model and constraints.
- Add aggregate rating calculations.
- Add update/retract behavior.
- Decide whether ownership/playtime is required before rating.

### Story J2 — Text reviews and moderation
**Outcome:** users can submit reviews without immediately creating trust/safety problems.

**Tasks**
- Add review submission/edit/delete flow.
- Add profanity/spam moderation strategy.
- Add report/helpful voting model.
- Add admin moderation queue.

### Story J3 — Catalog ranking by quality
**Outcome:** reviews impact discovery.

**Tasks**
- Add sort by rating.
- Add minimum-review thresholds.
- Define anti-review-bomb heuristics.

---

## Epic K — Achievements, Notifications, and Retention Loops

### Story K1 — Achievement framework
**Outcome:** games can award structured achievements.

**Tasks**
- Define achievement schema.
- Add user achievement records.
- Add profile badge rendering.
- Identify easy-launch achievements for 5–8 top titles first.

### Story K2 — Progress tracking
**Outcome:** HGS can track engagement beyond purchases.

**Tasks**
- Track sessions, plays, time played, streaks.
- Define game event taxonomy.
- Add analytics pipeline for player milestones.

### Story K3 — Notifications
**Outcome:** users receive meaningful re-engagement prompts.

**Tasks**
- Build in-app notification center.
- Add email notification preferences.
- Add transactional vs marketing notification distinction.
- Add weekly digest template.

### Story K4 — Friends/social loops
**Outcome:** users can connect and challenge each other.

**Tasks**
- Add friend model and requests.
- Add activity feed.
- Add score challenge links.
- Define privacy controls and abuse handling.

---

## Epic L — Publisher Portal & Growth Systems

### Story L1 — Third-party game submission pipeline
**Outcome:** platform can scale catalog beyond first-party content.

**Tasks**
- Build submission form.
- Define package format and technical requirements.
- Add malware/security scan pipeline.
- Add review queue and QA checklist.
- Add revenue-share configuration model.

### Story L2 — Advanced discovery and personalization
**Outcome:** users get smarter recommendations.

**Tasks**
- Add tags system.
- Add trending/new/popular calculations.
- Add curated collections.
- Add recommendation strategy v1 (rule-based before ML).

### Story L3 — Events, tournaments, and loyalty
**Outcome:** stronger repeat-play economics.

**Tasks**
- Design tournament model.
- Add event scheduling.
- Add prize/entry-fee ledger behavior.
- Add referral and daily bonus systems.
- Add abuse-prevention rules.

### Story L4 — Advanced analytics
**Outcome:** product decisions become measurable.

**Tasks**
- Add DAU/MAU, retention, session length, conversion funnel reporting.
- Add cohort reporting.
- Add game load-performance analytics.
- Add segmentation by category and spend behavior.

---

## 5. Cross-cutting dependencies and blockers

These are the most important dependencies that will otherwise create hidden rework:

### Blocker 1 — Compliance decisions before casino launch
If casino-category titles launch without settled age-gating, geography, and disclosure rules, HGS risks redesigning account flows, player gating, and catalog IA after the fact.

### Blocker 2 — Entitlement model before player shell polish
Do not overbuild the player UX until login state, unlock state, and game launch authorization are finalized.

### Blocker 3 — Event/message contract before leaderboards/achievements
Leaderboards, achievements, and analytics all depend on a stable event contract between the game and HGS host.

### Blocker 4 — Metadata completeness before storefront design sign-off
A polished storefront depends on complete thumbnails, screenshots, descriptions, pricing, and category normalization.

### Blocker 5 — Admin tooling before catalog scaling
Without admin workflows, every small content or pricing change becomes engineering work.

---

## 6. Suggested MVP slicing

If the goal is to get a credible MVP live quickly, I would recommend the following slice:

### MVP Slice 1 — Internal alpha
- Epic A (minimum decisions only)
- Epic B (minimum platform)
- Epic C
- Epic D (landing + catalog)
- Epic E1 only (email/password)
- Epic F1–F4
- Epic G1–G3
- Epic H1 minimum

### MVP Slice 2 — Limited beta
- Epic E2–E3
- Epic H2–H3
- Epic I1 minimum score ingestion
- basic reviews or ratings only if moderation is ready

### MVP Slice 3 — Public launch hardening
- stronger fraud controls
- receipts/support workflows
- metrics dashboards
- legal copy/privacy/terms finalization
- launch content operations playbook

### Explicit deferrals I would recommend
To reduce delivery risk, I would defer these until after commercial MVP validation:
- subscription model,
- social graph/friends,
- text reviews with helpful voting,
- tournaments,
- third-party publisher portal,
- cloud save for all games,
- sophisticated recommendations.

---

## 7. Rough sizing by Epic

These are relative sizes only, intended for planning conversation rather than commitment.

| Epic | Relative Size | Notes |
|------|---------------|-------|
| Epic A — Product/Compliance Foundation | M | High leverage, low code, high decision value |
| Epic B — Platform Architecture & DevOps | L | New platform creation |
| Epic C — Catalog & Content Model | M | Needs content ops + engineering |
| Epic D — Storefront & Discovery MVP | M | Straightforward if metadata is ready |
| Epic E — Accounts & Identity | M | Commodity but still integration-heavy |
| Epic F — Wallet/Credits & Payments | L | High correctness requirement |
| Epic G — Game Player Shell & Enforcement | L | Legacy game integration complexity |
| Epic H — Admin Console | M | Often underestimated |
| Epic I — Leaderboards & Score Integrity | L | Hard because legacy games are client-side |
| Epic J — Reviews, Ratings, and Social Proof | M | Moderation overhead matters |
| Epic K — Achievements/Notifications/Retention | M/L | Depends on event quality |
| Epic L — Publisher Portal & Growth Systems | XL | Separate product in its own right |

---

## 8. Suggested ownership map

For planning purposes, these workstreams likely need parallel owners:

- **Product/Founder:** Epic A, pricing, legal/compliance decisions, launch sequencing.
- **Frontend:** Epics D, E (UI), G (shell), H (admin UI).
- **Backend/Platform:** Epics B, E (auth), F, G (launch auth), I, analytics primitives.
- **Content Ops/Design:** Epic C, storefront copy, thumbnails/screenshots, policy pages.
- **Compliance/Legal:** age gate, market restrictions, terms/privacy, casino positioning.
- **QA:** device/browser matrix, payment flows, entitlement edge cases, game compatibility.

---

## 9. Research notes that materially affect scope

These are not exhaustive legal conclusions; they are scoping inputs.

### Research note 1 — Simulated gambling affects ratings/disclosure scope
Official ESRB content descriptors include **“Simulated Gambling”** and **“Real Gambling”**, which means casino-themed titles should be treated as a distinct compliance/content-label category in HGS metadata and UX.

**Scoping impact**
- Add content descriptor fields at the game level.
- Add casino-category gating rules independent of ordinary categories.
- Make age/rating labels part of catalog and detail page design.

### Research note 2 — Age screening changes COPPA obligations depending on product positioning
FTC guidance says a general-audience site may use a **neutral** age screen and may block under-13 users, but if the service is actually directed to children or mixed-audience, the rules are different and parental-consent obligations can be triggered.

**Scoping impact**
- HGS must explicitly define intended audience before implementing sign-up and age-gate UX.
- DOB collection cannot be treated as a trivial field; it affects privacy, flows, and data retention.
- Marketing/design choices should avoid accidentally creating a child-directed posture if that is not intended.

### Research note 3 — Stripe Checkout lowers payment-scope complexity, but fulfillment correctness still matters
Stripe’s current documentation indicates Checkout supports one-time and subscription payments via Checkout Sessions, and fulfillment should be driven from verified webhook events rather than client redirects alone.

**Scoping impact**
- Prefer Stripe Checkout for MVP over a more custom payment UI.
- Treat webhook verification and idempotent fulfillment as required MVP work, not polish.
- Credits need a ledger model because payment success and game unlocks are separate financial events.

### Research note 4 — The current game portfolio increases score-integrity scope
The repository’s existing security review already shows several casino titles and score/session integrations that are currently client-side or loosely coupled. That means “leaderboards” is not a simple UI feature; it is a trust and architecture project.

**Scoping impact**
- Leaderboards should not be in the same confidence bucket as catalog/auth/payments.
- Score validation should be phased, game-tiered, and initially conservative.
- Casino-related competitive features should be explicitly blocked until validation quality is acceptable.

---

## 10. Recommended next actions

If I were turning this into actual delivery planning, I would do these next:

1. Create a one-page **MVP decision memo** for pricing model, free tier, audience target, and casino access policy.
2. Create the initial **data model** for users, games, transactions, purchases, entitlements, and score events.
3. Build a **catalog import spreadsheet or seed file** for all 26 games.
4. Run a **technical spike** for the iframe host shell and one representative game from each engine/type.
5. Run a **payments spike** using Stripe Checkout + webhook fulfillment.
6. Decide whether **leaderboards** remain MVP or move to immediate post-launch.

---

## 11. Recommended first sprint candidates

A practical Sprint 1 could be:
- finalize MVP decisions for monetization + audience/compliance,
- stand up app/backend/database skeleton,
- define game metadata schema,
- import all 26 games into seed data,
- build catalog page with static cards,
- build one playable detail page + iframe shell prototype,
- run Stripe checkout proof of concept,
- write entitlement and transaction schemas.

That sprint would answer the biggest unknowns with the least waste.


## 12. External sources consulted

- ESRB/IARC content descriptors (includes “Simulated Gambling” / “Real Gambling”): https://www.globalratings.com/downloads/esrb_english-spanish_content_descriptors_interactive_elements.pdf
- FTC COPPA FAQ (neutral age screening, general-audience vs mixed-audience considerations): https://www.ftc.gov/business-guidance/resources/complying-coppa-frequently-asked-questions
- Stripe Checkout overview (Checkout Sessions; one-time and subscription support): https://docs.stripe.com/payments/checkout
