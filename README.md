# Collinson_test
## Priority Pass (PP) × Global Taxi App — Solution Deliverables

## Repository Structure
.
├── README.md
├── diagrams/
│   ├── context.mmd
│   ├── container.mmd
│   ├── journey_orchestrator-component.mmd
│   ├── context.png
│   ├── container.png
│   └── journey_orchestrator.png
├── ui-wireframes
├── tshirt-sizes
├── Omissions & Trade-offs
└── PCI Compliance

## Solution Overview (C4 Summary)

**Goal**  
Provide PP members a seamless way to plan & book airport ground transport and see **recommended departure times** (flight + traffic), while enabling **two-way inventory sharing** (taxi app ↔ PP content).

**Level 1 — Context**  
- **Actors**: Traveler (PP member), Taxi Partner, Flight Data provider, Maps/Traffic API.  
- **Systems**: PP App ↔ PP Platform ↔ Taxi Partner Platform; PP integrates with flight/traffic for departure advice.

**Level 2 — Containers (PP side)**  
- **PP Mobile App**: Journey planning, flight info, ETAs, booking, notifications.  
- **API Gateway (+WAF)**: Routing, policy enforcement, security controls.  
- **Journey Orchestrator**: Availability, pricing, reserve/confirm/cancel.  
- **Flight & Traffic Service**: Aggregates flight status + traffic to compute recommended leave time.  
- **Partner Integration Layer**: Publishes PP content to the taxi partner and ingests partner availability/content.  
- **Event Bus**: Booking/telemetry/audit topics.  
- **Data Stores**: User bookings/entitlements/reservations.

**Level 3 — Journey Orchestrator Components**  
- **Adapters**: Ride Partner (quotes & bookings), Flight, Traffic.

---

## ui-wireframes (incomplete)

- Five screens laid out left→right (Home → Link Flight → Ride Options → Booking Confirmation → Taxi app cross-surface).
- Labeled step arrows (1–4) showing the flow.
- Numbered callouts (①–⑤) with notes explaining UX intent and which backend services are involved (BFF, Orchestrator, Payments, Inventory Sharing).
- Clear UI elements (inputs, buttons, cards, chips) to convey content hierarchy and value props (recommended departure, ETA/price, PP benefits).

---

## tshirt-sizes

| Container                     | Size | Notes                                                       |
| ----------------------------- | :--: | ----------------------------------------------------------- |
| **PP Mobile App**             |   M  | Two new flows (flight linking, ride booking), feature flags |
| **API Gateway (+ WAF)**       |   S  | Mostly configuration, policies, and dashboards              |
| **Booking Orchestrator**      |   L  | Partner adapters, retries, webhooks, idempotency            |
| **Flight & Traffic Service**  |   M  | External API aggregation, depart-time logic, caching        |
| **Inventory Sharing Service** |   L  | API contracts + content exchange                            |
| **Event Bus**                 |   M  | Topics, schemas, retention, DLQs                            |

---

## Architectural Choices & Rationale

- **API Gateway (+ WAF)**  
  *Centralised authZ, rate limiting, and header policies; OWASP mitigations; partner-safe exposure with mTLS.*

- **Adapter Pattern for Partners**  
  *Clean separation from partner APIs; enables adding taxi partners behind stable interfaces; supports retries, backoff, and circuit breakers.*

> Optional (not shown above but compatible): a **BFF/App API** to tailor payloads for mobile clients and isolate the app from backend churn.

---
## PCI Compliance (Priority Pass Perspective)

**Assumption**: The Taxi Partner is **Merchant of Record**; PP does **not** facilitate payment → PP remains **out of PCI scope**.

**If PP facilitates payment (enhancement path):**
- Introduce a **Payments Facade** that:
  - Minimises PCI scope; unifies PSP interactions; standardises webhook verification.
  - Uses **hosted fields/SDK** or **payment intents** with a **PCI-certified processor**.
  - Stores only **PSP tokens/intent IDs** (no PAN/CVV in databases, caches, logs, or events).

---
## Omissions & Trade-offs

**MVP Scope**
- Single taxi partner; multi-partner support via additional adapters deferred.
- Departure-time logic simplified (flight + traffic). Future: security wait times, terminal walking distances, airline cut-offs, disruption data.
- Identity & Consent Service: OIDC login, account linking, consent registry.

**Event-driven backbone**
- Saga Cordinator - Taxi booking spans multiple steps (quote → reserve → confirm → capture). Sagas support compensations on failure (e.g., auto-cancel hold, refund).
- Loose coupling and reliable out-of-band processing (notifications, analytics, audit). 

**Resilience & Scale**
- Diagrams show one region for brevity; production should target **multi-region active/active** (stateless services, replicated DB, global pub/sub).
- Webhook retries & DLQs assumed; add replay tooling, schema registry, and contract testing.

**Experience Gaps**
- Accessibility and offline modes partially covered in the prototype; require full **WCAG** review and robust offline caches.
- No tipping/promo/receipt rendering in MVP (could live in partner app or be added later with a Payments Facade).

**Data Sharing**
- Inventory sharing is **consent-gated**; implement fine-grained scopes and purpose limitation before GA.

**Operational Complexity**
- Event-driven flows + orchestration add ops surface; mitigate with clear SLOs, tracing, and runbooks.

**Latency vs Freshness**
- Aggressive caching reduces latency/cost but risks staleness; tune TTLs and add cache-busting on critical paths.

---
