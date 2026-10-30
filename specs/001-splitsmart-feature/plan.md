# Implementation Plan: SplitSmart Java Swing MVP

**Branch**: `001-splitsmart-feature` | **Date**: 2026-10-29 | **Spec**: [spec.md](spec.md)

**Status**: Planning only. No Java implementation is authorized or included in this plan.

## Summary

Plan a Java 21 desktop MVP, built with Maven, that covers the candidate Swing subset in the feature specification: individual accounts, groups, shared expenses with equal or percentage splits and optional receipts, single-level expense review, in-app notifications, pairwise balances, payment recording, and date-filtered summaries. JUnit 5 will verify domain and application behavior independently of Swing; PlantUML will document the planned component, domain, and workflow designs.

**Platform scope is a material mismatch, not a technical synonym**: Java Swing delivers a desktop application. It does **not** deliver the customer's requested primarily web-based product or Android and iOS apps, and this MVP cannot be represented as satisfying those customer requirements. The full requirements inventory in `spec.md` remains authoritative. Product-owner approval of an initial desktop milestone and the platform gap is required before MVP implementation is treated as accepted.

This plan does not choose unresolved product behavior by assumption. In particular, currency/scale and deterministic rounding order, whether a payer must be a group member, account enrollment and recovery, admin delegation mechanics, zero-value shares, approval policy, payment corrections, report parameters, durable storage, and the final supported platform matrix require decisions or explicit implementation approval.

## Technical Context

**Language/Version**: Java 21

**Build**: Maven; standard `src/main/java`, `src/test/java`, and `src/test/resources` layout; compiler configured for Java 21.

**UI**: Java Swing. Views and event listeners are presentation-only; business decisions remain in application/domain layers. UI state changes occur on Swing's Event Dispatch Thread (EDT).

**Testing**: JUnit 5. Domain tests run without Swing; service tests use in-memory test doubles; persistence integration tests depend on the storage decision.

**Diagrams**: PlantUML sources stored under `specs/001-splitsmart-feature/diagrams/`; rendered diagrams are review artifacts and must be kept synchronized with the plan and implementation.

**Storage**: Unresolved. Define repository interfaces and test doubles first. Before a usable MVP is implemented, select durable storage, backup/restore expectations, and receipt-image storage. Do not silently interpret in-memory repositories as satisfying durable account, expense, payment, or receipt records.

**Target Platform**: Desktop JVM running Java 21. This is the MVP implementation target only and does not satisfy the requested web, Android, or iOS targets.

**Project Type**: Single Maven desktop application with layered packages.

**Performance Goals**: No numeric performance targets are specified. The desktop UI must not freeze while potentially blocking persistence or image operations run; response-time and supported dataset thresholds remain open.

**Constraints**:

- All monetary values and allocation calculations use `BigDecimal`; do not use `float` or `double`.
- Set scale and rounding only at explicit currency/allocation boundaries.
- Do not let Swing components or listeners implement splitting, approval, balance, payment, or report rules.
- Use only the single-level approval behavior as the candidate MVP subset; configurable multilevel approval is outside that subset and remains in the customer requirements inventory.
- Do not edit `UserRequirement.md`.
- Human review is required before AI-generated artifacts or implementation are accepted.

**Scale/Scope**: One Java desktop MVP. User count, group size, transaction volume, receipt limits, and concurrent access model are unresolved and must be stated before performance/storage acceptance.

## Constitution Check

