# DEVELOPER.md

# PaperScout.ai Developer Guide

This guide explains how to build, run, test, and extend the PaperScout.ai
codebase. It is written for human developers and AI coding agents implementing
PaperScout.ai from the original planning README.

PaperScout.ai is a web application that sends users daily or weekly updates
with a chosen number of recently published articles in their research field,
provides AI-generated summaries for selected articles, and teaches young
researchers how literature search strategies work.

The product source of truth is `README.md` or the PaperScout planning document.
The AI-agent workflow source of truth is `AGENTS.md`. Keep this file aligned
with both whenever architecture, commands, environment variables, tests, or
project structure change.

---

## Architecture

PaperScout.ai should use a layered web architecture with explicit side-effect
boundaries.

Core product responsibilities:

1. User registration, authentication, profile management, and secure sessions.
2. Search-query creation, editing, deletion, and history display.
3. Literature retrieval from PubMed and arXiv.
4. Article selection based on search terms, preferences, and user feedback.
5. AI-assisted search-query optimization and article summarization.
6. Scheduled update delivery by email, SMS, and/or in-app notification.
7. Search-strategy transparency for each update.
8. Feedback collection and later use in retrieval/summarization improvement.
9. Admin review tools for users, feedback, search logs, and schedule status.

Recommended implementation layers:

```text
React UI
  -> typed frontend API client
  -> FastAPI route handlers
  -> application services
  -> pure domain logic
  -> ports / protocols
  -> adapters / clients for databases, PubMed, arXiv, AI, email, SMS, scheduler
```

Domain logic should be pure where practical. Network calls, database writes,
file writes, clock reads, random generation, email/SMS delivery, and model calls
must be isolated behind clearly named boundary functions or adapter classes.

Do not place retrieval, summarization, scheduler, or database behavior directly
inside React components. React components should render validated API data,
collect user input, display loading/error states, and call the frontend API
client.

---

## Dependency direction

Use these directions unless an approved architecture change replaces them.

Allowed:

```text
frontend pages -> frontend components -> frontend API client
FastAPI routes -> application services -> domain + ports + schemas
application services -> adapters + domain + schemas
adapters -> ports + schemas + third-party clients
clients -> schemas
core/domain -> schemas + ports only
ports -> schemas only
schemas -> no internal dependencies
```

Forbidden:

```text
core/domain -> FastAPI
core/domain -> React
core/domain -> SQLAlchemy, psycopg, sqlite3, or ORM-specific code
core/domain -> requests/httpx/aiohttp
core/domain -> OpenAI/Postmark/Plivo SDKs
frontend -> database
frontend -> third-party literature APIs directly
schemas -> application services, adapters, clients, or frameworks
adapters -> React components
```

The main design goal is to keep product decisions testable without requiring a
network, database, browser, scheduler, or AI provider.

---

## Source-of-truth documents

Read these before changing production behavior:

1. `README.md` or the PaperScout planning Markdown document.
2. `AGENTS.md`.
3. `CODING_STANDARDS.md`, if present.
4. `LICENSE`.
5. `docs/architecture/*`, if present.
6. `docs/api/*`, if present.
7. `docs/user-guides/*`, if present.

When documentation conflicts, use this order:

1. Current explicit user-approved ticket plan.
2. `AGENTS.md`.
3. `CODING_STANDARDS.md`.
4. PaperScout planning README.
5. `DEVELOPER.md`.
6. Existing code patterns.

Never remove or weaken the AGPL license notice.

---

## Repository structure

The original planning README defines a unit-based repository layout. Preserve
that mapping so each feature can be traced back to a planned unit.

