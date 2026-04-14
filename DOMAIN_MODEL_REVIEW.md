# Diagram Suite for Playo DDD v7 — Recommendation

The workbook has 27 sheets covering strategic boundaries, tactical aggregates, cross-cutting behaviour, and operational quality. No single diagram covers all of that without becoming a wallpaper. The right answer is a **layered suite of ~15 diagram types**, organised so each layer answers one specific question and together they cover every sheet without redundancy.

I've organised them by the question they answer.

---

## Tier 1 — Strategic ("where are the lines drawn?")

These four exist before anything else. An architect joining the project should be productive after reading just these.

**D1 · Subdomain Heatmap**
*Scope:* the whole product, classified.
*Notation:* a single canvas split into Core / Supporting / Generic bands; each BC placed as a tile, sized by aggregate count, coloured by strategic value.
*Shows:* which contexts deserve A-team investment (Coordination, Recovery, Trust) vs which can be bought/outsourced (Identity-as-IDP, Financial-as-PSP-wrapper). Reveals the *strategic* shape that the Context Map alone hides.
*Source:* `05_Domain_Classification`.
*Audience:* leadership, hiring, build-vs-buy decisions.

**D2 · Bounded Context Map**
*Scope:* all 15 BCs and their inter-relationships.
*Notation:* classic Evans/Vernon context-map with named relationship arrows — Customer-Supplier (CS/U), Conformist (CF), Anti-Corruption Layer (ACL), Open Host Service (OHS), Published Language (PL), Partnership (P), Shared Kernel (SK).
*Shows:* upstream/downstream direction, where ACLs sit, which BCs publish a Published Language (events) vs which translate. Makes the asymmetry of power between BCs visible — e.g., Coordination is upstream of Trust (CS/U) but downstream of Inventory (CF behind ACL).
*Source:* `06_Context_Map`, `32_ACLs`.
*Audience:* every engineer, every PR review.

**D3 · Trust Submodel Constellation**
*Scope:* the 4 trust profiles + their use-case bindings.
*Notation:* hub-and-spoke with NO central hub — four independent profile circles (Skill, Reliability, Financial, Community), each linked to the *use-cases* that consume it (matchmaking, replacement search, BNPL eligibility, community gating). A red "X" overlay where a "TrustScore" composition would have lived, captioned with DG-1.
*Shows:* the architectural commitment of L4 + DG-1 made literal: the absence of composition is the whole point of the diagram.
*Source:* sheets `16-19_BC_Trust_*`, `00_Essence` L4/DG-1.
*Audience:* anyone tempted to add a `getReputation()` method.

**D4 · Ubiquitous Language Disambiguation Diagram**
*Scope:* the 6 most-confused nouns (Game, Session, Booking, Match, Seat, TimeSlot).
*Notation:* overlapping rings or a Venn-style layout with **explicit empty intersections** — captioned "Game ≠ Session", "Seat ≠ TimeSlot", "Booking ≠ Seat (1:1 not equal)". Each ring carries lifecycle phase markers.
*Shows:* exactly *why* each term is distinct, on one canvas. Resolves the most common onboarding confusion in 30 seconds.
*Source:* `00_Essence` §UBIQUITOUS LANGUAGE.
*Audience:* new joiners, product managers writing tickets.

---

## Tier 2 — Tactical ("how is each BC built?")

A template, instantiated once per BC. 15 instances, identical legend.

**D5 · Aggregate Constellation (template, ×15)**
*Scope:* one BC at a time.
*Notation:* aggregate roots as solid boxes, entities nested inside, VO references shown as thin pills attached to fields, commands listed on the left edge as arrows entering, events listed on the right edge as arrows leaving. Invariants in a footer band. Boundary box with the ACL gateways named on it.
*Shows:* the entire write-side of one BC on one page. Same legend across all 15 makes BC-to-BC comparison trivial.
*Source:* per-BC sheets §COMMANDS, §AGGREGATES, §ENTITIES, §VALUE OBJECTS, §EVENTS EMITTED, §INVARIANTS.
*Audience:* engineers implementing or reviewing that BC.

