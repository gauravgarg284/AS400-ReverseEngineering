# Business Processing Rules & Functional Specification

## HABADTE Patient Transfer Detail Extract (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

The HABADTE program produces a detailed extract of inpatient transfer activity for a single patient account and level, suitable for downstream reporting or integration. Operational staff and clinical informatics users rely on this output to understand how a patient's location and status have changed over time.

> Core business question: "For a given inpatient account at a specific hierarchy level, what eligible transfer records exist, and how should they be represented in the patient transfer detail extract?"

Summary outputs produced include:
- A sequence of transfer detail rows keyed by level (AFLVL6), account (AFACCT), transfer date (AFTRDT), transfer time (AFTRTM), and type (AFTYPE) from HAPTRFR.
- Enriched fields for benefit plan (from HXPBNFIT/OXPBNFIT) and patient status (from HXPNSTN/OXPNSTN).
- Optional linkage to broader patient master (OMPMAST) and risk information (HAPIRNK/OAPIRNK) when consumed by downstream processes.

## (2) Inputs (API Request Parameters)

When migrated to Spring Boot, HABADTE will be exposed as a REST API that accepts request parameters corresponding to the original selection criteria. Based on the transfer file keys and the presence of hierarchical level and account fields, the core request parameters are:

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

## (3) Organizational Hierarchy

The data dictionary indicates a multi-level hierarchy (HXPLVL1–HXPLVL6 physical files) and a patient status file (HXPNSTN/OXPNSTN) keyed by hierarchy level and status code. For HABADTE, the key field AFLVL6 in HAPTRFR corresponds to the level-6 hierarchy for patient accounts.

| Level Number | Name                  | Key Size Digits | AS400 Table |
|--------------|-----------------------|-----------------|-------------|
| 6            | Patient Level 6 Code  | variable        | HXPLVL6     |

BR Note on report header:
- Transfer detail extracts should display the level-6 description (from HXPLVL6 via XFXLDSC) in the header to orient users to the organizational unit (e.g., "Facility → Service Line → Unit").

## (4) Patient Transfer Data Source

### 4.1 Data Access Pattern

The primary data source for HABADTE is the physical file **HAPTRFR**:
- Record format: HAFTRFR
- Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

The AS400 program declares HAPTRFR and reads transfer records keyed by level and account, iterating in chronological order:
- Access pattern: keyed read by `(AFLVL6, AFACCT)` with range selection on `AFTRDT` and `AFTRTM`.
- Sort order: primarily by transfer date/time ascending, then transfer type.

SQL Server equivalent:

```sql
SELECT AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE, AFMRNO, /* other transfer fields */
FROM dbo.HAPTRFR
WHERE AFLVL6 = :level6
  AND AFACCT = :accountNumber
  AND AFTRDT BETWEEN :fromDate AND :toDate
ORDER BY AFTRDT ASC, AFTRTM ASC, AFTYPE ASC;
```

### 4.2 Key Fields

Key fields for the HAPTRFR table in SQL Server:

| SQL Column | AS400 Field | Type        | Description                                       |
|-----------|-------------|------------|---------------------------------------------------|
| level6    | AFLVL6      | CHAR(6)    | Organizational level for patient account.         |
| account   | AFACCT      | CHAR(10)   | Patient account identifier (PHI).                 |
| transferDate | AFTRDT   | DECIMAL(8) | Date of transfer (YYYYMMDD format).               |
| transferTime | AFTRTM   | DECIMAL(4) | Time of transfer (HHMM format).                   |
| transferType | AFTYPE   | CHAR(2)    | Transfer type code (admission, discharge, etc.).  |

## (5) Inclusion and Exclusion Rules

HABADTE contains three core inclusion/exclusion rules identified as BR-017, BR-018, and BR-019.

### BR-017 — File Indicator Skip

**Description**: Records where the file indicator equals zero are skipped and not included in the transfer detail output.

```pseudocode
for each record in HAPTRFR keyed by (AFLVL6, AFACCT):
    if FILE_INDICATOR = 0:
        goto SKIP_RECORD
    else:
        process_record()
```

SQL WHERE clause fragment:

```sql
-- BR-017: skip records where FILE_INDICATOR = 0
WHERE fileIndicator <> 0
```

### BR-018 — Voided Flag Skip

**Description**: Records with a flag indicator marked as void/voided are skipped to prevent inclusion of cancelled or logically deleted transfers.

```pseudocode
if FLAG_INDICATOR in ('VOID', 'VOIDED'):
    goto SKIP_RECORD
```

SQL WHERE clause fragment:

