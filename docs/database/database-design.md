# Database Design

## Overview

The database stores four related concerns:

1. ASP.NET Core Identity
2. Natural Dental Lab master data
3. Laboratory case transactions
4. Approval and notification audit

The current solution intentionally combines EF-managed Identity/email schema with version-controlled business SQL.

## Principal Tables

### Doctors

Represents doctors and dental clinics that send patient-specific work to Natural Dental Lab.

Typical columns:

- `DocId`
- `DocName`
- contact/address fields
- `IsActive`
- audit fields
- `RowVersion`, if configured

A disabled doctor cannot receive new active rates or new case selection.

### CaseTypes

Represents dental products manufactured by the laboratory.

Typical columns:

- `CaseTypeId`
- `CaseTypeCode`
- `CaseTypeName`
- `Scope`, where `T` is tooth and `A` is arch
- display order
- `IsActive`
- audit fields

### DoctorRates

Stores doctor-specific pricing using the composite key:

```text
DocId + CaseTypeId
```

Important columns:

- `DocId`
- `CaseTypeId`
- `Cost`
- `IsActive`
- audit or concurrency fields as configured

The rate can be active only when both the Doctor and CaseType are active.

### Records

Stores one row per case line. Rows sharing `CaseId` form a logical case aggregate.

Important fields:

- `CaseId`
- `LineNumber`
- `CaseDate` as `DateOnly`
- `DocId`
- patient name or patient reference
- `CaseTypeId`
- tooth quadrants: UR, UL, LR, LL
- Arch
- `Units`
- `UnitRate`
- `Amount`
- notes
- cancellation fields
- created and modified audit fields
- `RowVersion`

`UnitRate` and `Amount` are commercial snapshots. Historical records must not dynamically join to the current doctor rate when calculating billing.

### CaseApprovalRequests

Stores controlled Staff Edit and Cancellation requests.

Important fields:

- `RequestId`
- `CaseId`
- `RequestType`
- `Status`
- requester user ID and email
- assigned Admin user ID and email
- reason and additional details
- review user, email, timestamp, and comment
- completion user, email, and timestamp
- `RowVersion`

Recommended indexes:

```text
AssignedAdminUserId + Status
RequestedByUserId + Status
CaseId
```

### AspNetUsers

Uses the standard ASP.NET Core Identity columns plus:

- `MustChangePassword`
- `TemporaryPasswordIssuedOn`
- `PasswordChangedOn`
- `InvitationEmailStatus`
- `InvitationEmailSentOn`
- `InvitationEmailError`

Passwords remain hashed by Identity. The temporary password is never stored in plaintext.

### EmailNotifications

Audits email attempts.

- `EmailNotificationId`
- `RecipientEmail`
- `Subject`
- `NotificationType`
- `RelatedEntityType`
- `RelatedEntityId`
- `Status`
- `AttemptCount`
- `ErrorMessage`
- `CreatedOn`
- `SentOn`

Recommended index:

```text
Status + CreatedOn
```

## Case and Billing Calculations

```text
Tooth scope units = count(UR) + count(UL) + count(LR) + count(LL)
Arch scope units  = 1 per arch line
Line amount       = UnitRate x Units
Case total        = sum of non-cancelled line amounts
Monthly amount    = sum of non-cancelled line amounts in the selected month
```

## Data Integrity Rules

- `CaseId` is generated through `dbo.Seq_CaseId`
- `LineNumber` is unique within a case
- duplicate tooth selection within a case is rejected
- parent entities must be active before a rate is activated
- cancelled case lines remain available for audit
- only one Pending or Approved request is allowed per case
- rowversion prevents stale writes
- audit timestamps are stored in UTC except the business `CaseDate`

## Data Retention

Recommended policy:

- do not physically delete completed laboratory cases
- do not physically delete approval history
- retain email metadata according to operational and privacy policy
- disable master records rather than deleting referenced rows
- anonymize or archive sensitive data only through a controlled process

## Query Examples

### Monthly doctor billing

```sql
SELECT
    r.DocId,
    d.DocName,
    COUNT(DISTINCT r.CaseId) AS CaseCount,
    SUM(r.Units) AS Units,
    SUM(r.Amount) AS Amount
FROM dbo.Records AS r
INNER JOIN dbo.Doctors AS d
    ON d.DocId = r.DocId
WHERE r.IsCancelled = 0
  AND r.CaseDate >= @MonthStart
  AND r.CaseDate < @MonthEnd
GROUP BY r.DocId, d.DocName
ORDER BY Amount DESC;
```

### Active approval backlog

```sql
SELECT
    RequestId,
    CaseId,
    RequestType,
    Status,
    AssignedAdminEmail,
    RequestedOn
FROM dbo.CaseApprovalRequests
WHERE Status IN ('Pending', 'Approved')
ORDER BY RequestedOn;
```

### Failed email notifications

```sql
SELECT TOP (50)
    EmailNotificationId,
    RecipientEmail,
    NotificationType,
    RelatedEntityId,
    ErrorMessage,
    CreatedOn
FROM dbo.EmailNotifications
WHERE Status = 'Failed'
ORDER BY EmailNotificationId DESC;
```
