# Business Processing Rules & Functional Specification

## Patient Transfer XML Export (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

HABADTE is the primary batch driver for patient transfers in the PATIENT_MANAGEMENT domain. It reads patient transfer records, evaluates a set of inclusion and exclusion rules, enriches each record with level, status, benefit, and ranking data, and generates XML output records for downstream systems and reporting consumers.

Operationally, the process is executed as part of nightly or on-demand transfer runs. Typical consumers are patient accounting, bed management, and external integration services requiring a consolidated view of patient transfer history in XML form.

> **Core business question:** "For a given cohort of patient accounts, which transfers are valid and reportable, and how should each be represented and enriched in the outbound XML feed?"

The process produces:

- A set of XML header records (HXFXMLH / HXPXMLR) representing transfer batches and sequences.
- XML detail records (HXFXMLD / HXPXMLD) describing individual transfer events.
- Updated status indicators via XFFNSTN (status table) and related DDS structures.

---

## (2) Inputs (API Request Parameters)

In the modernized Spring Boot API, HABADTE will be exposed as a service that accepts a transfer selection request.

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
- Log all requests, including user ID, parameters, and result counts, to a PHI audit trail because XML output contains PHI-bearing fields from HAPTRFR, OMPMAST, HXPDICT, and related files.

---

## (3) Organizational Hierarchy

HABADTE operates primarily on account and level codes and does not model a full organizational hierarchy (such as facility → region → network) explicitly in the available metadata.

Given the compact schema, no dedicated organizational hierarchy table (e.g., hospital hierarchy) was identified beyond level tables HXPLVL1–HXPLVL6.

| Level Number | Name | Key Size (digits) | AS400 Table |
|-------------:|------|-------------------|-------------|
| 6 | Plan / Level 6 | variable | HXPLVL6 |

**BR Note (Report Header Format):**

When HABADTE generates XML headers, it must include the Level 6 identifier (AFLVL6 / MMPLV6) from HAPTRFR or OMPMAST, mapped through HXPLVL6 to obtain a human-readable plan/level description.

---

## (4) Patient Data Source

### 4.1 Data Access Pattern

The primary business entity is the **patient transfer record**, stored in physical file **HAPTRFR** with record format **HAFTRFR** and key fields:

- AFLVL6 (plan/level)
- AFACCT (account number)
- AFTRDT (transfer date)
- AFTRTM (transfer time)
- AFTYPE (transfer type)

In AS400, HABADTE declares HAPTRFR and reads records keyed by the above fields, often iterating by AFLVL6 and AFTRDT for a given run.

The typical access pattern is:

- Keyed reads by (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE) for specific accounts.
- Sequential scans over a subset where AFLVL6 and AFTRDT fall within the run criteria.

**SQL Server equivalent SELECT:**

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFTRDT,
    AFTRTM,
    AFTYPE,
    AFMRNO,
    /* other non-PHI fields: status, flags, control */
FROM HABADTE_HAPTRFR
WHERE AFTRDT = @runDate
  AND AFTRTM BETWEEN @runTimeFrom AND @runTimeTo
  AND AFLVL6 = @level6
ORDER BY AFLVL6, AFTRDT, AFTRTM, AFACCT;
```

### 4.2 Key Fields Table

| SQL Column | AS400 Field | Type | Description |
|-----------|-------------|------|-------------|
| aflvl6 | AFLVL6 | string | Plan/Level 6 code used to segment transfer records. |
| afacct | AFACCT | string | Patient account number; PHI-sensitive (`AccountNumber`). |
| aftrdt | AFTRDT | date | Transfer date. |
| aftrtm | AFTRTM | time | Transfer time (HHMM). |
| aftype | AFTYPE | string | Transfer type (e.g., admission, discharge, internal move). |
| afmrno | AFMRNO | string | Medical record number (MRN); PHI-sensitive. |

---

## (5) Inclusion and Exclusion Rules

HABADTE enforces three core inclusion/exclusion rules on each candidate transfer record.

The rules below are derived from approved rules BR-017, BR-018, and BR-019, along with the HABADTE narrative.

### BR-017 – File Indicator Required

**Description:**

When the file indicator for the transfer record is zero (i.e., the record is not marked for processing), the program branches to `SKIP` and does not include the record in XML output.

**Pseudocode:**

```pseudo
IF fileIndicator = 0 THEN
    markRecordAsSkipped(reason = "FILE_INDICATOR_ZERO")
    GOTO SKIP