```sql
-- BR-018: skip voided transfers
AND flagIndicator NOT IN ('VOID', 'VOIDED')
```

### BR-019 — Outpatient Flag Skip

**Description**: Records flagged as outpatient are excluded, focusing the extract on inpatient activity only.

```pseudocode
if INPATIENT_OUTPATIENT_FLAG = 'O':
    goto SKIP_RECORD
```

SQL WHERE clause fragment:

```sql
-- BR-019: skip outpatient transfers
AND inpatientOutpatientFlag <> 'O'
```

### Summary SQL WHERE Clause

Combining all three rules for HABADTE:

```sql
WHERE fileIndicator <> 0              -- BR-017
  AND flagIndicator NOT IN ('VOID', 'VOIDED')  -- BR-018
  AND inpatientOutpatientFlag <> 'O'  -- BR-019
ORDER BY AFTRDT ASC, AFTRTM ASC, AFTYPE ASC;
```

## (6) Benefit Plan Lookup (Coverage Enrichment)

HABADTE declares the logical file **HXPBNFIT** (pfile TXPBNFIT) and the physical file **OXPBNFIT** as part of enrichment.

### BR and Algorithm Steps

Enrichment goal: for each qualifying transfer record, attach benefit plan information based on plan codes and level-6 hierarchy.

Steps:
1. Extract benefit plan identifiers from the transfer record or associated account context (e.g., plan number and plan code).
2. Perform a keyed lookup into HXPBNFIT/OXPBNFIT using `XFBUBN` (benefit number) and `XFBPLN` (plan code).
3. If a matching row is found, attach phone contact information `XFBTEL` and other coverage attributes.
4. If no match is found, flag the record as having unknown coverage but do not fail the extract.

SQL SELECT equivalent:

```sql
SELECT b.XFBUBN, b.XFBPLN, b.XFBTEL, /* other benefit fields */
FROM dbo.OXPBNFIT AS b
WHERE b.XFBUBN = :benefitNumber
  AND b.XFBPLN = :planCode;
```

Edge cases:
- **Not found**: When no benefit row exists, set coverage fields to null and mark `coverageStatus = 'UNKNOWN'`.
- **Delete flag logic**: If the benefit row contains a logical delete flag, treat it as not found.
- **Derived flags**: Use benefit data to derive convenience flags such as `hasPhoneContact` based on `XFBTEL` presence.

## (7) Patient Status Lookup (Status Enrichment)

HABADTE uses the logical file **HXPNSTN** (pfile TXPNSTN) and physical file **OXPNSTN** to enrich transfer records with patient status descriptions.

### BR and Algorithm Steps

1. For each transfer record from HAPTRFR, determine the level-6 code `AFLVL6` and status code field (e.g., AFSTATUS).
2. Look up HXPNSTN/OXPNSTN with key fields `XFNLV6` (level-6) and `XFNSST` (status).
3. Attach status description and classification (e.g., observation, ICU, step-down) to the output.

SQL SELECT equivalent:

```sql
SELECT s.XFNLV6, s.XFNSST, /* status description */
FROM dbo.OXPNSTN AS s
WHERE s.XFNLV6 = :level6
  AND s.XFNSST = :statusCode;
```

Edge cases:
- **Not found**: Use a default status description such as `"UNKNOWN STATUS"` and log a warning.
- **Delete flag**: Skip rows marked as deleted and continue search if alternate mappings exist.

## (8) Transfer Risk and Master Linkage (Context Enrichment)

Although HABADTE primarily reads HAPTRFR, the data lineage shows flow paths from TMPMAST/HMLMAST5H and TAPIRNK/HAPIRNK as upstream sources that may contribute context when the extract is consumed downstream.

Combined enrichment:
- Use **HMLMAST5H/TMPMAST** (patient master by level and admit date/time) to attach patient demographics and master account context.
- Use **HAPIRNK/TAPIRNK/OAPIRNK** to attach risk or ranking attributes via `BRKLV6`, `BRKACC`, and `BRKSEQ`, including `BRKMRN`.

In HABADTE's migration, implement optional joins to OMPMAST and OAPIRNK keyed by level and account.

## (9) Counting Rules

HABADTE's interpretation summary mentions three rules (BR-017–BR-019) with high confidence, which drive count logic for eligible records.

| Counter Name         | Incremented When                                      | Description                                         |
|----------------------|-------------------------------------------------------|-----------------------------------------------------|
| totalRecordsRead     | Each record read from HAPTRFR                         | Raw volume of transfer records encountered.        |
| totalRecordsSkipped  | Any of BR-017, BR-018, or BR-019 conditions are true  | Number of transfers excluded from output.          |
| totalRecordsIncluded | None of BR-017–BR-019 conditions are true             | Number of transfers included in the extract.       |

