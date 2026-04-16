#  DDD v8 — Diagram Set

This file is a **view onto `Playo_DDD_v8.xlsx`**, not a separate model. Every node, edge, state, command, and event in these diagrams traces to an ID in the workbook. If a diagram disagrees with the workbook, the workbook wins — diagrams are regenerable.

## Reading order

1. **#4 Aggregate Lifecycle** — start here for intuition. What does a Game actually become?
2. **#1 Strategic Context Map** — now that you know the pieces, how are they related?
3. **#5 / #6 / #7 Saga sequences** — how does this actually run at request time?
4. **#8 – #11 Statecharts** — what transitions are legal and what guards them?
5. **#12 / #13 / #14 Cross-cutting** — where is the system coupled? Where are the hotspots?

## Conventions used throughout

| Notation | Meaning |
|---|---|
| Solid edge | Write / command call |
| Dashed edge | Read / query |
| Thick edge | Canonical event (Recovery only, per L1) |
| Bold **O / P / T / C** in matrix | Orchestrator / Participant / Trigger / Compensation |
| `EVT-XXX-nnn` | Event ID in workbook |
| `CMD-XXX-nnn` | Command ID in workbook |
| `INV-XXX-nnn` | Invariant ID in workbook |
| `L#` | Locked Decision in 00_Essence |

---

## 1. Strategic Context Map

**Answers:** Where are the bounded contexts, how are they grouped, and how do they depend on each other?
**Sources:** `03_Domain_Classification`, `04_Context_Map`, every BC sheet's `§CONSUMES`.
**What to look for:** Recovery sits at the centre of the deviation flow (all `*DeviationRequested` arrows point in; all canonical `*Cancelled` arrows come out). The four Trust BCs are read-only from decision contexts (dashed edges, per L4).

```mermaid
flowchart TB
    subgraph CORE[CORE]
        COR[Coordination<br/>Game·Session·Booking·Match<br/>BookingGroup·Waitlist·Series·CheckIn]
        REC[Recovery<br/>CancellationCase·NoShowCase<br/>ReplacementCase·DisputeCase<br/>SubsidyLedger]
        MMK[Matchmaking<br/>SkillRating·TeamBalance<br/>StackingFlag]
    end

    subgraph SUPPORTING[SUPPORTING]
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

    subgraph GENERIC[GENERIC]
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

    classDef core fill:#D9534F,stroke:#7A1F1C,color:#fff
    classDef sup fill:#5BC0DE,stroke:#1F5A73,color:#000
    classDef trust fill:#F0AD4E,stroke:#8A5A12,color:#000
    classDef gen fill:#999,stroke:#333,color:#fff
    class COR,REC,MMK core
    class INV,PRC,PRT,HOS,COM,GAM,TRN sup
    class TRS,TRR,TRF,TRC trust
    class IDN,FIN gen
```

---

## 2. Service Blocks — operational grouping

**Answers:** If we staffed this with pods and on-call rotations, how would it cluster?
**Sources:** Proposed grouping; not yet a sheet in the workbook. This is ops-layer, not domain-layer — the Blocks never share aggregates across BCs (L19).

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

## 3. Domain Classification Heatmap

**Answers:** Where should the best engineers and tightest SLOs be?
**Sources:** `03_Domain_Classification` plus judgment on change frequency and on-call pain.

| BC | Class | Differentiation | Change frequency | On-call pain |
|---|---|---|---|---|
| Coordination | CORE | 🔥🔥🔥 | High | High |
| Recovery | CORE | 🔥🔥🔥 | Medium | Highest |
| Matchmaking | CORE | 🔥🔥 | High | Medium |
| Inventory | SUP | 🔥 | Medium | High |
| Pricing | SUP | 🔥🔥 | High | Medium |
| Partner Relations | SUP | 🔥 | Low | Low |
| Hosting | SUP | 🔥 | Low | Low |
| Community | SUP | 🔥 | Medium | Low |
| Gamification | SUP | ❄ | Medium | Low |
| Training | SUP | 🔥 | Low | Low |
| Trust × 4 | SUP | 🔥🔥🔥 | Medium | Low |
| Identity | GEN | ❄ | Low | Medium |
| Financial | GEN | 🔥 | Medium | Highest |

