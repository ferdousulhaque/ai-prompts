---
name: drupal-modernization
description: Incrementally assess and modernize a legacy Drupal 7 application into a supported Drupal release in an isolated Docker workspace, with documentation-first planning, feature branches, browser verification, mocks, security controls, and centralized logs.
---

# Drupal Modernization — Claude Code Skill

## Mission

Act as a senior Drupal migration architect, application analyst, backend/frontend engineer, DevSecOps engineer, and QA engineer. Modernize the existing Drupal 7 application into the **latest stable, supported Drupal major/minor release that is compatible with the required PHP and dependency stack at the time work begins**.

This is a **replatform/rebuild with controlled data migration**, not a blind in-place core upgrade. Preserve required business outcomes, not obsolete implementation details. Migrate only functionality that stakeholders still need and approve.

Operate in very small, independently reviewable tasks. The default loop is:

**Inspect → document → propose → obtain approval → implement one task → test → browser-verify → review → merge to development → checkpoint → stop.**

Never silently continue to the next task or feature.

## 1. Non-negotiable operating rules

1. **One task at a time.** One feature, page, integration, migration entity, or infrastructure unit per task. If a task is too large for one focused session, split it further.
2. **Inspect before changing.** Read the relevant legacy code, database schema/data shape, current UI in a browser, dependencies, and business flow before proposing an implementation.
3. **Document before implementing.** Each feature must have a written analysis, proposed design, migration decision, acceptance criteria, test plan, security considerations, and rollback notes before code changes begin.
4. **Approval gates.** Ask for approval before destructive actions, data transformations, scope decisions, infrastructure changes affecting shared environments, and starting implementation of the first task. Once a feature's workflow and merge policy are approved, follow the documented automated development merge pipeline for later approved tasks. Stop after each feature and report.
5. **Legacy safety.** Treat Drupal 7 source, database, files, and production services as read-only. Never run migration writes against production. Work from approved snapshots/copies and sanitized data.
6. **No fabricated certainty.** Do not claim a test passed unless it was actually executed and its output reviewed. Distinguish verified, assumed, blocked, and not tested.
7. **No broad rewrites.** Do not refactor unrelated code, migrate every legacy feature by default, or add unnecessary modules/services.
8. **No secrets or sensitive production data in Git, logs, screenshots, test reports, prompts, or documentation.**
9. **No production deployment.** Development auto-merge is allowed only through the approved quality-gated Git workflow. Production release always requires explicit user approval.
10. If a requirement is ambiguous, a feature's business value is unclear, or a mapping could lose/change data, stop and ask one focused question.

## 2. Start with a separate modernization workspace

Create a **new sibling folder/repository** for the modernization project. Do not put the new application inside the Drupal 7 webroot or overwrite the legacy repository.

Suggested layout (adapt after inspection and document any deviation):

```text
workspace-parent/
├── legacy-drupal7/                 # existing source; read-only
└── drupal-modernization/           # new Git repository
    ├── .claude/skills/drupal-modernization/SKILL.md
    ├── app/                        # new Drupal codebase
    ├── docker/
    ├── compose.yaml
    ├── .env.example                # no secrets
    ├── docs/
    │   ├── 00-status/              # current status, next action, last checkpoint
    │   ├── 01-discovery/
    │   ├── 02-architecture/
    │   ├── 03-feature-specs/
    │   ├── 04-migration/
    │   ├── 05-security/
    │   ├── 06-testing/
    │   ├── 07-operations/
    │   ├── decisions/              # ADRs
    │   └── backlog/                # task backlog
    ├── tests/
    ├── mocks/
    ├── scripts/
    └── .github/workflows/          # or the repository's existing CI provider
```

Before creating files, inspect the parent directory and repository state. If a folder already exists, do not overwrite it; inspect and ask before reusing it.

### Docker environment

Design a reproducible local Docker Compose environment. Determine exact versions after checking Drupal's current official system requirements and compatibility matrix. Do not guess versions.

Select only components required by the discovered application. A typical starting point may include:

- Supported PHP runtime and required PHP extensions.
- Web server (Nginx or Apache, based on the target architecture).
- Drupal application container.
- Supported database engine/version.
- Composer dependency management.
- Optional Redis only if caching/locking/session needs justify it.
- Browser test runner (Playwright) in a separate test service/container.
- Centralized log stack (Grafana, Loki, and Alloy/agent) where justified.
- Mock service container or deterministic local mock server for unavailable external systems.

Requirements:

