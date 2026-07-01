# Sushastho Clinic — Clinic Management System

A full-stack clinic management web app built for clinics in Bangladesh: patient
records, appointment scheduling, prescriptions, and billing in BDT.

## Tech stack

- **Next.js 16** (App Router, TypeScript, Server Actions)
- **Tailwind CSS v4** for styling
- **NextAuth v5** for authentication (credentials login, role-based)
- **SQLite via Node's built-in `node:sqlite` module** — no external database
  service to set up. Data is stored locally in `data/clinic.db`.

> Note on the database choice: this project originally targeted Prisma +
> PostgreSQL, but Prisma's engine binaries and native SQLite drivers both
> require downloading platform binaries from external CDNs, which wasn't
> possible in the sandboxed environment this was built in. Node 22+ ships a
> built-in SQLite module, so that's what this uses instead — no separate
> database server to install or configure. If you'd like to migrate to
> PostgreSQL for production later (e.g. for multi-server deployments), the
> query layer is isolated in `src/lib/models/`, so that's the only place
> that would need to change.

## Roles

- **Admin** — manage doctors and receptionist accounts, view all appointments,
  billing, and clinic-wide stats.
- **Receptionist** — register patients, book/reschedule/cancel appointments,
  record payments.
- **Doctor** — see their daily schedule, write prescriptions, mark visits
  complete (which auto-generates an invoice).

There's no public sign-up — the Admin creates Doctor and Receptionist
accounts from the dashboard.

## Getting started

```bash
npm install
npm run seed   # creates the first Admin login
npm run dev
```

Then open http://localhost:3000 and log in with:

```
Email:    admin@clinic.com
Password: admin123
```

**Change this password** (or create a new admin and deactivate this one)
before using this with real patient data — there's no "change password" UI
yet, so for now that means editing the account directly or re-running the
seed script against a fresh database.

From the Admin dashboard, add your doctors and receptionists. They can then
log in with the email/phone + password you set for them.

## Project structure

```
src/
  app/                    Routes (App Router)
    login/                Public login page
    dashboard/admin/      Admin-only pages
    dashboard/doctor/     Doctor-only pages
    dashboard/receptionist/  Receptionist-only pages
    api/                  A couple of small JSON endpoints (slot lookup, patient search)
  components/             UI components (mostly client components for forms/interactivity)
  lib/
    db.ts                 SQLite connection + schema migration
    models/                Query functions, one file per entity
    actions/               Server Actions (mutations: create patient, book appointment, etc.)
    auth.ts                NextAuth config
    guard.ts               Role-based access control used in each dashboard layout
scripts/
  seed.ts                 Creates the initial Admin account
data/
  clinic.db               SQLite database file (created automatically, gitignored)
```

## What's included

- Patient records with search by name/phone, visit history, and Bangladeshi
  phone number validation
- Appointment booking with live slot availability per doctor (prevents
  double-booking)
- Confirm / reschedule / cancel appointment flows
- Prescriptions with a dynamic medicine list, printable prescription view
- Automatic invoice generation on visit completion, with Cash/bKash/Nagad/
  Rocket payment method tracking
- Admin overview dashboard with a revenue chart and today's stats
- Role-based access control — each dashboard route checks the signed-in
  user's role server-side

## What's simplified / not included

This covers the core clinic workflow end-to-end, but a few things from the
original scope were left out to keep this a manageable v1 — happy to build
any of these out next:

- **Bangla language toggle** — the UI is English-only for now
- **Change password / forgot password** — not built yet
- **Real payment gateway integration** — bKash/Nagad/Rocket are recorded as a
  payment method label, not processed as real transactions
- **Multi-clinic / multi-branch support**

## Deploying this for real use

This currently uses a local SQLite file, which is great for running on one
machine but won't work as-is on serverless hosts like Vercel (no persistent
disk). For a real deployment you have two straightforward options:

1. **Simplest**: run this on a small VPS (e.g. DigitalOcean, a local Linux
   server at the clinic) with `npm run build && npm run start` — the SQLite
   file persists on disk normally.
2. **For serverless hosting**: swap the database layer for a hosted
   Postgres/MySQL provider (e.g. Supabase, Neon, Railway) — this would mean
   rewriting `src/lib/db.ts` and the files in `src/lib/models/` to use that
   database's client instead of `node:sqlite`. Ask me and I can do this
   conversion for you.

Either way, make sure to set a real `AUTH_SECRET` in production (see
`.env.example`) and change the default admin password.
