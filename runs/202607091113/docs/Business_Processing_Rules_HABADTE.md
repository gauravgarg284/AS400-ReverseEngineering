# Business Processing Rules & Functional Specification

## HABADTE Transfer XML Batch (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

HABADTE is a high-complexity batch program in the PATIENT_MANAGEMENT domain that reads transfer records and related master data to produce XML-based output for downstream systems. The process evaluates transfer eligibility based on internal indicators, flags, and inpatient/outpatient status, and then writes structured XML header and detail records.

> Core business question: "For each transfer record, should it be included in the XML outbound feed, and if so, how should it be represented based on current patient, benefit, and station configuration?"

The program produces:

- XML header records in **HXFXMLH** and detail records in **HXFXMLD**.
- Derived counts of processed, skipped, and voided transfers.
- Enriched transfer information with level, benefit, and station metadata suitable for external integration.

---

## (2) Inputs (API Request Parameters)

In the modernized Spring Boot service, HABADTE’s batch job will be exposed as an API. The following request parameters map to underlying AS400 fields and flags:

| Parameter | AS400 Field / Concept | Type | Description |
|-----------|------------------------|------|-------------|
| runDate | AFTRDT (HAPTRFR) | date | Processing date to limit transfer selection. |
| runTimeFrom | AFTRTM (HAPTRFR) | time | Lower bound for transfer time in the batch window. |
| runTimeTo | AFTRTM (HAPTRFR) | time | Upper bound for transfer time in the batch window. |
| level6 | AFLVL6 (HAPTRFR) | string | Level-6 key segment used to filter transfers (e.g., plan/network level). |
| accountNumber | AFACCT (HAPTRFR) | string | Optional filter to restrict processing to a single account. |
| inpatientOnly | INP/OUT flag | boolean | When true, restricts output to inpatient transfers only. |
| includeVoided | FLAG indicator | boolean | When false, void/voided transfers are excluded. |
| maxRecords | derived | integer | Optional limit on number of transfers processed per run. |

Additional implicit inputs:

- **Current user identity** from the security context (e.g., executing user profile on AS400) is mapped to `XMDUSR`/`XMRUSR` in XML tables and to audit columns in SQL Server.
- **Job-level configuration** (e.g., level and table dictionaries) is loaded via enrichment routines (XFXLDSC, XFXTABL, XFXGETID).

Security mapping for the modernized service:

- The REST API will be protected using **OAuth2** with JWT bearer tokens.
- Role-based access control (RBAC) must ensure only authorized service accounts can invoke the batch endpoint, especially because PHI-bearing fields (AFACCT, AFMRNO) are processed.
- PHI access must be logged to a PHI audit trail that records user, timestamp, request parameters, and counts of records processed.

---

## (3) Organizational Hierarchy

HABADTE participates in a level-based hierarchy where level-6 (AFLVL6 / MMPLV6) identifies an organizational or plan hierarchy segment. However, there is no explicit multi-level organization tree in the compact schema beyond level keys in PFs.

| Level | Name | Key Size (digits) | AS400 Table |
|-------|------|-------------------|-------------|
| 6 | Plan/Network Level | variable (code) | HXPLVL6 (HXFLVL6) |

**BR Note:** Report and XML headers must include the level-6 identifiers and their descriptions from **HXPLVL6**. Header formats in the modernized system must preserve these level descriptions to keep reports and XML consistent with existing outputs.

---

## (4) Patient Transfer Data Source

### 4.1 Data Access Pattern

The primary data source for HABADTE is the **HAPTRFR** physical file:

- PF: **HAPTRFR** (record format HAFTRFR)
- Primary key: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE` (unique)
- PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

HABADTE uses keyed access over HAPTRFR (via AFLVL6 and AFACCT) and iterates through transfer records in transfer date/time order, applying filters based on indicators and flags.

Equivalent SQL Server query for the primary read:

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFTRDT,
    AFTRTM,
    AFTYPE,
    AFMRNO,
    /* additional transfer fields */
FROM dbo.HAPTRFR
WHERE AFTRDT = @runDate
  AND AFTRTM BETWEEN @runTimeFrom AND @runTimeTo
  AND (@level6 IS NULL OR AFLVL6 = @level6)
  AND (@accountNumber IS NULL OR AFACCT = @accountNumber)
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

The records are ordered by the composite primary key, preserving the AS400 processing order.

### 4.2 Key Fields

Key fields derived from the HAPTRFR schema:

| SQL Column | AS400 Field | Type | Description |
|------------|-------------|------|-------------|
| AFLVL6 | AFLVL6 | varchar | Level-6 hierarchy code used to group transfers by plan/network. |
| AFACCT | AFACCT | varchar | Patient or account number, PHI. |
| AFTRDT | AFTRDT | date | Transfer date (YYYYMMDD). |
| AFTRTM | AFTRTM | time | Transfer time (HHMM). |
| AFTYPE | AFTYPE | char | Transfer type code (e.g., admission, discharge, move). |

---

## (5) Inclusion and Exclusion Rules

HABADTE applies a set of high-confidence filter rules (BR-017, BR-018, BR-019) to decide whether a transfer record should be processed or skipped.

### BR-017 – FILE Indicator Zero (Skip)

**Description**  
When the internal "file indicator" is zero, the record is treated as not eligible for output and is skipped.

```pseudo
IF fileIndicator = 0
    THEN GOTO SKIP_RECORD
```

Equivalent SQL WHERE fragment:

```sql
-- BR-017: Skip records where file indicator is zero
AND FileIndicator <> 0
```

### BR-018 – Voided Transfers (Skip)

**Description**  
Transfers flagged as void or voided are excluded from the XML output to avoid sending cancelled or reversed events.

```pseudo
IF flagIndicator = 'V' OR flagIndicator = 'VOID'
    THEN GOTO SKIP_RECORD
```

Equivalent SQL WHERE fragment:

```sql
-- BR-018: Skip voided transfers
AND FlagIndicator NOT IN ('V', 'VOID')
```

### BR-019 – Outpatient Transfers (Skip)

**Description**  
When the inpatient/outpatient flag indicates an outpatient encounter, the transfer is skipped for this batch. HABADTE focuses on inpatient transfers only.

```pseudo
IF inOutFlag = 'O'
    THEN GOTO SKIP_RECORD
```

Equivalent SQL WHERE fragment:

```sql
-- BR-019: Skip outpatient-only transfers
AND InOutFlag <> 'O'
```

### Summary SQL WHERE Clause

Combining the inclusion and exclusion rules for HABADTE:

```sql
WHERE AFTRDT = @runDate
  AND AFTRTM BETWEEN @runTimeFrom AND @runTimeTo
  AND (@level6 IS NULL OR AFLVL6 = @level6)
  AND (@accountNumber IS NULL OR AFACCT = @accountNumber)
  -- BR-017: file indicator must be non-zero
  AND FileIndicator <> 0
  -- BR-018: exclude void/voided transfers
  AND FlagIndicator NOT IN ('V', 'VOID')
  -- BR-019: include only inpatient transfers
  AND InOutFlag <> 'O'
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

---

## (6) Level and Table Enrichment

HABADTE relies on multiple enrichment steps to convert raw codes into descriptive values.

### Level Description Lookup (XFXLDSC)

**Business Concept:** Level description enrichment for levels 1–6.  
**BR IDs:** BR-009, BR-010, BR-011, BR-012.

Algorithm:

1. For each transfer’s level codes, HABADTE calls **XFXLDSC**.  
2. XFXLDSC declares and reads from **HXPLVL1–HXPLVL6** using keyed access on `HX1NUM`..`HX6NUM`.
3. If a level code is outside the valid range, the corresponding BR rule rejects or flags the record.

```pseudo
CALL XFXLDSC(levelCodes)
IF levelCodeOutOfRange
    THEN reject/skip per BR-009..BR-012
ELSE
    attach level descriptions to the transfer context
```

SQL SELECT equivalent for a single level master (example for level 6):

```sql
SELECT *
FROM dbo.HXPLVL6
WHERE HX6NUM = @level6;
```

Edge cases:

- **Not found:** If no HXPLVL6 row exists for the given key, mark the transfer as having an invalid level and either skip or route to an error bucket.
- **Delete flags:** If the level master includes a delete/inactive flag, filter out inactive rows.
- **Derived flags:** Flags such as "network-level" or "plan-level" are computed from HXPLVLx fields and used for header descriptions.

### Table Lookup Enrichment (XFXTABL)

