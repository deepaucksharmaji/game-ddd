# Playo DDD Enhanced Diagram Suite

**Purpose:** This document extends the existing v7 and v8 diagrams with deeper architectural insights derived from understanding the domain model evolution from v6 → v6.1 → v7 → v8.

**Key Evolution Insights:**
- v6: Foundation (8 Locked Decisions, 10 BCs)
- v6.1: Trust split into 4 profiles (DG-1 enforcement)
- v7: Strategic expansion (Gamification, Community, Training)
- v8: Operational maturity (Service Blocks, Idempotency strategies)

---

## E1 · Domain Model Evolution Timeline

**Answers:** How did we get here? What were the key architectural breakthroughs?
**Why it matters:** Prevents regression to anti-patterns (like monolithic Trust). Documents the learning journey.

```mermaid
timeline
    title Domain Model Evolution Journey
    
    section v6 Foundation (2024 Q1)
        8 Locked Decisions : L1-L8 established core principles
        10 Bounded Contexts : Core/Supporting/Generic classification
        Monolithic Trust : Single TrustScore concept (recognized as problematic)
        Recovery Single Emitter : L1 locked - Recovery owns ALL deviations
        Capacity/Money Split : L2 locked - Async separation prevents deadlocks
        
    section v6.1 Tactical Refinement (2024 Q2)
        Trust Breakthrough : Split into 4 independent profiles (Skill, Reliability, Financial, Community)
        Design Guards : DG-1 to DG-7 formalized to enforce locked decisions
        Cell Demotion : Cell demoted from aggregate to projection (no invariants)
        Host Boundary : DG-2 enforces Host as capability provider only
        Policy Purity : DG-3 enforces stateless decision functions
        
    section v7 Strategic Expansion (2024 Q3)
        14 Bounded Contexts : Added Gamification, Community, Training, Matchmaking
        Matchmaking Core : Skill-based team balancing becomes core domain
        Dispute Resolution : Formalized with observation reversal capability
        13 Sagas : Complete choreography coverage
        Trust Composition : Explicit use-case bindings per decision
        
    section v8 Operational Maturity (2024 Q4)
        15 Bounded Contexts : Added SubsidyLedger within Recovery
        Service Blocks : Operational grouping for team topology
        Idempotency Strategies : L16-L19 per-aggregate guarantees
        Intent/Attempt/Payment : Financial split enables PSP failover
        Failure Blast Radius : DG-19 containment strategies
```

---

## E2 · Locked Decision Dependency Graph

**Answers:** Which design guards enforce which locked decisions? What's the dependency structure?
**Why it matters:** Shows that guards aren't arbitrary - they're enforcement mechanisms for architectural principles.

```mermaid
flowchart TD
    subgraph FOUNDATION["L1-L8: Foundation Locked Decisions"]
        L1["L1: Recovery owns ALL deviations
        Single emitter of canonical failure events"]
        L2["L2: Session owns capacity
        Booking owns financial commitment
        ASYNC separation"]
        L3["L3: Policies are stateless
        No orchestration, no side effects"]
        L4["L4: TrustScore = f(use_case, profiles)
        Use-case binding MANDATORY"]
        L5["L5: Host provides capabilities
        Coordination owns assembly"]
        L6["L6: Inventory = physical truth
        Partner = contractual truth"]
        L7["L7: Build order
        Events → Aggregates → Policies → Workflows"]
        L8["L8: Sheet = Layer
        Physical separation mirrors logical"]
    end

    subgraph GUARDS["DG-1 to DG-7: Enforcement Guards"]
        DG1["DG-1: Trust Composition Purity
        ❌ NO persisted composed score
        ❌ NO cache-dependent decisions
        ✅ Use-case parameter REQUIRED"]
        
        DG2["DG-2: Host Boundary
        ❌ NO coordination state mutation
        ❌ NO workflow triggering
        ✅ Capability events ONLY"]
        
        DG3["DG-3: Policy Purity
        ❌ NO database writes
        ❌ NO network calls
        ❌ NO event emission
        ✅ Pure decision functions"]
        
        DG45["DG-4/5: Deviation Translation
        Aggregates emit *DeviationRequested
        Recovery emits canonical *Cancelled"]
        
        DG6["DG-6: Value Object Immutability
        VOs are immutable after creation"]
        
        DG7["DG-7: Aggregate Operation Isolation
        Commands 1:1 with aggregate methods"]
    end

    subgraph OPERATIONAL["L9-L19: Operational Constraints"]
        L9["L9: Booking 1:1 with Seat
        Strict cardinality"]
        L10["L10: Recovery single emitter
        Canonical failure events"]
        L14["L14: Replacement filter
        skill ∧ geo ∧ reliability"]
        L15["L15: PeerReview sealing
        Reciprocal OR 14d"]
        L16["L16: Idempotency strategy
        Per aggregate"]
        L19["L19: Service block isolation
        No aggregate sharing"]
    end

    %% Enforcement relationships
    L4 -->|enforced by| DG1
    L3 -->|enforced by| DG3
    L1 -->|enforced by| DG45
    L5 -->|enforced by| DG2
    
    %% Operational builds on foundation
    L1 --> L10
    L2 --> L9
    
    %% Guards enable operational
    DG1 -.->|enables| L14
    DG3 -.->|enables| L15

    style L1 fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:3px
    style L4 fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:3px
    style DG1 fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style DG45 fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style DG3 fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
```

---

## E3 · Trust Composition Decision Tree (Enhanced)

**Answers:** How do different use cases compose different profile combinations? What's the decision logic?
**Why it matters:** Makes L4 + DG-1 concrete. Shows that trust is NEVER a single number.

