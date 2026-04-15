# Playo DDD Diagram Suite - Quick Reference

## 📚 Complete Diagram Inventory

### Existing Diagrams (v7/v8)

| ID | Name | Location | Purpose |
|----|------|----------|---------|
| **D1** | Subdomain Heatmap | DIAGRAMS.md | Strategic classification (Core/Supporting/Generic) |
| **D2** | Bounded Context Map | DIAGRAMS.md | All 15 BCs + Evans/Vernon relationships |
| **D3** | Trust Constellation | DIAGRAMS.md | 4 trust profiles + DG-1 enforcement |
| **D4** | Language Disambiguation | DIAGRAMS.md | Game ≠ Session ≠ Booking ≠ Match |
| **D5** | Aggregate Constellation | DIAGRAMS.md | Per-BC write-side template |
| **D6** | State Machines | DIAGRAMS.md | Aggregate lifecycle (Session example) |
| **D7** | Value Object Catalog | DIAGRAMS.md | VO cross-BC usage matrix |
| **D8** | Event Storm Wall | DIAGRAMS.md | Game-to-Match happy path |
| **D9** | Booking Saga | DIAGRAMS.md | SAG-002 full sequence |
| **D10** | Recovery Deviation | DIAGRAMS.md | L10/DG-4/DG-5 pattern |
| **D11** | Twin Track Capacity | DIAGRAMS.md | L2 invariant enforcement |
| **D12** | Read Model Projection | DIAGRAMS.md | CQRS read side |
| **D13** | Policy Purity | DIAGRAMS.md | DG-3 enforcement |

### Enhanced Diagrams (New)

| ID | Name | Location | Purpose |
|----|------|----------|---------|
| **E1** | Evolution Timeline | ENHANCED_DIAGRAMS.md | v6→v8 journey |
| **E2** | Locked Decision Dependency | ENHANCED_DIAGRAMS.md | Guard→Lock enforcement |
| **E3** | Trust Decision Tree | ENHANCED_DIAGRAMS.md | Use-case decision logic |
| **E4** | Invariant Cross-Reference | ENHANCED_DIAGRAMS.md | Invariant dependencies |
| **E5** | Compensation Flow Matrix | ENHANCED_DIAGRAMS.md | Trust impact of compensations |
| **E6** | Service Block Topology | ENHANCED_DIAGRAMS.md | On-call boundaries |
| **E7** | Event Storming Big Picture | ENHANCED_DIAGRAMS.md | Multi-timeline flows |
| **E8** | Staleness SLO Matrix | ENHANCED_DIAGRAMS.md | Read model user impact |
| **E9** | Failure Blast Radius | ENHANCED_DIAGRAMS.md | Failure containment |
| **E10** | Idempotency Strategy | ENHANCED_DIAGRAMS.md | De-dup patterns |
| **E11** | PeerReview Sealing Window | ENHANCED_DIAGRAMS.md | L15 time logic |
| **E12** | Dispute Reversal Flow | ENHANCED_DIAGRAMS.md | Observation reversal |
| **E13** | VO Shared Kernel Heatmap | ENHANCED_DIAGRAMS.md | High-reuse VOs |
| **E14** | Orchestration vs. Choreography | ENHANCED_DIAGRAMS.md | Saga patterns |
| **E15** | Concurrency Strategy | ENHANCED_DIAGRAMS.md | OCC vs. locking |
| **E16** | Partition Strategy | ENHANCED_DIAGRAMS.md | Horizontal scaling |
| **E17** | Anti-Pattern Detection | ENHANCED_DIAGRAMS.md | Forbidden patterns |

---

## 🎯 Use Case → Diagram Mapping

### "I'm a new architect joining the team"
**Start here:**
1. E1 (Evolution Timeline) - Understand the journey
2. D2 (Context Map) - See all 15 BCs
3. E2 (Locked Decision Dependency) - Understand enforcement
4. E3 (Trust Decision Tree) - Understand the crown jewel
5. D1 (Subdomain Heatmap) - Understand strategic priorities

### "I'm implementing a new bounded context"
**Use these:**
1. D5 (Aggregate Constellation) - Template for BC structure
2. D6 (State Machines) - Template for aggregate lifecycle
3. E4 (Invariant Cross-Reference) - Understand invariant dependencies
4. E13 (VO Shared Kernel) - Identify high-reuse VOs

### "I'm designing a new saga"
**Use these:**
1. D9 (Booking Saga) - Example of orchestrated saga
2. E5 (Compensation Flow) - Understand trust impact
3. E14 (Orchestration vs. Choreography) - Choose pattern
4. E10 (Idempotency Strategy) - Design de-dup logic

