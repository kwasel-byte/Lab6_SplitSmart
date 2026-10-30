# Feature Specification: SplitSmart Expense Sharing

**Feature Branch**: `001-splitsmart-feature`

**Created**: 2026-10-29

**Status**: Draft

**Input**: User description: "Read UserRequirement.md in this workspace as the authoritative customer requirements. Write a testable SplitSmart feature specification: actors, user scenarios, functional and non-functional requirements, acceptance criteria, requirement IDs (FR-001, etc.), and open questions. Preserve all original customer features in the requirements inventory; label the Java Swing MVP subset separately."

## Purpose and Scope

SplitSmart helps individuals and groups record shared expenses, review them, track resulting balances and payments, and report on that activity. `UserRequirement.md` is the authoritative source for customer-stated product scope. This specification preserves every feature from that request and distinguishes those requirements from implementation decisions.

The customer requests a **primarily web-based product plus Android and iOS apps**. The project constitution separately selects **Java 21, Maven, and Java Swing** as implementation decisions. A Swing desktop application does not satisfy the requested web or mobile platform coverage. The candidate desktop MVP below is therefore a separately labeled subset, not a replacement for the complete customer requirement or a claim that the platform requirement is met.

## Actors

- **User**: A person with an account who accesses their own expense and balance information and participates in groups.
- **Group creator/manager**: A user who creates a group or is delegated administrative authority to manage its membership. This role definition is a confirmed product decision, not a detail stated in the original customer request.
- **Expense submitter**: A user who records an expense and identifies the participants and split.
- **Expense reviewer/approver**: A group participant authorized to review a submitted expense. Reviewer authority and selection are open product questions.
- **Payer**: A user who paid an expense or records a payment made to settle an amount owed. For expenses, the payer may be different from and outside the set of participants.
- **Report viewer**: An authenticated user who views reports for expenses, balances, and payments they are entitled to access.

One person may perform multiple roles. The requirements do not assume that every group has distinct users for each role.

## User Scenarios & Testing

### User Story 1 - Access a personal account (Priority: P1)

As a person sharing expenses, I can access my own account and information so that my activity and balances are associated with me.

**Why this priority**: Account access is the foundation for associating expenses and balances with individuals.

**Independent Test**: Create two accounts, sign in with each account's email and password, and verify that each can access their own information but not the other's private account information.

**Acceptance Scenarios**:

1. **Given** a person has a valid account, **When** they authenticate with their email address and password, **Then** they can access their own expense and balance information.
2. **Given** two separate accounts with private account information, **When** either user accesses their account, **Then** the other user's private account information is not exposed.
3. **Given** a person submits an invalid or missing email address or password, **When** they attempt to sign in, **Then** access is denied and no account information is shown.

### User Story 2 - Organize participants in a group (Priority: P1)

As a user, I can create and manage a group of people so that shared expenses and balances can be tracked together.

**Why this priority**: Groups provide the context for shared activity among roommates, trip participants, families, coworkers, or other participants.

**Independent Test**: Create a group, add members, remove a member, and verify that only current members can participate in new expenses while the removed member's historical activity and balances remain.

**Acceptance Scenarios**:

1. **Given** an authenticated user, **When** they create a group, **Then** the group is available to its members for shared expense tracking.
2. **Given** a group creator or delegated admin and a group, **When** they manage the group's membership, **Then** the group reflects the resulting membership and only current members can be selected as participants in new group expenses.
3. **Given** a user who is not authorized to manage a group, **When** they attempt a membership change, **Then** the change is rejected and membership remains unchanged.
4. **Given** a member is removed from or leaves a group, **When** group members view historical activity, **Then** the former member's expenses, payments, and balances remain recorded, but the former member cannot participate in new group expenses.

### User Story 3 - Submit and review a shared expense (Priority: P1)

As a group member, I can submit an expense with its details, participants, and split, and authorized members can review it before it affects balances.

**Why this priority**: Accurate expense capture and approval are the source of the balances users rely on.

**Independent Test**: Submit an expense with each supported split type and a payer who is not a participant, verify it awaits review and does not change balances, then approve it and verify the calculated shares and balances.

**Acceptance Scenarios**:

