# Business Processing Rules & Functional Specification

## Patient Transfer and XML Export Report (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

HABADTE is the central patient-management controller that processes transfer records, applies inclusion/exclusion rules, enriches data from configuration tables, and writes XML output used for downstream reporting or integration. The program is invoked in operational contexts where patient transfers must be validated, structured, and persisted.

The report/process is used by patient accounting and bed-management teams to review transfers, distinguish inpatient from outpatient movement, and generate structured XML data for other systems.

> Core business question: "For a given operational run, which patient transfers are valid, non-voided inpatient movements, and how should they be represented in a structured XML payload for downstream systems?"

Summary outputs:
- Filtered set of transfer records from HAPTRFR (only active, non-voided inpatient records).
- Enriched attributes from level configuration (HXPLVL1–6, HXFLVL1–6) and table mappings (HXPTABLD/HXLTABL*).
- XML header and detail records written to HXFXMLH and HXFXMLD.
- Control counters maintained via XFXCNTR to signal processing completion.

---

## (2) Inputs (API Request Parameters)

In the AS400 implementation, HABADTE runs as a batch or invoked program, using data from files and system context. For the modernised API, inputs become explicit parameters.

### API Request Parameters

| Parameter                    | AS400 Field / Source | Type        | Description |
|-----------------------------|-----------------------|------------|-------------|
| runDate                     | VYY/VMM/VDD (XFXCYMD) | string (YYYY-MM-DD) | Processing date used to validate transfer date fields before querying. |
| inpatientOnly               | -INPATIENT/OUTPATIENT FLAG (HABADTE) | boolean     | When true, include only inpatient transfers; enforced by BR-019. |
| includeVoided               | -FLAG INDICATOR (HABADTE)           | boolean     | When false, skip records marked void/voided; enforced by BR-018. |
| requireActiveFileIndicator  | -FILE INDICATOR (HABADTE)           | boolean     | When true, skip records with file indicator = 0; enforced by BR-017. |
| levelMapCode                | LDAMAP (XFXLDSC)                    | integer     | Optional mapping code constraint; values above configured thresholds cause exits (BR-009–BR-012). |
| tableIndicator79            | *IN79 (XFXTABL)                     | boolean     | Internal flag reflecting table-driven conditions; when active, triggers exits (BR-013–BR-016). |

Current user identity (operator ID, role) is obtained from the AS400 job/user profile. In the Spring Boot implementation, this maps to the security context (e.g., `Authentication` principal).

Security mapping notes:
- Use OAuth2/OIDC for user authentication.
- Enforce RBAC scopes such as `patient.transfer.read` and `patient.transfer.export`.
- Log all access to PHI-bearing fields (AFACCT, AFMRNO, MMACCT, MRN fields) to a PHI audit trail.

---

## (3) Organizational Hierarchy

HABADTE relies on level configuration files (HXPLVL1–HXPLVL6, HXFLVL1–HXFLVL6) to interpret hierarchical codes. These behave like an organisational hierarchy of plan/level tiers but are not strictly org units.

| Level Number | Name         | Key Size Digits | AS400 Table |
|-------------|-------------|-----------------|-------------|
| 1           | Level 1     | variable        | HXPLVL1 / HXFLVL1 |
| 2           | Level 2     | variable        | HXPLVL2 / HXFLVL2 |
| 3           | Level 3     | variable        | HXPLVL3 / HXFLVL3 |
| 4           | Level 4     | variable        | HXPLVL4 / HXFLVL4 |
| 5           | Level 5     | variable        | HXPLVL5 / HXFLVL5 |
| 6           | Level 6     | variable        | HXPLVL6 / HXFLVL6 |

BR note on report header format:
- Report headers (or XML root attributes) should include the highest level value (e.g., Level 6 code) to contextualise transfers within plan or bed hierarchies.

---

## (4) Patient Data Source

The primary entity for HABADTE is the **patient transfer record**, stored in physical file **HAPTRFR**.

### 4.1 Data Access Pattern

AS400 behaviour:
- Primary file: HAPTRFR (record format HAFTRFR).
- Keyed access using composite key (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE).
- Sort order typically aligns with transfer date/time within level/account combinations.