```mermaid
flowchart TD
    subgraph PROFILES["4 Trust Profiles · Append-Only · Never Composed"]
        SKILL["SkillProfile
        📊 mu/sigma per sport
        Source: MatchCompleted, PeerReviewRevealed"]
        REL["ReliabilityProfile
        📈 attendance rate, sample size
        Source: CheckInRecorded, NoShowCaseOpened"]
        FIN["FinancialTrustProfile
        💰 payment discipline, BNPL rate
        Source: PaymentCaptured, BNPLDefaulted"]
        COMM["CommunityStanding
        👥 peer aggregate, connections
        Source: PeerReviewRevealed, PlayPalConfirmed"]
    end

    subgraph UC1["Use Case 1: Matchmaking Eligibility"]
        MM_INPUT["Inputs: Skill + Reliability"]
        MM_LOGIC{"Decision Logic
        skill.mu in range?
        reliability.rate > threshold?"}
        MM_OUTPUT["Output: Eligible / Ineligible
        + confidence score"]
    end

    subgraph UC2["Use Case 2: BNPL Eligibility"]
        BNPL_INPUT["Inputs: Financial + Reliability"]
        BNPL_LOGIC{"Decision Logic
        payment discipline > 0.9?
        BNPL default rate < 0.05?
        reliability.rate > 0.8?"}
        BNPL_OUTPUT["Output: Allow / Deny / RequireDeposit
        + credit limit"]
    end

    subgraph UC3["Use Case 3: Replacement Candidacy"]
        REPL_INPUT["Inputs: Skill + Reliability + Geo"]
        REPL_LOGIC{"Decision Logic
        skill match?
        reliability > 0.85?
        geo distance < 5km?
        no conflict with session?"}
        REPL_OUTPUT["Output: Ranked candidate list
        + subsidy eligibility"]
    end

    subgraph UC4["Use Case 4: Host Delegation"]
        HOST_INPUT["Inputs: Reliability + Community"]
        HOST_LOGIC{"Decision Logic
        reliability > 0.9?
        community standing > threshold?
        completion history > 20?"}
        HOST_OUTPUT["Output: Qualified / NotQualified
        + delegation level"]
    end

    subgraph UC5["Use Case 5: Review Display"]
        DISP_INPUT["Inputs: Community + Skill"]
        DISP_LOGIC{"Decision Logic
        peer aggregate credible?
        skill verified?
        dispute history?"}
        DISP_OUTPUT["Output: Display / Hide / Annotate
        + credibility badge"]
    end

    SKILL --> MM_INPUT
    REL --> MM_INPUT
    MM_INPUT --> MM_LOGIC
    MM_LOGIC --> MM_OUTPUT

    FIN --> BNPL_INPUT
    REL --> BNPL_INPUT
    BNPL_INPUT --> BNPL_LOGIC
    BNPL_LOGIC --> BNPL_OUTPUT

    SKILL --> REPL_INPUT
    REL --> REPL_INPUT
    REPL_INPUT --> REPL_LOGIC
    REPL_LOGIC --> REPL_OUTPUT

    REL --> HOST_INPUT
    COMM --> HOST_INPUT
    HOST_INPUT --> HOST_LOGIC
    HOST_LOGIC --> HOST_OUTPUT

    COMM --> DISP_INPUT
    SKILL --> DISP_INPUT
    DISP_INPUT --> DISP_LOGIC
    DISP_LOGIC --> DISP_OUTPUT

    FORBIDDEN["❌ FORBIDDEN
    NO getReputation(userId)
    NO single composed score
    NO cache-dependent decisions
    DG-1 VIOLATION"]

    SKILL -.->|❌| FORBIDDEN
    REL -.->|❌| FORBIDDEN
    FIN -.->|❌| FORBIDDEN
    COMM -.->|❌| FORBIDDEN

    style PROFILES fill:#F0AD4E,stroke:#8A5A12,stroke-width:2px
    style FORBIDDEN fill:#ef4444,color:white,stroke:#7A1F1C,stroke-width:3px
    style MM_LOGIC fill:#5BC0DE,stroke:#1F5A73
    style BNPL_LOGIC fill:#5BC0DE,stroke:#1F5A73
    style REPL_LOGIC fill:#5BC0DE,stroke:#1F5A73
    style HOST_LOGIC fill:#5BC0DE,stroke:#1F5A73
    style DISP_LOGIC fill:#5BC0DE,stroke:#1F5A73
```

---

## E4 · Aggregate Invariant Cross-Reference Matrix

**Answers:** Which invariants are shared? Which locked decisions enforce which invariants?
**Why it matters:** Shows the "blast radius" of invariant violations.

```mermaid
flowchart LR
    subgraph INVARIANTS["Critical Invariants"]
        INV1["INV-COR-002
        held + confirmed ≤ capacity
        NEVER violated"]
        INV2["INV-COR-006
        Booking 1:1 with Seat
        unique(session, user)"]
        INV3["INV-COR-004
        Match immutable after completion
        Write-once"]
        INV4["INV-COM-005
        PeerReview immutable after Seal
        Content locked"]
        INV5["INV-TRUST-001
        No composed TrustScore
        Use-case binding required"]
        INV6["INV-REC-003
        Recovery single emitter
        Canonical events only"]
        INV7["INV-FIN-003
        Intent → Attempt → Payment
        Retry without losing Intent"]
    end

    subgraph AGGREGATES["Aggregates Enforcing"]
        SESSION["Session
        Atomic counters + OCC"]
        BOOKING["Booking
        Unique constraint"]
        MATCH["Match
        Immutable after event"]
        PEER["PeerReview
        Sealed state"]
        TRUST["Trust Profiles
        Append-only"]
        RECOVERY["Recovery Cases
        Deviation lifecycle"]
        PAYMENT["PaymentIntent
        State machine"]
    end

    subgraph LOCKED["Enforced By Locked Decisions"]
        L2["L2: Capacity/Money async
        Prevents deadlocks"]
        L9["L9: Booking 1:1 Seat
        Strict cardinality"]
        L4["L4: Trust use-case bound
        No generic reputation"]
        L15["L15: PeerReview sealing
        Reciprocal OR 14d"]
        L1["L1: Recovery owns deviations
        Single emitter"]
        L10["L10: Canonical events
        Recovery only"]
    end

    subgraph GUARDS["Enforced By Design Guards"]
        DG1["DG-1: Trust Composition Purity"]
        DG45["DG-4/5: Deviation Translation"]
    end

    INV1 --> SESSION
    INV2 --> BOOKING
    INV3 --> MATCH
    INV4 --> PEER
    INV5 --> TRUST
    INV6 --> RECOVERY
    INV7 --> PAYMENT

    L2 -.->|enforces| INV1
    L9 -.->|enforces| INV2
    L4 -.->|enforces| INV5
    L15 -.->|enforces| INV4
    L1 -.->|enforces| INV6
    L10 -.->|enforces| INV6

    DG1 -.->|enforces| INV5
    DG45 -.->|enforces| INV6

    style INV1 fill:#dc2626,color:white
    style INV5 fill:#dc2626,color:white
    style INV6 fill:#dc2626,color:white
    style L2 fill:#f97316,color:white
    style L4 fill:#f97316,color:white
    style L1 fill:#f97316,color:white
```

