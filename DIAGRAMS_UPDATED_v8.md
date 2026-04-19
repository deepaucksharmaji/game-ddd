# Playo DDD v8 Diagram Suite

Complete enhanced diagram suite from the latest v8 model specification.

---

## 1. Strategic Context Map (v8 Update)

```mermaid
flowchart TB
    subgraph CORE[CORE DOMAIN]
        COR[Coordination<br/>Game·Session·Booking·Match<br/>BookingGroup·Waitlist·Series·CheckIn]
        REC[Recovery<br/>CancellationCase·NoShowCase<br/>ReplacementCase·DisputeCase<br/>SubsidyLedger]
        MMK[Matchmaking<br/>SkillRating·TeamBalance<br/>StackingFlag]
    end

    subgraph SUPPORTING[SUPPORTING DOMAIN]
        INV[Inventory<br/>TimeSlot·Venue·Court]
        PRC[Pricing<br/>PriceRule·PriceSnapshot<br/>YieldDecision·CommissionRate]
        PRT[Partner Relations<br/>PartnerRelationship·Settlement]
        HOS[Hosting<br/>HostProfile·Capabilities]
        COM[Community<br/>UserProfile·PlayPal·Squad·PeerReview]
        GAM[Gamification<br/>KarmaLedger·Achievement]
        TRN[Training<br/>Coach·Academy·TrainingSession]
    end

    subgraph TRUST[TRUST — 4 profiles, composed at query time only · L4]
        TRS[Trust/Skill]
        TRR[Trust/Reliability]
        TRF[Trust/Financial]
        TRC[Trust/Community]
    end

    subgraph GENERIC[GENERIC DOMAIN]
        IDN[Identity<br/>User·AuthProvider]
        FIN[Financial<br/>Intent·Attempt·Payment·Refund<br/>Wallet·BNPL·Split·Agreement]
    end

    COR -- "CS · SAG-001" --> INV
    COR -- "CS · SAG-001" --> PRC
    COR -- "ACL-Payment · SAG-002" --> FIN
    COR -- "publishes *DeviationRequested" --> REC
    FIN -- "publishes *DeviationRequested" --> REC
    INV -- "VenueUnavailable" --> REC
    HOS -- "HostCancelled" --> REC

    REC == "canonical *Cancelled · L1" ==> COR
    REC -- "IssueRefund · SAG-003" --> FIN
    REC -- "ReleaseTimeSlot" --> INV
    REC -- "Observation events" --> TRS
    REC -- "Observation events" --> TRR
    REC -- "Observation events" --> TRF
    REC -- "CandidatesRanked · SAG-004" --> MMK
    MMK -- "reads ratings" --> REC
    COR -- "MatchCompleted" --> MMK
    COR -- "MatchCompleted" --> TRS
    COR -- "MatchCompleted·CheckIn" --> TRR
    FIN -- "Payment·BNPL events" --> TRF
    COM -- "PeerReviewRevealed" --> TRC
    COM -- "PeerReviewRevealed" --> TRS
    PRT -- "consumes Payment/Refund" --> FIN
    INV -- "partnerId ref" --> PRT
    HOS -- "query capabilities · CF" --> COR
    IDN -- "OHS · IdentityACL" --> COR
    IDN -- "OHS" --> COM
    IDN -- "OHS" --> HOS
    COR -- "MatchCompleted" --> GAM
    COM -- "PeerReviewRevealed" --> GAM

    TRS -. "read at query · L4" .-> COR
    TRR -. "read at query · L4" .-> COR
    TRF -. "read at query · L4" .-> FIN
    TRC -. "read at query · L4" .-> COM

    style CORE fill:#D9534F,stroke:#7A1F1C,color:white,stroke-width:2px
    style SUPPORTING fill:#5BC0DE,stroke:#1F5A73,color:black,stroke-width:2px
    style TRUST fill:#F0AD4E,stroke:#8A5A12,color:black,stroke-width:2px
    style GENERIC fill:#999,stroke:#333,color:white,stroke-width:2px
```

---

## 2. Operational Service Blocks

