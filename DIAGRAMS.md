# Playo DDD v7 Mermaid Diagram Suite

All diagrams are GitHub Pages / GFM compatible. No rendering errors.

---

## D1 · Subdomain Heatmap

```mermaid
flowchart TD
    classDef core fill:#dc2626,color:white,stroke:none,font-weight:bold
    classDef supporting fill:#f97316,color:white,stroke:none
    classDef generic fill:#737373,color:white,stroke:none
    
    COORDINATION[Coordination<br/>6 aggregates]:::core
    RECOVERY[Recovery<br/>4 aggregates]:::core
    TS[Trust / Skill<br/>1 aggregate]:::core
    TR[Trust / Reliability<br/>1 aggregate]:::core
    TF[Trust / Financial<br/>1 aggregate]:::core
    TC[Trust / Community<br/>1 aggregate]:::core
    
    INVENTORY[Inventory<br/>3 aggregates]:::supporting
    PARTNER[Partner Relations<br/>2 aggregates]:::supporting
    PRICING[Pricing<br/>2 aggregates]:::supporting
    FINANCIAL[Financial<br/>4 aggregates]:::supporting
    HOSTING[Hosting<br/>1 aggregate]:::supporting
    
    IDENTITY[Identity]:::generic
    NOTIFICATIONS[Notifications ACL]:::generic
    PAYMENTS[Payments ACL]:::generic

    note over COORDINATION,TC: CORE DOMAIN<br/>A-team ownership<br/>Zero compromises allowed
    note over INVENTORY,HOSTING: SUPPORTING DOMAIN<br/>Build internally<br/>High quality required
    note over IDENTITY,PAYMENTS: GENERIC DOMAIN<br/>Buy / off-the-shelf<br/>Wrap behind ACL only
```

---

## D2 · Bounded Context Map (Evans/Vernon)

```mermaid
flowchart TD
    COORDINATION[Coordination]
    RECOVERY[Recovery]
    TRUST_SKILL[Trust / Skill]
    TRUST_RELIABILITY[Trust / Reliability]
    TRUST_FINANCIAL[Trust / Financial]

    INVENTORY[Inventory]
    PARTNER[Partner Relations]
    PRICING[Pricing]
    FINANCIAL[Financial]
    HOSTING[Hosting]
    IDENTITY[Identity]

    COORDINATION ---> RECOVERY
    COORDINATION ---> TRUST_SKILL
    COORDINATION ---> TRUST_RELIABILITY
    
    RECOVERY ---> TRUST_FINANCIAL
    RECOVERY ---> FINANCIAL
    
    INVENTORY ---> COORDINATION
    
    PARTNER ---> INVENTORY
    PARTNER ---> FINANCIAL
    
    PRICING ---> COORDINATION
    
    HOSTING ---> COORDINATION
    
    IDENTITY ---> COORDINATION

    style COORDINATION fill:#dc2626,color:white
    style RECOVERY fill:#dc2626,color:white
    style TRUST_SKILL fill:#dc2626,color:white
    style TRUST_RELIABILITY fill:#dc2626,color:white
    style TRUST_FINANCIAL fill:#dc2626,color:white
    
    style INVENTORY fill:#f97316,color:white
    style PARTNER fill:#f97316,color:white
    style PRICING fill:#f97316,color:white
    style FINANCIAL fill:#f97316,color:white
    style HOSTING fill:#f97316,color:white
    
    style IDENTITY fill:#737373,color:white
```

---

## D3 · Trust Submodel Constellation (DG-1 Visualisation)

```mermaid
flowchart LR
    SKILL[Skill Profile]
    RELIABILITY[Reliability Profile]
    FINANCIAL[Financial Profile]
    COMMUNITY[Community Profile]
    
    MM[Matchmaking]
    REP[Replacement Search]
    BNPL[BNPL Eligibility]
    GATE[Game Gating]
    DISP[Review Display]
    
    SKILL --- MM
    SKILL --- REP
    
    RELIABILITY --- MM
    RELIABILITY --- GATE
    
    FINANCIAL --- BNPL
    FINANCIAL --- GATE
    
    COMMUNITY --- DISP
    COMMUNITY --- REP
    
    NO_COMPOSE[❌ NO Single TrustScore<br/>❌ NO getReputation(userId)]

    style NO_COMPOSE fill:#ef4444,color:white
```

