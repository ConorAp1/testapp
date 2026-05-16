# TestApp

A simple task management app for small engineering teams to track their work without the overhead of complex project management tools.

## What It Does

TestApp gives small engineering teams (3–10 people) a shared, real-time view of who is working on what and where each task stands. No more status meetings just to find out what everyone is doing.

## The Problem It Solves

Small startup engineering teams waste significant time in daily standups and status check-ins because there's no single place to see task ownership and progress. TestApp provides that shared visibility so teams can stay aligned asynchronously.

## Features (MVP)

- **Task Creation** – Quickly create tasks with a title, description, and due date
- **Task Assignment** – Assign tasks to any team member
- **Status Tracking** – Move tasks through statuses: `To Do`, `In Progress`, `Blocked`, `Done`
- **Dashboard View** – A simple overview showing all tasks, their owners, and current statuses at a glance

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js (App Router), TypeScript, Tailwind CSS |
| Backend | Next.js API Routes / Server Actions |
| Database | PostgreSQL |
| ORM | Prisma |

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL running locally (or a free tier on Neon/Supabase)
- npm or pnpm

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/testapp.git
   cd testapp
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp .env.example .env
   ```
   Fill in your `DATABASE_URL` and any other required values in `.env`.

4. **Set up the database**
   ```bash
   npx prisma migrate dev --name init
   npx prisma generate
   ```

5. **Seed the database (optional)**
   ```bash
   npx prisma db seed
   ```

6. **Run the development server**
   ```bash
   npm run dev
   ```

   Open [http://localhost:3000](http://localhost:3000) in your browser.

## Project Structure

```
testapp/
├── app/                   # Next.js App Router pages and layouts
│   ├── dashboard/         # Dashboard view
│   ├── tasks/             # Task list and detail pages
│   └── api/               # API route handlers
├── components/            # Reusable React components
│   ├── ui/                # Base UI components (buttons, inputs, etc.)
│   └── tasks/             # Task-specific components
├── lib/                   # Utility functions, Prisma client, helpers
├── prisma/
│   ├── schema.prisma      # Database schema
│   └── migrations/        # Migration history
├── public/                # Static assets
├── styles/                # Global styles
├── types/                 # Shared TypeScript types
└── .env.example           # Example environment variables
```

## Database Schema Overview

- **User** – Team members who can create and be assigned tasks
- **Task** – Core entity with title, description, status, assignee, and due date
- **Status** – Enum: `TODO`, `IN_PROGRESS`, `BLOCKED`, `DONE`

## Available Scripts

```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript compiler check
npx prisma studio    # Open Prisma Studio to inspect the database
npx prisma migrate dev   # Run database migrations in development
```

## Target Users

Small engineering teams of 3–10 people at early-stage startups who need lightweight task visibility without the complexity of tools like Jira.

## Constraints & Decisions

- **2-week launch target** – The MVP is intentionally minimal. No auth complexity, no integrations, no notifications in v1.
- **Solo developer** – Code is kept simple and straightforward. No premature abstraction.
- **No paid services** – Uses free-tier PostgreSQL (e.g., Neon) and is deployable to Vercel's free tier.

## Deployment

TestApp is designed to deploy on **Vercel** with a **Neon** or **Supabase** PostgreSQL database — both free at the scale of a small team.

```bash
npm run build
# Deploy via Vercel CLI or connect your GitHub repo to Vercel
```

Make sure to set `DATABASE_URL` in your Vercel environment variables.

## Contributing

This is a solo MVP project. If you'd like to suggest improvements, open an issue.

## License

MIT