---

## E5 · Saga Compensation Flow Matrix

**Answers:** What can be rolled back vs. what's irreversible? How do compensations affect trust?
**Why it matters:** Shows the "compensation budget" and trust observation lifecycle.

```mermaid
flowchart TD
    subgraph SAG002["SAG-002: Booking Saga"]
        H1["1. SeatHeld
        TTL: 10min"] --> H2["2. PaymentAuthorized
        Funds reserved"]
        H2 --> H3["3. PaymentCaptured
        Funds transferred"]
        H3 --> H4["4. SeatConfirmed
        Booking complete"]
    end

    subgraph COMP_GREEN["✅ Clean Compensations · No Trust Impact"]
        C1["TTL Expired
        → SeatReleased
        → No observation
        Reason: System timeout"]
        C2["Payment Failed
        → BookingCancelled
        → No observation
        Reason: Technical failure"]
    end

    subgraph COMP_YELLOW["⚠️ Partial Compensations · Conditional Trust Impact"]
        C3["Player Cancel Early
        → RefundDecision Full
        → No observation
        Reason: Within window"]
        C4["Player Cancel Late
        → RefundDecision Partial
        → ReliabilityObserved
        Reason: Outside window"]
    end

    subgraph COMP_RED["❌ Irreversible · Trust Impact Guaranteed"]
        C5["No-Show
        → No refund
        → ReliabilityPenalty
        → Trust profile updated"]
        C6["Match Completed
        → Immutable
        → SkillProfileUpdated
        → Cannot reverse"]
    end

    subgraph REVERSAL["🔄 Dispute Reversal Path"]
        D1["DisputeCase Raised
        → Evidence submitted
        → Adjudication"]
        D2["DisputeResolved
        → EVT-REC-012
        → Observation reversed"]
    end

    H1 -.->|compensates| C1
    H2 -.->|compensates| C2
    H4 -.->|compensates| C3
    H4 -.->|compensates| C4
    H4 -.->|no compensation| C5
    H4 -.->|no compensation| C6

    C4 -.->|can be reversed| D1
    C5 -.->|can be reversed| D1
    D1 --> D2

    style COMP_GREEN fill:#16a34a,color:white
    style COMP_YELLOW fill:#f97316,color:white
    style COMP_RED fill:#dc2626,color:white
    style REVERSAL fill:#7c3aed,color:white
```

---


## E6 · Service Block On-Call Topology (Enhanced)

**Answers:** Who's on-call for what? What are the deployment boundaries? How do failures isolate?
**Why it matters:** v8 introduced Service Blocks but didn't show operational reality.

```mermaid
flowchart TB
    subgraph CB["Coordination Block
    🚨 On-call: Team A
    📦 Deploy: Independent
    🔥 Change Freq: High"]
        COR["Coordination
        8 aggregates
        SLO: 99.9%"]
        REC["Recovery
        5 aggregates
        SLO: 99.95%"]
        MMK["Matchmaking
        3 aggregates
        SLO: 99.5%"]
        HOS["Hosting
        1 aggregate
        SLO: 99.5%"]
    end

    subgraph MB["Money Block
    🚨 On-call: Team B
    📦 Deploy: Independent
    🔥 Change Freq: Medium"]
        FIN["Financial
        8 aggregates
        SLO: 99.99%"]
        PRC["Pricing
        4 aggregates
        SLO: 99.9%"]
        PRT["Partner Relations
        2 aggregates
        SLO: 99.5%"]
    end

    subgraph TBX["Trust Block
    🚨 On-call: Team C
    📦 Deploy: Independent
    🔥 Change Freq: Low"]
        TRS["Trust/Skill
        1 aggregate
        SLO: 99.5%"]
        TRR["Trust/Reliability
        1 aggregate
        SLO: 99.5%"]
        TRF["Trust/Financial
        1 aggregate
        SLO: 99.5%"]
        TRC["Trust/Community
        1 aggregate
        SLO: 99.5%"]
    end

    subgraph OB["Operator Block
    🚨 On-call: Team D
    📦 Deploy: Independent
    🔥 Change Freq: Low"]
        INV["Inventory
        3 aggregates
        SLO: 99.9%"]
        TRN["Training
        3 aggregates
        SLO: 99.5%"]
    end

    subgraph NB["Network Block
    🚨 On-call: Team E
    📦 Deploy: Independent
    🔥 Change Freq: Medium"]
        COM["Community
        4 aggregates
        SLO: 99.5%"]
        GAM["Gamification
        2 aggregates
        SLO: 99.0%"]
    end

    subgraph PB["Platform Block
    🚨 On-call: Platform Team
    📦 Deploy: Shared
    🔥 Change Freq: Low"]
        IDN["Identity
        2 aggregates
        SLO: 99.99%"]
        ACL["ACLs
        Payment·Maps·Identity
        SLO: 99.95%"]
    end

    CB -->|Payment events
    Async via event bus| MB
    CB -->|Observation events
    Async via event bus| TBX
    CB -->|TimeSlot requests
    Sync via API| OB
    MB -->|Trust events
    Async via event bus| TBX
    NB -->|Trust events
    Async via event bus| TBX
    PB -->|Identity events
    Async via event bus| CB
    PB -->|Identity events
    Async via event bus| MB

    style CB fill:#D9534F,stroke:#7A1F1C,color:#fff,stroke-width:3px
    style MB fill:#5BC0DE,stroke:#1F5A73,color:#000,stroke-width:3px
    style TBX fill:#F0AD4E,stroke:#8A5A12,color:#000,stroke-width:3px
    style OB fill:#16a34a,stroke:#15803d,color:#fff,stroke-width:2px
    style NB fill:#7c3aed,stroke:#5b21b6,color:#fff,stroke-width:2px
    style PB fill:#737373,stroke:#404040,color:#fff,stroke-width:2px
```

