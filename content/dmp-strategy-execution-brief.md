# dmp — Strategy & Execution Brief (v0)

## Context & Intent

### Why this product exists

- dmp appears to target a practical gap: non-technical users want a simple, phone-first way to produce physical output from digital content without setup complexity.
- The likely user value is convenience, immediacy, and reliability—not technical customization.
- For a solo founder, this can be attractive if the product is assembled from existing components (OEM hardware + lightweight software orchestration) instead of deep invention.

### Problem space

- Existing hardware + app experiences are often fragmented:
  - Hardware purchase friction (too many SKUs, confusing specs).
  - Software setup friction (pairing, configuration, permissions).
  - Ongoing usage friction (inconsistent output, unclear troubleshooting).
- Non-technical users need a “just works from phone” path, with minimal decisions.

### Why speed matters for v0

- Early demand signals are more valuable than polished architecture.
- Hardware + software products have hidden operational risks; these should be exposed quickly with small-scale selling.
- Speed to first sale reduces strategy ambiguity by grounding decisions in:
  - Real conversion behavior.
  - Real support burden.
  - Real unit economics.

---

## Strategic Decisions Already Made

### Constraints (explicit)

- v0 product stage.
- Solo founder execution.
- Fast advertising + fast selling are top priorities.
- Hardware + software hybrid.
- OEM / white-label hardware is acceptable.
- Proprietary lock-in is not a goal.
- Non-technical user base.
- Phone-first interaction required.
- No warehousing inventory.
- Target hardware markup around 2× cost.
- Founder retains all decision authority.

### Non-goals / ruled-out directions

- No custom hardware development for v0.
- No deep IP moat or ecosystem lock-in strategy at v0.
- No desktop-first experience.
- No complex multi-role org design or delegation model.
- No inventory-heavy operations model.
- No overbuilt platform before sales validation.

---

## Target Outcome for v0

### What success looks like

A realistic v0 success state in the first phase (30–90 days) is:

- A clearly defined single “starter” product offer is live and purchasable.
- At least one repeatable path from ad click (or content discovery) to paid order exists.
- Post-purchase setup can be completed by non-technical users from a phone with minimal founder intervention.
- Unit economics are directionally viable:
  - Gross margin near target assumptions (including shipping/fees).
  - Support burden low enough for one person.
- Fulfillment model works without owned inventory (drop-ship, supplier-forward-ship, or low-touch on-demand flow).

### Explicitly out of scope for v0

- Broad product line or many SKUs.
- Bespoke companion app with heavy custom engineering.
- Advanced automation, analytics stack, or CRM sophistication.
- International expansion.
- Wholesale/channel strategy.
- Formal team scaling.

---

## Execution Pillars

To keep execution compact and parallelizable, run five pillars:

1. **Offer Definition & Demand Capture**
2. **Hardware Sourcing & Commercial Terms**
3. **Phone-First Software Flow**
4. **Fulfillment, Support & Operating Backbone**
5. **Risk, Compliance & Policy Surface**

---

## Options & Approaches (by Pillar)

## 1) Offer Definition & Demand Capture

### Option A — Single flagship starter bundle

- **Description:** One bundle, one price, one core use case.
- **Pros:**
  - Fastest path to clarity.
  - Simplifies ads, landing page, and support.
- **Cons:**
  - Limits audience segments.
  - May leave some willingness-to-pay uncaptured.
- **Effort:** Low.
- **Risks:**
  - If positioning is wrong, conversion looks weak even if category demand exists.

### Option B — Two-tier offer (basic + better)

- **Description:** Entry bundle plus upgraded bundle.
- **Pros:**
  - Better price discrimination.
  - Creates an anchor effect for the better tier.
- **Cons:**
  - More operational complexity.
  - Harder messaging for non-technical buyers.
- **Effort:** Medium.
- **Risks:**
  - Decision fatigue reduces conversion.

### Option C — Reservation / pre-order before full launch

- **Description:** Collect intent and deposits before finalizing full stack.
- **Pros:**
  - Validates demand quickly.
  - Reduces upfront risk.
- **Cons:**
  - Requires expectation management.
  - Can create trust issues if timelines slip.
- **Effort:** Low to medium.
- **Risks:**
  - Refund/admin burden for a solo founder.

**Recommended default for v0:** Option A.

---

## 2) Hardware Sourcing & Commercial Terms

### Option A — Single OEM supplier with blind-drop shipping

- **Description:** Supplier fulfills directly to customer under neutral or branded packing where possible.
- **Pros:**
  - No warehousing.
  - Lean operational footprint.
