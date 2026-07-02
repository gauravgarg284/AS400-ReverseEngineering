# Business Processing Rules & Functional Specification

## HABADTE Admission Screening Report (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

The HABADTE admission screening process evaluates transfer and status records to decide which encounters are eligible for inpatient admission workflows. It reads transfer history, plan status and reference dictionaries, and produces XML-based outputs for downstream systems.

The process is used by clinical admission staff and back-office teams whenever patient transfer records are processed into the inpatient admission subsystem.

> Core business question: *"For a given transfer record and patient status, should this encounter be included in the inpatient admission workflow or skipped as void, inactive or outpatient?"*

Summary outputs:
- XML header and detail records describing accepted admissions.
- Per-record flags indicating whether encounters were skipped due to file indicator, void status or outpatient status.
- Derived status mappings for plans and levels used in downstream processing.

---

## (2) Inputs (API Request Parameters)

In the modernized API, HABADTE will be exposed as a REST endpoint that accepts filter criteria and context required to drive admission screening.

| Parameter                    | AS400 Field / Concept     | Type        | Description |
|------------------------------|---------------------------|-------------|-------------|
| transferLevel6               | AFLVL6                    | integer     | Level-6 transfer classification used to group transfer records. |
| accountNumber                | AFACCT                    | long        | Patient account number; identifies the financial account associated with the encounter. |
| transferDate                 | AFTRDT                    | string (CYMD) | Transfer date in calendar-YYYYMMDD format; must pass date validation rules. |
| transferTime                 | AFTRTM                    | string (HHMMSS) | Transfer time; combined with transferDate to sequence events. |
| transferType                 | AFTYPE                    | string      | Transfer type code driving specific admission flows. |
| inpatientOutpatientFlag      | domain flag (INPAT/OUTPAT)| string      | Indicates whether the encounter is inpatient or outpatient. |
| fileIndicator                | -FILE INDICATOR           | integer     | Binary/indicator field used to mark records as active (non-zero) or inactive (zero). |
| voidFlag                     | -FLAG INDICATOR           | string      | Flag indicating void/voided status of the record. |
| planStatusLevel6             | XFNLV6                    | integer     | Level-6 code linking to plan status in OXPNSTN/XFFNSTN. |
| planStatusCode               | XFNSST                    | string      | Plan status code; drives status mapping from plan status master. |
| userId                       | XMDUSR/XMRUSR             | string      | User identifier recorded in XML header/detail workfiles. |
| sequenceNumber               | XMDSEQ/XMRSEQ             | integer     | Sequence number tying together XML messages for the same request. |

Additional input: **current user identity from security context** (principal, roles, tenant) will be captured from the Spring Security/OAuth2 context, not explicitly in the request.

Security mapping note:
- OAuth2 bearer tokens will be used to authenticate callers.
- RBAC roles (e.g., ROLE_ADMISSIONS_CLERK, ROLE_BILLING_ADMIN) will restrict which users can invoke HABADTE processing and see PHI-bearing fields.
- All invocations will be audited with PHI access logs including accountNumber and MRN references sourced from HAPTRFR, OMPMAST and HXPDICT.

---

## (3) Organizational Hierarchy

HABADTE primarily operates on transfer and plan status records keyed by level-6 codes and account numbers. No explicit organizational hierarchy (e.g., hospital → region → department) is encoded directly in the approved rules or compact schema.

| Level | Name              | Key Size Digits | AS400 Table |
|-------|-------------------|-----------------|-------------|
| 1     | Admission Level 6 | 6               | HAPTRFR.AFLVL6 / OXPNSTN.XFNLV6 |

BR note: When level-6 codes are used in report headers (e.g., plan status summaries), they act as organizational buckets for admissions, but there is no multi-tier org hierarchy beyond this numeric level.

---

## (4) Patient Data Source

### 4.1 Data Access Pattern

The primary entity is the **patient admission/transfer record**, and the main program HABADTE reads the physical file **HAPTRFR**.

Access pattern:
- Primary file: `HAPTRFR` (record format `HAFTRFR`).
- Read mode: keyed access by `(AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE)` to retrieve transfers for a given level, account and time window.
- Sort order: ascending by `AFTRDT`, then `AFTRTM`, within `AFACCT` and `AFLVL6`, to process chronological transfer events.

SQL Server equivalent:

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFTRDT,
    AFTRTM,
    AFTYPE,
    /* additional fields omitted for brevity */
