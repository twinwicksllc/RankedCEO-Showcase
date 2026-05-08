<div align="center">

# 🏆 RankedCEO CRM

**A production-ready, multi-tenant CRM platform with industry-specific vertical products, Website-as-a-Service infrastructure, and AI-ready architecture.**

[![Live App](https://img.shields.io/badge/Live%20App-crm.rankedceo.com-blue?style=for-the-badge&logo=vercel)](https://crm.rankedceo.com)
[![TypeScript](https://img.shields.io/badge/TypeScript-78.7%25-3178C6?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![Next.js](https://img.shields.io/badge/Next.js-14-black?style=for-the-badge&logo=next.js)](https://nextjs.org/)
[![Supabase](https://img.shields.io/badge/Supabase-Backend-3ECF8E?style=for-the-badge&logo=supabase)](https://supabase.com/)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen?style=for-the-badge)](https://crm.rankedceo.com)

</div>

---

## 📖 What Is It?

RankedCEO CRM is a fully custom, production-deployed Customer Relationship Management platform built on **Next.js 14**, **Supabase PostgreSQL**, and **TypeScript**. It goes far beyond a standard CRM — the same monorepo also powers a **multi-subdomain industry vertical system** (Dental, HVAC, Plumbing, Electrical) where each vertical gets its own branded subdomain, lead intake form, and operator dashboard, plus a **Website-as-a-Service (WaaS)** layer that serves white-labeled websites to external clients on custom domains — all routed through a single unified middleware.

**67 total routes · 30+ database tables · 100% RLS coverage · 14+ completed build phases · Live at crm.rankedceo.com**

---

## ✨ Core CRM Features

### 👥 Contacts & Companies
- Complete contact database with 15+ fields (name, email, phone, company, title, source, etc.)
- Company profiles with associated contacts, deal history, and revenue statistics
- Advanced search, filtering, and sorting across all records
- Full CRUD with inline editing and detail pages

### 🎯 Deal Pipeline
- Visual pipeline management with customizable stages
- Full deal lifecycle: value, probability, close date, associated contacts/companies
- Automatic commission calculation when a deal is marked **Won**
- Deal-level activity timeline and history

### 📋 Activity Tracking & Timeline
- Log **Calls, Meetings, Emails, Notes,** and **Tasks** against any contact, company, or deal
- Chronological activity timeline on every record
- Filter by activity type, date range, or assigned user
- User attribution and timestamps on all activities

### 📧 Email Campaigns & Templates
- Create and send email campaigns via **SendGrid** integration
- Pre-built template library with custom template creation
- **Smart BCC Email Capture** — forward emails to a unique BCC address and they automatically appear in the CRM contact's timeline
- Complete email threading: view full conversation history inside the CRM
- SendGrid webhook for delivery, open, and click tracking

### 📐 Form Builder
- Drag-and-drop form builder with **17 field types** (text, email, phone, select, checkbox, date, file upload, etc.)
- Public shareable form URLs (`/f/[id]`) for lead capture and customer feedback
- Form submission tracking with export to CSV/JSON
- Conditional logic framework (ready for implementation)

### 💰 Commission Tracking
- Commission rate management (define rates per product/deal type)
- Automatic commission calculation triggered on deal close
- Commission dashboard with reporting and payout management history

### 📊 Analytics & Reporting
- **Revenue analytics**: total revenue, revenue by month, revenue by sales rep, average deal size, growth trend
- **Pipeline analytics**: value by stage, win rate, average deal cycle time, deals by lead source, pipeline velocity
- **Activity analytics**: activity by type, completion rate, user leaderboard, upcoming activities
- Interactive charts via **Recharts** with date range filtering
- CSV data export on all report views

### 👤 Multi-Step Onboarding Wizard
- Guided 5-step setup for new users: Welcome → Company Info → Team Setup → Preferences → Completion
- Onboarding state persisted to the database so new users always resume where they left off

### ⚙️ Settings Module
- Profile settings (name, phone, title, avatar)
- Account settings (company info, plan)
- Team management: invite members, manage roles
- Notification preferences (5 configurable toggles)
- Security settings (password management, 2FA UI)

---

## 🏭 Industry Vertical Products

The same monorepo powers four industry-specific subdomain products, each with its own branded entry point, public lead intake form, and protected operator dashboard:

| Product | Subdomain | Industry | Brand Color |
|---|---|---|---|
| **Smile MakeOver** | `smile.rankedceo.com` | 🦷 Dental | Purple |
| **HVAC Pro** | `hvac.rankedceo.com` | 🔥 HVAC | Blue `#2563EB` |
| **Plumb Pro** | `plumbing.rankedceo.com` | 🔧 Plumbing | Teal `#0D9488` |
| **Spark Pro** | `electrical.rankedceo.com` | 💡 Electrical | Amber `#D97706` |

### How the Vertical System Works

**Industry Isolation via Middleware** — `middleware.ts` reads the user's `industry` metadata (embedded in their Supabase auth token at signup) and enforces cross-subdomain isolation: a user registered on `hvac.rankedceo.com` is automatically redirected if they try to access `smile.rankedceo.com`.

**Pool Account Pattern** — When a customer submits a lead form without an operator's referral link, the lead is captured into an industry-specific **pool account** (pre-seeded with fixed UUIDs). No lead data is ever lost. Pool account admins can later claim, assign, or route these leads.

**Account-Level RLS** — All industry tables use `USING (account_id = get_current_user_account_id())` rather than user-level policies, correctly handling `NULL auth_user_id` rows from public submissions.

**HIPAA Compliance (Dental)** — The Smile MakeOver module includes HIPAA-compliant data handling: all PII processed server-side only, no PII in error messages or logs, SECURITY DEFINER functions with hardened `search_path`.

**5-Step Lead Forms** — Each vertical has a purpose-built, multi-step intake form with industry-specific fields stored in a flexible `JSONB service_details` column:

| Vertical | Step Highlights |
|---|---|
| **HVAC** | System type/age/brand, symptom checkboxes (not cooling, strange noise, leaking), property type, warranty status |
| **Plumbing** | Service type (leak/drain/water heater), symptom checkboxes (burst pipe, sewage smell), shutoff valve access, water source |
| **Electrical** | Panel type/amperage, symptom checkboxes (flickering, burning smell, sparks), permit requirements, property age |
| **Dental** | Smile goals, medical conditions, HIPAA consent, preferred treatment timeline |

---

## 🌐 Website-as-a-Service (WaaS) Layer

Built as an isolated module inside the same monorepo, the WaaS system enables RankedCEO to host white-labeled websites for external clients on their own custom domains.

### How It Works

```
Request: client-a.com/services
         ↓
1. Static asset? → Pass through
         ↓
2. Industry subdomain? (smile/hvac/plumbing/electrical) → /app/{industry}/*
         ↓
3. WaaS tenant? (DB lookup via resolve_tenant_by_hostname RPC)
   ├─ Found → Rewrite to /_sites/{slug}/services
   └─ Not found → Continue
         ↓
4. CRM domain? → Auth check → Dashboard or login redirect
```

Middleware injects brand config into request headers (`x-waas-brand-config`, `x-waas-package-tier`, `x-waas-industry`) so tenant pages render with the correct colors, logo, and contact info — **zero extra DB queries** in page components.

### WaaS Package Tiers

| Feature | Hosting | Standard | Premium |
|---|---|---|---|
| Website hosting | ✅ | ✅ | ✅ |
| Custom domain | ❌ | ✅ | ✅ |
| SEO audit tool | ❌ | ✅ | ✅ |
| Competitor analysis | ❌ | ✅ | ✅ |
| AI insights | ❌ | ❌ | ✅ |
| White-label branding | ❌ | ❌ | ✅ |
| Audits per month | 0 | 10 | Unlimited |

### WaaS API Routes

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET/POST | `/api/waas/tenants` | Required | List / create tenants |
| GET/PATCH/DELETE | `/api/waas/tenants/[id]` | Required | Tenant management |
| POST | `/api/waas/audits` | None | Create prospect SEO audit (public) |
| GET | `/api/waas/audits/[id]/status` | None | Poll audit status |

---

## 🏗️ Architecture

```
┌───────────────────────────────────────────────────────────┐
│                    middleware.ts                           │
│  Static → Industry Subdomain → WaaS Tenant → CRM Domain   │
└────────────────────────┬──────────────────────────────────┘
                         │
       ┌─────────────────┼──────────────────┬───────────────┐
       ▼                 ▼                  ▼               ▼
┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────┐
│  CRM App    │  │  Industry    │  │  WaaS        │  │  Public   │
│ (dashboard) │  │  Subdomains  │  │  /_sites/    │  │  Forms    │
│  /app/...   │  │  /smile/     │  │  [slug]/     │  │  /f/[id]  │
│             │  │  /hvac/      │  │              │  │           │
└──────┬──────┘  │  /plumbing/  │  └──────────────┘  └───────────┘
       │         │  /electrical/│
       │         └──────────────┘
       │
       ├─────────────────────────────────────────────┐
       ▼                                             ▼
┌─────────────────────┐                   ┌──────────────────────┐
│   Supabase Auth     │                   │  Supabase PostgreSQL  │
│   (Email + JWT)     │                   │  30+ Tables, 60+ RLS  │
│   reCAPTCHA v3      │                   │  15+ DB Functions     │
└─────────────────────┘                   └──────────────────────┘
                                                     │
                               ┌─────────────────────┼──────────────┐
                               ▼                     ▼              ▼
                        ┌──────────┐        ┌──────────────┐  ┌──────────┐
                        │ SendGrid │        │ Google       │  │  Stripe  │
                        │ (Email)  │        │ reCAPTCHA v3 │  │ (Billing)│
                        └──────────┘        └──────────────┘  └──────────┘
```

### Service Layer Architecture

All business logic lives in typed service classes, keeping API routes thin and testable:

| Service | Responsibility |
|---|---|
| `contact-service.ts` | Contact CRUD, search, filtering |
| `company-service.ts` | Company management, statistics |
| `deal-service.ts` | Deal lifecycle, pipeline operations |
| `activity-service.ts` | Activity logging and querying |
| `campaign-service.ts` | Email campaign management |
| `commission-service.ts` | Commission calculation and reporting |
| `email-service.ts` | SendGrid sending, BCC capture, threading |
| `form-service.ts` | Form CRUD, field management, submissions |
| `pipeline-service.ts` | Pipeline stage management |
| `onboarding-service.ts` | Multi-step onboarding state |
| `settings-service.ts` | Profile, account, team, notification settings |

---

## 🔐 Security Architecture

### Row Level Security (RLS)
- **100% table coverage** — every table has RLS enabled
- **Multi-tenant isolation** — all queries are scoped to `account_id`
- **60+ RLS policies** across all tables
- `SECURITY DEFINER` functions with `SET search_path = public` hardening for privileged operations
- Pool account patterns handle `NULL auth_user_id` rows safely without breaking policies

### Authentication & Bot Protection
- **Supabase Auth** with email/password and JWT sessions
- **Google reCAPTCHA v3 Enterprise** on all authentication flows (signup, login)
- Server-side reCAPTCHA score validation — scores below threshold reject the request before any DB operation
- Protected routes via Next.js middleware — unauthenticated requests redirect to `/login`

### Data Protection
- All sensitive keys (`SENDGRID_API_KEY`, `RECAPTCHA_SECRET_KEY`, `WAAS_SUPABASE_SERVICE_ROLE_KEY`) are **server-side only** — never exposed to the browser
- CORS restricted to authorized origins
- SQL injection protection via parameterized Supabase queries throughout
- Zod runtime validation on all API route inputs

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Framework** | Next.js 14 (App Router, React Server Components) |
| **Language** | TypeScript 5.3 (100% typed) |
| **Styling** | Tailwind CSS 3.4 |
| **UI Components** | Radix UI (full primitive set) + custom components |
| **Animation** | Framer Motion |
| **Charts** | Recharts |
| **Tables** | TanStack Table v8 |
| **Forms** | React Hook Form + Zod |
| **Drag & Drop** | dnd-kit (form builder, sequence builder) |
| **Icons** | Lucide React |
| **Auth** | Supabase Auth (JWT sessions) |
| **Database** | Supabase PostgreSQL + PL/pgSQL (17.4% of codebase) |
| **Email** | SendGrid (@sendgrid/mail) |
| **Bot Protection** | Google reCAPTCHA v3 Enterprise |
| **Payments** | Stripe |
| **AI (hooks ready)** | Google Gemini (lead scoring), Perplexity (research) |
| **Phone Validation** | libphonenumber-js |
| **CSV Parsing** | PapaParse |
| **Analytics** | @vercel/analytics |
| **Deployment** | Vercel (Edge Functions + Edge Middleware) |
| **CI/CD** | GitHub → Vercel auto-deploy on push to `main` |

---

## 🗄️ Database Schema

**30+ PostgreSQL tables · 60+ RLS policies · 15+ database functions · 50+ indexes**

### Core CRM Tables

| Table | Purpose |
|---|---|
| `accounts` | Multi-tenant account definitions and settings |
| `users` | User profiles and authentication metadata |
| `contacts` | Contact records (15+ fields) |
| `companies` | Company profiles and statistics |
| `deals` | Deal pipeline records |
| `pipelines` | Pipeline definitions |
| `pipeline_stages` | Individual stage configurations |
| `activities` | Activity log (calls, meetings, emails, notes, tasks) |

### Feature Tables

| Table | Purpose |
|---|---|
| `campaigns` | Email campaign management |
| `email_templates` | Pre-built and custom email templates |
| `email_messages` | Individual email message records |
| `email_threads` | Email conversation threading |
| `forms` | Custom form definitions |
| `form_fields` | Form field configuration per form |
| `form_submissions` | Captured form submission data |
| `commissions` | Commission records per deal |
| `commission_rates` | Commission rate definitions |
| `team_members` | Organization team members |
| `team_invitations` | Pending team invitations |

### Industry Vertical Tables

| Table | Purpose |
|---|---|
| `smile_assessments` | Dental patient assessment submissions (HIPAA) |
| `industry_leads` | Unified lead table for HVAC, Plumbing, Electrical verticals |

### WaaS Tables

| Table | Purpose |
|---|---|
| `tenants` | WaaS tenant definitions (domain, brand config, package tier) |
| `audits` | SEO audit requests and results (JSONB report data) |

---

## 📡 API Routes

**17 API routes** covering authentication, all CRM resources, analytics, webhooks, and the public form submission endpoint:

```
POST   /api/auth/logout
GET    /api/analytics/revenue
GET    /api/analytics/revenue/by-month
GET    /api/analytics/revenue/by-user
GET    /api/analytics/pipeline
GET    /api/analytics/activity
GET    /api/activities           POST /api/activities
GET    /api/activities/[id]      PUT  /api/activities/[id]      DELETE /api/activities/[id]
GET    /api/campaigns            POST /api/campaigns
GET    /api/companies            POST /api/companies
GET    /api/contacts             POST /api/contacts
GET    /api/deals                POST /api/deals
GET    /api/forms                POST /api/forms
POST   /api/forms/submit         (public — no auth required)
GET    /api/pipelines            POST /api/pipelines
POST   /api/webhook/sendgrid
GET    /api/waas/tenants         POST /api/waas/tenants
GET    /api/waas/tenants/[id]    PATCH /api/waas/tenants/[id]   DELETE /api/waas/tenants/[id]
POST   /api/waas/audits
GET    /api/waas/audits/[id]/status
```

All CRM API routes include authentication checks, Zod input validation, structured error handling, and proper HTTP status codes.

---

## 🗂️ App Pages & Routes (50 Pages)

| Section | Routes | Description |
|---|---|---|
| **Auth** | `/login`, `/signup` | Email/password auth with reCAPTCHA v3 |
| **Dashboard** | `/` | Overview with KPI cards and recent activity |
| **Contacts** | `/contacts`, `/contacts/new`, `/contacts/[id]`, `/contacts/[id]/edit` | Full contact management |
| **Companies** | `/companies`, `/companies/new`, `/companies/[id]`, `/companies/[id]/edit` | Company profiles |
| **Deals** | `/deals`, `/deals/new`, `/deals/[id]`, `/deals/[id]/edit` | Pipeline management |
| **Activities** | `/activities`, `/activities/[id]` | Activity log and timeline |
| **Campaigns** | `/campaigns`, `/campaigns/new`, `/campaigns/[id]/build` | Email campaign management |
| **Emails** | `/emails`, `/email-templates` | Inbox and template library |
| **Forms** | `/forms`, `/forms/new`, `/forms/[id]/edit`, `/f/[id]` | Builder + public forms |
| **Pipelines** | `/pipelines`, `/pipelines/new` | Pipeline configuration |
| **Reports** | `/reports` | Analytics dashboards |
| **Commissions** | `/commissions`, `/commissions/rates`, `/commissions/reports` | Commission management |
| **Settings** | `/settings`, `/settings/profile`, `/settings/team`, `/settings/billing` | Account management |
| **Onboarding** | `/onboarding` | 5-step new user wizard |
| **Smile** | `smile.rankedceo.com`, `smile.rankedceo.com/assessment` | Dental vertical |
| **HVAC** | `hvac.rankedceo.com`, `hvac.rankedceo.com/lead` | HVAC vertical |
| **Plumbing** | `plumbing.rankedceo.com`, `plumbing.rankedceo.com/lead` | Plumbing vertical |
| **Electrical** | `electrical.rankedceo.com`, `electrical.rankedceo.com/lead` | Electrical vertical |
| **WaaS Admin** | `/waas`, `/waas/tenants`, `/waas/audits` | WaaS platform management |

---

## 📈 Build Metrics

| Metric | Value |
|---|---|
| Total routes | **67** (50 pages + 17 API routes) |
| UI components | **40+** |
| Service classes | **13+** |
| Database tables | **30+** |
| RLS policies | **60+** |
| Database indexes | **50+** |
| PL/pgSQL functions | **15+** |
| Lines of code | **~21,000+** |
| TypeScript errors | **0** |
| Build time | ~60 seconds |
| Bundle size (shared) | 87.4 kB |

---

## 🚀 Getting Started

### Prerequisites

- Node.js 20.x+
- A [Supabase](https://supabase.com) project
- A [SendGrid](https://sendgrid.com) account (email features)
- [Google reCAPTCHA v3 Enterprise](https://www.google.com/recaptcha/admin) keys

### Installation

```bash
git clone https://github.com/twinwicksllc/rankedceo-crm-production.git
cd rankedceo-crm-production
npm install
```

### Environment Variables

Create `.env.local`:

```env
# Supabase (Required)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# reCAPTCHA v3 (Required)
NEXT_PUBLIC_RECAPTCHA_SITE_KEY=your_recaptcha_site_key
RECAPTCHA_SECRET_KEY=your_recaptcha_secret_key

# SendGrid (Required for email)
SENDGRID_API_KEY=your_sendgrid_api_key

# WaaS (Required for WaaS module)
NEXT_PUBLIC_WAAS_SUPABASE_URL=your_waas_supabase_url
NEXT_PUBLIC_WAAS_SUPABASE_ANON_KEY=your_waas_anon_key
WAAS_SUPABASE_SERVICE_ROLE_KEY=your_waas_service_role_key  # Server-side only

# AI Services (Optional)
NEXT_PUBLIC_GEMINI_API_KEY=your_gemini_api_key
NEXT_PUBLIC_PERPLEXITY_API_KEY=your_perplexity_api_key

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run the Dev Server

```bash
npm run dev
# App available at http://localhost:3000
```

### Database Setup

Run all migrations in order from `supabase/migrations/` in your Supabase SQL Editor. See `MIGRATION_FINAL_INSTRUCTIONS.md` for the complete ordered migration list.

---

## 🚢 Deployment

The app deploys to **Vercel** with automatic builds on every push to `main`.

1. Connect the GitHub repository to Vercel
2. Add all environment variables in Vercel project settings
3. Add wildcard domain `*.rankedceo.com` in Vercel for industry subdomain routing
4. Push to `main` — Vercel builds and deploys automatically

See `VERCEL_DEPLOYMENT_INSTRUCTIONS.md` for the complete checklist.

---

## 🔜 Roadmap

### v1.1
- Full TypeScript strict typing throughout all service classes
- Unit and integration test coverage
- Email verification on signup
- Full 2FA implementation

### v1.2
- AI Lead Scoring with Google Gemini
- AI Research Assistant with Perplexity
- Pool account admin dashboard and lead marketplace
- Email/SMS notifications on new lead submissions (industry verticals)

### v1.3+
- WaaS audit worker (Serper.dev / DataForSEO integration)
- WaaS page builder with industry-specific templates
- Calendar and appointment booking integration
- Workflow automation engine
- VoIP integration
- Mobile app (React Native)

---

## 📄 License

Proprietary software — © TwinWicks LLC. All rights reserved.

Built with ❤️ by [TwinWicks LLC](https://twinwicksllc.com)