---

## E7 · Event Storming Big Picture (Multi-Timeline)

**Answers:** What's the full temporal grain across all major flows?
**Why it matters:** Current D8 shows only Game-to-Match. This shows parallel flows.

```mermaid
timeline
    title Event Storming Big Picture - All Major Flows
    
    section Flow 1: Game Creation → Match
        GameProposed : User intent
        GameOpened : Accepts joiners
        SeatHeld : User reserves (TTL)
        PaymentCaptured : Funds transferred
        SeatConfirmed : Booking complete
        SessionScheduled : Venue + time locked
        MatchStarted : Physical play begins
        MatchCompleted : Post-event truth
        
    section Flow 2: Cancellation Cascade
        BookingDeviationRequested : Player cancels
        CancellationCaseOpened : Recovery owns
        RefundEligibilityEvaluated : Policy decision
        RefundIssued : Financial executes
        BookingCancelled : Canonical event (L1)
        SeatReleased : Capacity freed
        ReliabilityObserved : Trust signal
        
    section Flow 3: Replacement Search
        ReplacementCaseOpened : Seat vacated
        CandidatesRanked : Matchmaking filters
        CandidatesNotified : Push/SMS/email
        ReplacementFound : First confirmer
        SubsidyDecisionMade : Subsidy applied
        SubsidyLedgerAppended : Recorded
        
    section Flow 4: No-Show Detection
        MatchStarted : Attendance check
        NoShowCaseOpened : Missing player
        ReliabilityPenaltyApplied : Trust penalty
        ReliabilityProfileUpdated : Profile updated
        
    section Flow 5: Dispute Resolution
        DisputeCaseRaised : Player disputes
        DisputeEvidenceSubmitted : Evidence
        DisputeCaseResolved : Adjudication
        ObservationReversed : Trust reversed (EVT-REC-012)
        PeerReviewAnnotated : Review marked
        
    section Flow 6: BNPL Default
        BNPLObligationCreated : Buy-now-pay-later
        BNPLPaymentMissed : Payment missed
        BNPLDefaultRequested : Deviation
        BNPLDefaultCaseOpened : Recovery owns
        BNPLDefaulted : Canonical event
        FinancialTrustObserved : Trust penalty
```

---

## E8 · Read Model Staleness SLO Matrix

**Answers:** What's the staleness budget per projection? What's the user-facing impact?
**Why it matters:** Makes eventual consistency concrete and measurable.

```mermaid
flowchart LR
    subgraph PROJECTIONS["Read Models with Staleness SLOs"]
        GF["Game Feed
        📊 SLO: 30s eventual
        💥 Impact: Discovery delay
        🔄 Invalidation: GameProposed/Opened"]
        
        SD["Session Details
        📊 SLO: 5s eventual
        💥 Impact: Booking confusion
        🔄 Invalidation: SeatHeld/Confirmed"]
        
        UP["User Profile
        📊 SLO: 10s eventual
        💥 Impact: History delay
        🔄 Invalidation: MatchCompleted"]
        
        LB["Leaderboard
        📊 SLO: 5min eventual
        💥 Impact: Low (gamification)
        🔄 Invalidation: KarmaAwarded"]
        
        VC["Venue Catalog
        📊 SLO: 1h eventual
        💥 Impact: Low (browse)
        🔄 Invalidation: VenueOnboarded"]
        
        DF["Demand Forecast
        📊 SLO: 15min eventual
        💥 Impact: Internal only
        🔄 Invalidation: Booking patterns"]
    end

    subgraph EVENTS["Source Events"]
        E1["GameProposed
        GameOpened
        GameClosed"]
        E2["SessionScheduled
        SeatHeld
        SeatConfirmed"]
        E3["MatchCompleted
        TrustProfileUpdated"]
        E4["KarmaAwarded
        AchievementUnlocked"]
        E5["VenueOnboarded
        PartnerKYCCompleted"]
        E6["BookingCreated
        BookingConfirmed
        Historical patterns"]
    end

    subgraph CACHE["Cache Strategy"]
        CACHE1["Write-through
        Immediate invalidation"]
        CACHE2["Write-behind
        Async invalidation"]
        CACHE3["TTL-based
        Periodic refresh"]
    end

    E1 --> GF
    E2 --> SD
    E3 --> UP
    E4 --> LB
    E5 --> VC
    E6 --> DF

    GF --> CACHE1
    SD --> CACHE1
    UP --> CACHE2
    LB --> CACHE3
    VC --> CACHE3
    DF --> CACHE3

    style GF fill:#dc2626,color:white
    style SD fill:#dc2626,color:white
    style UP fill:#f97316,color:white
    style LB fill:#16a34a,color:white
    style VC fill:#16a34a,color:white
    style DF fill:#16a34a,color:white
```

---

## E9 · Failure Mode Blast Radius Diagram

**Answers:** What's the blast radius of each failure? Which saga contains it?
**Why it matters:** Shows failure isolation boundaries and recovery strategies.

