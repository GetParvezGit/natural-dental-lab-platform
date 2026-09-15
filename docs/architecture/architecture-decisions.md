# Architecture Decisions

This file records the key architecture decisions for the Natural Dental Lab Operations Platform. New decisions should be added as numbered entries rather than rewriting historical rationale.

## ADR-001: Use a Layered .NET Solution

**Status:** Accepted

**Decision:** Separate the solution into Domain, Application, Infrastructure, and Web projects.

**Rationale:**

- business entities remain independent from UI and SMTP concerns
- service contracts are stable and testable
- Infrastructure implementations can evolve without changing Razor components
- dependency direction remains clear

**Consequence:** Project references must preserve the dependency direction documented in the architecture overview.

## ADR-002: Use Blazor Server

**Status:** Accepted

**Decision:** Use Blazor Server with Interactive Server rendering for the internal operations application.

**Rationale:**

- one .NET stack across UI and backend
- rapid delivery of business forms
- strong authentication integration
- suitable for internal, authenticated laboratory users

**Consequence:** EF Core contexts must not be shared across long-lived circuits. Services use `IDbContextFactory<ApplicationDbContext>` and create a context per operation.

## ADR-003: Use Email as UserName

**Status:** Accepted

**Decision:** Set both Identity `Email` and `UserName` to the same email address and authenticate with email plus password.

**Rationale:**

- simpler invitation and recovery flow
- no separate username to communicate
- clear link between approval audit and authenticated identity

**Consequence:** A future email-change feature must update `Email`, `NormalizedEmail`, `UserName`, and `NormalizedUserName` through `UserManager` rather than direct SQL.

## ADR-004: Preserve Unit Rate on the Case Line

**Status:** Accepted

**Decision:** Copy the applicable doctor rate into `Record.UnitRate` when the case line is created or validly edited.

**Rationale:** Historical doctor billing must not change when the current rate card changes.

**Consequence:** Reports and dashboards use stored case values, not the latest `DoctorRate` value.

## ADR-005: One Active Approval Request per Case

**Status:** Accepted

**Decision:** A case can have only one request in `Pending` or `Approved` state, regardless of whether the type is Edit or Cancel.

**Rationale:** Concurrent edit and cancellation requests create ambiguous ownership and conflicting outcomes.

**Consequence:** A new request is allowed only after the previous request becomes `Completed`, `Rejected`, or `Withdrawn`.

## ADR-006: Approved Is an Active State

**Status:** Accepted

**Decision:** Approval and implementation are distinct states.

**Rationale:** An Admin may approve an Edit request but still need to open and save the case.

**Consequence:** Approved Edit requests appear as Awaiting Action and can be resumed by the assigned Admin or Owner. Approved Cancellation can be retried when execution fails.

## ADR-007: Preserve Multiple Admins and Explicit Assignment

**Status:** Accepted

**Decision:** Staff selects an active Admin when creating an Edit or Cancellation request.

**Rationale:** Natural Dental Lab may have multiple Admin accounts and requires clear ownership.

**Consequence:** A non-owner Admin can review only requests assigned to the same Admin account. Owner has recovery access to all requests.

## ADR-008: Keep Email Non-Blocking

**Status:** Accepted

**Decision:** Save the business transaction before sending the email. Email failure is audited but does not roll back the case, request, review, cancellation, or user creation.

**Rationale:** SMTP availability must not control laboratory data integrity.

**Consequence:** Operational monitoring and controlled retry are required for failed notifications.

## ADR-009: Use Temporary Password plus Forced Change

**Status:** Accepted

**Decision:** Generate a secure temporary password for newly created users, email it once, and require password change at first login.

**Rationale:** Owner/Admin should not manually invent or retain user passwords.

**Consequence:** `MustChangePassword`, issuance date, password-change date, invitation status, and invitation error are stored on `ApplicationUser`.

## ADR-010: Use Mixed Schema Ownership

**Status:** Accepted

**Decision:** Manage Identity customizations and `EmailNotifications` through EF migrations. Manage business tables using versioned SQL scripts or a database project.

**Rationale:** The solution already uses separate enterprise business-schema deployment while Identity naturally integrates with EF migrations.

**Consequence:** Business entity configurations use `ExcludeFromMigrations()`. `EmailNotifications` must not be excluded.

## ADR-011: Use DateOnly for Case Date

**Status:** Accepted

**Decision:** Store the business case date as `DateOnly`; store audit timestamps as UTC `DateTime`.

**Rationale:** Case date reflects a laboratory business day and does not require a time component.

**Consequence:** Dashboard month boundaries and comparisons use `DateOnly`. Audit timestamps are converted to local time only for display.

## ADR-012: Role-Aware Dashboard Data

**Status:** Accepted

**Decision:** Owner and Admin receive business-wide production and billing. Staff receives personal production data filtered by `CreatedBy` email and no organization-wide financial totals.

**Rationale:** Staff requires operational visibility without unnecessary access to business financial data.

**Consequence:** Dashboard authorization is implemented in the query service, not only through hidden UI sections.

## ADR-013: Do Not Fully Disable TLS Certificate Validation

**Status:** Accepted

**Decision:** Production SMTP uses normal certificate validation. Certificate revocation checking may be disabled only in a constrained development environment when the network cannot access revocation endpoints.

**Rationale:** Fully trusting any server certificate creates a man-in-the-middle risk.

**Consequence:** Development workarounds must be environment-controlled and removed before production deployment.
