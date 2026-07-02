# Business Processing Rules & Functional Specification

## HABADTE Patient Transfer XML Report (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

HABADTE is the main patient-management driver that produces a transfer-oriented report and associated XML output. It reads inpatient transfer records, enriches them with institutional and configuration data, and persists XML header/detail records used by downstream systems or external interfaces.

The primary users are operational and clinical staff who need visibility into inpatient stays and transfers, as well as integration services consuming the XML payload. The process is typically executed as a batch job or scheduled task whenever transfers are updated.

> Core business question: "For a given inpatient account, what valid, non-voided transfer events exist, and how should they be packaged into an XML payload for downstream consumers?"

Summary outputs:
- A filtered set of inpatient transfer records (HAPTRFR) eligible for reporting.
- XML header records in HXPXMLR / HXFXMLH keyed by user and sequence.
- XML detail records in HXPXMLD / HXFXMLD capturing per-transfer information.

---

## (2) Inputs (API Request Parameters)

HABADTE is a batch-style RPG program, but in a Java/Spring Boot migration it will be exposed via a REST API. The key request parameters map to AS400 fields used for selection and context.

| Parameter | AS400 Field / Concept | Type | Description |
|-----------|------------------------|------|-------------|
| accountLevel6 | AFLVL6 | int | Institution or level identifier used to partition transfer records. |
| accountNumber | AFACCT | long | Patient account number used as primary key in HAPTRFR and OMPMAST. |
| inpatientFlag | -INPATIENT/OUTPATIENT FLAG | string | Indicates whether the patient is inpatient or outpatient; only inpatient is processed. |
| fileIndicator | -FILE INDICATOR | int | File status indicator; 0 means no valid record and must be skipped (BR-017). |
| voidFlag | -FLAG INDICATOR | string | Transfer void status; "void" or "voided" records are skipped (BR-018). |
| userId | XMRUSR / XMDUSR | string | User identifier for XML header/detail keys. |
| sequenceNumber | XMRSEQ / XMDSEQ | long | Sequence value for XML message grouping. |

Additional implicit input:
- **Current user identity and roles** are obtained from the security context (e.g., Spring Security) and mapped to HABADTE’s authority checks.

Spring Boot security mapping:
- OAuth2/OIDC for authentication.
- Role-based access control (RBAC) ensuring only users with `ROLE_PATIENT_MGMT` can invoke the transfer XML endpoint.
- PHI access is audited; every call is logged with user, accountNumber, and timestamp.

---

## (3) Organizational Hierarchy

HABADTE operates against a multi-level hierarchy modeled by HXPLVL1–HXPLVL6 and related status tables.

| Level Number | Name / Concept | Key Size (digits) | AS400 Table |
|--------------|----------------|-------------------|-------------|
| 1 | Global Level | variable | HXPLVL1 |
| 2 | Region / Cluster | variable | HXPLVL2 |
| 3 | Facility Group | variable | HXPLVL3 |
| 4 | Facility | variable | HXPLVL4 |
| 5 | Department / Unit | variable | HXPLVL5 |
| 6 | Service / Bed-Level | variable | HXPLVL6 |

BR note on report header:
- HABADTE uses level identifiers (e.g., AFLVL6, MMPLV6, XFNLV6) to drive header fields, ensuring that transfers are grouped and reported at the correct organizational tier.

---

## (4) Patient Data Source

### 4.1 Data Access Pattern

The primary entity is **patient transfer** data, backed by the physical file **HAPTRFR**.

From `data_dict_schema.physical_files`:
- PF: **HAPTRFR**  
- Record format: `HAFTRFR`  
- Unique key: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- PHI: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

Access pattern in HABADTE:
- Keyed reads on HAPTRFR using the composite key (level, account, transfer date/time, type).
- Sort order aligns with transfer date/time and type to produce chronological transfer sequences.

SQL Server equivalent SELECT:

```sql
SELECT AFLVL6,
       AFACCT,
       AFTRDT,
       AFTRTM,
       AFTYPE,
       AFMRNO,
       /* other transfer attributes */
FROM dbo.HAPTRFR
WHERE AFLVL6 = @accountLevel6
  AND AFACCT = @accountNumber
ORDER BY AFTRDT ASC, AFTRTM ASC, AFTYPE ASC;
```

### 4.2 Key Fields