**Business Concept:** General table-driven code-to-description mapping.  
**BR IDs:** BR-013, BR-014, BR-015, BR-016.

Algorithm:

1. HABADTE calls **XFXTABL** for various status and type codes.  
2. XFXTABL declares **HXPTABLD** and associated logical files **HXLTABLD**, **HXLTABLP**, **HXLTABLS**.  
3. It reads **XFFTABLD** and related tables (`XFFTABL2`, `XFFTABL3`, `XFFTABL4`) to resolve codes.
4. When indicator *IN79 is on, BR-013..016 cause an early exit from table lookup.

```pseudo
CALL XFXTABL(tableCode)
IF *IN79 = *ON
    THEN EXIT  // BR-013..BR-016
ELSE
    map code -> description using HXPTABLD and related tables
```

SQL SELECT equivalent:

```sql
SELECT t.*
FROM dbo.HXPTABLD t
WHERE t.XFDTCD = @codeType
  AND t.XFDECD = @codeValue;
```

Edge cases:

- **Not found:** If no table row exists, return a default description or flag an error.
- **Delete flags:** Exclude rows flagged as inactive.
- **Derived flags:** Some tables may control whether certain codes are eligible for export.

---

## (7) Benefit Enrichment (Station/Benefit)

Although HABADTE’s lineage shows multiple declared files (HXPBNFIT, HXPNSTN, HAPIRNK), the primary enrichment for station/benefit details is expressed via these logical files.

### Benefit Lookup (HXPBNFIT / TXPBNFIT)

**Business Concept:** Enrich transfer with benefit plan information.

Algorithm:

1. HABADTE declares **HXPBNFIT** (logical over TXPBNFIT) to access benefit details keyed by `XFBUBN`, `XFBPLN`.
2. For each transfer, it determines the applicable plan and benefit codes.
3. It reads the benefit table to derive coverage descriptions and contact details (including PHI fields such as `XFBTEL`).

```pseudo
SELECT *
FROM dbo.TXPBNFIT
WHERE XFBUBN = @ubn
  AND XFBPLN = @plan;
```

Edge cases:

- Benefit record missing → mark transfer with "missing benefit" status and continue or skip, depending on configuration.
- Phone number (`XFBTEL`) is PHI and must be masked or omitted from certain outputs.

### Station Lookup (HXPNSTN / TXPNSTN)

**Business Concept:** Station / location enrichment.

Algorithm:

1. HABADTE declares **HXPNSTN** (logical over TXPNSTN) keyed by `XFNLV6`, `XFNSST`.
2. For each transfer, it resolves station-level attributes (e.g., unit, state, or station status).

```sql
SELECT *
FROM dbo.TXPNSTN
WHERE XFNLV6 = @level6
  AND XFNSST = @stationState;
```

Edge cases mirror those in benefit lookups.

---

## (8) XML Output and Printer Enrichment

### XML Header/Detail Handling (HXFXMLH/HXFXMLD)

**Business Concept:** Build XML header and detail documents for transfers.

Algorithm:

1. HABADTE declares **HXPXMLD** and uses HXFXMLH/HXFXMLD files for XML layout.  
2. For each accepted transfer, it constructs header data (user, sequence, identifiers) and writes detail rows.
3. XFXGETID reads **HXFXMLR** to obtain or reserve XML identifiers.

```pseudo
CALL XFXGETID
READ HXFXMLR by key (XMRUSR, XMRSEQ, XMRID)
ASSIGN new sequence to HXFXMLD record
WRITE HXFXMLD
UPDATE HXFXMLH counters and status
```

Edge cases:

- **Identifier collision:** When an ID already exists, XFXGETID must increment sequence or generate a new ID.
- **Partial writes:** If HXFXMLD write fails, HABADTE must rollback or mark the header as failed.

### Printer File (PRINTER)

PRINTER represents a legacy printer file used to generate listing outputs aligned with the XML feed. In modernization, this corresponds to a report or log file.

Edge cases:

- If PRINTER is unavailable, XML generation should still proceed, but a warning should be logged.

---

## (9) Counting Rules

Counters are driven by HABADTE and utility programs.

