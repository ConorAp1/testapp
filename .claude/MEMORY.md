# TestApp — Claude Memory File

## Project Identity
- **Name:** TestApp
- **Type:** Task management web application
- **Purpose:** Give small engineering teams shared visibility on who is doing what, eliminating wasted status meetings.
- **Stage:** MVP (targeting 2-week launch)

## Target Users
Small engineering teams of 3–10 people at early-stage startups. They are technical, move fast, and have no patience for bloated tools. They need something lightweight that just works.

## MVP Feature Set
1. **Task Creation** — Create tasks with title, description, priority, and due date.
2. **Task Assignment** — Assign tasks to team members.
3. **Status Tracking** — Move tasks through states: `TODO → IN_PROGRESS → DONE`.
4. **Simple Dashboard** — At-a-glance view of all tasks, filterable by assignee and status.

## Tech Stack
| Layer | Technology |
|-------|-----------|
| Framework | Next.js (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS |
| Database | PostgreSQL |
| ORM | Prisma |
| Auth | NextAuth.js (free tier, credentials or GitHub OAuth) |
| Deployment | Vercel (free tier) + Supabase or Railway for Postgres (free tier) |

## Key Constraints
- **Timeline:** 2 weeks to launch MVP — no scope creep.
- **Team size:** Solo developer — keep decisions simple, avoid over-engineering.
- **Budget:** $0 — only free-tier services allowed.

## Decisions Made
- Using Next.js App Router (not Pages Router) — modern, RSC-compatible.
- Prisma as ORM for type-safe DB access and easy migrations.
- No separate backend — API routes live inside Next.js under `app/api/`.
- Tailwind for styling — no component library to keep bundle lean; custom components only.
- Authentication handled by NextAuth.js with GitHub OAuth (free, no email service needed).
- No real-time updates for MVP — polling or manual refresh is acceptable.
- No billing, no teams/orgs model for MVP — single flat workspace.

## What Has Been Built (update as work progresses)
- [ ] Project scaffolded
- [ ] Prisma schema defined
- [ ] Auth configured
- [ ] Task CRUD API routes
- [ ] Dashboard page
- [ ] Task creation form
- [ ] Assignment UI
- [ ] Status update UI
- [ ] Deployed to Vercel

## Patterns to Remember
- All database access goes through Prisma client in `lib/prisma.ts`.
- API routes return consistent JSON shape: `{ data, error }`.
- Server Components fetch data directly; Client Components use fetch to API routes.
- Tailwind classes only — no inline styles, no CSS modules.
- TypeScript strict mode is ON — no `any`, no `ts-ignore` without comment.