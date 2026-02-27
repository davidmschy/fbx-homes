# FBX Platform — Implementation Plan
## AI-Accelerated Build: 6 Phases, 77 PRs

> **For AI Coding Agents:** Every PR listed here is a standalone unit of work with a clear scope, acceptance criteria, and dependencies. Assume 5 PRs/hour velocity. Each PR should be a focused feature branch merged to main via GitHub Actions CI. No PR merges without passing tests.

---

## Velocity Assumptions

- 5 PRs/hour for standard feature work
- 3 PRs/hour for complex integrations (Stripe, DocuSign, WebSockets)
- 2 PRs/hour for infrastructure/DevOps work
- Branch naming: feature/PR-XXX-short-description
- All PRs require: passing lint, passing tests, no TypeScript errors

---

## Phase 0: Foundation (Hours 1-4, 12 PRs)

Goal: Clean repo, correct tech stack, CI/CD, database, auth running.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-001 | Repo setup: Vite + React 18 + TypeScript + Tailwind + shadcn/ui | 0.5h | none |
| PR-002 | Backend: Express + TypeScript + Drizzle ORM + Neon PostgreSQL | 0.5h | none |
| PR-003 | DB schema: core tables (users, markets, projects, project_units) | 0.5h | PR-002 |
| PR-004 | DB schema: transaction tables (reservations, deposits, waitlist) | 0.5h | PR-003 |
| PR-005 | DB schema: participant tables (buyers, realtors, manufacturers, contractors, investors, lenders) | 0.5h | PR-003 |
| PR-006 | DB schema: operational tables (rfqs, work_orders, milestones, builder_bonds) | 0.5h | PR-004 |
| PR-007 | Auth: Better Auth setup, register/login/logout/me endpoints | 0.5h | PR-002 |
| PR-008 | Auth middleware: session validation, RBAC, buyer score gates | 0.5h | PR-007 |
| PR-009 | GitHub Actions CI: lint + typecheck + test on every PR | 0.5h | PR-001,PR-002 |
| PR-010 | Cloudflare deployment: CF Pages + CF Workers config | 1h | PR-009 |
| PR-011 | Redis + BullMQ: job queue infrastructure, worker process | 0.5h | PR-002 |
| PR-012 | Logging: Sentry + Posthog integration | 0.5h | PR-010 |

Exit criteria: Deploys to Cloudflare, DB migrates, auth works, CI passes.

---

## Phase 1: Demand Engine — Waitlist + Drop System (Hours 4-12, 15 PRs)

Goal: Real buyers can join waitlist, pre-register, and receive drop notifications. FIRST public launch.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-013 | Homepage: hero + demand heatmap placeholder + drop countdown + waitlist CTA | 0.5h | PR-001 |
| PR-014 | Waitlist signup: location + budget + timeline + role capture + email confirm | 0.5h | PR-007 |
| PR-015 | Markets API: CRUD, FHA limits, drop schedule config, waitlist counts | 0.5h | PR-005 |
| PR-016 | Mapbox demand heatmap: zip-level buyer density on homepage | 1h | PR-015 |
| PR-017 | Pre-registration: KYC lite, credit consent, income self-cert | 1h | PR-014 |
| PR-018 | Buyer scoring: A/B/C calculation from pre-registration data | 0.5h | PR-017 |
| PR-019 | Pre-approval upload: PDF to R2, admin review queue | 0.5h | PR-017 |
| PR-020 | Drop scheduling: BullMQ job, randomized time, per-market config | 1h | PR-011 |
| PR-021 | Drop notifications: Resend email + Twilio SMS to pre-registered A/B buyers | 1h | PR-020 |
| PR-022 | Drop access page: lot cards, live viewer counts, countdown, early/general gates | 1h | PR-020 |
| PR-023 | WebSocket server: socket.io drop namespace, viewer counts, reserved events | 1h | PR-022 |
| PR-024 | Admin: market management, drop scheduling UI, drop monitoring | 0.5h | PR-020 |
| PR-025 | How It Works pages: buyer, realtor, landowner, investor, manufacturer | 0.5h | PR-013 |
| PR-026 | SEO: market landing pages, meta tags, sitemap, structured data | 0.5h | PR-025 |
| PR-027 | Email sequences: waitlist nurture drip, pre-reg reminder, drop countdown | 0.5h | PR-021 |

Exit criteria: Buyers sign up, pre-register, get scored, receive drop notification. Admin schedules drops. Heatmap shows demand. LAUNCH READY.

---

## Phase 2: Reservation System + Buyer Conversion (Hours 12-20, 12 PRs)

