# Configuration Reference

## Configuration Sources

Configuration is loaded using the standard .NET configuration hierarchy. Later providers override earlier providers.

Recommended usage:

- `appsettings.json`: safe defaults and empty placeholders
- `appsettings.Development.json`: non-secret development behavior
- User Secrets: local credentials
- environment variables: hosted environment overrides
- Azure Key Vault references: production secrets

## Required Keys

### Database

```text
ConnectionStrings:DefaultConnection
```

Purpose: SQL Server or Azure SQL connection used by `ApplicationDbContext`.

### Bootstrap Owner

```text
BootstrapOwner:Email
BootstrapOwner:Password
```

The password is required only when the configured Owner does not exist. The seeder does not reset an existing Owner password.

### SMTP

```text
Smtp:Host
Smtp:Port
Smtp:UserName
Smtp:Password
Smtp:FromEmail
Smtp:FromName
Smtp:Security
```

Recommended Gmail-compatible values:

```text
Host       = smtp.gmail.com
Port       = 587
Security   = StartTls
FromName   = Natural Dental Lab
```

Use an application password, not the normal mailbox password.

## Safe appsettings Example

```json
{
  "ConnectionStrings": {
    "DefaultConnection": ""
  },
  "BootstrapOwner": {
    "Email": "",
    "Password": ""
  },
  "Smtp": {
    "Host": "",
    "Port": 587,
    "UserName": "",
    "Password": "",
    "FromEmail": "",
    "FromName": "Natural Dental Lab",
    "Security": "StartTls"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## Azure Environment Variable Names

```text
ConnectionStrings__DefaultConnection
BootstrapOwner__Email
BootstrapOwner__Password
Smtp__Host
Smtp__Port
Smtp__UserName
Smtp__Password
Smtp__FromEmail
Smtp__FromName
Smtp__Security
```

## Security Rules

- never commit real values
- do not print `dotnet user-secrets list` output in logs
- rotate exposed secrets immediately
- use a dedicated notification mailbox
- restrict database credentials to the required database
- prefer Key Vault references for production
- remove secrets from Git history after accidental commit; deleting only the latest file is insufficient

## SMTP Security Modes

| Value | Typical Port | Behavior |
|---|---:|---|
| `StartTls` | 587 | Connect then upgrade to TLS |
| `SslOnConnect` | 465 | TLS at connection start |
| `None` | provider-specific | No transport encryption; test-only in a trusted environment |

Do not mix port 465 with `StartTls` unless the provider explicitly requires that combination.

## Application Service Registrations

The Web composition root must register:

```csharp
builder.Services.Configure<SmtpOptions>(
    builder.Configuration.GetSection(SmtpOptions.SectionName));

builder.Services.AddScoped<IApplicationEmailService,
    SmtpApplicationEmailService>();

builder.Services.AddScoped<IEmailSender<ApplicationUser>,
    SmtpIdentityEmailSender>();

builder.Services.AddScoped<IDashboardService,
    DashboardService>();
```

Other domain service registrations must remain registered once only.
