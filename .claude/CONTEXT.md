# TestApp — Claude Context File

## What This Project Is
TestApp is a lightweight task management app built for small engineering teams (3–10 people) at startups. The core problem it solves: teams waste time in daily standups and status meetings because there is no single place to see who is working on what. TestApp provides that shared visibility with minimal overhead.

This is a solo-developer, zero-budget, 2-week MVP. Every decision should optimize for **shipping fast** over architectural perfection.

---

## Repository Structure
```
testapp/
├── .claude/
│   ├── MEMORY.md          # Persistent project memory for Claude
│   └── CONTEXT.md         # This file — working context and conventions
├── app/
│   ├── layout.tsx         # Root layout with Tailwind base styles
│   ├── page.tsx           # Landing / login redirect
│   ├── dashboard/
│   │   └── page.tsx       # Main dashboard — lists all tasks
│   ├── tasks/
│   │   ├── new/
│   │   │   └── page.tsx   # Task creation form
│   │   └── [id]/
│   │       └── page.tsx   # Task detail / edit view
│   └── api/
│       ├── auth/
│       │   └── [...nextauth]/route.ts
│       ├── tasks/
│       │   ├── route.ts           # GET all, POST create
│       │   └── [id]/route.ts      # GET one, PATCH update, DELETE
│       └── users/
│           └── route.ts           # GET team members (for assignment dropdown)
├── components/
│   ├── TaskCard.tsx        # Single task card used in dashboard
│   ├── TaskForm.tsx        # Shared create/edit form
│   ├── StatusBadge.tsx     # Colored badge: TODO / IN_PROGRESS / DONE
│   ├── AssigneeSelect.tsx  # Dropdown to assign task to team member
│   └── DashboardFilters.tsx # Filter bar: by status, by assignee
├── lib/
│   ├── prisma.ts           # Prisma client singleton
│   └── auth.ts             # NextAuth config and helpers
├── prisma/
│   ├── schema.prisma       # Database schema
│   └── migrations/        # Auto-generated migration files
├── types/
│   └── index.ts            # Shared TypeScript types (Task, User, Status)
├── docs/
│   └── plan.md             # Sprint plan and task breakdown
├── .env.local              # Local secrets (never committed)
├── .env.example            # Safe template for env vars
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## Prisma Schema (source of truth for data model)
```prisma
model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  image     String?
  tasks     Task[]   @relation("AssignedTasks")
  createdAt DateTime @default(now())
}

model Task {
  id          String   @id @default(cuid())
  title       String
  description String?
  status      Status   @default(TODO)
  priority    Priority @default(MEDIUM)
  dueDate     DateTime?
  assignee    User?    @relation("AssignedTasks", fields: [assigneeId], references: [id])
  assigneeId  String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

enum Status {
  TODO
  IN_PROGRESS
  DONE
}

enum Priority {
  LOW
  MEDIUM
  HIGH
}
```

---

## API Contract

### Tasks
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/tasks` | List all tasks. Query params: `?status=`, `?assigneeId=` |
| POST | `/api/tasks` | Create a new task |
| GET | `/api/tasks/[id]` | Get single task |
| PATCH | `/api/tasks/[id]` | Update task (status, assignee, title, etc.) |
| DELETE | `/api/tasks/[id]` | Delete task |

### Users
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/users` | List all users (for assignee dropdown) |

### Response Shape (always)
```ts
// Success
{ data: T, error: null }

// Error
{ data: null, error: string }
```

---

## Coding Conventions

### TypeScript
- Strict mode ON. No `any`. If a type is unknown, use `unknown` and narrow it.
- Define shared types in `types/index.ts` and import from there.
- Use `type` for object shapes, `interface` for anything that might be extended.

### React / Next.js
- Prefer **Server Components** by default. Only add `"use client"` when you need interactivity (forms, click handlers, useState).
- Data fetching in Server Components happens directly via Prisma (not fetch).
- Data fetching in Client Components uses `fetch('/api/...')`.
- Use Next.js `loading.tsx` files for suspense boundaries on slow pages.

### API Routes
- All API route handlers are in `app/api/`.
- Always wrap handler body in try/catch and return `{ data: null, error: message }` on failure.
- Validate request body before touching the database.
- Use `NextResponse.json()` for all responses.

### Styling
- Tailwind classes only. No inline `style={{}}`. No CSS modules.
- Use consistent spacing scale: `p-4`, `gap-4`, `space-y-4` (multiples of 4).
- Status colors: TODO = gray, IN_PROGRESS = blue, DONE = green.
- Priority colors: LOW = slate, MEDIUM = yellow, HIGH = red.

### File Naming
- Components: `PascalCase.tsx`
- Pages: `page.tsx` (Next.js convention)
- Utilities/lib: `camelCase.ts`
- All files use `.tsx` if they return JSX, `.ts` otherwise.

---

## Common Commands
```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Prisma: push schema changes to local DB (during development)
npx prisma db push

# Prisma: create a migration (for production-tracked changes)
npx prisma migrate dev --name <migration-name>

# Prisma: open Prisma Studio (DB GUI)
npx prisma studio

# Prisma: regenerate client after schema change
npx prisma generate

# Type check
npx tsc --noEmit

# Build for production
npm run build

# Lint
npm run lint
```

---

## Environment Variables
```
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=...
NEXTAUTH_URL=http://localhost:3000
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...
```

---

## Out of Scope for MVP (do not implement)
- Comments on tasks
- File attachments
- Email notifications
- Multiple workspaces / organizations
- Kanban drag-and-drop (dashboard is a list view for MVP)
- Mobile app
- Billing or subscription logic
- Role-based permissions (all team members can do everything in MVP)