ENDIF
```

**SQL WHERE clause fragment:**

```sql
file_indicator <> 0
```

### BR-018 – Voided Transfers Excluded

**Description:**

If the flag indicator on the transfer record equals `void` or `voided`, the record is excluded from processing and XML export.

**Pseudocode:**

```pseudo
IF flagIndicator IN ('VOID', 'VOIDED') THEN
    markRecordAsSkipped(reason = "VOIDED_TRANSFER")
    GOTO SKIP
ENDIF
```

**SQL WHERE clause fragment:**

```sql
flag_indicator NOT IN ('VOID', 'VOIDED')
```

### BR-019 – Outpatient Transfers Skipped in Inpatient Runs

**Description:**

For runs configured to process inpatient transfers only, records with an INPATIENT/OUTPATIENT flag equal to `OUTPATIENT` are skipped.

**Pseudocode:**

```pseudo
IF inpatientOnly = TRUE AND ioFlag = 'OUTPATIENT' THEN
    markRecordAsSkipped(reason = "OUTPATIENT_IN_INPATIENT_RUN")
    GOTO SKIP
ENDIF
```

**SQL WHERE clause fragment:**

```sql
(@inpatientOnly = 0 OR io_flag <> 'OUTPATIENT')
```

### Summary SQL WHERE Clause

```sql
WHERE AFTRDT = @runDate
  AND AFTRTM BETWEEN @runTimeFrom AND @runTimeTo
  AND AFLVL6 = @level6
  AND file_indicator <> 0          -- BR-017
  AND flag_indicator NOT IN ('VOID', 'VOIDED')  -- BR-018
  AND (@inpatientOnly = 0 OR io_flag <> 'OUTPATIENT')  -- BR-019
ORDER BY AFLVL6, AFTRDT, AFTRTM, AFACCT;
```

---

## (6) Level Description Enrichment

The first enrichment step is a **level description lookup** performed by XFXLDSC over HXPLVL1–HXPLVL6 and their record formats (HXFLVL1–HXFLVL6).

### BR and Algorithm Steps

- HABADTE calls **XFXLDSC** with a level code (e.g., AFLVL6).
- XFXLDSC validates that the code is within the valid range (BR-009 to BR-012: "Level lookup: reject if level code exceeds valid range").
- The program declares and reads the HXPLVL* tables and corresponding HXFLVL* formats.
- If the code is invalid, enrichment fails and HABADTE either logs an error or uses a default description.

**Algorithm:**

```pseudo
INPUT levelCode

IF levelCode < MIN_LEVEL OR levelCode > MAX_LEVEL THEN
    return error("LEVEL_CODE_OUT_OF_RANGE")  // BR-009..BR-012
ENDIF

// Determine which HXPLVL table to use (1–6) based on code family
tableName = resolveLevelTable(levelCode)  // e.g., HXPLVL6 for level 6

row = SELECT * FROM tableName WHERE HXxNUM = levelCode
IF row NOT FOUND THEN
    return error("LEVEL_NOT_FOUND")
ENDIF

RETURN row.descriptionFields
```

**SQL SELECT equivalent:**

```sql
SELECT HX6NUM, /* description fields */
FROM HABADTE_HXPLVL6
WHERE HX6NUM = @levelCode;
```

**Edge cases:**

- **Not-found:** If no row exists for the given code, HABADTE should mark the record as having an unknown level description and still generate XML, but with a placeholder description.
- **Delete-flag logic:** If level tables carry delete or inactive flags (not explicit in compact schema), the enrichment should ignore rows marked inactive.
- **Derived flags:** If level tables carry benefit or status indicators, they can be used to set derived flags in the XML payload.

---

## (7) Status Enrichment

The second enrichment step is status lookup using **HXPNSTN** (logical file over TXPNSTN) and XFFNSTN.

### BR and Algorithm Steps

- HABADTE declares HXPNSTN and reads status records keyed by level (XFNLV6) and status (XFNSST).
- The XFFNSTN record read from HABADTE is used to determine transfer status and any downstream routing.

**Algorithm:**

```pseudo
INPUT level6, statusCode