FROM dbo.HAPTRFR
WHERE AFLVL6 = @transferLevel6
  AND AFACCT = @accountNumber
  AND AFTRDT BETWEEN @fromDate AND @toDate
ORDER BY AFTRDT ASC, AFTRTM ASC;
```

### 4.2 Key Fields

Key fields for the primary table:

| SQL Column        | AS400 Field | Type    | Description |
|-------------------|------------|---------|-------------|
| AFLVL6            | AFLVL6     | INT     | Level-6 transfer classification code. |
| AccountNumber     | AFACCT     | BIGINT  | Account number representing the financial account associated with the encounter. |
| TransferDate      | AFTRDT     | INT     | Transfer date in CYMD numeric format; used for date-based filtering and ordering. |
| TransferTime      | AFTRTM     | INT     | Transfer time in HHMMSS numeric format; used to sequence transfers. |
| TransferTypeCode  | AFTYPE     | VARCHAR | Transfer type code controlling specific processing branches. |

---

## (5) Inclusion and Exclusion Rules

Inclusion/exclusion logic is driven by HABADTE’s key rules BR-017, BR-018 and BR-019.

### BR-017 – File Indicator Zero → Skip

**Description**: Records with a file indicator equal to zero are treated as inactive and skipped from admission processing.

**Pseudocode:**

```pseudo
for each transfer in HAPTRFR:
    if FILE_INDICATOR = 0 then
        mark transfer as SKIPPED_REASON = 'FILE_INDICATOR_ZERO'
        continue to next transfer
    else
        proceed with further checks
```

**SQL WHERE fragment:**

```sql
WHERE FILE_INDICATOR <> 0
```

### BR-018 – Void/VoidFlag → Skip

**Description**: Records flagged as void or voided are excluded from processing to prevent cancelled encounters from entering the admission workflow.

**Pseudocode:**

```pseudo
if FLAG_INDICATOR in ('VOID', 'VOIDED') then
    mark transfer as SKIPPED_REASON = 'VOID_RECORD'
    continue to next transfer
```

**SQL WHERE fragment:**

```sql
AND FLAG_INDICATOR NOT IN ('VOID', 'VOIDED')
```

### BR-019 – Outpatient Flag → Skip

**Description**: When the inpatient/outpatient flag indicates outpatient, the record is skipped because HABADTE focuses on inpatient admission.

**Pseudocode:**

```pseudo
if INPATIENT_OUTPATIENT_FLAG = 'OUTPATIENT' then
    mark transfer as SKIPPED_REASON = 'OUTPATIENT'
    continue to next transfer
```

**SQL WHERE fragment:**

```sql
AND INPATIENT_OUTPATIENT_FLAG <> 'OUTPATIENT'
```

### Summary SQL WHERE Clause

Combining the three rules:

```sql
SELECT ...
FROM dbo.HAPTRFR
WHERE FILE_INDICATOR <> 0      -- BR-017
  AND FLAG_INDICATOR NOT IN ('VOID', 'VOIDED')  -- BR-018
  AND INPATIENT_OUTPATIENT_FLAG <> 'OUTPATIENT' -- BR-019
ORDER BY AFTRDT ASC, AFTRTM ASC;
```

---

## (6) Plan Status Enrichment (Primary Enrichment Step)

The first enrichment in HABADTE attaches plan status information from the plan status master accessed via logical file **HXPNSTN** over physical file **TXPNSTN** and DDS file **OXPNSTN / XFFNSTN**.

**BR context**: BR-019 ensures outpatient encounters are skipped, but accepted inpatient records need plan status resolution.

Algorithm steps:
1. For each accepted transfer (after BR-017–019), read the plan status key `(XFNLV6, XFNSST)` associated with the encounter.
2. Use logical file `HXPNSTN` (PFILE `TXPNSTN`) keyed by `(XFNLV6, XFNSST)` to locate the plan status row.
3. Retrieve status attributes (e.g., active/inactive, coverage indicators) from record format `XFFNSTN`.
4. Attach plan status attributes to the in-memory admission record for downstream XML generation.

SQL SELECT equivalent:

```sql
SELECT
    s.XFNLV6,
    s.XFNSST,
    s.StatusDescription,
    s.IsActive,
    s.CoverageCode
FROM dbo.PlanStatus s
WHERE s.XFNLV6 = @planStatusLevel6
  AND s.XFNSST = @planStatusCode;
