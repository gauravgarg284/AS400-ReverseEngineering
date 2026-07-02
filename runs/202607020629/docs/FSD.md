# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

HABADTE is an RPGLE program in the Patient Management domain that produces a detailed inpatient transfer activity extract. It reads transfer records from HAPTRFR, applies three key business rules (file indicator, void flag, inpatient/outpatient flag), enriches data with benefit plan and station information, and outputs a structured header/detail/footer dataset suitable for reporting and downstream analytics.

## 1. Functional Scope

IN SCOPE business rules:

- BR-017: When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

## 2. API Contract

### Inputs (API Request Parameters)

The Java/Spring Boot API that replaces HABADTE should accept the following request parameters. These are derived from the transfer and level keys used in the DDS and narratives for HABADTE and related files.

| Parameter            | AS400 Field | Type        | Description |
|----------------------|------------|------------|-------------|
| accountNumber        | AFACCT     | String(12) | Patient account number at level 6 used to select transfer records from HAPTRFR. |
| level6Identifier     | AFLVL6     | String(6)  | Level-6 organizational or patient grouping key used in HAPTRFR and related files. |
| transferFromDate     | AFTRDT     | LocalDate  | Inclusive start date for transfer selection; maps to transfer date field AFTRDT (DECIMAL 8,0). |
| transferToDate       | AFTRDT     | LocalDate  | Inclusive end date for transfer selection; maps to AFTRDT; applied as between condition. |
| inpatientOnly        | AFTYPE / INOUT flag | Boolean | When true, restricts output to inpatient transfers using -INPATIENT/OUTPATIENT FLAG rule. |
| includeVoided        | FLAG indicator | Boolean | When false, skip records with void/voided flag based on BR-018. |
| fileIndicatorFilter  | FILE indicator | Integer   | Optional explicit filter for file indicator; default logic uses BR-017 to skip zero. |

The current AS400 implementation also implicitly uses the current user identity and security context (job user profile) to derive printer destination and XML header values. In the Spring Boot implementation, this should be mapped to OAuth2 authentication with RBAC roles (e.g. ROLE_BILLING_ANALYST, ROLE_CLINICAL_ANALYST) and PHI audit logging so that each extract request is tied to a user principal and purpose-of-use.

### Spring Boot API Design

#### 14.1 Recommended REST Endpoint

- Method: GET
- Path: `/api/patient-transfers`

Parameters:

| Name               | Type        | Required | Validation |
|--------------------|------------|----------|-----------|
| accountNumber      | String      | Yes      | Non-empty, matches account format. |
| level6Identifier   | String      | Yes      | Non-empty, length 6. |
| transferFromDate   | LocalDate   | Yes      | Valid date, not after transferToDate. |
| transferToDate     | LocalDate   | Yes      | Valid date, not before transferFromDate. |
| inpatientOnly      | Boolean     | No       | Defaults to true. |
| includeVoided      | Boolean     | No       | Defaults to false. |

#### 14.2 Layer Structure

- Controller: `TransferActivityController` – exposes REST endpoint, handles request/response mapping.
- Service: `TransferActivityService` – implements business rules BR-017–BR-019 and enrichment logic.
- Repository: `HaptrfrRepository`, `BenefitPlanRepository`, `StationRepository`, `HierarchyRepository` – encapsulate SQL queries.

#### 14.3 Response JSON Shape

```json
{
  "accountNumber": "123456789012",
  "level6Identifier": "LVL006",
  "transfers": [
    {
      "transferDate": "2024-06-01",
      "transferTime": "09:30",
      "transferType": "I",
      "mrn": "MRN123456",
      "stationCode": "STN01",
      "stationName": "Med-Surg East",
      "benefitPlan": "PLAN-A",
      "benefitPhone": "555-123-4567"
    }
  ],
  "summary": {
    "totalRecordsRead": 120,
    "totalValidTransfers": 80,
    "skippedFileIndicatorZero": 5,
    "skippedVoided": 10,
    "skippedOutpatient": 25
  }
}
```

#### 14.4 Java Entity/DTO Sketch

```java
public record TransferActivityRequest(
    String accountNumber,
    String level6Identifier,
    LocalDate transferFromDate,
    LocalDate transferToDate,
    boolean inpatientOnly,
    boolean includeVoided
) {}

public record TransferActivityRow(
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String mrn,
    String stationCode,
    String stationName,
    String benefitPlan,
    String benefitPhone
) {}

public record TransferActivityResponse(
    String accountNumber,
    String level6Identifier,
    List<TransferActivityRow> transfers,
    SummaryCounters summary
) {}

public record SummaryCounters(
    long totalRecordsRead,
    long totalValidTransfers,
    long skippedFileIndicatorZero,
    long skippedVoided,
    long skippedOutpatient
) {}
```

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

## 4. Data Model

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR      | dbo.HAPTRFR      | Primary transfer activity table capturing patient account movements. |
| HXPBNFIT     | dbo.HXPBNFIT     | Benefit plan and contact lookup for enrichment. |
| HXPNSTN      | dbo.HXPNSTN      | Nursing station and location master for station enrichment. |
| HXPLVL1      | dbo.HXPLVL1      | Level 1 organizational hierarchy. |
| HXPLVL2      | dbo.HXPLVL2      | Level 2 organizational hierarchy. |
| HXPLVL3      | dbo.HXPLVL3      | Level 3 organizational hierarchy. |
| HXPLVL4      | dbo.HXPLVL4      | Level 4 organizational hierarchy. |
| HXPLVL5      | dbo.HXPLVL5      | Level 5 organizational hierarchy. |
| HXPLVL6      | dbo.HXPLVL6      | Level 6 patient/account hierarchy. |
| OXPBNFIT     | dbo.OXPBNFIT     | Physical benefit plan storage (if separate from HXPBNFIT). |
| OXPNSTN      | dbo.OXPNSTN      | Physical station master (if separate from HXPNSTN). |

## 5. Acceptance Criteria

| Scenario                                 | Expected Behavior |
|------------------------------------------|-------------------|
| File indicator equals zero               | Exclude record from output; increment `skippedFileIndicatorZero`. |
| Flag indicator marks record as voided    | Exclude record; increment `skippedVoided`. |
| Inpatient/outpatient flag indicates 'O'  | Exclude record; increment `skippedOutpatient`. |
| Null or zero date fields (AFTRDT = 0)    | Treat as null; either exclude record or route to error handling depending on business policy. |
| Benefit plan not found for account/level | Include transfer, but leave benefit fields blank and flag in logs. |
| Station record not found                 | Include transfer, using raw station code and flag in logs. |
| Printer or XML preferences missing       | Use system defaults for output layout and destination. |
| Empty result set                         | Return empty `transfers` array with summary counters = 0. |

> Full detail: see Business_Processing_Rules_HABADTE.md