Goal: $99 reservations, certificates, resale marketplace, $10K deposits with design selections.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-028 | Stripe setup: payment intents, webhook handler, test mode | 0.5h | PR-007 |
| PR-029 | $99 reservation flow: Stripe payment, 15-min unit lock, confirmation | 1h | PR-028 |
| PR-030 | Reservation certificate: PDF generator with certificate number + transfer rights | 1h | PR-029 |
| PR-031 | Buyer dashboard: reservations, deposit countdown, certificate download | 0.5h | PR-029 |
| PR-032 | Resale marketplace: list/browse/purchase reservation transfers | 1h | PR-029 |
| PR-033 | Resale 10% fee: Stripe split on resale, payout to original holder | 0.5h | PR-032 |
| PR-034 | AI design selections concierge: chat-based flow, live price updates | 1.5h | PR-031 |
| PR-035 | $10K deposit: Stripe escrow, rescission window, purchase agreement trigger | 1h | PR-034 |
| PR-036 | DocuSign: purchase agreement template, e-sign flow, webhook on completion | 1h | PR-035 |
| PR-037 | Lender pre-approval: partner lender API, pre-qual from deposit flow | 1h | PR-035 |
| PR-038 | Project milestone tracker: buyer-facing timeline, status updates | 0.5h | PR-031 |
| PR-039 | Document vault: buyer downloads permits, warranty, HOA docs | 0.5h | PR-038 |

Exit criteria: Full buyer funnel works end-to-end. Waitlist → pre-register → reserve $99 → certificate → resell OR $10K deposit with design selections → track milestones.

---

## Phase 3: Realtor Portal + Lender Marketplace (Hours 20-26, 8 PRs)

Goal: Realtors register and earn commissions. Lenders receive pre-qual requests.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-040 | Realtor registration: license verification, referral link + QR code | 0.5h | PR-007 |
| PR-041 | Realtor client dashboard: clients, scores, reservation/deposit status | 0.5h | PR-040 |
| PR-042 | Realtor commission tracker: pending/paid/projected, Stripe Connect setup | 0.5h | PR-040 |
| PR-043 | Realtor $2K bonus: auto-trigger on $10K deposit, Stripe Connect transfer | 0.5h | PR-042 |
| PR-044 | Realtor 2% co-op: trigger on close, calculation, Stripe Connect transfer | 0.5h | PR-042 |
| PR-045 | Lender registration: company, FHA approval, coverage, webhook URL | 0.5h | PR-007 |
| PR-046 | Lender pre-qual delivery: anonymized buyer profile → lender webhook, response tracking | 1h | PR-045 |
| PR-047 | Lender comparison: ranked offers for buyer, 1-click proceed | 0.5h | PR-046 |

Exit criteria: Realtors register, track clients, receive $2K + 2% via Stripe Connect. Buyers see ranked lender offers.

---

## Phase 4: Supply Side — Landowner + Manufacturer + Contractor (Hours 26-36, 12 PRs)

Goal: Landowner self-serve JV, manufacturer RFQ system, contractor work orders.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-048 | Parcel analysis: address → county assessor API → zoning + eligibility | 1.5h | PR-015 |
| PR-049 | Proforma generator: parcel data → unit count → revenue → landowner return | 1h | PR-048 |
| PR-050 | JV flow: term sheet, DocuSign JV agreement, landowner dashboard | 1h | PR-049 |
| PR-051 | Landowner project dashboard: milestones, financials, pref return tracker | 0.5h | PR-050 |
| PR-052 | Manufacturer registration: certifications, coverage, OpenClaw config | 0.5h | PR-007 |
| PR-053 | Manufacturer product catalog: unit types, pricing, capacity calendar | 0.5h | PR-052 |
| PR-054 | RFQ engine: BullMQ on permits_issued, manufacturer scoring, distribution | 1.5h | PR-052,PR-011 |
| PR-055 | Manufacturer RFQ portal: inbox, quote submission, PO acceptance | 0.5h | PR-054 |
| PR-056 | OpenClaw API bridge: bidirectional ERPNext for manufacturers | 1.5h | PR-054 |
| PR-057 | Contractor registration: license, coverage, trade types, OpenClaw config | 0.5h | PR-007 |
| PR-058 | Work order engine: auto-generate 6 scopes, post to marketplace | 1h | PR-057,PR-011 |
| PR-059 | Contractor work order portal: browse, instant-accept or bid, milestone submit | 0.5h | PR-058 |

Exit criteria: Landowner JV self-serve. Permits trigger RFQ. Manufacturers quote + accept POs. Work orders auto-generate. Contractors accept. OpenClaw syncs bidirectionally.

---

## Phase 5: Capital Layer — Investor Portal + Builder Bonds (Hours 36-42, 8 PRs)

