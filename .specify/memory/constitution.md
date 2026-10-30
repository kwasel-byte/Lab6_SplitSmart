# SplitSmart Constitution

## Core Principles

### I. Customer Requirements Are the Product Authority
`UserRequirement.md` is the authoritative record of the original customer request and must not be changed to fit an implementation. Specifications and plans must distinguish customer-stated requirements from implementation decisions. If an implementation decision changes, narrows, defers, or conflicts with a customer requirement, record that explicitly and obtain the appropriate product decision; do not present the implementation choice as a customer requirement or claim unsupported coverage.

### II. Java 21 and Maven
Implement the application in Java 21 and manage builds, dependencies, and test execution with Maven. Keep the project build reproducible through the checked-in Maven configuration, and use the standard project structure and conventions unless a documented need justifies a deviation.

### III. Separation of GUI and Business Logic
Use Java Swing for the desktop user interface. Keep Swing components and event handling at the presentation boundary; expense, approval, payment, balance, and reporting rules belong in independently testable business logic. Coordinate UI updates on Swing's Event Dispatch Thread and avoid embedding domain calculations in views or listeners.

### IV. Object-Oriented Design and PlantUML
Model the domain with cohesive, well-encapsulated objects and explicit responsibilities. Prefer clear interfaces and composition over unnecessary coupling or inheritance. Use PlantUML diagrams when architecture, domain relationships, or important workflows need visual documentation, and keep diagrams consistent with the implemented design.

### V. Exact Monetary Arithmetic
Represent and calculate money with `BigDecimal`, never binary floating-point types. Define currency scale and rounding behavior at explicit business boundaries, and test allocation and settlement calculations—including rounding remainders—so totals remain correct and explainable.

### VI. JUnit 5 and Verifiable Behavior
Use JUnit 5 for automated tests. Test business rules independently of Swing, with particular attention to expense splits, approvals, balance changes, payments, and report calculations. Tests must verify observable behavior and relevant boundary cases; do not treat successful compilation as evidence that requirements are met.

### VII. Requirement Traceability
Maintain traceability from each customer requirement to its specification, implementation decision, and verification evidence. Mark whether an item is customer-stated or implementation-defined, identify deferred or unsupported requirements, and ensure tests or other explicit verification cover each accepted requirement. Do not silently substitute technical choices for product scope.

### VIII. Human Review of AI Output
AI-generated code, tests, diagrams, specifications, and other project artifacts are proposals, not approved decisions. A human must review them for correctness, requirement alignment, security, maintainability, and consistency with this constitution before acceptance. AI output must not independently authorize changes to customer requirements or replace human approval.

## Implementation Constraints and Scope

- The original customer request names a primary web-based product and Android and iOS apps. Java 21, Maven, and Java Swing are implementation choices for the requested project, not customer requirements and not evidence that those requested platforms are supported.
- Specifications must state the intended delivery platform and explicitly identify any mismatch, scope change, or deferred platform requirement before treating the work as complete.
- Apply the principles above to feature work and technical documentation. Any exception must be explicit, justified, and approved by a human reviewer.

## Governance

This constitution governs project specifications, implementation, and verification. Review changes for requirement traceability, test coverage, accurate platform claims, and adherence to the principles. Amendments require a documented rationale and human approval; they must not rewrite `UserRequirement.md`.

**Version**: 1.0.0 | **Ratified**: 2026-10-29 | **Last Amended**: 2026-10-29
