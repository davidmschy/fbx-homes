# FBX Platform — Complete System Design Document

> **For AI Coding Agents:** This document is the authoritative reference for how the FBX platform is architected, what every module does, how data flows between systems, and what decisions have already been made. Read this before writing any code. Every PR should reference the relevant section of this document.

---

## 1. What FBX Is

FBX (Factory Built Exchange) is a **weekly lot drop marketplace** for missing middle housing (duplexes, triplexes, fourplexes) built using modular/factory construction methods. It operates like a Supreme drop or Taylor Swift ticket sale — buyers pre-register, get notified of a weekly drop at a random time, compete to reserve lots for $99, then convert to a $10K deposit with design selections and full loan pre-approval.

The platform coordinates five participant types:
- **Buyers** — FHA-eligible homebuyers seeking $0-down new construction
- **Landowners** — Property owners who JV with FBX to develop their land
- **Manufacturers** — Modular home factories that bid on and fulfill build orders
- **Contractors** — Site work, foundation, utilities, finish trades
- **Investors** — Accredited investors funding construction via "Builder Bonds"
- **Realtors** — Licensed agents bringing pre-registered buyer clients

FBX makes money on development margin (build cost vs. sale price), investor capital fees, contractor marketplace fees, lender referral fees, and property management — without owning land or employing construction workers.

---

## 2. Core Business Rules

These rules are non-negotiable and must be enforced at the data layer, not just the UI:

1. **Only pre-registered buyers with Buyer Score A or B can access drops during the 2-hour early window.** Score C buyers see a "register to participate" gate.
2. **Drop time is randomized within a 4-hour window (8am–12pm PT) on Tuesdays.** The exact time is not revealed until the email/SMS fires.
3. **$99 reservation is charged immediately via Stripe.** It is refundable within 72 hours or transferable at any time via the resale marketplace.
4. **FBX takes 10% of all reservation resale transactions.**
5. **$10K deposit goes to Stripe escrow.** Non-refundable after 72-hour rescission window. Credited to purchase price at close.
6. **Realtor co-op: $2K paid at $10K deposit confirmation. 2% commission paid at close.** Realtors must be registered and linked to buyer before reservation.
7. **Builder Bond investors are called FIFO from the pool.** Capital sits in T-bill equivalent while idle, earning short-term yield.
8. **FBX guaranteed close timeline: 120 days from permit issuance.** Day 121-150: FBX pays 1.5%/month penalty. Day 151+: 2.5%/month.
9. **Landowner JV terms: 8% pref return on appraised land value + 7.5% of net profit.** FBX takes 5% development management fee + development margin.
10. **All FHA product must comply with HUD permanent foundation requirements** and be built to IRC or HUD code by HCD-certified manufacturers.

---

## 3. Tech Stack

### Frontend
- **Framework:** React 18 + TypeScript
- **Build Tool:** Vite
- **Styling:** Tailwind CSS + shadcn/ui component library
- **State Management:** TanStack Query (React Query) for server state, Zustand for local UI state
- **Forms:** React Hook Form + Zod validation
- **Routing:** React Router v6
- **Maps:** Mapbox GL JS
- **Payments:** Stripe.js
- **Real-time:** Socket.io-client
- **Charts:** Recharts

### Backend
- **Runtime:** Node.js 20 + TypeScript
- **Framework:** Express.js
- **ORM:** Drizzle ORM
- **Database:** PostgreSQL (Neon serverless)
- **Cache/Queue:** Redis + BullMQ
- **Auth:** Better Auth
- **File Storage:** Cloudflare R2
- **Email:** Resend + Loops
- **SMS:** Twilio
- **Payments:** Stripe
- **E-Signature:** DocuSign API
- **Real-time:** Socket.io

### Infrastructure
- **Hosting:** Cloudflare Workers + Pages
- **Database:** Neon PostgreSQL
- **Background Jobs:** BullMQ + Upstash Redis
- **CDN/Storage:** Cloudflare R2
- **CI/CD:** GitHub Actions
- **Monitoring:** Sentry + Posthog