### "I'm on-call and there's an incident"
**Use these:**
1. E9 (Failure Blast Radius) - Understand containment
2. E6 (Service Block Topology) - Identify owning team
3. D10 (Recovery Deviation) - Understand deviation flow
4. E8 (Staleness SLO) - Understand user impact

### "I'm scaling a bounded context"
**Use these:**
1. E16 (Partition Strategy) - Choose partition key
2. E15 (Concurrency Strategy) - Choose OCC vs. locking
3. E8 (Staleness SLO) - Understand read model impact
4. E6 (Service Block Topology) - Understand deployment boundaries

### "I'm reviewing code for architectural violations"
**Use these:**
1. E17 (Anti-Pattern Detection) - Check forbidden patterns
2. E2 (Locked Decision Dependency) - Verify guard enforcement
3. D3 (Trust Constellation) - Verify no composed score
4. D10 (Recovery Deviation) - Verify single emitter

---

## 🔑 Key Architectural Decisions

### L1: Recovery Owns ALL Deviations
- **Enforced by:** DG-4/5
- **Diagrams:** D10, E2, E5
- **Anti-pattern:** Aggregate emits *Cancelled directly
- **Detection:** E17

### L2: Capacity/Money Async Separation
- **Enforced by:** Saga choreography
- **Diagrams:** D11, E5
- **Anti-pattern:** Session blocks on payment
- **Detection:** E17

### L4: Trust Use-Case Binding
- **Enforced by:** DG-1
- **Diagrams:** D3, E3, E2
- **Anti-pattern:** getReputation(userId) without use_case
- **Detection:** E17

### L9: Booking 1:1 with Seat
- **Enforced by:** Unique constraint
- **Diagrams:** E4, D5
- **Anti-pattern:** Multiple bookings per seat
- **Detection:** Database constraint

### L15: PeerReview Sealing
- **Enforced by:** State machine
- **Diagrams:** E11, D6
- **Anti-pattern:** Content mutation after seal
- **Detection:** Immutability check

---

## 🚨 Common Violations & Fixes

### Violation 1: Composed TrustScore
**Symptom:** Column named `reputation` or `overall_trust`
**Diagram:** E3, E17
**Fix:** Delete column, use Trust Composition Policy with use_case parameter

### Violation 2: Host Mutates Coordination
**Symptom:** Host imports Coordination types
**Diagram:** E2, E17
**Fix:** Remove imports, emit capability events only

### Violation 3: Policy Writes to DB
**Symptom:** Repository injection in policy
**Diagram:** D13, E17
**Fix:** Return decision object, caller persists

### Violation 4: Aggregate Emits *Cancelled
**Symptom:** Event name ends with 'Cancelled' from non-Recovery
**Diagram:** D10, E2, E17
**Fix:** Emit *DeviationRequested, Recovery emits canonical

### Violation 5: Cache-Dependent Decision
**Symptom:** Decision breaks on cache miss
**Diagram:** E3, E17
**Fix:** Make cache optional, recompute on miss

---

## 📊 Metrics Dashboard

### Architectural Health
- **Trust Composition Violations:** Target 0 (E17)
- **Recovery Bypass Violations:** Target 0 (E17)
- **Policy Purity Violations:** Target 0 (E17)
- **Host Boundary Violations:** Target 0 (E17)

### Operational Health
- **Failure Containment Rate:** Target 95% (E9)
- **On-Call Response Time:** Target <10min (E6)
- **Staleness SLO Compliance:** Target 99% (E8)
- **Idempotency Violation Rate:** Target <0.1% (E10)

### Scalability Health
- **Partition Rebalance Frequency:** Target <1/month (E16)
- **Concurrency Conflict Rate:** Target <1% (E15)
- **Read Model Lag:** Target <SLO (E8)

---

## 🔄 Diagram Update Triggers

### Workbook Changes
- **Sheet added/removed:** Update E1, E2
- **Aggregate added:** Update D5, E4, E15, E16
- **Event added:** Update D8, E7, E12
- **Policy added:** Update D13, E3
- **Saga added:** Update E5, E14

### Operational Changes
- **Team added:** Update E6
- **Failure mode discovered:** Update E9
- **SLO changed:** Update E8
- **On-call rotation changed:** Update E6

### Code Changes
- **Anti-pattern detected:** Update E17
- **Concurrency strategy changed:** Update E15
- **Partition key changed:** Update E16
- **Idempotency strategy changed:** Update E10

---

## 🎓 Learning Path

### Week 1: Strategic Foundation
- [ ] Read E1 (Evolution Timeline)
- [ ] Read E2 (Locked Decision Dependency)
- [ ] Read E3 (Trust Decision Tree)
- [ ] Read D1 (Subdomain Heatmap)
- [ ] Read D2 (Context Map)