```

Edge cases:
- **Not found**: If no row is found for `(XFNLV6, XFNSST)`, the admission record should be flagged with `STATUS_MISSING = true` and still processed, but downstream consumers may apply default rules.
- **Delete flags**: If a soft-delete or inactive flag is present on the status row, mark `IsActive = false` and optionally skip or downgrade coverage.
- **Derived flags**: Derived booleans such as `IsCoverageValid` or `IsStatusBillable` should be computed in the service layer based on status attributes.

---

## (7) Benefit Plan Enrichment (Secondary Enrichment Step)

The second enrichment uses benefit plan data through logical file **HXPBNFIT** over physical file **TXPBNFIT** and DDS file **OXPBNFIT / XFFBNFIT**.

Algorithm steps:
1. From each accepted transfer, derive benefit keys `(XFBUBN, XFBPLN)` if available.
2. Use `HXPBNFIT` to lookup the benefit row keyed by `(XFBUBN, XFBPLN)`.
3. Retrieve benefit attributes such as coverage limits, co-pay percentages and contact phone (`XFBTEL`).
4. Attach benefit attributes to the admission record.

SQL SELECT equivalent:

```sql
SELECT
    b.XFBUBN,
    b.XFBPLN,
    b.CoverageLimit,
    b.CopayPercent,
    b.XFBTEL AS BenefitPhone
FROM dbo.BenefitPlan b
WHERE b.XFBUBN = @benefitNumber
  AND b.XFBPLN = @planCode;
```

Edge cases:
- **Not found**: Mark `BENEFIT_MISSING = true`; do not block the admission record, but downstream billing must handle missing benefits.
- **Delete/inactive flags**: If benefits are inactive, mark `IsBenefitActive = false` and adjust billing workflows.
- **Derived flags**: Derived attributes like `IsCoverageSufficient` can be computed for later rule evaluation.

---

## (8) Level Mapping Enrichment (Tertiary Enrichment Step)

The third enrichment step resolves level codes into descriptive attributes using HXPLVL1–HXPLVL6 and XFXLDSC.

Algorithm steps:
1. For each admission record, use level codes (e.g., `HX1NUM`–`HX6NUM`) to read configuration from HXPLVL tables.
2. XFXLDSC declares and reads `HXPLVL1`–`HXPLVL6` and runtime formats `HXFLVL1`–`HXFLVL6`.
3. It applies mapping codes (LDAMAP) with rules BR-009–BR-012 to ensure valid mappings.
4. The resulting descriptions and attributes (e.g., pricing tiers, coverage categories) are attached to the admission record.

SQL SELECT equivalent (simplified):

```sql
SELECT
    l6.HX6NUM,
    l6.LevelDescription,
    l6.PricingTier,
    l5.HX5NUM,
    l5.CategoryCode