### External Integrations
- County Assessor APIs (parcel data)
- Mapbox (visualization, geocoding)
- Stripe (all payments)
- DocuSign (agreements)
- Twilio (SMS)
- Resend (email)
- CalHFA API (DPA eligibility)
- ERPNext/OpenClaw (manufacturer + contractor bridge)
- MLS RETS/RESO API (lot listing ingestion)

---

## 4. Database Schema

### users
```sql
id uuid PRIMARY KEY
email text UNIQUE NOT NULL
phone text
full_name text
role text CHECK (role IN ('buyer','landowner','manufacturer','contractor','investor','realtor','admin'))
buyer_score text CHECK (buyer_score IN ('A','B','C'))
stripe_customer_id text
created_at timestamptz DEFAULT now()
updated_at timestamptz DEFAULT now()
```

### buyer_profiles
```sql
id uuid PRIMARY KEY
user_id uuid REFERENCES users(id)
location_preferences jsonb
home_type_preferences text[]
budget_min integer
budget_max integer
timeline text
pre_approval_status text
pre_approval_amount integer
pre_approval_lender text
pre_approval_expires_at date
income_self_certified integer
credit_range text
down_payment_available integer
kyc_verified boolean DEFAULT false
soft_pull_completed boolean DEFAULT false
```

### realtor_profiles
```sql
id uuid PRIMARY KEY
user_id uuid REFERENCES users(id)
license_number text NOT NULL
license_state text NOT NULL
license_expires_at date
brokerage_name text
referral_code text UNIQUE
commission_tier text DEFAULT 'standard'
total_referrals integer DEFAULT 0
total_commissions_paid numeric(12,2) DEFAULT 0
stripe_connect_account_id text
```

### markets
```sql
id uuid PRIMARY KEY
name text NOT NULL
state text NOT NULL
county text NOT NULL
zip_codes text[]
fha_limit_single integer
fha_limit_duplex integer
fha_limit_triplex integer
fha_limit_fourplex integer
is_active boolean DEFAULT false
waitlist_count integer DEFAULT 0
drop_day_of_week integer DEFAULT 2
drop_window_start_hour integer DEFAULT 8
drop_window_end_hour integer DEFAULT 12
next_drop_at timestamptz
created_at timestamptz DEFAULT now()
```

### waitlist_entries
```sql
id uuid PRIMARY KEY
user_id uuid REFERENCES users(id)
market_id uuid REFERENCES markets(id)
buyer_score text
pre_registration_completed boolean DEFAULT false
referral_code text
realtor_id uuid REFERENCES realtor_profiles(id)
notified_at timestamptz[]
created_at timestamptz DEFAULT now()
UNIQUE(user_id, market_id)
```

### projects
```sql
id uuid PRIMARY KEY
market_id uuid REFERENCES markets(id)
landowner_user_id uuid REFERENCES users(id)
address text
parcel_number text
latitude numeric(10,6)
longitude numeric(10,6)
lot_size_sqft integer
zoning text
product_type text CHECK (product_type IN ('duplex','triplex','fourplex','adu_stack'))
unit_count integer
total_sqft integer
build_cost_estimate integer
land_appraised_value integer
total_project_value integer
fha_limit integer
list_price integer
status text DEFAULT 'feasibility'
permits_filed_at timestamptz
permits_issued_at timestamptz
target_close_at timestamptz
actual_close_at timestamptz
manufacturer_id uuid REFERENCES manufacturer_profiles(id)
investor_capital_called integer DEFAULT 0
investor_capital_target integer
jv_agreement_docusign_id text
created_at timestamptz DEFAULT now()
```

### project_units
```sql
id uuid PRIMARY KEY
project_id uuid REFERENCES projects(id)
unit_number text
unit_type text
bedrooms integer
bathrooms numeric(3,1)
sqft integer
list_price integer
status text DEFAULT 'available'
reservation_id uuid REFERENCES reservations(id)
buyer_user_id uuid REFERENCES users(id)
hold_for_portfolio boolean DEFAULT false
created_at timestamptz DEFAULT now()
```