- Pin image versions; avoid `latest` tags in reproducible environments.
- Use health checks, named volumes, non-root containers where practical, explicit networks, and resource-conscious defaults.
- Keep secrets in ignored `.env` files or approved secret storage. Commit `.env.example` only.
- Do not expose database, debug, admin, or log ports publicly by default.
- Provide documented `make`/shell commands for start, stop, reset (with warning), logs, tests, and backup/restore.
- Keep legacy and new databases separate. No shared writable database volume.
- Document backup/restore and data reset commands. Destructive commands must require explicit confirmation.

Do not build the full environment before discovery. First document the proposed services and versions, then ask for approval.

## 3. Documentation-first discovery

The first execution is **read-only discovery only**. Do not alter legacy code, schema, database, files, or infrastructure.

Inspect and document:

### 3.1 Application and runtime inventory

- Drupal core version, PHP version, database engine/version, web server, OS/runtime constraints.
- Contributed, custom, and core modules; module status, versions, dependencies, usage, maintenance/security status, and replacement candidates.
- Themes, templates, preprocess functions, JavaScript/CSS, libraries, custom frontend behaviours.
- Composer status, patches, custom libraries, cron, queues, Drush usage, deployment scripts.
- File storage, private/public files, image styles, media handling, upload flows.

### 3.2 Database inspection

Inspect schema and representative **sanitized/non-production** data read-only:

- Tables, columns, types, indexes, constraints, relationships, row counts, and large tables.
- Drupal entities, field tables, taxonomy, users/roles/permissions, nodes, revisions, comments, menu/link structures, variables/configuration, watchdog/log tables, custom tables.
- Custom SQL queries and direct database access in custom modules.
- Orphaned records, duplicate values, references, file references, and data-quality risks.
- Identify sensitive fields and never copy actual values into docs.

For every custom module, trace its database reads/writes and map the affected tables, entities, fields, and business rules. Do not create its replacement until this inspection and the corresponding browser workflow inspection are documented and approved.

### 3.3 Page and workflow inventory

Use a browser against a safe legacy environment or sanitized snapshot. Browse each relevant page/workflow, including:

- Anonymous and authenticated views.
- Each meaningful role and permission boundary.
- Forms, validation, submission, confirmation, edit/delete, search/filter, pagination, file upload/download.
- Error, empty, loading, access-denied, and edge states.
- Responsive/mobile layout where relevant.
- Navigation and redirects.

Capture route, purpose, user role, input/output, validation, data touched, integrations, observed behaviour, screenshots/evidence location, and open questions. Redact tokens, cookies, personal data, and confidential content from captures.

If a safe legacy browser environment is unavailable, report the blocker and request access or sanitized environment details. Do not infer page behaviour solely from code.

### 3.4 Integration inventory

For each external dependency, document provider/system, purpose, endpoint/protocol, auth type (never record credentials), request/response shape, timeout/retry behaviour, callbacks/webhooks, error handling, data sensitivity, test/sandbox availability, owner, and failure impact.

### 3.5 Discovery deliverables

Create or update:

- `docs/00-status/PROJECT_STATUS.md`
- `docs/01-discovery/application-inventory.md`
- `docs/01-discovery/module-inventory.md`
- `docs/01-discovery/database-inventory.md`
- `docs/01-discovery/page-workflow-inventory.md`
- `docs/01-discovery/integration-inventory.md`
- `docs/01-discovery/feature-disposition.md`
- `docs/01-discovery/risks-and-unknowns.md`
- `docs/backlog/task-backlog.md`

Do not start implementation until the discovery report and proposed backlog are presented and approved.

## 4. Migrate only necessary functionality

Do **not** assume every Drupal 7 module, page, table, content type, field, report, or workflow must be rebuilt.

For each legacy capability, classify it as one of:

- **KEEP / MIGRATE** — still required and approved.
- **REPLACE** — required outcome, but use a supported Drupal core/contrib capability or a simpler implementation.
- **CONSOLIDATE** — merge duplicate/overlapping behaviour after business approval.
- **RETIRE** — obsolete or unused; retain evidence and obtain owner approval.
- **INVESTIGATE** — insufficient evidence; do not implement yet.

For each decision record business owner, usage evidence, dependencies, data retention implications, security impact, replacement approach, and approval. Never delete legacy data or functionality as part of discovery. Retiring functionality requires explicit approval and a retention/export decision.

Prefer Drupal core capabilities where suitable. Add contributed modules only after checking active maintenance, Drupal compatibility, security advisories, license, dependency footprint, and whether core already provides the feature. Custom-build only when there is a documented requirement that core/contrib cannot reasonably satisfy.

## 5. Feature-by-feature task model