| SQL Column | AS400 Field | Type | Description |
|------------|-------------|------|-------------|
| account_level6 | AFLVL6 | int | Level identifier for the institution or service line. |
| account_number | AFACCT | bigint | Patient account number. |
| transfer_date | AFTRDT | date | Transfer date. |
| transfer_time | AFTRTM | time | Transfer time (HHMM). |
| transfer_type | AFTYPE | char(1) | Code representing transfer type (admit, discharge, move). |
| mrn | AFMRNO | varchar(20) | Medical Record Number. |

---

## (5) Inclusion and Exclusion Rules

Selection logic combines explicit HABADTE rules with narrative key rules.

HABADTE key rules from `interpretations_detail`:
- BR-017, BR-018, BR-019.

### BR-017 – File Indicator Filter

**Description:** Records where the file indicator is zero are skipped, meaning that only records with a non-zero file indicator are processed.

```pseudo
IF fileIndicator = 0
    SKIP current record
ELSE
    CONTINUE processing
```

SQL WHERE fragment:

```sql
WHERE file_indicator <> 0
```

### BR-018 – Void Flag Filter

**Description:** Transfer records flagged as void/voided are excluded from processing. This ensures that reversed or invalidated transfers do not appear in the report/XML.

```pseudo
IF voidFlag IN ('VOID', 'VOIDED')
    SKIP current record
ELSE
    CONTINUE processing
```

SQL WHERE fragment:

```sql
AND UPPER(void_flag) NOT IN ('VOID', 'VOIDED')
```

### BR-019 – Inpatient-Only Filter

**Description:** Only inpatient records are included. Transfers associated with outpatient encounters are skipped.

```pseudo
IF inOutFlag = 'O' /* outpatient */
    SKIP current record
ELSE
    CONTINUE processing
```

SQL WHERE fragment:

```sql
AND in_out_flag <> 'O'
```

### Summary SQL WHERE Clause

```sql
WHERE AFLVL6 = @accountLevel6
  AND AFACCT = @accountNumber        -- BR-017/018/019 scope
  AND file_indicator <> 0             -- BR-017
  AND UPPER(void_flag) NOT IN ('VOID', 'VOIDED')  -- BR-018
  AND in_out_flag <> 'O'              -- BR-019
ORDER BY AFTRDT ASC, AFTRTM ASC, AFTYPE ASC;
```

---

## (6) Level Description Enrichment (HXPLVL1–6 via XFXLDSC)

HABADTE calls **XFXLDSC** to enrich records with human-readable level descriptions.

BR-driven behavior from XFXLDSC:
- BR-009–BR-012 limit `LDAMAP` values.

**Algorithm steps:**
1. For each transfer record, derive level identifiers (e.g., from AFLVL6) and mapping code LDAMAP.
2. Use XFXLDSC to read from HXPLVL1–HXPLVL6 (PFs) and their record formats (HXFLVL1–6) for descriptive fields.
3. If `LDAMAP` is greater than 9999 (BR-012) or exceeds other boundaries (BR-009–BR-011), abort enrichment and leave descriptions blank.
4. Attach descriptions to the output transfer record (e.g., facility, unit, bed descriptions).

SQL SELECT equivalent:

```sql
SELECT l6.HX6NUM,
       l6.level6_desc,
       l5.HX5NUM,
       l5.level5_desc,
       l4.HX4NUM,
       l4.level4_desc,
       l3.HX3NUM,
       l3.level3_desc,
       l2.HX2NUM,
       l2.level2_desc,
       l1.HX1NUM,
       l1.level1_desc
FROM dbo.HXPLVL6 l6
LEFT JOIN dbo.HXPLVL5 l5 ON l5.HX5NUM = l6.HX5NUM
LEFT JOIN dbo.HXPLVL4 l4 ON l4.HX4NUM = l6.HX4NUM
LEFT JOIN dbo.HXPLVL3 l3 ON l3.HX3NUM = l6.HX3NUM
LEFT JOIN dbo.HXPLVL2 l2 ON l2.HX2NUM = l6.HX2NUM
LEFT JOIN dbo.HXPLVL1 l1 ON l1.HX1NUM = l6.HX1NUM
WHERE l6.HX6NUM = @accountLevel6
  AND @LDAMAP <= 9999;   -- BR-009..BR-012
```

**Edge cases:**
- If any level record is not found, default descriptions to generic values (e.g., "Unknown Level").
- If `LDAMAP` exceeds thresholds, treat the configuration as invalid and either log the issue or fall back to code-only display.