```mermaid
flowchart TD
    subgraph FAILURES["Failure Sources"]
        F1["💥 Payment Gateway Down
        Severity: Critical
        Frequency: Rare"]
        F2["💥 Venue Cancels
        Severity: High
        Frequency: Medium"]
        F3["💥 Host No-Show
        Severity: Medium
        Frequency: Low"]
        F4["💥 Player No-Show
        Severity: Low
        Frequency: High"]
        F5["💥 TimeSlot Unavailable
        Severity: Medium
        Frequency: Low"]
        F6["💥 BNPL Default
        Severity: Medium
        Frequency: Medium"]
    end

    subgraph BLAST["Blast Radius"]
        BR1["🔥 Financial BC
        All payments blocked
        Affects: All bookings"]
        BR2["🔥 Coordination BC
        Sessions cancelled
        Affects: All players in session"]
        BR3["🔥 Trust BC
        Reliability impacted
        Affects: Single player"]
        BR4["🔥 Recovery BC
        Cases opened
        Affects: Deviation lifecycle"]
        BR5["🔥 Inventory BC
        TimeSlot released
        Affects: Single session"]
    end

    subgraph CONTAINMENT["Saga Containment Strategy"]
        S1["SAG-002: Booking
        ✅ Retry with different PSP
        ✅ Intent stays Confirmed
        ✅ Failover to PSP2"]
        
        S2["SAG-006: Venue Cancel
        ✅ Cascade to all bookings
        ✅ Full refunds
        ✅ Replacement search"]
        
        S3["SAG-005: Host Cancel
        ✅ Replacement search
        ✅ Subsidy decision
        ✅ Notify candidates"]
        
        S4["SAG-003: Player Cancel
        ✅ Refund decision
        ✅ Replacement search
        ✅ Reliability observation"]
        
        S5["SAG-001: Game-to-Session
        ✅ TimeSlot release
        ✅ GameAbandoned
        ✅ No trust impact"]
        
        S6["SAG-010: BNPL Default
        ✅ Recovery case
        ✅ Financial trust penalty
        ✅ Collection workflow"]
    end

    F1 --> BR1
    F1 -.->|contained by| S1

    F2 --> BR2
    F2 -.->|contained by| S2

    F3 --> BR2
    F3 -.->|contained by| S3

    F4 --> BR3
    F4 -.->|contained by| S4

    F5 --> BR5
    F5 -.->|contained by| S5

    F6 --> BR3
    F6 -.->|contained by| S6

    style FAILURES fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:2px
    style BLAST fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style CONTAINMENT fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
```

---

## E10 · Idempotency Key Strategy Diagram

**Answers:** Which operations use which idempotency keys? Time-bounded vs. forever-unique?
**Why it matters:** v8 added L16 (Idempotency strategy per aggregate) but no visualization.

```mermaid
flowchart LR
    subgraph TIME_BOUNDED["⏱️ Time-Bounded Idempotency"]
        O1["HoldSeat
        Key: sessionId+userId+seatHoldId
        Window: TTL (10min)
        Reason: Seat hold expires"]
        
        O2["BeginPaymentAttempt
        Key: intentId+attemptId
        Window: Intent lifecycle
        Reason: Retry window"]
    end

    subgraph FOREVER["♾️ Forever-Unique Idempotency"]
        O3["CreateBooking
        Key: sessionId+userId
        Window: Forever
        Reason: Unique constraint"]
        
        O4["CreatePaymentIntent
        Key: bookingId
        Window: Forever
        Reason: 1:1 with booking"]
        
        O5["CapturePayment
        Key: pgRef
        Window: Forever
        Reason: External system ref"]
        
        O6["EmitCanonicalCancellation
        Key: caseId
        Window: Forever
        Reason: Single emission (L1)"]
        
        O7["AppendSubsidyLedger
        Key: replacementCaseId
        Window: Forever
        Reason: Audit trail"]
    end

    subgraph AGGREGATE_BOUNDED["🔒 Aggregate-Bounded Idempotency"]
        O8["ConfirmSeat
        Key: sessionId+userId
        Window: Session lifecycle
        Reason: Tied to aggregate"]
        
        O9["SealPeerReview
        Key: reviewId
        Window: Review lifecycle
        Reason: Immutable after seal"]
    end

    subgraph STRATEGIES["De-dup Strategy"]
        S1["TTL-based expiry
        Cleanup after window"]
        S2["Never expires
        Permanent record"]
        S3["Aggregate lifecycle
        Cleanup on aggregate delete"]
    end

    O1 --> S1
    O2 --> S1
    O3 --> S2
    O4 --> S2
    O5 --> S2
    O6 --> S2
    O7 --> S2
    O8 --> S3
    O9 --> S3

    style TIME_BOUNDED fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style FOREVER fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
    style AGGREGATE_BOUNDED fill:#5BC0DE,stroke:#1F5A73,stroke-width:2px
```

---

## E11 · PeerReview Sealing Window Visualization

**Answers:** How does the sealing window work? When does reveal happen?
**Why it matters:** L15 (PeerReview sealing) is time-based but not visualized.

```mermaid
gantt
    title PeerReview Sealing Window (L15 Enforcement)
    dateFormat X
    axisFormat %H:%M

    section Player A Reviews Player B
    Draft review    :a1, 0, 3600000
    Seal review     :milestone, a2, 3600000, 0

    section Player B Reviews Player A
    Draft review    :b1, 0, 1800000
    Seal review     :milestone, b2, 1800000, 0

    section Reveal Logic
    Reciprocal seal triggers reveal :milestone, m1, 3600000, 0
    OR 14d expiry (if no reciprocal) :milestone, m2, 1209600000, 0

    section After Reveal
    Content immutable :crit, r1, 3600000, 1209600000
    Annotation via Dispute only :crit, r2, 3600000, 1209600000
```

**Key Rules (L15):**
1. Content IMMUTABLE after Seal
2. Reveal = earlier of (reciprocal seal OR session.end + 14d)
3. After Reveal, only DisputeResolved can annotate
4. Prevents retaliatory rating

---

## E12 · Dispute Resolution Reversal Flow

**Answers:** How does DisputeCase reverse prior trust observations?
**Why it matters:** v8 added EVT-REC-012 (observation reversal) but no visualization.

```mermaid
sequenceDiagram
    autonumber
    actor P as Player
    participant D as DisputeCase
    participant TR as Trust Profiles
    participant PR as PeerReview
    participant FIN as Financial

    Note over P: Player disputes no-show penalty

    P->>D: CMD-REC-007 OpenDisputeCase(reason)
    Note over D: unique(target, initiator, window)

    P->>D: CMD-REC-008 SubmitDisputeEvidence(proof)
    Note over D: State: Raised → UnderReview

    alt Adjudicator reviews
        D->>D: CMD-REC-009 ResolveDisputeCase(decision)
        
        alt Resolved in favor of player
            D->>TR: EVT-REC-012 DisputeResolved(reversal)
            Note over TR: Reverse prior ReliabilityObserved
            TR->>TR: Recompute reliability rate
            
            D->>PR: DisputeResolved(annotation)
            Note over PR: Annotate PeerReview as disputed
            
            opt If refund decision disputed
                D->>FIN: DisputeResolved(refund adjustment)
                Note over FIN: Manual escalation for refund
            end
            
        else Resolved against player
            D->>TR: DisputeResolved(upheld)
            Note over TR: Observation stands
        end
        
    else Window expires (14d)
        D->>D: Auto-transition to Expired
        Note over D: Default to Void - No impact on trust
    end

    Note over D: State: Resolved/Expired (terminal)
```