1. **Given** a group member and a valid expense, **When** they submit its amount, date, description, split method, participants, exactly one payer, and optional receipt, **Then** the system records it as awaiting review.
2. **Given** an expense awaiting review, **When** a group member reviews it, **Then** it cannot change balances before the required approval is complete.
3. **Given** an expense awaiting a required approval, **When** it is rejected, **Then** it does not affect balances and its review outcome is visible to authorized group members.
4. **Given** an expense whose required approvals are complete, **When** it is approved, **Then** it is included in group expense history and its shares are applied to balances exactly once.
5. **Given** a valid equally split expense, **When** the system calculates participants' shares, **Then** all shares together equal the expense total at the applicable currency precision.
6. **Given** a valid percentage split, **When** the system calculates participants' shares, **Then** each share reflects its entered percentage and the entered percentages total exactly 100% before the expense can be submitted.
7. **Given** a new expense is created by another group member, **When** the notification system processes it, **Then** the other group members receive a notification that identifies the new expense without changing its approval state.
8. **Given** an expense has a zero or negative amount, **When** a user submits it, **Then** the system rejects it without creating an expense or changing balances.
9. **Given** a percentage split has negative percentages or percentages that do not total exactly 100%, **When** a user submits it, **Then** the system rejects it and identifies the invalid split.
10. **Given** an expense has no payer or has multiple payers, **When** a user submits it, **Then** the system rejects it without creating the expense.

### User Story 4 - Check balances and record settlement (Priority: P1)

As a group member, I can see what each person owes each other and record a payment so that settlements are reflected in current balances.

**Why this priority**: Showing the consequences of approved expenses and recorded payments is the core outcome of expense sharing.

**Independent Test**: Approve an expense with multiple participants, verify pairwise balances, record a payment between two users, and verify the corresponding balance changes.

**Acceptance Scenarios**:

1. **Given** approved expenses in a group, **When** a member views group balances, **Then** the system shows the amount owed by each user to each other user based on those expenses.
2. **Given** an expense that is not approved, **When** a member views balances, **Then** that expense has no effect on the balances.
3. **Given** two users with an amount owed, **When** the payer records a valid payment to the payee, **Then** the payment is recorded and the relevant outstanding balance is reduced by the payment amount.
4. **Given** a payment amount greater than the outstanding amount between the payer and payee, **When** a user tries to record it, **Then** the system rejects the payment and leaves the outstanding balance unchanged.
5. **Given** a payer owes an amount to a payee, **When** the payer records a payment less than or equal to that outstanding amount, **Then** the system accepts the partial or full payment and reduces the outstanding amount accordingly.
6. **Given** a payment has a zero or negative amount, **When** a user tries to record it, **Then** the system rejects it and leaves the payment history and balance unchanged.

### User Story 5 - Review activity reports (Priority: P2)

As a user, I can view summaries and reports of expenses, balances, and payments, including a report for a specified date range and other selected parameters.

**Why this priority**: Reports help users understand and review activity after expense sharing and settlement are in use.

**Independent Test**: Record approved expenses and payments across dates, request a report for a bounded date range, and verify that the result includes only matching activity and reconciles with the corresponding balances.

**Acceptance Scenarios**:

1. **Given** recorded expenses, balances, and payments, **When** an authorized user requests a summary, **Then** the report displays the requested categories for the selected group or scope.
2. **Given** a valid start and end date, **When** a user requests a date-filtered report, **Then** only activity within the selected inclusive date range is included.
3. **Given** an invalid range whose start date is after its end date, **When** a user requests a report, **Then** the request is rejected with a clear validation message.
4. **Given** a report of balances, **When** it is reconciled against approved expenses and recorded payments for the same scope, **Then** the reported values agree.

### User Story 6 - Use SplitSmart on the requested platforms (Priority: P1; customer scope, not covered by the Swing MVP)

As a user, I can use SplitSmart primarily through the web and also through Android and iOS apps, so that the service is available on the platforms requested by the customer.

**Why this priority**: These platforms are explicitly named in the original customer request and represent product scope, not a technical implementation preference.

**Independent Test**: Verify the agreed core user journeys on the production web experience and supported Android and iOS apps, using the platform support criteria agreed in the open questions.

**Acceptance Scenarios**:

1. **Given** a supported web browser, **When** a user completes the agreed core journeys, **Then** the primary web-based experience supports those journeys.
2. **Given** a supported Android device or iOS device, **When** a user completes the agreed core journeys in the corresponding app, **Then** the app supports those journeys and presents the same account, group, expense, approval, balance, payment, and reporting data as the web experience.
3. **Given** a user changes shared data on one supported platform, **When** they access it from another supported platform, **Then** the latest saved data is consistent across platforms.

## Requirements

### Original Customer Requirements Inventory

Every feature below is explicitly present in `UserRequirement.md`; none is removed by the desktop MVP boundary.

| Customer feature | Requirement coverage | Candidate Java Swing MVP |
|---|---|---|
| User accounts: each person has an account, can log in, and access their own expense and balance information | FR-001, FR-002 | Included |
| Group management: create and manage groups of people | FR-003–FR-004 | Included |
| Expense creation: amount, date, description, equal or percentage split, optional receipt image, and involved users | FR-005–FR-008 | Included, with details subject to open questions |
| Notifications when other group members create an expense | FR-009 | Included as an in-app notification |
| Review and approval before expenses affect balances; single or multiple approval levels depending on group needs | FR-010–FR-011 | Included as a single-level approval candidate; configurable multi-level policy not included unless selected |
| Pairwise balance tracking based on expenses entered | FR-013 | Included for approved expenses |
| Record payments to settle balances and reflect them accurately | FR-014–FR-015 | Included |
| Reports and summaries of expenses, balances, and payments, including custom date ranges and parameters | FR-016–FR-018 | Included with date range and agreed filters |
| Primary web-based product and Android and iOS apps | NFR-001 | Not included; a Swing desktop MVP does not satisfy this requirement |

### Functional Requirements

**Account and access**

- **FR-001**: The system MUST allow each person to create or receive an individual user account and authenticate using an email address and password to access SplitSmart.
- **FR-002**: The system MUST associate expenses and balances with the relevant users and MUST prevent a user from accessing private account information belonging to another user. Group information MUST be available only to users authorized for that group.

**Groups**

- **FR-003**: The system MUST allow users to create groups and allow the group creator and delegated admins to add or remove members.
- **FR-004**: The system MUST restrict expense participants to users who are members of the applicable group at the time the expense is submitted.
- **FR-019**: When a member leaves or is removed from a group, the system MUST preserve that member's historical expenses, payments, and balances and MUST prevent the former member from participating in new group expenses.

**Expenses and approvals**

- **FR-005**: The system MUST allow an authorized group member to submit an expense with an amount, date, description, split method, one or more involved group members, and exactly one payer. The payer MAY be outside the set of expense participants.
- **FR-006**: The system MUST support equally divided and percentage-based expense splits.
- **FR-007**: Before accepting an expense or payment, the system MUST reject a non-positive amount. Before accepting an expense, the system MUST also reject a missing required field, missing or multiple payers, a participant outside the group, or a percentage split whose percentages are negative or do not total exactly 100%.
- **FR-008**: The system MUST allow an expense to include an optional receipt image and make that image available with the expense to authorized users.
- **FR-009**: The system MUST provide an in-app notification to other members of the relevant group when a member creates an expense.
- **FR-010**: The system MUST place each submitted expense into a review state and MUST NOT include it in balances or approved-expense reports before it has the required approval.
- **FR-011**: The system MUST support an approval outcome that allows an authorized reviewer to approve or reject an expense before it affects balances. Whether groups configure one or multiple approval levels, how reviewers are chosen, and rejection/editing behavior require product decisions.
- **FR-012**: The system MUST apply the financial effect of an approved expense no more than once, even if an approval action is repeated or retried.

**Balances and payments**

- **FR-013**: The system MUST calculate and display the amount owed by each user to every other user from approved expenses and recorded payments in the applicable group.
- **FR-014**: The system MUST allow a user to record a partial or full payment between a payer and payee, including the amount and parties, and MUST update the applicable outstanding balance accordingly.
- **FR-015**: The system MUST retain payment records so that users can review payments and reports can include them.
- **FR-020**: The system MUST reject a payment greater than the outstanding amount between the payer and payee and MUST leave the balance unchanged when rejecting it.

**Reports**