---

## 4. Aggregate Lifecycle — the core narrative

**Answers:** What is the happy path a Game travels? What adjacents exist? Where does Recovery branch in?
**Sources:** `10_BC_Coordination`, `11_BC_Recovery`, all statechart rows.
**What to look for:** Three tracks. The Happy track flows left-to-right. Adjacents (BookingGroup, Waitlist, SessionSeries) sit above. The Deviation track is entirely Recovery-owned — L1 made visual.

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

## 5. Booking Saga (SAG-002) — full sequence

**Answers:** What happens end-to-end when a user books? How does retry/failover work? What compensates what?
**Sources:** `30_Sagas` SAG-002; `10_BC_Coordination` CMD-COR-005/009/006/010; `15_BC_Financial` CMD-FIN-001..006.
**What to look for:** The Intent/Attempt/Payment split lets retries happen *within* one Intent — the Intent stays Confirmed while Attempts fail and new ones begin against a different PSP.

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

## 6. Cancellation Cascade (SAG-003) — full sequence

**Answers:** When a user cancels, what actually happens, and who emits the canonical cancellation?
**Sources:** `30_Sagas` SAG-003; `11_BC_Recovery` CMD-REC-001..003; `15_BC_Financial` CMD-FIN-007.
**What to look for:** Recovery is the *only* emitter of `BookingCancelled` (L1). The Reliability observation is a signal, not a profile write — DisputeCase can reverse it later.

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

## 7. Replacement Search (SAG-004) — full sequence

**Answers:** How does Playo fill a vacated seat automatically? Where does subsidy get recorded?
**Sources:** `30_Sagas` SAG-004; `11_BC_Recovery` CMD-REC-005/006; `24_BC_Matchmaking` CMD-MMK-004; POL-REC-002 (L14), POL-REC-003.
**What to look for:** The SubsidyLedger append is the only place the subsidy decision is persisted — recomputation is a bug.

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

## 8. Session statechart (capacity sub-states)

**Answers:** What are the legal Session states, and how do the capacity counters move?
**Sources:** `10_BC_Coordination` §STATECHART rows for Session; INV-COR-002..005.
**What to look for:** `Confirmed` has internal Filling/Full sub-states — the statechart tracks saturation without needing a flag field.

```mermaid
stateDiagram-v2
    [*] --> Scheduled : CMD-COR-004<br/>timeSlot held by saga

    state Scheduled {
        [*] --> AwaitingQuorum
        AwaitingQuorum --> QuorumMet : confirmed ≥ capacity.min<br/>[CMD-COR-008]
    }

    Scheduled --> Confirmed : ConfirmSession<br/>[guard: confirmed ≥ min]

    state Confirmed {
        [*] --> Filling
        Filling --> Full : confirmed = capacity.max
        Full --> Filling : ReleaseSeat<br/>[hold/conf exists]
        Filling --> Filling : HoldSeat/ConfirmSeat/ReleaseSeat<br/>[INV-COR-002]
    }

    Confirmed --> Started : CMD-COR-012 StartMatch<br/>[∀ user has CheckIn · INV-COR-004]
    Started --> Completed : CMD-COR-013 CompleteMatch<br/>[completedAt > startedAt]

    Scheduled --> Cancelled : Recovery.SessionCancelled<br/>[L1 · INV-COR-005]
    Confirmed --> Cancelled : Recovery.SessionCancelled

    Completed --> [*]
    Cancelled --> [*]
```

---

## 9. Payment statechart — Intent / Attempt / Payment

**Answers:** What's the retry/failover story? Why does the split earn its keep?
**Sources:** `15_BC_Financial` §STATECHART; INV-FIN-001..003.
**What to look for:** The Intent remains `Confirmed` across Failed Attempts — retry is a new Attempt, not a new Intent. This is why the split matters.