**Key Insights:**
- DisputeResolved is the ONLY way to reverse a trust observation
- Reversal is explicit via EVT-REC-012
- PeerReview gets annotated, not deleted
- Refund adjustments require manual escalation

---


## E13 · Value Object Shared Kernel Heatmap

**Answers:** Which VOs are used by 3+ BCs? What's the shared kernel risk?
**Why it matters:** High-reuse VOs need extra design care (DG-6).

```mermaid
flowchart TD
    subgraph HIGH_REUSE["🔥 High Reuse VOs (3+ BCs) - Shared Kernel Candidates"]
        MONEY["Money
        VO-04
        Currency + amount
        Used by: Financial, Pricing, Booking, Partner
        Reuse: 4 BCs"]
        
        TIMEWINDOW["TimeWindow
        VO-02
        Start/end validation
        Used by: Coordination, Inventory, Training
        Reuse: 3 BCs"]
        
        GEO["Geo
        VO-06
        Lat/lng validation
        Used by: Inventory, Community, Training
        Reuse: 3 BCs"]
        
        SPORT["Sport
        VO-01
        Enum validation
        Used by: Coordination, Inventory, Training
        Reuse: 3 BCs"]
    end

    subgraph MEDIUM_REUSE["⚠️ Medium Reuse VOs (2 BCs)"]
        SKILL_RANGE["SkillRange
        VO-07
        Min/max validation
        Used by: Coordination, Trust/Skill
        Reuse: 2 BCs"]
        
        LOCATION["Location
        VO-03
        Address + geo
        Used by: Coordination, Inventory
        Reuse: 2 BCs"]
        
        BOOKING_STATUS["BookingStatus
        VO-08
        State enum
        Used by: Coordination, Financial
        Reuse: 2 BCs"]
        
        COMMISSION_RATE["CommissionRate
        VO-11
        Percentage validation
        Used by: Pricing, Partner
        Reuse: 2 BCs"]
    end

    subgraph SINGLE_USE["✅ Single-Use VOs (1 BC)"]
        SESSION_STATUS["SessionStatus
        VO-05
        State enum
        Used by: Coordination only"]
        
        PAYMENT_STATUS["PaymentStatus
        VO-09
        State enum
        Used by: Financial only"]
        
        TRUST_SCORE["TrustScore
        VO-10
        0-100 validation
        Used by: All 4 Trust BCs
        BUT: Never composed (DG-1)"]
    end

    subgraph DESIGN_CARE["Design Care Required"]
        DC1["Immutability enforcement
        DG-6"]
        DC2["Versioning strategy
        Breaking changes"]
        DC3["Validation consistency
        Across BCs"]
        DC4["Serialization format
        Wire compatibility"]
    end

    MONEY --> DC1
    MONEY --> DC2
    MONEY --> DC3
    MONEY --> DC4

    TIMEWINDOW --> DC1
    TIMEWINDOW --> DC3

    GEO --> DC1
    GEO --> DC3

    SPORT --> DC1
    SPORT --> DC2

    style HIGH_REUSE fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:3px
    style MEDIUM_REUSE fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style SINGLE_USE fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
    style DESIGN_CARE fill:#7c3aed,color:white,stroke:#5b21b6,stroke-width:2px
```

---

## E14 · Saga Orchestration vs. Choreography Decision Matrix

**Answers:** Which sagas are orchestrated vs. choreographed? Why?
**Why it matters:** Shows the architectural trade-offs in saga design.

```mermaid
flowchart TD
    subgraph ORCHESTRATED["🎯 Orchestrated Sagas (Explicit Coordinator)"]
        O1["SAG-001: Game-to-Session
        Orchestrator: Coordination
        Why: Complex state machine
        Participants: Inventory, Pricing"]
        
        O2["SAG-002: Booking
        Orchestrator: Coordination
        Why: Financial commitment
        Participants: Financial, Recovery"]
        
        O3["SAG-003: Player Cancel
        Orchestrator: Recovery
        Why: Deviation lifecycle
        Participants: Financial, Trust"]
        
        O4["SAG-004: Replacement
        Orchestrator: Recovery
        Why: Multi-step search
        Participants: Matchmaking, Coordination"]
        
        O5["SAG-011: Waitlist Promote
        Orchestrator: Coordination
        Why: Seat allocation
        Participants: Financial"]
    end

    subgraph CHOREOGRAPHED["💃 Choreographed Sagas (Event-Driven)"]
        C1["SAG-007: Gamification
        Trigger: MatchCompleted
        Why: Low coupling
        Consumers: Gamification, Community"]
        
        C2["SAG-009: Community→Trust
        Trigger: PeerReviewRevealed
        Why: Observation pattern
        Consumers: Trust profiles"]
        
        C3["SAG-008: Yield/Subsidy
        Trigger: Cell thinness
        Why: Policy-driven
        Consumers: Pricing"]
    end

    subgraph HYBRID["🔀 Hybrid Sagas (Mixed Pattern)"]
        H1["SAG-005: Host Cancel
        Orchestrator: Recovery
        Choreography: Cascade to bookings
        Why: Deviation + broadcast"]
        
        H2["SAG-006: Venue Cancel
        Orchestrator: Recovery
        Choreography: Cascade to bookings
        Why: Deviation + broadcast"]
        
        H3["SAG-013: Dispute
        Orchestrator: Recovery
        Choreography: Observation reversal
        Why: Adjudication + broadcast"]
    end

    subgraph DECISION_FACTORS["Decision Factors"]
        DF1["Orchestration when:
        • Complex compensation
        • Financial commitment
        • Multi-step coordination"]
        
        DF2["Choreography when:
        • Low coupling desired
        • Observation pattern
        • Broadcast to many"]
        
        DF3["Hybrid when:
        • Deviation + cascade
        • Adjudication + broadcast
        • Mixed concerns"]
    end

    O1 -.-> DF1
    O2 -.-> DF1
    C1 -.-> DF2
    C2 -.-> DF2
    H1 -.-> DF3
    H2 -.-> DF3

    style ORCHESTRATED fill:#5BC0DE,stroke:#1F5A73,stroke-width:2px
    style CHOREOGRAPHED fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
    style HYBRID fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
```

