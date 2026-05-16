# TestApp — Technical Design Document

## Overview

TestApp is a lightweight task management web application built for small engineering teams (3–10 people) at startups. It eliminates the need for recurring status meetings by providing a shared, real-time view of who is working on what and what the current status of every task is.

This document describes the technical architecture, data model, API design, and component structure for the MVP.

---

## Goals & Constraints

| Goal | Detail |
|---|---|
| Launch deadline | 2 weeks from project start |
| Developer capacity | 1 solo developer |
| Budget | $0 — no paid third-party services |
| Target scale | 3–10 users per team, tens to low hundreds of tasks |
| Primary value | Shared task visibility without meeting overhead |

These constraints heavily influence every architectural decision. Simplicity, speed of implementation, and zero-cost infrastructure are prioritized over scalability or extensibility.

---

## Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| Framework | Next.js 14 (App Router) | Full-stack in one repo; API routes + SSR; fast to build with |
| Language | TypeScript | Type safety across frontend and backend; Prisma requires it |
| Styling | Tailwind CSS | Utility-first; no design system overhead; fast UI iteration |
| Database | PostgreSQL | Relational model fits tasks/users/assignments well; free to self-host |
| ORM | Prisma | Type-safe DB queries; schema migrations; great DX with TypeScript |
| Hosting | Vercel (frontend) + Supabase or Railway (Postgres) | Free tiers available; zero DevOps overhead for solo dev |
| Auth | NextAuth.js (credentials or GitHub OAuth) | Free; integrates with Next.js App Router; no auth service costs |

---

## Architecture Overview

```
┌─────────────────────────────────────────────┐
│                   Browser                   │
│         Next.js React Components            │
│         (App Router, Client Components)     │
└────────────────────┬────────────────────────┘
                     │ HTTP / Server Actions
┌────────────────────▼────────────────────────┐
│             Next.js Server Layer            │
│   - Server Components (data fetching)       │
│   - API Routes (/api/*)                     │
│   - Server Actions (mutations)              │
└────────────────────┬────────────────────────┘
                     │ Prisma Client
┌────────────────────▼────────────────────────┐
│              PostgreSQL Database            │
│   - Hosted on Supabase / Railway free tier  │
└─────────────────────────────────────────────┘
```

### Key Architectural Decisions

- **App Router over Pages Router**: Enables server components for direct DB access without extra API round-trips on initial load, reducing boilerplate.
- **Server Actions for mutations**: Task creation, assignment changes, and status updates are handled via Next.js Server Actions rather than REST endpoints where possible. This keeps the code co-located with the UI and avoids building a separate API layer for internal operations.
- **No separate API service**: Given the 2-week timeline and solo developer, a separate backend service would introduce unnecessary complexity. Next.js handles everything.
- **No real-time WebSockets for MVP**: The dashboard will use standard page navigation and manual refresh. Real-time updates (e.g., via Supabase Realtime) are a post-MVP consideration.

---

## Data Model

### Entity Relationship Overview

```
User ──< TaskAssignment >── Task
                            Task ── Status (enum)
```

### Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(cuid())
  name          String
  email         String    @unique
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  assignedTasks Task[]    @relation("AssignedTo")
  createdTasks  Task[]    @relation("CreatedBy")
  accounts      Account[]
  sessions      Session[]
}

model Task {
  id          String     @id @default(cuid())
  title       String
  description String?
  status      TaskStatus @default(TODO)
  priority    Priority   @default(MEDIUM)
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  assigneeId  String?
  assignee    User?      @relation("AssignedTo", fields: [assigneeId], references: [id])

  creatorId   String
  creator     User       @relation("CreatedBy", fields: [creatorId], references: [id])
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  IN_REVIEW
  DONE
}

enum Priority {
  LOW
  MEDIUM
  HIGH
}

// NextAuth required models
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```

---

## Folder Structure

```
testapp/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx            # Login page
│   │   └── layout.tsx
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   │   └── page.tsx            # Main dashboard — task overview
│   │   ├── tasks/
│   │   │   ├── page.tsx            # Task list view
│   │   │   ├── new/
│   │   │   │   └── page.tsx        # Create task form
│   │   │   └── [id]/
│   │   │       └── page.tsx        # Task detail / edit
│   │   └── layout.tsx              # Dashboard shell with nav
│   ├── api/
│   │   └── auth/
│   │       └── [...nextauth]/
│   │           └── route.ts        # NextAuth handler
│   ├── layout.tsx                  # Root layout
│   └── page.tsx                    # Root redirect to /dashboard
├── components/
│   ├── ui/                         # Base UI primitives (Button, Input, Badge, etc.)
│   ├── tasks/
│   │   ├── TaskCard.tsx            # Individual task card for dashboard
│   │   ├── TaskForm.tsx            # Create/edit task form
│   │   ├── TaskList.tsx            # Filtered/sorted task list
│   │   ├── TaskStatusBadge.tsx     # Status pill component
│   │   └── AssigneeSelector.tsx    # User dropdown for assignment
│   └── dashboard/
│       └── DashboardStats.tsx      # Summary counts by status
├── lib/
│   ├── prisma.ts                   # Prisma client singleton
│   ├── auth.ts                     # NextAuth config
│   └── utils.ts                    # Shared utility functions
├── actions/
│   ├── tasks.ts                    # Server Actions: createTask, updateTask, deleteTask
│   └── users.ts                    # Server Actions: getTeamMembers
├── types/
│   └── index.ts                    # Shared TypeScript types (beyond Prisma types)
├── prisma/
│   ├── schema.prisma
│   └── seed.ts                     # Dev seed script with sample tasks/users
├── public/
├── .env.local
├── .env.example
├── tailwind.config.ts
├── next.config.ts
├── tsconfig.json
└── package.json
```

---

## MVP Feature Implementation

### 1. Task Creation

**Route**: `POST` via Server Action in `actions/tasks.ts`
**UI**: `/tasks/new` — a form with fields: title (required), description (optional), assignee (dropdown of team members), priority (LOW/MEDIUM/HIGH)

```typescript
// actions/tasks.ts
'use server'
import { prisma } from '@/lib/prisma'
import { revalidatePath } from 'next/cache'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'

