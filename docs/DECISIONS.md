# TestApp — Architecture Decision Records (ADR)

This document records the significant technical and product decisions made during the development of TestApp, along with the context, options considered, and rationale. This helps future contributors (or the solo developer returning after time away) understand *why* things are built the way they are.

---

## ADR-001: Use Next.js App Router Instead of Pages Router

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp is a full-stack application where data fetching, rendering, and mutations all need to work together efficiently. The project must be built by a solo developer and launched in 2 weeks, so the framework choice heavily impacts development speed.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| Next.js App Router | Server Components reduce boilerplate; Server Actions eliminate separate API layer for mutations; co-located data fetching | Newer API with some rough edges; less community examples |
| Next.js Pages Router | More mature; abundant examples | `getServerSideProps` is verbose; needs separate API routes for mutations |
| Remix | Great data model with loaders/actions | Less familiar; smaller ecosystem; steeper learning curve for solo dev |
| Create React App + Express | Full control | Way too much setup for a 2-week timeline |

### Decision
Use **Next.js 14 with App Router**.

### Rationale
Server Components allow the dashboard page to fetch tasks directly from Postgres via Prisma without an intermediate API call. Server Actions handle task creation, assignment, and status updates with minimal boilerplate. `revalidatePath` keeps the UI in sync after mutations. This combination eliminates the need to build and maintain a separate API layer for internal operations, saving significant time for a solo developer.

---

## ADR-002: Server Actions for Mutations Instead of REST API Endpoints

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp needs to support task creation, task assignment changes, and status updates. These mutations could be implemented as REST API routes (`/api/tasks`, `/api/tasks/[id]`) or as Next.js Server Actions.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| Server Actions | Co-located with UI; no fetch boilerplate; type-safe end-to-end; automatic CSRF protection | Not suitable if external clients (mobile, third-party) need the same endpoints |
| REST API Routes | Reusable by external clients; familiar pattern | More files to maintain; need to write fetch calls on client; more boilerplate |

### Decision
Use **Server Actions** for all MVP mutations.

### Rationale
TestApp has no external API consumers in the MVP. A mobile app and external integrations are post-MVP considerations. Server Actions let mutations live next to the components that trigger them, reduce code surface area, and are type-safe through the TypeScript/Prisma stack. If external API access is needed later, REST routes can be added incrementally without changing the existing Server Actions.

---

## ADR-003: NextAuth.js for Authentication

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp requires user authentication so that tasks can be attributed to creators and assignees. The constraint is zero budget — no paid auth services (Auth0, Clerk, etc.).

### Options Considered

| Option | Cost | Pros | Cons |
|---|---|---|---|
| NextAuth.js (GitHub OAuth) | Free | Easy setup; no email service; users already have GitHub at startups | Requires users to have GitHub accounts |
| NextAuth.js (Credentials) | Free | Works for any user | Need to handle password hashing, reset flows |
| Clerk | Free tier, then paid | Excellent DX; pre-built UI | Free tier limits; vendor dependency |
| Auth0 | Free tier, then paid | Industry standard | Overkill for MVP; free tier limits |
| Roll your own JWT | Free | Full control | Too much time to build securely on a 2-week timeline |

### Decision
Use **NextAuth.js with GitHub OAuth** as the primary provider, with a Credentials provider as a fallback for teams who prefer email/password.

### Rationale
Target users are small engineering teams at startups — the vast majority will have GitHub accounts. GitHub OAuth requires no email service, no password reset flow, and no SMTP configuration, all of which would consume development time. NextAuth.js is free, integrates cleanly with the Next.js App Router, and handles the session/JWT complexity automatically. The Credentials provider fallback ensures no team is locked out.

---

## ADR-004: PostgreSQL via Supabase Free Tier

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp needs a relational database to store users, tasks, and assignments. The constraint is zero budget for paid services.

### Options Considered

| Option | Cost | Pros | Cons |
|---|---|---|---|
| Supabase (Postgres) | Free tier: 500MB | Managed Postgres; free PgBouncer; easy setup | 500MB limit (more than enough for MVP) |
| Railway (Postgres) | Free tier available | Simple DX | Credit-based free tier can expire |
| PlanetScale (MySQL) | Free tier deprecated | Was popular | No longer free |
| SQLite (local/Turso) | Free | Zero setup | Less familiar; Turso has connection limits on free tier |
| Self-hosted Postgres | Free (if on VPS) | Full control | Requires a VPS; no budget for that |

### Decision
Use **Supabase for PostgreSQL hosting**.

### Rationale
Supabase provides a managed PostgreSQL instance with 500MB storage and built-in connection pooling (PgBouncer) on its free tier. For a team of 3–10 people with hundreds of tasks, this is more than adequate. The setup time is under 5 minutes. PgBouncer support is important because Prisma can exhaust Postgres connections in serverless environments — Supabase handles this for free.

---

## ADR-005: Prisma as the ORM

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp is written in TypeScript and needs a way to interact with PostgreSQL. The ORM choice affects type safety, migration workflow, and query ergonomics.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| Prisma | Excellent TypeScript integration; auto-generated types; declarative migrations; great DX | Slightly slower queries than raw SQL; generates large client bundle |
| Drizzle ORM | Type-safe; lightweight; SQL-like syntax | Less mature; fewer examples; migration tooling less mature |
| Kysely | Type-safe query builder; lightweight | More verbose; no schema-based type generation |
| Raw SQL (pg) | Maximum control and performance | No type safety; manual migration management |

### Decision
Use **Prisma**.

