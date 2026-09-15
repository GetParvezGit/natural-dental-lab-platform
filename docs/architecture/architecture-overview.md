# Architecture Overview

## Purpose

The Natural Dental Lab Operations Platform supports the commercial and operational lifecycle of a dental laboratory that manufactures patient-specific dental prosthetic products for doctors and dental clinics.

The architecture prioritizes:

- clear project boundaries
- secure role-based access
- historical billing integrity
- auditable case corrections and cancellations
- repeatable database deployment
- non-blocking email notifications
- maintainable future expansion

![System context](../images/system-context.png)

## Context

```mermaid
flowchart LR
    D[Doctors and Clinics] -->|Patient-specific dental work| P[Natural Dental Lab Platform]
    O[Owner and Admin] -->|Master data, rates, users, approvals| P
    S[Laboratory Staff] -->|Case entry and requests| P
    P -->|Production and billing records| DB[(SQL Server or Azure SQL)]
    P -->|Invitations and workflow notifications| SMTP[SMTP Provider]
```

Doctors are the commercial customers. Patients identify individual laboratory cases but do not authenticate into the current platform.

## Logical Architecture

![Logical architecture](../images/logical-architecture.png)

```mermaid
graph TD
    WEB[DentalLab.Web]
    APP[DentalLab.Application]
    INFRA[DentalLab.Infrastructure]
    DOMAIN[DentalLab.Domain]
    DATA[(SQL Server or Azure SQL)]
    MAIL[SMTP]

    WEB --> APP
    WEB --> INFRA
    INFRA --> APP
    APP --> DOMAIN
    INFRA --> DOMAIN
    INFRA --> DATA
    INFRA --> MAIL
```

### DentalLab.Domain

Contains persistent business state and domain entities:

- `Doctor`
- `CaseType`
- `DoctorRate`
- `Record`
- `CaseApprovalRequest`
- `EmailNotification`

The Domain project does not contain UI, SMTP, EF orchestration, or configuration code.

### DentalLab.Application

Contains stable contracts consumed by the Web and Infrastructure layers:

- service interfaces
- request and response models
- case, approval, dashboard, and user DTOs
- role constants
- approval status and request-type constants
- validation attributes

### DentalLab.Infrastructure

Implements external and persistence concerns:

- `ApplicationDbContext`
- entity configurations
- ASP.NET Core Identity custom user model
- Owner and role seeding
- doctor, rate, case, approval, dashboard, and user services
- SMTP transport and email audit
- per-operation `IDbContextFactory` usage

### DentalLab.Web

Provides the interactive internal application:

- Blazor Server pages and components
- role-aware navigation
- account management UI
- temporary-password middleware
- dashboard visualization
- dependency-injection composition root
- authentication endpoints

## Runtime Request Pattern

```mermaid
sequenceDiagram
    participant Browser
    participant Razor as Blazor Component
    participant Service as Application Service
    participant Factory as DbContext Factory
    participant SQL as SQL Database

    Browser->>Razor: Authenticated action
    Razor->>Service: Typed request model
    Service->>Factory: Create DbContext
    Factory-->>Service: New context instance
    Service->>SQL: Validated query or transaction
    SQL-->>Service: Data or result
    Service-->>Razor: Result model
    Razor-->>Browser: Updated UI
```

A new `ApplicationDbContext` is created per service operation. This avoids sharing a non-thread-safe EF Core context across long-lived Blazor circuits.

## Principal Design Rules

1. Business authorization is enforced inside services as well as pages.
2. `Record.UnitRate` is a historical transaction snapshot.
3. Cancelled cases remain available for audit but are excluded from active production and billing.
4. Only one `Pending` or `Approved` request can exist for a case.
5. Email failure never rolls back a valid business transaction.
6. Identity and email schema are managed through EF migrations.
7. Business tables remain managed through versioned SQL scripts or a database project.
8. Secrets are injected through trusted configuration providers and never committed.

## Deployment Topology

![Deployment topology](../images/deployment-topology.png)

The recommended Azure topology consists of:

- Azure App Service hosting the Blazor Server application
- Azure SQL Database
- SMTP provider
- App Service settings or Azure Key Vault references
- HTTPS-only access
- Application Insights as a recommended operational enhancement

## Non-Functional Characteristics

### Security

- ASP.NET Core Identity
- confirmed-account requirement
- Owner/Admin/Staff authorization
- password hashing and tokenized reset flow
- forced initial password change
- server-side validation
- SQL parameterization through EF Core
- rowversion concurrency

### Reliability

- transactional case creation and updates
- retry-capable SQL connection options
- resumable approved workflow actions
- non-blocking SMTP notifications
- migration history and reproducible schema

### Maintainability

- layered projects
- narrow interfaces
- typed DTOs
- isolated entity configuration
- version-controlled documentation
- architecture decision records

## Related Documents

- [Enterprise HLD and LLD](DentalLab_Enterprise_HLD_LLD.docx)
- [Architecture decisions](architecture-decisions.md)
- [Database design](../database/database-design.md)
- [Migration strategy](../database/migration-strategy.md)
- [Local setup](../deployment/local-setup.md)
