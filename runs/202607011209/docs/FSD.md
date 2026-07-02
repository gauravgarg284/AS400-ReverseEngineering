# Business Processing Rules & Functional Specification

## HABADTE Inpatient Transfer XML Export (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server.

---

(1) Business Purpose
---------------------

The HABADTE program produces an inpatient transfer history extract and associated XML payloads for downstream systems and printing. It filters transfer records from the patient transfer file, enriches them with station, benefits, and rank information, and writes XML detail records for valid inpatient stays.

> Core business question: *"For a given inpatient account, what transfer and station events should be included in the official inpatient transfer XML and printed output, and under what conditions should records be suppressed?"*

The process produces:

- A sequenced set of eligible inpatient transfer rows.
- XML header and detail records in HXFXMLH/HXFXMLD for integration or archiving.
- Printer-oriented output using the PRINTER file definition.

(2) Inputs (API Request Parameters)
-----------------------------------

In the modernized Spring Boot API, HABADTE is exposed as a service that returns inpatient transfer information and XML payloads. Request parameters map to AS400 fields used by the RPGLE program.

| Parameter Name       | AS400 Field / Concept | Type      | Description |
|----------------------|-----------------------|-----------|-------------|
| accountNumber        | AFACCT / MMACCT      | string    | Inpatient account identifier used to select transfers and patient master data. |
| mrn                  | AFMRNO / MMMRNO      | string    | Medical record number to identify the patient across files. |
| inpatientOnly        | -INPATIENT/OUTPATIENT FLAG | boolean | When true, exclude outpatient transfers (see BR-019). |
| includeVoided        | -FLAG INDICATOR      | boolean   | When false, skip voided records (see BR-018). |
| fileIndicator        | -FILE INDICATOR      | integer   | Technical control; when zero, entire file is skipped (see BR-017). |
| stationLevel         | XFNLV6 / HX6NUM      | string    | Station or level key influencing station lookups via HXPNSTN and HXPLVL* files. |
| userId               | XMDUSR               | string    | User context for XML logging entries. |

Notes:

- The current user identity is inferred from the AS400 job/user profile; in Spring Boot this is provided by the authenticated principal.
- Spring Boot security mapping:
  - **Authentication:** OAuth2/OpenID Connect tokens.
  - **Authorization:** RBAC roles (e.g. `ROLE_INPATIENT_REPORTS`) controlling access to PHI-bearing endpoints.
  - **PHI audit:** Each request and response containing PHI fields (account, MRN, name, phone, SSN) must be logged with user, timestamp, purpose-of-use, and correlation ID.

(3) Organizational Hierarchy
-----------------------------

HABADTE uses level-oriented station files (HXPLVL1–HXPLVL6 and logical file HXPNSTN) to represent hospital organizational hierarchy (e.g. facility → building → unit → ward → room/station).

| Level | Name          | Key Size (digits) | AS400 Table |
|-------|---------------|-------------------|-------------|
| 1     | Facility      | HX1NUM (numeric)  | HXPLVL1     |
| 2     | Campus/Block  | HX2NUM (numeric)  | HXPLVL2     |
| 3     | Building      | HX3NUM (numeric)  | HXPLVL3     |
| 4     | Unit          | HX4NUM (numeric)  | HXPLVL4     |
| 5     | Ward          | HX5NUM (numeric)  | HXPLVL5     |
| 6     | Station/Room  | HX6NUM (numeric)  | HXPLVL6     |

BR note (header format): Report and XML header lines typically present the hierarchy as "Facility / Unit / Station" using fields resolved from the level files. When station-level lookups fail, HABADTE should fall back to the raw station code from the transfer file.

(4) Patient Data Source
------------------------

### 4.1 Data Access Pattern

The primary entity is the **inpatient transfer record**. HABADTE declares and reads from the physical file `HAPTRFR` (record format `HAFTRFR`) and logical structures related to station and benefits.

From `data_dict_schema.physical_files`:

- `HAPTRFR` (PF):
  - Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`.
  - PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN).

Access pattern:

- Primary read over `HAPTRFR` keyed by `AFACCT` and `AFTRDT`/`AFTRTM` to obtain all transfer events for an account.
- Sort order is effectively: `AFLVL6` (station), `AFACCT` (account), then chronological transfer date/time.
- The program uses `AFTYPE` to distinguish transfer types (admission, transfer, discharge) during selection.

SQL Server equivalent:

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFMRNO,
    AFTRDT,
    AFTRTM,
    AFTYPE,
    /* additional clinical and administrative columns */
FROM dbo.InpatientTransfer AS t
WHERE t.AFACCT = @accountNumber
ORDER BY t.AFTRDT ASC, t.AFTRTM ASC;
```