### reservations
```sql
id uuid PRIMARY KEY
certificate_number text UNIQUE
unit_id uuid REFERENCES project_units(id)
buyer_user_id uuid REFERENCES users(id)
realtor_id uuid REFERENCES realtor_profiles(id)
stripe_payment_intent_id text
amount_paid integer DEFAULT 99
status text DEFAULT 'active'
reserved_at timestamptz DEFAULT now()
refund_eligible_until timestamptz
converted_at timestamptz
resold_at timestamptz
resale_price integer
resale_buyer_user_id uuid REFERENCES users(id)
resale_fee_paid integer
certificate_url text
created_at timestamptz DEFAULT now()
```

### deposits
```sql
id uuid PRIMARY KEY
reservation_id uuid REFERENCES reservations(id)
project_unit_id uuid REFERENCES project_units(id)
buyer_user_id uuid REFERENCES users(id)
realtor_id uuid REFERENCES realtor_profiles(id)
stripe_payment_intent_id text
amount integer DEFAULT 10000
status text DEFAULT 'pending'
paid_at timestamptz
rescission_deadline timestamptz
design_selections jsonb
purchase_agreement_docusign_id text
purchase_agreement_signed_at timestamptz
lender_id uuid REFERENCES lender_profiles(id)
full_pre_approval_amount integer
full_pre_approval_expires_at date
realtor_bonus_paid_at timestamptz
created_at timestamptz DEFAULT now()
```

### manufacturer_profiles
```sql
id uuid PRIMARY KEY
user_id uuid REFERENCES users(id)
company_name text NOT NULL
hcd_certified boolean DEFAULT false
hud_certified boolean DEFAULT false
energy_star_certified boolean DEFAULT false
coverage_states text[]
coverage_radius_miles integer
unit_types_produced text[]
annual_capacity_units integer
current_capacity_units integer
avg_lead_time_weeks integer
erpnext_instance_url text
erpnext_api_key text
base_price_per_sqft numeric(8,2)
rating numeric(3,2) DEFAULT 0
total_projects_completed integer DEFAULT 0
stripe_connect_account_id text
created_at timestamptz DEFAULT now()
```

### rfqs
```sql
id uuid PRIMARY KEY
project_id uuid REFERENCES projects(id)
status text DEFAULT 'open'
sent_at timestamptz DEFAULT now()
response_deadline_at timestamptz
awarded_at timestamptz
awarded_manufacturer_id uuid REFERENCES manufacturer_profiles(id)
awarded_amount integer
purchase_order_number text
created_at timestamptz DEFAULT now()
```

### rfq_quotes
```sql
id uuid PRIMARY KEY
rfq_id uuid REFERENCES rfqs(id)
manufacturer_id uuid REFERENCES manufacturer_profiles(id)
quoted_amount integer
lead_time_weeks integer
delivery_date date
notes text
score numeric(5,2)
status text DEFAULT 'submitted'
submitted_at timestamptz DEFAULT now()
```

### builder_bond_accounts
```sql
id uuid PRIMARY KEY
investor_user_id uuid REFERENCES users(id)
tier text
balance integer DEFAULT 0
idle_balance integer DEFAULT 0
deployed_balance integer DEFAULT 0
total_earned integer DEFAULT 0
target_annual_return numeric(5,4) DEFAULT 0.09
auto_roll boolean DEFAULT true
stripe_connect_account_id text
accreditation_verified boolean DEFAULT false
accreditation_verified_at timestamptz
created_at timestamptz DEFAULT now()
```

### builder_bond_allocations
```sql
id uuid PRIMARY KEY
account_id uuid REFERENCES builder_bond_accounts(id)
project_id uuid REFERENCES projects(id)
amount_allocated integer
allocated_at timestamptz DEFAULT now()
expected_return_at timestamptz
actual_return_at timestamptz
principal_returned integer
return_paid integer
penalty_paid integer DEFAULT 0
status text DEFAULT 'allocated'
```

### contractor_profiles
```sql
id uuid PRIMARY KEY
user_id uuid REFERENCES users(id)
company_name text
license_number text
license_state text
license_types text[]
coverage_zip_codes text[]
erpnext_instance_url text
erpnext_api_key text
rating numeric(3,2) DEFAULT 0
total_work_orders_completed integer DEFAULT 0
stripe_connect_account_id text
created_at timestamptz DEFAULT now()
```

