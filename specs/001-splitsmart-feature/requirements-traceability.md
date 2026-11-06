# SplitSmart Requirements-to-Design Traceability

**Date:** 2026-11-05  
**Evidence status:** Proposed verification only; no application code or passing automated acceptance tests are supplied by Lab 7 consolidation.

Sources: [customer requirements](../../UserRequirement.md), [specification](spec.md), [selected architecture](architecture-selected.md), [design](design.md), and [tasks](tasks.md). Customer capabilities, clarified product rules and implementation-derived reliability constraints remain distinguished below. Existing FR/NFR IDs and acceptance criteria are preserved.

Diagram key: [D1 class model](design.md#class-diagram), [D2 submission](design.md#expense-submission-activity), [D3 review](design.md#expense-approvalrejection-activity), [D4 deployment](design.md#desktop-deployment-diagram). G1–G8 refer to [open decisions](architecture-selected.md#open-decision-gates). Table diagram references use this key. Task references are future verification activities, not completed evidence.

| ID | Origin / interpretation | Selected components | Diagrams | Proposed verification / limits |
|---|---|---|---|---|
| FR-001 | Customer account capability; email/password clarified | AccountService, UserRepository, PasswordVerifier | D1, D4 | T015–T021: two accounts, valid/invalid authentication, restart; G5 |
| FR-002 | Customer privacy/access capability; authorization detail specified | CurrentSession, application authorization on every use case | D1–D4 | T020, T038: account/group/receipt isolation; logout during background work |
| FR-003 | Customer groups; creator/admin authority clarified | GroupService, GroupRepository, GroupMembership | D1 | T022–T028: authorized creation/membership, reject unauthorized changes; G5 |
| FR-004 | Specified eligibility detail for customer expense participants | SubmitExpense, GroupMembership | D1, D2 | T022, T029, T038: reject nonmembers/former members at commit |
| FR-005 | Customer expense details; single payer/outside participants clarified | SubmitExpense, Expense, ExpenseShare | D1, D2 | T029–T030, T033, T039: required fields and exactly one payer outside participants; G4 |
| FR-006 | Customer equal/percentage splits | SplitAllocator | D1, D2 | T031–T032: both split methods and indivisible totals; G3 |
| FR-007 | Validation details and clarified positive/exact-percentage rules | SplitAllocator, SubmitExpense, RecordPayment | D1, D2 | T029, T032, T042: zero/negative amount, missing fields/payer, negative/non-100% percentages; no mutation |
| FR-008 | Customer optional receipt | ReceiptStore, Expense receipt reference, authorized retrieval | D1, D2, D4 | T034, T038: optional image persistence/access, staging failure/rollback/cleanup; G2 |
| FR-009 | Customer notification; in-app channel clarified | NotificationRepository, SubmitExpense | D1, D2 | T036, T038: other-member notices once across retry; no approval/balance effect |
| FR-010 | Customer review before adjustment; specified report exclusion | SubmitExpense, ReviewExpense, BalanceService, GenerateReport | D1–D3 | T038–T041, T048: pending/rejected zero financial effect |
| FR-011 | Customer approval; policy still open | ReviewExpense, ApprovalPolicy, Approval | D1, D3 | T035, T038: authorized approve/reject; candidate single-level only; G6 |
| FR-012 | Specified reliability rule for customer approval | ReviewExpense, UnitOfWork, conditional status/version | D1, D3 | T038: repeated/retried/concurrent approval and approve/reject conflict; financial effect once |
| FR-013 | Customer pairwise balances | BalanceService using approved shares and Payment records | D1, D3 | T040–T041, T046–T047: pair/group/currency reconciliation, payer self-share exclusion; G4 |
| FR-014 | Customer settlement; partial payment clarified | RecordPayment, PaymentRepository, UnitOfWork | D1 | T042–T047: positive partial/full settlement, atomic debt validation, response-loss retry |
| FR-015 | Customer payment history/report capability | PaymentRepository, GenerateReport | D1, D4 | T044, T046, T050: retained records after restart and backup restore |
| FR-016 | Customer reports/summaries | GenerateReport, ExpenseRepository, PaymentRepository | D1 | T048–T052: authorized expense/balance/payment summaries |
| FR-017 | Customer custom reporting capability; parameter set open | GenerateReport, ReportCriteria | D1 | T048–T052: inclusive dates and invalid range; filters/export G8 |
| FR-018 | Specified report correctness rule | GenerateReport, BalanceService, consistent repository snapshot | D1 | T048, T052: totals reconcile to matching approved records/payments; period/opening semantics G8 |
| FR-019 | Clarified former-member retention rule | GroupService, historical GroupMembership, repositories | D1, D2 | T022, T027–T028: retained expenses/payments/balances; no new participation; G5 settlement access |
| FR-020 | Clarified overpayment rejection rule | RecordPayment, UnitOfWork, OperationRepository | D1 | T042–T047: excess payment rejected unchanged; concurrent payments cannot jointly overpay |
| NFR-001 | Original customer platform requirement | Separate future platform program; Swing cannot meet it | D4 | UNMET by desktop; T058–T059 and US6/SC-005 need actual web/Android/iOS journeys and synchronization |
| NFR-002 | Constitution/spec monetary correctness constraint | BigDecimal, SplitAllocator, persisted final shares | D1, D2 | T031–T032, T040–T046: exact minor-unit conservation and deterministic policy; G3 |
| NFR-003 | Specified Swing responsiveness constraint | Background application work, presenter/session boundary | D2–D4 | T019, T037, T045, T051: delayed I/O leaves EDT responsive; session switch discards old results; thresholds G8 |
| NFR-004 | Constitution/spec testability constraint | Pure domain, application ports, JUnit 5/Maven | D1 | T015–T055: independent rule tests and actual-adapter integration; compilation is insufficient |
| NFR-005 | Constitution/spec human-review constraint | Human design/code/test review | D1–D4 | T053, T056–T057: artifact, security, arithmetic and platform-claim review before acceptance |

## Acceptance and success criteria

| Criteria | Design and proposed evidence |
|---|---|
| AC-001 | Every FR above has planned component/diagram/test mapping; record future test paths/results and human review here (T014, T055–T057) |
| AC-002 | D2/D3; pending/rejected exclusion, atomic once-only approval, competing review and retry checks (T038–T041) |
| AC-003 / SC-001 | D1/D2; exact share conservation across approved currencies and deterministic remainder cases (T031–T032); G3 open |
| AC-004 / SC-003 | D1/D3; approved allocations minus settlements reconcile by pair/group/currency, including concurrency/retry (T040–T047) |
| AC-005 / SC-004 | D1; reports exclude out-of-range records and reconcile to selected scope (T048–T052); G8 open |
| AC-006 | D4 and release review explicitly state desktop platform gap (T057) |
| SC-002 | D2/D3; all pending/rejected acceptance cases leave balances unchanged (T038–T039) |
| SC-005 | Unmet: separate web/Android/iOS support matrix and core journeys required (T058–T059); G1 open |

## Evidence recording and gates

No row asserts implementation coverage. Add actual test/artifact paths, Maven command results, adapter/version, review date and reviewer when evidence exists. Durable restart/rollback/concurrency/backup claims require the approved adapter, not in-memory doubles. Human review is pending; PlantUML rendering is not claimed. Requirements blocked by G1–G8 remain blocked for their affected implementation or product acceptance, even though the architecture has been selected for design.