- **FR-016**: The system MUST provide authorized users with reports or summaries of expenses, balances, and payments.
- **FR-017**: The system MUST allow a user to specify a date range and other supported parameters when requesting a custom report. The initial parameter set is an open product decision.
- **FR-018**: Reported totals MUST reconcile with the underlying approved expenses and recorded payments within the same scope and date/filter criteria.

### Non-Functional Requirements

- **NFR-001 (Customer platform requirement)**: SplitSmart MUST be primarily web-based and MUST also be available as applications that run on Android and iOS. Supported versions, browser/device matrix, and release criteria are open questions. The Java Swing MVP is explicitly not coverage of this requirement.
- **NFR-002 (Monetary correctness)**: Monetary values and calculations MUST avoid binary floating-point representation. Split allocations and settlements MUST use exact decimal arithmetic, apply the currency's smallest unit, distribute rounding remainder units across participants in a deterministic order, and ensure the allocated shares reconcile to the expense total. Supported currencies and the deterministic participant ordering rule remain open questions.
- **NFR-003 (UI responsiveness, Swing MVP only)**: The desktop UI MUST remain interactive while operations that may block for disk or network access are in progress. This criterion applies only if such operations exist in the MVP; acceptable response-time thresholds are open.
- **NFR-004 (Testability)**: Business rules for splits, approval effects, balances, payments, and reporting MUST be verifiable independently of GUI interaction. Automated acceptance evidence MUST be maintained for every accepted functional requirement.
- **NFR-005 (AI-output review)**: AI-generated specifications, diagrams, tests, and implementation artifacts MUST receive human review for requirement alignment and correctness before being accepted as project output.

### Candidate Java Swing MVP Subset (Implementation Decision)

This subsection labels a proposed desktop implementation boundary; it does not modify the customer requirements inventory above.

- **Platform**: Java 21 desktop application using Swing and Maven. This is an implementation decision, not a platform requested by the customer. It does not satisfy NFR-001.
- **Included candidate capabilities**: Individual account access using email and password; group creation and membership management; expense entry with payer, equal or percentage splits, and optional receipt; a single-level review flow; in-app notifications; pairwise balances; partial and full payment recording with overpayments rejected; and summaries with a date-range filter.
- **Not covered by this subset**: A primary web product, Android app, iOS app, and configurable multi-level approvals. This list describes the candidate subset only; all customer features remain in the full requirements inventory.
- **MVP exit condition**: The subset is complete only when its included requirements pass their acceptance scenarios and the product owner has explicitly accepted the separate platform gap. It MUST NOT be described as satisfying the customer's complete platform request.

### Acceptance Criteria Across the Feature

- **AC-001**: Each FR has a demonstrable test or review artifact that verifies its stated observable outcome.
- **AC-002**: An expense in any pre-approval or rejected state contributes zero to balances; an approved expense contributes exactly once.
- **AC-003**: For each approved expense, the sum of participant shares equals the expense amount at the currency precision selected for the group.
- **AC-004**: For each user pair and reporting scope, outstanding balances reconcile to approved expense allocations minus applicable recorded payments.
- **AC-005**: A report filtered by date range and other selected parameters contains no out-of-scope records, and its totals reconcile with the records it includes.
- **AC-006**: No specification, demo, or release claim represents the Swing MVP as web, Android, or iOS support.

### Key Entities

- **User**: An account holder who can belong to groups and participate in expenses, approvals, and payments.
- **Group**: A collection of users whose shared expenses, approvals, balances, and reports are tracked together.
- **Expense**: A submitted monetary obligation with amount, date, description, split method, participants, optional receipt, and review status.
- **Expense share**: A participant's allocated portion of an expense.
- **Approval**: A review decision associated with an expense and reviewer; may be one or more stages depending on the unresolved group policy.
- **Notification**: A notice to group members that another member has created an expense.
- **Balance**: An amount one user owes another within a defined group and scope.
- **Payment**: A recorded transfer from a payer to a payee that settles some or all of an amount owed.
- **Report**: A summary or filtered view of expenses, balances, and payments.

## Success Criteria

### Measurable Outcomes

