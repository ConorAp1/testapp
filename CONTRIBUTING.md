# Contributing to TestApp

Thank you for your interest in contributing to TestApp — a task management tool built for small engineering teams at startups. This document outlines how to contribute effectively to the project.

## Project Overview

TestApp helps small engineering teams (3–10 people) eliminate unnecessary status meetings by providing shared visibility into who is working on what. The MVP includes task creation, task assignment, status tracking, and a simple dashboard view.

## Tech Stack

- **Framework:** Next.js (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Database:** PostgreSQL
- **ORM:** Prisma

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL (local instance or Docker)
- npm or yarn
- Git

### Local Setup

1. **Fork and clone the repository:**
   ```bash
   git clone https://github.com/your-username/testapp.git
   cd testapp
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Set up environment variables:**
   ```bash
   cp .env.example .env.local
   ```
   Fill in your local values (see `.env.example` for all required variables).

4. **Set up the database:**
   ```bash
   npx prisma migrate dev
   npx prisma db seed
   ```

5. **Run the development server:**
   ```bash
   npm run dev
   ```

   The app will be available at `http://localhost:3000`.

## How to Contribute

### Reporting Bugs

Before opening a bug report, please:
- Search existing issues to avoid duplicates.
- Reproduce the bug on the latest `main` branch.

When opening a bug report, include:
- Steps to reproduce
- Expected vs. actual behavior
- Browser and OS if relevant to the UI
- Any relevant console errors or logs
- Which MVP feature area is affected (task creation, assignment, status tracking, or dashboard)

### Suggesting Features

TestApp is intentionally scoped for small engineering teams at startups. Before suggesting a feature, ask:
- Does this reduce time wasted in status meetings?
- Is this useful for teams of 3–10 people?
- Does it fit within the MVP scope or a clearly defined next phase?

Open a GitHub Issue with the label `enhancement` and describe the use case clearly.

### Submitting Code

#### Branch Naming

Use descriptive branch names:
```
feat/task-assignment-ui
fix/dashboard-status-filter
chore/prisma-schema-update
docs/contributing-guide
```

#### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
feat: add task assignment dropdown to task creation form
fix: resolve dashboard not reflecting updated task statuses
chore: update Prisma schema to add Task priority field
docs: add setup instructions to CONTRIBUTING.md
```

#### Pull Request Process

1. Create your branch from `main`.
2. Make your changes with clear, focused commits.
3. Ensure the app builds without errors:
   ```bash
   npm run build
   ```
4. Run type checks:
   ```bash
   npm run type-check
   ```
5. Run linting:
   ```bash
   npm run lint
   ```
6. Test your changes manually, especially across the four MVP feature areas.
7. Open a pull request against `main` with:
   - A clear description of what was changed and why
   - Screenshots or recordings for any UI changes (dashboard, task views)
   - A note on any database migrations included

#### PR Review Criteria

PRs will be reviewed for:
- TypeScript type safety (avoid `any`)
- Correct Prisma schema usage and migration hygiene
- Tailwind CSS conventions (utility-first, no custom CSS unless necessary)
- Next.js App Router patterns (server components vs. client components used appropriately)
- Clarity and maintainability of code

## Code Style Guidelines

- **TypeScript:** Strict mode is enabled. All props, return types, and API responses must be explicitly typed.
- **Tailwind:** Use Tailwind utility classes directly. Avoid inline styles. Group related classes logically.
- **Prisma:** All database changes must go through migrations (`npx prisma migrate dev`). Do not manually edit the database schema outside of Prisma.
- **Next.js:** Prefer server components for data fetching. Use client components only when interactivity is required.
- **File naming:** Use `kebab-case` for files and folders. Component files use `PascalCase` only for the component itself inside the file.

## Project Structure (Reference)

```
testapp/
├── app/                  # Next.js App Router pages and layouts
│   ├── dashboard/        # Dashboard view for task overview
│   ├── tasks/            # Task creation and detail pages
│   └── api/              # API route handlers
├── components/           # Reusable UI components
├── lib/                  # Utility functions and shared logic
├── prisma/               # Prisma schema and migrations
│   ├── schema.prisma
│   └── migrations/
├── public/               # Static assets
├── styles/               # Global styles (minimal — Tailwind handles most)
├── types/                # Shared TypeScript type definitions
└── .env.example          # Environment variable template
```

## What We're Not Accepting Right Now

Given the 2-week launch timeline and solo developer constraint, the following are out of scope for initial contributions:
- Large architectural refactors
- New third-party paid service integrations
- Features outside the MVP (e.g., time tracking, notifications, integrations with Slack/Jira)

If you have ideas for post-MVP features, open an issue tagged `post-mvp` to start the conversation.

## Questions?

Open a GitHub Issue with the label `question` and we'll get back to you as soon as possible.