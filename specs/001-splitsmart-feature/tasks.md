# Tasks: SplitSmart Java Swing MVP

**Input**: Design documents in `specs/001-splitsmart-feature/` (`spec.md`, `plan.md`) and `.specify/memory/constitution.md`

**Scope**: Action plan for the candidate Java 21 desktop MVP only. The original customer request for a primarily web-based product plus Android and iOS apps remains authoritative; Swing is a desktop implementation choice and does not satisfy those platform requirements. Future-platform work is listed separately and is not included in the MVP.

**Implementation status**: Planning only. These tasks describe future work; no Java implementation is included in this file.

**Task format**: `[ ] Tnnn [P?] [Story] Action, paths, requirement IDs, and verification`

- `[P]` means the task can proceed in parallel with other marked tasks once its listed dependencies are met and it does not edit the same files.
- `Depends on` references task IDs that must be complete first.
- Story references: **US1** account; **US2** groups; **US3** expenses/approvals; **US4** balances/payments; **US5** reports; **US6** requested web/mobile platforms.

## Phase 0: Scope and Product Decision Gates

**Purpose**: Confirm the desktop MVP boundary and resolve only the product decisions that block safe implementation. Do not silently substitute defaults.

- [ ] T001 Obtain and record human/product-owner approval for a Java Swing desktop MVP as an initial milestone, documenting that it does not satisfy the customer's primary web, Android, or iOS requirements; retain those requirements as unmet until separately delivered. Verify the decision is reflected in `specs/001-splitsmart-feature/spec.md` without editing `UserRequirement.md`. (NFR-001, AC-006)
- [ ] T002 Select the durable storage, transaction, backup/restore, receipt-storage, and desktop distribution approach; record approved decisions in `specs/001-splitsmart-feature/plan.md` before implementing persistence. (FR-001–FR-018)
- [ ] T003 Confirm supported currencies, currency precision, and the stable participant ordering used to distribute minor-unit remainders; record decisions in `specs/001-splitsmart-feature/spec.md` before finalizing allocation behavior. (NFR-002, AC-003)
- [ ] T004 Decide whether the expense payer must be a current group member and define how an outside-group payer is represented in pairwise group balances; update the spec before implementing balance effects. (FR-005, FR-013)
- [ ] T005 Confirm account enrollment, password policy/recovery, and group invitation/admin delegation and creator-departure behavior; record product decisions in the spec before completing account or membership workflows. (FR-001–FR-003, FR-019)

**Gate**: T001 is required before representing the Swing work as an approved MVP. T002–T005 are prerequisites for the affected persistence, allocation, balance, account, and membership tasks; unrelated scaffolding can proceed without resolving every gate.

## Phase 1: Setup and Shared Foundation

**Purpose**: Establish a reproducible Java 21/Maven project, its test foundation, architecture boundaries, and shared domain/application infrastructure.

- [ ] T006 Create the Maven single-project directory structure from `plan.md`, including `src/main/java/com/splitsmart/`, `src/main/resources/`, `src/test/java/com/splitsmart/`, `src/test/resources/`, and `specs/001-splitsmart-feature/diagrams/`. Verify the directories and package layout match the plan.
- [ ] T007 Configure `pom.xml` for Java 21 compilation, JUnit 5, and Maven Surefire; add no unapproved persistence or UI dependencies. Verify `mvn test` discovers a minimal JUnit 5 test and the compiler targets Java 21. Depends on T006.
- [ ] T008 [P] Add project-level usage/build guidance to `README.md`, documenting the Java 21/Maven prerequisites, build/test commands, desktop-only scope, and explicit unmet web/mobile requirements. Verify commands and platform claims against `spec.md`. Depends on T001, T007.
- [ ] T009 [P] Create `specs/001-splitsmart-feature/diagrams/architecture.puml` to show UI → application → domain dependency direction and infrastructure ports/adapters. Verify the diagram renders and agrees with `plan.md`. Depends on T006.
- [ ] T010 [P] Create `specs/001-splitsmart-feature/diagrams/domain-model.puml` for users, group membership, expenses/shares/approvals, payments, notifications, and derived balances. Verify cardinalities and historical membership semantics against `spec.md`. Depends on T003, T004, T006.
- [ ] T011 [P] Create `specs/001-splitsmart-feature/diagrams/expense-approval-balance.puml` for submit, validate, notify, review, approve/reject, and exactly-once balance effect. Verify pending/rejected paths have no balance effect. Depends on T006.
- [ ] T012 Establish base packages and application composition/bootstrap boundaries under `src/main/java/com/splitsmart/` (`domain`, `application`, `infrastructure`, `ui`, `config`) without embedding domain rules in Swing. Verify dependency direction through package-level review. Depends on T006, T007, T009.
- [ ] T013 Define shared domain error/validation conventions and application port interfaces for transactions, current-user access, and repositories under `src/main/java/com/splitsmart/application/port/`. Verify invalid operations return explicit errors and do not produce success-shaped fallbacks. Depends on T002, T012.
- [ ] T014 [P] Add a requirement-to-code/test traceability checklist to `specs/001-splitsmart-feature/README.md` or the project `README.md` (choose one canonical location) mapping every accepted FR/AC to planned component and evidence. Keep unsupported/deferred platform scope visibly separate. Verify all IDs in `spec.md` appear. Depends on T006.