- **SC-001**: In acceptance testing, 100% of valid expense scenarios using each supported split method produce participant shares whose sum equals the expense total at the selected currency precision.
- **SC-002**: In acceptance testing, 100% of submitted but unapproved and rejected expense scenarios leave balances unchanged.
- **SC-003**: In acceptance testing, 100% of approved expense and payment scenarios produce pairwise balances that reconcile with the underlying transactions.
- **SC-004**: In acceptance testing, 100% of report date-range scenarios exclude records outside the selected range and reconcile to included records.
- **SC-005**: Before the complete customer request is declared delivered, the agreed web, Android, and iOS acceptance journeys pass on the agreed supported platform matrix. A Swing-only MVP cannot meet this criterion.

## Assumptions

- A person is represented by one account, and a user may belong to more than one group.
- The expense creator enters the total expense amount; the creator may record a different payer, who may be outside the participants. Whether that payer must belong to the group remains unresolved.
- The approval status is distinct from expense submission; only approved expenses affect balances.
- The candidate Swing MVP is a proposed implementation subset, not an approved change to the requested web and mobile product scope.
- Expense, group, currency, and reporting behavior not defined in `UserRequirement.md` remains subject to the open questions below; assumptions here do not override customer decisions.

## Open Questions

1. Is a Java Swing desktop MVP an explicitly approved first delivery, given that the customer requested web as the primary platform and Android/iOS apps? What delivery milestone and acceptance are required for each platform?
2. Which web browsers, Android versions/devices, and iOS versions/devices must be supported, and what constitutes the primary web experience?
3. How are accounts created (self-registration, invitation, or another process), and what account recovery behavior is required?
4. Which users may invite members and change group settings? How are group creators or delegated admins selected, and can a user have no remaining group admin when a creator leaves? How should a former member settle any retained outstanding balance?
5. Which currencies are supported, and how is the deterministic order for allocating rounding remainder units selected?
6. Must the expense payer be a member of the group? Can participants have zero shares, and may the payer also be a participant without being included in the split?
7. Which approval levels are supported, how are reviewers selected, can the submitter approve their own expense, and what happens after rejection or a requested correction?
8. Which members receive notifications, and should notices also be sent on approval, rejection, or other status changes?
9. How should users correct or reverse an already recorded payment, and can a payment settle multiple payees or groups?
10. Which report parameters beyond date range are required, and should reports be exportable?
11. What are the required response-time, availability, accessibility, data retention, and privacy/security acceptance thresholds?

## Confirmed Clarifications

The decisions below were confirmed during clarification and resolve ambiguities not answered by the original customer request. They are product decisions, not statements from `UserRequirement.md`.

| Topic | Confirmed decision |
|---|---|
| Expense payer | The payer may be outside the expense participants. |
| Number of payers | Each expense must have exactly one payer. |
| Group administration | The group creator and delegated admins may manage group membership. |
| Member departure/removal | Preserve historical expenses, payments, and balances; prevent new participation. |
| Monetary remainder | Distribute smallest-currency-unit remainders across participants in a deterministic order; the ordering rule remains open. |
| Invalid amounts | Reject zero and negative expense and payment amounts. |
| Percentage totals | Reject percentage splits unless entered percentages total exactly 100%. |
| Authentication | Use email address and password. |
| Expense notifications | Notify other group members in-app when a member creates an expense. |
| Settlement | Allow partial payments; reject payments above the current outstanding amount between payer and payee. |

## Requirement Traceability

| Source in `UserRequirement.md` | Specification IDs |
|---|---|
| User accounts and access to own expense and balance information | FR-001–FR-002 |
| Create and manage groups | FR-003 |
| Create expenses with amount, date, description, equal/percentage split, receipt image, and involved users | FR-004–FR-008 |
| Notify users when another group member creates an expense | FR-009 |
| Review and approve expenses before balance adjustment; single or multiple approval levels | FR-010–FR-012 |
| Track amount owed by each user to every other user from expenses | FR-013 |
| Record payments and reflect them in balances | FR-014–FR-015, FR-020 |
| Reports and summaries of expenses, balances, payments, date ranges, and parameters | FR-016–FR-018 |
| Primary web-based product plus Android and iOS apps | NFR-001; User Story 6 |
| Confirmed clarification: payer may be outside the expense participants | FR-005 |
| Confirmed clarification: preserve a former member's history and balances | FR-019 |
| Confirmed clarification: reject payments above the current amount owed | FR-020 |
