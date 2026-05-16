# TestApp — MVP Sprint Plan

## Goal
Ship a working task management app for small engineering teams in **2 weeks**, solo. The definition of done: a real team can sign in, create tasks, assign them, update statuses, and see the dashboard.

---

## Timeline Overview
| Week | Focus |
|------|-------|
| Week 1 | Foundation: project setup, auth, data model, core API |
| Week 2 | UI: dashboard, task forms, status updates, deploy |

---

## Week 1 — Foundation

### Day 1 — Project Bootstrap
- [ ] Scaffold Next.js app with TypeScript: `npx create-next-app@latest testapp --typescript --tailwind --app`
- [ ] Set up ESLint + Prettier config
- [ ] Initialize Git repo, push to GitHub
- [ ] Create `.env.local` from `.env.example`
- [ ] Install Prisma: `npm install prisma @prisma/client`
- [ ] Run `npx prisma init` — configure `DATABASE_URL` pointing to local Postgres
- [ ] Set up free Postgres on Railway or Supabase for development
- [ ] Define initial Prisma schema: `User`, `Task`, `Status` enum, `Priority` enum
- [ ] Run `npx prisma db push` to sync schema
- [ ] Create `lib/prisma.ts` singleton

**Checkpoint:** App boots, Prisma connects to DB, schema is live.

---