- **Cons:**
  - Supplier reliability concentration risk.
  - Packaging/insert control may be limited.
- **Effort:** Medium.
- **Risks:**
  - SLA slippage harms early reputation.

### Option B — Primary + backup supplier setup

- **Description:** Qualify two suppliers early; route by availability and performance.
- **Pros:**
  - Lower stockout risk.
  - Better negotiation leverage.
- **Cons:**
  - More onboarding and QA work.
  - Potential quality variance across batches.
- **Effort:** High (for solo founder).
- **Risks:**
  - Inconsistent customer experience.

### Option C — Domestic distributor partner (higher cost, lower uncertainty)

- **Description:** Source through a local intermediary with stronger support/SLAs.
- **Pros:**
  - Faster shipping and easier returns.
  - Lower communication friction.
- **Cons:**
  - Higher unit cost challenges 2× markup goals.
  - Less margin headroom.
- **Effort:** Low to medium.
- **Risks:**
  - Margin compression.

**Recommended default for v0:** Option A, with a lightweight contingency list (not full dual-source setup yet).

---

## 3) Phone-First Software Flow

### Option A — Mobile web app + QR onboarding (no app store)

- **Description:** User scans QR, lands on responsive web flow for setup and use.
- **Pros:**
  - Fast deployment.
  - No app-store review overhead.
  - Easy updates.
- **Cons:**
  - Hardware/browser integration may be limited.
  - Offline behavior weaker.
- **Effort:** Low to medium.
- **Risks:**
  - Browser compatibility edge cases.

### Option B — “No app” integration via existing OEM app + branded guide layer

- **Description:** Leverage OEM companion app and provide simplified branded instructions/workflows.
- **Pros:**
  - Minimal engineering.
  - Can launch fastest.
- **Cons:**
  - UX dependency on third-party app quality.
  - Lower product defensibility.
- **Effort:** Low.
- **Risks:**
  - OEM app updates can break flow.

### Option C — Thin wrapper app (cross-platform)

- **Description:** Lightweight app as orchestrator around proven APIs.
- **Pros:**
  - Better control of UX.
  - Future extensibility.
- **Cons:**
  - Longer time to market.
  - Ongoing maintenance burden.
- **Effort:** Medium to high.
- **Risks:**
  - Solo-founder throughput bottleneck.

**Recommended default for v0:** Option B immediately; parallel prototype Option A to reduce third-party dependency.

---

## 4) Fulfillment, Support & Operating Backbone

### Option A — Supplier direct ship + founder-run support inbox

- **Description:** Orders routed to supplier; support handled via one shared inbox + FAQ.
- **Pros:**
  - Low overhead.
  - Easy to start.
- **Cons:**
  - Manual processes can accumulate quickly.
- **Effort:** Low.
- **Risks:**
  - Founder overload during spikes.

### Option B — Supplier direct ship + basic help center + scripted triage

- **Description:** Add structured macros, issue tags, and self-serve docs from day one.
- **Pros:**
  - Better scalability with little extra tooling.
  - More consistent responses.
- **Cons:**
  - Slight setup overhead.
- **Effort:** Low to medium.
- **Risks:**
  - Docs can go stale if not maintained.

### Option C — 3PL-assisted exception handling only

- **Description:** Keep direct shipping but use 3PL for returns/exceptions.
- **Pros:**
  - Reduces founder handling of edge cases.
- **Cons:**
  - Added cost and vendor complexity.
- **Effort:** Medium.
- **Risks:**
  - Process mismatch across vendors.

**Recommended default for v0:** Option B.

---

## 5) Risk, Compliance & Policy Surface

### Option A — Minimal baseline legal stack

- **Description:** Clear terms, returns policy, privacy policy, and usage disclaimers.
- **Pros:**
  - Fast, practical baseline.
- **Cons:**
  - Limited coverage for niche scenarios.
- **Effort:** Low.
- **Risks:**
  - Missed edge-case liabilities.

### Option B — Sector-specific legal review before launch

- **Description:** Short paid review by counsel for product claims and liability exposure.
- **Pros:**
  - Better risk confidence.
- **Cons:**
  - Slower launch and additional cost.
- **Effort:** Medium.
- **Risks:**
  - Time drift on non-revenue tasks.

### Option C — Launch with baseline + staged legal hardening post-first sales

- **Description:** Start with clear baseline docs, schedule targeted legal pass after initial validation.
- **Pros:**
  - Preserves launch speed.
  - Rational cost timing.
- **Cons:**
  - Interim risk tolerance required.