| Principle | Plan alignment |
|---|---|
| Customer requirements are authoritative | All original features remain in `spec.md`; this plan labels desktop Swing as an implementation choice and explicitly records its web/mobile gap. |
| Java 21 and Maven | Java 21 and Maven are the selected implementation toolchain. |
| GUI/business logic separation | Swing views call application use cases; domain logic does not depend on Swing. |
| Object-oriented design and PlantUML | Cohesive domain/application/infrastructure responsibilities and reviewed PlantUML diagrams are planned. |
| Exact monetary arithmetic | `BigDecimal`, explicit currency scale, deterministic remainder distribution, and reconciliation tests are required. |
| JUnit 5 and verifiable behavior | Domain, application, repository, and GUI-boundary tests are included, mapped to requirement IDs. |
| Requirement traceability | The traceability matrix below maps spec requirements to planned components and verification. |
| Human review of AI output | Human review is a release gate for the specification, diagrams, tests, and implementation. |

**Gate result**: Aligned, with two explicit blockers to a complete product claim: the Swing/web-mobile platform mismatch and unresolved product/storage decisions below. Neither is silently waived by this plan.

## Architecture

Use a single-process layered design with inward dependencies:

1. **Presentation (`ui`)**: Swing windows, panels, forms, and table models. Collect input, render application state, display validation/errors, and dispatch user intent. No direct repository access or domain calculations.
2. **Application (`application`)**: Use cases coordinate validation, authorization, domain operations, repository transactions, and notifications. Return view-ready results or explicit errors.
3. **Domain (`domain`)**: User, group membership, expense, split, approval, payment, and balance concepts and invariants. No Swing, Maven, or persistence-framework dependencies.
4. **Ports (`application.port`)**: Repository, current-user/authentication, receipt-store, notification, and transaction interfaces required by use cases.
5. **Adapters (`infrastructure`)**: Implement ports for the selected durable storage, password verification, receipt storage, and desktop in-app notification persistence. Exact adapters are not selected until open decisions are resolved.

A composition root constructs adapters and services and launches the Swing UI on the EDT. Services do not hold mutable Swing state. Long-running storage/image tasks execute away from the EDT and return UI updates to the EDT. Domain/application errors are surfaced to the user; no success-shaped fallback or silent swallowing is planned.

### Planned Project Structure

```text
pom.xml
src/
├── main/
│   ├── java/com/splitsmart/
│   │   ├── SplitSmartApplication.java
│   │   ├── config/
│   │   │   └── ApplicationComposition.java
│   │   ├── domain/
│   │   │   ├── user/
│   │   │   ├── group/
│   │   │   ├── expense/
│   │   │   ├── payment/
│   │   │   └── balance/
│   │   ├── application/
│   │   │   ├── account/
│   │   │   ├── groups/
│   │   │   ├── expenses/
│   │   │   ├── approvals/
│   │   │   ├── payments/
│   │   │   ├── reports/
│   │   │   └── port/
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   ├── receipts/
│   │   │   ├── notifications/
│   │   │   └── security/
│   │   └── ui/
│   │       ├── common/
│   │       ├── login/
│   │       ├── groups/
│   │       ├── expenses/
│   │       ├── approvals/
│   │       ├── balances/
│   │       └── reports/
│   └── resources/
├── test/
│   ├── java/com/splitsmart/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   └── ui/
│   └── resources/
└── [generated application data: location and format TBD; not checked in]

specs/001-splitsmart-feature/
├── spec.md
├── plan.md
└── diagrams/
    ├── architecture.puml
    ├── domain-model.puml
    └── expense-approval-balance.puml
```

The package names are a proposed Java organization, not requirements from the customer. Keep persistence-dependent code isolated so the storage choice does not leak into domain types.

### Java Classes and Responsibilities

The following is a planned class inventory, not Java code or a final API contract.