row = SELECT * FROM HABADTE_OXPNSTN
WHERE XFNLV6 = level6 AND XFNSST = statusCode

IF row NOT FOUND THEN
    statusDesc = "UNKNOWN_STATUS"
ELSE
    statusDesc = row.description
ENDIF

RETURN statusDesc
```

**SQL SELECT equivalent:**

```sql
SELECT XFNLV6, XFNSST, /* description fields */
FROM HABADTE_OXPNSTN
WHERE XFNLV6 = @level6 AND XFNSST = @statusCode;
```

**Edge cases:**

- If status code is void/voided, BR-018 will already have excluded the record.
- If no status is found, the XML should carry an explicit "UNKNOWN" or omit the status element.

---

## (8) Benefit and Rank Enrichment

The third enrichment step is **benefit and rank enrichment**, performed via HXPBNFIT (logical over TXPBNFIT), OXPBNFIT, HAPIRNK (logical over TAPIRNK), and OAPIRNK.

Because the lineage shows PFILE_OF relationships TAPIRNK → HAPIRNK and TXPBNFIT → HXPBNFIT, HABADTE uses these LFs to read benefit and ranking data.

### Algorithm Steps

```pseudo
INPUT accountNumber, level6

// Benefit lookup via TXPBNFIT / OXPBNFIT
benefitRow = SELECT * FROM HABADTE_OXPBNFIT
WHERE XFBUBN = @benefitCode AND XFBPLN = @planCode;

// Rank lookup via TAPIRNK / OAPIRNK
rankRow = SELECT * FROM HABADTE_OAPIRNK
WHERE BRKLV6 = level6 AND BRKACC = accountNumber
ORDER BY BRKSEQ;

