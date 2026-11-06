# CIS 285 Lab 7 — Consolidation Instructions for Codex / Copilot

## Architecture decision

Use `specs/001-splitsmart-feature/architecture-baseline.md` as the selected architecture for the **proposed Java 21, Maven, Swing desktop milestone**. Keep `architecture-alternative.md` unchanged as the rejected-for-now alternative. Read `UserRequirement.md`, `.specify/memory/constitution.md`, `spec.md`, `plan.md`, `tasks.md`, and `Lab7_Architecture_Review.md` before editing.

## Required actions

1. Create `specs/001-splitsmart-feature/architecture-selected.md` that consolidates the layered baseline with approved-for-design reliability improvements: atomic/repeat-safe approval and payment handling, durable repository/transaction ports, retained audit records, off-EDT I/O, and traceable tests. Explain why CQRS/event sourcing was not selected.
2. Create `specs/001-splitsmart-feature/design.md` with component responsibilities, data flow, conceptual data model, PlantUML class diagram, activity diagrams for expense submission and approval/rejection, and desktop deployment diagram. Preserve domain independence from Swing and persistence.
3. Update `specs/001-splitsmart-feature/spec.md`, `plan.md`, and `tasks.md` **only as needed for traceability and consistency**. Do not silently change customer requirements or treat unresolved choices as approved. Add an explicit architecture-decision reference and clearly labeled open-decision gates.
4. Add `specs/001-splitsmart-feature/requirements-traceability.md` mapping existing FR/NFR IDs to selected components, diagrams, and proposed verification activities.
5. Keep `UserRequirement.md`, the constitution, `architecture-baseline.md`, and `architecture-alternative.md` unchanged. Do not implement Java application code unless separately requested.
6. Mark the original web/Android/iOS platform requirement as **unmet by the Swing desktop milestone**; do not claim full platform coverage.
7. Validate internal Markdown links, PlantUML fenced blocks, diagram consistency, and Git diff. Report files changed and unresolved decisions. Do not claim PlantUML diagrams rendered unless rendering actually succeeds.

## Constraints

Use Java 21, Maven, Swing, JUnit 5, PlantUML, and `BigDecimal`. Preserve exactly one payer per expense, positive amounts, exact 100% percentage splits, deterministic fractional-cent handling under approved policy, former-member history, in-app notifications, pending approval exclusion from balances, partial payments, and overpayment rejection. Respect the current requirements and label all still-undecided policies as open. Prefer the simpler layered design over introducing a new event journal or CQRS.