```text
project-root/
├── source/
│   ├── unit-01-landing-page/
│   ├── unit-02-about-app-page/
│   ├── unit-03-about-developer-page/
│   ├── unit-04-sign-up-page/
│   ├── unit-05-login-page/
│   ├── unit-06-user-dashboard/
│   ├── unit-07-user-profile/
│   ├── unit-08-search-history-page/
│   ├── unit-09-new-query-page/
│   ├── unit-10-previous-search-page/
│   ├── unit-11-user-database/
│   ├── unit-12-user-feedback-database/
│   ├── unit-13-search-history-database/
│   ├── unit-14-scheduler-database/
│   ├── unit-15-dashboard-interactivity/
│   ├── unit-16-dashboard-display/
│   ├── unit-17-user-login/
│   ├── unit-18-user-sign-up/
│   ├── unit-19-new-query/
│   ├── unit-20-user-profile-backend/
│   ├── unit-21-search-history/
│   ├── unit-22-user-feedback/
│   ├── unit-23-update-notifications/
│   ├── unit-24-pubmed-api/
│   ├── unit-25-arxiv-api/
│   ├── unit-26-article-selection-summarization/
│   ├── unit-27-search-query-optimization/
│   ├── unit-28-update-formatting/
│   ├── unit-29-admin-capabilities/
│   ├── unit-30-access-scheduler-db/
│   ├── unit-31-access-user-db/
│   ├── unit-32-access-search-history-db/
│   ├── unit-33-access-feedback-db/
│   ├── unit-34-update-scheduler-db/
│   ├── unit-35-update-user-db/
│   ├── unit-36-update-search-history-db/
│   ├── unit-37-update-feedback-db/
│   ├── unit-38-encrypt-password/
│   ├── unit-39-create-scheduler-item/
│   ├── unit-40-create-user/
│   ├── unit-41-create-search-history-item/
│   └── unit-42-create-feedback-item/
├── test/
│   ├── unit_test/
│   └── integration_test/
├── docs/
│   ├── architecture/
│   ├── api/
│   └── user-guides/
├── scripts/
│   ├── build/
│   ├── deploy/
│   └── test/
├── config/
│   ├── development/
│   ├── staging/
│   └── production/
├── docker-compose.yml
├── Dockerfile.api
├── Dockerfile.web
├── Makefile
├── README.md
├── DEVELOPER.md
├── AGENTS.md
├── LICENSE
└── package.json
```

The README tree omitted units 11-20 even though the design section defines
those units. Create and test directories for all 42 units so the source tree and
design section remain consistent.

Conventional application folders may be added inside `source/` when useful, but
the planned unit mapping must remain obvious. For example:

```text
source/backend/paperscout_api/
source/backend/paperscout_core/
source/backend/paperscout_ports/
source/backend/paperscout_adapters/
source/frontend/paperscout_web/
```

If conventional folders are added, each module should document which planned
unit or units it implements.

---

## File and directory responsibilities

| Path | Purpose |
| --- | --- |
| `source/unit-01-landing-page/` | Landing page UI, routing, and tests. |
| `source/unit-02-about-app-page/` | Static educational page explaining PaperScout.ai. |
| `source/unit-03-about-developer-page/` | Developer bio/contact page and tests. |
| `source/unit-04-sign-up-page/` | React sign-up form, validation, submit behavior. |
| `source/unit-05-login-page/` | React login form, errors, redirect behavior. |
| `source/unit-06-user-dashboard/` | Dashboard UI for updates, profile summary, history, navigation. |
| `source/unit-07-user-profile/` | Profile/preferences UI and edit workflow. |
| `source/unit-08-search-history-page/` | Search-history list and previous-result navigation. |
| `source/unit-09-new-query-page/` | Query-entry UI, AI suggestions, frequency/delivery preferences. |
| `source/unit-10-previous-search-page/` | Previous query metadata and summary-card display. |
| `source/unit-11-user-database/` | User table/model definitions and persistence boundaries. |
| `source/unit-12-user-feedback-database/` | Feedback table/model definitions and constraints. |
| `source/unit-13-search-history-database/` | Search-history persistence schema and access contracts. |
| `source/unit-14-scheduler-database/` | Scheduler persistence schema and next-run state. |
| `source/unit-15-dashboard-interactivity/` | Backend handlers for dashboard actions. |
| `source/unit-16-dashboard-display/` | Backend dashboard-data aggregation service. |
| `source/unit-17-user-login/` | Authentication service and token/session workflow. |
| `source/unit-18-user-sign-up/` | Registration service, uniqueness checks, password hashing. |
| `source/unit-19-new-query/` | Query submission orchestration and initial retrieval path. |
| `source/unit-20-user-profile-backend/` | Profile retrieval/update API and validation. |
| `source/unit-21-search-history/` | Search-history API and display-ready response assembly. |
| `source/unit-22-user-feedback/` | Feedback API and feedback-processing logic. |
| `source/unit-23-update-notifications/` | Notification orchestration, status logging, delivery adapters. |
| `source/unit-24-pubmed-api/` | PubMed query construction, request boundary, response parsing. |
| `source/unit-25-arxiv-api/` | arXiv query construction, XML parsing, error handling. |
| `source/unit-26-article-selection-summarization/` | Relevance selection and AI summary generation/validation. |
| `source/unit-27-search-query-optimization/` | Search-term analysis, suggestion, and strategy explanation logic. |
| `source/unit-28-update-formatting/` | HTML/text/in-app update formatting and escaping. |
| `source/unit-29-admin-capabilities/` | Admin-only user, feedback, schedule, and log actions. |
| `source/unit-30-access-scheduler-db/` | Scheduler read queries. |
| `source/unit-31-access-user-db/` | User/profile read queries with sensitive-field masking. |
| `source/unit-32-access-search-history-db/` | Search-history read queries and ordering. |
| `source/unit-33-access-feedback-db/` | Feedback read queries for review and optimization. |
| `source/unit-34-update-scheduler-db/` | Scheduler update operations and concurrency rules. |
| `source/unit-35-update-user-db/` | User/profile update operations. |
| `source/unit-36-update-search-history-db/` | Search-history metadata update operations. |
| `source/unit-37-update-feedback-db/` | Feedback status/admin-note update operations. |
| `source/unit-38-encrypt-password/` | Password hashing and verification. The implementation should hash, not reversibly encrypt, passwords. |
| `source/unit-39-create-scheduler-item/` | Scheduler creation/replacement logic for new or edited searches. |
| `source/unit-40-create-user/` | User creation persistence boundary. |
| `source/unit-41-create-search-history-item/` | Search-history creation persistence boundary. |
| `source/unit-42-create-feedback-item/` | Feedback creation persistence boundary. |
| `test/unit_test/` | Unit tests aligned to every planned source unit. |
| `test/integration_test/` | Full workflow and cross-boundary tests. |
| `docs/architecture/` | System diagrams, data-flow notes, security architecture, scheduler architecture. |
| `docs/api/` | FastAPI route contracts, request/response schemas, auth behavior. |
| `docs/user-guides/` | End-user instructions for young researchers. |
| `scripts/build/` | Build scripts and Docker helpers. |
| `scripts/deploy/` | Deployment scripts for staging/production. |
| `scripts/test/` | Test runners and CI helper scripts. |
| `config/development/` | Local configuration examples. |
| `config/staging/` | Staging configuration examples. |
| `config/production/` | Production configuration templates. |

