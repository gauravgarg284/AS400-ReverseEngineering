# Business Processing Rules & Functional Specification

## HABADTE Patient Transfer XML Report (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

(1) Business Purpose

HABADTE is a patient-management report generator that reads transfer and station data and produces XML output describing patient movements across organisational levels. The report is used by clinical operations and integration teams to feed downstream systems with a structured view of transfers and related context. It typically runs as a batch or scheduled job.

> Core business question: "For a given organisational scope and time window, what patient transfers occurred, and how should they be represented in a standard XML format?"

The process produces a sequence of XML header and detail records written to HXFXMLH/HXFXMLD (and related HXPXML* staging), including counts of processed and skipped records.

(2) Inputs (API Request Parameters)

| Parameter         | AS400 Field / Context | Type        | Description |
|-------------------|-----------------------|-------------|-------------|
| orgLevel6         | AFLVL6                | String(6)   | Level-6 organisational code (e.g., facility or unit). |
| accountNumber     | AFACCT                | String      | Patient account identifier; optional filter when constraining to a single account. |
| fromTransferDate  | AFTRDT                | LocalDate   | Start date for transfer selection (inclusive). |
| toTransferDate    | AFTRDT                | LocalDate   | End date for transfer selection (inclusive). |
| fromTransferTime  | AFTRTM                | LocalTime   | Optional start time within the fromTransferDate. |
| toTransferTime    | AFTRTM                | LocalTime   | Optional end time within the toTransferDate. |
| transferType      | AFTYPE                | String(1)   | Optional transfer type filter (e.g., admission, discharge). |
| inpatientOnly     | INPATIENT_FLAG        | Boolean     | Whether to include only inpatient records. |
| includeVoided     | FLAG_INDICATOR        | Boolean     | Whether to include records marked as voided. |

Additional contextual input is the current user identity and security context (derived from the job/user profile on AS400). In the target architecture this is mapped from the Spring Security context.

- The current user principal is supplied via OAuth2 / OpenID Connect and propagated through Spring Security.
- Role-based access control (RBAC) is enforced so that only users with PHI-authorised roles can call the endpoint.
- All calls are logged with a PHI audit trail (user, timestamp, parameters, record counts).

(3) Organizational Hierarchy

HABADTE works with a multi-level organisational hierarchy, particularly through HXPLVL1–HXPLVL6 and station tables.

| Level Number | Name           | Key Size (digits) | AS400 Table |
|--------------|----------------|-------------------|-------------|
| 1            | Level 1        | variable          | HXPLVL1     |
| 2            | Level 2        | variable          | HXPLVL2     |
| 3            | Level 3        | variable          | HXPLVL3     |
| 4            | Level 4        | variable          | HXPLVL4     |
| 5            | Level 5        | variable          | HXPLVL5     |
| 6            | Level 6 (unit) | variable          | HXPLVL6     |

BR Note: Report headers are typically printed or emitted with the top-level organisational context (e.g., Level 6 name and its parent hierarchy) resolved via XFXLDSC reads on HXPLVL1–HXPLVL6.

(4) Patient Data Source

#### 4.1 Data Access Pattern

The primary PF feeding HABADTE is **HAPTRFR** (Transfer File), keyed by AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE. HABADTE reads this file sequentially by key, using the Level-6 and Account fields to group records and the date/time fields for ordering.

In SQL Server the primary query is:

```sql
SELECT *
FROM HAPTRFR
WHERE AFLVL6 = :orgLevel6
  AND AFTRDT BETWEEN :fromTransferDate AND :toTransferDate
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

#### 4.2 Key Fields

| SQL Column | AS400 Field | Type      | Description |
|-----------|-------------|-----------|-------------|
| AFLVL6    | AFLVL6      | CHAR(6)   | Level-6 organisational code. |
| AFACCT    | AFACCT      | CHAR      | Patient account number. |
| AFTRDT    | AFTRDT      | DEC(8,0)  | Transfer date in YYYYMMDD format. |
| AFTRTM    | AFTRTM      | DEC(4,0)  | Transfer time in HHMM format. |
| AFTYPE    | AFTYPE      | CHAR(1)   | Transfer type code. |

(5) Inclusion and Exclusion Rules

For HABADTE the key inclusion/exclusion rules are BR-017, BR-018, and BR-019.

### BR-017 – Skip when file indicator is zero

If the file indicator indicates no active record (e.g., end-of-file or logical close), the current record is skipped.

```pseudo
IF FILE_INDICATOR = 0
  GOTO SKIP_RECORD
