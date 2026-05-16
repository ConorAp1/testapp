# TestApp — Product Requirements Document

**Version:** 1.0  
**Status:** Active  
**Author:** Solo Developer  
**Target Launch:** 2 weeks from project start

---

## 1. Overview

### 1.1 Product Summary

TestApp is a lightweight task management web application built for small engineering teams at startups. It provides a shared, real-time view of tasks, ownership, and status so teams can stay aligned without relying on status meetings.

### 1.2 Problem Statement

Small startup engineering teams (3–10 people) frequently lose time to status check-in meetings — standups that run long, Slack threads asking "where is X?", and duplicated work caused by a lack of shared visibility. Existing tools like Jira or Linear are either too heavyweight, too expensive, or require too much setup for a team that just needs to know who is doing what right now.

### 1.3 Proposed Solution

TestApp is a focused, no-frills task board where team members can create tasks, assign them to teammates, update their status, and see everything at a glance on a shared dashboard. It does one thing well: replaces the status meeting with a shared source of truth.

### 1.4 Success Metrics (MVP)

- A team of 3–10 people can onboard and start tracking tasks within 10 minutes
- All MVP features are functional and bug-free at launch
- The app loads the dashboard in under 2 seconds on a standard connection
- Zero paid infrastructure required to run the app

---

## 2. Target Users

### Primary Users: Small Engineering Teams at Startups

| Attribute | Detail |
|---|---|
| Team size | 3–10 engineers |
| Company stage | Early-stage startup (seed to Series A) |
| Technical sophistication | High — engineers are comfortable with web tools |
| Pain point | No shared visibility on task status; too many sync meetings |
| Current workarounds | Spreadsheets, Notion pages, Slack threads, or nothing |

### User Personas

**Alex – Lead Engineer**
- Wants to know what everyone on the team is working on without having to ask
- Needs to reassign tasks when priorities shift
- Frustrated by Jira's overhead for a 5-person team

**Jordan – Software Engineer**
- Wants a simple place to see their own task list and update status
- Doesn't want to spend time on process or tooling
- Needs to communicate blockers quickly

---

## 3. MVP Feature Specifications

The MVP must ship in 2 weeks, built by a single developer. Features are scoped ruthlessly to what delivers core value.

---

### 3.1 Task Creation

**Description:** Any team member can create a new task.

**Acceptance Criteria:**
- User can fill out a form with the following fields:
  - `Title` (required, string, max 150 characters)
  - `Description` (optional, text, max 1000 characters)
  - `Due Date` (optional, date picker)
  - `Assignee` (optional at creation, select from team members)
  - `Status` (defaults to `To Do`)
- Submitting the form creates a task record in the database via Prisma
- The new task appears immediately on the dashboard without a full page reload
- Empty title shows an inline validation error, form does not submit
- Task creation is accessible from the dashboard via a clearly visible "New Task" button

**Out of Scope (v1):**
- File attachments
- Task templates
- Labels or tags
- Priority levels

---

### 3.2 Task Assignment

**Description:** Tasks can be assigned to a specific team member.

**Acceptance Criteria:**
- A task can be assigned to one team member (single assignee)
- Assignment can be set at task creation or edited afterward
- The assignee is displayed on the task card on the dashboard
- The assignee dropdown lists all users in the system
- A task can be unassigned (assignee field set to null)
- Reassigning a task updates the record in the database immediately

**Out of Scope (v1):**
- Multiple assignees per task
- Email or Slack notifications on assignment
- Workload view (tasks per person)

---

### 3.3 Status Tracking

**Description:** Each task has a status that reflects its current state in the workflow.

**Acceptance Criteria:**
- Tasks support exactly four statuses:
  - `To Do` — default for new tasks
  - `In Progress` — actively being worked on
  - `Blocked` — cannot progress due to a dependency or blocker
  - `Done` — completed
- Status can be updated from the task detail view or inline on the dashboard
- Status changes are persisted to the database immediately (no manual save required)
- The dashboard visually distinguishes tasks by status (e.g., color-coded badges using Tailwind)
- `Blocked` tasks are visually distinct to draw attention

**Out of Scope (v1):**
- Custom statuses
- Status change history/audit log
- Automations triggered by status change

---

### 3.4 Simple Dashboard View

**Description:** A single-page view showing all tasks across the team.

**Acceptance Criteria:**
- Dashboard is the default landing page (`/dashboard`)
- Displays all tasks in a list or card layout
- Each task card shows: Title, Assignee name (or "Unassigned"), Status badge, Due date (if set)
- Tasks can be filtered by:
  - Status (All / To Do / In Progress / Blocked / Done)
  - Assignee (All / specific team member)
- Clicking a task opens a detail/edit view
- Dashboard refreshes data on load; no stale data shown
- Works on desktop screens (mobile-responsive is a nice-to-have, not required for MVP)