| Area | Planned types | Responsibility |
|---|---|---|
| Bootstrap | `SplitSmartApplication`, `ApplicationComposition` | Start the application, create adapters/services/views, and show the initial screen on the EDT. |
| Identity | `User`, `AccountService`, `AuthenticateUser`, `UserRepository`, `PasswordVerifier` | Represent account identity; create/authenticate accounts and provide the current principal. Account creation, password policy, recovery, and password storage remain to be decided. |
| Groups | `Group`, `GroupMembership`, `GroupRole`, `GroupService`, `CreateGroup`, `ManageMembership`, `GroupRepository` | Represent a group and active/former membership; enforce creator/delegated-admin authority and preserve former-member history. How delegation is granted/revoked is unresolved. |
| Expenses | `Expense`, `ExpenseShare`, `SplitMethod`, `ExpenseStatus`, `ExpenseService`, `SubmitExpense`, `ExpenseRepository`, `SplitAllocator` | Capture the submitted amount, date, description, exactly one payer, participants/shares, optional receipt reference, and review status. Enforce required fields and call the allocator. |
| Approvals | `Approval`, `ApprovalDecision`, `ReviewExpense`, `ApprovalService` | Record an approval decision and change expense status. MVP is one-level only; reviewer eligibility, self-approval, correction, and resubmission policy remain unresolved. |
| Payments | `Payment`, `RecordPayment`, `PaymentService`, `PaymentRepository` | Record positive partial/full settlement between a payer and payee; reject amounts greater than the current pairwise outstanding amount. Reversal/correction policy remains unresolved. |
| Balances | `BalancePosition`, `BalanceService`, `CalculateGroupBalances` | Derive pairwise obligations from approved expenses and recorded payments. Keep balances derived from source records unless storage design later justifies a reconciled cache. |
| Notifications | `Notification`, `NotificationService`, `NotificationRepository`, `InAppNotificationAdapter` | Create and retrieve in-app notices for other current group members on expense submission; do not alter expense approval state. |
| Reports | `ReportCriteria`, `ReportResult`, `GenerateReport` | Query and summarize authorized expenses, balances, and payments for a group/date range and agreed parameters. |
| Shared policies | `CurrencyPolicy`, `MoneyAllocationPolicy`, `DomainValidationException` | Centralize currency precision, rounding, validation, and explicit failure reporting. Their values must come from product decisions, not guesses. |
| Persistence ports/adapters | `UnitOfWork`, repositories per aggregate | Define atomic use-case boundaries and durable storage. Concrete database/file technology, migration strategy, backup, and transaction capabilities are unresolved. |
| UI | `MainFrame`, `LoginPanel`, `GroupListPanel`, `GroupDashboardPanel`, `ExpenseFormPanel`, `ExpenseListPanel`, `ApprovalQueuePanel`, `BalancePanel`, `PaymentDialog`, `ReportPanel`, associated controllers/presenters and table models | Display data, gather input, invoke use cases, show errors/results. UI code performs no split or balance arithmetic. |

Prefer immutable value objects and explicit state transitions where useful. Avoid introducing frameworks or a general-purpose plugin architecture without an approved need.

## Data Model

The domain model should preserve source transactions as the auditable record and derive balances from them.

| Model | Planned information and invariants |
|---|---|
| `User` | Stable ID, display name, email/login identifier, account status. Credentials are handled through a separate security boundary, never stored as plaintext. Enrollment, recovery, and lifecycle fields need product/security decisions. |
| `Group` | Stable ID, name, creator, creation metadata, selected currency policy, membership and group preferences. Whether group currency is fixed or per-expense is unresolved. |
| `GroupMembership` | User ID, group ID, role, active/former state, joined/left timestamps. Former-member records remain available for historical expenses, payments, and balances; former members are not eligible for new expenses. Exact invitation lifecycle is unresolved. |
| `Expense` | Stable ID, group ID, submitter ID, exactly one payer ID, amount, currency, expense date, description, split method, receipt reference (optional), status, creation metadata. Payer may be outside the participant set; whether payer must be a group member is unresolved. |
| `ExpenseShare` | Expense ID, participant user ID, share amount, optional input percentage. Shares are non-negative, expressed at the selected currency scale, and sum exactly to the expense total. Zero-share participation remains unresolved. |
| `Approval` | Expense ID, reviewer ID, decision, timestamp, optional review note, approval stage. The candidate MVP uses a single stage; reviewer eligibility and correction/rejection transitions are unresolved. |
| `Payment` | Stable ID, group ID, payer ID, payee ID, positive amount, currency, payment date, recorder ID, creation metadata. Partial payments are allowed; overpayments are rejected. Correction/reversal behavior remains unresolved. |
| `BalancePosition` | Derived group-scoped debtor, creditor, currency, and net outstanding amount. It is not authoritative independently of expenses/payments; group scoping and netting rules must remain consistent in UI and reports. |
| `Notification` | Recipient user ID, group ID, expense ID, event type, creation time, read state. In the candidate policy, notify other current members on expense creation; status-change notifications remain unresolved. |
| `ReportCriteria` / `ReportResult` | Authorized scope, inclusive date range, selected filters, and summarized expense/payment/balance values with explicit currency. Initial filters and export formats remain unresolved. |