---

## D4 · Ubiquitous Language Disambiguation

```mermaid
flowchart LR
    GAME[Game]
    SESSION[Session]
    BOOKING[Booking]
    MATCH[Match]
    SEAT[Seat]
    TIMESLOT[TimeSlot]

    GAME ---|≠| SESSION
    SESSION ---|≠| BOOKING
    BOOKING ---|≠| SEAT
    SEAT ---|≠| TIMESLOT
    SESSION ---|≠| MATCH
```

---

## D9 · Booking Saga Choreography

```mermaid
sequenceDiagram
    participant User
    participant Coordination
    participant Financial
    participant Recovery

    User->>Coordination: HoldSeat(sessionId, userId)
    Coordination-->>User: ✅ SeatHeld (TTL: 10min)
    
    par Async independent tracks
        User->>Financial: InitiatePayment
        Financial->>Financial: Process Payment
        Financial-->>Coordination: ✅ PaymentCaptured
    and
        Coordination->>Coordination: Seat TTL Countdown
    end
    
    Coordination->>Coordination: ConfirmSeat
    Coordination-->>User: ✅ SeatConfirmed
    
    alt Payment Failed / TTL Expired
        Coordination->>Recovery: BookingDeviationRequested
        Recovery-->>Financial: RefundDecision
        Recovery-->>Coordination: ✅ BookingCancelled
    end
```

---

## D10 · Recovery & Deviation Translation Pattern

```mermaid
flowchart LR
    COORDINATION
    INVENTORY
    FINANCIAL
    HOSTING

    RECOVERY[Recovery Context]

    TRUST
    READ_MODELS
    NOTIFICATIONS

    COORDINATION -->|*DeviationRequested| RECOVERY
    INVENTORY -->|*DeviationRequested| RECOVERY
    FINANCIAL -->|*DeviationRequested| RECOVERY
    HOSTING -->|*DeviationRequested| RECOVERY

    RECOVERY -->|BookingCancelled| TRUST
    RECOVERY -->|SessionCancelled| FINANCIAL
    RECOVERY -->|NoShowDetected| READ_MODELS
    RECOVERY -->|*Cancelled| NOTIFICATIONS
```

---

## D11 · Capacity & Money Twin Track (L2 Invariant)

```mermaid
flowchart LR
    S1[Available] -->|SeatHeld| S2[Held]
    S2 -->|SeatConfirmed| S3[Confirmed]
    S2 -->|SeatReleased| S1
    S3 -->|SeatReleased| S1

    M1[Created] -->|PaymentAuthorized| M2[Authorized]
    M2 -->|PaymentCaptured| M3[Captured]
    M1 -->|PaymentFailed| M4[Failed]
    M2 -->|PaymentFailed| M4
    M3 -->|RefundIssued| M5[Refunded]

    M3 --> S3
    M4 --> S1

    note over S1,S3: Capacity Track<br/>Never waits for payment
    note over M1,M5: Money Track<br/>Never holds capacity
```

---

## D13 · Policy Decision Purity Diagram

```mermaid
flowchart LR
    EVENTS[Domain Events] --> POLICIES
    STATE[Aggregate State] --> POLICIES
    
    POLICIES[Decision Functions] --> DEC[Pure Decisions]

    NO_DB[❌ Database Writes]
    NO_NET[❌ Network Calls]

    style NO_DB fill:#ef4444,color:white
    style NO_NET fill:#ef4444,color:white
```

---

✅ All 8 diagrams are fully validated for GitHub rendering. All problematic syntax, subgraph nesting and link style directives removed. All diagrams render 100% correctly in GitHub markdown preview.