In SQL Server, `dbo.InpatientTransfer` is the table mapped from `HAPTRFR`.

### 4.2 Key Fields

| SQL Column          | AS400 Field | Type        | Description |
|---------------------|------------|-------------|-------------|
| AFLVL6              | AFLVL6     | numeric     | Station/room level code representing the sixth hierarchy level. |
| AccountNumber       | AFACCT     | string      | Inpatient account number linking to patient master. |
| MRN                 | AFMRNO     | string      | Medical record number identifying the patient. |
| TransferDate        | AFTRDT     | decimal(8)  | Transfer date in YYYYMMDD format. |
| TransferTime        | AFTRTM     | decimal(4)  | Transfer time in HHMM format. |
| TransferType        | AFTYPE     | string      | Code for transfer event type (admit, move, discharge, etc.). |

(5) Inclusion and Exclusion Rules
---------------------------------

HABADTE applies three principal business rules to determine which transfer records are processed and written to XML.

### BR-017 – File Indicator Control

**Description:** When `-FILE INDICATOR` equals zero, the program branches to `SKIP`, effectively skipping processing of the current file or block of records. This acts as a technical guard, often driven by upstream configuration or presence/absence of data.

**Pseudocode:**

```pseudo
IF fileIndicator = 0 THEN
    GOTO SKIP_SECTION;
ENDIF;
```

**SQL WHERE Clause impact:**

This rule is implemented as a pre-condition rather than a record-level filter. In the API layer, if `fileIndicator = 0`, the service returns an empty result set without querying the database.

### BR-018 – Voided Record Exclusion

**Description:** When `-FLAG INDICATOR` equals "void" or "voided", branch to `SKIP`. Voided transfer records must be excluded from inpatient reporting and XML output.

**Pseudocode:**

```pseudo
IF flagIndicator IN ('VOID', 'VOIDED') THEN
    CONTINUE; // skip record
ENDIF;
```

**SQL WHERE Clause:**

```sql
WHERE flagIndicator NOT IN ('VOID', 'VOIDED')
```

### BR-019 – Outpatient Transfer Exclusion

**Description:** When `-INPATIENT/OUTPATIENT FLAG` equals `outpatient`, branch to `SKIP`. Outpatient transfer activity is excluded from the inpatient transfer export.

**Pseudocode:**

```pseudo
IF inOutFlag = 'O' /* outpatient */ THEN
    CONTINUE; // skip record
ENDIF;
```

**SQL WHERE Clause:**

```sql
WHERE inOutFlag <> 'O'
```

### Summary SQL WHERE Clause

Combined, the rules yield an overall selection filter for eligible inpatient transfer records:

```sql
SELECT ...
FROM dbo.InpatientTransfer AS t
WHERE 1 = 1
    -- BR-018: Exclude voided records
    AND t.FlagIndicator NOT IN ('VOID', 'VOIDED')
    -- BR-019: Exclude outpatient activity
    AND t.InOutFlag <> 'O'
ORDER BY t.TransferDate ASC, t.TransferTime ASC;
```

Note: BR-017 is implemented in service logic as a short-circuit condition; when `fileIndicator = 0`, the query is not executed.

(6) Station and Benefits Enrichment
-----------------------------------

HABADTE performs several enrichment steps using secondary files identified in `data_lineage` and `data_dict_schema`.

### 6.1 Station Master Lookup – HXPNSTN

**Business concept:** Resolve human-readable station descriptions (unit, ward, location) from station codes carried on transfer records.

**Relevant data lineage:** `TXPNSTN` → `HXPNSTN` (PFILE_OF), declared and read by HABADTE.

**Algorithm steps:**

1. For each eligible transfer, read `AFLVL6` (station code).
2. Use `AFLVL6` and a higher-level status key (`XFNSST`) to form the primary key into `HXPNSTN` (LF over `TXPNSTN`, record format `XFFNSTN`).
3. Retrieve station description fields (e.g. unit name, ward code, bed or room details).
4. Attach station description to the in-memory transfer DTO.

**SQL SELECT equivalent:**

```sql
SELECT n.*
FROM dbo.StationMaster AS n
WHERE n.StationLevel6 = @AFLVL6
  AND n.StationStatus  = @XFNSST;
```

**Edge cases:**