```mermaid
stateDiagram-v2
    direction LR
    state PaymentIntent {
        [*] --> I_Created : CreatePaymentIntent<br/>[amount>0]
        I_Created --> I_Confirmed : ConfirmPaymentIntent<br/>[method ∈ options]
        I_Confirmed --> I_Fulfilled : winning Attempt Captured<br/>[INV-FIN-003]
        I_Confirmed --> I_Cancelled : saga/user cancel
    }

    state PaymentAttempt {
        [*] --> A_Initiated : BeginPaymentAttempt<br/>[Intent Confirmed · no active]
        A_Initiated --> A_Authorized : AuthorizeAttempt<br/>[PSP ok]
        A_Authorized --> A_Captured : CaptureAttempt
        A_Initiated --> A_Failed : FailAttempt
        A_Authorized --> A_Failed : FailAttempt
        A_Failed --> [*]
        note right of A_Failed
          Intent stays Confirmed;
          may start new Attempt
          (retry / PSP failover)
        end note
    }

    state Payment {
        [*] --> P_Captured : on winning Attempt
        P_Captured --> P_Refunded : Σrefunds = amount<br/>[INV-FIN-004]
    }

    I_Confirmed --> A_Initiated : spawns
    A_Captured --> P_Captured : promotes
```

---

## 10. Cancellation & Dispute composite statechart

**Answers:** How does a cancellation actually progress? When does a dispute close?
**Sources:** `11_BC_Recovery` §STATECHART for CancellationCase and DisputeCase; INV-REC-001..007.
**What to look for:** Both machines are guaranteed to close — the Dispute can exit via adjudicator resolution *or* window expiry (defaults to Void). Never stuck.

```mermaid
stateDiagram-v2
    state CancellationCase {
        [*] --> Open : OpenCancellationCase<br/>[unique per target+initiator]
        Open --> ReadyToClose : RefundDecision(None)
        Open --> AwaitingRefund : RefundDecision(Full|Partial)
        AwaitingRefund --> ReadyToClose : RefundIssued observed<br/>[amount matches · INV-REC-002]
        ReadyToClose --> Closed : EmitCanonicalCancellation<br/>[INV-REC-003 · L1]
        Closed --> [*]
    }

    state DisputeCase {
        [*] --> Raised : OpenDisputeCase<br/>[raisedBy ∈ parties · window open]
        Raised --> UnderReview : SubmitDisputeEvidence
        Raised --> Resolved : ResolveDisputeCase
        UnderReview --> Resolved : ResolveDisputeCase<br/>[adjudicator ∧ window valid]
        Raised --> Expired : window expired<br/>[default Void]
        UnderReview --> Expired : window expired
        Resolved --> [*]
        Expired --> [*]
    }

    note right of DisputeCase
      Resolved may reverse:
      · a prior trust observation (EVT-REC-012)
      · a refund decision (edge case → manual)
      · annotate a revealed PeerReview (L15)
    end note
```

---

## 11. PeerReview statechart — sealing window

**Answers:** How is retaliatory rating blocked? When does a review become visible?
**Sources:** `22_BC_Community` §STATECHART for PeerReview; INV-COM-004, INV-COM-005; L15.
**What to look for:** Content is locked at Seal. Reveal is the earlier of: reciprocal seal, or session end + 14d. Nothing else.

```mermaid
stateDiagram-v2
    [*] --> Drafted : CMD-COM-008 DraftPeerReview<br/>[both attended · unique(from,to,session)]
    Drafted --> Sealed : CMD-COM-009 SealPeerReview<br/>[submitter = fromUserId]
    note right of Sealed
      L15: content IMMUTABLE after Sealed
      Only annotation via DisputeResolved
    end note
    Sealed --> Revealed : CMD-COM-010 RevealPeerReview<br/>[reciprocal sealed<br/>OR session.end + 14d]
    Revealed --> [*]
```

---

## 12. Event flow / ownership graph

**Answers:** Who owns each canonical cross-BC event, and who consumes it?
**Sources:** Every BC sheet's §EVENTS (owner column) and §CONSUMES.
**What to look for:** Thick double-arrows = canonical cancellations, all originating from Recovery. Dashed = DeviationRequested, all terminating at Recovery. The visual shape *is* L1.