### work_orders
```sql
id uuid PRIMARY KEY
project_id uuid REFERENCES projects(id)
scope_type text
description text
scope_document_url text
fbx_suggested_price integer
awarded_price integer
status text DEFAULT 'posted'
contractor_id uuid REFERENCES contractor_profiles(id)
posted_at timestamptz DEFAULT now()
bid_deadline_at timestamptz
awarded_at timestamptz
start_date date
completion_date date
completed_at timestamptz
payment_released_at timestamptz
stripe_transfer_id text
```

### lender_profiles
```sql
id uuid PRIMARY KEY
user_id uuid REFERENCES users(id)
company_name text NOT NULL
nmls_number text
fha_approved boolean DEFAULT false
construction_to_perm boolean DEFAULT false
manufactured_home_approved boolean DEFAULT false
coverage_states text[]
contact_email text
api_webhook_url text
monthly_platform_fee integer
referral_fee_bps integer
created_at timestamptz DEFAULT now()
```

### project_milestones
```sql
id uuid PRIMARY KEY
project_id uuid REFERENCES projects(id)
milestone_type text
status text DEFAULT 'pending'
target_date date
actual_date date
notes text
completed_at timestamptz
created_at timestamptz DEFAULT now()
```

---

## 5. Module Architecture

### 5.1 Public Marketing Site
- Homepage: demand heatmap + drop countdown + waitlist CTA
- How It Works pages (buyer, realtor, landowner, investor, manufacturer)
- Market pages (SEO per active market)
- Project listing pages
- Waitlist signup

### 5.2 Buyer Portal
- Pre-registration (KYC lite + soft credit pull + pre-approval upload)
- Drop access page (early window for A/B)
- $99 reservation flow
- Reservation certificate + resale marketplace
- AI design selections concierge
- $10K deposit + DocuSign
- Project milestone tracker
- Document vault

### 5.3 Realtor Portal
- Registration + license verification
- Unique referral link + QR code
- Client dashboard
- Commission tracker + Stripe Connect payouts

### 5.4 Landowner Portal
- Address → instant parcel analysis
- Proforma generator
- JV agreement (DocuSign)
- Project dashboard

### 5.5 Manufacturer Portal
- Registration + certifications
- Product catalog
- RFQ inbox + quote submission
- OpenClaw API bridge
- Payment history

### 5.6 Investor Portal
- Accreditation verification
- Builder Bond account funding
- Active allocations dashboard
- Compounding returns calculator
- 1031 exchange portfolio tool
- Tax documents

### 5.7 Admin Dashboard
- Market management + drop scheduling
- Project pipeline + exception flags
- Buyer pipeline + conversion rates
- RFQ management
- Financial overview

### 5.8 Autonomous Backend Services
- **Drop Engine** (BullMQ): schedules, randomizes, sends email+SMS, manages early/general access
- **RFQ Engine** (BullMQ): triggered on permits_issued, scores manufacturers, distributes RFQs, auto-awards
- **Work Order Engine** (BullMQ): auto-generates 6 scopes, posts to marketplace, awards, triggers payments
- **Capital Call Engine** (BullMQ): FIFO allocation on permit_issued, notifies investors
- **Notification Service**: all alerts across all participants
- **OpenClaw Bridge**: bidirectional ERPNext sync for manufacturers + contractors

---

## 6. API Structure

All APIs: RESTful, versioned at `/api/v1/`, authenticated via session cookie or Bearer token.

### Auth
- POST /api/v1/auth/register
- POST /api/v1/auth/login
- POST /api/v1/auth/logout
- GET /api/v1/auth/me

### Buyers
- POST /api/v1/buyers/waitlist
- POST /api/v1/buyers/pre-register
- GET /api/v1/buyers/score
- GET /api/v1/buyers/drops
- POST /api/v1/buyers/reserve/:unitId
- POST /api/v1/buyers/deposit/:reservationId
- GET /api/v1/buyers/reservations
- GET /api/v1/buyers/projects

### Reservations Marketplace
- GET /api/v1/reservations/marketplace
- POST /api/v1/reservations/:id/list-for-sale
- POST /api/v1/reservations/:id/purchase
- POST /api/v1/reservations/:id/refund

### Realtors
- POST /api/v1/realtors/register
- GET /api/v1/realtors/clients
- GET /api/v1/realtors/commissions
- GET /api/v1/realtors/referral-link

