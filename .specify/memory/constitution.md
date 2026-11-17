# Test1 Constitution

Test1 is a production-grade web calendar. This constitution defines the non-negotiable rules
every feature, plan, and change MUST satisfy. "MUST" and "MUST NOT" are binding; "SHOULD" is
binding unless a written justification is recorded in the plan.

## Core Principles

### I. Spec-First (NON-NEGOTIABLE)
No implementation code MAY be written for a feature until its `spec.md`, `plan.md`, and
`tasks.md` exist and are approved. Code changes MUST trace to a task in an approved
`tasks.md`. If implementation reveals a spec or plan error, the artifact MUST be updated and
re-approved before the affected code continues.

Rationale: calendar behavior (recurrence, time zones, sharing) is subtle; agreeing on intent
first prevents expensive rework.

### II. Test-First (NON-NEGOTIABLE)
TDD is mandatory: tests are written, run, and seen failing for the expected reason before the
implementation is written (Red-Green-Refactor). Unit line coverage MUST be >= 80% on domain
logic (recurrence, time zones, conflict detection), enforced in CI. Every user story in a
`spec.md` MUST have at least one end-to-end test that exercises it through the real UI/API.

Rationale: domain logic is where calendar bugs hide; end-to-end tests prove stories work as
users experience them.

### III. Correct Time Handling
All instants MUST be stored and transmitted in UTC. Conversion to a user's time zone MUST
happen only at the presentation boundary. Events that recur or are defined in local wall-clock
time MUST additionally store the IANA time zone identifier so DST transitions are handled
correctly. Time zone and recurrence logic MUST use a well-maintained library (see VIII), not
hand-rolled offset arithmetic, and MUST have tests covering DST gaps/overlaps, month ends, and
leap days.

### IV. Type Safety
TypeScript strict mode MUST be enabled end to end (frontend, backend, shared packages, tests,
and tooling configs). `any` and unchecked `@ts-ignore` MUST NOT be used without an inline
justification. Data crossing a trust boundary (HTTP requests, storage, third-party input)
MUST be validated at runtime, and types SHOULD be derived from those schemas or shared
between client and server so contracts cannot drift.

### V. Security
- Every API route MUST require authentication; any public route MUST be explicitly listed and
  justified in its plan.
- Every read or write of calendar data MUST perform an authorization check that the
  authenticated user owns (or has been explicitly granted access to) that calendar. Tests MUST
  cover cross-user access being denied.
- Secrets MUST NOT be committed to the repository; they MUST be supplied via environment or a
  secret manager, and CI MUST include secret scanning.
- Dependencies MUST be scanned for known vulnerabilities in CI; high/critical findings block
  merge.

### VI. Accessibility
Every calendar view (day, week, month, agenda, and any added later) MUST be fully keyboard
operable and screen-reader navigable: logical focus order, visible focus, appropriate
roles/labels/live regions, and no keyboard traps. Each view's spec MUST define its keyboard
interactions, and automated accessibility checks MUST run in the end-to-end suite alongside
keyboard-only tests.

### VII. Performance Budgets
- Initial JavaScript MUST be <= 200 KB gzipped, checked in CI at build time.
- The month view MUST render 500 events in < 100 ms, verified by an automated benchmark with
  a documented environment.
- API p95 latency for event queries MUST be < 300 ms under the documented load profile.

A change that breaches a budget MUST NOT merge; raising a budget is a constitutional amendment.

### VIII. Simplicity
Prefer the platform and well-maintained libraries over custom code. Custom implementations of
solved problems (date math, recurrence rule parsing, auth, validation) MUST be justified in the
plan's complexity-tracking section. Build only what the approved spec requires (YAGNI); new
dependencies MUST be actively maintained, justified, and weighed against the bundle budget.

