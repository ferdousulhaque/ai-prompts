# CLAUDE.md — Global Engineering Rules

Applies to every project. A repository's own CLAUDE.md adds stack, commands and branch
names, and wins where it conflicts with this file.

## 1. How to work

- Plan first: before implementing, give me a plan with tasks, affected components, risks
  and test plan. Once I approve it, work through it without stopping after each task.
- Stop and ask before: changing scope, excluding functionality or data, changing
  production-like data, changing infrastructure, accepting a security risk, any
  destructive or irreversible command, and merging, deploying or releasing.
- Do not expand scope opportunistically. Log newly found work as backlog items.
- Ask one focused question when a decision blocks safe progress. Do not invent missing
  business rules.
- Separate facts, assumptions and unknowns. Never claim work, tests or verification
  that did not actually happen.

## 2. Understand before changing

- Read the relevant code, configuration, schema and dependencies first. Trace the
  feature end to end: UI → application logic → database → external systems.
- For existing user-facing behaviour, look at it in a browser before rebuilding it.
- Inspect databases read-only first. Never use production customer data in development
  or test; mask or synthesize it.

## 3. Git safety

- Follow the repository's branch model. If none is documented, ask.
- Check `git status` before branching or committing, and preserve my existing changes.
- Never commit directly to protected branches.
- Never discard, reset, overwrite, amend or force-push my work without approval.
- Keep commits focused; do not mix unrelated changes.

## 4. Docker environment

- Provide a reproducible Docker dev environment with pinned versions, health checks and
  documented start/stop/reset commands.
- Keep secrets out of source control: env vars, ignored local env files, `.env.example`.
- Do not connect containers to production databases or services by default.

## 5. Verification

- Run the relevant tests and review the full diff before reporting. Record the exact
  commands and results.
- User-facing changes need browser verification (Playwright preferred) with the relevant
  roles, covering normal, empty, invalid, unauthorized and error states, and confirming
  the resulting backend and database state. Unit tests alone are not enough.
- If browser automation is unavailable, say so and give a manual checklist.

## 6. External integrations

- Document each external dependency: contract, auth, payloads, errors, retries, timeouts.
- If a system is unavailable, build a clearly labelled mock that follows the contract,
  is isolated from production code and cannot be enabled in production.
- Report what was tested against a mock versus a real sandbox. A mock never counts as
  verifying the real integration.

## 7. Security

- Validate and authorize server-side on every sensitive operation; apply least privilege.
- Use the framework's own protections for output escaping, parameterized queries, CSRF,
  sessions and file uploads rather than hand-rolled ones.
- Check dependencies for known vulnerabilities and maintenance status.
- Never suppress a security finding to make a pipeline pass. Accepted risks need my
  explicit approval and a record in `docs/security/`.

## 8. Logging

- Route application, server, worker and container logs to the project's central logging
  stack, structured, with timestamp, severity, service, environment and request ID.
- Never log secrets, tokens, session IDs, full request bodies or unneeded personal data.
- Verify ingestion with a known test event before calling observability ready.

## 9. Migration and legacy work (only when the project involves it)

- Migrate only functionality I have confirmed as needed. Old does not mean unnecessary:
  record the evidence and get approval before excluding anything.
- Keep the new implementation in a separate workspace from the legacy source unless I
  approve otherwise.
- Document source-to-target mappings, transformations, exclusions and reconciliation.
- Make migrations repeatable. Test on a disposable non-production database first, then
  validate counts, references and files after each batch.
- Take and verify a backup before any approved data-changing operation.
- Never silently drop fields, records, tables, users, roles or files.

## 10. Documentation

Maintain in each project:

- `README.md` — setup and common commands
- `docs/architecture/` — context, components, data flows
- `docs/discovery/` — inventory of existing pages, modules, database, integrations
- `docs/migration/` — scope, mapping, backlog, exclusions, reconciliation
- `docs/testing/` — strategy, cases, results, evidence
- `docs/security/` — findings and accepted risks
- `docs/operations/` — Docker, logs, backups, deployment, rollback
- `docs/decisions/` — architecture decision records
- `docs/progress/CURRENT-STATUS.md` — session handover

Before ending a session, update `CURRENT-STATUS.md` with: current task, last completed
action, files changed, commands and tests run with results, branch and git status,
blockers and decisions needed, and the next recommended task.

At the start of a session, read `CURRENT-STATUS.md` and the relevant decision records.
Do not assume undocumented work was completed.

## 11. Done means

Approved scope implemented with nothing unrelated, tests run and recorded, browser
verification done for user-facing changes, security implications reviewed, data and
integration effects checked, docs and status updated, diff reviewed.
