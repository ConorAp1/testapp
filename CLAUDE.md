# CLAUDE.md — TestApp

This file provides guidance for Claude when working on the TestApp codebase. Read this before making any changes.

## Project Overview

**TestApp** is a task management application built for small engineering teams (3–10 people) at startups. It solves the problem of wasted time in status meetings by giving teams shared, real-time visibility into who is doing what.

**MVP Features:**
- Task creation
- Task assignment (to team members)
- Status tracking (e.g., Todo, In Progress, Done)
- Simple dashboard view

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS |
| Database | PostgreSQL |
| ORM | Prisma |
| Runtime | Node.js 18+ |

## Repository Structure

```
testapp/
├── prisma/
│   ├── schema.prisma          # Prisma schema — tasks, users, team models
│   └── migrations/            # Auto-generated migration files
├── src/
│   ├── app/                   # Next.js App Router pages and layouts
│   │   ├── layout.tsx         # Root layout with Tailwind base styles
│   │   ├── page.tsx           # Landing / redirect to dashboard
│   │   ├── dashboard/
│   │   │   └── page.tsx       # Main dashboard view showing all tasks
│   │   ├── tasks/
│   │   │   ├── page.tsx       # Task list page
│   │   │   ├── new/
│   │   │   │   └── page.tsx   # Create new task form
│   │   │   └── [id]/
│   │   │       └── page.tsx   # Task detail / edit page
│   │   └── api/
│   │       └── tasks/
│   │           ├── route.ts   # GET all tasks, POST new task
│   │           └── [id]/
│   │               └── route.ts # GET, PATCH, DELETE single task
│   ├── components/
│   │   ├── ui/                # Low-level reusable UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Badge.tsx      # Used for status labels
│   │   │   └── Card.tsx
│   │   ├── tasks/
│   │   │   ├── TaskCard.tsx   # Single task summary card for dashboard
│   │   │   ├── TaskForm.tsx   # Shared form for create/edit
│   │   │   ├── TaskList.tsx   # Renders a list of TaskCards
│   │   │   └── StatusSelect.tsx # Dropdown for Todo/In Progress/Done
│   │   └── layout/
│   │       ├── Header.tsx
│   │       └── Sidebar.tsx
│   ├── lib/
│   │   ├── prisma.ts          # Prisma client singleton
│   │   ├── types.ts           # Shared TypeScript types and enums
│   │   └── utils.ts           # Helper functions (formatting, etc.)
│   └── hooks/
│       └── useTasks.ts        # Client-side data fetching hook
├── public/
├── .env.local                 # Local environment variables (never commit)
├── .env.example               # Example env file (committed)
├── tailwind.config.ts
├── tsconfig.json
├── next.config.ts
└── package.json
```

## Development Commands

```bash
# Install dependencies
npm install

# Set up the database (run after cloning or schema changes)
npx prisma migrate dev

# Generate Prisma client (after schema edits)
npx prisma generate

# Seed the database with sample tasks and users
npx prisma db seed

# Start development server (http://localhost:3000)
npm run dev

# Type-check without building
npm run typecheck

# Lint the codebase
npm run lint

# Format with Prettier
npm run format

# Build for production
npm run build

# Open Prisma Studio (visual DB browser)
npx prisma studio
```

## Database Schema (Prisma)

The core models are:

```prisma
model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  tasks     Task[]   @relation("AssignedTasks")
  createdAt DateTime @default(now())
}

model Task {
  id          String     @id @default(cuid())
  title       String
  description String?
  status      TaskStatus @default(TODO)
  assigneeId  String?
  assignee    User?      @relation("AssignedTasks", fields: [assigneeId], references: [id])
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  DONE
}
```

When modifying the schema, always run `npx prisma migrate dev --name <descriptive-name>` and commit the migration file.

## Coding Conventions