IDs, database schema, deletion/anonymization rules, and retention periods must be settled with the storage and privacy decisions before durable persistence is implemented.

## Expense Splitting and Rounding

The allocator is a pure domain service. It receives a positive total, currency policy, split method, and unique participant set with optional percentages. It returns a share for every participant or a validation error; it never mutates balances.

1. Validate amount is strictly positive, currency is supported, participants are non-empty and valid group members, and split inputs are complete.
2. Use `BigDecimal` constructed from decimal text or exact decimal input, never from binary floating-point values.
3. For equal splits, compute each participant's exact rational share of the total. For percentage splits, require non-negative percentages summing exactly to `100` and multiply the total by each percentage divided by `100` using sufficient intermediate precision.
4. Determine the currency's smallest unit only from the approved currency policy. Round each exact share down to whole minor units as a non-negative allocation base.
5. Compute the number of unallocated minor units as total minor units minus the sum of base share minor units. Assign remainder units one at a time across eligible participants in a deterministic order, cycling only if mathematically needed.
6. Require the final shares to be at the currency scale and sum exactly to the original expense amount. Persist the final shares with the expense so subsequent balance/report calculations use the approved allocation rather than recomputing under a changed policy.
7. Test equal and percentage splits with indivisible totals, repeating fractions, small amounts, many participants, exact 100% totals, and invalid totals.

**Unresolved policy gate**: Supported currencies/scales and the stable ordering used to distribute remainder units are not selected. Candidate orderings (for example, stable participant ID order or group membership order) are not interchangeable product facts; do not implement a final policy until selected.

After approval, the balance service should derive the payer's credit and each participant's debit from the persisted shares, netting a payer's own participant share when the payer is also a participant. The expense payer may be outside the participants, but whether they must be a member of the group is unresolved; define whether/how an outside-group payer is represented before approving balance semantics.

## GUI Design and Separation

Use one `MainFrame` with a clear navigation area and content region. Candidate screens:

- **Login**: email and password form, validation, and explicit authentication errors.
- **Groups**: user's groups, create-group flow, membership management subject to role authorization.
- **Group dashboard**: outstanding balances, recent activity, unread notifications, and navigation to group features.
- **Expenses**: list/detail views and submission form for date, description, amount/currency, payer, participants, split type, percentages when applicable, and optional receipt selection.
- **Approval queue**: pending expenses, receipt/detail review, and approve/reject actions; one-level candidate workflow only.
- **Balances and payments**: pairwise debtor/creditor view and partial/full payment form with overpayment validation.
- **Reports**: expenses, payments, balances, inclusive date range, and only the filters selected for MVP.

Forms perform user-input formatting and present field errors, but authoritative validation is in application/domain services. Controllers translate UI actions into use-case calls and translate results into view state. Repository calls and receipt I/O must not block the EDT; use an appropriate background task mechanism and marshal completion/errors back to the EDT. Swing UI tests assert navigation, validation presentation, and dispatch boundaries, not domain calculations.