| Counter Name | Incremented When | Description |
|--------------|------------------|-------------|
| processedCount | Transfer passes all filters (BR-017..BR-019) | Number of transfers successfully exported. |
| skippedFileIndicator | File indicator = 0 (BR-017) | Transfers skipped due to internal file indicator off. |
| skippedVoided | Flag indicator = void (BR-018) | Transfers skipped because they were voided or cancelled. |
| skippedOutpatient | In/out flag = outpatient (BR-019) | Transfers skipped because they are outpatient. |

Relationship:  
`processedCount + skippedFileIndicator + skippedVoided + skippedOutpatient = totalTransfersRead`.

Business meaning:  
These counters are used in batch control reports and should appear in audit logs and API responses to prove completeness.

---

## (10) Output Data Structure

### 10.1 Header Fields (HXFXMLH)

Derived from the HXPXMLD/HXPXMLR/HXFXMLH schema and lineage:

| Field | Description |
|-------|-------------|
| XMRUSR | User that owns the XML batch. |
| XMRSEQ | Batch sequence number. |
| XMRID | XML message identifier. |
| BatchDate | Date/time the batch was generated. |

### 10.2 Detail Row Fields (HXFXMLD)

| Field | Description |
|-------|-------------|
| XMDUSR | User that wrote the detail row. |
| XMDSEQ | Detail sequence. |
| XMDSQ2 | Sub-sequence within batch. |
| Payload | Serialized transfer and enrichment data (patient, level, benefit, station info). |

### 10.3 Footer/Summary Fields

Modeled in the modern system as a summary record or separate table:

| Field | Description |
|-------|-------------|
| totalTransfersRead | Total number of HAPTRFR records read. |
| totalProcessed | Number of records written to HXFXMLD. |
| totalSkipped | Number of skipped transfers, broken down by reason. |

### 10.4 Sort Order

Detail rows are sorted logically by `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, and then by XML sequence IDs to reflect AS400 ordering.

---

## (11) Complete Processing Flow (Step-by-Step)

```text
STEP 1 – Initialize
  - Load job parameters (runDate, runTimeFrom, runTimeTo, level6, accountNumber).
  - Initialize counters: processedCount, skippedFileIndicator, skippedVoided, skippedOutpatient.
  - Open HAPTRFR, HXPLVL1–HXPLVL6, HXPTABLD, TXPBNFIT, TXPNSTN, HXFXMLR, HXFXMLH, HXFXMLD.

STEP 2 – Preferences / Lookup Initialization
  - Preload level master configuration via XFXLDSC declarations.
  - Declare table dictionary HXPTABLD via XFXTABL.
  - Declare benefit and station files (HXPBNFIT, HXPNSTN).

STEP 3 – Context Setup
  - Determine current user (XMDUSR/XMRUSR) from job context.
  - Obtain or initialize XML header/sequence information using XFXGETID on HXFXMLR.
  - Prepare PRINTER context if reporting is enabled.

STEP 4 – Query Transfers
  - For each HAPTRFR record matching the key and filter range:
      - Apply BR-017: skip when file indicator = 0.
      - Apply BR-018: skip when flag indicator indicates void/voided.
      - Apply BR-019: skip when in/out flag = outpatient.

STEP 5 – Per-Record Enrichment
  5a – Level Enrichment (XFXLDSC)
      - Resolve level descriptions for AFLVL6 (and related levels) via HXPLVL1–HXPLVL6.
      - Reject or flag invalid levels per BR-009..BR-012.

  5b – Table-Driven Code Enrichment (XFXTABL)
      - Map status and other codes to descriptions via HXPTABLD/XFFTABL*.
      - Exit early when *IN79 is on per BR-013..BR-016.

  5c – Benefit and Station Enrichment
      - Lookup benefits in TXPBNFIT via HXPBNFIT.
      - Lookup stations in TXPNSTN via HXPNSTN.

STEP 6 – Assemble Response and Write Output
  - Build XML header if not yet written for the batch.
  - Construct HXFXMLD detail record with enriched fields.
  - WRITE HXFXMLD; UPDATE HXFXMLH counts and status.
  - Increment processedCount.
  - At end of file, write summary information to PRINTER or log.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields

- AS400 representation: DECIMAL(8,0) in `YYYYMMDD` format (e.g., AFTRDT).  
- Java: `java.time.LocalDate`.  
- Conversion rule: value `0` or `00000000` → `null`.

### 12.2 Time Fields