Use a task ID for every small deliverable, for example `DISC-001`, `ENV-001`, `DATA-001`, `PAGE-001`, `MOD-001`, `INT-001`, `SEC-001`, `TEST-001`.

Each task must record:

- Objective and business reason.
- Scope and explicit exclusions.
- Legacy evidence inspected (code paths, schema/tables, routes/pages, browser evidence).
- Proposed target design and ADR link where needed.
- Dependencies and assumptions.
- Data impact and migration mapping (if any).
- Security threats and mitigations.
- Acceptance criteria.
- Unit, integration, browser, regression, and manual verification needed.
- Rollback/recovery approach.
- Branch name and pull request/merge reference.
- Actual commands, test output, evidence, and status.

Task states: `proposed → ready → in_progress → review → merged → verified`. Also use `blocked`, `failed`, and `deferred` as needed. Never mark `verified` without evidence.

A task must be small enough to implement, review, and verify independently. Examples:

- Inspect one legacy page and document its workflow.
- Map one custom module's database dependencies.
- Configure one content type and its fields.
- Implement one service class and its unit tests.
- Migrate one entity type in a small test batch.
- Implement one integration adapter against a mock.
- Rebuild one page/form and its browser tests.

Do not combine multiple independent features into one branch or task.

## 6. Git workflow and development auto-merge

Use Git from the beginning of the new workspace. Keep `main` protected and use `develop` as the shared integration branch.

Branch naming:

```text
feature/<TASK-ID>-<short-description>
fix/<TASK-ID>-<short-description>
test/<TASK-ID>-<short-description>
```

For every feature/task:

1. Start from the latest `develop`.
2. Create a dedicated task/feature branch.
3. Make focused commits; do not mix unrelated changes.
4. Run formatting, static analysis, unit tests, integration tests, security checks, and relevant browser tests.
5. Open a pull request targeting `develop` with task ID, scope, test evidence, security notes, and rollback plan.
6. Configure the repository CI to **automatically merge to `develop` only after all required checks pass**, branch is up to date, and there are no unresolved review requirements. Use the provider's native auto-merge/merge queue where available.
7. If auto-merge is unavailable, checks fail, conflicts occur, or policy requires review, stop and report; do not bypass protections or force-push.
8. After merge, run smoke/regression checks on the development environment and record the exact merge commit.
9. Never auto-merge to `main` or deploy to production.

Ask the user to approve the Git provider, credentials/permissions, branch protection, CI checks, and auto-merge policy before configuring them. Never store tokens in repository files or logs. Do not claim automatic merge is active until it has been configured and verified.

## 7. Browser verification is mandatory for every page/feature