## Phase 2: User Story 1 - Access a Personal Account (Priority: P1)

**Goal**: Authenticate an individual account with email and password and restrict personal/group data access.

**Independent test**: Create two test accounts, authenticate separately, verify each can access their own information and unauthorized account/group information is denied.

**Requirements**: FR-001–FR-002.

- [ ] T015 [P] [US1] Write JUnit 5 domain tests for user identity, account status, and email identifier validation in `src/test/java/com/splitsmart/domain/user/`; include valid, malformed, missing, and duplicate identity cases according to approved enrollment rules. (FR-001)
- [ ] T016 [P] [US1] Define the `User` domain model and account status in `src/main/java/com/splitsmart/domain/user/`; avoid storing raw credentials on the domain entity. Verify invariants with T015. (FR-001–FR-002)
- [ ] T017 [US1] Define `UserRepository`, `PasswordVerifier`, and the account use cases/services in `src/main/java/com/splitsmart/application/`; implement email/password authentication and explicit authorization failures. Depends on T005, T013, T016. (FR-001–FR-002)
- [ ] T018 [US1] Implement the selected credential/persistence adapters in `src/main/java/com/splitsmart/infrastructure/security/` and `src/main/java/com/splitsmart/infrastructure/persistence/`; use the storage/security decisions from T002 and T005. Verify account reload and that credentials are not persisted as plaintext. (FR-001–FR-002)
- [ ] T019 [P] [US1] Build `LoginPanel` and account/session presentation under `src/main/java/com/splitsmart/ui/login/`; collect email/password, show explicit authentication errors, and invoke account use cases only. Verify Swing updates occur on the EDT. Depends on T012, T017.
- [ ] T020 [US1] Add application tests for valid/invalid authentication and attempts to access another user's account or unauthorized group data in `src/test/java/com/splitsmart/application/account/`. Verify no private information is returned after denial. Depends on T017.
- [ ] T021 [US1] Run the US1 acceptance test with two accounts through the Swing flow and document results in the traceability checklist. Depends on T018–T020. (FR-001–FR-002)

## Phase 3: User Story 2 - Organize Participants in a Group (Priority: P1)

**Goal**: Create groups, manage membership with creator/delegated-admin authority, and retain former members' history.

**Independent test**: Create a group, add members, remove a member, verify unauthorized membership changes fail, historical records remain, and the former member cannot join new expenses.

**Requirements**: FR-003–FR-004, FR-019.

- [ ] T022 [P] [US2] Write JUnit 5 tests for group membership status, creator/admin permissions, active-member eligibility, and historical former-member retention in `src/test/java/com/splitsmart/domain/group/`. (FR-003–FR-004, FR-019)
- [ ] T023 [P] [US2] Define `Group`, `GroupMembership`, and `GroupRole` domain types in `src/main/java/com/splitsmart/domain/group/`; distinguish active and former membership without deleting history. Verify against T022. (FR-003–FR-004, FR-019)
- [ ] T024 [US2] Implement group and membership repository ports/adapters and `CreateGroup`/`ManageMembership` use cases in `src/main/java/com/splitsmart/application/groups/` and `src/main/java/com/splitsmart/infrastructure/persistence/`; enforce creator/delegated-admin permissions. Depends on T002, T005, T013, T023.
- [ ] T025 [US2] Implement invitations/admin delegation lifecycle and preserve expenses, payments, and balances when membership ends, using the decisions from T005. Verify unauthorized changes are rejected without mutation. Depends on T024. (FR-003–FR-004, FR-019)
- [ ] T026 [P] [US2] Build group list, create-group, and membership-management screens in `src/main/java/com/splitsmart/ui/groups/`; route all changes through use cases. Verify role-specific controls and explicit errors. Depends on T012, T024.
- [ ] T027 [US2] Add application/integration tests for group creation, membership changes, active-member checks, admin authorization, and former-member history retention in `src/test/java/com/splitsmart/application/groups/`. Depends on T024–T025. (FR-003–FR-004, FR-019)
- [ ] T028 [US2] Run the US2 membership acceptance scenarios and record evidence in the traceability checklist. Depends on T025–T027.