Relationship and business meaning:
- `totalRecordsRead = totalRecordsSkipped + totalRecordsIncluded`.
- Skipped records represent voided, outpatient, or structurally invalid transfers that should not appear in inpatient analytics.

## (10) Output Data Structure

Based on HAPTRFR and enrichment files, the output is a header-detail-footer structure.

### 10.1 Header Fields

| Field              | Source      | Description                                      |
|--------------------|------------|--------------------------------------------------|
| level6Description  | HXPLVL6    | Text description of AFLVL6 hierarchy level.      |
| accountNumber      | HAPTRFR    | Patient account identifier (AFACCT).             |
| patientMRN         | HAPTRFR    | Medical record number (AFMRNO).                  |
| extractFromDate    | Request    | FromDate parameter.                              |
| extractToDate      | Request    | ToDate parameter.                                |

### 10.2 Detail Row Fields

| Field               | Source      | Description                                      |
|---------------------|------------|--------------------------------------------------|
| transferDate        | HAPTRFR    | AFTRDT (date of transfer).                      |
| transferTime        | HAPTRFR    | AFTRTM (time of transfer).                      |
| transferType        | HAPTRFR    | AFTYPE code.                                    |
| statusCode          | HXPNSTN    | XFNSST status code.                             |
| statusDescription   | HXPNSTN    | Descriptive text for status.                    |
| benefitNumber       | OXPBNFIT   | XFBUBN benefit number.                          |
| benefitPlan         | OXPBNFIT   | XFBPLN plan code.                               |
| benefitPhone        | OXPBNFIT   | XFBTEL contact phone (PHI).                     |

### 10.3 Footer/Summary Fields

| Field               | Source           | Description                                         |
|---------------------|-----------------|-----------------------------------------------------|
| totalRecordsRead     | Derived          | Count of all transfer records scanned.             |
| totalRecordsSkipped  | Derived          | Count of skipped records per BR-017–019.           |
| totalRecordsIncluded | Derived          | Count of included records.                         |

### 10.4 Sort Order

The output is sorted chronologically by transfer date and time:

```sql
ORDER BY transferDate ASC, transferTime ASC, transferType ASC;
```

## (11) Complete Processing Flow (Step-by-Step)

```text
STEP 1: Init
  - Read input parameters: level6, accountNumber, fromDate, toDate.
  - Initialize counters: totalRecordsRead, totalRecordsSkipped, totalRecordsIncluded.
  - Establish database connections and security context for PHI access.

STEP 2: Preferences/Lookup
  - Optionally read configuration from XFXTABL (data maintenance tables).
  - Load any preferences controlling inclusion of outpatient, voided, or special flags.

STEP 3: Context
  - Derive organizational context via HXPLVL6 (if available) for AFLVL6.
  - Determine whether additional context from HMLMAST5H/TMPMAST or HAPIRNK/TAPIRNK is required.

STEP 4: Query (primary transfer selection)
  - Build primary SQL query over HAPTRFR keyed by AFLVL6 and AFACCT.
  - Apply date range and BR-017–BR-019 filters:
    SELECT ... FROM dbo.HAPTRFR
    WHERE AFLVL6 = :level6
      AND AFACCT = :accountNumber
      AND AFTRDT BETWEEN :fromDate AND :toDate
      AND fileIndicator <> 0              -- BR-017
      AND flagIndicator NOT IN ('VOID','VOIDED') -- BR-018
      AND inpatientOutpatientFlag <> 'O' -- BR-019
    ORDER BY AFTRDT ASC, AFTRTM ASC, AFTYPE ASC;

STEP 5: Per-Record Enrichment
  5a: Benefit plan lookup (Section 6)
    - For each record, look up OXPBNFIT by benefitNumber/planCode.
    - Attach benefitPhone and coverage attributes.

  5b: Patient status lookup (Section 7)
    - For each record, look up OXPNSTN by level6/statusCode.
    - Attach statusDescription and classification.

  5c: Optional master/risk context (Section 8)
    - If configured, join to OMPMAST and OAPIRNK for additional context.

STEP 6: Assemble Response
  - Map enriched fields into the header/detail/footer output structure.
  - Serialize to JSON for API response.
  - Return the assembled transfer detail extract to the client.
```

## (12) Data Type Conversions

### 12.1 Date Fields

