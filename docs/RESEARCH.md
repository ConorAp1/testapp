# TestApp — Research & Discovery

## Problem Definition

### Core Problem
Small engineering teams at startups waste significant time in recurring status meetings because there is no single shared place to see who is working on what. Without visibility, teams default to meetings, Slack threads, or asking colleagues directly — all of which interrupt deep work and slow everyone down.

### Problem Statement
> "Teams of 3–10 engineers at early-stage startups lose productive hours every week to status-update meetings and async check-ins that exist only because there is no shared, always-current view of task ownership and progress."

### Who Experiences This
- **Primary users**: Individual engineers on small startup teams who create, own, and complete tasks.
- **Secondary users**: Team leads or founders who need a quick read on team progress without scheduling a meeting.
- **Team size**: 3–10 people. Below 3, the overhead of a tool isn't worth it. Above 10, teams typically have budget and scale for heavier tools like Jira or Linear.

---

## User Research Insights

### Key Frustrations (Target Persona: Engineer at a 5–8 Person Startup)
1. **Meeting fatigue**: Daily standups often run long because people aren't prepared and have no shared reference point.
2. **Invisible work**: Work-in-progress is invisible until it's done or blocked. Teams find out about blockers late.
3. **Assignment ambiguity**: It's unclear who picked up a task or whether something is being worked on at all.
4. **Tool complexity**: Jira and similar tools are seen as too heavy, too slow, and require too much maintenance overhead for a small team.
5. **Notification overload**: Slack-based task tracking drowns in unrelated messages.

### What Users Actually Want
- See at a glance what everyone on the team is working on right now.
- Know the status of any task without having to ask.
- Assign and track tasks in under 30 seconds.
- A dashboard they'd actually open every morning rather than a tool they maintain reluctantly.

### What Users Do NOT Want for MVP
- Complex workflow automation or custom status pipelines.
- Time tracking or story point estimation.
- Integrations with GitHub, Slack, or Figma (nice to have later, not now).
- Gantt charts, roadmaps, or sprint planning views.
- Mobile apps — web is sufficient for the MVP.

---

## Competitive Analysis

### Jira
- **Strengths**: Extremely feature-rich, deep integrations, widely known.
- **Weaknesses for our users**: Slow, complex, requires a dedicated admin, overkill for teams under 15 people, expensive at scale.
- **TestApp advantage**: Zero configuration needed. Open in browser, create a task, done.

### Linear
- **Strengths**: Beautiful UI, fast keyboard shortcuts, opinionated workflow, loved by engineers.
- **Weaknesses for our users**: Paid product after free tier limits, more features than a 5-person startup needs in the first month.
- **TestApp advantage**: Free to self-host (Postgres + Vercel free tier), simpler mental model.

### Trello
- **Strengths**: Visual kanban, very low learning curve, free tier is generous.
- **Weaknesses for our users**: No real team-level visibility dashboard, card ownership is an afterthought, no status tracking beyond column position.
- **TestApp advantage**: Explicit task assignment and status are first-class, not bolt-ons.

### GitHub Issues
- **Strengths**: Already where the code lives, free, familiar to engineers.
- **Weaknesses for our users**: Tied to a repository, no cross-repo team dashboard, non-engineers struggle with the interface.
- **TestApp advantage**: Standalone, accessible to non-technical team members, purpose-built for task visibility.

### Notion
- **Strengths**: Flexible, can be made into a task tracker with effort, good for docs too.
- **Weaknesses for our users**: Requires significant setup and discipline to maintain, the flexibility becomes a liability — every team builds it differently.
- **TestApp advantage**: Opinionated defaults, no setup required, works the same for every team.

### Key Differentiator for TestApp
TestApp is not trying to beat any of these tools on features. The differentiator is **radical simplicity with the right defaults for small teams**. A new team member should be able to understand the full system in under 2 minutes.

---

## Technical Research

### Why Next.js (App Router)?
- Server Components allow data fetching without API round-trips for most views, keeping the dashboard fast.
- Server Actions simplify form handling for task creation and status updates — no separate API layer needed for mutations from the UI.
- Vercel's free tier supports Next.js perfectly, enabling zero-cost deployment within budget constraints.
- TypeScript is first-class in Next.js, which is non-negotiable for a solo developer who needs the compiler as a second reviewer.

### Why Postgres + Prisma?
- Postgres is the industry standard for relational data and handles the task/user/team relationships cleanly.
- Prisma provides type-safe database access — queries return typed results that match our TypeScript interfaces automatically.
- Prisma Migrate gives us a clear, version-controlled schema history even on a solo project.
- **Free Postgres options for MVP**: Supabase free tier (500MB), Neon free tier (3GB), or a local Postgres instance during development. No cost required.
- The relational model is the right fit: Tasks belong to Teams, Tasks are assigned to Users, Users belong to Teams — this is a classic relational schema.

### Why Tailwind CSS?
- No context switching between CSS files and JSX. For a solo developer on a deadline, this is a meaningful productivity gain.
- Consistent design system out of the box via spacing, color, and typography tokens.
- Tree-shaking ensures the production CSS bundle is small.
- No CSS naming decisions to make — another cognitive load reduction for a 2-week sprint.