In the HABADTE narrative, patient management rules gate which HAPTRFR records are processed. Date validation is delegated to XFXCYMD before or while reading HAPTRFR.

SQL Server SELECT equivalent:

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFTRDT,
    AFTRTM,
    AFTYPE,
    AFMRNO,
    -- other non-PHI attributes
FROM dbo.HAPTRFR
WHERE AFTRDT BETWEEN :runDateStart AND :runDateEnd
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

### 4.2 Key Fields

| SQL Column | AS400 Field | Type          | Description |
|-----------|-------------|--------------|-------------|
| level6    | AFLVL6      | INT          | Highest configuration or plan level indicator. |
| account   | AFACCT      | VARCHAR      | Patient/account identifier; PHI (AccountNumber). |
| transferDate | AFTRDT   | DATE (DECIMAL 8,0) | Transfer date. |
| transferTime | AFTRTM   | TIME (DECIMAL 4,0) | Transfer time (HHMM). |
| transferType | AFTYPE   | VARCHAR(…)   | Type of transfer (e.g., admission, discharge). |

These keys correspond directly to the DDS key fields and should be preserved as the clustered primary key in SQL Server.

---

## (5) Inclusion and Exclusion Rules

Inclusion/exclusion logic in HABADTE is driven by three main rules.

### BR-017 – File Indicator Active

**Description:** Records with a file indicator of zero are skipped and not processed.

```pseudo
IF fileIndicator = 0
    BRANCH TO SKIP_RECORD
ENDIF
```

SQL WHERE predicate:

```sql
WHERE fileIndicator <> 0
```

### BR-018 – Skip Voided Records

**Description:** Records where the flag indicator represents a voided/void status are skipped.

```pseudo
IF flagIndicator IN ('VOID', 'VOIDED')
    BRANCH TO SKIP_RECORD
ENDIF
```

SQL WHERE predicate:

```sql
AND flagIndicator NOT IN ('VOID', 'VOIDED')
```

### BR-019 – Inpatient-Only Filtering

**Description:** When processing inpatient runs, outpatient records are skipped.

```pseudo
IF inOutFlag = 'O'  // outpatient
    BRANCH TO SKIP_RECORD
ENDIF
```

SQL WHERE predicate:

```sql
AND inOutFlag <> 'O'
```

### Summary SQL WHERE Clause

```sql
WHERE fileIndicator <> 0          -- BR-017
  AND flagIndicator NOT IN ('VOID', 'VOIDED') -- BR-018
  AND inOutFlag <> 'O'           -- BR-019
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

---

## (6) Level Description Enrichment

HABADTE enriches transfer records using level configuration and mapping logic provided by XFXLDSC and HXPLVL*/HXFLVL*.

### Enrichment Concept: Level Description Lookup (Configuration Hierarchy)

BRs: BR-009–BR-012 (LDAMAP bounds).

Algorithm steps:
1. For each transfer record, derive a mapping code `LDAMAP` based on level6 or related configuration.
2. Call XFXLDSC with `LDAMAP`.
3. In XFXLDSC, if `LDAMAP > 9999` or `LDAMAP > 99` depending on context, exit without enrichment (BR-009–BR-012).
4. Otherwise, read associated level records from HXFLVL1–HXFLVL6 using declared PFs HXPLVL1–HXPLVL6.
5. Build a composite description (e.g., "Level1-Level2-...-Level6") and attach to the transfer record.

SQL SELECT equivalent:

```sql
SELECT
    l1.Description AS Level1Desc,
    l2.Description AS Level2Desc,
    l3.Description AS Level3Desc,
    l4.Description AS Level4Desc,
    l5.Description AS Level5Desc,
    l6.Description AS Level6Desc