ENDIF
```

SQL WHERE equivalent (conceptual):

```sql
-- FILE_INDICATOR is derived; records with no backing file are not selected
WHERE FILE_INDICATOR <> 0
```

### BR-018 – Skip voided records

Records marked as void/voided must not appear in the XML output.

```pseudo
IF FLAG_INDICATOR = 'VOID'
  GOTO SKIP_RECORD
ENDIF
```

```sql
WHERE FLAG_INDICATOR <> 'VOID'
```

### BR-019 – Skip outpatient records

The process only includes inpatient transfers when the inpatient/outpatient flag is set.

```pseudo
IF INPATIENT_OUTPATIENT_FLAG = 'O'
  GOTO SKIP_RECORD
ENDIF
```

```sql
WHERE INPATIENT_OUTPATIENT_FLAG <> 'O'
```

### Summary SQL WHERE Clause

```sql
WHERE FILE_INDICATOR <> 0           -- BR-017
  AND FLAG_INDICATOR <> 'VOID'      -- BR-018
  AND INPATIENT_OUTPATIENT_FLAG <> 'O'  -- BR-019
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

(6) Room Number Lookup (Station Hierarchy)

HABADTE enriches transfer records with station and room information via the station tables.

- Primary lookup: **OXPNSTN** (PF, XFFNSTN format) via logical **HXPNSTN** and external **TXPNSTN**.
- Enrichment program: HABADTE (declares and reads XFFNSTN); XFXLDSC is used for higher-level hierarchy.

Steps:

1. Read transfer record from HAPTRFR.
2. Using AFLVL6 and station code, look up the station in XFFNSTN.
3. Obtain descriptive fields: station name, ward, unit group.
4. If not found, mark the transfer with a default "Unknown Station" label.

```sql
SELECT s.*
FROM XFFNSTN s
WHERE s.XFNLV6 = :AFLVL6
  AND s.XFNSST = :stationCode;
```

Edge cases:
- If station not found, default descriptive text is used and the record is still output.
- Delete flags in station records are honoured; deleted stations are ignored.

(7) Level Description Enrichment

XFXLDSC enriches transfers with descriptions for levels 1–6 via HXPLVL1–HXPLVL6 and associated HXFLVL* format files.

Algorithm:

1. For each level code on the transfer (e.g., AFLVL6), derive lower-level hierarchy as needed.
2. Call XFXLDSC with the level code.
3. XFXLDSC reads HXFLVL1–HXFLVL6 to pick up names and attributes.
4. HABADTE receives descriptive fields for inclusion in XML headers.

Edge cases:
- Level codes outside valid ranges cause BR-009–BR-012 to reject the lookup and fall back to generic labels.

(8) Table Lookup Enrichment

XFXTABL provides generic table lookup for coded values (e.g., transfer type descriptions, flag descriptions) via HXPTABLD and logicals HXLTABLD/HXLTABLP/HXLTABLS.

SQL equivalent:

```sql
SELECT *
FROM XFFTABLD
WHERE XFDTCD = :dataTypeCode
  AND XFDECD = :dataElementCode;
```

If multiple logical views are used (e.g., by XFDMAP, XFDLDS, XFDSDS), corresponding queries use those columns in the WHERE clause.

Edge cases:
- When *IN79 indicator is on (BR-013–BR-016), XFXTABL exits early, meaning the calling logic skips table enrichment.

(9) Counting Rules

Counters are derived from HABADTE and XFXCNTR key rules.

| Counter Name      | Incremented When                                 | Description |
|-------------------|---------------------------------------------------|-------------|
| processedRecords  | After a record passes all BR-017/018/019 checks  | Number of records written to XML. |
| skippedFileZero   | FILE_INDICATOR = 0                               | Records skipped because no backing file (BR-017). |
| skippedVoided     | FLAG_INDICATOR = 'VOID'                          | Records skipped because they are voided (BR-018). |
| skippedOutpatient | INPATIENT_OUTPATIENT_FLAG = 'O'                  | Outpatient records excluded (BR-019). |