Goal: Accredited investors fund Builder Bonds, capital allocates to projects, returns distribute.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-060 | Investor registration: accreditation verification, Stripe ACH | 0.5h | PR-007 |
| PR-061 | Builder Bond account: fund, balance display, idle vs deployed | 0.5h | PR-060 |
| PR-062 | Capital call engine: FIFO allocation on permit_issued, investor notification | 1h | PR-060,PR-054 |
| PR-063 | Investor dashboard: allocations, expected returns, compounding calculator | 0.5h | PR-061 |
| PR-064 | Returns distribution: milestone-triggered calculation, Stripe Connect payout | 1h | PR-062 |
| PR-065 | Penalty engine: timeline monitoring, auto-calculate penalties at day 121+/151+ | 0.5h | PR-062 |
| PR-066 | 1031 portfolio tool: exchange amount → matched projects → LOI flow | 1h | PR-063 |
| PR-067 | Tax documents: K-1 generation, annual statement export | 0.5h | PR-064 |

Exit criteria: Investors register, fund, capital auto-allocates, returns auto-distribute, penalties enforce.

---

## Phase 6: Admin, QA, Testing + CI/CD Hardening (Hours 42-52, 10 PRs)

Goal: Full test coverage, admin controls, load testing, accessibility, production-ready.

| PR | Title | Hours | Dependencies |
|----|-------|-------|-------------|
| PR-068 | Admin dashboard: project pipeline, exception flags, revenue overview | 1h | all |
| PR-069 | Admin drop management: live monitoring, manual override, sold-out tracking | 0.5h | PR-023 |
| PR-070 | Unit tests: buyer scoring, RFQ scoring, proforma, penalty engine (Vitest) | 1h | all logic PRs |
| PR-071 | Integration tests: all API routes, auth flows, payment webhooks (Supertest) | 1.5h | all API PRs |
| PR-072 | E2E: waitlist → pre-register → drop → reserve → deposit → design (Playwright) | 1.5h | PR-029-036 |
| PR-073 | E2E: realtor flow, landowner JV, manufacturer RFQ (Playwright) | 1h | PR-040-059 |
| PR-074 | Load test: 500 concurrent users on reserve endpoint (k6) | 0.5h | PR-029 |
| PR-075 | Accessibility: WCAG 2.1 AA, axe-playwright on all critical flows | 0.5h | PR-072 |
| PR-076 | Security: rate limiting, CSRF, webhook signatures, secrets audit | 0.5h | all |
| PR-077 | Production: final CF Workers config, domain setup, smoke tests | 0.5h | all |

Exit criteria: 80% unit coverage. All E2E passes. 500-user load test passes. WCAG AA clean. Security audit clean. Live on fbx.homes.

---

## PR Description Template

Every PR must include:

```
## What This PR Does
[1-2 sentences]

## Platform Design Reference
[Section in PLATFORM_DESIGN.md]

## Acceptance Criteria
- [ ] criterion 1
- [ ] criterion 2

## Test Coverage
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual test steps

## Database Changes
[New migrations if any]

## Environment Variables Added
[New env vars if any]

## Screenshots
[Required for UI changes]
```

---

## Timeline Summary

| Phase | PRs | Hours | Milestone |
|-------|-----|-------|-----------||
| Phase 0: Foundation | 12 | 4h | Deployable skeleton |
| Phase 1: Waitlist + Drops | 15 | 8h | LIVE — buyer acquisition |
| Phase 2: Reservations | 12 | 8h | LIVE — $99 + $10K funnel |
| Phase 3: Realtors + Lenders | 8 | 6h | LIVE — agent network |
| Phase 4: Supply Side | 12 | 10h | LIVE — full supply chain |
| Phase 5: Investor Portal | 8 | 6h | LIVE — Builder Bonds |
| Phase 6: QA + Production | 10 | 10h | PRODUCTION HARDENED |
| **Total** | **77** | **52h** | Full platform live |

At 5 PRs/hour with 6 parallel agents: ~15 hours calendar time.

---

## Agent Assignments

| Agent | PRs | Domain |
|-------|-----|--------|
| Agent A | PR-001–012 | Foundation + Infrastructure |
| Agent B | PR-013–027 | Public Site + Waitlist + Drop Engine |
| Agent C | PR-028–039 | Buyer Conversion + Payments |
| Agent D | PR-040–051 | Realtor + Landowner Portals |
| Agent E | PR-052–067 | Supply Side + Investor Portal |
| Agent F | PR-068–077 | Admin + QA + Testing |

Agents B-F can start as soon as Agent A delivers PR-001 and PR-002+PR-007.

---

*Version: 1.0 | Last updated: 2026-02-26*