FROM dbo.HXPLVL1 AS l1
JOIN dbo.HXPLVL2 AS l2 ON l2.HX2NUM = l1.NextLevelKey
JOIN dbo.HXPLVL3 AS l3 ON l3.HX3NUM = l2.NextLevelKey
JOIN dbo.HXPLVL4 AS l4 ON l4.HX4NUM = l3.NextLevelKey
JOIN dbo.HXPLVL5 AS l5 ON l5.HX5NUM = l4.NextLevelKey
JOIN dbo.HXPLVL6 AS l6 ON l6.HX6NUM = l5.NextLevelKey
WHERE l1.MapCode = :LDAMAP;
```

Edge cases:
- **Not-found levels:** If any level record is missing, log an error and proceed with partial descriptions.
- **Invalid LDAMAP (>9999 or >99):** Do not attempt enrichment; flag the transfer record as having invalid mapping.
- **Delete flags:** If level records are soft-deleted (not visible in DDS), ensure SQL queries filter out deleted rows.
- **Derived flags:** Optionally derive a `isConfigured` boolean based on successful level enrichment.

---

## (7) Table-Driven Enrichment

HABADTE relies on XFXTABL to interpret configuration tables HXPTABLD and its logical views HXLTABLD/HXLTABLP/HXLTABLS.

### Enrichment Concept: Table Lookup (Code/Description Mapping)

BRs: BR-013–BR-016 (*IN79 indicator).

Algorithm steps:
1. From each transfer record or level configuration, derive a data code `XFDTCD` and related mapping keys.
2. XFXTABL declares HXPTABLD and HXLTABLD/P/S and reads XFFTABLD/XFFTABL2–4.
3. If indicator *IN79 becomes active (based on table content or context), XFXTABL branches to EXIT (BR-013–BR-016), signalling that no further table-driven logic should run.
4. Otherwise, XFXTABL returns mapped description values or flags that HABADTE uses in XML output.

SQL SELECT equivalent:

```sql
SELECT
    t.XFDTCD,
    t.XFDECD,
    t.Description,
    t.StatusFlag
FROM dbo.HXPTABLD AS t
WHERE t.XFDTCD = :dataCode
  AND t.XFDECD = :detailCode;
```

Edge cases:
- **Not-found mappings:** If no matching code/description pair exists, mark the transfer record with a `mappingMissing` flag.
- **Indicator *IN79 logic:** In Java, represent *IN79 as a boolean field; when true, treat it as a rule to short-circuit mapping logic or mark the record as excluded from certain operations.
- **Delete flags:** Filter out soft-deleted codes.
- **Derived flags:** Derive convenience flags like `isMapped` and `requiresManualReview`.

---

## (8) XML ID Enrichment

XFXGETID and related XML files (HXPXMLR, HXFXMLR, HXPXMLD) provide XML-specific enrichment.

### Enrichment Concept: XML ID and Header/Detail Construction

Algorithm steps:
1. HABADTE declares XML-related files HXPXMLD/HXPXMLR and copybooks (CXXXMLP/CXXXMLC). It also declares ****HXPXML and PRINTER as external interfaces.
2. For each valid transfer record, HABADTE calls XFXGETID.
3. XFXGETID declares HXPXMLR and reads HXFXMLR to obtain XML sequence and identity values.
4. HABADTE uses these IDs to write header records to HXFXMLH and detail records to HXFXMLD.

SQL SELECT equivalent for XML IDs:

```sql
SELECT
    XMRUSR,
    XMRSEQ,
    XMRID
