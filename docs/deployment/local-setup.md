# Local Setup

## Prerequisites

- .NET 10 SDK
- Visual Studio with ASP.NET and EF Core tooling, or .NET CLI
- SQL Server, Local SQL Server, or an Azure SQL development database
- SMTP mailbox and application password
- Git

## 1. Clone

```powershell
git clone https://github.com/<your-account>/natural-dental-lab-platform.git
cd natural-dental-lab-platform
```

## 2. Select Startup Project

Use `DentalLab.Web` as the startup project and migration assembly.

## 3. Configure User Secrets

Run from the `DentalLab.Web` project directory:

```powershell
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "<connection-string>"
dotnet user-secrets set "BootstrapOwner:Email" "owner@example.com"
dotnet user-secrets set "BootstrapOwner:Password" "<strong-initial-password>"

dotnet user-secrets set "Smtp:Host" "smtp.gmail.com"
dotnet user-secrets set "Smtp:Port" "587"
dotnet user-secrets set "Smtp:UserName" "notifications@example.com"
dotnet user-secrets set "Smtp:Password" "<application-password>"
dotnet user-secrets set "Smtp:FromEmail" "notifications@example.com"
dotnet user-secrets set "Smtp:FromName" "Natural Dental Lab"
dotnet user-secrets set "Smtp:Security" "StartTls"
```

Verify configured key names:

```powershell
dotnet user-secrets list
```

Do not paste the output into public issues because the output contains secret values.

## 4. Restore and Build

```powershell
dotnet restore
dotnet build
```

## 5. Apply EF Migrations

If projects are at the repository root:

```powershell
dotnet ef database update `
  --context ApplicationDbContext `
  --project DentalLab.Web `
  --startup-project DentalLab.Web
```

If projects are under `src/`:

```powershell
dotnet ef database update `
  --context ApplicationDbContext `
  --project src/DentalLab.Web `
  --startup-project src/DentalLab.Web
```

## 6. Apply Business Schema

Run versioned scripts from `database/` in dependency order:

1. Doctors
2. CaseTypes
3. DoctorRates
4. Records and `Seq_CaseId`
5. CaseApprovalRequests and indexes
6. seed/reference data

## 7. Start

```powershell
dotnet run --project DentalLab.Web
```

or:

```powershell
dotnet run --project src/DentalLab.Web
```

## 8. First Login

At startup, `IdentitySeeder`:

- creates Owner, Admin, and Staff roles if missing
- reads Bootstrap Owner email/password from configuration
- creates the Owner only if missing
- confirms and activates the Owner
- assigns the Owner role

Login with the configured Owner email and initial password. Change the password from Account Management when required.

## 9. Initial Business Setup

1. Create Admin and Staff accounts.
2. Confirm invitation delivery.
3. Create Doctors.
4. Create Case Types.
5. Configure doctor-specific rates.
6. Create test cases.
7. Verify dashboard totals.
8. Test Staff approval requests.

## Troubleshooting

### Duplicate root route

Only one component can own `@page "/"`. Keep Dashboard at `/dashboard` or redirect from a single Home component.

### Dashboard service missing

Register:

```csharp
builder.Services.AddScoped<IDashboardService, DashboardService>();
```

### DateOnly compile errors

Use `DateOnly` month boundaries and compare `CaseDate` with `DateOnly` values.

### Email Failed

Inspect `EmailNotifications.ErrorMessage`, correct SMTP, restart, and use Resend.
