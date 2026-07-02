# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE system produces an inpatient transfer detail extract for a given level-6 organizational hierarchy and patient account, filtering out invalid, voided, and outpatient transfer records, and enriching eligible records with benefit plan and patient status information. This extract is used by clinical and operational users to understand patient movement over time and must be faithfully migrated to a Java / Spring Boot / SQL Server stack with equivalent inclusion/exclusion logic, data enrichment, and PHI protection.

## 1. Functional Scope

IN SCOPE business rules (from HABADTE):
- **BR-017**: When -FILE INDICATOR equals zero, branch to `SKIP`.
- **BR-018**: When -FLAG INDICATOR equals void/voided, branch to `SKIP`.
- **BR-019**: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP`.

## 2. API Contract

Copied verbatim from section (2) Inputs and section (14) Spring Boot API Design.

### Inputs (API Request Parameters)

| Parameter        | AS400 Field | Type       | Description                                     |
|------------------|-------------|-----------|-------------------------------------------------|
| level6           | AFLVL6      | string(6) | Hierarchy level code for the patient/account.   |
| accountNumber    | AFACCT      | string(10)| Patient account identifier.                      |
| fromDate         | AFTRDT      | date      | Inclusive start date for transfer activity.     |
| toDate           | AFTRDT      | date      | Inclusive end date for transfer activity.       |
| includeOutpatient| flag        | boolean   | Whether to include outpatient transfers.        |
| flagsFilter      | -FLAG IND   | string    | Optional filter on void/voided indicators.      |

Additional implicit input:
- **Current user identity** is obtained from the security context (e.g., JWT token) and mapped to RBAC roles; it is not a direct parameter but is required for PHI access checks.

Security mapping note for migration:
- The API must be protected using OAuth2 (e.g., Spring Security with JWT) and RBAC roles such as `ROLE_CLINICAL`, `ROLE_BILLING`, and `ROLE_ANALYTICS`.
- All access to PHI-bearing fields (AFACCT, AFMRNO from HAPTRFR; MMACCT, MMMRNO, MMNAME, MMPSSN, MMMMRN from OMPMAST; BRKMRN from OAPIRNK; phone and name fields from HXPDICT/OXPBNFIT) must be logged with an audit trail (who accessed, when, purpose).

### Spring Boot API Design

#### Recommended REST Endpoint

Endpoint pattern:
- `GET /api/patient-transfers`

Parameters:

| Name             | Type      | Required | Validation                                  |
|------------------|----------|----------|----------------------------------------------|
| level6           | string   | yes      | Non-empty, matches known hierarchy codes.    |
| accountNumber    | string   | yes      | Non-empty, digits only or configured format. |
| fromDate         | date     | yes      | Valid date, <= toDate.                       |
| toDate           | date     | yes      | Valid date, >= fromDate.                     |
| includeOutpatient| boolean  | no       | Defaults to false.                           |

#### Layer Structure

- **Controller**: `PatientTransferController` (exposes REST endpoint, validates inputs).
- **Service**: `PatientTransferService` (implements BR-017–BR-019 logic and enrichment steps).
- **Repository**: `HaptrfrRepository`, `BenefitPlanRepository`, `PatientStatusRepository`, `PatientMasterRepository`, `RiskRankingRepository` using Spring Data JPA or JDBC.

#### Response JSON Shape

```json
{
  "level6": "001234",
  "accountNumber": "0009876543",
  "fromDate": "2026-01-01",
  "toDate": "2026-01-31",
  "transfers": [
    {
      "transferDate": "2026-01-05",
      "transferTime": "13:45",
      "transferType": "AD",
      "statusCode": "INP",
      "statusDescription": "Inpatient",
      "benefitNumber": "BN123",
      "benefitPlan": "PL01",
      "benefitPhone": "555-123-4567"
    }
  ],
  "summary": {
    "totalRecordsRead": 42,
    "totalRecordsSkipped": 12,
    "totalRecordsIncluded": 30
  }
}
```

#### Java Entity/DTO Sketch

```java
public record PatientTransferRequest(
    String level6,
    String accountNumber,
    LocalDate fromDate,
    LocalDate toDate,
    boolean includeOutpatient
) {}

public record PatientTransferDetail(
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String statusCode,
    String statusDescription,
    String benefitNumber,
    String benefitPlan,
    String benefitPhone
) {}

public record PatientTransferResponse(
    String level6,
    String accountNumber,
    LocalDate fromDate,
    LocalDate toDate,
    List<PatientTransferDetail> transfers,
    int totalRecordsRead,
    int totalRecordsSkipped,
    int totalRecordsIncluded
) {}
```

## 3. Business Rules (Summary)

Copied from section (16) Business Rules Reference Summary.

| Rule ID | Description                                                           |
|---------|-----------------------------------------------------------------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to `SKIP`.                   |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to `SKIP`.           |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP`. |

## 4. Data Model

Copied verbatim from section (13) SQL Server Table Mapping.

| AS400 Object | SQL Server Table | Purpose                                           |
|--------------|------------------|---------------------------------------------------|
| HAPTRFR      | dbo.HAPTRFR      | Primary transfer detail source.                   |
| HAPIRNK      | dbo.HAPIRNK      | Transfer risk/ ranking logical view.             |
| TAPIRNK      | dbo.TAPIRNK      | Underlying transfer risk physical file.          |
| HMLMAST5H    | dbo.HMLMAST5H    | Logical view over patient master by level/date.  |
| TMPMAST      | dbo.TMPMAST      | Patient master physical file.                    |
| HXPBNFIT     | dbo.HXPBNFIT     | Logical benefit plan lookup.                     |
| TXPBNFIT     | dbo.TXPBNFIT     | Physical benefit plan file.                      |
| HXPNSTN      | dbo.HXPNSTN      | Logical patient status lookup.                   |
| TXPNSTN      | dbo.TXPNSTN      | Physical patient status file.                    |
| OXPBNFIT     | dbo.OXPBNFIT     | Alternate benefit plan storage.                  |
| OMPMAST      | dbo.OMPMAST      | Patient master with PHI (MRN, name, SSN).        |
| OAPIRNK      | dbo.OAPIRNK      | Risk ranking physical file with MRN.             |

## 5. Acceptance Criteria

Copied from section (17) Edge Cases to Implement.

| Scenario                          | Expected Behavior                                                  |
|-----------------------------------|--------------------------------------------------------------------|
| Date field value is 0             | Treat as null; exclude from date range checks where appropriate.  |
| Time field value is 0             | Treat as midnight (00:00) or null based on business guidance.     |
| Benefit plan lookup not found     | Set benefit fields to null; mark coverageStatus = 'UNKNOWN'.      |
| Status lookup not found           | Use default statusDescription = 'UNKNOWN STATUS'; log a warning.  |
| Logical delete flag on lookup row | Treat row as not found; do not attach deleted data.              |
| All records skipped               | Return empty transfers array with summary counts indicating zero included. |
| Preferences not configured        | Apply HABADTE default behavior (inpatient only, skip voided).     |

> Full detail: see Business_Processing_Rules_HABADTE.md