FROM dbo.Level6 l6
LEFT JOIN dbo.Level5 l5 ON l5.HX5NUM = l6.HX5NUM
WHERE l6.HX6NUM = @level6Num;
```

Edge cases:
- **Mapping out of range**: If `LDAMAP > 99` or `LDAMAP > 9999`, XFXLDSC branches to EXIT per BR-009–BR-012; in the modern system, treat such records as configuration errors and log them while applying safe defaults.
- **Missing level rows**: Mark `LEVEL_MISSING = true` and proceed with minimal attributes.

---

## (9) Counting Rules

Counting is primarily implemented in supporting utilities like XFXCNTR but conceptually applies to HABADTE’s workflow.

| Counter Name        | Incremented When                                      | Description |
|---------------------|-------------------------------------------------------|-------------|
| processedTransfers   | Every transfer record read from HAPTRFR              | Total number of transfers evaluated in the run. |
| skippedFileIndicator| FILE_INDICATOR = 0 (BR-017)                           | Number of records skipped due to inactive file indicator. |
| skippedVoidFlag     | FLAG_INDICATOR in ('VOID', 'VOIDED') (BR-018)        | Number of records skipped due to void status. |
| skippedOutpatient   | INPATIENT_OUTPATIENT_FLAG = 'OUTPATIENT' (BR-019)    | Number of records skipped as outpatient. |
| acceptedAdmissions   | Records that pass all three filters and enrichments  | Number of admissions entering XML generation.

Relationship and business meaning:
- `processedTransfers = skippedFileIndicator + skippedVoidFlag + skippedOutpatient + acceptedAdmissions` (ignoring other error paths).
- High values of skipped counters indicate data quality issues or mismatched workflow configuration.

---

## (10) Output Data Structure

XML header/detail outputs are written to `HXFXMLH` and `HXFXMLD` (via HXPXMLD/PF) with keys `XMDUSR`, `XMDSEQ`, `XMDSQ2`.

### 10.1 Header Fields

| Field        | Source         | Description |
|--------------|----------------|-------------|
| userId       | XMDUSR         | User who initiated the admission run. |
| sequence     | XMDSEQ         | Primary sequence number of the run. |
| subSequence  | XMDSQ2         | Secondary sequence identifier for batches. |
| runTimestamp | derived        | Timestamp of execution, from system clock. |

### 10.2 Detail Row Fields

| Field           | Source         | Description |
|-----------------|----------------|-------------|
| accountNumber   | AFACCT         | Account number from primary transfer record. |
| transferDate    | AFTRDT         | Transfer date after validation. |
| transferTime    | AFTRTM         | Transfer time. |
| transferType    | AFTYPE         | Transfer type. |
| planStatusCode  | XFNSST         | Plan status code from OXPNSTN/XFFNSTN. |
| benefitPlanCode | XFBPLN         | Benefit plan code from OXPBNFIT/XFFBNFIT. |
| level6Code      | AFLVL6/HX6NUM  | Level-6 code after mapping. |
| statusFlags     | derived        | Combined flags for file indicator, void and outpatient conditions. |

### 10.3 Footer/Summary Fields

| Field                  | Source             | Description |
|------------------------|--------------------|-------------|
| totalProcessedTransfers| processedTransfers | Count of evaluated records. |
| totalAcceptedAdmissions| acceptedAdmissions | Count of records that passed filters. |
| totalSkippedFile       | skippedFileIndicator| Count of records skipped due to file indicator. |
| totalSkippedVoid       | skippedVoidFlag    | Count of records skipped due to void status. |
| totalSkippedOutpatient | skippedOutpatient  | Count of records skipped due to outpatient flag. |

### 10.4 Sort Order

Final output rows should be sorted by:

1. AccountNumber (AFACCT)
2. TransferDate (AFTRDT)
3. TransferTime (AFTRTM)

---

## (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
    - Read request parameters (accountNumber, date range, level6Code, etc.).
    - Resolve current user identity from security context.
    - Initialize counters: processedTransfers, skippedFileIndicator, skippedVoidFlag, skippedOutpatient, acceptedAdmissions.

STEP 2: Preferences/Lookup
    - Load configuration for level mappings via XFXLDSC (HXPLVL1–HXPLVL6).
    - Load dictionary tables via XFXTABL (HXPTABLD, HXLTABLD, HXLTABLP, HXLTABLS).

STEP 3: Context
    - Construct an execution context object containing:
        - accountNumber, level6Code
        - date range (fromDate, toDate)
        - userId and sequence numbers (from XMDUSR/XMDSEQ)

STEP 4: Query (primary data fetch)
    - Execute SQL query against HAPTRFR:

      SELECT AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE, ...
      FROM dbo.HAPTRFR
      WHERE FILE_INDICATOR <> 0
        AND FLAG_INDICATOR NOT IN ('VOID', 'VOIDED')
        AND INPATIENT_OUTPATIENT_FLAG <> 'OUTPATIENT'
        AND AFACCT = @accountNumber
        AND AFTRDT BETWEEN @fromDate AND @toDate
      ORDER BY AFTRDT ASC, AFTRTM ASC;

STEP 5: Per-Record Enrichment
    For each row in the result set:

    5a: Plan Status Enrichment
        - Use XFNLV6/XFNSST to query dbo.PlanStatus.
        - Attach IsActive, StatusDescription, CoverageCode.
        - Handle not-found and inactive statuses.

    5b: Benefit Plan Enrichment
        - Use XFBUBN/XFBPLN to query dbo.BenefitPlan.
        - Attach coverage limits, copay percentages, benefit contact phone.

    5c: Level Mapping Enrichment
        - Pass level codes and LDAMAP to LevelService (XFXLDSC equivalent).
        - Enforce BR-009–BR-012 to guard against invalid mappings.
        - Attach mapped level descriptions and pricing tiers.

STEP 6: Assemble Response
    - For each enriched admission record:
        - Construct XML header/detail structures based on HXFXMLH/HXFXMLD layout.
        - Write rows to dbo.XmlHeader/dbo.XmlDetail tables.
    - Return a JSON response mirroring the XML output for API consumers.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields (DECIMAL 8,0 → LocalDate)

AS400 numeric dates (e.g., AFTRDT, WBDATE) are stored as DECIMAL(8,0) in CYMD format.

Conversion rules:
- Parse `YYYYMMDD` from the integer.
- If value is `0`, treat as `null`.
- Otherwise construct `java.time.LocalDate`.

### 12.2 Time Fields (DECIMAL 4–6,0 → LocalTime)

Time fields such as AFTRTM are stored as DECIMAL(6,0) in HHMMSS or DECIMAL(4,0) in HHMM.

Conversion rules:
- Extract hours, minutes (and seconds if present).
- Construct `java.time.LocalTime`.

### 12.3 Packed Decimal Keys (DECIMAL 6,0 → Long)

Level and status keys (e.g., AFLVL6, XFNLV6) are numeric fields used as identifiers.

Conversion rules:
- Map DECIMAL(6,0) to Java `long` or `int` based on range.
- Preserve leading zeros where semantically significant.

### 12.4 String Trimming

Fixed-length character fields in DDS are right-padded with spaces.

Conversion rules:
- Use `.trim()` when mapping to Java `String` and when serializing to JSON.
- Avoid trimming fields where trailing spaces are meaningful (rare in this domain).

---

## (13) SQL Server Table Mapping

Mapping AS400 objects to SQL Server tables:

| AS400 Object | SQL Server Table       | Purpose |
|--------------|------------------------|---------|
| HAPTRFR      | dbo.HAPTRFR            | Transfer/admission source records. |
| OMPMAST      | dbo.PatientMaster      | Patient master (MRN, account, demographics). |
| OAPIRNK      | dbo.TransferIndex      | Indexed break/transfer records by level and sequence. |
| OXPBNFIT     | dbo.BenefitPlan        | Benefit plan definitions and coverage details. |
| OXPNSTN      | dbo.PlanStatus         | Plan status master keyed by level and status code. |
| HXPDICT      | dbo.Dictionary         | Reference and dictionary values; PHI-bearing fields. |
| HXPXMLD      | dbo.XmlDetail          | XML detail workfile. |
| HXPXMLR      | dbo.XmlResponse        | XML response workfile. |

### Suggested SQL Server Indexes

```sql
-- Primary query index on HAPTRFR
CREATE INDEX IX_HAPTRFR_Main
    ON dbo.HAPTRFR (AFACCT, AFTRDT, AFTRTM, AFLVL6, AFTYPE);