- **Effort:** Low to medium.
- **Risks:**
  - Need disciplined follow-through.

**Recommended default for v0:** Option C.

---

## High-Level Work Breakdown

## A. Commercial Offer Setup

- Define one primary customer job-to-be-done.
- Draft one hero offer (bundle contents, delivery promise, support scope).
- Price to target ~2× hardware cost while accounting for:
  - Payment fees.
  - Shipping pass-through or subsidy.
  - Expected return/refund rate.
- Build one conversion-focused landing page with direct checkout.

## B. Supplier & Product Qualification

- Identify 5–10 OEM/white-label candidates.
- Shortlist 2–3 based on lead time, MOQ flexibility, defect rates, and communication quality.
- Order samples and run a practical usability test with phone-first onboarding.
- Negotiate:
  - Unit pricing tiers.
  - Shipping method and SLAs.
  - Defect/replacement terms.
  - Packing insert allowance.

## C. Phone-First User Flow

- Design a 5-step “first successful outcome” journey for non-technical users.
- Produce setup artifacts:
  - QR code card.
  - One-page quick start.
  - 60–120 second micro-tutorial video.
- Implement minimal software layer (or OEM app guidance) required for the first success event.

## D. Ops, Support & Analytics Baseline

- Set up order notifications and a manual fulfillment confirmation loop.
- Create support inbox with issue taxonomy and response macros.
- Publish a concise FAQ covering top 10 likely failures.
- Track core metrics in a simple weekly operating sheet:
  - Visits → checkout starts → paid orders.
  - Delivery lead time.
  - First-week support contact rate.
  - Refund rate.

## E. Risk Baseline

- Publish clear store policies (returns, warranty handling approach, privacy, terms).
- Define acceptable product claims (what is promised vs not promised).
- Prepare incident response checklist for defective units and delayed shipments.

---

## Sequencing & Dependencies

## Must happen first (Week 1)

- Select one use case and one offer structure.
- Begin supplier outreach and sample ordering.
- Draft landing page architecture and checkout flow.

## Can run in parallel (Weeks 1–3)

- Supplier vetting + commercial negotiation.
- Landing page build + demand capture setup.
- Support/FAQ/policy drafting.
- Phone-first onboarding asset production.

## Should be delayed until validation (post-first sales)

- Expanding SKUs.
- Building full custom mobile app.
- Formal multi-supplier balancing.
- Brand system refinements beyond conversion-critical assets.

---

## Risks, Unknowns, and Validation Needs

## Core assumptions to test quickly

- Non-technical users can complete setup from phone with current flow.
- A simple starter offer converts without heavy education.
- Target margin survives real shipping + support + refunds.
- Supplier reliability is sufficient for early reputation protection.

## Highest-risk unknowns

- Actual support burden per 100 orders.
- Defect/DOA rate from selected OEM.
- True CAC for first profitable orders.
- Return/refund drivers (expectation mismatch vs product quality).

## Fast validation methods

- Run small paid traffic tests against one landing page and one offer.
- Conduct 5–10 concierge onboarding sessions with early buyers.
- Instrument first-success completion step and support ticket reasons.
- Review weekly contribution margin by cohort/order batch.

---

## Immediate Next Actions (14–30 days)

## Days 1–3

- Lock v0 scope to one use case and one starter bundle.
- Draft pricing model with conservative assumptions.
- Build supplier outreach list and send first contact wave.

## Days 4–7

- Order samples from top candidates.
- Draft landing page with explicit value proposition and checkout CTA.
- Publish baseline policies and support contact path.

## Days 8–14

- Run sample testing against phone-first setup checklist.
- Produce quick-start guide and QR onboarding card.
- Finalize supplier choice for initial launch window.

## Days 15–21

- Launch acquisition experiments (small-budget ads and/or direct outbound content).
- Capture leads or orders through one clear funnel.
- Begin structured support logging from first user interactions.

## Days 22–30

- Review early performance:
  - Conversion.
  - Delivery performance.
  - Support load.
  - Margin reality.
- Decide one of three paths:
  1. **Proceed:** metrics are viable, scale cautiously.
  2. **Adjust:** fix onboarding/offer/pricing and rerun.
  3. **Pause/Pivot:** demand or economics do not support current thesis.

---

## Operating Heuristics for the Solo Founder

- Prefer reversible decisions when uncertain.
- Treat every process as “future template,” but only automate after repetition.
- Keep one source of truth for weekly metrics and decisions.
- Optimize for fewer SKUs, fewer workflows, fewer promises.
- Use real customer friction to set roadmap priority, not abstract feature ideas.
