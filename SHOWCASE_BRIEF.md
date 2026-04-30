# Showcase README Brief

**Task for the new session:** Rewrite `README.md` to be a visually impressive, portfolio-quality README for a code-less public GitHub showcase repo. The README is the *entire product* — no source code is visible, so it has to do all the work.

---

## What this repo is

`github.com/ertanm/waitly-showcase` — a **public portfolio case study repo**. It contains:
- `README.md` (the file you're rewriting)
- `LICENSE` (MIT)
- `screenshots/` — 3 PNGs:
  - `screenshots/dashboard.png` — logged-in dashboard (projects list, stats, sparkline)
  - `screenshots/hosted-page.png` — a public hosted waitlist page `/<slug>`
  - `screenshots/embed.png` — the embed code generator page

No source code. No `src/`, no `package.json`, no configs. Visitors see the README rendered on GitHub and click through to the live demo.

**Live demo:** https://waitly-blush.vercel.app

---

## What Waitly is

A SaaS for indie makers who want to validate demand before building. You sign up, create a "project," share a URL, and collect emails on a hosted landing page — or drop a one-line embed widget into an existing site. All signups go into a subscriber list you can export as CSV when you're ready to launch.

**Production verdict:** Full-stack working app. Friends-and-family beta ready. Not yet monetised (Stripe planned).

---

## Exact feature set (everything listed must be accurate — don't invent)

| Feature | Works? |
| --- | --- |
| Email + password signup with email verification | ✅ |
| Google + GitHub OAuth | ✅ |
| Password reset (token-based, expiring links via Resend) | ✅ |
| Project CRUD — custom slug, accent color, optional logo URL | ✅ |
| Hosted public waitlist page at `/<slug>` | ✅ |
| One-line embeddable widget at `/api/embed/<slug>/embed.js` | ✅ |
| Live subscriber counter on public page | ✅ |
| Subscriber list view | ✅ |
| CSV export | ✅ |
| 7-day signup sparkline | ✅ |
| Account management — change email, change password, export data, delete account | ✅ |
| Honeypot bot defense + IP-hashed rate limiting on subscribe | ✅ |
| Mobile-responsive dashboard with sheet drawer nav | ✅ |
| Security headers (CSP, HSTS, X-Frame-Options, Referrer-Policy, Permissions-Policy) | ✅ |
| Stripe billing | ⏳ planned |
| Custom domains | ⏳ planned |
| Webhook notifications | ⏳ planned |

---

## Stack (verified, post-simplification — no Stripe, no Sentry, no Upstash, no Turnstile)

| Layer | Choice |
| --- | --- |
| Framework | Next.js 16 (App Router, React 19, React Compiler) |
| Language | TypeScript strict |
| Styling | Tailwind CSS 4 + shadcn/ui |
| Animations | `motion` (Framer Motion v12) |
| Database | PostgreSQL via Neon |
| ORM | Prisma 7 with `@prisma/adapter-pg` |
| Auth | Auth.js v5 (JWT sessions, credentials + Google + GitHub) |
| Email | Resend (verification + reset only) |
| Validation | Zod |
| Tests | Vitest — 30 tests passing |
| CI | GitHub Actions (lint + test + build) |
| Hosting | Vercel (frontend + API routes) |

**Vendor count: 5** — Vercel, Neon, Resend, Google OAuth, GitHub OAuth.

---

## Architecture decisions worth highlighting

These are real design choices — good interview-bait for the author:

- **JWT sessions, no Session table** — reads are cheap, no DB hop on auth checks. Tradeoff: revocation needs a `tokenVersion` strategy.
- **In-memory rate limiter** (`Map`) — fine for single-instance dev/showcase. Known upgrade path: swap in `@upstash/ratelimit` with the same call signature.
- **Honeypot instead of CAPTCHA** — hidden `hp` field; if filled, silent 200 with no DB write. Zero user friction, no external script.
- **Prisma driver adapter** — `PrismaPg` from `@prisma/adapter-pg` for Neon compatibility; connection string normalized in `src/lib/db.ts`.
- **Per-PR Neon database branches** — each Vercel preview gets its own DB branch via the Neon integration; `vercel-build` runs `prisma migrate deploy && next build`.
- **Native form + Zod over react-hook-form** — `useState` + `safeParse` on submit. Less ceremony, smaller bundle.
- **Dark mode only** — no toggle, one variable set, less to test.

---

## README design goals

- **Visually strong on GitHub** — good use of badges, centered header, images displayed at readable width
- **Screenshots placed high** — near the top, before the feature table, so visitors see the app immediately
- **Honest** — only lists features that actually exist; planned features are clearly marked ⏳
- **Scannable** — tables, headers, short sentences. A recruiter should get the gist in 30 seconds
- **No filler** — no "The Problem" / "The Solution" marketing speak unless it's concise. Developer-to-developer tone
- **Architecture section stays** — this is a portfolio piece; the "why" behind decisions signals seniority

---

## Current README location

`D:/Projects/Waitly/README.md` — read this as the starting point. You are improving/redesigning it, not starting from scratch.

## Screenshot paths (for `<img>` tags)

```
screenshots/dashboard.png
screenshots/hosted-page.png
screenshots/embed.png
```

Reference them with a relative path from the repo root — GitHub renders them inline.
