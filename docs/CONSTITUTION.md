# Nexus Core Engineering Constitution
**Version:** 1.0.0

> **Assumption Note:** Because the project name, brief, maturity, shape, and languages were unspecified, this constitution assumes a production-grade, full-stack TypeScript web application named "Nexus Core" (a multi-tenant SaaS platform). All delegated technical decisions have been derived to support this idiomatic TypeScript/React/Node.js stack.

This document is the supreme engineering authority for the Nexus Core project. All code, architecture, tooling, and AI-assisted development must strictly adhere to these rules. No pull request may be merged if it violates this constitution.

## Mission
To provide a highly reliable, scalable, and secure multi-tenant SaaS platform for managing enterprise engineering workflows and resource allocation, prioritizing data integrity and system availability.

## Core Values
1. **Testing over trust**: We do not assume code works; we prove it works through automated verification.
2. **Security by design**: Security is a foundational requirement, not an afterthought.
3. **Predictability over cleverness**: Code must be readable, maintainable, and explicit.

## Technology Stack

### Required Technologies
| Layer | Technology |
| :--- | :--- |
| **Language** | TypeScript (Strict Mode) |
| **Frontend** | React, Tailwind CSS |
| **Backend** | Node.js, Express |
| **Database** | PostgreSQL, Prisma ORM |
| **Infrastructure** | Docker, AWS |

### Forbidden Technologies / Practices
* Plain JavaScript in application code
* Unmaintained dependencies
* Experimental libraries in production without approval
* Bypassing the ORM for raw SQL (unless explicitly approved for performance via ADR)

## Repository Structure
The project utilizes a monorepo structure managed by Turborepo.

```text
nexus-core/
├── apps/
│   ├── web/               # React frontend application
│   └── api/               # Node.js/Express backend API
├── packages/
│   ├── ui/                # Shared React component library
│   ├── database/          # Prisma schema and generated client
│   ├── config/            # Shared ESLint, TypeScript, and Prettier configs
│   └── types/             # Shared TypeScript interfaces and types
└── package.json
```

## Language/Code Standards
* **Typing:** TypeScript `strict` mode is mandatory. `any` is strictly forbidden; use `unknown` if the type is truly dynamic and narrow it via type guards.
* **Naming Conventions:**
  * Variables, functions, methods: `camelCase`
  * Classes, React Components, Interfaces, Types: `PascalCase`
  * Files and directories: `kebab-case`
  * Constants and environment variables: `UPPER_SNAKE_CASE`
* **Component/File Size Limits:** 300 lines target, 500 mandatory refactor.

## Frontend Standards
* **Architecture:** Functional components exclusively. Use React Hooks for state and lifecycle management.
* **State Management:** Keep state as close to where it is used as possible. Prop drilling is limited to a maximum of 3 levels.
* **Data Fetching:** Use React Query (TanStack Query) for server state; do not use `useEffect` for data fetching.

## Backend/API & Validation Standards
* **Architecture:** RESTful API design. Controllers must be thin, delegating business logic to service classes.
* **Validation:** All incoming requests (body, query, params) must be strictly validated at the boundary using Zod.
* **Database Access:** All database interactions must go through the Prisma ORM.

## Error Handling
Errors must be categorized and mapped to appropriate HTTP status codes. Raw errors must never be leaked to the client.

| Category | HTTP Status | Description |
| :--- | :--- | :--- |
| `VALIDATION_ERROR` | 400 | Malformed request or invalid data |
| `AUTHENTICATION_ERROR` | 401 | Missing or invalid credentials |
| `AUTHORIZATION_ERROR` | 403 | Insufficient permissions |
| `BUSINESS_ERROR` | 422 | Unmet business rule or state conflict |
| `EXTERNAL_SERVICE_ERROR` | 502 | Upstream dependency failure |
| `INFRASTRUCTURE_ERROR` | 503 | Database or cache unavailability |
| `UNKNOWN_ERROR` | 500 | Unhandled exceptions |

## Logging
All logs must be structured JSON. `console.log` is forbidden in production code.

**Required structured log fields:**
* `event`: String identifier for the action (e.g., `user_login_attempt`)
* `timestamp`: ISO 8601 UTC string
* `requestId`: UUID for distributed tracing
* `userId?`: UUID of the authenticated user (if applicable)
* `metadata?`: Additional contextual JSON data

