# Playo DDD Diagram Suite - Complete Guide

## 📖 Overview

This repository contains a comprehensive diagram suite for the Playo Domain-Driven Design (DDD) model, covering strategic, tactical, and operational perspectives across 4 major versions (v6 → v6.1 → v7 → v8).

**Total Diagrams:** 30 (13 existing + 17 enhanced)

---

## 📂 File Structure

```
game-ddd/
├── DIAGRAMS.md                          # Existing v7 diagrams (D1-D13)
├── Playo_DDD_v8_Diagrams.md            # v8 diagrams with operational detail
├── ENHANCED_DIAGRAMS.md                 # New enhanced diagrams (E1-E17)
├── DIAGRAM_RECOMMENDATIONS_SUMMARY.md   # Executive summary & recommendations
├── QUICK_REFERENCE.md                   # Quick lookup guide
├── README_DIAGRAMS.md                   # This file
├── DOMAIN_MODEL_REVIEW.md               # Original diagram recommendations
└── Playo_DDD_v8.xlsx                    # Source of truth workbook
```

---

## 🎯 Quick Start

### For New Architects
**Goal:** Understand the domain model in 1 week

**Day 1-2: Strategic Context**
1. Read [E1: Evolution Timeline](ENHANCED_DIAGRAMS.md#e1--domain-model-evolution-timeline) - Understand the journey
2. Read [E2: Locked Decision Dependency](ENHANCED_DIAGRAMS.md#e2--locked-decision-dependency-graph) - Understand enforcement
3. Read [E3: Trust Decision Tree](ENHANCED_DIAGRAMS.md#e3--trust-composition-decision-tree-enhanced) - Understand the crown jewel

**Day 3-4: Tactical Details**
4. Read [D2: Context Map](DIAGRAMS.md#d2--bounded-context-map-evansvernon-relationships) - See all 15 BCs
5. Read [D5: Aggregate Constellation](DIAGRAMS.md#d5--aggregate-constellation-template-coordination-bc-example) - Understand BC structure
6. Read [D9: Booking Saga](DIAGRAMS.md#d9--booking-saga-choreography-sag-002) - Understand saga patterns

**Day 5: Operational Reality**
7. Read [E6: Service Block Topology](ENHANCED_DIAGRAMS.md#e6--service-block-on-call-topology-enhanced) - Understand on-call boundaries
8. Read [E9: Failure Blast Radius](ENHANCED_DIAGRAMS.md#e9--failure-mode-blast-radius-diagram) - Understand failure containment

### For Developers
**Goal:** Implement a new feature without violating architectural principles

**Before coding:**
1. Check [E17: Anti-Pattern Detection](ENHANCED_DIAGRAMS.md#e17--anti-pattern-detection-checklist) - Avoid forbidden patterns
2. Check [D5: Aggregate Constellation](DIAGRAMS.md#d5--aggregate-constellation-template-coordination-bc-example) - Use template
3. Check [E10: Idempotency Strategy](ENHANCED_DIAGRAMS.md#e10--idempotency-key-strategy-diagram) - Design de-dup logic

**During coding:**
4. Check [E15: Concurrency Strategy](ENHANCED_DIAGRAMS.md#e15--concurrency-strategy-per-aggregate) - Choose OCC vs. locking
5. Check [E3: Trust Decision Tree](ENHANCED_DIAGRAMS.md#e3--trust-composition-decision-tree-enhanced) - Use trust correctly

**After coding:**
6. Check [E17: Anti-Pattern Detection](ENHANCED_DIAGRAMS.md#e17--anti-pattern-detection-checklist) - Verify no violations

### For SREs
**Goal:** Respond to incidents quickly

**During incident:**
1. Check [E9: Failure Blast Radius](ENHANCED_DIAGRAMS.md#e9--failure-mode-blast-radius-diagram) - Understand containment
2. Check [E6: Service Block Topology](ENHANCED_DIAGRAMS.md#e6--service-block-on-call-topology-enhanced) - Identify owning team
3. Check [D10: Recovery Deviation](DIAGRAMS.md#d10--recovery--deviation-translation-pattern-l10dg-4dg-5) - Understand deviation flow

**After incident:**
4. Update [E9: Failure Blast Radius](ENHANCED_DIAGRAMS.md#e9--failure-mode-blast-radius-diagram) - Add new failure mode
5. Update [E8: Staleness SLO](ENHANCED_DIAGRAMS.md#e8--read-model-staleness-slo-matrix) - Adjust SLOs if needed

---

## 📊 Diagram Categories

### Strategic (Where are the lines drawn?)
- **D1:** Subdomain Heatmap - Core/Supporting/Generic classification
- **D2:** Bounded Context Map - All 15 BCs + relationships
- **D3:** Trust Constellation - 4 profiles + DG-1 enforcement
- **D4:** Language Disambiguation - Game ≠ Session ≠ Booking
- **E1:** Evolution Timeline - v6→v8 journey
- **E2:** Locked Decision Dependency - Guard→Lock enforcement
- **E6:** Service Block Topology - On-call boundaries

### Tactical (How is each BC built?)
- **D5:** Aggregate Constellation - Per-BC write-side template
- **D6:** State Machines - Aggregate lifecycle
- **D7:** Value Object Catalog - VO cross-BC usage
- **E3:** Trust Decision Tree - Use-case decision logic
- **E4:** Invariant Cross-Reference - Invariant dependencies
- **E13:** VO Shared Kernel Heatmap - High-reuse VOs

### Cross-Cutting (How do BCs collaborate?)
- **D8:** Event Storm Wall - Game-to-Match happy path
- **D9:** Booking Saga - SAG-002 full sequence
- **D10:** Recovery Deviation - L10/DG-4/DG-5 pattern
- **D11:** Twin Track Capacity - L2 invariant
- **D12:** Read Model Projection - CQRS read side
- **D13:** Policy Purity - DG-3 enforcement
- **E5:** Compensation Flow Matrix - Trust impact
- **E7:** Event Storming Big Picture - Multi-timeline flows
- **E12:** Dispute Reversal Flow - Observation reversal
- **E14:** Orchestration vs. Choreography - Saga patterns

### Operational (What happens when things break?)
- **E8:** Staleness SLO Matrix - Read model user impact
- **E9:** Failure Blast Radius - Failure containment
- **E11:** PeerReview Sealing Window - L15 time logic

### Scalability (How does it scale?)
- **E10:** Idempotency Strategy - De-dup patterns
- **E15:** Concurrency Strategy - OCC vs. locking
- **E16:** Partition Strategy - Horizontal scaling

### Quality (How do we prevent drift?)
- **E17:** Anti-Pattern Detection - Forbidden patterns

---

## 🔑 Key Architectural Decisions

### The Big 4 (Most Critical)

#### 1. Trust Split (v6.1) - The Crown Jewel
**Decision:** Split Trust from monolithic concept into 4 independent profiles

**Diagrams:** D3, E1, E2, E3

**Why it matters:**
- Prevents "reputation score" anti-pattern
- Forces explicit use-case binding
- Enables independent evolution

**Enforcement:** DG-1 (Trust Composition Purity)

#### 2. Recovery Single Emitter (v6)
**Decision:** Recovery owns ALL deviations, emits canonical failure events

**Diagrams:** D10, E1, E2, E5

**Why it matters:**
- Clean separation of concerns
- Single source of truth for failures
- Consistent compensation logic

**Enforcement:** L1, DG-4/5 (Deviation Translation)

#### 3. Capacity/Money Async (v6)
**Decision:** Session owns capacity, Booking owns financial commitment, ASYNC separation

**Diagrams:** D11, E1, E2, E5

**Why it matters:**
- Prevents deadlocks
- Enables independent scaling
- Enforced through saga choreography

**Enforcement:** L2

#### 4. Policy Purity (v6)
**Decision:** Policies are stateless, no orchestration, no side effects

**Diagrams:** D13, E1, E2, E17

**Why it matters:**
- Enables versioning and A/B testing
- Prevents god layer
- Enables independent deployment

**Enforcement:** L3, DG-3

---

## 📈 Evolution Journey

### v6 Foundation (2024 Q1)
**Focus:** Establish core principles

**Key Decisions:**
- L1-L8 locked decisions
- 10 bounded contexts
- Monolithic Trust (recognized as problematic)

**Diagrams:** Foundation for all current diagrams

### v6.1 Tactical Refinement (2024 Q2)
**Focus:** Trust split + Design Guards

**Key Changes:**
- Trust → 4 independent profiles (Skill, Reliability, Financial, Community)
- DG-1 to DG-7 formalized
- Cell demoted from aggregate to projection

**Diagrams:** D3, E3 show the breakthrough

### v7 Strategic Expansion (2024 Q3)
**Focus:** Add Gamification, Community, Training

**Key Changes:**
- 14 bounded contexts (from 10)
- Matchmaking becomes core domain
- Dispute resolution formalized
- 13 sagas defined

**Diagrams:** D1-D13 implemented

### v8 Operational Maturity (2024 Q4)
**Focus:** Service Blocks + Scalability

**Key Changes:**
- 15 bounded contexts (added SubsidyLedger)
- Service Blocks for team topology
- L16-L19 (Idempotency, Concurrency, Partition strategies)
- Intent/Attempt/Payment split in Financial

**Diagrams:** E1-E17 add operational detail

---

## 🚨 Common Pitfalls & Solutions

### Pitfall 1: Composed TrustScore
**Symptom:** Column named `reputation` or `overall_trust`

**Why it's wrong:** Violates L4 + DG-1

**Solution:** Delete column, use Trust Composition Policy with use_case parameter

**Diagrams:** E3, E17

### Pitfall 2: Host Mutates Coordination
**Symptom:** Host imports Coordination types

**Why it's wrong:** Violates L5 + DG-2

**Solution:** Remove imports, emit capability events only

**Diagrams:** E2, E17

### Pitfall 3: Policy Writes to DB
**Symptom:** Repository injection in policy

**Why it's wrong:** Violates L3 + DG-3

**Solution:** Return decision object, caller persists

**Diagrams:** D13, E17

### Pitfall 4: Aggregate Emits *Cancelled
**Symptom:** Event name ends with 'Cancelled' from non-Recovery

**Why it's wrong:** Violates L1 + DG-4/5

**Solution:** Emit *DeviationRequested, Recovery emits canonical

**Diagrams:** D10, E2, E17

### Pitfall 5: Synchronous Capacity/Money
**Symptom:** Session blocks waiting for payment

**Why it's wrong:** Violates L2

**Solution:** Use saga choreography, async separation

**Diagrams:** D11, E5

---

## 🎓 Learning Resources

### Recommended Reading Order

**Week 1: Strategic Foundation**
1. [E1: Evolution Timeline](ENHANCED_DIAGRAMS.md#e1--domain-model-evolution-timeline)
2. [E2: Locked Decision Dependency](ENHANCED_DIAGRAMS.md#e2--locked-decision-dependency-graph)
3. [E3: Trust Decision Tree](ENHANCED_DIAGRAMS.md#e3--trust-composition-decision-tree-enhanced)
4. [D1: Subdomain Heatmap](DIAGRAMS.md#d1--subdomain-heatmap-strategic-classification)
5. [D2: Context Map](DIAGRAMS.md#d2--bounded-context-map-evansvernon-relationships)

**Week 2: Tactical Details**
6. [D5: Aggregate Constellation](DIAGRAMS.md#d5--aggregate-constellation-template-coordination-bc-example)
7. [D6: State Machines](DIAGRAMS.md#d6--state-machine-template-session-aggregate-example)
8. [D9: Booking Saga](DIAGRAMS.md#d9--booking-saga-choreography-sag-002)
9. [D11: Twin Track](DIAGRAMS.md#d11--capacity--money-twin-track-l2-invariant)
10. [E5: Compensation Flow](ENHANCED_DIAGRAMS.md#e5--saga-compensation-flow-matrix)

**Week 3: Operational Reality**
11. [E6: Service Block Topology](ENHANCED_DIAGRAMS.md#e6--service-block-on-call-topology-enhanced)
12. [E9: Failure Blast Radius](ENHANCED_DIAGRAMS.md#e9--failure-mode-blast-radius-diagram)
13. [E8: Staleness SLO](ENHANCED_DIAGRAMS.md#e8--read-model-staleness-slo-matrix)
14. [D10: Recovery Deviation](DIAGRAMS.md#d10--recovery--deviation-translation-pattern-l10dg-4dg-5)
15. [E12: Dispute Reversal](ENHANCED_DIAGRAMS.md#e12--dispute-resolution-reversal-flow)

**Week 4: Scalability & Anti-Patterns**
16. [E15: Concurrency Strategy](ENHANCED_DIAGRAMS.md#e15--concurrency-strategy-per-aggregate)
17. [E16: Partition Strategy](ENHANCED_DIAGRAMS.md#e16--partition-key-strategy-for-scalability)
18. [E10: Idempotency Strategy](ENHANCED_DIAGRAMS.md#e10--idempotency-key-strategy-diagram)
19. [E17: Anti-Pattern Detection](ENHANCED_DIAGRAMS.md#e17--anti-pattern-detection-checklist)
20. [E14: Orchestration vs. Choreography](ENHANCED_DIAGRAMS.md#e14--saga-orchestration-vs-choreography-decision-matrix)

### External Resources
- **Domain-Driven Design:** Eric Evans, "Domain-Driven Design: Tackling Complexity in the Heart of Software"
- **Event Sourcing:** Martin Fowler, "Event Sourcing" (martinfowler.com)
- **Saga Patterns:** Chris Richardson, "Microservices Patterns" (microservices.io)
- **CQRS:** Greg Young, "CQRS Documents" (cqrs.files.wordpress.com)

---

## 🛠️ Tools & Automation

### Diagram Generation
```bash
# Generate diagrams from workbook
npm run generate-diagrams

# Validate diagram syntax
npm run validate-diagrams

# Export to PNG/SVG
npm run export-diagrams
```

### Anti-Pattern Detection
```bash
# Run architecture tests
npm run arch-test

# Check for DG-1 violations (Trust composition)
npm run check-trust-purity

# Check for DG-4/5 violations (Recovery single emitter)
npm run check-recovery-emitter
```

### Metrics Dashboard
```bash
# Start Grafana dashboard
docker-compose up grafana

# View at http://localhost:3000
# Dashboards: Staleness SLO, Failure Blast Radius, Architectural Health
```

---

## 📞 Support & Contribution

### Questions?
- **Strategic questions (L1-L8):** Architecture team (#architecture-team)
- **Tactical questions (Aggregates, Sagas):** Domain team leads (#domain-leads)
- **Operational questions (On-call, Failures):** SRE team (#sre-team)
- **Scalability questions (Partitioning, Concurrency):** Platform team (#platform-team)

### Found an issue?
1. Check [E17: Anti-Pattern Detection](ENHANCED_DIAGRAMS.md#e17--anti-pattern-detection-checklist)
2. Check [QUICK_REFERENCE.md](QUICK_REFERENCE.md) for common violations
3. If still unclear, create an issue in the repo

### Want to contribute?
1. Read [DIAGRAM_RECOMMENDATIONS_SUMMARY.md](DIAGRAM_RECOMMENDATIONS_SUMMARY.md)
2. Follow the diagram templates in [ENHANCED_DIAGRAMS.md](ENHANCED_DIAGRAMS.md)
3. Submit a PR with updated diagrams

---

## 📝 Changelog

### 2024-04-15: Enhanced Diagram Suite (E1-E17)
- Added 17 new enhanced diagrams
- Created comprehensive documentation
- Added quick reference guide

### 2024-Q4: v8 Operational Maturity
- Added Service Blocks
- Added Idempotency/Concurrency/Partition strategies
- Added Intent/Attempt/Payment split

### 2024-Q3: v7 Strategic Expansion
- Added Gamification, Community, Training BCs
- Added Matchmaking as core domain
- Implemented D1-D13 diagram suite

### 2024-Q2: v6.1 Tactical Refinement
- Split Trust into 4 profiles
- Formalized Design Guards (DG-1 to DG-7)
- Demoted Cell to projection

### 2024-Q1: v6 Foundation
- Established L1-L8 locked decisions
- Defined 10 bounded contexts
- Created initial domain model

---

## 🎯 Success Metrics

### Onboarding
- **Target:** 1 week to productivity (down from 2-3 weeks)
- **Measure:** Time to first PR
- **Diagrams:** E1-E3

### Architecture Violations
- **Target:** 0 violations per sprint
- **Measure:** CI/CD lint report
- **Diagrams:** E17

### Incident Response
- **Target:** <10 min MTTR
- **Measure:** Time to understand blast radius
- **Diagrams:** E9, E6

### Scalability
- **Target:** <1 rebalance per month
- **Measure:** Partition rebalance frequency
- **Diagrams:** E16

---

## 📚 Additional Resources

- **[DIAGRAMS.md](DIAGRAMS.md)** - Existing v7 diagrams (D1-D13)
- **[ENHANCED_DIAGRAMS.md](ENHANCED_DIAGRAMS.md)** - New enhanced diagrams (E1-E17)
- **[DIAGRAM_RECOMMENDATIONS_SUMMARY.md](DIAGRAM_RECOMMENDATIONS_SUMMARY.md)** - Executive summary
- **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** - Quick lookup guide
- **[DOMAIN_MODEL_REVIEW.md](DOMAIN_MODEL_REVIEW.md)** - Original recommendations
- **[Playo_DDD_v8.xlsx](Playo_DDD_v8.xlsx)** - Source of truth workbook

---

**Last Updated:** 2024-04-15  
**Version:** 1.0  
**Maintainer:** Architecture Team