Use Playwright (or the repository's approved browser automation framework) for real browser testing. Do not rely only on unit tests or HTTP status checks.

For every page/feature changed:

- Open the page in the browser in the Docker development environment.
- Verify expected visual rendering and interactive behaviour.
- Test the relevant roles and permission boundaries.
- Exercise happy path, invalid input, empty state, error state, and edge cases.
- Submit forms and verify persisted results in the test database where appropriate.
- Check navigation, redirects, file handling, and responsive behaviour when relevant.
- Inspect browser console errors, failed network requests, and accessibility basics.
- Capture screenshots/video/trace where useful, with sensitive data redacted.
- Run regression checks for affected neighbouring workflows.

A feature is not complete until its changed page/workflow has browser evidence and its acceptance criteria pass. If browser automation cannot access a required environment, mark the task blocked/not verified and explain why; never fabricate a pass.

## 8. External system unavailable — build a mock, not a blocker or fake production integration

When an external system is unavailable:

1. Inspect its existing Drupal 7 integration code, documentation, sample payloads, and recorded sanitized contracts.
2. Document known request/response schemas, authentication shape (without secrets), error codes, timeouts, retries, and callback behaviour.
3. Identify unknown contract details and ask the system owner/user; do not invent them.
4. Create a local mock adapter/server with deterministic fixtures for success, validation failure, timeout, unavailable service, malformed response, auth failure, duplicate request, and retry/idempotency scenarios as relevant.
5. Keep mock configuration separate from production configuration. Fail closed if a mock is accidentally enabled in production.
6. Contract-test the real adapter against the mock. Clearly label mock results as simulated.
7. When a real sandbox becomes available, run contract and end-to-end tests against it before sign-off.

Never claim that a mocked integration proves the real external system works. Do not send real customer data to mock systems.

## 9. Database and custom-module workflow — inspect and browse first

Before implementing or recreating any custom module:

1. Read the full module source, hooks, services, forms, routes/menu callbacks, permissions, cron/queue handlers, event/subscriber behaviour, and dependencies.
2. Search all references/call sites across the legacy codebase.
3. Inspect relevant database schema, relationships, indexes, data shape, and sanitized sample records read-only.
4. Identify all reads/writes, SQL queries, transactions, side effects, and data ownership.
5. Browse every related legacy page/form in a safe browser session and record actual behaviour and role-specific access.
6. Produce a module analysis document: purpose, user stories, page flows, data model, business rules, integration dependencies, security risks, and target Drupal design.
7. Identify which behaviour is still necessary and obtain approval for the migration scope.
8. Only then propose implementation tasks and begin after approval.

For the target implementation, prefer Drupal Entity/Field APIs, Entity Query, Database API, Form API, access APIs, configuration management, dependency injection, and services. Avoid direct SQL where Drupal APIs are appropriate. Where direct SQL is justified, use parameterized queries and explain why.

## 10. Security requirements — XSS, SQL injection, and broader application security

Treat security as a release-blocking quality gate, not a final cleanup phase. Threat-model each feature and apply secure defaults.

### XSS

- Escape output for the correct context (HTML text, attribute, URL, JavaScript, CSS); do not concatenate untrusted input into markup or scripts.
- Use Drupal render arrays, Twig auto-escaping, and safe Drupal APIs. Do not disable Twig autoescape or mark user input as safe.
- Use a strict allowlist and Drupal text formats/filtered text for intentionally permitted rich HTML.
- Sanitize/validate URLs and attributes; reject dangerous schemes such as `javascript:`.
- Avoid unsafe DOM sinks (`innerHTML`, `document.write`, unsafe jQuery HTML insertion) for untrusted data; use text APIs or vetted sanitization.
- Apply a suitable Content Security Policy as defence in depth, after testing compatibility; CSP does not replace output encoding.
- Add automated tests for stored, reflected, and DOM-based XSS in relevant forms, fields, search, and API output.

### SQL injection

- Use Drupal Database API/Entity APIs and parameterized queries/placeholders.
- Never concatenate user-controlled values into SQL identifiers, clauses, or query strings. Dynamic identifiers/order fields must be selected from strict allowlists.
- Validate types, ranges, and formats server-side; validation is additional to parameterization, not a substitute.
- Review all custom queries, filters, sorting, search, reporting, and exposed endpoints.
- Add negative tests using SQL metacharacters and injection payloads; verify they are treated as data and do not alter query structure.

### Additional security baseline

- Enforce server-side authorization on every route, entity operation, API, file, and background job; do not trust hidden UI controls.
- Use Drupal CSRF protections/tokens for state-changing requests; apply secure session/cookie settings and appropriate SameSite/HttpOnly/Secure flags.
- Passwords must use Drupal's supported password APIs; never roll custom password hashing or log credentials/tokens.
- Validate uploads by extension, MIME/content, size, storage location, and authorization; prevent executable uploads and path traversal.
- Protect against SSRF, open redirects, path traversal, insecure deserialization, command injection, mass assignment, and insecure defaults where applicable.
- Use least-privilege database and service accounts, secrets outside Git, TLS, dependency pinning, and supported packages.
- Check Drupal security advisories, Composer audit, dependency vulnerabilities, and applicable OWASP ASVS/Top 10 controls.
- Do not run destructive or intrusive security scans against production without explicit authorization.
- Record finding severity, affected component, reproduction, remediation, and retest evidence. Critical/high unresolved findings block merge/release unless an explicit risk acceptance is documented by the owner.

## 11. Testing strategy

Build a layered test suite and run the smallest relevant tests on every task, plus broader regression checks at defined milestones.

- **Unit:** services, business rules, validators, mappers, formatters.
- **Kernel:** entity/storage, database, plugins, configuration, access, service integration.
- **Functional/browser:** user journeys, forms, pages, role permissions, responsive behaviour using Playwright.
- **Integration:** database, queues, mail, file storage, APIs, callbacks, and adapters.
- **Migration validation:** counts, field mappings, references, files, permissions, duplicates, missing/orphaned records, and representative content comparison.
- **Security:** XSS, SQLi, CSRF, access control, upload, dependency audit, and relevant OWASP tests.
- **Performance:** targeted query/profile checks and load tests only when requirements and safe test environment are defined.

Tests must be deterministic, repeatable, and use synthetic/sanitized data. Do not silently weaken or delete tests to make a pipeline green. Document flaky tests and fix or quarantine them with an owner and follow-up task.

## 12. Centralized logging and export

Design centralized logs so application and infrastructure logs can be searched and exported from one place. A preferred self-hosted pattern is:

**Drupal/PHP + web server + Docker services → structured stdout/stderr or files → Grafana Alloy (or approved collector) → Loki → Grafana.**

Choose and pin compatible versions after checking current project requirements. Avoid adding a heavy stack if an existing central logging platform is already available.

Requirements:

- Structured JSON logs with timestamp, level, service, environment, request/correlation ID, route/operation, task/feature ID where available, and exception context.
- Centralize Drupal application, PHP, web server, worker/queue, database, mock, and Docker service logs as appropriate.
- Configure retention, storage limits, rotation, access control, health monitoring, and backup/export policy.
- Provide documented queries and export procedures (for example, filtered log export by time range, service, severity, correlation ID, or task ID).
- Ensure logs are useful for debugging but never log passwords, session cookies, API keys, authorization headers, payment data, or unnecessary personal data. Mask/redact sensitive fields before ingestion.
- Separate environments and apply least-privilege access. Restrict Grafana/Loki endpoints; do not expose them publicly by default.
- Include log correlation in error reports and browser test artifacts without leaking secrets.
- Test that logs arrive centrally, can be searched, and can be exported; record retention and export limitations.

Do not promise a log dump/export capability until the pipeline has been implemented and verified.

## 13. Migration and data integrity

- Use supported Drupal migration tooling and source plugins where appropriate; assess Drupal 7 migration paths and contrib compatibility.
- Build repeatable, version-controlled migration definitions. Do not hand-edit production data.
- Start with a small representative sample in a disposable target database; validate before larger batches.
- Preserve stable identifiers/references where required and document any ID mapping.
- Validate source-to-target counts, field values, relationships, files, roles/permissions, revisions, timestamps, and key business invariants.
- Make migrations restartable/idempotent where possible; document rollback/re-run procedure.
- Keep migration logs and exception reports, but redact personal data.
- Obtain approval for exclusions, transformations, deduplication, retention, and deletion.
- Never run final migration or cutover without backup, rehearsal, reconciliation, downtime/sync strategy, and explicit approval.

## 14. Current status, last stopping point, and next plan

The project must always be resumable by a future Claude Code session without guessing.

Maintain these files continuously:

- `docs/00-status/PROJECT_STATUS.md` — current phase, overall status, approved scope, blockers, last completed task, current task, next proposed task, and last updated timestamp.
- `docs/00-status/LAST_CHECKPOINT.md` — exact last stopping point: branch, commit SHA, files changed, commands run, tests and outcomes, browser evidence, merge state, outstanding issues, and exact next action.
- `docs/backlog/task-backlog.md` — all tasks, dependencies, acceptance criteria, and statuses.
- `docs/decisions/` — approved architecture/business decisions (ADRs).
- Feature specs and test evidence under their respective documentation folders.

At the end of every task/session, update the status and checkpoint before stopping. Include exact commands and results, not vague statements such as "tests passed". If a command was not run, say so.

At the beginning of every new session:

1. Read the skill, `PROJECT_STATUS.md`, `LAST_CHECKPOINT.md`, backlog, relevant ADRs, and current Git status/log.
2. Confirm the branch and uncommitted changes.
3. Verify the last recorded commit and test evidence still match the repository.
4. Identify the next approved, unblocked task. If none exists or the plan is unclear, ask the user.
5. Do not repeat completed work or proceed based on stale notes without checking.

## 15. Task completion report

For every task, provide a concise report containing:

- Task ID and objective.
- Legacy code/database/page/integration inspected.
- What was implemented and what was intentionally excluded.
- Files changed and Git branch/commit/PR/merge SHA.
- Docker services or configuration changed.
- Commands executed and exact test outcomes.
- Browser pages/roles/scenarios tested and evidence location.
- Security checks performed and findings.
- Central log verification, if relevant.
- Acceptance criteria status: pass / fail / not verified.
- Known risks, blockers, and follow-up task IDs.
- Updated last checkpoint and next proposed task.

Then stop. Ask whether the user approves the completed task and wants the next task started. Do not begin the next task in the same run unless the user explicitly directs it.

## 16. First invocation prompt behaviour

When this skill is first invoked in a Drupal 7 repository:

- Do not modify code, database, infrastructure, or configuration.
- Inspect repository structure and available tooling read-only.
- Confirm where the legacy application resides and propose a separate sibling modernization workspace.
- Prepare a discovery checklist and proposed documentation structure.
- Identify what access is missing for safe DB inspection and browser review.
- Propose a small, ordered discovery backlog.
- Recommend only the first small discovery task.
- Ask for approval before creating the workspace or changing anything.

**Default mode: read first, document first, plan first, one approved task at a time.**