---

## (7) Table Dictionary Enrichment (HXPTABLD via XFXTABL)

XFXTABL provides enrichment for code values using **HXPTABLD** and its logical files HXLTABLD/LP/LS.

Input codes: generic dictionary codes (XFDTCD) from transfer or institution records.

**Algorithm steps:**
1. HABADTE supplies a table code (XFDTCD) and context (mapping vs long vs short description).
2. XFXTABL, governed by BR-013–BR-016, uses indicator *IN79 to control when lookups are complete.
3. It reads from XFFTABLD and related PFs, mapping code to description.
4. The enriched description is returned to HABADTE for inclusion in XML and report fields.

Sample SQL using HXLTABLP (long description):

```sql
SELECT lp.XFDTCD,
       lp.XFDLDS AS long_desc
FROM dbo.HXPTABLD base
JOIN dbo.HXLTABLP lp
  ON lp.XFDTCD = base.XFDTCD
WHERE lp.XFDTCD = @code
  AND @indicator79 = 0;  -- when active, XFXTABL branches to EXIT
```

**Edge cases:**
- If *IN79 is set to active (BR-013–BR-016), the lookup terminates; in migration, this is modeled as an early break condition.
- If no dictionary entry is found, HABADTE retains the raw code and may flag the record for data cleanup.

---

## (8) MRN Rollover & Profile Enrichment (XFXMRNROL / HXXAPPPRF)

MRN rollover is handled by **XFXMRNROL**, which calls **HXHAPPPRF** (missing) and **HXXAPPPRF**.

**Algorithm steps:**
1. For each transfer record (AFMRNO), detect whether the MRN is subject to rollover rules (e.g., based on profile settings).
2. XFXMRNROL invokes HXHAPPPRF to determine application profiles (missing program; to be re-specified in migration).
3. XFXMRNROL then calls HXXAPPPRF (SQLRPGLE), which reads profile tables and returns the current canonical MRN.
4. HABADTE replaces or augments AFMRNO with the rolled MRN in XML output.

SQL SELECT equivalent (conceptual):

```sql
SELECT p.current_mrn,
       p.status
FROM dbo.PatientProfile p
WHERE p.account_number = @AFACCT
  AND p.original_mrn = @AFMRNO;
```

**Edge cases:**
- If no profile record exists, HABADTE uses AFMRNO as-is.
- If profile status indicates inactive or merged, MRN rollover may produce a new MRN; HABADTE must ensure consistency across XML header/detail.

---

## (9) Counting Rules

Counters and control fields are managed primarily in XFXCNTR and XFXCYMD.

| Counter Name | Incremented When | Description |
|--------------|------------------|-------------|
| processedTransfers | Each transfer record that passes BR-017–BR-019 filters | Total number of valid inpatient transfers processed for the request. |
| skippedVoidTransfers | Each transfer record where voidFlag is void/voided (BR-018) | Count of voided transfers excluded from output. |
| skippedOutpatientTransfers | Each transfer record where inOutFlag is outpatient (BR-019) | Count of outpatient transfers not represented in XML. |
| dateValidationFailures | Each time XFXCYMD exits due to invalid VYY/VMM/VDD rules | Number of records failing date validation, used for quality monitoring. |

Relationships and business meaning:
- `processedTransfers + skippedVoidTransfers + skippedOutpatientTransfers` approximates the total transfer records read.
- `dateValidationFailures` signals data quality issues in upstream systems.

---

## (10) Output Data Structure

HABADTE produces a hierarchical output: header, detail rows, and footer summaries.

### 10.1 Header Fields

Derived from account-level and organizational context.

| Field | Source | Description |
|-------|--------|-------------|
| reportId | Generated via XFXGETID (HXFXMLR) | Unique identifier for the XML batch. |
| accountNumber | HAPTRFR.AFACCT | Patient account. |
| mrn | Rolled MRN (XFXMRNROL/HXXAPPPRF) or AFMRNO | Primary MRN reported. |
| facilityLevel6 | AFLVL6 / HXPLVL6 | Level-6 identifier and description. |
| runTimestamp | System clock | Timestamp of batch execution. |

### 10.2 Detail Row Fields

Each detail row represents one transfer.