### Landowners
- POST /api/v1/landowners/analyze
- POST /api/v1/landowners/jv-request
- GET /api/v1/landowners/projects
- GET /api/v1/landowners/returns

### Manufacturers
- POST /api/v1/manufacturers/register
- GET /api/v1/manufacturers/rfqs
- POST /api/v1/manufacturers/rfqs/:id/quote
- POST /api/v1/manufacturers/rfqs/:id/accept-po
- GET /api/v1/manufacturers/orders

### Investors
- POST /api/v1/investors/register
- POST /api/v1/investors/fund-account
- GET /api/v1/investors/allocations
- GET /api/v1/investors/returns
- POST /api/v1/investors/1031-inquiry

### Projects
- GET /api/v1/projects
- GET /api/v1/projects/:id
- GET /api/v1/projects/:id/milestones
- GET /api/v1/projects/:id/units
- POST /api/v1/projects (admin)
- PATCH /api/v1/projects/:id/milestone

### Markets
- GET /api/v1/markets
- GET /api/v1/markets/:id/demand
- GET /api/v1/markets/:id/next-drop

### Webhooks
- POST /api/v1/webhooks/stripe
- POST /api/v1/webhooks/docusign
- POST /api/v1/webhooks/twilio
- POST /api/v1/webhooks/erpnext

---

## 7. Real-Time Events (WebSocket)

**Namespace: /drops**
- drop:started — {marketId, units[]}
- drop:unit:viewer_count — live count per unit
- drop:unit:reserved — {unitId, remaining}
- drop:unit:sold_out
- drop:ended

**Namespace: /projects**
- project:milestone:updated
- project:milestone:delayed

---

## 8. File Storage (Cloudflare R2)

```
fbx-assets/
  reservations/certificates/{certificate_number}.pdf
  projects/{project_id}/photos/
  projects/{project_id}/documents/jv_agreement.pdf
  projects/{project_id}/documents/purchase_agreement_{unit_id}.pdf
  projects/{project_id}/documents/permits/
  projects/{project_id}/factory_progress/{date}_{photo}.jpg
  manufacturers/{manufacturer_id}/certifications/
  users/{user_id}/pre_approval_letter.pdf
```

---

## 9. Security Requirements

- Session middleware on all authenticated routes
- RBAC: requireRole(['buyer','admin']) middleware
- Buyer score gate: requireBuyerScore(['A','B']) for drop access
- Stripe webhook signature verification
- DocuSign HMAC verification
- PII encrypted at rest (Neon)
- Rate limiting: 100 req/15min per IP on auth routes
- CSRF protection on all mutations
- All secrets in Cloudflare secrets manager

---

## 10. Environment Variables

```
DATABASE_URL=
BETTER_AUTH_SECRET=
BETTER_AUTH_URL=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PUBLISHABLE_KEY=
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_FROM_NUMBER=
RESEND_API_KEY=
DOCUSIGN_INTEGRATION_KEY=
DOCUSIGN_ACCOUNT_ID=
DOCUSIGN_PRIVATE_KEY=
MAPBOX_TOKEN=
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=
UPSTASH_REDIS_URL=
UPSTASH_REDIS_TOKEN=
CALHFA_API_KEY=
ERPNEXT_BRIDGE_SECRET=
SENTRY_DSN=
POSTHOG_API_KEY=
```

---

## 11. Deployment Architecture

```
Cloudflare CDN
     |
     +-- CF Pages (React SPA)
     +-- CF Workers (Express API)
          |
          +-- Neon PostgreSQL
          +-- Upstash Redis --> BullMQ Workers (Fly.io)
          +-- CF R2 Storage
```

---

## 12. Testing Requirements

- Unit tests (Vitest): buyer scoring, RFQ scoring, proforma, penalty engine — 80% coverage
- Integration tests (Supertest): all API routes with test DB — 60% coverage
- E2E (Playwright): full buyer funnel, realtor flow, landowner JV, manufacturer RFQ
- Load test (k6): 500 concurrent users on reserve endpoint during drop simulation
- Accessibility (axe-playwright): WCAG 2.1 AA on all critical flows
- Payment tests: Stripe test mode for all payment flows

---

*Version: 1.0 | Last updated: 2026-02-26*