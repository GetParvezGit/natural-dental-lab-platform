# Migration Strategy

## Objective

The database deployment model must reproduce the same schema in local development, test, staging, and production without duplicate objects, manual drift, or hidden one-off changes.

## Schema Ownership

### EF Core migrations manage

- ASP.NET Core Identity tables
- custom `AspNetUsers` columns
- `EmailNotifications`
- related indexes and defaults
- `__EFMigrationsHistory`

### Versioned business SQL manages

- `Doctors`
- `CaseTypes`
- `DoctorRates`
- `Records`
- `CaseApprovalRequests`
- `Seq_CaseId`
- business views
- stored procedures
- reference seed data

Business entity configurations remain mapped for EF queries but use `ExcludeFromMigrations()`.

## Development Migration Workflow

### 1. Update the model

Change `ApplicationUser`, `EmailNotification`, the DbContext, or entity configuration.

### 2. Build

```powershell
dotnet restore
dotnet build
```

### 3. Generate migration

```powershell
dotnet ef migrations add <MigrationName> `
  --context ApplicationDbContext `
  --project DentalLab.Web `
  --startup-project DentalLab.Web
```

Adjust project paths when projects are under `src/`.

### 4. Inspect the migration

The migration must not create, drop, or unexpectedly alter business-owned tables.

Review:

- migration class
- designer file
- `ApplicationDbContextModelSnapshot`
- generated SQL

### 5. Preview SQL

```powershell
dotnet ef migrations script `
  --context ApplicationDbContext `
  --project DentalLab.Web `
  --startup-project DentalLab.Web `
  --output migration-preview.sql
```

### 6. Apply to development

```powershell
dotnet ef database update `
  --context ApplicationDbContext `
  --project DentalLab.Web `
  --startup-project DentalLab.Web
```

## Existing Database Baseline

The Phase 2 Identity/email schema was initially created through SQL and later represented by an idempotent EF migration.

The migration uses checks such as:

```sql
IF COL_LENGTH('dbo.AspNetUsers', 'MustChangePassword') IS NULL
```

and:

```sql
IF OBJECT_ID(N'dbo.EmailNotifications', N'U') IS NULL
```

This supports both conditions:

- existing database: skip existing object and record the migration
- fresh database: create missing object

## Fresh Database Deployment

1. Create the empty SQL database.
2. Run EF migrations for Identity and email schema.
3. Run business schema scripts in dependency order.
4. Run seed scripts.
5. Start the application so roles and the missing Bootstrap Owner are seeded.
6. Verify migration history, roles, Owner login, SMTP, and dashboard.

## Production Deployment

Do not rely on an operator running `Update-Database` interactively against production.

Generate an idempotent deployment script:

```powershell
dotnet ef migrations script --idempotent `
  --context ApplicationDbContext `
  --project DentalLab.Web `
  --startup-project DentalLab.Web `
  --output Database/Identity/IdentityDeployment.sql
```

Recommended release order:

1. backup or restore point
2. reviewed Identity/email migration SQL
3. reviewed business-schema deployment
4. application deployment
5. configuration deployment
6. smoke tests

## Source-Control Rules

Commit together:

- migration `.cs`
- migration designer `.cs`
- model snapshot
- related model/configuration changes

Do not commit:

- generated scripts containing secrets
- local database files
- database backups
- production connection strings

## Rollback

- prefer forward-fix migrations for production
- use `Down()` only when data loss is understood and approved
- restore from backup for destructive rollback
- never manually delete a row from `__EFMigrationsHistory` without reconciling the actual schema

## Validation Queries

```sql
SELECT MigrationId, ProductVersion
FROM dbo.__EFMigrationsHistory
ORDER BY MigrationId;
```

```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'dbo'
  AND TABLE_NAME = 'AspNetUsers'
ORDER BY ORDINAL_POSITION;
```