```mermaid
flowchart LR
    subgraph CB[Coordination Block]
        direction TB
        COR[Coordination]
        REC[Recovery]
        MMK[Matchmaking]
        HOS[Hosting]
    end
    subgraph MB[Money Block]
        direction TB
        FIN[Financial]
        PRC[Pricing]
        PRT[Partner Relations]
    end
    subgraph TBX[Trust Block]
        direction TB
        TRS[Trust/Skill]
        TRR[Trust/Reliability]
        TRF[Trust/Financial]
        TRC[Trust/Community]
    end
    subgraph OB[Operator Block]
        direction TB
        INV[Inventory]
        TRN[Training]
    end
    subgraph NB[Network Block]
        direction TB
        COM[Community]
        GAM[Gamification]
    end
    subgraph PB[Platform Block]
        direction TB
        IDN[Identity]
        ACL[ACLs<br/>Payment·Maps·Identity·Partner·Notifications]
    end

    CB --> MB
    CB --> TBX
    CB --> OB
    CB --> NB
    MB --> TBX
    NB --> TBX
    PB --> CB
    PB --> MB
    PB --> OB
```

---

## 3. Aggregate Lifecycle (Core Narrative)

```mermaid
flowchart LR
    subgraph HAPPY[Happy path · left to right]
        direction LR
        G[Game<br/>Proposed → Open → Scheduled]
        S[Session<br/>Scheduled → Confirmed → Started → Completed<br/>held+confirmed ≤ max · INV-COR-002]
        B[Booking<br/>Created → Confirmed<br/>1:1 with seat · L7]
        CI[CheckIn<br/>Recorded · proof:VO-23]
        M[Match<br/>Started → Completed<br/>attendance from CheckIn · INV-COR-004]
        G --> S
        S --> B
        B -. captures seat .-> S
        CI -. gates StartMatch .-> M
        S --> M
    end

    subgraph ADJ[Adjacents · created with or inside Session]
        BG[BookingGroup<br/>Forming → Closed]
        WL[Waitlist<br/>Queued → Promoted → Fulfilled·Expired]
        SS[SessionSeries<br/>Active → Terminated]
        BG -. N bookings .-> B
        WL -. promote on release .-> S
        SS -. generates .-> G
    end

    subgraph DEV[Deviation path · Recovery owns · L1]
        GA[GameAbandoned]
        SC[SessionCancelled]
        BC[BookingCancelled]
        NS[NoShowRecorded]
        PF[PaymentFailed]
        BD[BNPLDefaulted]
        DR[Recovery cases<br/>Cancellation·NoShow·Replacement<br/>PartialFulfillment·Dispute]
        G -. DeviationRequested .-> DR
        S -. DeviationRequested .-> DR
        B -. DeviationRequested .-> DR
        DR --> GA
        DR --> SC
        DR --> BC
        DR --> NS
        DR --> PF
        DR --> BD
    end
```

---

## 4. Booking Saga (SAG-002) Complete Sequence

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant COR as Coordination
    participant FIN as Financial
    participant REC as Recovery
    participant PSP as PSP (via PaymentACL)

    U->>COR: CMD-COR-005 HoldSeat(ttl)
    Note over COR: guard: held+confirmed<max<br/>INV-COR-002
    COR-->>U: SeatHeld (TTL set)

    U->>COR: CMD-COR-009 CreateBooking(snapshotId)
    Note over COR: unique(session,user) · INV-COR-006
    COR-->>FIN: BookingCreated

    FIN->>FIN: CMD-FIN-001 CreatePaymentIntent
    FIN->>U: prompt method
    U->>FIN: CMD-FIN-002 ConfirmPaymentIntent(method)
    FIN->>FIN: CMD-FIN-003 BeginPaymentAttempt(psp)
    FIN->>PSP: Authorize

    alt PSP authorizes
        PSP-->>FIN: pgRef
        FIN->>FIN: CMD-FIN-004 AuthorizeAttempt
        FIN->>PSP: Capture
        alt Capture ok
            PSP-->>FIN: capturedAt
            FIN->>FIN: CMD-FIN-005 CaptureAttempt<br/>(Payment created · INV-FIN-003)
            FIN-->>COR: PaymentCaptured
            COR->>COR: CMD-COR-010 MarkBookingConfirmed
            COR->>COR: CMD-COR-006 ConfirmSeat<br/>(held-=1; confirmed+=1)
            COR-->>U: BookingConfirmed
        else Capture fails
            FIN->>FIN: CMD-FIN-006 FailAttempt
            FIN-->>REC: PaymentDeviationRequested
            REC->>REC: OpenCancellationCase
            REC-->>COR: BookingCancelled (canonical · L1)
            COR->>COR: ReleaseSeat (compensation)
        end
    else Authorize fails
        FIN->>FIN: FailAttempt<br/>(Intent stays Confirmed for retry)
        opt retry with different PSP
            FIN->>FIN: BeginPaymentAttempt(psp2)
        end
        alt retries exhausted
            FIN-->>REC: PaymentDeviationRequested
            REC-->>COR: BookingCancelled
            COR->>COR: ReleaseSeat
        end
    end