// Use benefitRow and rankRow to enrich XML
```

**Edge cases:**

- If phone numbers (XFBTEL) are present in benefit data, they must be treated as PHI and masked or limited based on consumer requirements.
- Rank sequences may be empty; XML should then omit rank sections.

---

## (9) Counting Rules

Counting rules are derived from key rules in interpretations_detail and HABADTE’s behavior, even though explicit counter rules are in XFXCNTR.

| Counter Name | Incremented When | Description |
|-------------|------------------|-------------|
| processedTransferCount | A transfer record passes BR-017/018/019 and is enriched successfully. | Total number of transfers included in XML output. |
| skippedFileIndicatorZero | fileIndicator = 0 (BR-017). | Number of transfers skipped because they were not marked for processing. |
| skippedVoidedTransfers | flagIndicator IN ('VOID', 'VOIDED') (BR-018). | Number of transfers skipped due to void/voided flags. |
| skippedOutpatientInInpatientRun | inpatientOnly = TRUE AND ioFlag = 'OUTPATIENT' (BR-019). | Number of transfers skipped because they are outpatient in an inpatient-only run. |

**Relationship and business meaning:**

- `processedTransferCount` + all skipped counters should equal the total number of candidate records read from HAPTRFR.
- The distribution of skipped reasons is a key quality metric and should be surfaced in reports or monitoring dashboards.

---

## (10) Output Data Structure

HABADTE writes XML-like records using HXFXMLH/HXPXMLR for headers and HXFXMLD/HXPXMLD for details.

### 10.1 Header Fields

| Field | Source | Description |
|------|--------|-------------|
| batchId | HXFXMLH/XMRSEQ | Unique sequence for the batch of transfers. |
| userId | HXFXMLH/XMRUSR | User or system identifier initiating the run. |
| recordId | HXFXMLH/XMRID | Header record identifier. |
| runDate | HAPTRFR/AFTRDT | Transfer run date. |
| level6 | HAPTRFR/AFLVL6 | Level 6 plan/facility code. |

### 10.2 Detail Row Fields

| Field | Source | Description |
|------|--------|-------------|
| accountNumber | HAPTRFR/AFACCT | Patient account number. |
| mrn | HAPTRFR/AFMRNO | Medical record number (MRN). |
| transferDate | HAPTRFR/AFTRDT | Transfer date. |
| transferTime | HAPTRFR/AFTRTM | Transfer time. |
| transferType | HAPTRFR/AFTYPE | Transfer type. |
| levelDescription | HXPLVL6/HXFLVL6 | Resolved level description from level tables. |
| statusDescription | OXPNSTN/XFFNSTN | Resolved status description. |
| benefitCode | OXPBNFIT/XFBUBN | Benefit code if enrichment succeeds. |
| benefitPlan | OXPBNFIT/XFBPLN | Benefit plan code. |

### 10.3 Footer/Summary Fields

| Field | Source | Description |
|------|--------|-------------|
| processedTransferCount | Derived counter | Total processed transfers. |
| skippedFileIndicatorZero | Derived counter | Skipped due to file indicator zero. |
| skippedVoidedTransfers | Derived counter | Skipped due to void/voided flag. |
| skippedOutpatientInInpatientRun | Derived counter | Skipped due to outpatient flag. |

### 10.4 Sort Order

Detail records in XML are ordered primarily by:

1. Level 6 (AFLVL6)
2. Transfer date (AFTRDT)
3. Transfer time (AFTRTM)
4. Account number (AFACCT)

---

## (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
    - Read job parameters: runDate, runTimeFrom, runTimeTo, level6, inpatientOnly, includeVoided, fileIndicatorRequired.
    - Initialize counters: processedTransferCount, skippedFileIndicatorZero, skippedVoidedTransfers, skippedOutpatientInInpatientRun.
    - Initialize XML header sequence from HXFXMLH/HXPXMLR using XFXGETID.

STEP 2: Preferences/Lookup
    - Pre-load level tables HXPLVL1–HXPLVL6 for quick lookup (via XFXLDSC).
    - Pre-load status table OXPNSTN/XFFNSTN.
    - Pre-load benefit table OXPBNFIT.

STEP 3: Context
    - Establish security context (userId from Spring Security principal).
    - Resolve facility/plan context from level6.

STEP 4: Query
    - SELECT candidate transfer records from HABADTE_HAPTRFR:
        WHERE AFTRDT = @runDate
          AND AFTRTM BETWEEN @runTimeFrom AND @runTimeTo
          AND AFLVL6 = @level6;

STEP 5: Per-Record Enrichment

    For each transfer record:

    5a: Apply inclusion/exclusion rules
        - If file_indicator = 0 (BR-017) then
              skippedFileIndicatorZero++
              CONTINUE
        - If flag_indicator IN ('VOID', 'VOIDED') (BR-018) then
              skippedVoidedTransfers++
              CONTINUE
        - If inpatientOnly = TRUE AND io_flag = 'OUTPATIENT' (BR-019) then
              skippedOutpatientInInpatientRun++
              CONTINUE

    5b: Level description enrichment (XFXLDSC)
        - Validate level6 code against HXPLVL6.
        - Read HXPLVL6/HXFLVL6 row to obtain description.

    5c: Status enrichment
        - Read OXPNSTN/XFFNSTN row for (level6, statusCode).

    5d: Benefit and rank enrichment
        - Read OXPBNFIT row for benefit/plan codes.
        - Read OAPIRNK/HAPIRNK row(s) for ranking based on accountNumber and level6.

    5e: XML header/detail creation
        - Use XFXGETID to obtain XML record IDs via HXFXMLR.
        - Write header record to HXFXMLH/HXPXMLR if needed.
        - Write detail record to HXFXMLD/HXPXMLD with enriched fields.

    5f: Counters
        - processedTransferCount++ for each successfully written detail record.

STEP 6: Assemble Response
    - Commit XML records (HXFXMLH/HXFXMLD) to database.
    - Return summary including counters and run parameters in API response.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields

- AS400 DECIMAL 8,0 dates (e.g., AFTRDT, WBDATE) become `LocalDate`.
- Value `0` represents "no date" and must be mapped to `null`.

### 12.2 Time Fields

- AS400 DECIMAL 4,0 time (HHMM) (e.g., AFTRTM) becomes `LocalTime`.
- Convert `HHMM` to `HH:MM` format; `0000` can be treated as midnight or `null` based on business decision.

### 12.3 Packed Decimal Keys

- Key fields like AFLVL6 or account ranges stored as DECIMAL should become `Long` or `Integer` in Java.

### 12.4 String Trimming

- Fixed-length, right-padded strings from DDS (e.g., patient name, account numbers) must be trimmed using `.trim()` in Java before comparison or display.

---

## (13) SQL Server Table Mapping

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

### Suggested SQL Server Indexes

```sql
-- Primary query on HAPTRFR
CREATE INDEX IX_HAPTRFR_RUN
    ON HABADTE_HAPTRFR (AFTRDT, AFTRTM, AFLVL6, AFACCT);