AS400 date fields such as `AFTRDT` and admit dates in TMPMAST are stored as DECIMAL(8,0) in `YYYYMMDD` format.
- Conversion to Java `LocalDate`:
  - If value is 0, treat as `null`.
  - Otherwise, parse using `DateTimeFormatter.BASIC_ISO_DATE`.

### 12.2 Time Fields

Time fields such as `AFTRTM` are stored as DECIMAL(4,0) in `HHMM` format.
- Conversion to Java `LocalTime`:
  - Parse hours as `value / 100`, minutes as `value % 100`.

### 12.3 Packed Decimal Keys

Key fields like account numbers and sequence numbers are DECIMAL with zero scale.
- Map to Java `Long` or `Integer` depending on precision.

### 12.4 String Trimming

Fixed-length character fields (right-padded with spaces) must be trimmed.
- Apply `String.trim()` or `StringUtils.trim()` in Java before exposing values in JSON to avoid trailing spaces.

## (13) SQL Server Table Mapping

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

### Suggested SQL Server Indexes

```sql
-- Primary query index for HAPTRFR
CREATE INDEX IX_HAPTRFR_AccountLevelDateTime
ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE);

-- Benefit plan lookup
CREATE INDEX IX_OXPBNFIT_BenefitPlan
ON dbo.OXPBNFIT (XFBUBN, XFBPLN);

-- Patient status lookup
CREATE INDEX IX_OXPNSTN_LevelStatus
ON dbo.OXPNSTN (XFNLV6, XFNSST);

-- Patient master
CREATE INDEX IX_OMPMAST_LevelAccount
ON dbo.OMPMAST (MMPLV6, MMACCT);

-- Risk ranking
CREATE INDEX IX_OAPIRNK_LevelAccountSeq
ON dbo.OAPIRNK (BRKLV6, BRKACC, BRKSEQ);
```

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

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

### 14.2 Layer Structure

- **Controller**: `PatientTransferController` (exposes REST endpoint, validates inputs).
- **Service**: `PatientTransferService` (implements BR-017–BR-019 logic and enrichment steps).
- **Repository**: `HaptrfrRepository`, `BenefitPlanRepository`, `PatientStatusRepository`, `PatientMasterRepository`, `RiskRankingRepository` using Spring Data JPA or JDBC.

### 14.3 Response JSON Shape

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

### 14.4 Java Entity/DTO Sketch

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

## (15) Performance Considerations

The per-record enrichment pattern (benefit plan lookup and status lookup per transfer) introduces N+1 query risk if implemented with naive row-by-row calls.

Recommended approach:
- Use a single query joining HAPTRFR to OXPBNFIT and OXPNSTN:

```sql
SELECT t.*, b.XFBUBN, b.XFBPLN, b.XFBTEL, s.XFNSST, /* status description */
FROM dbo.HAPTRFR AS t
LEFT JOIN dbo.OXPBNFIT AS b
  ON b.XFBUBN = t.benefitNumber
 AND b.XFBPLN = t.planCode
LEFT JOIN dbo.OXPNSTN AS s
  ON s.XFNLV6 = t.AFLVL6
 AND s.XFNSST = t.statusCode
WHERE t.AFLVL6 = :level6
  AND t.AFACCT = :accountNumber
  AND t.AFTRDT BETWEEN :fromDate AND :toDate
  AND t.fileIndicator <> 0
  AND t.flagIndicator NOT IN ('VOID','VOIDED')
  AND t.inpatientOutpatientFlag <> 'O';
```

This reduces the number of database round-trips and ensures consistent application of inclusion/exclusion rules.

## (16) Business Rules Reference Summary

| Rule ID | Description                                                           |
|---------|-----------------------------------------------------------------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to `SKIP`.                   |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to `SKIP`.           |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP`. |

## (17) Edge Cases to Implement

| Scenario                          | Expected Behavior                                                  |
|-----------------------------------|--------------------------------------------------------------------|
| Date field value is 0             | Treat as null; exclude from date range checks where appropriate.  |
| Time field value is 0             | Treat as midnight (00:00) or null based on business guidance.     |
| Benefit plan lookup not found     | Set benefit fields to null; mark coverageStatus = 'UNKNOWN'.      |
| Status lookup not found           | Use default statusDescription = 'UNKNOWN STATUS'; log a warning.  |
| Logical delete flag on lookup row | Treat row as not found; do not attach deleted data.              |
| All records skipped               | Return empty transfers array with summary counts indicating zero included. |
| Preferences not configured        | Apply HABADTE default behavior (inpatient only, skip voided).     |