-- Plan status lookup
CREATE INDEX IX_PlanStatus_Key
    ON dbo.PlanStatus (XFNLV6, XFNSST);

-- Benefit plan lookup
CREATE INDEX IX_BenefitPlan_Key
    ON dbo.BenefitPlan (XFBUBN, XFBPLN);

-- Patient master by account and MRN
CREATE INDEX IX_PatientMaster_AccountMrn
    ON dbo.PatientMaster (MMACCT, MMMRNO);

-- XML detail by user and sequence
CREATE INDEX IX_XmlDetail_UserSeq
    ON dbo.XmlDetail (XMDUSR, XMDSEQ, XMDSQ2);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

- Method: `POST`
- Path: `/api/admissions/screen`

Parameters (request body JSON):

| Name                    | Type      | Required | Validation |
|-------------------------|-----------|----------|-----------|
| accountNumber           | long      | yes      | > 0 |
| transferLevel6          | int       | yes      | 1–999999 |
| fromDate                | string    | yes      | YYYYMMDD, valid date |
| toDate                  | string    | yes      | YYYYMMDD, valid date, >= fromDate |
| inpatientOutpatientFlag | string    | no       | 'INPATIENT' or 'OUTPATIENT' |

### 14.2 Layer Structure

- **Controller**: `AdmissionScreenController`
  - Accepts REST requests and returns JSON responses.
- **Service**: `AdmissionScreenService`
  - Implements Steps 1–6 of the processing flow.
- **Repositories**:
  - `TransferRepository` (HAPTRFR)
  - `PlanStatusRepository` (PlanStatus)
  - `BenefitPlanRepository` (BenefitPlan)
  - `PatientMasterRepository` (PatientMaster)
  - `XmlDetailRepository` / `XmlHeaderRepository`

### 14.3 Response JSON Shape