| Field | Source | Description |
|-------|--------|-------------|
| transferDate | HAPTRFR.AFTRDT | Transfer date. |
| transferTime | HAPTRFR.AFTRTM | Transfer time. |
| transferType | HAPTRFR.AFTYPE | Transfer type code. |
| unitDescription | HXPLVL5/HXPLVL6 via XFXLDSC | Unit or bed-level description. |
| institutionStatus | OXPNSTN/XFFNSTN | Institution status code and description. |
| benefitCode | OXPBNFIT / HXPBNFIT | Benefit-related code for the encounter. |

### 10.3 Footer/Summary Fields

| Field | Source | Description |
|-------|--------|-------------|
| totalProcessedTransfers | Counter `processedTransfers` | Number of transfers included in XML. |
| totalSkippedVoid | Counter `skippedVoidTransfers` | Number of voided transfers excluded. |
| totalSkippedOutpatient | Counter `skippedOutpatientTransfers` | Outpatient transfers not processed. |
| dateValidationFailures | Counter `dateValidationFailures` | Invalid date records. |

### 10.4 Sort Order

- Primary sort: `transferDate ASC`, `transferTime ASC`.  
- Secondary sort: `transferType ASC`.  
- Within the batch, records are grouped by `accountNumber` and `facilityLevel6`.

---

## (11) Complete Processing Flow (Step-by-Step)

```text
STEP 1 – Initialization
    - Load configuration from HXXLDA and HXXLEVEL via COPY.
    - Initialize counters: processedTransfers, skippedVoidTransfers,
      skippedOutpatientTransfers, dateValidationFailures.
    - Resolve organizational context using HXPLVL1–6 if needed.

STEP 2 – Preferences / Lookup
    - For the given accountLevel6/accountNumber, prepare lookup handles:
        * XFXLDSC for level descriptions.
        * XFXTABL for dictionary/table codes.
        * XFXMRNROL/HXXAPPPRF for MRN rollover.
        * XFXCYMD for date validation.
        * XFXGETID for XML batch ID assignment.

STEP 3 – Context
    - Obtain current userId and sequenceNumber for XML from HXFXMLR/HXPXMLR
      via XFXGETID.
    - Create or update HXFXMLH header record for this batch.

STEP 4 – Query
    - Read HAPTRFR records keyed by (AFLVL6, AFACCT).

    SQL:
        SELECT *
        FROM dbo.HAPTRFR
        WHERE AFLVL6 = @accountLevel6
          AND AFACCT = @accountNumber
        ORDER BY AFTRDT ASC, AFTRTM ASC, AFTYPE ASC;

STEP 5 – Per-Record Enrichment
    For each transfer record:

    5a – Apply HABADTE filters (BR-017, BR-018, BR-019)
        - If fileIndicator = 0 => increment skippedVoidTransfers? (skip as invalid)
        - If voidFlag in ('VOID','VOIDED') => increment skippedVoidTransfers; continue.
        - If inOutFlag = 'O' => increment skippedOutpatientTransfers; continue.

    5b – Validate date via XFXCYMD
        - Pass VYY, VMM, VDD to XFXCYMD.
        - If any BR-003..BR-008 triggers exit => increment dateValidationFailures; skip.

    5c – Enrich level descriptions via XFXLDSC
        - Use AFLVL6 and LDAMAP to look up multi-level descriptions.
        - Ensure LDAMAP within allowed ranges (BR-009..BR-012).

    5d – Enrich dictionary codes via XFXTABL
        - Translate internal codes to human-readable values.
        - Respect *IN79 indicator behavior (BR-013..BR-016).

    5e – Perform MRN rollover via XFXMRNROL/HXXAPPPRF
        - If profile indicates MRN change, replace AFMRNO with current MRN.

    5f – Assemble XML detail (HXPXMLD/HXFXMLD)
        - Write HXFXMLD row for each valid transfer.

STEP 6 – Assemble Response
    - Persist HXFXMLH header and HXFXMLD detail rows.
    - Return a JSON response summarizing counts and including references
      to the XML batch id.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields (DECIMAL 8,0 → LocalDate)

AS400 dates often use `DECIMAL(8,0)` in `YYYYMMDD` format.

Conversion:
- If value = 0, treat as `null` in Java.
- Otherwise, split into year, month, day and construct `LocalDate`.

```java
Long rawDate = record.getAftrdt();
LocalDate transferDate = (rawDate == 0)
    ? null
    : LocalDate.of(
        (int)(rawDate / 10000),
        (int)((rawDate / 100) % 100),
        (int)(rawDate % 100)
      );