- **Not found:** When no matching station is found, keep raw `AFLVL6` in the response and set a flag `stationResolved = false`.
- **Delete flag logic:** If the station record has a logical delete flag (not shown in compact schema), skip enrichment and treat record as "inactive station".
- **Derived flags:** Derive `isICU`, `isObservation`, etc., based on station type codes, for use in downstream analytics.

### 6.2 Benefit Plan Enrichment – HXPBNFIT / OXPBNFIT

**Business concept:** Attach benefit plan and contact details to transfer events, based on unit/plan combinations.

**Relevant data lineage:** `TXPBNFIT` → `HXPBNFIT` (LF), plus `OXPBNFIT` (PF) holding PHI phone numbers.

**Algorithm steps:**

1. Determine benefit plan key (e.g. `XFBUBN`, `XFBPLN`) from the patient account or transfer context.
2. Lookup in `HXPBNFIT` (LF over `TXPBNFIT`) using these keys.
3. Retrieve plan-level information (coverage type, business unit).
4. Optionally join to `OXPBNFIT` to retrieve contact phone (`XFBTEL`) when needed for notifications.

**SQL SELECT equivalent:**

```sql
SELECT b.*
FROM dbo.BenefitPlan AS b
WHERE b.BusinessUnit = @XFBUBN
  AND b.PlanCode     = @XFBPLN;
```

**Edge cases:**

- **Not found:** Mark the transfer with `benefitPlanMissing = true` and exclude phone contact fields from output.
- **Delete/inactive:** If benefit record is inactive, still show historical coverage but avoid using contact info.
- **PHI considerations:** Phone numbers (`XFBTEL`) are PHI and must be masked/redacted in views not intended for direct patient contact.

### 6.3 Rank and Master Data – HAPIRNK / OAPIRNK / OMPMAST

**Business concept:** Attach patient rank, master demographic, and account-level info to each transfer.

**Relevant data lineage:** `TAPIRNK` → `HAPIRNK` (LF), `OAPIRNK` (PF) with MRN, `TMPMAST` → `HMLMAST5H` (LF), `OMPMAST` (PF) with MRN and core demographics.

**Algorithm steps:**

1. For given `AFACCT`/`AFMRNO`, lookup rank records in `HAPIRNK` (LF over `TAPIRNK`).
2. Use `BRKLV6`, `BRKACC`, `BRKSEQ` as keys into `OAPIRNK` to obtain `BRKMRN` (MRN).
3. Use MRN and account number to query `OMPMAST` for patient-level master data (name, SSN, additional MRN fields).
4. Attach rank and master info to transfer DTOs for reporting and XML generation.

**SQL SELECT equivalent:**

```sql
SELECT m.*
FROM dbo.PatientMaster AS m
WHERE m.AccountNumber = @AFACCT
  AND m.MRN           = @AFMRNO;
```

**Edge cases:**

- **Missing rank:** If no rank record exists, continue processing but leave rank-related fields empty.
- **Master mismatch:** If MRN from transfer (`AFMRNO`) does not match MRN from master (`MMMRNO`), raise an audit warning and prefer master MRN.
- **PHI handling:** Fields `MMNAME`, `MMPSSN`, `MMMMRN`, `MMACCT`, `BRKMRN` are PHI; restrict output based on caller role and mask sensitive identifiers in non-clinical contexts.

(7) Counting Rules
-------------------

Counting logic for HABADTE is inferred from its dependence on counter utilities.

| Counter Name         | Incremented When                                | Description |
|----------------------|--------------------------------------------------|-------------|
| totalTransfers       | Each eligible transfer record after all SKIP rules | Total number of inpatient transfer events exported. |
| skippedVoided        | BR-018 triggered for a record                    | Count of voided transfers excluded from output. |
| skippedOutpatient    | BR-019 triggered for a record                    | Count of outpatient transfers excluded from inpatient report. |
| skippedByFileFlag    | BR-017 short-circuit condition                   | Number of accounts/files suppressed due to file indicator zero. |

Relationship notes:

- `totalTransfers + skippedVoided + skippedOutpatient` equals the number of transfer rows encountered (when file indicator is non-zero).
- These counters provide business meaning for quality and audit reports about data suppression.

(8) Output Data Structure
--------------------------

HABADTE writes header and detail XML records to `HXFXMLH` and `HXFXMLD`, and uses PRINTER definitions for physical printouts. Based on data_dict_schema and lineage, fields are grouped as follows.

### 8.1 Header Fields