export async function createTask(formData: FormData) {
  const session = await getServerSession(authOptions)
  if (!session?.user?.id) throw new Error('Unauthorized')

  await prisma.task.create({
    data: {
      title: formData.get('title') as string,
      description: formData.get('description') as string | undefined,
      assigneeId: formData.get('assigneeId') as string | undefined,
      priority: formData.get('priority') as Priority,
      creatorId: session.user.id,
    },
  })

  revalidatePath('/dashboard')
  revalidatePath('/tasks')
}
```

### 2. Task Assignment

Assignment is handled at creation time or via an edit action. The `AssigneeSelector` component fetches the list of team users and renders a `<select>` dropdown. For MVP, "team" means all users in the database — no multi-tenant workspace isolation.

### 3. Status Tracking

Status updates are handled via an `updateTaskStatus` Server Action. On task cards, the status is shown as a colored badge and can be changed via a dropdown select directly on the card, triggering the action on change.

```typescript
export async function updateTaskStatus(taskId: string, status: TaskStatus) {
  const session = await getServerSession(authOptions)
  if (!session?.user?.id) throw new Error('Unauthorized')

  await prisma.task.update({
    where: { id: taskId },
    data: { status },
  })

  revalidatePath('/dashboard')
  revalidatePath('/tasks')
}
```

### 4. Dashboard View

**Route**: `/dashboard`
**Implementation**: Server Component that fetches all tasks with assignee data in a single Prisma query. Displays:
- Summary row: counts of tasks by status (TODO, IN_PROGRESS, IN_REVIEW, DONE)
- Task cards grouped by status column (Kanban-lite layout using CSS Grid/Flexbox)
- Each card shows: title, assignee avatar/name, priority badge, last updated time

```typescript
// app/(dashboard)/dashboard/page.tsx
import { prisma } from '@/lib/prisma'

export default async function DashboardPage() {
  const tasks = await prisma.task.findMany({
    include: { assignee: true, creator: true },
    orderBy: { updatedAt: 'desc' },
  })

  // Group by status for column layout
  const grouped = {
    TODO: tasks.filter(t => t.status === 'TODO'),
    IN_PROGRESS: tasks.filter(t => t.status === 'IN_PROGRESS'),
    IN_REVIEW: tasks.filter(t => t.status === 'IN_REVIEW'),
    DONE: tasks.filter(t => t.status === 'DONE'),
  }

  return <DashboardView grouped={grouped} />
}
```

---

## API Routes

For MVP, most data operations go through Server Actions. The only REST API route needed is for NextAuth.

| Route | Method | Purpose |
|---|---|---|
| `/api/auth/[...nextauth]` | GET/POST | NextAuth session management |

Post-MVP candidates: `/api/tasks` REST endpoints if a mobile client or external integration is needed.

---

## Authentication

- **Provider**: NextAuth.js with GitHub OAuth (free, no email service required)
- **Fallback**: Credentials provider with bcrypt-hashed passwords for teams without GitHub
- **Session strategy**: JWT (no database session table needed for MVP, reducing complexity)
- **Authorization**: Every Server Action checks `getServerSession` and throws if unauthenticated. No role-based access control for MVP — all authenticated users can create, assign, and update any task.

---

## Styling Conventions

- Tailwind utility classes only — no custom CSS files except `globals.css` for base resets
- Color palette:
  - TODO: `gray`
  - IN_PROGRESS: `blue`
  - IN_REVIEW: `yellow`
  - DONE: `green`
- Priority colors: LOW = `slate`, MEDIUM = `orange`, HIGH = `red`
- Mobile-responsive layouts using Tailwind responsive prefixes (`sm:`, `md:`)
- Dashboard columns scroll horizontally on small screens

---

## Database & Hosting (Free Tier Plan)

| Service | Free Tier Limits | Usage in TestApp |
|---|---|---|
| Vercel | 100GB bandwidth, serverless functions | Next.js hosting |
| Supabase | 500MB DB, 2GB bandwidth | PostgreSQL database |
| GitHub | Unlimited public/private repos | Source control |

**Connection pooling**: Supabase provides PgBouncer. Use the pooled connection string for Prisma in production (`?pgbouncer=true&connection_limit=1`).

---

## Performance Considerations (MVP Scope)

- Server Components fetch data directly — no client-side API calls for initial renders
- `revalidatePath` after mutations keeps data fresh without full page reloads
- No pagination for MVP (teams are small, task counts will be low)
- Images/avatars: use GitHub OAuth avatar URLs directly — no image hosting needed

---

## Post-MVP Considerations (Out of Scope for Launch)

- Real-time updates via Supabase Realtime or Server-Sent Events
- Task comments / activity feed
- Due dates and deadline notifications
- Multi-team / workspace isolation
- Email notifications
- Drag-and-drop Kanban reordering
- Task labels/tags
- Search and filtering