PlantUML diagrams to produce and keep reviewed:

- `architecture.puml`: dependency direction from Swing UI to application use cases to domain, with ports and infrastructure adapters.
- `domain-model.puml`: user/group membership, expense/share/approval, payment, notification, and derived balance relationships.
- `expense-approval-balance.puml`: submit → validate → notify → review → approve/reject → apply once → derive balances.

## Testing Strategy

Use Maven Surefire with JUnit 5 for unit tests. Add integration/system test layers only after storage and application boundary choices are settled.

| Test layer | Focus | Evidence |
|---|---|---|
| Domain unit tests | Split validation; equal/percentage allocations; remainder conservation; expense state transitions; pairwise netting; payment constraints; report filters. | Deterministic, independent JUnit tests, including boundary and invalid-input cases. |
| Application service tests | Account authorization, membership rules, expense submission/approval effects, notifications, payment workflows, report scope. | In-memory repositories/test doubles; verify no side effects on rejected use cases and exactly-once approval effects. |
| Persistence integration tests | Repository mapping, transaction boundaries, reload/retention, receipt references, migrations. | Run against the chosen durable adapter; do not treat in-memory tests as persistence evidence. |
| Swing/UI tests | Form validation presentation, navigation, errors, safe EDT interaction. | Headless tests where reliable plus a documented manual acceptance checklist for desktop behavior. |
| Acceptance tests | Spec acceptance scenarios and each FR. | Requirement-ID-tagged tests/checklist; test results reviewed by a human. |
| Diagram/document review | Match PlantUML to actual boundaries, entities, and workflow. | Human-reviewed source diagrams and rendered output. |

Tests must prove share totals exactly equal the expense total, unapproved/rejected expenses have no balance effect, approvals apply once, payments reduce balances only up to the outstanding amount, reports reconcile to included records, and former members cannot enter new expenses while history remains.

## Requirement Traceability

| Requirement(s) | Planned components | Verification |
|---|---|---|
| FR-001–FR-002 | `User`, `AccountService`, authentication ports, login UI | Email/password authentication, authorization/privacy tests; enrollment/recovery details gated by open decisions. |
| FR-003–FR-004, FR-019 | `Group`, `GroupMembership`, `GroupService`, membership use cases/UI | Role/active-member validation and historical-retention tests. |
| FR-005–FR-008 | `Expense`, `ExpenseShare`, `SplitAllocator`, receipt port, expense use cases/UI | Required fields, one payer, participant validation, split conservation, receipt authorization tests. |
| FR-009 | `NotificationService`, notification port/adapter, group dashboard | Other current group members receive an in-app creation notice; verify no approval/balance side effect. |
| FR-010–FR-012 | `ExpenseStatus`, `Approval`, approval use cases | Pending/rejected balance exclusion, authorized decisions, exactly-once approved application. |
| FR-013 | `BalanceService`, `BalancePosition`, balance UI | Pairwise reconciliation against approved shares and payments. |
| FR-014–FR-015, FR-020 | `Payment`, payment use cases, payment repository/UI | Positive partial/full payment accepted; zero, negative, and overpayment rejected without mutation. |
| FR-016–FR-018 | Report use case, criteria/result, report UI | Date/parameter filtering and reconciled report totals. |
| NFR-001 | Separate platform delivery program; not Swing modules | Web, Android, and iOS acceptance matrix; Swing cannot pass this requirement. |
| NFR-002 | `CurrencyPolicy`, `SplitAllocator`, balance service | BigDecimal and minor-unit conservation tests across decided currencies. |
| NFR-003–NFR-005 | UI task boundaries, test suite, review gates | UI responsiveness checks, independent domain tests, human review records. |

The candidate MVP matrix in `spec.md` determines inclusion; this plan does not silently drop customer features. The single-level approval flow, Java desktop UI, and other implementation boundaries must remain labeled as MVP decisions rather than customer requirements.