| Field Name         | Source File | Description |
|--------------------|------------|-------------|
| AccountNumber      | HAPTRFR / OMPMAST | Inpatient account identifier. |
| MRN                | HAPTRFR / OMPMAST | Medical record number. |
| PatientName        | OMPMAST    | Patient display name (MMNAME). |
| StationHierarchy   | HXPLVL1–6  | Concatenated facility / unit / station description. |
| ReportGeneratedAt  | HXFXMLH    | Date/time the XML header record is created. |
| RequestUser        | HXPXMLD/HXPXMLR | User ID responsible for the request (XMDUSR/XMRUSR). |

### 8.2 Detail Row Fields

| Field Name         | Source File | Description |
|--------------------|------------|-------------|
| TransferDate       | HAPTRFR    | Transfer date (AFTRDT). |
| TransferTime       | HAPTRFR    | Transfer time (AFTRTM). |
| TransferType       | HAPTRFR    | Event type (AFTYPE). |
| StationCode        | HAPTRFR    | Code for inpatient station (AFLVL6). |
| StationDescription | HXPNSTN    | Resolved unit/ward/station description. |
| BenefitPlanCode    | HXPBNFIT   | Plan code for coverage attached to the account. |
| RankCode           | HAPIRNK/OAPIRNK | Rank or status code for the patient. |

### 8.3 Footer/Summary Fields

| Field Name         | Source      | Description |
|--------------------|------------|-------------|
| TotalTransfers     | HABADTE counters | Total eligible inpatient transfer records. |
| SkippedVoided      | HABADTE counters | Count of voided transfers excluded. |
| SkippedOutpatient  | HABADTE counters | Count of outpatient transfers excluded. |
| SkippedByFileFlag  | HABADTE counters | Count of files/accounts suppressed by file indicator. |

### 8.4 Sort Order

Detail rows are sorted by transfer date/time for each account and, within that, by station hierarchy:

1. AccountNumber (AFACCT)
2. TransferDate (AFTRDT, ascending)
3. TransferTime (AFTRTM, ascending)
4. StationCode (AFLVL6)

(9) Complete Processing Flow (Step-by-Step)
-------------------------------------------

```pseudo
STEP 1 – Init
-------------
1. Read request parameters: accountNumber, mrn, inpatientOnly, includeVoided, fileIndicator.
2. Initialize counters: totalTransfers = 0; skippedVoided = 0; skippedOutpatient = 0; skippedByFileFlag = 0.
3. Initialize logging/context: userId from security context; correlationId for audit.

STEP 2 – Preferences and Lookup Setup
--------------------------------------
1. Declare station master files: HXPLVL1–HXPLVL6, HXPNSTN.
2. Declare benefit plan file: HXPBNFIT and OXPBNFIT.
3. Declare rank and patient master files: HAPIRNK, OAPIRNK, HMLMAST5H, OMPMAST.
4. Declare XML header/detail files: HXFXMLH, HXFXMLD.
5. Declare printer file: PRINTER.

STEP 3 – Context and Pre-Checks
--------------------------------
1. If fileIndicator = 0 (BR-017), set skippedByFileFlag = 1 and return empty response.
2. Resolve patient master record from OMPMAST using accountNumber and mrn.
3. Resolve station hierarchy defaults for the patient from HXPLVL* files.

STEP 4 – Query Inpatient Transfers
-----------------------------------
1. Execute primary query over dbo.InpatientTransfer:

   SQL:
   SELECT *
   FROM dbo.InpatientTransfer AS t
   WHERE t.AccountNumber = @accountNumber
   ORDER BY t.TransferDate ASC, t.TransferTime ASC;

2. Iterate through result set row by row.

STEP 5 – Per-Record Enrichment and Filtering
---------------------------------------------
For each row:

5a – Apply SKIP rules
---------------------
1. If flagIndicator IN ('VOID', 'VOIDED') (BR-018):
   - skippedVoided++;
   - CONTINUE.
2. If inOutFlag = 'O' (outpatient, BR-019):
   - skippedOutpatient++;
   - CONTINUE.

5b – Station enrichment
-----------------------
1. Use AFLVL6 and status to lookup StationMaster (HXPNSTN).
2. If found, attach stationDescription and derived flags; else mark stationResolved = false.

5c – Benefit plan enrichment
----------------------------
1. Lookup benefit plan in HXPBNFIT (and optionally OXPBNFIT) using unit/plan keys.
2. Attach coverage information and, where authorized, contact phone numbers.

5d – Rank and patient master enrichment
---------------------------------------
1. Lookup rank record in HAPIRNK/OAPIRNK using AFACCT/AFMRNO.
2. Merge patient master data from OMPMAST (name, SSN, MRN) into DTO, subject to PHI policy.

5e – XML detail creation
------------------------
1. Construct XML detail payload with enriched fields.
2. Write to HXFXMLD; update HXFXMLH header if first record.
3. totalTransfers++.

STEP 6 – Assemble Response
---------------------------
1. Construct response object containing:
   - Patient header with master data and station hierarchy.
   - List of enriched inpatient transfer events (XML or JSON).
   - Summary counters.
2. Log audit trail with correlationId and PHI access details.
3. Return response to caller.
```