## Phase 4: User Story 3 - Submit and Review a Shared Expense (Priority: P1)

**Goal**: Record complete expense details and exact shares, notify group members, and apply financial effects only after required approval.

**Independent test**: Submit valid equal and percentage expenses, including one payer outside the participant set; verify notification and pending status, approve and verify exactly-once balance effect, and reject invalid submissions.

**Requirements**: FR-005–FR-012; NFR-002.

- [ ] T029 [P] [US3] Write JUnit 5 tests for positive amount/required-field validation, exactly one payer, participant eligibility, percentage non-negativity/exact 100% total, and zero/negative inputs in `src/test/java/com/splitsmart/domain/expense/`. (FR-005–FR-007)
- [ ] T030 [P] [US3] Define `Expense`, `ExpenseShare`, `SplitMethod`, `ExpenseStatus`, and receipt-reference value types in `src/main/java/com/splitsmart/domain/expense/`; include one payer and persisted final shares. Verify invariants with T029. (FR-005–FR-008)
- [ ] T031 [US3] Implement pure `SplitAllocator` using `BigDecimal` in `src/main/java/com/splitsmart/domain/expense/`; support equal and percentage splits, approved currency scale, and deterministic remainder ordering from T003. Verify share sum exactly equals total for each supported currency and invalid input fails explicitly. Depends on T003, T029–T030. (FR-006–FR-007, NFR-002, AC-003)
- [ ] T032 [P] [US3] Write JUnit 5 allocator tests for indivisible equal splits, percentage splits, smallest amounts, repeating fractions, many participants, exact minor-unit conservation, and invalid percentages in `src/test/java/com/splitsmart/domain/expense/`. Tests must cover each currency selected in T003. (FR-006–FR-007, NFR-002)
- [ ] T033 [US3] Implement `SubmitExpense` and expense repository/transaction integration in `src/main/java/com/splitsmart/application/expenses/` and `src/main/java/com/splitsmart/infrastructure/persistence/`; validate active participants, payer eligibility decision, receipt reference, and pending initial state. Depends on T002, T013, T025, T030–T032. (FR-005–FR-008, FR-010)
- [ ] T034 [P] [US3] Implement receipt selection/storage adapter in `src/main/java/com/splitsmart/infrastructure/receipts/`; enforce approved format/size/access rules once decided. Keep image I/O out of the EDT. Verify unauthorized members cannot retrieve a receipt. Depends on T002, T013.
- [ ] T035 [US3] Implement single-level `Approval`, `ApprovalDecision`, and `ReviewExpense` in `src/main/java/com/splitsmart/domain/expense/` and `src/main/java/com/splitsmart/application/approvals/`; preserve pending/rejected balance exclusion and exactly-once approval effects. Depends on T030, T033.
- [ ] T036 [P] [US3] Implement in-app notification model, repository adapter, and `NotificationService` in `src/main/java/com/splitsmart/application/` and `src/main/java/com/splitsmart/infrastructure/notifications/`; notify other current group members on expense submission only. Verify notification does not approve or apply balances. Depends on T025, T033.
- [ ] T037 [P] [US3] Build expense entry, detail/list, approval queue, and in-app notification views under `src/main/java/com/splitsmart/ui/expenses/`, `src/main/java/com/splitsmart/ui/approvals/`, and `src/main/java/com/splitsmart/ui/common/`; call use cases and show validation/review outcomes. Verify no business calculations occur in UI. Depends on T012, T033, T035–T036.
- [ ] T038 [US3] Add application tests for submission, receipt authorization, notification audience, approve/reject lifecycle, repeated approval, and unchanged balances before approval in `src/test/java/com/splitsmart/application/expenses/` and `src/test/java/com/splitsmart/application/approvals/`. Depends on T033–T036. (FR-008–FR-012, AC-002)
- [ ] T039 [US3] Run the US3 acceptance scenarios using the selected storage/receipt adapter; verify shares reconcile, notifications reach other current members, and approval affects balances once. Record evidence. Depends on T034–T038.

