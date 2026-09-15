# Changelog

All notable changes to the Natural Dental Lab Operations Platform are documented in this file.

The project follows the principles of [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and uses semantic versioning conventions for planned releases.

## [Unreleased]

### Planned

- Monthly doctor billing statement and PDF export
- Email-notification administration and controlled retry
- Approval SLA and aging indicators
- Production-stage tracking
- Dispatch and delivery tracking
- Material inventory

## [0.4.0] - 2026-09-15

### Added

- Role-based Owner, Admin, and Staff dashboards
- Daily cases, units, and production metrics
- Doctor-wise monthly billing summary
- Case-type production mix
- Daily production chart
- Recent case register
- Approval backlog and awaiting-action indicators
- Owner-only failed-email warning
- Month, year, and doctor dashboard filters

### Fixed

- Dashboard date handling now uses `DateOnly` consistently with `Record.CaseDate`
- Resolved duplicate root-route ambiguity by assigning one component to `/`
- Added required `IDashboardService` dependency-injection registration guidance

## [0.3.0] - 2026-09-15

### Added

- SMTP delivery through MailKit
- Secure temporary-password generation
- Forced password change at first login
- Invitation status and resend support
- Forgot-password and reset-password email delivery
- `EmailNotifications` delivery audit
- Approval-request lifecycle emails
- Idempotent EF migration for custom Identity columns and email history

### Security

- Moved database, Owner, and SMTP secrets to User Secrets or hosted environment configuration
- Bootstrap Owner password is used only when the Owner account is missing
- Temporary passwords are never stored in plaintext

## [0.2.0] - 2026-09-14

### Added

- Staff Edit and Cancellation approval workflow
- Assignment to one of multiple active Admin users
- Pending, Approved, Rejected, Completed, and Withdrawn states
- One active request per case across Edit and Cancellation types
- Continue Approved Edit recovery
- Retry Approved Cancellation recovery
- Rowversion concurrency for approval decisions

### Changed

- Pending and Approved are treated as active states
- Completed, Rejected, and Withdrawn requests remain as history and do not block future requests

## [0.1.0] - 2026-09-13

### Added

- Doctor management
- Dental case-type management
- Doctor-specific rate cards
- Composite Doctor/CaseType pricing
- Multi-line laboratory case entry
- Tooth and arch selection
- Unit and amount calculation
- Case search, filtering, pagination, detail, edit, and cancellation
- Audit fields and optimistic concurrency
- ASP.NET Core Identity roles: Owner, Admin, and Staff

[Unreleased]: https://github.com/<your-account>/natural-dental-lab-platform/compare/v0.4.0...HEAD
[0.4.0]: https://github.com/<your-account>/natural-dental-lab-platform/releases/tag/v0.4.0
[0.3.0]: https://github.com/<your-account>/natural-dental-lab-platform/releases/tag/v0.3.0
[0.2.0]: https://github.com/<your-account>/natural-dental-lab-platform/releases/tag/v0.2.0
[0.1.0]: https://github.com/<your-account>/natural-dental-lab-platform/releases/tag/v0.1.0