Relationships and meaning:
- processedRecords + skipped* counters = total records read.
- These counters are included in footer summaries or operational logs to verify completeness.

(10) Output Data Structure

Output is modelled on XML header/detail files.

#### 10.1 Header Fields (HXFXMLH / HXPXMLR)

| Field      | Description |
|-----------|-------------|
| XMRUSR    | User ID that generated the report. |
| XMRSEQ    | Sequence number of the XML run. |
| XMRID     | Unique identifier for the XML header. |

#### 10.2 Detail Row Fields (HXFXMLD / HXPXMLD)

| Field      | Description |
|-----------|-------------|
| XMDUSR    | User ID tied to the detail record. |
| XMDSEQ    | Sequence number aligned with header. |
| XMDSQ2    | Secondary sequence for individual transfer rows. |

Additional fields include transfer-specific content derived from HAPTRFR and enrichments.

#### 10.3 Footer/Summary Fields

Footer information is typically embedded as summary records in HXFXMLH or separate XML segments and includes:

- processedRecords
- skippedFileZero
- skippedVoided
- skippedOutpatient

#### 10.4 Sort Order

Detail records are ordered by:

1. AFLVL6 (organisational level)
2. AFACCT (account)
3. AFTRDT/AFTRTM (date/time)
4. AFTYPE (transfer type)

(11) Complete Processing Flow (Step-by-Step)

```text
STEP 1 – Init
  - Read runtime parameters (orgLevel6, date range, flags).
  - Load shared control structures from HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC.
  - Initialise counters and open HAPTRFR, HXFXMLH, HXFXMLD.

STEP 2 – Preferences / Lookup Setup
  - Prepare access to HXPLVL1–HXPLVL6 via XFXLDSC for hierarchy names.
  - Prepare access to XFFTABLD and related tables via XFXTABL for code translations.
  - Prepare station lookup via XFFNSTN.

STEP 3 – Context
  - Resolve current user and organisational context.
  - Create initial header record in HXFXMLH with XMRUSR, XMRSEQ, XMRID.

STEP 4 – Query
  - Sequentially read HAPTRFR in key order (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE).
  - For each record, apply inclusion/exclusion rules BR-017/018/019.

STEP 5 – Per-Record Enrichment
  5a – Station hierarchy
    - Lookup station in XFFNSTN using AFLVL6 and station code.
    - Attach station name and level codes.
  5b – Level descriptions
    - Call XFXLDSC to get descriptive labels for levels 1–6.
  5c – Code translations
    - Call XFXTABL to translate coded values (transfer type, flags) where *IN79 is off.

STEP 6 – Assemble Response
  - For each retained record, build an XML detail segment and write to HXFXMLD.
  - Update summary counters.
  - After all records processed, update HXFXMLH with totals and close all files.
```

(12) Data Type Conversions

12.1 Date Fields
- AFTRDT and similar DECIMAL(8,0) date fields map to `LocalDate` using pattern YYYYMMDD.
- A value of 0 is treated as null.

12.2 Time Fields
- AFTRTM and similar DECIMAL(4,0) time fields map to `LocalTime` using HHMM.

12.3 Packed Decimal Keys
- Numeric keys (e.g., HX1NUM–HX6NUM) map to `Long` or `Integer` types in Java.

12.4 String Trimming
- Fixed-length character fields are right-padded with spaces. Java layer should `.trim()` inputs and outputs.