## Risks and Limitations

| Risk/limitation | Impact | Planned mitigation or gate |
|---|---|---|
| Desktop technology conflicts with requested primary web and Android/iOS apps. | A technically complete Swing MVP still fails the customer's stated platform requirement. | Obtain explicit MVP/scope approval; keep NFR-001 open as unmet until separately delivered and tested. |
| Storage and deployment are unspecified. | Without durable storage, accounts, groups, expenses, receipts, and payments may not survive restart or be safely backed up. | Decide database/file storage, transactions, backup/restore, receipt handling, and deployment before implementing production persistence. |
| Currency and rounding order are unspecified. | Different reasonable policies can produce different participant shares. | Resolve supported currencies, scale, and deterministic participant order before finalizing `SplitAllocator`; test conservation. |
| Payer may be outside participants; payer group membership is unresolved. | Pairwise debt representation may not work if the payer is not part of the group. | Define payer eligibility and group-scoped balance semantics before balance implementation. |
| Approval wording permits a single or multiple review process depending on group needs. | Single-level MVP may not satisfy groups requiring configurable approvals. | Keep multilevel support in the full inventory; confirm scope before claiming requirement completion. |
| Authentication details are incomplete despite email/password choice. | Enrollment, password recovery, password policy, and secure credential storage are needed for usable accounts. | Resolve account lifecycle/security decisions and human-review the security design before account implementation. |
| Group invitation and admin lifecycle are incomplete. | Admin lockout or unauthorized membership changes could impair groups. | Decide invitations, delegation/revocation, creator departure, and group-setting authority. |
| Notification audience/content beyond expense creation is unspecified. | Users may miss important approval/rejection or settlement changes. | Implement only the confirmed creation notification for the MVP; clarify other events before adding them. |
| Payment correction and multiple-payee settlement are unspecified. | Mistakes may be difficult to correct; pairwise settlement semantics can be ambiguous. | Define reversal/audit behavior and payment scope before finalizing payment persistence. |
| Report filters/export and operational quality targets are unspecified. | “Custom reports” and “responsive” cannot be fully acceptance-tested without bounds. | Agree initial filters, export needs, supported scale, latency, accessibility, privacy, and retention criteria. |

## Unresolved Questions and Decision Gates

The following items remain open in `spec.md` and must not be answered by implementation defaults without human/product-owner confirmation:

1. **Platform scope**: Is Swing authorized as an initial desktop milestone? What are delivery milestones and support matrices for primary web, Android, and iOS?
2. **Storage/deployment**: Which durable store, backup/restore model, installation/deployment approach, and receipt-image storage/limits are required?
3. **Currency policy**: Which currencies and scales are supported, and what stable ordering receives remainder minor units?
4. **Payer eligibility**: Must the exactly-one payer be a current group member? How are balances represented if the payer is outside the group?
5. **Zero-share participants**: Are zero percentage allocations allowed, and must every listed participant have a positive share?
6. **Account lifecycle**: How are accounts created, and what password recovery, credential policy, and account closure behavior is required?
7. **Group lifecycle**: Who can invite users or change settings, how are delegated admins granted/revoked, and what happens if a creator leaves with outstanding balances?
8. **Approval policy**: Reviewer selection, self-approval, multilevel configuration, rejection reasons, corrections, and resubmission behavior.
9. **Notification policy**: Which members receive notices and whether approval/rejection/status events also notify.
10. **Payment lifecycle**: How to reverse/correct a recorded payment and whether a payment can settle multiple payees or groups.
11. **Reports and quality targets**: Initial custom filters/export and measurable response-time, availability, accessibility, retention, privacy, and scale targets.

## Complexity Tracking

No constitution violations are proposed. A persistence adapter and layered boundaries are included to isolate unresolved infrastructure choices and keep domain rules independently testable; concrete database selection remains deferred rather than adding an unapproved technology.