FROM dbo.HXPXMLR
WHERE XMRUSR = :userId
ORDER BY XMRSEQ DESC;
```

Edge cases:
- **Not-found XML ID:** If no XML record exists for the user, initialise sequence counters and create a fresh header entry.
- **Delete flags:** If XML rows are marked inactive, skip them when generating new XML output.
- **Derived flags:** Use `isXmlExported` to mark transfer records that have successfully generated XML.

---

## (9) Counting Rules

Counters in HABADTE are implemented via calls to XFXCNTR.

| Counter Name   | Incremented When                                           | Description |
|---------------|------------------------------------------------------------|-------------|
| processedCount| Each transfer record passes all inclusion rules (BR-017–19) and is successfully enriched. | Tracks total processed transfers. |
| skippedCount  | Any rule causes branching to SKIP_RECORD.                 | Tracks excluded transfers. |
| errorCount    | Enrichment or XML write fails.                             | Tracks records requiring manual review. |

Relationship notes:
- `processedCount + skippedCount` should equal the total number of HAPTRFR records considered for the run.

Business meaning:
- These counters drive operational KPIs (e.g., percentage of inpatient transfers successfully exported to XML).

---

## (10) Output Data Structure

HABADTE writes XML header and detail records.

### 10.1 Header Fields (HXFXMLH)

| Field        | Description |
|-------------|-------------|
| headerId    | Unique XML header identifier (from HXPXMLR/HXFXMLR IDs). |
| runDate     | Processing date. |
| levelSummary| Aggregated level hierarchy description for the batch. |

### 10.2 Detail Row Fields (HXFXMLD)

| Field           | Description |
|----------------|-------------|
| detailId       | Unique XML detail record ID. |
| transferKey    | Composite key (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE). |
| patientMrn     | MRN from HAPTRFR/OMPMAST/HXPDICT paths. |
| inpatientFlag  | Inpatient/outpatient indicator. |
| voidFlag       | Void status indicator. |
| levelDesc      | Derived level description from HXPLVL* and HXFLVL*. |
| mappingDesc    | Table-driven mapping description from HXPTABLD/HXLTABL*. |

### 10.3 Footer/Summary Fields

| Field          | Description |
|----------------|-------------|
| processedCount | Total processed transfers. |
| skippedCount   | Total skipped transfers. |
| errorCount     | Total error cases. |

### 10.4 Sort Order

- Primary sort: level6, account, transferDate, transferTime, transferType.
- Secondary sort: XML detailId for deterministic XML output ordering.

---

## (11) Complete Processing Flow (Step-by-Step)

```text
STEP 1: Init
  - Read configuration from level files HXPLVL1–HXPLVL6.
  - Initialise counters: processedCount, skippedCount, errorCount.
  - Initialise XML sequence via HXPXMLR/HXFXMLR.

STEP 2: Preferences / Lookup
  - Resolve table-based mappings using HXPTABLD and logical files HXLTABLD/HXLTABLP/HXLTABLS via XFXTABL.
  - Validate mapping codes LDAMAP; if >99 or >9999, exit enrichment (BR-009–BR-012).

STEP 3: Context
  - Determine run date (VYY/VMM/VDD) and validate using XFXCYMD (BR-003–BR-008).
  - Evaluate inclusion flags: fileIndicator, flagIndicator (void), inOutFlag.

STEP 4: Query
  - SELECT transfer records from HAPTRFR using the primary key and run date window.
  - Apply the summary WHERE clause:
    WHERE fileIndicator <> 0
      AND flagIndicator NOT IN ('VOID', 'VOIDED')
      AND inOutFlag <> 'O'
    ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE.

STEP 5: Per-Record Enrichment
  5a: Level Hierarchy
    - For each record, derive LDAMAP and call XFXLDSC to read HXFLVL1–HXFLVL6.
    - Build levelDesc; handle LDAMAP bounds and missing records.

  5b: Table Mapping
    - Call XFXTABL with relevant codes to read XFFTABLD/XFFTABL2–4.
    - If *IN79 is ON, branch to EXIT for table logic (BR-013–BR-016).
    - Attach mappingDesc to record.

  5c: XML ID and Header/Detail
    - Call XFXGETID to read HXFXMLR and derive XML IDs.
    - Write/update HXFXMLH header for the run.
    - Write HXFXMLD detail record for the transfer.

STEP 6: Assemble Response
  - Commit XML header and detail records.
  - Return counters and status summary to caller.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields

- AS400 DECIMAL(8,0) dates (e.g., AFTRDT, WBDATE) are stored as YYYYMMDD.
- Conversion to Java `LocalDate`:
  - If value = 0, treat as `null`.
  - Else parse substring into year, month, day.

### 12.2 Time Fields

- AS400 DECIMAL(4,0) times (AFTRTM) are stored as HHMM.
- Conversion to Java `LocalTime`:
  - If value = 0, treat as `null`.
  - Else derive hour = value / 100, minute = value % 100.

### 12.3 Packed Decimal Keys

- Key fields (e.g., AFLVL6, HX1NUM–HX6NUM) may be packed decimals.
- Conversion to Java/SQL:
  - Map to `INT` or `BIGINT` (Long) in Java.
  - Ensure scale = 0; store as numeric types in SQL Server.