### Week 2: Tactical Details
- [ ] Read D5 (Aggregate Constellation)
- [ ] Read D6 (State Machines)
- [ ] Read D9 (Booking Saga)
- [ ] Read D11 (Twin Track)
- [ ] Read E5 (Compensation Flow)

### Week 3: Operational Reality
- [ ] Read E6 (Service Block Topology)
- [ ] Read E9 (Failure Blast Radius)
- [ ] Read E8 (Staleness SLO)
- [ ] Read D10 (Recovery Deviation)
- [ ] Read E12 (Dispute Reversal)

### Week 4: Scalability & Anti-Patterns
- [ ] Read E15 (Concurrency Strategy)
- [ ] Read E16 (Partition Strategy)
- [ ] Read E10 (Idempotency Strategy)
- [ ] Read E17 (Anti-Pattern Detection)
- [ ] Read E14 (Orchestration vs. Choreography)

---

## 🔗 Cross-References

### Trust Model
- **Strategic:** D1, D3, E1, E2
- **Tactical:** E3, E4
- **Operational:** E17
- **Scalability:** E15, E16

### Recovery Context
- **Strategic:** D1, D2, E1, E2
- **Tactical:** D10, E5, E12
- **Operational:** E6, E9
- **Scalability:** E15

### Saga Patterns
- **Strategic:** D2, E1
- **Tactical:** D9, E5, E14
- **Operational:** E9
- **Scalability:** E10

### Service Blocks
- **Strategic:** E1, E6
- **Tactical:** D2
- **Operational:** E6, E9
- **Scalability:** E16

---

## 📞 Who to Ask

### Strategic Questions (L1-L8)
- **Contact:** Architecture team
- **Diagrams:** E1, E2, E3
- **Examples:** "Why did we split Trust?" "Why is Recovery the single emitter?"

### Tactical Questions (Aggregates, Sagas)
- **Contact:** Domain team leads
- **Diagrams:** D5, D6, D9, E5
- **Examples:** "How do I design a new aggregate?" "What's the compensation logic?"

### Operational Questions (On-call, Failures)
- **Contact:** SRE team
- **Diagrams:** E6, E9, E8
- **Examples:** "Who's on-call for this BC?" "What's the blast radius?"

### Scalability Questions (Partitioning, Concurrency)
- **Contact:** Platform team
- **Diagrams:** E15, E16, E10
- **Examples:** "How do I partition this aggregate?" "Should I use OCC or locking?"

---

## 🛠️ Tools & Automation

### Diagram Generation
- **Tool:** Mermaid CLI
- **Source:** Playo_DDD_v8.xlsx
- **Output:** DIAGRAMS.md, ENHANCED_DIAGRAMS.md
- **Trigger:** Workbook commit

### Anti-Pattern Detection
- **Tool:** ArchUnit / NetArchTest
- **Source:** E17 (Anti-Pattern Detection)
- **Output:** CI/CD lint report
- **Trigger:** Pull request

### Metrics Dashboard
- **Tool:** Grafana
- **Source:** E8 (Staleness SLO), E9 (Failure Blast Radius)
- **Output:** Real-time dashboard
- **Trigger:** Continuous

### Onboarding Checklist
- **Tool:** Confluence / Notion
- **Source:** Learning Path (above)
- **Output:** Onboarding checklist
- **Trigger:** New hire

---

## 📝 Quick Tips

### For Architects
- Start with E1-E3 for strategic context
- Use E2 to understand enforcement relationships
- Use E17 to prevent violations

### For Developers
- Use D5-D6 as templates for new aggregates
- Use E10 for idempotency patterns
- Use E15 for concurrency patterns

### For SREs
- Use E9 for failure response
- Use E6 for on-call boundaries
- Use E8 for monitoring SLOs

### For Product Managers
- Use D1 for strategic priorities
- Use D4 for ubiquitous language
- Use E1 for evolution context

---

## 🎯 Success Metrics

### Onboarding
- **Before:** 2-3 weeks to productivity
- **After:** 1 week with E1-E3
- **Measure:** Time to first PR

### Architecture Violations
- **Before:** Discovered in code review
- **After:** Caught in CI/CD with E17
- **Measure:** Violations per sprint

### Incident Response
- **Before:** 30-60 min to understand blast radius
- **After:** 5-10 min with E9
- **Measure:** MTTR (Mean Time To Resolve)

### Scalability Planning
- **Before:** Ad-hoc decisions
- **After:** Pattern-based with E15-E16
- **Measure:** Rebalance frequency

