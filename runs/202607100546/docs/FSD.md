# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE batch process selects patient transfer records from HAPTRFR, applies exclusion rules for voided and outpatient transfers, enriches each record with level, status, benefit, and ranking data, and produces XML output via HXFXMLH/HXFXMLD (and related tables) for downstream consumers. The objective of this migration is to deliver equivalent functionality as a Spring Boot service backed by SQL Server, preserving all business rules around inclusion/exclusion and enrichment.

## 1. Functional Scope

The following filter rules are **in scope** and must be implemented exactly as in the legacy system:

- **BR-017 – File Indicator Required**
  - When -FILE INDICATOR equals zero, branch to 'SKIP'. Records with fileIndicator = 0 must not be included in XML output.

- **BR-018 – Voided Transfers Excluded**
  - When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. Records flagged as voided are excluded from processing.

- **BR-019 – Outpatient Transfers Skipped in Inpatient Runs**
  - When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' for runs configured as inpatient-only.

## 2. API Contract

### 2.1 Inputs (API Request Parameters)

| Parameter | AS400 Field / Concept | Type | Description |
|----------|------------------------|------|-------------|
| runDate | AFTRDT | date | Business date for which transfers should be processed. |
| runTime | AFTRTM | time | Time window boundary used in transfer selection (optional; may default to full day). |
| level6 | AFLVL6 | string | Level 6 plan or facility code used to scope transfers. |
| accountRangeFrom | AFACCT (start) | string | Lower bound of account range included in the run. |
| accountRangeTo | AFACCT (end) | string | Upper bound of account range included in the run. |
| inpatientOnly | -INPATIENT/OUTPATIENT FLAG | boolean | When true, restricts processing to inpatient records; outpatient records are skipped. |
| includeVoided | -FLAG INDICATOR | boolean | When false, records flagged as void/voided are excluded. |
| fileIndicatorRequired | -FILE INDICATOR | boolean | When true, only records with non-zero file indicator are processed. |

Additional implicit inputs:

- **Current user identity** – In the AS400 job context, user identity is taken from the job/user profile. In Spring Boot, this will be carried via the security principal.

Security mapping notes:

- Authenticate API clients via OAuth2/OpenID Connect.
- Apply RBAC so that only roles such as `PATIENT_TRANSFER_BATCH_RUNNER` can invoke the endpoint.
- Log all requests, including user ID, parameters, and result counts, to a PHI audit trail because XML output contains PHI-bearing fields.

### 2.2 Spring Boot API Design

**Method & Path:**

- `POST /api/patient-transfers/export`

**Parameters:**

| Name | Type | Required | Validation |
|------|------|----------|-----------|
| runDate | LocalDate | yes | Must be a valid date; not in the future by more than configured limit. |
| runTimeFrom | String (HHMM) | no | If provided, must match `^\d{4}$` and represent a valid time. |
| runTimeTo | String (HHMM) | no | Same constraints as runTimeFrom. |
| level6 | String | yes | Non-empty; must match an existing HXPLVL6 code. |
| accountRangeFrom | String | no | If provided, must be <= accountRangeTo lexically. |
| accountRangeTo | String | no | If provided, must be >= accountRangeFrom lexically. |
| inpatientOnly | Boolean | no | Defaults to false. |
| includeVoided | Boolean | no | Defaults to false. |
| fileIndicatorRequired | Boolean | no | Defaults to true. |

**Layer Structure:**

- **Controller:** `PatientTransferExportController`
- **Service:** `PatientTransferExportService`
- **Repositories:**
  - `HaptrfrRepository` (HABADTE_HAPTRFR)
  - `OxpnstnRepository` (HABADTE_OXPNSTN)
  - `OxpbnfitRepository` (HABADTE_OXPBNFIT)
  - `OapirnkRepository` (HABADTE_OAPIRNK)
  - `HxplvlRepository` (HABADTE_HXPLVL* tables)
  - `XmlHeaderRepository` (HABADTE_HXPXMLR)
  - `XmlDetailRepository` (HABADTE_HXPXMLD)

**Response JSON Shape:**

```json
{
  "runDate": "2026-07-10",
  "level6": "L6CODE",
  "processedTransferCount": 120,
  "skippedFileIndicatorZero": 5,
  "skippedVoidedTransfers": 3,
  "skippedOutpatientInInpatientRun": 12,
  "transfers": [
    {
      "accountNumber": "0001234567",
      "mrn": "MRN001234",
      "transferDate": "2026-07-09",
      "transferTime": "13:45",
      "transferType": "ADMISSION",
      "levelDescription": "Acute Inpatient",
      "statusDescription": "ACTIVE",
      "benefitCode": "BEN001",
      "benefitPlan": "PLAN01"
    }
  ]
}
```

**Java Entity/DTO Sketch:**