**Out of Scope (v1):**
- Kanban/column board view
- Sorting options
- Search
- Pagination (MVP assumes <100 tasks)

---

## 4. Technical Specifications

### 4.1 Tech Stack

| Layer | Choice | Rationale |
|---|---|---|
| Framework | Next.js 14 (App Router) | Full-stack capability, easy Vercel deployment, no separate backend needed |
| Language | TypeScript | Type safety, better developer experience, fewer runtime bugs |
| Styling | Tailwind CSS | Fast UI development, no need for a component library in v1 |
| Database | PostgreSQL | Reliable relational DB, free tier available on Neon |
| ORM | Prisma | Type-safe database access, easy migrations, great DX with TypeScript |
| Hosting | Vercel (free tier) | Zero-config Next.js deployment, free for this scale |
| DB Hosting | Neon or Supabase (free tier) | Managed Postgres, no cost at MVP scale |

### 4.2 Data Models

#### User
```prisma
model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  tasks     Task[]   @relation("AssignedTasks")
}
```

#### Task
```prisma
model Task {
  id          String    @id @default(cuid())
  title       String
  description String?
  status      Status    @default(TODO)
  dueDate     DateTime?
  assigneeId  String?
  assignee    User?     @relation("AssignedTasks", fields: [assigneeId], references: [id])
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}

enum Status {
  TODO
  IN_PROGRESS
  BLOCKED
  DONE
}
```

### 4.3 Key Pages & Routes

| Route | Description |
|---|---|
| `/` | Redirects to `/dashboard` |
| `/dashboard` | Main task dashboard view |
| `/tasks/new` | Task creation form |
| `/tasks/[id]` | Task detail and edit view |
| `/api/tasks` | GET all tasks, POST new task |
| `/api/tasks/[id]` | GET, PATCH, DELETE a specific task |
| `/api/users` | GET all users (for assignee dropdown) |

### 4.4 Non-Functional Requirements

- **Performance:** Dashboard must load in under 2 seconds with up to 100 tasks
- **Reliability:** No data loss on task creation or status update
- **Compatibility:** Works in latest versions of Chrome, Firefox, and Safari
- **Security:** No sensitive user data beyond name and email; no auth in v1 (team trust model)
- **Cost:** Zero paid services required to run the application

---

## 5. Out of Scope for MVP

The following features are explicitly excluded from the MVP to meet the 2-week deadline. They may be considered for future iterations.

| Feature | Reason Excluded |
|---|---|
| User authentication / login | Adds significant complexity; small team can operate on trust |
| Email notifications | Requires external service (SendGrid, etc.); no budget |
| Mobile app | Web-only is sufficient for MVP |
| Comments on tasks | Nice-to-have, not core to the problem |
| Activity / audit log | Adds complexity; not needed for initial value |
| Integrations (Slack, GitHub) | Scope risk; can be v2 |
| Kanban board view | List view is sufficient for MVP |
| Search | Not needed with <100 tasks |
| Multiple workspaces / orgs | Single team per deployment is fine for MVP |
| Dark mode | UI nice-to-have, not a priority |

---

## 6. Launch Plan

### Timeline

| Week | Goals |
|---|---|
| Week 1 | Project setup, database schema, task CRUD (create, read, update), basic dashboard |
| Week 2 | Filtering, status updates, UI polish, bug fixing, deploy to Vercel + Neon |

### Definition of Done (MVP Launch)

- [ ] All four MVP features are functional end-to-end
- [ ] App is deployed and accessible via a public URL
- [ ] No known data-loss bugs
- [ ] Dashboard loads successfully with seeded test data
- [ ] README documents how to run the project locally

### Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Scope creep stretches timeline | Medium | Refer back to this PRD; defer anything not listed as MVP |
| Postgres setup issues on free tier | Low | Use Neon (simple connection string, no config needed) |
| Next.js App Router complexity | Medium | Keep data fetching simple; use Server Components for reads, API routes for writes |
| Solo developer burnout / blockers | Medium | Keep the scope minimal; ship something working over something perfect |

---

## 7. Future Considerations (Post-MVP)

These are ideas worth revisiting once the MVP is validated with real users:

- **Authentication** – Add NextAuth.js for proper user login so teams can self-serve onboarding
- **Task comments** – Allow teammates to leave notes on a task
- **Notifications** – Notify assignees when a task is assigned or a due date is approaching
- **Kanban view** – Drag-and-drop board view as an alternative to the list dashboard
- **Slack integration** – Post task updates to a team Slack channel
- **Due date reminders** – Highlight or flag overdue tasks prominently
- **Multi-team/workspace support** – Allow multiple teams to use separate instances or workspaces

---

*This PRD is the source of truth for the TestApp MVP. Any feature requests or scope changes during the 2-week build period should be evaluated against the launch deadline and explicitly added or deferred here.*