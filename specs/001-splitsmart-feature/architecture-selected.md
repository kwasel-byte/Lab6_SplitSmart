# SplitSmart Selected Architecture — CIS 285 Lab 7

**Decision date:** 2026-11-05  
**Status:** Selected for documentation/design; implementation and desktop product-scope acceptance are not implied.

## Decision and sources

Select the [Spec Kit layered baseline](architecture-baseline.md), consistent with [plan.md](plan.md), the [architecture review](Lab7_Architecture_Review.md), and [consolidation instructions](Lab7_Consolidation_Instructions.md). Preserve the [alternative](architecture-alternative.md) as comparison evidence, rejected for now. The original [customer request](../../UserRequirement.md), [constitution](../../.specify/memory/constitution.md), and [specification](spec.md) retain their authority. See [design.md](design.md) for diagrams and [requirements-traceability.md](requirements-traceability.md) for proposed verification.

Use Java 21, Maven, Swing, JUnit 5, PlantUML, and `BigDecimal`. No application code is supplied by this consolidation.

## Style and responsibilities

A single-process layered application has inward dependencies: presentation invokes application use cases; application coordinates domain objects through application-owned ports; infrastructure implements those ports. Bootstrap wires concrete adapters. Domain classes depend on neither Swing nor storage technology.

| Boundary | Responsibility |
|---|---|
| `ui` | Swing views, presenters, table models, immutable request/result objects; EDT-only widget updates |
| `application` | Account/group/expense/review/payment/report use cases; current-principal authorization and transaction coordination |
| `domain` | Users, historical membership, expense shares/status, approval, payment, pure allocation and derived balances |
| `application.port` | Repositories, current session, credential verification, receipt storage, durable UnitOfWork and operation deduplication contracts |
| `infrastructure` | Approved durable store, transactions, credential/receipt adapters, persisted in-app notices |
| `config` | Composition root and application launch; no business rules |

Expense, final share, approval, membership, and payment records are authoritative. Balances and report results are calculated from these records through ordinary application services. There is no separate write/read storage model, financial projection pipeline, event journal, replay system, or event-schema lifecycle.

## Reliability improvements adopted for design

- Approval status and review record commit atomically using a conditional pending-state/version transition. A conflicting or repeated review cannot add another financial effect. Derived balances include each approved expense once.
- Payment eligibility is checked against current authorized debt inside the same isolated transaction as insertion. Competing payments must not jointly overpay; the adapter must supply suitable locking, serializable isolation, or equivalent conditional coordination covering relevant expense/review/payment writes.
- A durable operation key scoped to actor and operation, with an input fingerprint and result reference, protects submission/payment retries after a lost response. Store it atomically with the business records. Reuse with changed input fails; returning a prior result still requires current authorization. This is a repository consistency mechanism, not CQRS.
- Retain expense, approval, payment and former-member records with actor/time metadata for audit and restart verification. Audit records document business operations; they are not events used to reconstruct application state.
- Keep persistence and receipt I/O off the Swing EDT. Bind background requests/results to the initiating session; discard obsolete results after logout/account changes. Application authorization remains authoritative.
- Link requirements to components, diagrams, planned JUnit 5 checks and later evidence. Test rollback, concurrency, retries, restart, receipt privacy, and backup restoration against the actual selected adapter.

## Persistence and data flow

Swing input → background use case → current-principal and domain validation → repository operations in UnitOfWork → commit → immutable result → session-checked EDT update. Failed validation or persistence returns an explicit error and no partial financial records. Read services use a consistent authorized scope/snapshot of the same authoritative records.

Submission persists a pending expense, exact final shares, optional receipt reference and other-member creation notices. Pending/rejected expenses contribute zero to balances. Review atomically records approve/reject and changes status. Payment atomically validates and records positive partial/full settlement, rejecting overpayment. Reports reconcile approved shares and recorded payments within the agreed dates/scope.

No database is selected. Evaluate SQLite/JDBC alongside alternatives against transactions, concurrency, deployment, migrations, recovery and security; the alternative's SQLite and BLOB proposal is not approved by this decision. In-memory repositories are test doubles, not durable storage. Receipt bytes may share the database transaction or need an approved staging/cleanup protocol. Backups must cover authoritative data, credentials, operation keys and receipt bytes/references consistently.

## Rationale and trade-offs

The layered baseline fits the existing Spec Kit plan/tasks and course scope, keeps domain testing straightforward, and avoids coordinating duplicate financial state. The alternative offers stronger transition history/replay but adds schema evolution, replay, projections, migration and privacy complexity without a confirmed requirement. Therefore event sourcing and CQRS are not selected.

Derived balances may cost more to calculate for large histories; supported dataset size and latency must be agreed and measured before considering optimizations. Conventional audit records do not provide arbitrary historical state replay. Transactions, idempotency and durable storage still require careful implementation and integration tests; architectural selection alone proves none of them.

## Open-decision gates

All gates below remain **open**. Architecture selection does not waive them.

| Gate | Decision needed before affected implementation/acceptance | Existing task reference |
|---|---|---|
| G1 | Desktop-only milestone acceptance, separate web/mobile milestones and support matrix | T001, T058–T059 |
| G2 | Durable technology/location, concurrency/isolation, receipt limits/atomicity, migrations, backup/restore, distribution | T002 |
| G3 | Supported currencies, scale, excess-precision treatment, deterministic remainder order and zero shares | T003; spec open questions |
| G4 | Whether payer must be a group member; outside-group balance representation | T004 |
| G5 | Enrollment/recovery/password handling, invitation/admin delegation/creator departure, former-member settlement access | T005 |
| G6 | Reviewer eligibility/self-approval, single/multiple levels, rejection/correction/resubmission policy | FR-011; spec open questions |
| G7 | Notification audience details/additional events, payment correction/reversal, multi-payee/group settlements | Spec open questions |
| G8 | Report filters/export and date/balance labels, quality/accessibility/security/privacy/retention/scale criteria | T049; spec open questions |

Exactly one payer (possibly outside participants), positive amounts, exact 100% percentage totals, retained former-member history, in-app creation notices, pre-approval balance exclusion, partial payments and overpayment rejection remain unchanged.

## Deployment and acceptance boundary

The delivery target is a desktop JVM with Swing. Runtime packaging, desktop OS matrix, local versus central storage and multi-user access are unresolved under G1/G2. The design does not supply a hosted service or synchronization.

**The original primary web, Android and iOS requirement is unmet by the Swing milestone (NFR-001 / US6 / SC-005).** Swing cannot deliver those clients; shared identity/data, APIs, cross-platform consistency and platform acceptance require separately scoped work. Customer requirements are preserved, not reclassified as satisfied or removed.

Human review of documents and future code/tests remains required. Proposed verification is not passing test evidence; diagram rendering must be recorded separately when actually performed.