```

### 12.2 Time Fields (DECIMAL 4,0 HHMM → LocalTime)

Conversion of `AFTRTM`:

```java
int rawTime = record.getAftrtm();
int hour = rawTime / 100;
int minute = rawTime % 100;
LocalTime transferTime = LocalTime.of(hour, minute);
```

### 12.3 Packed Decimal Keys (DECIMAL 6,0 → Long)

Packed decimal account or sequence fields are mapped to Java `long`.

```java
long accountNumber = record.getAfacct();
```

### 12.4 String Trimming (Fixed-Length Right-Padded → .trim())

Fixed-length char fields (names, codes) are right-padded with spaces.

```java
String mrn = rpgStringMrn.trim();
String transferType = rpgTransferType.trim();
```

---

## (13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR | dbo.HAPTRFR | Patient transfer records keyed by level, account, date/time, type. |
| HXPDICT | dbo.HXPDICT | Central dictionary/reference file with MRNs, account numbers, room and contact info. |
| HXPLVL1 | dbo.HXPLVL1 | Level-1 configuration (global level hierarchy). |
| HXPLVL2 | dbo.HXPLVL2 | Level-2 configuration (region/cluster). |
| HXPLVL3 | dbo.HXPLVL3 | Level-3 configuration (facility group). |
| HXPLVL4 | dbo.HXPLVL4 | Level-4 configuration (facility). |
| HXPLVL5 | dbo.HXPLVL5 | Level-5 configuration (department/unit). |
| HXPLVL6 | dbo.HXPLVL6 | Level-6 configuration (service/bed-level details). |
| HXPTABLD | dbo.HXPTABLD | Table dictionary codes. |
| HXPXMLD | dbo.HXPXMLD | XML detail records for transfer messaging. |
| HXPXMLR | dbo.HXPXMLR | XML header identifier records. |
| OAPIRNK | dbo.OAPIRNK | Break/rank records with MRN. |
| OMPMAST | dbo.OMPMAST | Patient master records with MRN, account, name, SSN. |
| OXPBNFIT | dbo.OXPBNFIT | Benefit records keyed by benefit and plan. |
| OXPNSTN | dbo.OXPNSTN | Institution status records keyed by level and status. |

### Suggested SQL Server Indexes

```sql
-- Primary query on HAPTRFR
CREATE INDEX IX_HAPTRFR_LVL6_ACCT_DT_TM_TYPE
ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE);

-- MRN lookup on OMPMAST
CREATE INDEX IX_OMPMAST_ACCT_MRN
ON dbo.OMPMAST (MMACCT, MMMRNO);

-- Level hierarchy
CREATE INDEX IX_HXPLVL6_NUM
ON dbo.HXPLVL6 (HX6NUM);

-- Dictionary table codes
CREATE INDEX IX_HXPTABLD_DTCD_DECD
ON dbo.HXPTABLD (XFDTCD, XFDECD);

-- Institution status
CREATE INDEX IX_OXPNSTN_LVL6_STATUS
ON dbo.OXPNSTN (XFNLV6, XFNSST);

-- XML header/detail
CREATE INDEX IX_HXPXMLR_USER_SEQ
ON dbo.HXPXMLR (XMRUSR, XMRSEQ);

CREATE INDEX IX_HXPXMLD_USER_SEQ
ON dbo.HXPXMLD (XMDUSR, XMDSEQ);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

- **Method:** `GET`  
- **Path:** `/api/patient-transfers/{accountNumber}`

Parameters:

| Name | Type | Required | Validation |
|------|------|----------|-----------|
| accountNumber | long | yes | Must be positive; must exist in OMPMAST. |
| accountLevel6 | int | yes | Must match a valid HXPLVL6.HX6NUM. |
| inpatientFlag | string | no | Defaults to "I" (inpatient); must be `I` or `O`. |

### 14.2 Layer Structure

- **Controller:** `PatientTransferController`  
  - Handles HTTP requests, validates parameters, invokes service.
- **Service:** `PatientTransferService`  
  - Implements the flow from Steps 1–6, orchestrating repositories and utilities.