-- Status lookup
CREATE INDEX IX_OXPNSTN_LEVEL_STATUS
    ON HABADTE_OXPNSTN (XFNLV6, XFNSST);

-- Level lookup
CREATE INDEX IX_HXPLVL6_LEVEL
    ON HABADTE_HXPLVL6 (HX6NUM);

-- Benefit lookup
CREATE INDEX IX_OXPBNFIT_BENEFIT_PLAN
    ON HABADTE_OXPBNFIT (XFBUBN, XFBPLN);

-- Rank lookup
CREATE INDEX IX_OAPIRNK_LEVEL_ACCOUNT
    ON HABADTE_OAPIRNK (BRKLV6, BRKACC, BRKSEQ);

-- XML header/detail mappings
CREATE INDEX IX_HXPXMLR_USER_SEQ
    ON HABADTE_HXPXMLR (XMRUSR, XMRSEQ);

CREATE INDEX IX_HXPXMLD_USER_SEQ
    ON HABADTE_HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

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

### 14.2 Layer Structure

- **Controller:** `PatientTransferExportController`
  - Accepts HTTP requests, performs basic validation, and delegates to service layer.
- **Service:** `PatientTransferExportService`
  - Implements steps 1–6 of the processing flow, orchestrating repositories and utilities.
- **Repositories:**
  - `HaptrfrRepository` – access to HABADTE_HAPTRFR.
  - `OxpnstnRepository` – access to HABADTE_OXPNSTN.
  - `OxpbnfitRepository` – access to HABADTE_OXPBNFIT.
  - `OapirnkRepository` – access to HABADTE_OAPIRNK.
  - `HxplvlRepository` – access to HABADTE_HXPLVL* tables.
  - `XmlHeaderRepository` – access to HABADTE_HXPXMLR.
  - `XmlDetailRepository` – access to HABADTE_HXPXMLD.

### 14.3 Response JSON Shape

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

### 14.4 Java Entity/DTO Sketch

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

---

## (15) Performance Considerations

The original AS400 implementation performs per-record enrichment using multiple lookups:

- Level lookup via HXPLVL*.
- Status lookup via OXPNSTN/XFFNSTN.
- Benefit and rank lookup via OXPBNFIT/OAPIRNK.

If implemented as one query per record, this can lead to N+1 query patterns.

### Recommended Approach

- Use **JOINs** where possible to retrieve data in a single query.
- Preload reference data into in-memory maps when joins would be too complex.

**Example SQL LEFT JOIN:**

```sql
SELECT
    t.AFACCT,
    t.AFMRNO,
    t.AFTRDT,
    t.AFTRTM,
    t.AFTYPE,
    l.HX6DESC,
    s.STATUS_DESC,
    b.XFBUBN,
    b.XFBPLN
FROM HABADTE_HAPTRFR t
LEFT JOIN HABADTE_HXPLVL6 l
  ON l.HX6NUM = t.AFLVL6
LEFT JOIN HABADTE_OXPNSTN s
  ON s.XFNLV6 = t.AFLVL6
 AND s.XFNSST = t.STATUS_CODE
LEFT JOIN HABADTE_OXPBNFIT b
  ON b.XFBUBN = t.BENEFIT_CODE
 AND b.XFBPLN = t.BENEFIT_PLAN
WHERE t.AFTRDT = @runDate
  AND t.AFTRTM BETWEEN @runTimeFrom AND @runTimeTo
  AND t.AFLVL6 = @level6
  AND t.file_indicator <> 0
  AND t.flag_indicator NOT IN ('VOID', 'VOIDED')
  AND (@inpatientOnly = 0 OR t.io_flag <> 'OUTPATIENT');
```

---

## (16) Business Rules Reference Summary

| Rule ID | Description |
|--------:|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

Additional rules from utility programs used by HABADTE:

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

---

## (17) Edge Cases to Implement

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