### IX. Observability
- Logs MUST be structured JSON, with no secrets or unnecessary personal data.
- Every request MUST carry a request ID, propagated through logs and returned in responses.
- A health endpoint MUST be exposed for liveness/readiness checks.
- Unhandled errors MUST be reported to an error-tracking service with the request ID attached.

### X. Delivery
Development is trunk-based: short-lived branches merged to `main` frequently. Pull requests
MUST be small (< 400 changed lines, excluding generated files and lockfiles) and use
Conventional Commits. CI MUST pass lint, typecheck, unit tests, end-to-end tests, and build
before merge; failing checks MUST NOT be bypassed.

## Technology & Quality Constraints

- Language: TypeScript (strict) for all application code.
- The specific framework, database, and tooling choices are made per plan and MUST be
  evaluated against principles IIIâ€“VIII; once chosen they are recorded in the plan.
- CI is the single source of truth for gate results (coverage, bundle size, vulnerability and
  secret scans, accessibility checks, performance benchmarks).

## Development Workflow & Quality Gates

1. Specify â†’ Plan (with Constitution Check) â†’ Tasks â†’ tests written and failing â†’ implement â†’
   CI green â†’ review â†’ merge.
2. Reviewers MUST verify compliance with this constitution; unjustified violations block
   merge.
3. Features larger than the PR limit MUST be split into independently mergeable slices
   (e.g., behind a feature flag) per `tasks.md`.

## Governance

This constitution supersedes all other practices and conventions. Where a conflict exists, this
document wins.

**Amendments**: Any contributor MAY propose an amendment via a PR that edits this file and
includes (a) the rationale, (b) the version bump type, (c) a migration plan for existing
specs, plans, and code affected, and (d) an updated Sync Impact Report during review. An
amendment takes effect only when approved by the project maintainer(s) and merged. Dependent
templates and in-flight feature artifacts MUST be reviewed for consistency within the same PR
or a tracked follow-up.

**Versioning**: The constitution uses semantic versioning (MAJOR.MINOR.PATCH):
- MAJOR: backward-incompatible removal or redefinition of a principle or governance rule.
- MINOR: a new principle or section, or materially expanded guidance.
- PATCH: clarifications, wording, typo fixes, and other non-semantic refinements.

**Compliance**: Every plan MUST pass the Constitution Check below before design work begins
and again after design. Violations are allowed only when recorded with justification in the
plan's complexity-tracking table and approved by a maintainer. Compliance is also verified in
code review.

### Constitution Check (evaluated by /speckit.plan)

A plan MUST answer each item as Pass, N/A (with reason), or Justified Violation:

1. **Spec-First**: Are `spec.md`, `plan.md`, and `tasks.md` approved and are all tasks traceable
   to spec requirements?
2. **Test-First**: Does the plan sequence failing tests before implementation, state how the
   >= 80% domain-logic coverage is met, and map every user story to at least one end-to-end
   test?
3. **Time Handling**: Are all instants stored in UTC, IANA zones stored where wall-clock
   recurrence applies, and DST/edge cases covered by tests?
4. **Type Safety**: Is strict TypeScript used throughout, with runtime validation at trust
   boundaries?
5. **Security**: Does every new route require authN, enforce per-calendar authZ (with
   cross-user denial tests), avoid committed secrets, and keep dependency scanning in CI?
6. **Accessibility**: Are keyboard interactions and screen-reader semantics defined for every
   affected view, with tests?
7. **Performance**: Is the impact on the 200 KB JS, 100 ms/500-event month render, and
   300 ms p95 API budgets assessed and measurable in CI?
8. **Simplicity**: Does it use the platform or maintained libraries, with any custom code or new
   dependency justified?
9. **Observability**: Are structured logs, request IDs, health checks, and error tracking
   covered for new code paths?
10. **Delivery**: Can the work be split into PRs under 400 changed lines using Conventional
    Commits, passing all required CI gates?

**Version**: 1.0.0 | **Ratified**: 2026-11-17 | **Last Amended**: 2026-11-17