- **Repositories:**
  - `HaptrfrRepository` (HAPTRFR PF).
  - `LevelRepository` (HXPLVL1–6 PFs).
  - `DictionaryRepository` (HXPTABLD/HXLTABL*).
  - `PatientMasterRepository` (OMPMAST PF).
  - `BenefitRepository` (OXPBNFIT PF).
  - `InstitutionStatusRepository` (OXPNSTN PF).
  - `XmlHeaderRepository`, `XmlDetailRepository` (HXPXMLR/HXPXMLD).

### 14.3 Response JSON Shape

```json
{
  "batchId": "XML202607020907-0001",
  "accountNumber": 123456789,
  "facilityLevel6": 999999,
  "transfers": [
    {
      "transferDate": "2026-07-02",
      "transferTime": "14:30",
      "transferType": "A",
      "unitDescription": "Cardiology Ward",
      "institutionStatus": "ACTIVE",
      "benefitCode": "PLAN-A",
      "mrn": "MRN00012345"
    }
  ],
  "summary": {
    "totalProcessedTransfers": 10,
    "totalSkippedVoid": 1,
    "totalSkippedOutpatient": 2,
    "dateValidationFailures": 0
  }
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record TransferDto(
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String unitDescription,
    String institutionStatus,
    String benefitCode,
    String mrn
) {}

public record TransferBatchDto(
    String batchId,
    long accountNumber,
    int facilityLevel6,
    List<TransferDto> transfers,
    int totalProcessedTransfers,
    int totalSkippedVoid,
    int totalSkippedOutpatient,
    int dateValidationFailures
) {}
```

---

## (15) Performance Considerations

The original HABADTE pattern implies per-record enrichment for levels, dictionary codes, MRN rollover, and XML writes. A naive migration risks N+1 queries.

Recommendations:
- Batch-fetch level descriptions for the relevant AFLVL6 values using a single query over HXPLVL1–6.
- Preload dictionary codes from HXPTABLD/HXLTABLP/HXLTABLS into an in-memory map.
- Cache MRN profiles from OMPMAST for the batch accountNumber.

Example optimized SQL with JOINs:

```sql
SELECT t.AFLVL6,
       t.AFACCT,
       t.AFTRDT,
       t.AFTRTM,
       t.AFTYPE,
       t.AFMRNO,
       l6.level6_desc,
       s.XFNSST AS inst_status,
       b.XFBPLN AS benefit_plan
FROM dbo.HAPTRFR t
LEFT JOIN dbo.HXPLVL6 l6 ON l6.HX6NUM = t.AFLVL6
LEFT JOIN dbo.OXPNSTN s ON s.XFNLV6 = t.AFLVL6
LEFT JOIN dbo.OXPBNFIT b ON b.XFBUBN = t.AFACCT
WHERE t.AFLVL6 = @accountLevel6
  AND t.AFACCT = @accountNumber
  AND t.file_indicator <> 0
  AND UPPER(t.void_flag) NOT IN ('VOID','VOIDED')
  AND t.in_out_flag <> 'O';
```

---

## (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'. |
| BR-003 | When VYY is less than 1800, branch to 'EXIT'. |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'. |
| BR-005 | When VMM is less than 01, branch to 'EXIT'. |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'. |
| BR-007 | When VDD is less than 01, branch to 'EXIT'. |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |

---

## (17) Edge Cases to Implement

| Scenario | Expected Behavior |
|----------|-------------------|
| File indicator is zero | Do not process the record; log as invalid file status (BR-017). |
| Void flag is void/voided | Skip record; increment `skippedVoidTransfers` (BR-018). |
| Inpatient/outpatient flag indicates outpatient | Skip record; increment `skippedOutpatientTransfers` (BR-019). |
| Date fields outside 1800–2100 range | Treat as invalid date; increment `dateValidationFailures`; skip record (BR-003, BR-004). |
| Month not in 1–12 or day not in valid range | Treat as invalid date; increment `dateValidationFailures`; skip record (BR-005–BR-008). |
| LDAMAP mapping code exceeds allowed ranges | Do not enrich level descriptions; use default or code-only representation (BR-009–BR-012). |
| Indicator *IN79 is active during dictionary lookup | Stop further table lookups for that record; rely on existing data (BR-013–BR-016). |
| MRN profile not found | Use AFMRNO as-is; mark record for potential manual review. |
| XML header/detail write fails | Log error, mark batch as partial failure, return error status to API caller. |
| No transfer records after filtering | Return an empty `transfers` array with summary counts set to 0.
