# CIS 285 Lab 7 — SplitSmart Architecture Review

**Review date:** November 5, 2026  
**Reviewed artifacts:** `architecture-baseline.md` (Spec Kit / Copilot) and `architecture-alternative.md` (Codex)  
**Decision:** Select the **Spec Kit layered baseline**, with targeted reliability improvements drawn from the alternative. This is an architecture recommendation for the classroom desktop MVP, not approval of unresolved product requirements.

## 1. Architecture summaries

**Baseline — layered, dependency-inward desktop application.** A Java 21/Maven/Swing application separates `ui`, `application`, `domain`, application ports, infrastructure adapters, and bootstrap. Expense and payment records are authoritative; balances are derived from approved expense shares and payments. The durable database, receipt store, and deployment details remain undecided. JUnit 5 tests target domain rules, use cases, persistence integration, and Swing boundaries.

**Alternative — event-journal modular monolith.** A Java 21/Maven/Swing application uses business modules, CQRS, a versioned append-only group event journal, and synchronous query projections. It proposes SQLite/JDBC, command deduplication, stream concurrency checks, projection rebuilds, and transactional receipt storage. Accounts and credentials use conventional storage rather than event sourcing.

## 2. Comparison

| Criterion | Baseline | Alternative | Review |
|---|---|---|---|
| Fit to existing Spec Kit plan | Direct alignment with plan, tasks, and package structure | Requires redesign around commands, events, and projections | Baseline favored |
| Implementation complexity | Lower; conventional use cases and repositories | Higher; event schemas, replay, projection synchronization | Baseline favored for course scope |
| Financial correctness | Exact `BigDecimal` shares, atomic approval/payment requirements, balances derived from sources | Same rules plus transactional projections, command IDs, stream-version checks | Both viable if implemented/tested correctly |
| Audit and recovery | Preserved expense, approval, and payment records | Append-only event history, replayable projections | Alternative stronger, with additional cost |
| Persistence readiness | Store and receipt adapter not yet selected | Concrete SQLite proposal, but still needs approval | Alternative more specific, not automatically approved |
| UI and deployment | Swing desktop only | Swing desktop only | Both miss original web/Android/iOS scope |
| Testability | Pure domain and use-case tests with JUnit 5 | Same plus event replay/rebuild/integration tests | Baseline easier to verify within lab scope |
| Unresolved product decisions | Explicitly preserved | Explicitly preserved, plus event/privacy decisions | Neither is implementation-ready without decisions |

## 3. Decision and rationale

**Select the baseline layered architecture** as the primary architecture for the proposed desktop milestone. It satisfies the lab's architecture/design-document goal while minimizing implementation and verification burden, and it stays closest to the already generated Spec Kit plan and tasks. Its derived-balance model avoids maintaining an additional financial projection as an independent source of truth.

**Adopt limited ideas from the alternative without adopting event sourcing:**
1. Use an approved durable transaction-capable store; evaluate SQLite/JDBC as an option, not a foregone selection.
2. Make approval transitions and payment validation atomic, repeat-safe, and concurrency-safe; test retries and conflicting actions.
3. Retain auditable expense, approval, and payment records, and test recovery/restart behavior.
4. Keep I/O off Swing's event-dispatch thread, and verify session-bound authorization for asynchronous results.
5. Use explicit requirement-to-design and requirement-to-test traceability.

Do **not** add an event journal, CQRS projections, replay machinery, or event-schema versioning unless new project requirements justify them.

## 4. Risks and decision gates

- **Platform gap:** The original request includes a primarily web-based application plus Android/iOS; neither Swing architecture meets NFR-001 / US6 / SC-005. Obtain explicit acceptance for a desktop-only milestone or plan a separate web/mobile architecture.
- **Storage and recovery:** Decide the durable store, transaction isolation, receipt handling, backup/restore, and migration strategy before implementing financial workflows.
- **Business policies:** Resolve supported currencies, precision/remainder order, payer eligibility, zero shares, approval authority/self-approval, correction flows, account lifecycle, and report/export scope.
- **Security and quality:** Define password handling, access controls, receipt privacy, retention, responsiveness, and acceptance criteria.
- **Diagrams:** PlantUML source blocks are present, but successful visual rendering was not demonstrated by the uploaded files.

## 5. Acceptance and verification plan

Confirm that expense shares sum exactly to the original amount; pending/rejected expenses never affect balances; approval changes financial effects once; positive partial payments reduce only the authorized debt; overpayments and unauthorized actions leave records unchanged; reports reconcile with approved source records; former members retain history without new participation; and UI updates occur on the Swing event-dispatch thread. Use JUnit 5 unit and integration tests once a durable adapter is chosen.

## 6. Conclusion

The baseline is the recommended course-project architecture. The alternative is a credible but more expensive design for stronger audit/replay requirements. Preserve both proposals and this review in GitHub as evidence of comparison and decision-making. This review is a design recommendation, not proof that application code or tests have been implemented.