(10) Data Type Conversions
---------------------------

### 10.1 Date Fields

AS400 date fields are typically `DECIMAL(8,0)` in `YYYYMMDD` format (e.g. `AFTRDT`). Conversion:

- Parse the numeric into components: year = value / 10^4; month = (value / 10^2) % 100; day = value % 100.
- Map to Java `LocalDate.of(year, month, day)`.
- Value `0` or outside valid ranges (see BR-003–BR-008) should be treated as `null`.

### 10.2 Time Fields

Time fields like `AFTRTM` are `DECIMAL(4,0)` in `HHMM` format.

- hour = value / 100; minute = value % 100.
- Map to `LocalTime.of(hour, minute)`.

### 10.3 Packed Decimal Keys

Keys such as station codes or sequence numbers are stored as decimals (`DECIMAL(6,0)` etc.).

- Map to Java `Long` or `Integer` depending on range.
- Preserve leading zeros in textual representations for external identifiers.

### 10.4 String Trimming

AS400 fixed-length character fields are right-padded with spaces.

- Use `.trim()` in Java when mapping to DTOs.
- Avoid trimming values where spaces are significant (e.g. formatted addresses).

(11) SQL Server Table Mapping
------------------------------

Key AS400 objects and their SQL Server equivalents:

| AS400 Object | SQL Server Table      | Purpose |
|--------------|----------------------|---------|
| HAPTRFR      | dbo.InpatientTransfer | Core inpatient transfer events. |
| HXPDICT      | dbo.Dictionary        | Large dictionary of cross-reference and PHI fields. |
| HXPLVL1–6    | dbo.StationLevels     | Hierarchical definitions of facility/station levels. |
| HXPTABLD     | dbo.TableDefinitions  | Table-driven configuration for codes and mappings. |
| HXPBNFIT     | dbo.BenefitPlanLF     | Logical view of benefit plans. |
| OXPBNFIT     | dbo.BenefitPlan       | Physical benefit plan table including phone contacts. |
| HAPIRNK      | dbo.RankLF            | Logical view of rank data. |
| OAPIRNK      | dbo.Rank              | Physical rank table including MRN. |
| HMLMAST5H    | dbo.PatientMasterLF   | Logical file over patient master. |
| OMPMAST      | dbo.PatientMaster     | Core patient master demographics and accounts. |
| HXPNSTN      | dbo.StationMaster     | Logical view of station/room information. |
| HXPXMLD/R    | dbo.XmlDefinitions    | XML layout and user definitions. |

### Suggested SQL Server Indexes

```sql
-- Primary query over inpatient transfers
CREATE INDEX IX_InpatientTransfer_AccountDateTime
ON dbo.InpatientTransfer (AccountNumber, TransferDate, TransferTime);

-- Station lookup
CREATE INDEX IX_StationMaster_LevelStatus
ON dbo.StationMaster (StationLevel6, StationStatus);

-- Benefit plan lookup
CREATE INDEX IX_BenefitPlan_BusinessUnitPlan
ON dbo.BenefitPlan (BusinessUnit, PlanCode);

-- Patient master lookup
CREATE INDEX IX_PatientMaster_AccountMrn
ON dbo.PatientMaster (AccountNumber, MRN);

-- Rank lookup
CREATE INDEX IX_Rank_AccountSeq
ON dbo.Rank (AccountNumber, Sequence);
```

(12) Spring Boot API Design
----------------------------

### 12.1 Recommended REST Endpoint

- Method: `GET`
- Path: `/api/inpatient-transfers/{accountNumber}`

Parameters:

| Name           | Type    | Required | Validation |
|----------------|---------|----------|-----------|
| accountNumber  | String  | Yes      | Non-empty, matches hospital account pattern. |
| mrn            | String  | No       | Optional; when provided, must match MRN pattern. |
| inpatientOnly  | Boolean | No       | Default `true`. |
| includeVoided  | Boolean | No       | Default `false`. |