```json
{
  "runId": "202607021710",
  "accountNumber": 1234567890,
  "fromDate": "20260101",
  "toDate": "20260131",
  "summary": {
    "processedTransfers": 120,
    "acceptedAdmissions": 80,
    "skippedFileIndicator": 10,
    "skippedVoidFlag": 20,
    "skippedOutpatient": 10
  },
  "admissions": [
    {
      "accountNumber": 1234567890,
      "transferDate": "20260110",
      "transferTime": "083000",
      "transferType": "TR",
      "planStatusCode": "ACT",
      "benefitPlanCode": "PLN01",
      "level6Code": 100001,
      "statusFlags": ["OK"]
    }
  ]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record AdmissionRequest(
    long accountNumber,
    int transferLevel6,
    String fromDate,
    String toDate,
    String inpatientOutpatientFlag
) {}

public record AdmissionSummary(
    int processedTransfers,
    int acceptedAdmissions,
    int skippedFileIndicator,
    int skippedVoidFlag,
    int skippedOutpatient
) {}

public record AdmissionRecord(
    long accountNumber,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String planStatusCode,
    String benefitPlanCode,
    int level6Code,
    List<String> statusFlags
) {}

public record AdmissionResponse(
    String runId,
    long accountNumber,
    String fromDate,
    String toDate,
    AdmissionSummary summary,
    List<AdmissionRecord> admissions
) {}
```

---

## (15) Performance Considerations

The main performance risk is **N+1 queries** arising from per-record enrichment of plan status and benefit plans.

Pattern:
- Primary query retrieves N transfer records from HAPTRFR.
- For each transfer, one query to PlanStatus and one query to BenefitPlan.

Mitigation:
- Batch-fetch plan status and benefit plan rows using `IN` lists or joins.

Example SQL with joins:

```sql
SELECT
    t.AFACCT,
    t.AFTRDT,
    t.AFTRTM,
    t.AFTYPE,
    s.XFNSST,
    s.StatusDescription,
    b.XFBPLN,
    b.CoverageLimit,
    b.CopayPercent
FROM dbo.HAPTRFR t
LEFT JOIN dbo.PlanStatus s
    ON s.XFNLV6 = t.AFLVL6
   AND s.XFNSST = t.StatusCode
LEFT JOIN dbo.BenefitPlan b
    ON b.XFBUBN = t.BenefitNumber
   AND b.XFBPLN = t.PlanCode
WHERE t.FILE_INDICATOR <> 0
  AND t.FLAG_INDICATOR NOT IN ('VOID', 'VOIDED')
  AND t.INPATIENT_OUTPATIENT_FLAG <> 'OUTPATIENT'
  AND t.AFACCT = @accountNumber
  AND t.AFTRDT BETWEEN @fromDate AND @toDate;
```

Additional considerations:
- Use pagination for large result sets.
- Apply database-level caching for dictionary and level tables.

---

## (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

Supporting rules from enrichment utilities (for completeness):

| Rule ID | Description |
|---------|-------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'. |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'. |
| BR-005  | When VMM is less than 01, branch to 'EXIT'. |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'. |
| BR-007  | When VDD is less than 01, branch to 'EXIT'. |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'. |
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT'. |
| BR-013  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-020  | SQL program accesses table 'HXPAPPPRF'. |

---

## (17) Edge Cases to Implement

| Scenario                                   | Expected Behavior |
|--------------------------------------------|-------------------|
| FILE_INDICATOR = 0                         | Record is skipped; SKIPPED_REASON = 'FILE_INDICATOR_ZERO'. |
| FLAG_INDICATOR = 'VOID' or 'VOIDED'       | Record is skipped; SKIPPED_REASON = 'VOID_RECORD'. |
| INPATIENT_OUTPATIENT_FLAG = 'OUTPATIENT'  | Record is skipped; SKIPPED_REASON = 'OUTPATIENT'. |
| Transfer date AFTRDT = 0                  | Treat as null; record fails date validation and is skipped or flagged as error. |
| Transfer date outside valid range         | VYY < 1800 or > 2100, VMM < 1 or > 12, VDD < 1 or > days-in-month → skip per XFXCYMD rules. |
| LDAMAP > 99 or > 9999                     | Treat as invalid mapping; log configuration error and apply safe defaults; avoid crashing flow. |
| Missing plan status row                   | Mark STATUS_MISSING = true; proceed with admission but flag for downstream review. |
| Missing benefit plan row                  | Mark BENEFIT_MISSING = true; proceed with admission; billing must handle missing benefits. |
| Missing level configuration               | Mark LEVEL_MISSING = true; use default level attributes. |
| Empty result set for account/date range   | Return summary with zero acceptedAdmissions and no admissions array entries. |
| Preference not configured (e.g., no default status)| Fail fast with error response indicating missing configuration; log event for operations. |