```

---

## 5. Cancellation Cascade (SAG-003)

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant COR as Coordination
    participant REC as Recovery
    participant POL as RefundEligibility Policy
    participant FIN as Financial
    participant TRR as Trust/Reliability
    participant MMK as Matchmaking

    U->>COR: CMD-COR-011 RequestBookingDeviation(reason)
    COR-->>REC: BookingDeviationRequested (EVT-COR-011)

    REC->>REC: CMD-REC-001 OpenCancellationCase
    REC->>POL: evaluate(case, session.window, now)
    POL-->>REC: decision {Full|Partial|None}, amount

    alt decision ≠ None
        REC->>FIN: CMD-FIN-007 IssueRefund
        FIN-->>REC: RefundIssued (INV-FIN-004)
    end

    REC->>REC: CMD-REC-003 EmitCanonicalCancellation
    REC-->>COR: BookingCancelled (canonical · L1 · INV-REC-003)
    COR->>COR: CMD-COR-007 ReleaseSeat

    alt time-to-session ≥ threshold
        REC->>REC: CMD-REC-005 OpenReplacementCase (spawns SAG-004)
        REC->>MMK: CMD-MMK-004 RankReplacementCandidates (L14 filter)
        MMK-->>REC: CandidatesRanked
        Note over REC,MMK: continues in SAG-004
    end

    REC-->>TRR: ReliabilityObserved (signal, not profile write)
    Note over TRR: DisputeCase can reverse this via EVT-REC-012
```

---

## 6. Replacement Search (SAG-004)

```mermaid
sequenceDiagram
    autonumber
    participant REC as Recovery
    participant MMK as Matchmaking
    participant NOT as NotificationsACL
    participant U as Candidate users
    participant COR as Coordination
    participant FIN as Financial

    REC->>MMK: CMD-MMK-004 RankReplacementCandidates(pool)
    Note over MMK: L14 filter<br/>skill ∧ geo ∧ non-conflict ∧ reliability
    MMK-->>REC: ranked list

    loop top-N candidates, TTL bounded by session.start − safety
        REC->>NOT: notify candidate
        NOT-->>U: push/SMS/email
    end

    alt first confirmer within TTL
        U->>COR: HoldSeat (vacated seat)
        COR-->>FIN: BookingCreated (subsidised per POL-REC-003)
        Note over FIN: SAG-002 runs nested
        FIN-->>COR: PaymentCaptured
        COR-->>REC: BookingConfirmed
        REC->>REC: CMD-REC-006 RecordReplacementOutcome(Found, subsidy)
        REC->>REC: SubsidyLedger.append (INV-REC-008)
    else TTL expires
        REC->>REC: RecordReplacementOutcome(Failed)
    end
```

---

## 7. Session Statechart (Capacity Sub-States)

```mermaid
stateDiagram-v2
    [*] --> Scheduled : CMD-COR-004<br/>timeSlot held by saga

    state Scheduled {
        [*] --> AwaitingQuorum
        AwaitingQuorum --> Filling : min_participants_reached
        Filling --> Full : confirmedCount == max
    }

    Scheduled --> Started : MatchStarted<br/>CheckIn records locked
    Scheduled --> Cancelled : SessionCancelled<br/>Recovery owns

    Started --> Completed : MatchCompleted
    Started --> Cancelled : SessionCancelled

    Completed --> [*]
    Cancelled --> [*]

    note right of Scheduled: Invariants:
    note right of Scheduled: heldCount + confirmedCount ≤ max<br/>heldCount decreases only on release<br/>confirmedCount is monotonic increasing
```

---

## Diagram Status

| Diagram | Status | v8 Version |
|---|---|---|
| ✅ Strategic Context Map | Complete | v8 |
| ✅ Operational Service Blocks | Complete | v8 |
| ✅ Aggregate Lifecycle | Complete | v8 |
| ✅ Booking Saga (SAG-002) | Complete | v8 |
| ✅ Cancellation Cascade (SAG-003) | Complete | v8 |
| ✅ Replacement Search (SAG-004) | Complete | v8 |
| ✅ Session Statechart | Complete | v8 |

All diagrams are fully enhanced based on the latest v8 model changes, include proper labels, invariants, locked decisions, and workbook ID references.