### Rationale
Prisma's schema-first approach means the `schema.prisma` file is the single source of truth for the database structure and TypeScript types. This eliminates a whole class of bugs where the DB schema and application types drift apart. For a solo developer under time pressure, the `prisma migrate dev` and `prisma studio` tools dramatically speed up iteration. The performance trade-off is irrelevant at TestApp's MVP scale.

---

## ADR-006: No Real-Time Updates for MVP

**Date**: Project kickoff
**Status**: Accepted

### Context
One of TestApp's core value propositions is shared visibility into team tasks. Real-time updates (where the dashboard refreshes when a teammate changes a task status) would enhance this. Supabase offers a free Realtime service.

### Options Considered

| Option | Complexity | Value |
|---|---|---|
| Supabase Realtime (WebSockets) | Medium | High — live updates without refresh |
| Server-Sent Events | Medium | High — one-way live updates |
| Polling (setInterval fetch) | Low | Medium — near-real-time with overhead |
| Manual refresh only | None | Low — user must refresh to see changes |

### Decision
**Manual refresh only for MVP.** Users navigate between pages or refresh to see latest data.

### Rationale
The 2-week launch deadline is the deciding factor. Implementing real-time updates requires additional client-side state management, connection handling, and error recovery logic. The core problem TestApp solves (eliminating status meetings) does not require real-time updates — it requires *shared* visibility. A team checking the dashboard before their standup will see fresh data. Real-time is a polish feature, not a prerequisite for the MVP value proposition. This can be added post-launch using Supabase Realtime with relatively isolated changes.

---

## ADR-007: No Multi-Team / Workspace Isolation for MVP

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp's target users are small teams at startups. A common SaaS pattern is to support multiple "workspaces" or "organizations" so different teams can use the same app in isolation. This affects the data model significantly.

### Options Considered

| Option | Data Model Impact | Timeline Impact |
|---|---|---|
| Multi-tenant workspaces | Add `Workspace` model; all queries scoped by workspace | +3–5 days of development |
| Single shared space (MVP) | No workspace concept; all users see all tasks | 0 days |

### Decision
**No workspace isolation for MVP.** All users in the database are considered one team and can see all tasks.

### Rationale
TestApp is targeting a single small team deploying their own instance. In the MVP scenario, the app is deployed once for one team. There is no need to separate data between multiple teams in the same deployment. If TestApp grows into a multi-tenant SaaS product, workspace isolation can be added by introducing a `Workspace` model and scoping all Prisma queries accordingly — but this is a post-launch architectural upgrade, not an MVP requirement.

---

## ADR-008: Tailwind CSS for Styling

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp needs a consistent, professional-looking UI built quickly by a single developer with no dedicated designer.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| Tailwind CSS | Fast iteration; no context switching; good defaults | Verbose class names; no design system out of the box |
| CSS Modules | Scoped styles; familiar | Slow to write; requires naming things |
| Styled Components / Emotion | Component-level styles in JS | Runtime overhead; extra setup |
| shadcn/ui (Tailwind-based) | Pre-built accessible components | Additional dependency; learning curve |
| MUI / Ant Design | Full component library | Heavy; opinionated; hard to customize |

### Decision
Use **Tailwind CSS** with selective use of **shadcn/ui** components for common UI primitives (Button, Input, Select, Badge).

### Rationale
Tailwind eliminates the CSS naming problem and enables rapid UI construction without leaving the component file. shadcn/ui provides accessible, pre-built components (forms, dropdowns, badges) that match exactly what TestApp's task management UI needs, without the lock-in of a full component library. Both work together natively and have zero runtime cost beyond the CSS bundle.

---

## ADR-009: Vercel for Hosting

**Date**: Project kickoff
**Status**: Accepted

### Context
TestApp needs to be deployed and accessible to the team. The constraint is zero budget.

### Options Considered

| Option | Cost | Setup Time | Suitability for Next.js |
|---|---|---|---|
| Vercel | Free for hobby/small projects | < 5 minutes | First-class |
| Netlify | Free tier | 10 minutes | Good but not native |
| Railway | Free tier (credit-based) | 15 minutes | Good |
| Self-hosted (VPS) | ~$5/mo minimum | Hours | Full control |
| Fly.io | Free tier available | 30 minutes | Good |

### Decision
Use **Vercel** for Next.js hosting.

### Rationale
Vercel is the creator of Next.js and provides first-class support for all its features including Server Actions, App Router, and edge functions. Deployment is triggered automatically on `git push`. The free Hobby tier supports the MVP entirely. There is no configuration, no Dockerfile, and no ops overhead — critical for a solo developer with a 2-week deadline.

---

## ADR-010: JWT Sessions Instead of Database Sessions

**Date**: Project kickoff
**Status**: Accepted

### Context
NextAuth.js supports two session strategies: JWT (stateless, stored in a cookie) and Database (session records stored in Postgres).

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| JWT sessions | No extra DB table queries per request; stateless; simpler | Cannot immediately invalidate sessions server-side |
| Database sessions | Can revoke sessions instantly; auditable | Extra DB query on every authenticated request |

### Decision
Use **JWT sessions** for MVP.

### Rationale
For a small internal team tool, the inability to immediately revoke sessions is an acceptable trade-off. The JWT strategy means no `Session` table queries on every page load, reducing database load and latency. If a user needs to be removed from the team, their tasks can be unassigned and their account deleted — their JWT will expire within the configured session duration (default 30 days). This is fine for MVP. Database sessions can be switched on with a one-line config change in NextAuth if needed later.