```mermaid
flowchart LR
    COR((Coordination))
    FIN((Financial))
    INV((Inventory))
    PRC((Pricing))
    PRT((Partner))
    HOS((Hosting))
    COM((Community))
    GAM((Gamification))
    MMK((Matchmaking))
    TRS((Trust/S))
    TRR((Trust/R))
    TRF((Trust/F))
    TRC((Trust/C))
    REC((Recovery))

    COR -- SessionScheduled --> INV
    COR -- SessionScheduled --> PRC
    COR -- BookingCreated --> FIN
    COR -- MatchCompleted --> TRS
    COR -- MatchCompleted --> TRR
    COR -- MatchCompleted --> GAM
    COR -- MatchCompleted --> MMK
    COR -- CheckInRecorded --> TRR

    FIN -- PaymentCaptured --> COR
    FIN -- PaymentCaptured --> PRT
    FIN -- RefundIssued --> PRT
    FIN -- RefundIssued --> TRF
    FIN -- BNPLCreated --> TRF
    FIN -- BNPLSettled --> TRF

    INV -- TimeSlotHeld/Confirmed/Released --> COR
    INV -. VenueUnavailable .-> REC

    PRC -- PriceSnapshotSealed --> COR
    PRT -- PartnerKYCCompleted --> INV
    PRT -- CommissionAgreed --> PRC

    HOS -. HostCancelled .-> REC
    COR -. GameDeviationRequested .-> REC
    COR -. BookingDeviationRequested .-> REC
    FIN -. PaymentDeviationRequested .-> REC
    FIN -. BNPLDefaultRequested .-> REC

    REC == canonical BookingCancelled ==> COR
    REC == canonical SessionCancelled ==> COR
    REC == canonical SessionCancelled ==> INV
    REC == canonical GameAbandoned ==> COR
    REC == canonical PaymentFailed ==> COR
    REC == canonical BNPLDefaulted ==> FIN
    REC == canonical BNPLDefaulted ==> TRF

    REC -- NoShowCaseOpened --> TRR
    REC -- DisputeResolved --> TRS
    REC -- DisputeResolved --> TRR
    REC -- DisputeResolved --> TRF
    REC -- DisputeResolved --> TRC
    REC -- ReplacementCaseOpened --> MMK

    COM -- PeerReviewRevealed --> TRS
    COM -- PeerReviewRevealed --> TRC
    COM -- PeerReviewRevealed --> GAM
    COM -- PlayPalConfirmed --> TRC
```

---

## 13. Trust composition fan-in

**Answers:** How can there be four profiles but no composed score? Where does a trust decision actually live?
**Sources:** Trust BC `§CONSUMES` whitelists in sheets 16–19; Trust Composition Policy POL-COR-002; L4.
**What to look for:** Profiles accumulate from events. The Policy reads them per use case, returns a decision, and stores nothing. Different use cases pick different subsets of profiles.

```mermaid
flowchart LR
    subgraph Events[Events · only those in whitelists]
        direction TB
        E1[MatchCompleted]
        E2[CheckInRecorded]
        E3[NoShowCaseOpened]
        E4[BookingCancelled]
        E5[PaymentCaptured]
        E6[RefundIssued]
        E7[BNPLCreated/Settled/Defaulted]
        E8[PeerReviewRevealed]
        E9[PlayPalConfirmed/SquadMemberJoined]
        E10[DisputeResolved]
    end

    subgraph Profiles[4 Trust Profiles · append-only · no composed score · L4]
        direction TB
        PS[SkillProfile<br/>mu/sigma per sport]
        PR[ReliabilityProfile<br/>rate, sampleSize]
        PF[FinancialTrustProfile<br/>paymentDiscipline, bnplRate]
        PC[CommunityStanding<br/>peerAggregate, connections]
    end

    E1 --> PS
    E8 --> PS
    E1 --> PR
    E2 --> PR
    E3 --> PR
    E4 --> PR
    E10 --> PR
    E5 --> PF
    E6 --> PF
    E7 --> PF
    E10 --> PF
    E8 --> PC
    E9 --> PC
    E10 --> PC

    subgraph Composition[Trust Composition Policy · STATELESS · use-case-bound]
        direction TB
        UC1[Use case: match-join eligibility]
        UC2[Use case: BNPL eligibility]
        UC3[Use case: replacement candidacy]
        UC4[Use case: host delegation]
    end

    PS -. reads .-> UC1
    PR -. reads .-> UC1
    PS -. reads .-> UC3
    PR -. reads .-> UC3
    PF -. reads .-> UC2
    PR -. reads .-> UC4
    PC -. reads .-> UC4

    subgraph Consumers[Decision callers]
        direction TB
        D1[Coordination HoldSeat]
        D2[Financial CreateBNPL]
        D3[Recovery Replacement]
        D4[Hosting Delegation]
    end

    UC1 --> D1
    UC2 --> D2
    UC3 --> D3
    UC4 --> D4

    classDef ev fill:#fff,stroke:#888
    classDef pr fill:#F0AD4E,stroke:#8A5A12
    classDef co fill:#5BC0DE,stroke:#1F5A73
    classDef dc fill:#D9534F,stroke:#7A1F1C,color:#fff
    class E1,E2,E3,E4,E5,E6,E7,E8,E9,E10 ev
    class PS,PR,PF,PC pr
    class UC1,UC2,UC3,UC4 co
    class D1,D2,D3,D4 dc
```