---

## E15 · Concurrency Strategy Per Aggregate

**Answers:** Which aggregates use OCC vs. pessimistic locking? Why?
**Why it matters:** v8 added L17 (Concurrency strategy per aggregate) but no visualization.

```mermaid
flowchart LR
    subgraph OCC["⚡ Optimistic Concurrency Control (OCC)"]
        OCC1["Session
        Strategy: Version counter
        Why: High contention on seat operations
        Retry: Client-side"]
        
        OCC2["TimeSlot
        Strategy: Version counter
        Why: High contention on holds
        Retry: Client-side"]
        
        OCC3["PaymentIntent
        Strategy: Version counter
        Why: Multiple attempts per intent
        Retry: Client-side"]
    end

    subgraph PESSIMISTIC["🔒 Pessimistic Locking"]
        PESS1["Booking
        Strategy: Row-level lock
        Why: Financial commitment
        Retry: Server-side"]
        
        PESS2["Payment
        Strategy: Row-level lock
        Why: Funds transfer
        Retry: Server-side"]
        
        PESS3["SubsidyLedger
        Strategy: Row-level lock
        Why: Audit trail
        Retry: Server-side"]
    end

    subgraph SINGLE_WRITER["✅ Single-Writer (No Concurrency Control)"]
        SW1["Match
        Strategy: Write-once
        Why: Immutable after completion
        Retry: Not applicable"]
        
        SW2["PeerReview
        Strategy: Write-once after seal
        Why: Immutable after seal
        Retry: Not applicable"]
        
        SW3["Trust Profiles
        Strategy: Append-only
        Why: Event sourcing
        Retry: Idempotent append"]
    end

    subgraph DECISION_FACTORS["Decision Factors"]
        DF1["OCC when:
        • High read:write ratio
        • Low conflict probability
        • Client can retry"]
        
        DF2["Pessimistic when:
        • Financial operations
        • High conflict probability
        • Server must guarantee"]
        
        DF3["Single-writer when:
        • Immutable aggregates
        • Append-only logs
        • No conflicts possible"]
    end

    OCC1 -.-> DF1
    OCC2 -.-> DF1
    PESS1 -.-> DF2
    PESS2 -.-> DF2
    SW1 -.-> DF3
    SW3 -.-> DF3

    style OCC fill:#5BC0DE,stroke:#1F5A73,stroke-width:2px
    style PESSIMISTIC fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style SINGLE_WRITER fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
```

---

## E16 · Partition Key Strategy for Scalability

**Answers:** How are aggregates partitioned for horizontal scaling?
**Why it matters:** v8 added L18 (Partition key strategy) but no visualization.

```mermaid
flowchart TD
    subgraph USER_PARTITIONED["👤 User-Partitioned Aggregates"]
        UP1["Booking
        Partition: userId
        Why: User-centric queries
        Shard: Consistent hash"]
        
        UP2["Trust Profiles
        Partition: userId
        Why: User-centric queries
        Shard: Consistent hash"]
        
        UP3["PaymentIntent
        Partition: userId
        Why: User payment history
        Shard: Consistent hash"]
    end

    subgraph SESSION_PARTITIONED["🎮 Session-Partitioned Aggregates"]
        SP1["Session
        Partition: sessionId
        Why: Session-centric operations
        Shard: Consistent hash"]
        
        SP2["Match
        Partition: sessionId
        Why: Co-located with Session
        Shard: Consistent hash"]
        
        SP3["Seat (entity)
        Partition: sessionId
        Why: Within Session aggregate
        Shard: Consistent hash"]
    end

    subgraph TIME_PARTITIONED["📅 Time-Partitioned Aggregates"]
        TP1["TimeSlot
        Partition: date + venueId
        Why: Time-range queries
        Shard: Range-based"]
        
        TP2["SubsidyLedger
        Partition: date
        Why: Audit trail queries
        Shard: Range-based"]
        
        TP3["KarmaLedger
        Partition: date
        Why: Historical queries
        Shard: Range-based"]
    end

    subgraph COMPOSITE_PARTITIONED["🔀 Composite-Partitioned Aggregates"]
        CP1["PeerReview
        Partition: sessionId + userId
        Why: Session + user queries
        Shard: Composite hash"]
        
        CP2["CancellationCase
        Partition: targetId + initiatorId
        Why: Bilateral queries
        Shard: Composite hash"]
    end

    subgraph SCALING_PATTERNS["Scaling Patterns"]
        PAT1["User-partitioned:
        • Scales with user growth
        • Hot users = hot shards
        • Rebalance on growth"]
        
        PAT2["Session-partitioned:
        • Scales with sessions
        • Even distribution
        • No hot shards"]
        
        PAT3["Time-partitioned:
        • Scales with time
        • Archive old partitions
        • Predictable growth"]
        
        PAT4["Composite-partitioned:
        • Scales with both
        • Complex rebalancing
        • Use sparingly"]
    end

    UP1 -.-> PAT1
    SP1 -.-> PAT2
    TP1 -.-> PAT3
    CP1 -.-> PAT4

    style USER_PARTITIONED fill:#5BC0DE,stroke:#1F5A73,stroke-width:2px
    style SESSION_PARTITIONED fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
    style TIME_PARTITIONED fill:#f97316,color:white,stroke:#d97706,stroke-width:2px
    style COMPOSITE_PARTITIONED fill:#7c3aed,color:white,stroke:#5b21b6,stroke-width:2px
```

---

