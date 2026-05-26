
 ![](assets/rung-mark.svg)
#  Rung 

[getrung.app](https://getrung.app)

**Validate before you build.**  
A hosted waitlist SaaS for indie makers - landing page, embeddable form, real signups, CSV export.

**[Live demo →](https://getrung.app)**  ·  [Screenshots](#screenshots)  ·  [Features](#features)  ·  [Tech stack](#tech-stack)  ·  [Architecture notes](#architecture-notes)

---



## What this is

Rung is a working SaaS for indie makers who want to test demand before they build. You can sign up, create a project, share the link, watch signups come in, and export the list when it's time to launch.

This repo is also a **portfolio piece**: a full-stack Next.js 16 app built with Auth.js v5, Prisma, and Postgres, deployed end to end on Vercel with branch-per-preview Neon databases. The source lives in a private repo, so this README serves as the case study.

> **Status:** Public beta. Free tier is live. Stripe billing, custom domains, and webhook notifications are next.

---

## Screenshots

### Interactive product preview

The marketing page includes a live customizer, so visitors can toggle dark mode, the subscriber counter, and the logo on a mock waitlist page before they even sign up.



### Three steps to launch day

Name your project. Pick a color. Toggle social proof. Share the link.



### Six features. Zero bloat.

Hosted pages, real-time analytics, email collection, custom domains, embeddable forms, and one-click CSV export.



### Social proof

A horizontal carousel of indie maker testimonials sits between the feature grid and pricing.



### Simple pricing

Free forever for one project. Pro at $9/mo when you need more.



---

## Features


| Feature                                                                            | Status    |
| ---------------------------------------------------------------------------------- | --------- |
| Email + password signup with email verification                                    | ✅         |
| Google + GitHub OAuth                                                              | ✅         |
| Password reset (token-based, expiring links via Resend)                            | ✅         |
| Project CRUD - custom slug, accent color, optional logo URL                        | ✅         |
| Hosted public waitlist page at `/<slug>`                                           | ✅         |
| One-line embeddable widget at `/api/embed/<slug>/embed.js`                         | ✅         |
| Live subscriber counter on public page                                             | ✅         |
| Subscriber list view + CSV export                                                  | ✅         |
| 7-day signup sparkline                                                             | ✅         |
| Account management - change email, change password, export data, delete account    | ✅         |
| Honeypot bot defense + IP-hashed rate limiting on subscribe                        | ✅         |
| Mobile-responsive dashboard with sheet drawer nav                                  | ✅         |
| Security headers (CSP, HSTS, X-Frame-Options, Referrer-Policy, Permissions-Policy) | ✅         |
| Stripe billing                                                                     | ⏳ planned |
| Custom domains for hosted pages                                                    | ⏳ planned |
| Webhook notifications on subscriber milestones                                     | ⏳ planned |


---

## Tech stack


| Layer      | Choice                                                   |
| ---------- | -------------------------------------------------------- |
| Framework  | Next.js 16 (App Router, React 19, React Compiler)        |
| Language   | TypeScript (strict mode)                                 |
| Styling    | Tailwind CSS 4 + shadcn/ui (scoped Radix primitives)     |
| Animations | `motion` (Framer Motion v12)                             |
| Database   | PostgreSQL via Neon                                      |
| ORM        | Prisma 7 with `@prisma/adapter-pg` driver adapter        |
| Auth       | Auth.js v5 (JWT sessions, credentials + Google + GitHub) |
| Email      | Resend (verification + password reset only)              |
| Validation | Zod (every API boundary + every form)                    |
| Toasts     | Sonner                                                   |
| Icons      | Lucide                                                   |
| Tests      | Vitest - 30 tests passing                                |
| CI         | GitHub Actions (lint + test + build)                     |
| Hosting    | Vercel (frontend + API routes)                           |


**Vendor count: 5** - Vercel, Neon, Resend, Google OAuth, GitHub OAuth. No Stripe, no Sentry, no Upstash, no Turnstile. Kept deliberately small until the product actually needs more.

---

## Architecture notes

A few decisions worth a closer look - the *why* behind each choice:

- **JWT sessions over DB sessions.** No `Session` table - every request decodes a signed cookie. Cheap reads, no DB hop on auth checks. Tradeoff: revocation requires a `tokenVersion` field (not implemented; acceptable for the current trust model).
- **In-memory rate limiter.** A `Map` in `src/lib/rate-limit.ts`. Fine for single-instance dev and the showcase deploy; on multi-instance serverless it limits per function instance, not globally. Documented upgrade path: drop in `@upstash/ratelimit` with the same call signature.
- **Honeypot instead of CAPTCHA.** A hidden `hp` field on the subscribe form. Filled = silent 200, no DB write. Cheaper, no external script, no per-request network call, no user friction. Won't stop a determined attacker but neutralizes mass form-spam bots.
- **Prisma driver adapter.** `PrismaPg` from `@prisma/adapter-pg` instead of the bundled binary. Connection string is normalized in `src/lib/db.ts` to opt into libpq SSL semantics (silences the pg-connection-string deprecation warning on Neon).
- **Per-PR Neon database branches.** `vercel-build` runs `prisma migrate deploy && next build`; each preview gets its own database branch via the Neon Vercel integration.
- **Native form + Zod over react-hook-form.** Project create/edit form is plain `useState` + `safeParse` on submit. Less ceremony, smaller bundle, equivalent validation surface.
- **Dark only.** No theme toggle. One CSS variable set, less to test, less to maintain.

---

## Testing

Vitest, 30 tests passing. Coverage is concentrated on the bits that fail silently if regressed:

- Rate-limit utilities - window expiry, per-key isolation, header parsing
- Zod validators - slug rules, reserved slug list, hex color, safe URL redirect

Lint + test + build run on every push via GitHub Actions.

---

## Project structure

```
src/
├── app/
│   ├── (marketing)/              Landing page, privacy, terms
│   ├── (auth)/                   Login, signup, forgot/reset password
│   ├── (dashboard)/              Protected app routes (project list, detail, account)
│   ├── [slug]/                   Public waitlist page
│   ├── embed/[slug]/             Iframe-friendly embed page
│   ├── api/                      All API routes (auth, projects, public subscribe, etc.)
│   ├── layout.tsx                Root layout
│   └── globals.css               Tailwind config + design tokens + keyframes
│
├── components/
│   ├── ui/                       shadcn primitives
│   ├── layout/                   Logo, navbar, footer
│   ├── marketing/                Hero, features, pricing, FAQ, CTA, background orbs
│   ├── forms/                    Login, signup, forgot/reset password
│   ├── public/                   Subscribe form (used by hosted page + embed)
│   ├── dashboard/                Project card, stats card, subscriber table
│   └── account/                  Change email, change password, delete, export
│
├── hooks/                        useSubscription
├── lib/                          db, auth, validators, email, tokens, rate-limit, site
├── proxy.ts                      Next.js 16 middleware (route protection)
└── lib/__tests__/                Vitest specs
```

---

## Roadmap

In roughly the order they'd ship:

1. **Stripe subscriptions** + Pro tier feature gating
2. **Distributed rate limiter** (Upstash) for real public launch
3. **Custom domains** for hosted waitlist pages
4. **Webhook notifications** on subscriber milestones

---

## License

MIT - see [LICENSE](./LICENSE).

---

**Built for indie makers who validate before they build.**  