```java
public record PatientTransferExportRequest(
    LocalDate runDate,
    String runTimeFrom,
    String runTimeTo,
    String level6,
    String accountRangeFrom,
    String accountRangeTo,
    boolean inpatientOnly,
    boolean includeVoided,
    boolean fileIndicatorRequired
) {}

public record PatientTransferDto(
    String accountNumber,
    String mrn,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String levelDescription,
    String statusDescription,
    String benefitCode,
    String benefitPlan
) {}

public record PatientTransferExportResponse(
    LocalDate runDate,
    String level6,
    int processedTransferCount,
    int skippedFileIndicatorZero,
    int skippedVoidedTransfers,
    int skippedOutpatientInInpatientRun,
    List<PatientTransferDto> transfers
) {}
```

## 3. Business Rules (Summary)

### 3.1 Main HABADTE Rules

| Rule ID | Description |
|--------:|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

### 3.2 Supporting Utility Rules

| Rule ID | Description |
|--------:|-------------|
| BR-003 | Date validation: reject year < 1800 (historical minimum). |
| BR-004 | Date validation: reject year > 2100 (forecast maximum). |
| BR-005 | Date validation: reject month < 01 (calendar constraint). |
| BR-006 | Date validation: reject month > 12 (calendar constraint). |
| BR-007 | Date validation: reject day < 01 (calendar constraint). |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |
| BR-009–BR-012 | Level lookup: reject if level code exceeds valid range. |
| BR-013–BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |

## 4. Data Model

### 4.1 SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| HAPTRFR | HABADTE_HAPTRFR | Patient transfer records. |
| HXPDICT | HABADTE_HXPDICT | Cross-reference dictionary of patient/account data. |
| HXPLVL1–HXPLVL6 | HABADTE_HXPLVL1–6 | Level/plan metadata tables. |
| HXPTABLD | HABADTE_HXPTABLD | Generic table dictionary. |
| HXPXMLD | HABADTE_HXPXMLD | XML detail records. |
| HXPXMLR | HABADTE_HXPXMLR | XML header records. |
| OAPIRNK | HABADTE_OAPIRNK | Ranking data associated with accounts. |
| OMPMAST | HABADTE_OMPMAST | Patient master records (account, MRN, name, SSN). |
| OXPBNFIT | HABADTE_OXPBNFIT | Benefit data per plan. |
| OXPNSTN | HABADTE_OXPNSTN | Status codes per level. |
| TAPIRNK | HABADTE_TAPIRNK | Staging rank data. |
| TMPMAST | HABADTE_TMPMAST | Staging patient master data. |
| TXPBNFIT | HABADTE_TXPBNFIT | Staging benefit data. |
| TXPNSTN | HABADTE_TXPNSTN | Staging status data. |

### 4.2 Suggested SQL Server Indexes

```sql
CREATE INDEX IX_HAPTRFR_RUN
    ON HABADTE_HAPTRFR (AFTRDT, AFTRTM, AFLVL6, AFACCT);

CREATE INDEX IX_OXPNSTN_LEVEL_STATUS
    ON HABADTE_OXPNSTN (XFNLV6, XFNSST);

CREATE INDEX IX_HXPLVL6_LEVEL
    ON HABADTE_HXPLVL6 (HX6NUM);

CREATE INDEX IX_OXPBNFIT_BENEFIT_PLAN
    ON HABADTE_OXPBNFIT (XFBUBN, XFBPLN);

CREATE INDEX IX_OAPIRNK_LEVEL_ACCOUNT
    ON HABADTE_OAPIRNK (BRKLV6, BRKACC, BRKSEQ);

CREATE INDEX IX_HXPXMLR_USER_SEQ
    ON HABADTE_HXPXMLR (XMRUSR, XMRSEQ);

CREATE INDEX IX_HXPXMLD_USER_SEQ
    ON HABADTE_HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2);
```

## 5. Acceptance Criteria

The following edge cases and outcomes must be verified in QA and UAT:

| Scenario | Expected Behavior |
|---------|-------------------|
| Transfer date = 0 or invalid | Reject or mark record as error; do not generate XML; log incident. |
| Transfer time = 0000 with no business meaning | Treat as midnight or null based on configuration; ensure consistent display. |
| fileIndicator = 0 | Skip record (BR-017); increment `skippedFileIndicatorZero`. |
| flagIndicator IN ('VOID', 'VOIDED') | Skip record (BR-018); increment `skippedVoidedTransfers`. |
| outpatient record in inpatient-only run | Skip record (BR-019); increment `skippedOutpatientInInpatientRun`. |
| Level code outside valid range | Treat as error in level enrichment; use default description; consider raising alert. |
| Level code not found in HXPLVL6 | Use "UNKNOWN_LEVEL" description; continue processing so XML remains complete. |
| Status code not found in OXPNSTN | Use "UNKNOWN_STATUS"; continue processing. |
| Benefit record missing in OXPBNFIT | Omit benefit fields in XML or mark as "NO_BENEFIT"; do not fail run. |
| Rank data missing in OAPIRNK | No rank section in XML; processing continues. |
| Empty result set (no transfers match criteria) | Return response with zero counts and no XML records; consider informational message to caller. |
| Preference not configured for run parameters | Fail fast with error code; do not perform partial processing.

> Full detail: see Business_Processing_Rules_HABADTE.md