**D6 · State Machine (template, ×7 critical aggregates)**
*Scope:* lifecycle of one aggregate. Apply to: Game, Session, Booking, Payment, BNPLObligation, ReplacementCase, HostingSession.
*Notation:* state nodes as rounded rectangles, transitions labelled with the *Command* that triggers them and the *Event* that records them (so each arrow has both `cmd:` and `evt:` annotations). Terminal states shaded. Forbidden transitions explicitly drawn as red dashed arrows with the invariant they violate.
*Shows:* not just what *can* happen but what *cannot* — the impossible transitions are evidence of the invariants.
*Source:* per-BC §STATE MACHINE rows; cross-checked against §INVARIANTS.
*Audience:* implementers, QA writing state-coverage tests.

**D7 · Value Object Catalog Map**
*Scope:* all VOs and their reach across BCs.
*Notation:* matrix or sankey — VOs on one axis, BCs on the other; cells filled where the VO is referenced. Highlight VOs that are referenced by ≥3 BCs (those are the true "shared kernel" candidates and deserve extra design care).
*Shows:* which VOs are foundational (Money, TimeWindow, Geo) vs single-BC. Surfaces VO sprawl risk and validates DG-6.
*Source:* `04_Value_Objects` "Used By" column.
*Audience:* type-system stewards, API designers.

---

## Tier 3 — Cross-Cutting Behaviour ("how do BCs collaborate?")

These are the diagrams that make the per-BC sheets *cohere*.

**D8 · Event Storm Wall**
*Scope:* every domain event in the system, time-ordered left-to-right within a happy-path scenario, BCs on horizontal swim-lanes.
*Notation:* classic event-storm orange stickies for events, blue for commands, yellow for aggregates, lilac for policies. Multiple horizontal scenarios stacked: Game-to-Match, Booking-to-Refund, Replacement-Found, BNPL-Default.
*Shows:* the temporal *grain* of the system — how many events occur, in what order, across which BC boundaries. Reveals chattiness, missing events, and orphan commands.
*Source:* every per-BC §EVENTS EMITTED + §EVENTS CONSUMED, plus `30_Sagas`.
*Audience:* event-schema reviewers, observability/instrumentation team.

**D9 · Saga Choreography (template, ×10)**
*Scope:* one saga at a time. SAG-001..SAG-010.
*Notation:* sequence diagram with BCs as lifelines, but **no central orchestrator** — events flow between lifelines, with policy boxes shown as side-annotations where a decision is made. Compensation paths drawn in a parallel red lane below the happy path.
*Shows:* per-saga, the choreography vs orchestration choice for that flow, plus the explicit compensation routes. One diagram per saga so each is reviewable in isolation.
*Source:* `30_Sagas`.
*Audience:* implementers of cross-BC flows; SREs designing alerts.

**D10 · Recovery & Deviation Translation Diagram**
*Scope:* the L10 + DG-4 + DG-5 pattern, system-wide.
*Notation:* every BC on the left publishing `*DeviationRequested(reason)` events into a central Recovery context, which then publishes canonical `*Cancelled / *Failed / *NoShowed` events out the right side to consumers (Trust, Financial, Read Models). Reasons grouped by category.
*Shows:* the single-emitter rule made literal. Anyone who tries to emit `BookingCancelled` from Coordination is visibly violating the diagram.
*Source:* `11_BC_Recovery`, every other BC's §EVENTS EMITTED filtered to `*DeviationRequested`, `00_Essence` L10/DG-4/DG-5.
*Audience:* every BC owner — this is the diagram that prevents drift.

**D11 · Capacity & Money Twin-Track Diagram**
*Scope:* the L2 invariant — Session capacity counters and Payment lifecycle running on independent timelines, joined only by event subscription.
*Notation:* two parallel horizontal swim-lanes (Capacity track on top, Money track on bottom), each with its own state transitions; vertical dashed lines mark the events that *cross* between them (`SeatHeld`, `PaymentAuthorized`, `PaymentCaptured`, `SeatConfirmed`, `BookingDeviationRequested`). Show the *forbidden* synchronous arrow between tracks crossed out.
*Shows:* L2 visually — that capacity and money never block each other. The deadlock-prevention rationale becomes obvious.
*Source:* `10_BC_Coordination` (Session, Booking, Seat), `15_BC_Financial` (Payment), `00_Essence` L2.
*Audience:* anyone implementing the Booking saga; the most-violated lock in any e-commerce-adjacent system.

**D12 · Read Model Projection Map**
*Scope:* every read model and its event provenance.
*Notation:* events on the left, read models in the middle, query consumers on the right; arrows show which events feed which projection and which screens/APIs read which projection. Each projection annotated with eventual-consistency lag SLO.
*Shows:* the entire CQRS read side. Makes "where does this number on the screen come from?" answerable in one lookup.
*Source:* `33_Read_Models`, every per-BC §EVENTS EMITTED.
*Audience:* full-stack engineers, analytics, debugging staleness complaints.