- AS400 representation: DECIMAL(4,0) in `HHMM` format (e.g., AFTRTM).  
- Java: `java.time.LocalTime`.  
- Conversion rule: `HH = value / 100`, `MM = value % 100`; `0000` → `null`.

### 12.3 Packed Decimal Keys

- Keys such as AFLVL6, AFACCT may be stored as packed decimals.  
- In SQL Server, use `NUMERIC` or `BIGINT` and map to Java `Long`.  
- Preserve leading zeros by storing a textual representation when required.

### 12.4 String Trimming

- AS400 fixed-length char fields are right-padded with spaces.  
- Java strings must be trimmed using `.trim()` when reading and padded on write when necessary.

---

## (13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR | dbo.HAPTRFR | Primary transfer data source. |
| HXPLVL1 | dbo.HXPLVL1 | Level 1 master data. |
| HXPLVL2 | dbo.HXPLVL2 | Level 2 master data. |
| HXPLVL3 | dbo.HXPLVL3 | Level 3 master data. |
| HXPLVL4 | dbo.HXPLVL4 | Level 4 master data. |
| HXPLVL5 | dbo.HXPLVL5 | Level 5 master data. |
| HXPLVL6 | dbo.HXPLVL6 | Level 6 master data. |
| HXPTABLD | dbo.HXPTABLD | Table dictionary for code mappings. |
| HXPBNFIT/TXPBNFIT | dbo.TXPBNFIT | Benefit plan definitions. |
| HXPNSTN/TXPNSTN | dbo.TXPNSTN | Station/state definitions. |
| HXPXMLD/HXFXMLD | dbo.HXFXMLD | XML detail output. |
| HXPXMLR/HXFXMLR | dbo.HXFXMLR | XML request/identifier store. |

### Suggested SQL Server Indexes