---

## Runtime services

The target local development stack should include these services:

1. `paperscout-web`: React frontend.
2. `paperscout-api`: FastAPI backend.
3. `postgres`: PostgreSQL database for integration and production parity.
4. `scheduler`: scheduled-job runner for due literature updates.
5. Optional `mailhog` or equivalent local email sink for development tests.

SQLite may be used for lightweight development, but PostgreSQL is the
production database target and should be used for integration tests.

---

## Runtime defaults

Use these environment variable names unless the implementation plan approves a
change.

```bash
PAPERSCOUT_ENV=development
PAPERSCOUT_API_HOST=0.0.0.0
PAPERSCOUT_API_PORT=8080
PAPERSCOUT_WEB_HOST=0.0.0.0
PAPERSCOUT_WEB_PORT=3000
PAPERSCOUT_API_BASE_URL=http://localhost:8080

DATABASE_URL=postgresql://paperscout:paperscout@postgres:5432/paperscout
TEST_DATABASE_URL=postgresql://paperscout:paperscout@postgres:5432/paperscout_test
SQLITE_DATABASE_URL=sqlite:///./data/paperscout_dev.sqlite3

JWT_SECRET_KEY=replace-me-in-local-env
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
PASSWORD_HASH_SCHEME=argon2id

PUBMED_BASE_URL=https://eutils.ncbi.nlm.nih.gov/entrez/eutils
PUBMED_TOOL=PaperScout.ai
PUBMED_EMAIL=developer@example.com
NCBI_API_KEY=

ARXIV_BASE_URL=https://export.arxiv.org/api/query

AI_PROVIDER=openai
AI_MODEL=gpt-4
AI_API_KEY=
AI_SUMMARY_MAX_WORDS=200
AI_QUERY_OPTIMIZATION_ENABLED=true

POSTMARK_SERVER_TOKEN=
POSTMARK_FROM_EMAIL=
PLIVO_AUTH_ID=
PLIVO_AUTH_TOKEN=
PLIVO_FROM_NUMBER=

SCHEDULER_TIMEZONE=America/Chicago
SCHEDULER_POLL_INTERVAL_SECONDS=60
UPDATE_DELIVERY_DEADLINE_SECONDS=120

CORS_ORIGINS=http://localhost:3000
LOG_LEVEL=INFO
```

Rules:

1. Do not commit real secrets.
2. Do not log passwords, tokens, phone numbers, API keys, or raw email payloads.
3. Missing required production secrets must fail startup explicitly.
4. Development defaults may use local sinks, mocks, or disabled delivery, but
   production must not silently pretend that delivery succeeded.
5. The two-minute scheduled-update target is a product requirement; measure it
   in integration tests when scheduler code exists.

---

## Core data model

The backend should define schemas and database tables for at least these domain
entities:

| Entity | Required purpose |
| --- | --- |
| `User` | Account identity, unique email, password hash, role, creation timestamp. |
| `UserProfile` | Name, fields of interest, delivery method, email/SMS destination, preferences. |
| `SearchQuery` | User-owned search query, required terms, optional terms, frequency, delivery method, active/deleted status. |
| `SearchStrategy` | Final PubMed/arXiv query strings, AI suggestions, filters, strategy explanation shown to the user. |
| `Article` | Retrieved article metadata: title, authors, abstract, DOI, source database, URL, published date. |
| `ArticleSummary` | AI-generated or non-AI fallback summary tied to article and query/update run. |
| `SearchHistoryItem` | Immutable record of a query execution and returned articles. |
| `SchedulerItem` | User/query schedule, frequency, next update, last run, status. |
| `UpdateRun` | Execution record for each scheduled or manual update. |
| `NotificationLog` | Delivery channel, destination hash or masked value, status, provider message ID, error category. |
| `FeedbackItem` | User feedback on article relevance, summary usefulness, comments, timestamps. |
| `AdminAuditLog` | Admin actions and security-sensitive changes. |

Database rules:

1. Passwords are hashed with a password-hashing algorithm; never store raw or
   reversibly encrypted passwords.
2. User email must be unique.
3. Foreign keys must enforce ownership relationships.
4. Search history should remain available when a user deletes or deactivates a
   query, unless a privacy deletion workflow explicitly removes it.
5. Tests that modify the database must run in transactions or reset state after
   execution.
6. All user-facing data access must be scoped by authenticated user ID unless an
   admin-only route explicitly allows broader access.

---

## API route groups

Use stable route groups and validated request/response schemas.

```text
/api/v1/auth/signup
/api/v1/auth/login
/api/v1/auth/logout
/api/v1/profile
/api/v1/queries
/api/v1/queries/{query_id}
/api/v1/search-history
/api/v1/search-history/{history_id}
/api/v1/updates/run-once
/api/v1/feedback
/api/v1/admin/users
/api/v1/admin/feedback
/api/v1/admin/schedules
/api/v1/health
```

Route handlers should only validate HTTP concerns and call application
services. Business rules belong in pure or mostly pure service/domain functions.

Every protected route must require authentication. Every admin route must check
an admin role or equivalent authorization policy.

---

## Search-query submission workflow

Operational flow:

```text
User submits required/preferred terms and preferences
→ frontend validates required fields
→ FastAPI validates request schema
→ query optimizer proposes optional additions when AI is enabled
→ user-approved/full query is stored
→ search strategy record is created
→ scheduler item is created or replaced
→ immediate retrieval may run for preview/manual update
→ search history item records query execution
→ frontend displays articles, summaries, and search strategy
```

Rules:

1. Users may fully or partially opt out of AI integration.
2. If AI query optimization is disabled, the system should still allow a full
   user-written query.
3. Search-strategy transparency is required: store and show the actual PubMed
   and arXiv query strings or parameters used for each update.
4. Do not overwrite the user's original terms when optimization runs. Store the
   original input and optimized strategy separately.
5. Query edits should create or update scheduler state without destroying prior
   search history.

---

## PubMed integration

The PubMed adapter should be the only code that knows PubMed E-utilities HTTP
syntax.

Important behavior:

1. Build deterministic query strings from validated search strategy objects.
2. Parse article metadata into internal `Article` schemas.
3. Preserve identifiers such as PMID, DOI, source database, article URL, and
   published date when available.
4. Handle empty results, malformed responses, timeouts, rate-limit responses,
   and partial metadata.
5. Never fabricate article metadata.
6. Return typed success/error values or raise precise adapter exceptions.

Testing expectations:

1. Unit-test query construction with fixed inputs.
2. Unit-test response parsing with saved fixtures.
3. Mock HTTP responses for timeout, malformed XML/JSON, empty-result, and
   successful-result paths.

---

## arXiv integration

The arXiv adapter should be the only code that knows arXiv API HTTP/XML syntax.

Important behavior:

1. Build deterministic query strings from validated search strategy objects.
2. Parse XML feed entries into internal `Article` schemas.
3. Preserve arXiv ID, DOI when available, authors, title, abstract, PDF URL,
   source URL, and published date.
4. Handle empty feeds, malformed XML, missing fields, timeouts, and rate-limit
   responses.
5. Never fabricate article metadata.

Testing expectations:

1. Unit-test XML parsing from fixtures.
2. Unit-test query construction.
3. Mock arXiv API responses in integration tests.

---

## Article selection and summarization