**D13 · Policy Decision Diagram**
*Scope:* every Policy, what it observes, what it decides, what it never does.
*Notation:* policies as hexagons (decision-only), inputs as event arrows in, outputs as decision arrows out. A red-bordered "no" zone around each policy showing what it MUST NOT do (call services, mutate state) — DG-3 made visual.
*Shows:* the policy layer's purity at a glance. New policies get added to this diagram as a checklist.
*Source:* `31_Policies`, `00_Essence` L3/DG-3.
*Audience:* whoever is tempted to put a HTTP call in a policy.

---

## Tier 4 — Quality & Operations ("what happens when things break?")

**D14 · Failure Mode & Recovery Tree**
*Scope:* every entry in `41_Failure_Scenarios`.
*Notation:* fault tree — root-cause failures at the leaves, propagating up through the BCs they affect, with recovery sagas shown as the "cuts" that prevent propagation. Colour-coded by severity.
*Shows:* blast radius of each failure and which saga contains it. Doubles as an SRE runbook index.
*Source:* `41_Failure_Scenarios`, `30_Sagas` (compensation paths), `42_Pressure_Tests`.
*Audience:* SREs, on-call, post-mortem authors.

**D15 · Idempotency Boundary Map**
*Scope:* every operation requiring idempotency and the *dimension* on which uniqueness is enforced.
*Notation:* table-as-diagram — operations on rows, the (aggregateId, operation, idempotencyKey) tuple drawn as a 3-axis lock icon per row. Highlights operations where the key includes a time component (de-dup window) vs forever-unique.
*Shows:* the system's at-least-once-with-de-dup guarantees made explicit. Each external integration callback is on this map.
*Source:* `40_Idempotency_Concurrency`, every per-BC §IDEMPOTENCY.
*Audience:* anyone wiring up a webhook, retry, or queue consumer.

---

## What this suite covers — coverage check against the workbook

| Sheet | Covered by |
|---|---|
| 00_Essence (Locks/Guards/Language) | D3 (DG-1), D4 (Language), D10 (L10/DG-4/DG-5), D11 (L2), D13 (DG-3) |
| 04_Value_Objects | D7 |
| 05_Domain_Classification | D1 |
| 06_Context_Map | D2 |
| 09–23 BC sheets (×15) | D5 ×15, D6 ×7 (critical ones) |
| 30_Sagas | D8, D9 ×10 |
| 31_Policies | D13 |
| 32_ACLs | D2 (relationship arrows), D5 (boundary box) |
| 33_Read_Models | D12 |
| 40_Idempotency | D15 |
| 41_Failure_Scenarios | D14 |
| 42_Pressure_Tests | D14 (referenced as test cases for the tree) |
| 43_Changelog | not diagrammed (text artefact, doesn't need a picture) |

Every sheet except the changelog is reached by at least one diagram, and most are reached by two — once at strategic level, once at tactical.

---

## What I deliberately did NOT include and why

- **A single "everything" architecture diagram.** Inevitably becomes wallpaper. The Context Map (D2) plus Event Storm (D8) together do the same job at appropriate zoom levels.
- **Per-policy state diagrams.** Policies are stateless by L3 — a state diagram would itself violate the model.
- **Per-VO class diagrams.** VOs are too small to deserve their own picture; D7's catalog map is enough.
- **Sequence diagram per command.** Commands are 1:1 with aggregate methods (DG-7); D5 + D6 together already encode this. A per-command sequence would just be `actor → API → aggregate → event`, repeated.
- **Org/team topology diagram.** Out of scope for a domain model; belongs in a separate Team Topologies artefact.

---

## Recommended build order

If we're going to actually draw these, sequence matters. Build in order: **D2 → D1 → D4 → D5 (template) → D6 → D10 → D11 → D8 → D9 → D12 → D13 → D14 → D15 → D3 → D7**. Rationale: Context Map first because every later diagram references it; D10/D11 early because they're the diagrams that protect the most-violated rules; D9 sagas later because they assume D5/D6 are already drawn.

Want me to start drafting the actual visuals for any of these, or first finish the sheet-by-sheet review and then build the diagrams against the corrected workbook?
