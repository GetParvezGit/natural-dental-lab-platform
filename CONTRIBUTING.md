# Contributing to Natural Dental Lab Operations Platform

Thank you for improving the Natural Dental Lab Operations Platform. This project manages laboratory cases, doctor-specific pricing, billing, users, approvals, and notifications. Changes must preserve security, auditability, and historical billing integrity.

## Code of Conduct

Contributors must communicate respectfully, protect confidential business and patient-related information, and avoid publishing private credentials or data.

## Before Starting

1. Review the root `README.md`.
2. Review `docs/architecture/architecture-overview.md`.
3. Review applicable Architecture Decision Records.
4. Search existing issues and pull requests.
5. Create or link an issue for non-trivial work.

## Branch Naming

Use short, focused branch names:

```text
feature/monthly-doctor-report
feature/production-status
feature/email-admin-page
fix/dashboard-dateonly-filter
fix/approval-concurrency
docs/update-deployment-guide
chore/upgrade-packages
```

Recommended prefixes:

- `feature/`
- `fix/`
- `docs/`
- `test/`
- `refactor/`
- `chore/`

## Commit Messages

Use conventional, descriptive messages:

```text
feat: add monthly doctor billing summary
fix: preserve inactive rate during edit
fix: prevent multiple active requests per case
docs: add Azure deployment runbook
test: cover approved cancellation retry
chore: update MailKit package
```

Avoid messages such as `done`, `latest`, `changes`, or `fix issue`.

## Architecture Rules

### Dependency direction

```text
DentalLab.Web -> Application + Infrastructure
Infrastructure -> Application + Domain
Application -> Domain
Domain -> no project dependencies
```

### Layer ownership

- Domain: business entities and state
- Application: interfaces, DTOs, validation contracts, security constants
- Infrastructure: EF Core, Identity, SMTP, service implementations
- Web: Blazor components, middleware, routing, DI composition

Do not add direct SQL or business mutation logic to Razor components.

## Business Invariants

Every change must preserve these rules unless an approved Architecture Decision changes them:

- `Record.UnitRate` is a historical pricing snapshot
- cancelled cases remain auditable and do not contribute to active billing
- only one Pending or Approved request may exist per case
- non-owner Admin can action only assigned approval requests
- email failure does not roll back valid business state
- Staff cannot directly edit or cancel existing cases
- authorization is enforced in services, not only UI
- server-side validation is mandatory

## Development Setup

Follow `docs/deployment/local-setup.md`.

Use User Secrets for local credentials. Never add real values to `appsettings.json`.

## Database Changes

### Identity and email schema

Use EF Core migrations.

```powershell
dotnet ef migrations add <MigrationName> `
  --context ApplicationDbContext `
  --project DentalLab.Web `
  --startup-project DentalLab.Web
```

Review the migration, designer, snapshot, and generated SQL.

### Business schema

Use the versioned business SQL or database project. Do not silently move business tables into EF migrations.

### Required checks

- fresh database deployment succeeds
- existing database deployment succeeds
- no unexpected business table is created or dropped
- rollback and forward-fix implications are documented

## Coding Standards

- follow `.editorconfig`
- enable nullable reference types
- use asynchronous database and network APIs
- pass `CancellationToken` through service calls
- use `IDbContextFactory<ApplicationDbContext>` for Blazor service operations
- avoid shared DbContext instances across circuits
- use explicit DTOs rather than binding entities directly
- avoid hard-coded roles, statuses, routes, and configuration keys
- encode user-supplied values in HTML email content
- never log passwords, reset tokens, connection strings, or SMTP credentials

## Testing Requirements

At minimum, test affected scenarios:

- valid path
- validation failure
- authorization failure
- concurrency conflict
- disabled parent entity
- cancellation behavior
- email failure isolation
- Owner/Admin/Staff differences

Before opening a pull request:

```powershell
dotnet restore
dotnet build
dotnet test
```

## Pull Request Checklist

- [ ] Change has a focused purpose
- [ ] Business invariants are preserved
- [ ] Authorization is enforced server-side
- [ ] Tests cover critical paths
- [ ] Database changes are versioned and reviewed
- [ ] No credentials or private data are included
- [ ] Documentation is updated
- [ ] Screenshots contain synthetic data only
- [ ] Build and tests pass
- [ ] Migration and deployment implications are described

## Sensitive Information

Never include:

- production or personal connection strings
- Owner passwords
- SMTP passwords or application passwords
- temporary passwords
- reset tokens
- private certificates
- real patient names
- real doctor contact data without authorization
- database backups

If sensitive material is committed, stop work, rotate the credential, report the exposure privately, and remove it from Git history.

## Documentation

Update related files under `docs/` when changing:

- architecture boundaries
- database entities or migration ownership
- user or approval workflows
- configuration keys
- deployment procedures
- operational recovery

## Review Expectations

Reviewers evaluate:

- correctness
- security
- maintainability
- audit integrity
- deployment safety
- compatibility with existing data
- clarity for future contributors