### 12.2 Layer Structure

- **Controller:** `InpatientTransferController` handles HTTP requests and validation.
- **Service:** `InpatientTransferService` orchestrates business rules, enrichment, and persistence operations.
- **Repository:** `InpatientTransferRepository`, `StationRepository`, `BenefitPlanRepository`, `PatientMasterRepository`, `RankRepository` provide data access.

### 12.3 Response JSON Shape

```json
{
  "accountNumber": "12345678",
  "mrn": "MRN0001",
  "patientName": "DOE, JOHN",
  "stationHierarchy": "FAC1/UNITA/STN12",
  "transfers": [
    {
      "transferDate": "2024-06-01",
      "transferTime": "13:45",
      "transferType": "ADT",
      "stationCode": "STN12",
      "stationDescription": "Medical ICU",
      "benefitPlanCode": "PLAN01",
      "rankCode": "R1"
    }
  ],
  "summary": {
    "totalTransfers": 5,
    "skippedVoided": 1,
    "skippedOutpatient": 2,
    "skippedByFileFlag": 0
  }
}
```

### 12.4 Java Entity/DTO Sketch

```java
public record InpatientTransferDto(
    String accountNumber,
    String mrn,
    String patientName,
    String stationHierarchy,
    List<TransferEventDto> transfers,
    TransferSummary summary
) {}

public record TransferEventDto(
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String stationCode,
    String stationDescription,
    String benefitPlanCode,
    String rankCode
) {}

public record TransferSummary(
    int totalTransfers,
    int skippedVoided,
    int skippedOutpatient,
    int skippedByFileFlag
) {}
```

(13) Performance Considerations
-------------------------------

The legacy HABADTE pattern suggests potential **N+1 query risks** due to per-record lookups:

- For each transfer, HABADTE looks up station (HXPNSTN), benefit (HXPBNFIT/OXPBNFIT), rank (HAPIRNK/OAPIRNK), and patient master (OMPMAST).

Recommended optimization:

- Use **set-based queries** with JOINs instead of per-record lookups.

Example SQL JOIN pattern:

```sql
SELECT
    t.AccountNumber,
    t.TransferDate,
    t.TransferTime,
    t.TransferType,
    t.StationCode,
    n.StationDescription,
    b.PlanCode,
    r.RankCode,
    m.PatientName,
    m.MRN
FROM dbo.InpatientTransfer AS t
LEFT JOIN dbo.StationMaster AS n
    ON n.StationLevel6 = t.StationCode
LEFT JOIN dbo.BenefitPlan AS b
    ON b.AccountNumber = t.AccountNumber
LEFT JOIN dbo.Rank AS r
    ON r.AccountNumber = t.AccountNumber
LEFT JOIN dbo.PatientMaster AS m
    ON m.AccountNumber = t.AccountNumber AND m.MRN = t.MRN
WHERE t.AccountNumber = @accountNumber
  AND t.FlagIndicator NOT IN ('VOID', 'VOIDED')
  AND t.InOutFlag <> 'O'
ORDER BY t.TransferDate, t.TransferTime;
```

Additionally:

- Cache station and benefit lookups for commonly used codes.
- Avoid writing XML per record synchronously; buffer and batch writes where appropriate.

(14) Business Rules Reference Summary
-------------------------------------

| Rule ID | Description |
|---------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP' (suppress entire file/account). |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (exclude voided transfers). |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (exclude outpatient activity). |

These rules collectively determine the eligibility of transfer records for inclusion in the inpatient transfer export.

(15) Edge Cases to Implement
-----------------------------

| Scenario                       | Expected Behavior |
|--------------------------------|-------------------|
| File indicator equals zero     | Do not query database; return empty transfer list with `skippedByFileFlag = 1`. |
| Voided transfer record         | Exclude record from response; increment `skippedVoided`. |
| Outpatient transfer record     | Exclude record from response; increment `skippedOutpatient`. |
| Station not found in HXPNSTN   | Use raw station code; set `stationResolved = false`. |
| Benefit plan not found         | Leave benefit fields empty; set `benefitPlanMissing = true`. |
| Patient master record missing  | Return transfers with limited identifiers; log audit warning. |
| XML write failure to HXFXMLD   | Retry write; on failure, log error and return HTTP 500 with correlationId. |
| Empty result set after filters | Return empty transfers array with counters showing skipped reasons. |