---

## 14. Saga × BC coupling matrix

**Answers:** Which BCs are most coupled? Which sagas cross the most boundaries? Where is Recovery actually the orchestrator?
**Sources:** `30_Sagas` Orchestrator column.

| Saga | COR | REC | FIN | INV | PRC | MMK | HOS | TRR | TRS | TRF | COM | GAM |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **SAG-001 Game-to-Session** | **O** | · | · | P | P | · | · | · | · | · | · | · |
| **SAG-002 Booking** | **O** | C | P | · | · | · | · | · | · | · | · | · |
| **SAG-003 Player Cancel** | C | **O** | P | · | · | · | · | obs | · | · | · | · |
| **SAG-004 Replacement** | P | **O** | P | · | · | P | · | · | · | · | · | · |
| **SAG-005 Host Cancel** | C | **O** | P | P | · | · | T | obs | · | · | · | · |
| **SAG-006 Venue Cancel** | C | **O** | P | T | · | · | · | · | · | · | · | · |
| **SAG-007 Gamification** | T | · | · | · | · | · | · | · | · | · | T | **O (chor)** |
| **SAG-008 Yield/Subsidy** | · | · | · | · | **O** | · | · | · | · | · | · | · |
| **SAG-009 Community→Trust** | · | · | · | · | · | · | · | · | · | · | **O (chor)** | · |
| **SAG-010 BNPL Default** | C | **O** | T | · | · | · | · | · | · | obs | · | · |
| **SAG-011 Waitlist Promote** | **O** | · | P | · | · | · | · | · | · | · | · | · |
| **SAG-012 Series Occurrence** | **O** | · | · | P | P | · | · | · | · | · | · | · |
| **SAG-013 Dispute** | · | **O** | P\* | · | · | · | · | rev | rev | rev | rev | · |

**Legend:** **O** = Orchestrator · P = Participant (command call) · T = Trigger source · C = Compensation target · obs = observation emitted · rev = observation reversal possible · P\* = manual-escalate edge · chor = choreographed.

**Visible hotspots:**
- **Recovery orchestrates 6 of 13 sagas** and participates in most of the rest.
- **Financial** is a participant in nearly every money-critical saga but orchestrates none (by design — it owns state, not choreography).
- **Trust BCs never orchestrate**, only observe or reverse. This is L4 made visual.
- The **Core block (COR + REC + MMK)** touches every other BC except Training. Training is the most loosely coupled.

---

## How to keep these in sync with the workbook

- The workbook is the source of truth. These diagrams are a *view*.
- When a BC sheet changes, the relevant diagram is stale. Re-render from the workbook, don't edit in place.
- If a diagram shows an edge that isn't in any `§CONSUMES` or `§COMMANDS.calls` row, the workbook is the authority — delete the edge from the diagram.
- When adding a new BC, the Strategic Context Map, the Event Flow, and the Saga Matrix are the three that must be updated first. Statecharts only need to be added when the aggregate has branching behaviour worth showing.