### Day 2 — Authentication
- [ ] Install NextAuth.js: `npm install next-auth`
- [ ] Create `app/api/auth/[...nextauth]/route.ts`
- [ ] Configure GitHub OAuth provider in `lib/auth.ts`
- [ ] Register GitHub OAuth app at github.com/settings/developers (free)
- [ ] Add `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, `NEXTAUTH_SECRET` to `.env.local`
- [ ] Implement Prisma adapter for NextAuth so users are persisted to DB: `npm install @auth/prisma-adapter`
- [ ] Add `Account` and `Session` models to Prisma schema (required by NextAuth Prisma adapter)
- [ ] Run `npx prisma db push` again
- [ ] Add sign-in / sign-out buttons to root `layout.tsx`
- [ ] Protect dashboard route: redirect unauthenticated users to sign-in

**Checkpoint:** Can sign in with GitHub, user record appears in DB, protected routes work.

---

### Day 3 — Task API (Create & List)
- [ ] Create `app/api/tasks/route.ts`
  - `GET /api/tasks` — fetch all tasks, include assignee name; support `?status=` and `?assigneeId=` query params
  - `POST /api/tasks` — validate body (title required), create task in DB, return created task
- [ ] Create `types/index.ts` — define `Task`, `TaskCreateInput`, `User`, `Status`, `Priority` types
- [ ] Test both endpoints with `curl` or Postman
- [ ] Add `Content-Type: application/json` validation to POST handler

**Checkpoint:** Can create and list tasks via API.

---

### Day 4 — Task API (Update & Delete) + Users API
- [ ] Create `app/api/tasks/[id]/route.ts`
  - `GET /api/tasks/[id]` — fetch single task with assignee
  - `PATCH /api/tasks/[id]` — partial update (status, assigneeId, title, description, dueDate, priority)
  - `DELETE /api/tasks/[id]` — delete task, return 204
- [ ] Create `app/api/users/route.ts`
  - `GET /api/users` — list all users (id, name, email, image) for assignee dropdown
- [ ] Add basic auth check to all mutation endpoints (only signed-in users can create/update/delete)
- [ ] Test all CRUD operations end-to-end

**Checkpoint:** Full task CRUD works. Users can be listed for assignment.

---

### Day 5 — Shared Components
- [ ] Build `components/StatusBadge.tsx` — pill badge, color-coded by status (gray/blue/green)
- [ ] Build `components/AssigneeSelect.tsx` — dropdown that fetches `/api/users`, renders name + avatar
- [ ] Build `components/TaskCard.tsx` — shows title, status badge, assignee avatar, priority indicator, due date
- [ ] Build `components/DashboardFilters.tsx` — filter controls for status and assignee
- [ ] Build `components/TaskForm.tsx` — shared form for create and edit (title, description, priority, due date, assignee)
- [ ] Add Tailwind `tailwind.config.ts` customization: custom colors for status/priority if needed

**Checkpoint:** All reusable UI components built and render correctly in isolation.

---

## Week 2 — UI & Deploy

### Day 6 — Dashboard Page
- [ ] Create `app/dashboard/page.tsx` as a Server Component
- [ ] Fetch all tasks server-side directly via Prisma (no extra API hop needed from server)
- [ ] Render list of `TaskCard` components
- [ ] Wire up `DashboardFilters` — filter by status, filter by assignee (client-side state)
- [ ] Add empty state: "No tasks yet — create your first one" with link to `/tasks/new`
- [ ] Add task count summary at top: "X tasks · Y in progress · Z done"
- [ ] Add `app/dashboard/loading.tsx` skeleton loader

**Checkpoint:** Dashboard renders all tasks, filters work, looks clean.

---

### Day 7 — Task Creation Page
- [ ] Create `app/tasks/new/page.tsx` — render `TaskForm` in create mode
- [ ] Handle form submission: POST to `/api/tasks`, redirect to `/dashboard` on success
- [ ] Show inline validation errors (title required, due date must be future)
- [ ] Add "Cancel" link back to dashboard
- [ ] Add "New Task" button to dashboard header that links to `/tasks/new`

**Checkpoint:** Can create a task from the UI and see it appear on the dashboard.

---

### Day 8 — Task Detail & Edit Page
- [ ] Create `app/tasks/[id]/page.tsx` — fetch task server-side, render in edit mode using `TaskForm`
- [ ] Handle form submission: PATCH to `/api/tasks/[id]`, redirect to `/dashboard` on success
- [ ] Add status update buttons directly on the detail page: "Mark In Progress" / "Mark Done" / "Reopen"
- [ ] Add delete button with confirmation prompt (`window.confirm` is fine for MVP)
- [ ] Make `TaskCard` on dashboard clickable — links to `/tasks/[id]`

**Checkpoint:** Can click a task, edit it, change its status, and delete it.

---

### Day 9 — Polish & Edge Cases
- [ ] Add a proper landing page at `app/page.tsx` — if logged in, redirect to `/dashboard`; if not, show sign-in prompt with 1-sentence value prop for TestApp
- [ ] Add navigation header: TestApp logo, link to Dashboard, user avatar + sign-out
- [ ] Handle loading and error states gracefully across all pages
- [ ] Test the full user flow: sign in → create task → assign to teammate → update status → view on dashboard
- [ ] Fix any TypeScript errors (`npx tsc --noEmit`)
- [ ] Fix any lint errors (`npm run lint`)
- [ ] Test on mobile viewport (Tailwind responsive classes as needed)
- [ ] Seed DB with a few sample tasks for demo purposes (`prisma/seed.ts`)

**Checkpoint:** App is fully functional, no crashes, looks presentable.

---

### Day 10 — Deploy
- [ ] Create production Postgres database on Railway or Supabase (free tier)
- [ ] Set production `DATABASE_URL` in Vercel environment variables
- [ ] Run `npx prisma migrate deploy` against production DB
- [ ] Connect GitHub repo to Vercel — import project
- [ ] Set all environment variables in Vercel dashboard:
  - `DATABASE_URL`
  - `NEXTAUTH_SECRET` (generate with `openssl rand -base64 32`)
  - `NEXTAUTH_URL` (set to production Vercel URL)
  - `GITHUB_CLIENT_ID`
  - `GITHUB_CLIENT_SECRET`
- [ ] Update GitHub OAuth app callback URL to production URL
- [ ] Trigger Vercel deployment
- [ ] Smoke test production: sign in, create task, assign, update status, view dashboard
- [ ] Share URL with at least one other person to validate it works for a real user

**Checkpoint:** TestApp is live. Real users can access it.

---

## MVP Definition of Done
- [ ] User can sign in with GitHub
- [ ] User can create a task with title, description, priority, and due date
- [ ] User can assign a task to any team member who has signed in
- [ ] User can move a task through `TODO → IN_PROGRESS → DONE`
- [ ] User can see all tasks on a dashboard filtered by status or assignee
- [ ] App is deployed and accessible at a public URL
- [ ] App does not crash on any happy-path user action

---

## Parking Lot (post-MVP only)
These ideas are captured here so they don't distract from the 2-week goal:

- Email notifications when a task is assigned to you
- Drag-and-drop Kanban board view
- Comments / activity feed on tasks
- Due date reminders
- Task labels / tags
- Multiple team workspaces
- Role-based permissions (admin vs. member)
- Slack integration