Article selection should be deterministic when given the same articles, query,
preferences, feedback, and scoring configuration.

Selection rules:

1. Respect the user's requested number of articles.
2. Prefer recent articles matching required terms.
3. Use preferred terms and user feedback as ranking signals.
4. Keep rejected or lower-ranked articles available for audit/debugging where
   useful.
5. Do not silently exceed the requested article count in delivered updates.

AI summary rules:

1. AI summaries must be tied to a real retrieved article.
2. Summaries must not invent DOI, authors, publication date, links, methods, or
   conclusions not supported by the article metadata/abstract/full text used.
3. Summary length must respect `AI_SUMMARY_MAX_WORDS` or the user preference.
4. The summary object must include model/provider metadata when AI is used.
5. If the user opts out of AI summaries, provide a non-AI fallback such as
   article metadata plus abstract excerpt, clearly labeled as not AI-generated.
6. If the AI provider fails, fail the update stage explicitly or use an
   approved fallback that is visible to the user and logged.

Do not present AI output as a verified scientific conclusion. PaperScout.ai is
an educational literature-discovery tool, not a clinical or scientific authority.

---

## Search-strategy transparency

Search-strategy transparency is a product requirement, not an optional UI
feature.

For every manual or scheduled update, persist enough information to show:

1. User's original input terms.
2. Required terms.
3. Preferred/optional terms.
4. User-provided full query, if supplied.
5. AI-suggested terms, if AI was used.
6. Final PubMed query string or parameters.
7. Final arXiv query string or parameters.
8. Date/time the search ran.
9. Number of articles retrieved and selected.
10. Any filters applied.
11. Plain-English explanation suitable for a young researcher.

The frontend should make this visible from update cards, previous-search pages,
and search-history details.

---

## Scheduler and update delivery

Scheduled update flow:

```text
Scheduler tick
→ fetch due scheduler items
→ lock or mark each due item as running
→ load user profile and active query
→ build search strategy
→ retrieve PubMed/arXiv articles
→ select requested number of articles
→ summarize or create approved fallback
→ format email/SMS/in-app update
→ deliver through configured adapter
→ write notification log and search history
→ update scheduler next-run timestamp
→ prompt user for optional feedback
```

Rules:

1. Scheduled updates must be delivered within two minutes of the scheduled time
   when infrastructure is healthy.
2. Do not send duplicate updates for the same scheduler item and run window.
3. Do not delete search history when a user deletes or disables a query.
4. Delivery adapters must report real success/failure status.
5. Development mocks may simulate success only when explicitly configured.
6. Production must not fabricate successful delivery.
7. If email/SMS is disabled, in-app notification can still be produced when
   supported, but delivery status must reflect the actual channel used.

---

## Notification formatting

`unit-28-update-formatting` should keep user-facing updates simple and
educational.

Each update should include:

1. User's query name or topic.
2. Search run date/time.
3. Article count requested and selected.
4. Article title, authors, source database, publication date, DOI or ID, and
   link when available.
5. AI-generated summary or approved fallback.
6. Feedback links/actions.
7. Link to view the search strategy used.
8. Plain-language note encouraging users to verify articles directly.

Escape HTML and sanitize all user-controlled text before rendering email or web
content.

---

## Feedback workflow

Feedback flow:

```text
User reviews update or previous search
→ user rates article relevance and/or summary usefulness
→ optional comment is submitted
→ API validates ownership and payload
→ feedback record is persisted
→ feedback becomes available to future ranking/summarization logic
→ admin can review aggregate feedback
```

Rules:

1. Feedback is optional.
2. Feedback must be tied to a user, query/update, and article/summary when
   applicable.
3. Only the owning user or an authorized admin can view detailed feedback.
4. Future optimization logic may use feedback, but it must not mutate historic
   search results.

---

## Admin capabilities

Admin features are high-risk and must be explicit.

Admin actions may include:

1. View user accounts and masked contact information.
2. Disable or delete users according to approved privacy policy.
3. Review feedback.
4. Inspect scheduler state.
5. Reschedule or disable problematic jobs.
6. View search/update logs.

Rules:

1. Every admin route requires admin authorization.
2. Every admin write must create an audit log.
3. Admin views must mask sensitive values by default.
4. Admin tools must not reveal raw passwords, tokens, or provider secrets.

---

## Security and privacy requirements

PaperScout.ai handles user identity, email addresses, phone numbers, research
interests, feedback, and generated update content. Treat this as sensitive data.

Required behavior:

1. Use HTTPS/TLS in deployed environments.
2. Hash passwords with a modern password-hashing algorithm.
3. Use secure session or token handling.
4. Encrypt sensitive data at rest where supported by deployment infrastructure.
5. Never store raw passwords.
6. Never log secrets, full tokens, raw passwords, or full SMS/email payloads.
7. Mask contact information in admin and log views.
8. Scope all user reads/writes by authenticated user ID.
9. Validate every API request with schemas.
10. Sanitize user-controlled text before rendering HTML.
11. Keep third-party API keys in environment variables or secret managers.
12. Reject unauthorized access with explicit status codes.

The README uses the phrase "encrypt password" for Unit 38. Implement this as
password hashing and verification, not reversible encryption.

---

## Frontend development

Frontend expectations:

1. Use React components for pages and reusable UI components.
2. Use a single typed or well-documented API client boundary.
3. Keep route-level components small.
4. Validate forms client-side for user experience, but rely on backend
   validation for security.
5. Include loading, empty, and error states for every network call.
6. Make the UI simple and clear for young researchers.
7. Keep search-strategy explanations readable and educational.
8. Use accessible labels, keyboard navigation, semantic HTML, and responsive
   layouts.

Recommended page routes:

```text
/
/about
/about-developer
/signup
/login
/dashboard
/profile
/search-history
/search-history/:historyId
/queries/new
/queries/:queryId/edit
/admin
```

---

## Backend development

Backend expectations:

1. Use FastAPI for HTTP routes.
2. Use Pydantic schemas for request and response validation.
3. Keep route handlers thin.
4. Put business logic in services/domain modules.
5. Put external API, database, AI, email, SMS, and scheduler calls behind
   adapter interfaces.
6. Use explicit exceptions or typed error results.
7. Avoid mutable global state.
8. Keep functions deterministic when they do not require I/O.
9. Pass time, randomness, and provider clients as explicit dependencies when
   possible.
10. Do not perform network calls in domain scoring, formatting, or validation
    functions.

---

## Functional-programming expectations for Python

When writing Python, follow the project coding standards and prefer a
functional style:

1. Use pure functions for transformations, validation, scoring, formatting, and
   schema conversion.
2. Return new values instead of mutating caller-owned values.
3. Prefer frozen dataclasses or immutable Pydantic models for domain records
   where practical.
4. Use first-class functions and dispatch tables for mapping channels, sources,
   or strategy types to handlers.
5. Use comprehensions and functional transformations when they improve clarity.
6. Avoid hidden global state.
7. Isolate side effects in boundary functions.
8. Use full type annotations.
9. Use explicit exceptions or an `Either`-style result type for recoverable
   domain errors when the codebase provides one.
10. Follow the Google Python Style Guide and the repository coding standards.

---

## Commands

The Makefile should expose these commands when the implementation reaches the
corresponding stage. Do not document commands as working in `README.md` until
implemented and tested.

| Command | Purpose | Expected behavior |
| --- | --- | --- |
| `make install` | Install frontend and backend dependencies. | Creates local environments and installs packages. |
| `make install-backend` | Install Python dependencies. | Creates/uses a virtual environment or Docker image. |
| `make install-frontend` | Install React dependencies. | Runs package-manager install for frontend. |
| `make dev` | Start local full stack. | Starts API, web, database, and scheduler in development mode. |
| `make dev-api` | Start FastAPI only. | Runs API with reload enabled. |
| `make dev-web` | Start React frontend only. | Runs frontend dev server. |
| `make scheduler-run-once` | Execute due scheduled jobs once. | Processes due scheduler items and exits. |
| `make db-upgrade` | Apply database migrations. | Updates database to latest schema. |
| `make db-downgrade` | Roll back one migration. | Reverts schema according to migration tool. |
| `make db-reset-dev` | Reset local development database. | Drops/recreates dev database only. |
| `make test` | Run all tests. | Runs backend, frontend, integration, and E2E tests as configured. |
| `make test-unit` | Run all unit tests. | Runs pytest unit tests and frontend unit tests. |
| `make test-backend` | Run backend tests. | Runs pytest. |
| `make test-frontend` | Run frontend tests. | Runs Jest/React Testing Library tests. |
| `make test-integration` | Run integration tests. | Uses PostgreSQL and mocked third-party APIs. |
| `make test-e2e` | Run browser E2E tests. | Runs Cypress against local stack. |
| `make coverage` | Generate coverage report. | Produces backend/frontend coverage output. |
| `make lint` | Run linters. | Checks Python and JS style. |
| `make format` | Format code. | Applies configured formatters. |
| `make docker-build` | Build Docker images. | Builds API and web images. |
| `make docker-up` | Start Docker Compose stack. | Starts local services. |
| `make docker-down` | Stop Docker Compose stack. | Stops and removes local containers. |
| `make docker-logs` | Show service logs. | Streams or prints Compose logs. |
| `make preflight` | Validate local configuration. | Checks required env vars, DB connectivity, and provider configuration. |