### Authentication Approach
- For MVP with zero budget: use **NextAuth.js** (free, open source) with the Email (magic link) provider using a free transactional email service, or Credentials provider for email/password.
- Alternative: Skip auth entirely for a very early internal MVP and use a shared team login. **Recommended only if the team is 1–2 people and trust is absolute.**
- Do NOT use Auth0, Clerk, or similar services — they have free tiers but introduce vendor dependency and complexity.

### Hosting & Infrastructure (Zero Budget)
| Service | Purpose | Free Tier |
|---|---|---|
| Vercel | Next.js hosting | Free for hobby/personal projects |
| Supabase or Neon | Managed Postgres | Free tier sufficient for MVP usage |
| GitHub | Version control + CI | Free for public and private repos |

**Total monthly cost at MVP scale: $0.**

---

## Data Model Research

### Core Entities
The TestApp data model for MVP is intentionally minimal:

**User**
- id, email, name, createdAt
- Belongs to one Team (for MVP simplicity)

**Team**
- id, name, createdAt
- Has many Users, has many Tasks

**Task**
- id, title, description (optional), status, createdAt, updatedAt
- Belongs to a Team
- Has one assignee (User, optional — tasks can be unassigned)
- Created by a User

**Task Status Enum**
```
TODO → IN_PROGRESS → IN_REVIEW → DONE
```
This linear flow covers 90% of how small teams actually work. No custom statuses for MVP.

### Key Relationships
- A Team has many Tasks.
- A Task is assigned to at most one User.
- A User belongs to one Team.
- Tasks are always scoped to a Team — cross-team visibility is out of scope for MVP.

---

## MVP Feature Rationale

### Task Creation
**Why it's in MVP**: Without creating tasks, nothing else works. The creation form must be minimal: title (required), description (optional), assignee (optional dropdown of team members), initial status defaults to `TODO`.

**Scope boundary**: No file attachments, no subtasks, no custom fields, no due dates (v2).

### Task Assignment
**Why it's in MVP**: Assignment is the core solution to the visibility problem. If no one is assigned, the task is unowned and invisible.

**Scope boundary**: Single assignee only. No multi-assignee, no watcher/follower system for MVP.

### Status Tracking
**Why it's in MVP**: Status is what replaces the status meeting. Team members can see at a glance that a task moved from `IN_PROGRESS` to `IN_REVIEW` without asking.

**Scope boundary**: Fixed four-status system. No custom statuses, no status transition rules/gates for MVP.

### Simple Dashboard View
**Why it's in MVP**: The dashboard is the primary value delivery mechanism. It answers "what is everyone working on right now?" in a single page load.

**Dashboard must show**:
- Tasks grouped by status (kanban-style columns or list with status filters)
- Assignee for each task visible without clicking
- Quick count: how many tasks are in each status for the team

**Scope boundary**: No analytics, no burndown charts, no velocity tracking for MVP.

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Auth takes too long to implement | Medium | High | Use NextAuth.js with Credentials provider — ~4 hours to implement. If still too slow, ship with a single hardcoded team for the first demo. |
| Postgres setup on free tier has cold-start latency | Low | Medium | Use Neon (serverless Postgres) which handles Next.js serverless functions better than Supabase for cold starts. |
| Scope creep derails 2-week deadline | High | High | No feature is added to the MVP without removing another. The four features above are fixed. |
| Solo developer gets blocked on a bug | Medium | High | Keep the stack conventional. Avoid clever abstractions. Stack Overflow and the Next.js docs cover 99% of problems with this stack. |
| Users find the app too simple and don't adopt it | Low (short-term) | Low (short-term) | Simple is the point for MVP. Adoption feedback after launch will drive v2 priorities. |

---

## Open Questions (To Resolve Before or During Build)

1. **Authentication**: Magic link email or email/password credentials? Magic link is better UX but requires a working email sender. Credentials is simpler to implement in 2 weeks.
2. **Real-time updates**: Should the dashboard auto-refresh when a teammate updates a task? Options: polling every 30 seconds (simple), WebSockets (complex), or manual refresh (simplest). **Recommendation**: Manual refresh or 30-second polling for MVP.
3. **Team creation flow**: For MVP, does a team get created by an admin invite flow, or do we hardcode a single team in the seed? **Recommendation**: Seed a single team, focus on task features for the 2-week build.
4. **Due dates**: Out of MVP scope, but the database column should be added as nullable from the start to avoid a migration headache in v2.

---

## Success Metrics for MVP

After launching TestApp to the first team:
- **Primary**: Does the team hold fewer or shorter status meetings in the first 2 weeks of use?
- **Secondary**: Are tasks being updated (status changes) at least once per day per active user?
- **Engagement**: Do team members open the dashboard at least once per workday?
- **Retention**: Is the team still using TestApp after 30 days without being prompted?

These are qualitative and observational metrics for an MVP. No analytics tooling is needed — just ask the team directly.