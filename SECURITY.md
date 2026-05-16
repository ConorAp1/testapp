# Security Policy for TestApp

TestApp is a task management application for small engineering teams at startups. It handles team member data, task assignments, and work status information. We take security seriously and appreciate responsible disclosure of any vulnerabilities.

## Supported Versions

As TestApp is in early-stage development targeting a 2-week MVP launch, only the latest version on the `main` branch is actively maintained and receives security updates.

| Version | Supported |
|---------|-----------|
| Latest (`main`) | ✅ Yes |
| Older commits / forks | ❌ No |

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub Issues.**

If you discover a security vulnerability in TestApp, please report it responsibly by emailing:

**security@testapp.example.com** *(replace with your actual contact)*

### What to Include in Your Report

To help us triage and resolve the issue quickly, please include:

1. **Description** of the vulnerability and its potential impact
2. **Steps to reproduce** the issue clearly and concisely
3. **Which feature area is affected:**
   - Task creation
   - Task assignment
   - Status tracking
   - Dashboard view
   - Authentication / session management
   - API routes (`/api/*`)
   - Database layer (Prisma/PostgreSQL)
4. **Proof of concept** (code snippet, screenshot, or recording) if available
5. **Suggested fix** if you have one in mind

### Response Timeline

- **Acknowledgement:** Within 48 hours of receiving your report
- **Initial assessment:** Within 5 business days
- **Resolution or mitigation plan:** Communicated within 10 business days

We are a solo developer project with no security team, so timelines may vary. We will keep you updated throughout the process and credit you in the release notes if you wish.

## Security Considerations Specific to TestApp

### Authentication & Authorization

- TestApp uses session-based authentication. Ensure you are not able to access another team member's tasks or dashboard data without proper authorization.
- All API routes under `/api/` must validate that the requesting user belongs to the relevant team before returning or mutating data.

### Database (PostgreSQL + Prisma)

- Prisma parameterizes all queries by default, protecting against SQL injection. Raw query usage (`prisma.$queryRaw`) should be avoided. If used, it must use tagged template literals for safe parameterization.
- The `DATABASE_URL` in `.env.local` must never be committed to the repository. The `.env.example` file contains only placeholder values.
- Database credentials must never be exposed in client-side code or API responses.

### Environment Variables

- All secrets (database credentials, auth secrets, API keys) are managed via environment variables.
- Never commit `.env.local` or any file containing real credentials.
- Refer to `.env.example` for the list of required variables. All values in that file are non-sensitive placeholders.

### Next.js API Routes

- Server-side API routes handle all sensitive operations. No database credentials or internal logic should be exposed to the client.
- Validate and sanitize all inputs received by API routes, especially for task creation and assignment endpoints.

### Data Exposure

- The dashboard view must only display tasks and team members belonging to the authenticated user's team. Cross-team data leakage is a critical vulnerability in this context.
- Avoid logging sensitive data (user emails, task contents, session tokens) to the console in production.

### Dependencies

- Dependencies are managed via `npm`. Keep dependencies up to date and audit regularly:
  ```bash
  npm audit
  ```
- Critical or high-severity vulnerabilities in dependencies should be patched promptly.

## Out of Scope

The following are currently outside the scope of TestApp's security review, given the MVP constraints:

- Attacks requiring physical access to a device
- Social engineering attacks
- Denial-of-service attacks on the development/preview server
- Issues in third-party services not under our control

## Disclosure Policy

We follow a **coordinated disclosure** approach:

1. Reporter submits vulnerability privately.
2. We investigate and develop a fix.
3. Fix is deployed to production.
4. We publicly acknowledge the vulnerability (with reporter's permission) in the release notes or changelog.

We ask that reporters give us a reasonable amount of time to address the issue before any public disclosure.

## Thank You

We genuinely appreciate the time and effort security researchers and contributors put into making TestApp safer for the small engineering teams who rely on it. Responsible disclosure helps protect real users and we are grateful for your help.