## Security
* **Authentication & Authorization Location:** Must occur server-side. Client-side checks are for UX only, never for security.
* **Secrets Handling:**
  * From environment variables.
  * From a secret manager (e.g., AWS Secrets Manager) in production.
  * Never commit secrets.
* **Data Protection:** PII must be encrypted at rest.

## Accessibility
* **Standard:** All frontend interfaces must comply with WCAG 2.1 AA standards.
* **Implementation:** Semantic HTML is required. ARIA attributes must be used only when semantic HTML is insufficient. Keyboard navigation must be fully supported.

## Performance
* **Frontend:** Lighthouse performance score must remain > 90.
* **Backend:** API response time must be < 200ms (p95).
* **Database:** Queries returning lists must implement pagination.

## Testing
* **Minimum Coverage:** 80% minimum overall, 95% for critical business logic.
* **Required Test Types:**
  * **Unit Tests:** (Jest/Vitest) For pure functions, utilities, and isolated components.
  * **Integration Tests:** For API endpoints and database interactions.
  * **E2E Tests:** (Playwright) For critical user journeys.

## CI/CD
All code must pass through automated Continuous Integration. Direct pushes to `main` are forbidden.

**CI gates per PR:**
1. Lint (ESLint/Prettier)
2. Typecheck (`tsc --noEmit`)
3. Unit tests
4. Integration tests

## Documentation
* **README:** Every app and package must have a README explaining its purpose and setup.
* **Architecture:** Significant architectural changes must be documented via Architecture Decision Records (ADRs).
* **API:** Backend endpoints must be documented using OpenAPI/Swagger specifications.

## Observability
* **Tracing:** OpenTelemetry must be implemented across the stack to propagate `requestId`.
* **Monitoring:** Datadog (or equivalent) is used for APM, log aggregation, and infrastructure metrics.
* **Alerting:** Alerts must be configured for `UNKNOWN_ERROR` spikes and p95 latency breaches.

## AI Development Rules
* **AI-generated code policy:** AI code is UNTRUSTED — must be reviewed, tested, validated before merge.
* **Agent restrictions (each without human approval):**
  * May NOT deploy to production
  * May NOT rotate credentials
  * May NOT modify infrastructure
  * May NOT approve pull requests

## Prompt/MCP/RAG Standards
* **Prompts:** Prompts must be version-controlled, documented, and tested. Prompt changes require peer review.
* **MCP (Model Context Protocol):** Integrations must be least-privilege, auditable, and revocable.
* **RAG (Retrieval-Augmented Generation):** Sources must be trusted, versioned, and source-attributed in the output.

## Code Review Standards
Every Pull Request description must explicitly answer the following questions:
1. What changed?
2. Why?
3. Risks?
4. Rollback plan?
5. Testing evidence?

## Git Standards
* **Branch conventions:** `feature/*`, `bugfix/*`, `hotfix/*`, `chore/*`
* **Commit conventions:** Conventional Commits required (`feat`, `fix`, `refactor`, `test`, `docs`, `perf`, `chore`).

## Dependency Rules
* Must pass security scan (e.g., `npm audit` / Snyk).
* Must pass license review (no copyleft licenses like GPL in proprietary code).
* Must be actively maintained (commits within the last 6 months).
* Prefer building over adding a dependency when the implementation is smaller than 50 lines of code.

## Definition of Done
A feature or fix is only "Done" when all of the following are true:
* [ ] Requirements implemented
* [ ] Tests written
* [ ] Tests passing
* [ ] Typecheck passing
* [ ] Lint passing
* [ ] Security review completed
* [ ] Documentation updated
* [ ] Accessibility validated
* [ ] Performance validated
* [ ] Code reviewed

## Non-Negotiable Rules (NEVER / ALWAYS)
* **NEVER** commit secrets, API keys, or credentials to version control.
* **NEVER** bypass CI checks or force-merge a Pull Request.
* **ALWAYS** validate and sanitize all external input on the backend.
* **ALWAYS** write a regression test when fixing a bug.

## Amendment Process
This constitution is a living document. Changes require:
1. Written proposal (Pull Request to this document).
2. Architecture review.
3. Team approval (majority consensus).
4. Version increment.