## Phase 5: User Story 4 - Check Balances and Record Settlement (Priority: P1)

**Goal**: Derive pairwise obligations from approved shares and record positive partial/full payments without overpayment.

**Independent test**: Approve expense fixtures, verify pairwise balances, record partial and full payments, then verify overpayment and invalid amounts leave records/balances unchanged.

**Requirements**: FR-013–FR-015, FR-020; NFR-002; AC-002–AC-004.

- [ ] T040 [P] [US4] Write JUnit 5 tests for pairwise debit/credit derivation, payer-participant netting, payer-outside-participants behavior, group isolation, and currency consistency in `src/test/java/com/splitsmart/domain/balance/`. Depends on T004, T031.
- [ ] T041 [US4] Implement `BalanceService` and derived `BalancePosition` calculations in `src/main/java/com/splitsmart/domain/balance/`; use persisted approved shares and recorded payments, support group-scoped balances, and reconcile exactly. Depends on T004, T031, T035, T040. (FR-013, AC-002–AC-004)
- [ ] T042 [P] [US4] Write JUnit 5 tests for positive partial/full payments, exact-balance payments, zero/negative values, overpayment rejection, wrong-group parties, and unchanged state after rejection in `src/test/java/com/splitsmart/domain/payment/`. (FR-014–FR-015, FR-020)
- [ ] T043 [US4] Define `Payment` and implement `RecordPayment`/`PaymentService` with atomic persistence and current-balance validation under `src/main/java/com/splitsmart/domain/payment/` and `src/main/java/com/splitsmart/application/payments/`. Depends on T002, T013, T041–T042. (FR-014–FR-015, FR-020)
- [ ] T044 [US4] Implement payment repository and ensure recorded payments remain auditable and survive reload in `src/main/java/com/splitsmart/infrastructure/persistence/`; verify the operation and balance update are atomic. Depends on T002, T043. (FR-015)
- [ ] T045 [P] [US4] Build pairwise balance view and payment dialog in `src/main/java/com/splitsmart/ui/balances/`; present outstanding amount, accept partial/full payments, and display overpayment/invalid-amount errors. Verify form does not compute balances. Depends on T041, T043.
- [ ] T046 [US4] Add application/integration tests that reconcile approved expenses minus payments by user pair/group and test concurrent or repeated payment submission according to the selected storage transaction model. Depends on T043–T044. (FR-013–FR-015, FR-020, AC-004)
- [ ] T047 [US4] Run the US4 acceptance scenarios and record pairwise reconciliation and rejected-payment evidence in the traceability checklist. Depends on T041–T046.

## Phase 6: User Story 5 - Review Activity Reports (Priority: P2)

**Goal**: Produce authorized expense, balance, and payment summaries with an inclusive date range and approved filters.

**Independent test**: Use records inside/outside a selected range; verify only matching activity is reported and the totals reconcile with source records.

**Requirements**: FR-016–FR-018.

- [ ] T048 [P] [US5] Write JUnit 5 tests for inclusive date boundaries, invalid ranges, authorization/scope isolation, expense/payment summaries, and reconciliation in `src/test/java/com/splitsmart/application/reports/`. (FR-016–FR-018, AC-005)
- [ ] T049 [US5] Confirm initial report parameters beyond date range with the product owner; update `spec.md` before implementing unconfirmed filters or export formats. (FR-017)
- [ ] T050 [US5] Implement `ReportCriteria`, `ReportResult`, and `GenerateReport` in `src/main/java/com/splitsmart/application/reports/`; query approved expenses, balances, and payments within the authorized inclusive date range and confirmed filters. Depends on T041, T044, T049. (FR-016–FR-018)
- [ ] T051 [P] [US5] Build the report screen in `src/main/java/com/splitsmart/ui/reports/`; allow date-range and confirmed parameter selection, surface invalid ranges, and display reconciled summaries. Depends on T050.
- [ ] T052 [US5] Run report acceptance tests against persisted fixtures and verify totals reconcile with included records; record FR-016–FR-018 evidence. Depends on T048–T051.

## Phase 7: Cross-Story Integration and MVP Verification

**Purpose**: Verify the full desktop subset, keep architecture/documentation aligned, and ensure the MVP is never misrepresented as the complete customer product.

