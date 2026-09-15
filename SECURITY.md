# Security Policy

## Supported Versions

Security fixes are applied to the current development branch and the latest released version.

| Version | Supported |
|---|:---:|
| Latest release | Yes |
| `main` | Yes |
| Older unreleased snapshots | No |

## Reporting a Vulnerability

Do not open a public GitHub issue for a suspected vulnerability, exposed credential, patient-data leak, authentication bypass, or database exposure.

Report the issue privately to the repository owner or Natural Dental Lab's authorized technical contact. If GitHub Private Vulnerability Reporting is enabled, use the repository's **Security** tab.

Include only the information required to reproduce the issue:

- affected version or commit
- affected route, service, or component
- user role involved
- sanitized reproduction steps
- impact assessment
- suggested mitigation, if known

Do not include:

- passwords
- temporary passwords
- reset tokens
- connection strings
- SMTP credentials
- private certificates
- real patient records
- database backups

## Response Targets

Target response times, subject to business availability:

- acknowledgment: within 3 business days
- initial assessment: within 7 business days
- remediation plan: based on severity and exploitability

Critical exposures involving credentials or patient-related data should be treated as immediate incidents.

## Severity Guidance

### Critical

- authentication or authorization bypass
- remote code execution
- SQL injection with production impact
- exposed database or SMTP credentials
- access to real patient or doctor data without authorization
- ability for Staff to perform unrestricted Admin/Owner operations

### High

- approval ownership bypass
- password-reset token exposure
- insecure certificate validation in production
- mass assignment causing billing or case manipulation
- stored cross-site scripting in case notes or email templates

### Medium

- missing rate limiting on sensitive endpoints
- excessive information in errors or logs
- unauthorized business-wide dashboard access
- email audit information exposed to an inappropriate role

### Low

- non-sensitive information disclosure
- security headers or hardening improvements without direct exploitation

## Security Architecture

The application uses:

- ASP.NET Core Identity
- Owner, Admin, and Staff role authorization
- confirmed-account sign-in requirement
- hashed passwords
- secure temporary-password generation
- forced initial password change
- tokenized password reset
- optimistic concurrency through SQL `rowversion`
- server-side validation
- EF Core parameterization
- HTTPS and SMTP TLS
- User Secrets, environment settings, or Key Vault references

## Secret Management

Never commit secrets to the repository.

Local development:

```text
.NET User Secrets
```

Hosted environments:

```text
Azure App Service settings
Azure Key Vault references
Approved pipeline secret stores
```

If a secret is exposed:

1. revoke or rotate the secret immediately
2. determine the exposure window
3. inspect logs for misuse
4. remove the secret from Git history
5. update affected environments
6. document the incident privately

Deleting the secret only from the latest commit is not sufficient.

## Patient and Business Data

Use synthetic data in:

- public repositories
- screenshots
- documentation
- issues
- tests
- demos

Do not commit database exports or backups. Patient names, doctor contact details, pricing, and production records must be handled under Natural Dental Lab's approved privacy and access procedures.

## Authorization Requirements

Security-sensitive operations must be protected in the service layer, not only by hidden UI controls.

Examples:

- Staff cannot directly edit or cancel a case
- Admin can review only assigned requests
- Owner has recovery access
- Admin cannot create another Admin
- user cannot disable the currently authenticated account
- Staff dashboard cannot expose organization-wide financial data

## SMTP Security

- use TLS
- use an application password or provider-specific secret
- do not use a personal mailbox password
- do not fully disable certificate validation in production
- record send failures without exposing credentials
- keep email failure non-blocking for valid business transactions

## Dependency Security

- use supported .NET and package versions
- enable Dependabot or equivalent monitoring
- review package updates before merging
- run CodeQL or equivalent static analysis
- remove unused packages

## Security Verification Checklist

- [ ] No secrets in repository or history
- [ ] Role checks exist in pages and services
- [ ] Temporary-password middleware cannot be bypassed through business routes
- [ ] Reset tokens are never logged
- [ ] User-provided email content is HTML encoded
- [ ] SQL is parameterized
- [ ] Rowversion conflicts are handled
- [ ] Cancelled cases are immutable through normal Staff workflows
- [ ] Failed email errors are visible only to authorized users
- [ ] Production certificate validation is enabled
- [ ] Application uses HTTPS-only hosting