### TypeScript
- Use **strict mode** — no `any` types. Use `unknown` and narrow with guards if needed.
- Define all shared types in `src/lib/types.ts`. Import from there, do not redefine locally.
- Use `interface` for object shapes that represent domain models (Task, User). Use `type` for unions and utility types.
- Always type API route handler params and return values explicitly.

### React / Next.js
- Prefer **Server Components** by default. Only add `"use client"` when the component needs interactivity (forms, hooks, click handlers).
- Keep pages thin — move logic into components or server actions.
- Use Next.js **Route Handlers** (`src/app/api/`) for all data mutation endpoints. Keep them small; delegate DB logic to lib functions.
- Use `loading.tsx` and `error.tsx` files in route segments for proper UX states.

### Tailwind CSS
- Do not use inline `style` props. Use Tailwind utility classes only.
- Keep class strings readable — break into multiple lines using template literals or `clsx`/`cn` when conditionals are needed.
- Use the `cn()` utility (from `src/lib/utils.ts`, wrapping `clsx` + `tailwind-merge`) for all conditional class merging.
- Follow a consistent color scheme: use `slate` for neutral backgrounds/text, `blue` for primary actions, and status badge colors: `gray` = Todo, `yellow` = In Progress, `green` = Done.

### Prisma / Database
- Never import `PrismaClient` directly in components or route files. Always use the singleton from `src/lib/prisma.ts`.
- Wrap mutations in try/catch and return typed error responses from API routes.
- Keep queries as close to the data layer as possible — do not scatter raw Prisma calls throughout components.

### File & Component Naming
- React components: `PascalCase.tsx`
- Utility files: `camelCase.ts`
- Folders: `kebab-case` (except where Next.js requires it, e.g., `[id]`)
- One component per file. Co-locate tests next to the file they test (e.g., `TaskCard.test.tsx`).

### API Route Conventions
- Return consistent JSON shapes: `{ data: T }` on success, `{ error: string }` on failure.
- Use appropriate HTTP status codes: `200` (GET success), `201` (POST success), `400` (bad input), `404` (not found), `500` (server error).
- Validate request bodies before hitting the database. Use simple manual validation for MVP (no extra libraries needed).

## Key Constraints & Decisions

- **2-week launch deadline** — no scope creep. Stick strictly to MVP features: task creation, assignment, status tracking, dashboard.
- **Solo developer** — keep the architecture simple and linear. Avoid over-abstraction.
- **No paid services** — use a local PostgreSQL instance for development. For deployment, use free tiers (e.g., Railway or Supabase free tier for Postgres, Vercel for hosting).
- **No auth in MVP** — user identity is simplified (select a team member from a list). Do not add authentication unless explicitly requested.

## Common Patterns in This Codebase

### Fetching tasks in a Server Component
```typescript
// src/app/dashboard/page.tsx
import { prisma } from '@/lib/prisma';

export default async function DashboardPage() {
  const tasks = await prisma.task.findMany({
    include: { assignee: true },
    orderBy: { createdAt: 'desc' },
  });
  return <TaskList tasks={tasks} />;
}
```

### Updating task status via API
```typescript
// PATCH /api/tasks/[id]
export async function PATCH(req: Request, { params }: { params: { id: string } }) {
  const body = await req.json();
  const task = await prisma.task.update({
    where: { id: params.id },
    data: { status: body.status },
  });
  return Response.json({ data: task });
}
```

### Using the `cn` utility
```typescript
import { cn } from '@/lib/utils';

<div className={cn('rounded px-2 py-1 text-sm', status === 'DONE' && 'bg-green-100 text-green-800')}>
  {status}
</div>
```

## What Claude Should NOT Do

- Do not add authentication, user sessions, or JWT logic unless explicitly asked.
- Do not install new npm packages without flagging it — this project has a no-budget constraint.
- Do not change the Prisma schema without also providing the migration command.
- Do not use `any` type under any circumstances.
- Do not create new top-level folders outside the structure defined above without a clear reason.
- Do not add client-side state management libraries (Redux, Zustand, etc.) — this is overkill for MVP.