- [ ] T053 [P] Review and update all PlantUML sources in `specs/001-splitsmart-feature/diagrams/` against the implemented package boundaries, data model, and approval/balance workflow; render them and record human review. (Constitution IV, VIII)
- [ ] T054 Update `specs/001-splitsmart-feature/spec.md` and the traceability checklist with accepted implementation decisions and test evidence; preserve the distinction between customer-stated requirements and MVP decisions. Do not edit `UserRequirement.md`. (Constitution I, VII)
- [ ] T055 Run the complete Maven test suite, verify Java 21 compilation, and resolve failures in the changed scope. Record command/results in the project documentation; compilation alone is not acceptance evidence. (NFR-004, AC-001)
- [ ] T056 Perform a human review of account/privacy boundaries, expense and payment arithmetic, persistence/receipt access, Swing EDT behavior, PlantUML consistency, and requirement traceability before accepting the MVP. (Constitution VIII)
- [ ] T057 Verify all candidate MVP user-story acceptance scenarios and AC-001–AC-006; mark unmet customer platforms and unresolved scope visibly in the release notes/README. Do not claim complete SplitSmart delivery while NFR-001 is unmet. (FR-001–FR-020, NFR-001, AC-001–AC-006)

## Future Platform Work - Not Part of the Java Swing MVP

These are separate discovery/delivery tracks for the customer's platform requirements. They are intentionally not Swing implementation tasks and must not be treated as completed by this MVP.

- [ ] T058 [FUTURE] Obtain product approval, platform-specific architecture, support matrices, shared-data synchronization, security, and release criteria for the primary web product and Android/iOS applications. Map the agreed work back to NFR-001 and User Story 6.
- [ ] T059 [FUTURE] Create separately scoped implementation plans and acceptance tasks for the primary web product, Android application, and iOS application after T058; verify core journeys and consistent account/group/expense/approval/balance/payment/report data on each supported platform. (NFR-001, User Story 6, SC-005)

## Dependencies and Parallel Execution

### Phase Dependencies

- **Phase 0**: Product/scope decisions. T001 gates acceptance of the Swing MVP. T002–T005 gate their affected implementation topics.
- **Phase 1**: Project foundation; T006 precedes Maven setup, packages, and diagrams. T007 and T012 unblock implementation stories. The three initial diagrams can be authored in parallel once their stated dependencies are met.
- **US1 and US2**: Start after shared foundation. Their domain tests/models can proceed in parallel after T012; account and group use cases depend on product decisions and repository ports.
- **US3**: Depends on active group membership from US2 and currency/remainder decisions from T003. UI, receipts, and notification adapter can proceed in parallel after their interfaces are agreed; allocator tests are separate from implementation files.
- **US4**: Depends on expense approval/share persistence from US3 and payer/balance semantics from T004. Payment tests can be authored in parallel with balance tests.
- **US5**: Depends on balance and payment read models from US4 plus the confirmed report filter set.
- **Integration/polish**: Depends on the MVP user stories selected for delivery; human review and explicit platform gap acknowledgement are release gates.
- **Future platforms**: Discovery task T058 is separate. Web/mobile implementation planning depends on its decisions and is not a dependency that turns Swing into platform coverage.

### Parallel Opportunities

- After T006, T009, T010, and T011 may be authored in parallel, subject to their currency/payer inputs; T008 can proceed once scope is approved and Maven exists.
- US1 and US2 test/model work may run in parallel after the base project structure; do not parallelize edits to shared repository interfaces without coordination.
- In US3, domain validation tests (T029), model design (T030), notification work (T036), and UI scaffolding (T037) can be divided by separate files once interfaces are stable; allocator tests (T032) should remain independent of allocator implementation (T031).
- In US4, balance tests and payment tests (T040, T042) are parallel; UI (T045) follows the use-case contract and can proceed separately from persistence integration after those contracts are stable.
- Report tests, report UI, and cross-cutting diagram review can be parallelized only after the referenced service/view contracts are stable.

## MVP Boundary and Exclusions

- **MVP**: Java 21 + Maven desktop application using Swing; candidate user stories US1–US5, subject to explicit decision gates and the approved MVP subset in `spec.md`.
- **Not covered by Swing**: The customer's primarily web-based product, Android app, and iOS app (NFR-001 / US6). These remain future separately planned deliverables.
- **Not silently included**: Configurable multi-level approvals, unspecified currency choices, unapproved storage/security/deployment decisions, unresolved payer membership semantics, payment corrections, and unspecified reporting filters. Resolve decisions before implementing affected behavior.
- `UserRequirement.md` is read-only and remains the authoritative customer request.