## E17 · Anti-Pattern Detection Checklist

**Answers:** What are the most common violations? How to detect them?
**Why it matters:** Codifies the "forbidden" patterns from the evolution journey.

```mermaid
flowchart TD
    subgraph TRUST_ANTIPATTERNS["❌ Trust Anti-Patterns (DG-1 Violations)"]
        TAP1["Composed TrustScore
        Detection: Column named 'reputation'
        Fix: Delete column, use policy"]
        
        TAP2["Cache-dependent decision
        Detection: Decision breaks on cache miss
        Fix: Make cache optional"]
        
        TAP3["Missing use_case parameter
        Detection: getReputation(userId)
        Fix: Add use_case parameter"]
    end

    subgraph HOST_ANTIPATTERNS["❌ Host Anti-Patterns (DG-2 Violations)"]
        HAP1["Host mutates coordination
        Detection: Host imports Coordination types
        Fix: Remove imports"]
        
        HAP2["Host in saga orchestration
        Detection: Host as saga step
        Fix: Use capability check only"]
        
        HAP3["Host triggers workflows
        Detection: Host emits non-capability events
        Fix: Emit capability events only"]
    end

    subgraph POLICY_ANTIPATTERNS["❌ Policy Anti-Patterns (DG-3 Violations)"]
        PAP1["Policy writes to DB
        Detection: Repository injection
        Fix: Return decision object"]
        
        PAP2["Policy calls external system
        Detection: HTTP client injection
        Fix: Pass data as input"]
        
        PAP3["Policy emits events
        Detection: Event publisher injection
        Fix: Return decision, caller emits"]
    end

    subgraph RECOVERY_ANTIPATTERNS["❌ Recovery Anti-Patterns (DG-4/5 Violations)"]
        RAP1["Aggregate emits *Cancelled
        Detection: Event name ends with 'Cancelled'
        Fix: Emit *DeviationRequested"]
        
        RAP2["Multiple emitters of canonical
        Detection: *Cancelled from non-Recovery
        Fix: Route through Recovery"]
        
        RAP3["Recovery bypassed
        Detection: Direct cancellation
        Fix: Always go through Recovery"]
    end

    subgraph DETECTION_TOOLS["🔍 Detection Tools"]
        DT1["Static analysis
        Lint rules for imports"]
        DT2["Architecture tests
        ArchUnit / NetArchTest"]
        DT3["Event schema validation
        Event naming conventions"]
        DT4["Code review checklist
        PR template"]
    end

    TAP1 --> DT1
    TAP2 --> DT2
    HAP1 --> DT1
    HAP2 --> DT2
    PAP1 --> DT1
    PAP2 --> DT1
    RAP1 --> DT3
    RAP2 --> DT3

    style TRUST_ANTIPATTERNS fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:2px
    style HOST_ANTIPATTERNS fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:2px
    style POLICY_ANTIPATTERNS fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:2px
    style RECOVERY_ANTIPATTERNS fill:#dc2626,color:white,stroke:#7A1F1C,stroke-width:2px
    style DETECTION_TOOLS fill:#16a34a,color:white,stroke:#15803d,stroke-width:2px
```

---

## Summary: What Makes These Diagrams "Richer"

### 1. **Evolution Context**
- E1 shows the journey, not just the destination
- Prevents regression to v6 anti-patterns
- Documents architectural breakthroughs

### 2. **Enforcement Relationships**
- E2 shows how guards enforce locked decisions
- Makes the dependency structure explicit
- Shows that guards aren't arbitrary

### 3. **Decision Logic**
- E3 shows trust composition decision trees
- Makes L4 + DG-1 concrete
- Shows use-case-specific logic

### 4. **Operational Reality**
- E6 shows on-call topology and deployment boundaries
- E8 shows staleness SLOs and user impact
- E9 shows failure blast radius and containment

### 5. **Scalability Strategies**
- E15 shows concurrency strategies per aggregate
- E16 shows partition key strategies
- E10 shows idempotency strategies

### 6. **Anti-Pattern Detection**
- E17 codifies forbidden patterns
- Shows detection tools
- Prevents drift

---

## Comparison: Existing vs. Enhanced Diagrams

| Existing Diagram | Enhancement | What's Added |
|------------------|-------------|--------------|
| D1 Subdomain Heatmap | E1 Evolution Timeline | Shows how we got here |
| D2 Context Map | E6 Service Block Topology | Shows on-call boundaries |
| D3 Trust Constellation | E3 Trust Decision Tree | Shows decision logic |
| D5 Aggregate Constellation | E4 Invariant Cross-Reference | Shows invariant dependencies |
| D9 Booking Saga | E5 Compensation Flow | Shows trust impact |
| D12 Read Model Projection | E8 Staleness SLO Matrix | Shows user impact |
| (Missing) | E9 Failure Blast Radius | Shows containment |
| (Missing) | E10 Idempotency Strategy | Shows de-dup logic |
| (Missing) | E12 Dispute Reversal | Shows observation reversal |
| (Missing) | E14 Orchestration vs. Choreography | Shows saga patterns |
| (Missing) | E15 Concurrency Strategy | Shows OCC vs. locking |
| (Missing) | E16 Partition Strategy | Shows scaling patterns |
| (Missing) | E17 Anti-Pattern Detection | Shows forbidden patterns |

---

## Recommended Reading Order

1. **E1 Evolution Timeline** — Understand the journey
2. **E2 Locked Decision Dependency** — Understand the enforcement structure
3. **E3 Trust Decision Tree** — Understand the most critical architectural decision
4. **E6 Service Block Topology** — Understand operational reality
5. **E9 Failure Blast Radius** — Understand failure isolation
6. **E5 Compensation Flow** — Understand saga compensation
7. **E17 Anti-Pattern Detection** — Understand forbidden patterns

---

## Next Steps

1. **Validate with stakeholders:** Review E1-E17 with architects and team leads
2. **Add to CI/CD:** Generate diagrams from workbook on every commit
3. **Architecture tests:** Implement E17 detection tools
4. **Onboarding:** Use E1-E3 for new architect onboarding
5. **Runbooks:** Use E9 for SRE runbooks