(13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| HAPTRFR     | HAPTRFR          | Core transfer records. |
| HXPLVL1–6   | HXPLVL1–HXPLVL6  | Organisational hierarchy levels. |
| HXPTABLD    | HXPTABLD         | Generic code table definitions. |
| HXPXMLD     | HXPXMLD          | XML detail records. |
| HXPXMLR     | HXPXMLR          | XML header records. |
| OXPNSTN     | OXPNSTN          | Station master data. |
| HAPIRNK     | HAPIRNK          | Rank view over TAPIRNK. |
| HMLMAST5H   | HMLMAST5H        | Master data view over TMPMAST. |

### Suggested SQL Server Indexes

```sql
CREATE INDEX IX_HAPTRFR_MAIN
  ON HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE);

CREATE INDEX IX_OXPNSTN_KEY
  ON OXPNSTN (XFNLV6, XFNSST);

CREATE INDEX IX_HXPLVL6_KEY
  ON HXPLVL6 (HX6NUM);

CREATE INDEX IX_HXPTABLD_KEY
  ON HXPTABLD (XFDTCD, XFDECD);

CREATE INDEX IX_HXPXMLD_SEQ
  ON HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2);

CREATE INDEX IX_HXPXMLR_SEQ
  ON HXPXMLR (XMRUSR, XMRSEQ, XMRID);
```

(14) Spring Boot API Design

#### 14.1 Recommended REST Endpoint

- Method: `GET`
- Path: `/api/patient-transfers`

| Parameter         | Type        | Required | Validation |
|-------------------|------------|----------|------------|
| orgLevel6         | String      | Yes      | Not blank; matches known level-6 pattern. |
| accountNumber     | String      | No       | Optional; length and format checks. |
| fromTransferDate  | LocalDate   | Yes      | Must be <= toTransferDate. |
| toTransferDate    | LocalDate   | Yes      | Must be >= fromTransferDate. |
| fromTransferTime  | LocalTime   | No       | Optional; if provided, must be before toTransferTime when same date. |
| toTransferTime    | LocalTime   | No       | Optional. |
| inpatientOnly     | Boolean     | No       | Defaults to true. |
| includeVoided     | Boolean     | No       | Defaults to false. |

#### 14.2 Layer Structure

- Controller: `PatientTransferController`
- Service: `PatientTransferService`
- Repository: `HaptrfrRepository`, `StationRepository`, `HierarchyRepository`, `CodeTableRepository`

#### 14.3 Response JSON Shape

```json
{
  "orgLevel": {
    "level6Code": "123456",
    "level6Name": "Unit A",
    "hierarchyPath": ["Region", "Hospital", "Campus", "Building", "Wing", "Unit A"]
  },
  "filters": {
    "fromDate": "2024-01-01",
    "toDate": "2024-01-31",
    "inpatientOnly": true
  },
  "counters": {
    "processedRecords": 120,
    "skippedFileZero": 2,
    "skippedVoided": 3,
    "skippedOutpatient": 15
  },
  "transfers": [
    {
      "accountNumber": "ACCT123",
      "mrn": "MRN001",
      "transferDateTime": "2024-01-02T10:15:00",
      "transferType": "A",
      "station": {
        "code": "ST01",
        "name": "Ward 1"
      }
    }
  ]
}
```

#### 14.4 Java Entity/DTO Sketch

```java
public record PatientTransfer(
    String orgLevel6,
    String accountNumber,
    String mrn,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    Station station
) {}

public record Station(
    String code,
    String name
) {}
```

(15) Performance Considerations

The legacy pattern reads HAPTRFR sequentially and performs per-record lookups to station, hierarchy, and code tables. This poses N+1 risks in a naive implementation.

Recommended approach in SQL Server:

```sql
SELECT t.*, s.*, l6.*, ctype.*
FROM HAPTRFR t
LEFT JOIN OXPNSTN s
  ON s.XFNLV6 = t.AFLVL6 AND s.XFNSST = :stationCode
LEFT JOIN HXPLVL6 l6
  ON l6.HX6NUM = t.AFLVL6
LEFT JOIN HXPTABLD ctype
  ON ctype.XFDTCD = :typeCode AND ctype.XFDECD = t.AFTYPE
WHERE FILE_INDICATOR <> 0
  AND FLAG_INDICATOR <> 'VOID'
  AND INPATIENT_OUTPATIENT_FLAG <> 'O';
```

This reduces per-record calls and better exploits database indexes.

(16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

(17) Edge Cases to Implement

| Scenario                       | Expected Behavior |
|--------------------------------|-------------------|
| FILE_INDICATOR = 0            | Do not output XML for this record; increment skippedFileZero. |
| FLAG_INDICATOR = 'VOID'       | Do not output XML; increment skippedVoided. |
| INPATIENT_OUTPATIENT_FLAG = 'O'| Exclude outpatient records; increment skippedOutpatient. |
| Station not found             | Output record with default "Unknown Station" label. |
| Hierarchy level invalid       | Apply BR-009–BR-012; use generic labels or omit hierarchy. |
| No records in date range      | Produce header with zero counters and no detail records. |
| Voided-only dataset           | All records skipped; counters reflect skippedVoided count. |