```sql
-- Primary query index on transfers
CREATE INDEX IX_HAPTRFR_RunWindow
ON dbo.HAPTRFR (AFTRDT, AFTRTM, AFLVL6, AFACCT, AFTYPE);

-- Level master lookup
CREATE INDEX IX_HXPLVL6_Key
ON dbo.HXPLVL6 (HX6NUM);

-- Table dictionary lookups
CREATE INDEX IX_HXPTABLD_Code
ON dbo.HXPTABLD (XFDTCD, XFDECD);

-- Benefit lookup
CREATE INDEX IX_TXPBNFIT_Key
ON dbo.TXPBNFIT (XFBUBN, XFBPLN);

-- Station lookup
CREATE INDEX IX_TXPNSTN_Key
ON dbo.TXPNSTN (XFNLV6, XFNSST);

-- XML header/detail correlation
CREATE INDEX IX_HXFXMLD_UserSeq
ON dbo.HXFXMLD (XMDUSR, XMDSEQ, XMDSQ2);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

- Method: `POST`  
- Path: `/api/batch/habadte/transfers`  

| Parameter | Type | Required | Validation |
|-----------|------|----------|-----------|
| runDate | LocalDate | yes | Must be >= 1900-01-01; cannot be in distant future beyond 2100 (align with BR-003/BR-004 via XFXCYMD). |
| runTimeFrom | LocalTime | yes | Must be <= runTimeTo. |
| runTimeTo | LocalTime | yes | Must be >= runTimeFrom. |
| level6 | String | no | Max length as per AFLVL6; trimmed. |
| accountNumber | String | no | Must match account format; trimmed. |
| inpatientOnly | Boolean | no | Default true. |
| includeVoided | Boolean | no | Default false. |
| maxRecords | Integer | no | Positive integer; reasonable upper bound (e.g., 100000). |

### 14.2 Layer Structure

- **Controller:** `HabadteBatchController`  
  - Accepts REST requests and triggers batch execution.
- **Service:** `HabadteBatchService`  
  - Implements the step-by-step flow from sections (5) and (11).
- **Repositories:**
  - `HaptrfrRepository` – JPA repository for HAPTRFR.  
  - `HxplvlRepository` – for HXPLVL1–6.  
  - `HxptabldRepository` – for HXPTABLD and related tables.
  - `TxpbnfitRepository`, `TxpnstnRepository` – for enrichment.  
  - `HxxmlRepository` – for HXFXMLH/HXFXMLD/HXFXMLR.

### 14.3 Response JSON Shape

```json
{
  "runId": "202607091113",
  "runDate": "2026-07-09",
  "parameters": {
    "level6": "...",
    "accountNumber": "...",
    "inpatientOnly": true,
    "includeVoided": false
  },
  "counters": {
    "totalTransfersRead": 1000,
    "totalProcessed": 750,
    "skippedFileIndicator": 50,
    "skippedVoided": 100,
    "skippedOutpatient": 100
  },
  "status": "COMPLETED",
  "xmlBatchId": "XMR123456",
  "messages": [
    "XML generation completed.",
    "Printer output written."]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record HabadteRequest(
    LocalDate runDate,
    LocalTime runTimeFrom,
    LocalTime runTimeTo,
    String level6,
    String accountNumber,
    Boolean inpatientOnly,
    Boolean includeVoided,
    Integer maxRecords
) {}

public record HabadteCounters(
    long totalTransfersRead,
    long totalProcessed,
    long skippedFileIndicator,
    long skippedVoided,
    long skippedOutpatient
) {}

public record HabadteResponse(
    String runId,
    LocalDate runDate,
    HabadteRequest parameters,
    HabadteCounters counters,
    String status,
    String xmlBatchId,
    List<String> messages
) {}
```

---

## (15) Performance Considerations

HABADTE’s design uses per-record enrichment via multiple READE/READ operations over level, table, benefit, station, and XML tables. In a naive migration, this pattern can cause N+1 query issues.

Recommended optimization:

- Preload level masters (HXPLVLx) and relevant table dictionaries (HXPTABLD/XFFTABL*) into memory caches.
- Use batched queries or JOINs for benefit and station enrichment.

Example SQL with JOINs:

```sql
SELECT
  t.*, 
  l6.*, 
  b.*, 
  s.*
FROM dbo.HAPTRFR t
LEFT JOIN dbo.HXPLVL6 l6
  ON l6.HX6NUM = t.AFLVL6
LEFT JOIN dbo.TXPBNFIT b
  ON b.XFBUBN = t.BenefitUbn
 AND b.XFBPLN = t.BenefitPlan
LEFT JOIN dbo.TXPNSTN s
  ON s.XFNLV6 = t.AFLVL6
 AND s.XFNSST = t.StationState
WHERE /* filters from section (5) */
;
```

---

## (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |
| BR-001 | Field formatting: exit if all blank (40 chars). |
| BR-002 | Field formatting: exit if first char non-blank. |
| BR-003 | Date validation: reject year < 1800 (historical minimum). |
| BR-004 | Date validation: reject year > 2100 (forecast maximum). |
| BR-005 | Date validation: reject month < 01 (calendar constraint). |
| BR-006 | Date validation: reject month > 12 (calendar constraint). |
| BR-007 | Date validation: reject day < 01 (calendar constraint). |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |
| BR-009 | Level lookup: reject if level code exceeds valid range. |
| BR-010 | Level lookup: reject if level code exceeds valid range. |
| BR-011 | Level lookup: reject if level code exceeds valid range. |
| BR-012 | Level lookup: reject if level code exceeds valid range. |
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-020 | SQL program accesses table 'HXPAPPPRF'. |

---

## (17) Edge Cases to Implement

| Scenario | Expected Behavior |
|----------|-------------------|
| File indicator = 0 | Apply BR-017; skip record and increment `skippedFileIndicator`. |
| Flag indicator = void/voided | Apply BR-018; skip record and increment `skippedVoided`. |
| In/out flag = outpatient | Apply BR-019; skip record and increment `skippedOutpatient`. |
| Invalid level code | Apply BR-009..BR-012; mark record as invalid and either skip or route to error bucket based on configuration. |
| Missing benefit record | Write record with default benefit description or skip; log warning. |
| Missing station record | Write record with default station description or skip; log warning. |
| XML ID collision | Regenerate or increment XML ID via XFXGETID before writing HXFXMLD. |
| HXFXMLD write failure | Roll back header state or mark batch as FAILED; report via API response. |
| Date field = 0 | Treat as null date in Java and SQL Server; do not fail the batch. |
| Time field = 0 | Treat as null time; avoid populating invalid times in output. |
| No transfers found | Return COMPLETED status with zero counts; no XML written. |
| Preferences not configured | Fail batch with configuration error; log details and expose error message in API.