---

## Command flags and parameters

Prefer environment variables over ad hoc CLI flags for service configuration.
When scripts need parameters, use explicit names.

Recommended script parameters:

| Script | Parameter | Purpose |
| --- | --- | --- |
| `scripts/test/run_backend_tests.py` | `--unit` | Run only backend unit tests. |
| `scripts/test/run_backend_tests.py` | `--integration` | Run backend integration tests. |
| `scripts/test/run_backend_tests.py` | `--coverage` | Generate coverage reports. |
| `scripts/test/run_e2e.py` | `--base-url` | Frontend URL for Cypress tests. |
| `scripts/build/build_images.py` | `--service` | Build one service image. |
| `scripts/deploy/deploy.py` | `--environment` | Select staging or production. |
| `scripts/scheduler/run_once.py` | `--now` | Supply an explicit timestamp for deterministic testing. |
| `scripts/retrieval/smoke_test.py` | `--source` | Select `pubmed`, `arxiv`, or `all`. |
| `scripts/retrieval/smoke_test.py` | `--query` | Provide a test query string. |

Script behavior must be deterministic when `--now`, fixed fixtures, and mocked
providers are supplied.

---

## Docker and Docker Compose workflow

Docker should be the primary full-stack development and validation workflow.

Expected services:

```yaml
services:
  paperscout-api:
    build:
      context: .
      dockerfile: Dockerfile.api
    env_file:
      - config/development/.env
    depends_on:
      - postgres

  paperscout-web:
    build:
      context: .
      dockerfile: Dockerfile.web
    environment:
      PAPERSCOUT_API_BASE_URL: http://paperscout-api:8080
    depends_on:
      - paperscout-api

  scheduler:
    build:
      context: .
      dockerfile: Dockerfile.api
    command: python -m paperscout_scheduler.run
    env_file:
      - config/development/.env
    depends_on:
      - paperscout-api
      - postgres

  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: paperscout
      POSTGRES_PASSWORD: paperscout
      POSTGRES_DB: paperscout
```

Rules:

1. Do not require host-only services for normal development.
2. Do not commit real `.env` files with secrets.
3. Keep Docker Compose defaults development-safe.
4. Production deployment must provide real secrets through the deployment
   platform or secret manager.
5. Use separate test database names for integration tests.

---

## Testing

Testing is a core design requirement. The README target is greater than 90%
coverage and early detection of edge cases.

Run:

```bash
make test
```

Unit testing should cover:

1. React component rendering and interactivity with Jest and React Testing
   Library.
2. Frontend routing, form inputs, conditional rendering, and responsive states.
3. Backend pure functions with pytest.
4. Database access functions with isolated transactions or rollback.
5. PubMed and arXiv request construction and response parsing with mocked
   responses.
6. AI summary structure, word limits, opt-out behavior, and provider-failure
   behavior.
7. Update formatting, escaping, and link/metadata inclusion.
8. Password hashing and verification.
9. Scheduler next-run calculations.
10. Authorization checks for protected and admin routes.

Integration testing should cover:

1. User sign-up and login.
2. Profile update and scheduler preference propagation.
3. New query submission and search-history creation.
4. Automated literature retrieval with mocked PubMed/arXiv responses.
5. Update generation, formatting, and delivery logging.
6. Search-strategy transparency from retrieval through UI display.
7. Feedback submission and later availability for optimization.
8. Frontend-backend request/response behavior through Cypress.
9. PostgreSQL constraints, rollback behavior, and foreign keys.

Do not skip tests for behavior changes unless the approved implementation plan
explicitly allows it.

---

## Test data and mocking

Use controlled fixtures for reproducible tests.

Recommended fixture categories:

1. PubMed successful response.
2. PubMed empty response.
3. PubMed malformed response.
4. PubMed timeout/rate-limit response.
5. arXiv successful XML feed.
6. arXiv empty feed.
7. arXiv malformed XML feed.
8. AI summary success response.
9. AI provider failure response.
10. Email delivery success/failure response.
11. SMS delivery success/failure response.
12. Scheduler due/not-due examples.

Third-party API tests should use mocks by default. Live API smoke tests may be
added, but they must be opt-in and excluded from normal CI unless explicitly
approved.

---

## CI/CD workflow

GitHub Actions should run on pull requests and major feature branches.

Required CI stages:

1. Install dependencies.
2. Lint.
3. Format check.
4. Backend unit tests.
5. Frontend unit tests.
6. Integration tests with PostgreSQL service container.
7. Cypress E2E tests when the full stack is ready.
8. Coverage reporting through Coveralls or the selected coverage service.
9. Docker image build check.

CI must fail on test failures, lint failures, migration failures, or missing
required configuration for the selected environment.

---

## Development workflow

Branch rules from the planning README:

1. `main` contains production-ready code only.
2. `develop` is the feature-integration branch.
3. Feature branches branch from `develop`.
4. Bug branches branch from `develop`.
5. Rebase feature and bug branches onto `develop` regularly.
6. Merge to `develop` through PRs after code review and passing tests.
7. Merge `develop` to `main` through a release PR.
8. Use Semantic Versioning in `MAJOR.MINOR.PATCH` format.

Before merging:

1. Confirm the approved ticket scope is complete.
2. Run relevant unit and integration tests.
3. Update documentation for changed commands, behavior, or setup.
4. Confirm secrets are not committed.
5. Confirm license notices are preserved.

---

## Documentation requirements

Update documentation whenever a change affects:

1. User behavior.
2. Developer workflow.
3. Commands, flags, or parameters.
4. Setup steps.
5. Docker or Docker Compose behavior.
6. Environment variables.
7. Database schema or migrations.
8. API request/response contracts.
9. Search strategy behavior.
10. Scheduler behavior.
11. Notification delivery behavior.
12. AI summarization or AI opt-out behavior.
13. Privacy, security, or data retention behavior.

Documentation locations:

| File or directory | Required contents |
| --- | --- |
| `README.md` | User-facing overview, setup, run, and usage instructions. |
| `DEVELOPER.md` | Developer onboarding, architecture, commands, testing, runtime behavior. |
| `AGENTS.md` | AI-agent planning, scope, coding, and testing instructions. |
| `docs/architecture/` | System diagrams, data flow, scheduler design, security model. |
| `docs/api/` | API routes, schemas, status codes, auth requirements. |
| `docs/user-guides/` | Simple instructions for young researchers using the product. |

---

## Quality gates

A change is not ready unless:

1. It satisfies the approved requirements.
2. It includes unit tests for pure logic.
3. It includes integration tests for cross-boundary behavior when relevant.
4. It preserves user security and privacy.
5. It preserves search-strategy transparency.
6. It does not fabricate third-party API, AI, or notification success.
7. It keeps UI language simple and accessible.
8. It updates documentation when behavior changes.
9. It runs through Docker or Docker Compose when full-stack behavior is
   affected.
10. It keeps the AGPL license intact.

---

## Known implementation cautions

1. The planning README says SQLite for development and PostgreSQL for
   production. Integration tests should use PostgreSQL to catch constraint and
   SQL behavior differences.
2. The planning README mentions Django-style database tests in one subsection,
   but FastAPI is the chosen backend framework. Do not add Django unless an
   approved plan changes the architecture.
3. Unit 38 is named "encrypt password" in the README. Implement password
   hashing and verification, not reversible encryption.
4. The source tree in the README omits unit directories 11-20, while the design
   section defines Units 11-20. Create all 42 directories or document any
   approved consolidation.
5. PaperScout.ai should help young researchers learn. Avoid opaque search and
   AI behavior; expose strategy explanations wherever updates are shown.
6. AI opt-out is a functional requirement. Do not make AI mandatory for query
   submission, retrieval, or viewing article metadata.
7. Search history must remain useful after query edits/deletions unless a
   privacy deletion feature explicitly removes it.
8. Email/SMS delivery involves sensitive contact information. Mask and redact
   aggressively.
9. Production must fail loudly on missing required provider configuration;
   local development may use explicit mocks.

---

## Minimum first implementation slice

For a safe MVP scaffold, implement in this order:

1. Repository scaffolding, Docker Compose, Makefile, and configuration examples.
2. Backend schemas and pure domain functions.
3. User database, password hashing, sign-up, and login.
4. React landing, sign-up, login, and dashboard skeletons.
5. Search query creation and search-history persistence.
6. PubMed/arXiv adapters with mocked tests.
7. Search strategy transparency records and UI display.
8. Article selection and non-AI fallback formatting.
9. AI summarization behind an explicit provider boundary.
10. Scheduler and update delivery logging.
11. Email/SMS adapters behind explicit configuration gates.
12. Feedback workflow and feedback-informed ranking.
13. Admin capabilities with audit logging.
14. Full E2E tests and deployment hardening.

Do not build admin tools, SMS delivery, or AI optimization before the core user,
query, retrieval, history, and transparency workflow exists.
