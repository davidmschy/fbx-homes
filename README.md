# FBX — The Weekly Lot Drop Marketplace for Missing Middle Housing

> Factory-built duplexes, triplexes, and fourplexes with $0 down FHA financing. Built like Supreme drops. Delivered in 4 months.

[![Deploy](https://img.shields.io/badge/deployed-cloudflare-orange)](https://fbx.homes)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

---

## What Is FBX?

FBX (Factory Built Exchange) is a **weekly lot drop marketplace** for missing middle housing. Buyers pre-register, receive a random Tuesday notification, and compete to reserve factory-built homes for $99 — then convert to a $10K deposit with design selections and FHA financing. No down payment. Move-in ready in 4 months.

The platform coordinates six participant types automatically:
- **Buyers** — FHA-eligible homebuyers, $0 down with CalHFA DPA
- **Landowners** — JV with FBX via "Let FBX Develop" button
- **Manufacturers** — Modular factories bid on auto-generated RFQs
- **Contractors** — Accept site work orders via marketplace or OpenClaw/ERPNext
- **Investors** — Fund construction via Builder Bonds (9% annualized, 90-day deployment)
- **Realtors** — Register clients, earn $2K bonus + 2% co-op at close

## The Drop Model

1. **Pre-register** — KYC lite + soft credit pull → Buyer Score A/B/C
2. **Tuesday drop** — Random time 8am–12pm PT, email + SMS blast
3. **2-hour early access** — Score A/B buyers only
4. **$99 reservation** — Transferable digital certificate, resale marketplace
5. **Design selections** — AI concierge, live price updates
6. **$10K deposit** — Stripe escrow + DocuSign purchase agreement
7. **Build** — Factory-built, delivered in 10–12 weeks after permits
8. **Close** — FHA loan closes at Certificate of Occupancy

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TypeScript + Vite + Tailwind + shadcn/ui |
| Backend | Node.js 20 + Express + TypeScript |
| Database | PostgreSQL (Neon serverless) + Drizzle ORM |
| Cache/Queue | Redis (Upstash) + BullMQ |
| Auth | Better Auth (session-based) |
| Payments | Stripe (payments, escrow, Connect payouts) |
| E-Sign | DocuSign |
| Email | Resend (transactional) + Loops (sequences) |
| SMS | Twilio |
| Maps | Mapbox GL JS |
| Storage | Cloudflare R2 |
| Hosting | Cloudflare Workers + Pages |
| Real-time | Socket.io (drop viewer counts, milestone updates) |
| CI/CD | GitHub Actions |
| Monitoring | Sentry + Posthog |

## Repository Structure

```
fbx-homes/
├── client/               # React frontend (Vite)
│   ├── src/
│   │   ├── pages/        # Route-level pages
│   │   ├── components/   # Shared UI components
│   │   ├── hooks/        # Custom React hooks
│   │   ├── stores/       # Zustand state stores
│   │   └── lib/          # Utilities, API client
├── server/               # Express backend
│   ├── routes/           # API route handlers
│   ├── services/         # Business logic
│   ├── workers/          # BullMQ job workers
│   ├── middleware/        # Auth, roles, validation
│   └── lib/              # DB client, external APIs
├── shared/               # Shared types (client + server)
│   └── types/
├── db/                   # Drizzle schema + migrations
├── docs/                 # Architecture documentation
│   ├── PLATFORM_DESIGN.md
│   └── IMPLEMENTATION_PLAN.md
└── .github/
    └── workflows/        # CI/CD pipelines
```

## Key Documents

- [Platform Design](docs/PLATFORM_DESIGN.md) — Complete system architecture, database schema, API spec, module design
- [Implementation Plan](docs/IMPLEMENTATION_PLAN.md) — 77 PRs across 6 phases, agent assignments, timeline

## Getting Started

```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Fill in DATABASE_URL, STRIPE_SECRET_KEY, etc. (see docs/PLATFORM_DESIGN.md section 10)

# Run database migrations
npm run db:migrate

# Start development server
npm run dev
# Frontend: http://localhost:5173
# Backend: http://localhost:3000
```

## Environment Variables

See [docs/PLATFORM_DESIGN.md#10-environment-variables](docs/PLATFORM_DESIGN.md) for the complete list.

## Development Workflow

- Branch naming: `feature/PR-XXX-short-description`
- All PRs target `main`
- Required: lint + typecheck + tests pass
- Squash merge only
- Deploy preview on every PR via Cloudflare Pages

## The Business Model

FBX makes money without owning land or employing construction workers:

| Revenue Stream | Rate | At 100 Projects/Year |
|---------------|------|---------------------|
| Development margin | Build cost vs. sale price | $80M–$140M |
| Investor capital origination | 1% of capital called | $800K–$1.5M |
| Contractor marketplace fee | 8% of work order value | $1.2M–$2M |
| Lender referral | 1% of funded loan | $1M–$1.8M |
| Realtor co-op (net) | Platform facilitates | Volume driver |
| Property management | 8% of rent | $400K–$800K |

## License

MIT

---

Built with ❤️ for the missing middle housing crisis.