### 12.4 String Trimming

- Fixed-length CHAR fields are right-padded in AS400.
- In Java, always apply `.trim()` when displaying or serialising to JSON.

---

## (13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| HAPTRFR     | dbo.HAPTRFR      | Patient transfer records (primary entity). |
| HXPDICT     | dbo.HXPDICT      | Patient dictionary/master data with PHI fields. |
| HXPLVL1–6   | dbo.HXPLVL1–6    | Level configuration hierarchy. |
| HXPTABLD    | dbo.HXPTABLD     | Code/description mapping table. |
| HXPXMLD     | dbo.HXPXMLD      | XML detail control table. |
| HXPXMLR     | dbo.HXPXMLR      | XML sequence/identity table. |
| OAPIRNK     | dbo.OAPIRNK      | Inquiry records with MRN. |
| OMPMAST     | dbo.OMPMAST      | Patient master with MRN, account, name, SSN. |
| OXPBNFIT    | dbo.OXPBNFIT     | Benefit master records. |
| OXPNSTN     | dbo.OXPNSTN      | Station/status records. |

### Suggested SQL Server Indexes

```sql
-- Primary query over patient transfers
CREATE INDEX IX_HAPTRFR_RunDate
ON dbo.HAPTRFR (AFTRDT, AFTRTM, AFLVL6, AFACCT, AFTYPE);

-- Level configuration lookups
CREATE INDEX IX_HXPLVL1_Key ON dbo.HXPLVL1 (HX1NUM);
CREATE INDEX IX_HXPLVL2_Key ON dbo.HXPLVL2 (HX2NUM);
CREATE INDEX IX_HXPLVL3_Key ON dbo.HXPLVL3 (HX3NUM);
CREATE INDEX IX_HXPLVL4_Key ON dbo.HXPLVL4 (HX4NUM);
CREATE INDEX IX_HXPLVL5_Key ON dbo.HXPLVL5 (HX5NUM);
CREATE INDEX IX_HXPLVL6_Key ON dbo.HXPLVL6 (HX6NUM);

-- Table mapping lookups
CREATE INDEX IX_HXPTABLD_Code
ON dbo.HXPTABLD (XFDTCD, XFDECD);

-- XML ID lookups
CREATE INDEX IX_HXPXMLR_UserSeq
ON dbo.HXPXMLR (XMRUSR, XMRSEQ);

-- PHI-related lookups (carefully governed)
CREATE INDEX IX_OMPMAST_Account
ON dbo.OMPMAST (MMACCT);

CREATE INDEX IX_OMPMAST_MRN
ON dbo.OMPMAST (MMMRNO, MMMMRN);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

- HTTP method: `GET`
- Path: `/api/patient-transfers/export`

Parameters:

| Name            | Type    | Required | Validation                   |
|----------------|--------|---------|------------------------------|
| runDate        | String | Yes     | ISO-8601 date, not in future.|
| inpatientOnly  | Boolean| No      | Default `true`.              |
| includeVoided  | Boolean| No      | Default `false`.             |
| levelMapCode   | Integer| No      | Must be >=0 and <=9999.      |

### 14.2 Layer Structure

- **Controller:** `PatientTransferExportController`
  - Validates request parameters.
  - Delegates to service.
- **Service:** `PatientTransferExportService`
  - Encapsulates steps 1–6 of the processing flow.
  - Uses repositories for data access.
- **Repository:** `HaptrfrRepository`, `LevelConfigRepository`, `TableMappingRepository`, `XmlRepository`
  - Provide CRUD and query methods over SQL Server tables.

### 14.3 Response JSON Shape

```json
{
  "runDate": "2026-07-02",
  "processedCount": 120,
  "skippedCount": 15,
  "errorCount": 3,
  "transfers": [
    {
      "transferKey": {
        "level6": 6,
        "account": "A12345",
        "date": "2026-07-01",
        "time": "14:30",
        "type": "ADMIT"
      },
      "patientMrn": "MRN0001",
      "inpatient": true,
      "voided": false,
      "levelDesc": "L1-L2-L3-L4-L5-L6",
      "mappingDesc": "BedTransfer",
      "xmlDetailId": "XMLD-000001"
    }
  ]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record TransferKey(
    int level6,
    String account,
    LocalDate date,
    LocalTime time,
    String type
) {}

public record PatientTransferDto(
    TransferKey transferKey,
    String patientMrn,
    boolean inpatient,
    boolean voided,
    String levelDesc,
    String mappingDesc,
    String xmlDetailId
) {}

public record PatientTransferExportResponse(
    LocalDate runDate,
    int processedCount,
    int skippedCount,
    int errorCount,
    List<PatientTransferDto> transfers
) {}
```

---

## (15) Performance Considerations

HABADTE’s enrichment flow can produce N+1 queries if each transfer record independently queries level configuration and table mappings.

Recommended approach:
- Use batch queries or joins to pre-load configuration data.
- For level hierarchies, load all relevant HXPLVL* records once and cache them.
- For table mappings, load HXPTABLD records into an in-memory map keyed by XFDTCD/XFDECD.

Example SQL LEFT JOIN to avoid N+1:

```sql
SELECT
    h.AFLVL6,
    h.AFACCT,
    h.AFTRDT,
    h.AFTRTM,
    h.AFTYPE,
    l6.Description AS Level6Desc,
    t.Description  AS MappingDesc
FROM dbo.HAPTRFR AS h
LEFT JOIN dbo.HXPLVL6 AS l6
  ON l6.HX6NUM = h.AFLVL6
LEFT JOIN dbo.HXPTABLD AS t
  ON t.XFDTCD = h.MappingCode
WHERE h.fileIndicator <> 0
  AND h.flagIndicator NOT IN ('VOID', 'VOIDED')
  AND h.inOutFlag <> 'O';
```

---

## (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | Skip records where file indicator = 0. |
| BR-018  | Skip records where flag indicator denotes void/voided status. |
| BR-019  | Skip records where inpatient/outpatient flag indicates outpatient. |
| BR-001  | Exit control routine when X = 0. |
| BR-002  | Exit control routine when X = 40. |
| BR-003  | Exit date validation when year < 1800. |
| BR-004  | Exit date validation when year > 2100. |
| BR-005  | Exit date validation when month < 1. |
| BR-006  | Exit date validation when month > 12. |
| BR-007  | Exit date validation when day < 1. |
| BR-008  | Exit date validation when day exceeds days-in-month. |
| BR-009  | Exit level description when LDAMAP > 99. |
| BR-010  | Exit level description when LDAMAP > 99. |
| BR-011  | Exit level description when LDAMAP > 99. |
| BR-012  | Exit level description when LDAMAP > 9999. |
| BR-013  | Exit table routine when indicator *IN79 is ON. |
| BR-014  | Exit table routine when indicator *IN79 is ON. |
| BR-015  | Exit table routine when indicator *IN79 is ON. |
| BR-016  | Exit table routine when indicator *IN79 is ON. |
| BR-020  | SQL profile program accesses table HXPAPPPRF. |

---

## (17) Edge Cases to Implement

| Scenario                                | Expected Behavior |
|----------------------------------------|-------------------|
| AFTRDT = 0 or AFTRTM = 0               | Treat as null; exclude record from main report or flag as error. |
| Date components invalid (XFXCYMD rules)| Do not process transfer; increment errorCount. |
| LDAMAP > 99 or > 9999                  | Skip level enrichment; proceed with transfer flagged as `mappingInvalid`. |
| Missing level records in HXPLVL*       | Use partial levelDesc; log warning. |
| *IN79 = ON in table logic              | Skip table enrichment for that record; possibly skip record depending on business decision. |
| No XML ID found in HXPXMLR/HXFXMLR     | Initialise new XML sequence; create header record before details. |
| Failed write to HXFXMLD/HXFXMLH        | Increment errorCount; log error; do not retry automatically. |
| No transfers match inclusion filters   | Return empty `transfers` array with processedCount = 0; HTTP 200 response. |
| Preferences not configured (missing mapping) | Mark records as `requiresManualReview`; do not block processing. |
