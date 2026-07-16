# Business Processing Rules & Functional Specification

## HABADTE Patient Census Report (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

The HABADTE program generates an inpatient census report based on transfer and status tables. It filters out non-eligible accounts (voided, outpatient, pre‑admission) and assembles XML detail records and printer output for downstream consumers (reporting, integration). Operational users (bed management, nursing administration, and finance) run this process during census cycles and end‑of‑day batch windows.

> **Core question:** "Which accounts represent actively admitted inpatients at the current census snapshot, and how should they be surfaced in XML and print output for downstream processing?"

Summary outputs produced:
- An inpatient census dataset derived from **HAPTRFR** (transfer records) and status data.
- XML header and detail records in **HXFXMLH** and **HXFXMLD / HXPXMLD** for interface consumption.
- Printer/spool output via the missing **PRINTER** file for human‑readable census reports.

---

## (2) Inputs (API Request Parameters)

The modernized REST API will expose parameters that mirror HABADTE’s implicit inputs (selection date/time, organisational level, and context flags).

| Parameter            | AS400 Field / Concept | Type        | Description                                                        |
|----------------------|-----------------------|------------|--------------------------------------------------------------------|
| `censusDate`         | AFTRDT                | `LocalDate`| Transfer date used to select eligible census records.              |
| `censusTime`         | AFTRTM                | `LocalTime`| Transfer time used with date to determine current status.          |
| `level6Code`         | AFLVL6                | `String`   | Organisational level‑6 code identifying the unit or facility.      |
| `includeVoided`      | Voided flag           | `boolean`  | If `true`, includes voided accounts; default is `false` (BR‑018).  |
| `includeOutpatient`  | I/O indicator         | `boolean`  | If `true`, includes outpatient accounts; default `false` (BR‑019). |
| `includePreAdmission`| File indicator        | `boolean`  | If `true`, includes pre‑admission (indicator=0); default `false`.  |
| `maxRecords`         | Internal counter limit| `int`      | Soft cap on the number of records returned (performance safety).   |

Additional implicit input:
- **Current user identity** is taken from the security context (e.g., HXFXMLH/HXPXMLD store `User` fields). In the modern API, this comes from the authenticated principal.

Spring Boot security mapping notes:
- Use OAuth2 / OpenID Connect for authentication, with roles mapped to organisational functions (e.g., `ROLE_CENSUS_ADMIN`, `ROLE_BED_MGMT`).
- Enforce RBAC on endpoints that expose PHI from **HAPTRFR**, **OMPMAST**, and **HXPDICT**.
- Persist audit events for every request that touches PHI‑bearing PFs, including user ID, timestamp, and filter parameters.

---

## (3) Organizational Hierarchy

The HABADTE process relies on a multi‑level organisational hierarchy hosted in **HXPLVL1–HXPLVL6** and status tables.

| Level Number | Name                         | Key Size (digits) | AS400 Table |
|-------------|------------------------------|-------------------|-------------|
| 1           | Corporate / System           | varies            | HXPLVL1     |
| 2           | Region / Division            | varies            | HXPLVL2     |
| 3           | Facility / Hospital          | varies            | HXPLVL3     |
| 4           | Service Line / Department    | varies            | HXPLVL4     |
| 5           | Ward / Unit Group            | varies            | HXPLVL5     |
| 6           | Ward / Nursing Station (L6)  | varies            | HXPLVL6     |

BR note on report header:
- HABADTE’s census header resolves level descriptions via **XFXLDSC**, which reads HXFLVL1–HXFLVL6 and applies rules BR‑009–BR‑012. Invalid level codes result in empty descriptions, and the header should clearly indicate the organisational path (Level1→Level6).

---

## (4) Patient Data Source (Primary Entity Data Source)

### 4.1 Data Access Pattern

The primary entity is the **inpatient account**. HABADTE derives the census from the **HAPTRFR** physical file:

- Primary PF: **HAPTRFR** (record format HAFTRFR).
- Access pattern: keyed or sequential over `AFLVL6`, `AFTRDT`, `AFTRTM`, `AFTYPE` and account fields.
- Sort order: effectively grouped by Level‑6 (AFLVL6), then by transfer date/time, to reflect the current census snapshot.
- PHI fields: **AFACCT** (AccountNumber) and **AFMRNO** (MRN).

Example SQL Server SELECT equivalent:

```sql
SELECT
    AFLVL6      AS Level6Code,
    AFACCT      AS AccountNumber,
    AFTRDT      AS TransferDate,
    AFTRTM      AS TransferTime,
    AFTYPE      AS TransferType,
    AFMRNO      AS MedicalRecordNumber,
    /* other HAPTRFR fields */
FROM dbo.HAPTRFR
WHERE AFLVL6 = @level6Code
  AND AFTRDT <= @censusDate
  AND (AFTRDT < @censusDate OR AFTRTM <= @censusTime)
ORDER BY AFLVL6, AFTRDT, AFTRTM, AFACCT;
```

### 4.2 Key Fields Table

From the data dictionary schema:

| SQL Column              | AS400 Field | Type           | Description                          |
|-------------------------|------------|---------------|--------------------------------------|
| `Level6Code`            | AFLVL6     | numeric       | Organisational level‑6 code.         |
| `AccountNumber`         | AFACCT     | string/number | Patient account identifier (PHI).    |
| `TransferDate`          | AFTRDT     | date/decimal  | Date of transfer/use in census.      |
| `TransferTime`          | AFTRTM     | time/decimal  | Time of transfer/use in census.      |
| `TransferType`          | AFTYPE     | string        | Transfer type/category.              |
| `MedicalRecordNumber`   | AFMRNO     | string/number | MRN for the account (PHI).           |

---

## (5) Inclusion and Exclusion Rules

This section describes HABADTE’s core filter rules (BR‑017, BR‑018, BR‑019) and their impact on the census.

### BR-017 – Exclude Pre-Admitted Patients

**Description**

A patient with file indicator = 0 is a pre‑admission — not yet formally admitted. These accounts must be excluded from the active census.

**Pseudocode**

```pseudo
for each record in HAPTRFR:
    if fileIndicator = 0 then
        // pre-admission; not in active census
        skip record
    else
        include record in further checks
```

**SQL WHERE Clause Fragment**

```sql
AND FileIndicator <> 0  -- BR-017: exclude pre-admissions
```

### BR-018 – Exclude Voided Accounts

**Description**

Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census.

**Pseudocode**

```pseudo
for each candidate record:
    if voideFlag = 'V' then
        // cancelled account; remove from census
        skip record
```

**SQL WHERE Clause Fragment**

```sql
AND VoidedFlag <> 'V'  -- BR-018: exclude voided accounts
```

### BR-019 – Exclude Outpatients

**Description**

Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.

**Pseudocode**

```pseudo
for each candidate record:
    if ioIndicator = 'O' then
        // outpatient; outside inpatient census
        skip record
```

**SQL WHERE Clause Fragment**

```sql
AND IOIndicator <> 'O'  -- BR-019: exclude outpatients
```

### Summary SQL WHERE Clause

Combined WHERE clause incorporating all three filters, plus organisational and date/time selection:

```sql
WHERE AFLVL6 = @level6Code
  AND AFTRDT <= @censusDate
  AND (AFTRDT < @censusDate OR AFTRTM <= @censusTime)
  AND FileIndicator <> 0      -- BR-017: exclude pre-admissions
  AND VoidedFlag <> 'V'       -- BR-018: exclude voided accounts
  AND IOIndicator <> 'O'      -- BR-019: exclude outpatients
ORDER BY AFLVL6, AFTRDT, AFTRTM, AFACCT;
```

---

## (6) Room/Status Enrichment (First Enrichment Step)

This enrichment derives status and organisational attributes used to classify records in the census. It leverages **HXPNSTN** (LF over TXPNSTN) and **XFFNSTN** via the HABADTE read operations.

**Business Concept:** "Status/Station Lookup for Level‑6 Accounts"

**BR reference:** Status lookup is implicitly controlled by organisational level rules BR‑009–BR‑012 (via XFXLDSC) and table‑driven logic in XFXTABL.

### Algorithm Steps

1. For each candidate account from HAPTRFR, identify its Level‑6 code (`AFLVL6`).
2. Query **HXPNSTN** (logical over TXPNSTN) using keys `XFNLV6` and `XFNSST` to determine nursing station and status.
3. Optionally consult **XFFNSTN** directly (as seen in HABADTE READ) for additional status attributes.
4. Attach resolved status (e.g., active, discharged, transferred) and station descriptions to the census row.

### SQL SELECT Equivalent

```sql
SELECT
    h.AFLVL6       AS Level6Code,
    h.AFACCT       AS AccountNumber,
    s.XFNSST       AS StatusCode,
    s.StationDesc  AS StationDescription
FROM dbo.HAPTRFR h
LEFT JOIN dbo.TXPNSTN s
    ON s.XFNLV6 = h.AFLVL6
   AND s.XFNSST = h.StatusCode
WHERE h.AFLVL6 = @level6Code
  -- plus base census filters from section (5)
;
```

### Edge Cases

- **Not-found status:** If no TXPNSTN record exists, leave status fields null and mark the row for follow‑up.
- **Delete/obsolete flags in TXPNSTN:** Status rows flagged as obsolete should be ignored in the join; new statuses should be preferred.
- **Derived flags:** Use table‑driven logic (XFXTABL) for mapping certain status codes to derived flags (e.g., `isOverflowBed`, `isObservation`).

---

## (7) Benefit Plan Enrichment (Second Enrichment Step)

**Business Concept:** "Benefit/Plan Lookup for Census Accounts"

This step enriches accounts with benefit and plan information using **HXPBNFIT** (LF over TXPBNFIT) and **OXPBNFIT**.

### Algorithm Steps

1. From HAPTRFR and potentially OMPMAST, determine benefit number (`XFBUBN`) and plan code (`XFBPLN`).
2. Lookup **HXPBNFIT** by these keys to retrieve benefit plan attributes (e.g., coverage type, payer group, contact telephone `XFBTEL`).
3. Attach benefit plan metadata to the census record for downstream financial reporting.

### SQL SELECT Equivalent

```sql
SELECT
    h.AFACCT        AS AccountNumber,
    b.XFBUBN        AS BenefitNumber,
    b.XFBPLN        AS PlanCode,
    b.XFBTEL        AS ContactTelephone
FROM dbo.HAPTRFR h
LEFT JOIN dbo.TXPBNFIT b
    ON b.XFBUBN = h.BenefitNumber
   AND b.XFBPLN = h.PlanCode
WHERE h.AFLVL6 = @level6Code
  -- plus base census filters
;
```

### Edge Cases

- **Not-found benefit:** If no plan exists for an account, default to a 'SELF PAY' or equivalent indicator, depending on configuration.
- **Delete flag:** Benefit rows flagged as deleted or inactive should not be used; implement `WHERE DeleteFlag <> 'Y'` in the join.
- **Derived coverage:** A configuration rule may derive `coverageTier` (e.g., Gold/Silver/Bronze) based on plan code prefixes.

---

## (8) XML Output Enrichment (Third Enrichment Step)

**Business Concept:** "XML Header and Detail Construction"

This step turns census records into XML messages using **HXFXMLH**, **HXFXMLD/HXPXMLD**, and **HXPXMLR/HXFXMLR**, orchestrated via HABADTE and utility programs such as XFXGETID.

### Algorithm Steps

1. Initialize XML header (**HXFXMLH**) for the current run, recording user ID, timestamp, and run identifier.
2. For each census row, call **XFXGETID** to resolve or generate XML record IDs from **HXPXMLR/HXFXMLR**.
3. Write XML detail segments to **HXFXMLD/HXPXMLD** with keys (`XMDUSR`, `XMDSEQ`, `XMDSQ2`).
4. Maintain sequence numbers to preserve record ordering.
5. After processing all records, update **HXFXMLH** with completion status and counts.

### SQL SELECT/INSERT Equivalent

```sql
INSERT INTO dbo.HXFXMLH (RunId, UserId, StartTime, Level6Code)
VALUES (@runId, @userId, SYSDATETIME(), @level6Code);

-- for each census row
INSERT INTO dbo.HXFXMLD (XMDUSR, XMDSEQ, XMDSQ2, XMDSEG)
VALUES (@userId, @seq, @subSeq, @xmlSegment);

UPDATE dbo.HXFXMLH
SET EndTime   = SYSDATETIME(),
    RowCount  = @totalRows,
    Status    = 'COMPLETE'
WHERE RunId = @runId;
```

### Edge Cases

- **ID collision:** XFXGETID must ensure uniqueness of XML record IDs; collisions should result in retries or error escalation.
- **Header update failure:** If HXFXMLH cannot be updated, consider the run failed and log the incident for re‑run.
- **Partial detail write:** Implement transactional behaviour to avoid orphaned header or detail segments.

---

## (9) Counting Rules

Count metrics are not explicitly named in rules but can be inferred from HABADTE’s key_rules and MRN/status processing.

| Counter Name          | Incremented When                                                 | Description                                       | Relationship                   | Business Meaning                                        |
|-----------------------|------------------------------------------------------------------|---------------------------------------------------|--------------------------------|--------------------------------------------------------|
| `totalCandidates`     | Each HAPTRFR record read for the target Level‑6 and date/time    | All potential census records                      | Based on HAPTRFR selection    | Raw workload size prior to filters.                    |
| `excludedPreAdmission`| A record meets BR‑017 (FileIndicator = 0)                        | Number of pre‑admission accounts excluded         | Subset of totalCandidates     | Volume of not‑yet‑admitted patients.                   |
| `excludedVoided`      | A record meets BR‑018 (VoidedFlag = 'V')                         | Number of voided accounts excluded                | Subset of totalCandidates     | Cancelled accounts that should never appear in census. |
| `excludedOutpatient`  | A record meets BR‑019 (IOIndicator = 'O')                        | Number of outpatient accounts excluded            | Subset of totalCandidates     | Accounts outside inpatient scope.                      |
| `finalCensusCount`    | A record passes all exclusion filters and enrichment steps       | Number of accounts in final output                | Derived from candidates minus exclusions | Size of active inpatient census.            |

---

## (10) Output Data Structure

The inpatient census report has three structural segments: header, detail, and footer/summary.

### 10.1 Header Fields

Derived primarily from level tables and configuration:

| Field                | Source            | Description                                   |
|----------------------|-------------------|-----------------------------------------------|
| `runId`              | Generated         | Unique identifier for census run.             |
| `levelPath`          | HXPLVL1–HXPLVL6   | Concatenated organisational path description.|
| `censusDateTime`     | Request / HAPTRFR | Census snapshot date and time.                |
| `generatedByUser`    | Security context  | User who triggered the run.                   |

### 10.2 Detail Row Fields

Fields per census account, mainly from HAPTRFR and enrichment tables:

| Field                  | Source      | Description                                    |
|------------------------|------------|-----------------------------------------------|
| `level6Code`           | HAPTRFR    | Organisational Level‑6 code.                  |
| `accountNumber`        | HAPTRFR    | Patient account (PHI).                         |
| `medicalRecordNumber`  | HAPTRFR    | MRN (PHI).                                     |
| `transferDate`         | HAPTRFR    | Transfer date.                                 |
| `transferTime`         | HAPTRFR    | Transfer time.                                 |
| `transferType`         | HAPTRFR    | Type of movement (admission, transfer, etc.). |
| `statusCode`           | TXPNSTN    | Resolved status code.                          |
| `stationDescription`   | TXPNSTN    | Nursing station description.                  |
| `benefitNumber`        | TXPBNFIT   | Benefit identifier.                            |
| `planCode`             | TXPBNFIT   | Benefit plan code.                             |
| `coverageTelephone`    | TXPBNFIT   | Contact telephone (PHI).                       |

### 10.3 Footer/Summary Fields

Aggregated counters and status indicators:

| Field                 | Source                 | Description                                         |
|-----------------------|------------------------|-----------------------------------------------------|
| `totalCandidates`     | Counting rules         | Number of HAPTRFR records investigated.             |
| `excludedPreAdmission`| Counting rules         | Number of pre‑admissions removed (BR‑017).          |
| `excludedVoided`      | Counting rules         | Number of voided accounts removed (BR‑018).         |
| `excludedOutpatient`  | Counting rules         | Number of outpatients removed (BR‑019).             |
| `finalCensusCount`    | Counting rules         | Final census row count.                             |
| `phiExposureSummary`  | PHI registry           | Summary of PHI fields touched during processing.    |

### 10.4 Sort Order

The default sort order should match legacy behaviour:

- Primary: `level6Code`.
- Secondary: `transferDate`, `transferTime`.
- Tertiary: `accountNumber`.

```sql
ORDER BY level6Code, transferDate, transferTime, accountNumber;
```

---

## (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1 – Init
    - Read configuration from HXXAPPPRF (application profile tables).
    - Resolve organisational hierarchy via HXPLVL1–HXPLVL6 using XFXLDSC.
    - Initialize counters (totalCandidates, excludedPreAdmission, excludedVoided, excludedOutpatient, finalCensusCount).
    - Create XML header record in HXFXMLH with runId, userId, level6Code, censusDateTime.

STEP 2 – Preferences / Lookup
    - Using XFXTABL, load table-driven preferences from HXPTABLD and related dictionaries.
    - Cache status and station metadata via TXPNSTN/HXPNSTN.
    - Cache benefit plan metadata via TXPBNFIT/HXPBNFIT.

STEP 3 – Context
    - Determine the census horizon (censusDateTime) from request parameters.
    - Determine Level‑6 scope from `level6Code` and the organisational path.
    - Load any XML or printer configuration from ****HXPXML and PRINTER (when available).

STEP 4 – Query
    - Read candidate records from HAPTRFR using key fields.
    - For each record:
        - totalCandidates++.
        - Apply BR‑017: if FileIndicator = 0 then excludedPreAdmission++ and continue.
        - Apply BR‑018: if VoidedFlag = 'V' then excludedVoided++ and continue.
        - Apply BR‑019: if IOIndicator = 'O' then excludedOutpatient++ and continue.
    - Remaining records constitute the base inpatient dataset.

STEP 5 – Per-Record Enrichment
    5a – Status/Station Enrichment (Section 6)
        - Join TXPNSTN/HXPNSTN to resolve statusCode and stationDescription.
    5b – Benefit Plan Enrichment (Section 7)
        - Join TXPBNFIT/HXPBNFIT to enrich with benefitNumber, planCode, coverageTelephone.
    5c – XML ID Enrichment (Section 8)
        - Call XFXGETID to generate or resolve XML identifiers using HXPXMLR/HXFXMLR.
        - Build XML detail segments for each record.

STEP 6 – Assemble Response
    - Write XML details to HXFXMLD/HXPXMLD.
    - Update HXFXMLH with finalCensusCount and completion status.
    - Produce printer output via PRINTER (layout derived from table-driven configuration).
    - Return JSON response from API reflecting the census dataset and summary metrics.
```

---

## (12) Data Type Conversions

The following conversions are required when moving from DDS/packed decimal to Java/SQL Server types.

### 12.1 Date Fields

- Legacy format: DECIMAL(8,0) representing `YYYYMMDD`.
- Java: `java.time.LocalDate`.
- Conversion: parse as string and map to LocalDate; value `0` or `00000000` becomes `null`.

```java
String raw = String.format("%08d", dateDecimal);
if ("00000000".equals(raw)) return null;
LocalDate date = LocalDate.parse(raw, DateTimeFormatter.BASIC_ISO_DATE);
```

### 12.2 Time Fields

- Legacy format: DECIMAL(4,0) representing `HHMM`.
- Java: `java.time.LocalTime`.
- Conversion: split hours and minutes; `0000` can be treated as `null` or midnight based on business rules.

```java
String raw = String.format("%04d", timeDecimal);
if ("0000".equals(raw)) return null;
int hh = Integer.parseInt(raw.substring(0, 2));
int mm = Integer.parseInt(raw.substring(2, 4));
LocalTime time = LocalTime.of(hh, mm);
```

### 12.3 Packed Decimal Keys

- Legacy: DECIMAL(6,0) or similar for keys like `AFLVL6`, `XFNLV6`.
- Java: `long` or `int`.
- Conversion: direct numerical mapping; ensure no leading‑zero semantics are lost.

### 12.4 String Trimming

- Legacy: fixed‑length CHAR fields, right‑padded with spaces.
- Java: `String.trim()` should be applied when reading.

```java
String stationDesc = r.getString("StationDescription").trim();
```

---

## (13) SQL Server Table Mapping

Key AS400 objects and their target SQL Server tables:

| AS400 Object | SQL Server Table | Purpose                                              |
|-------------|------------------|------------------------------------------------------|
| HAPTRFR     | dbo.HAPTRFR      | Transfer records / base inpatient census source.     |
| TXPNSTN     | dbo.TXPNSTN      | Status and nursing station reference data.          |
| HXPNSTN     | dbo.HXPNSTN      | Logical view over TXPNSTN (optional).               |
| TXPBNFIT    | dbo.TXPBNFIT     | Benefit plan base table.                            |
| HXPBNFIT    | dbo.HXPBNFIT     | Logical view over TXPBNFIT (optional).              |
| HXPTABLD    | dbo.HXPTABLD     | Table‑driven configuration dictionary.              |
| HXPLVL1–6   | dbo.HXPLVL1–6    | Organisational level hierarchy tables.              |
| HXPXMLD     | dbo.HXPXMLD      | XML detail segments storage.                         |
| HXPXMLR     | dbo.HXPXMLR      | XML record identifiers / response records.          |
| HXFXMLH     | dbo.HXFXMLH      | XML header / run metadata.                          |
| HXFXMLD     | dbo.HXFXMLD      | XML detail segments (mirror of HXPXMLD).           |

### Suggested SQL Server Indexes

Indexes should support primary census filtering and enrichment lookups.

```sql
-- Primary census selection on HAPTRFR
CREATE INDEX IX_HAPTRFR_Level6_DateTime
ON dbo.HAPTRFR (AFLVL6, AFTRDT, AFTRTM, AFACCT);

-- Status/Station lookup
CREATE INDEX IX_TXPNSTN_Level6_Status
ON dbo.TXPNSTN (XFNLV6, XFNSST);

-- Benefit plan lookup
CREATE INDEX IX_TXPBNFIT_Benefit_Plan
ON dbo.TXPBNFIT (XFBUBN, XFBPLN);

-- Organisational level searches
CREATE INDEX IX_HXPLVL6_LevelNum
ON dbo.HXPLVL6 (HX6NUM);

-- XML detail access
CREATE INDEX IX_HXPXMLD_User_Seq
ON dbo.HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2);

CREATE INDEX IX_HXPXMLR_User_Seq_Id
ON dbo.HXPXMLR (XMRUSR, XMRSEQ, XMRID);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

Endpoint: `GET /api/census`

| Parameter            | Type        | Required | Validation                                   |
|----------------------|------------|---------:|----------------------------------------------|
| `censusDate`         | `LocalDate`| yes      | Must be between 1800-01-01 and 2100-12-31.   |
| `censusTime`         | `LocalTime`| yes      | Must be a valid time.                        |
| `level6Code`         | `String`   | yes      | Non-empty; must match existing HXPLVL6 code. |
| `includeVoided`      | `boolean`  | no       | Defaults to `false`.                         |
| `includeOutpatient`  | `boolean`  | no       | Defaults to `false`.                         |
| `includePreAdmission`| `boolean`  | no       | Defaults to `false`.                         |
| `maxRecords`         | `int`      | no       | Range 1–10000; default 1000.                 |

### 14.2 Layer Structure

- **Controller** – `CensusController`
  - Validates input parameters.
  - Enforces security and audit.

- **Service** – `CensusService`
  - Implements steps 1–6 from the processing flow.
  - Coordinates enrichment services (`StatusService`, `BenefitService`, `XmlService`).

- **Repository** – `HaptrfrRepository`, `TxpnstnRepository`, `TxpbnfitRepository`, `HxptabldRepository`, `HxplvlRepository`, `XmlHeaderRepository`, `XmlDetailRepository`.

### 14.3 Response JSON Shape

Sample JSON response:

```json
{
  "runId": "202607130251-001",
  "censusDateTime": "2026-07-13T02:51:00Z",
  "levelPath": "System > Region > Facility > Ward > Station",
  "summary": {
    "totalCandidates": 120,
    "excludedPreAdmission": 5,
    "excludedVoided": 3,
    "excludedOutpatient": 12,
    "finalCensusCount": 100
  },
  "rows": [
    {
      "level6Code": "123456",
      "accountNumber": "A00001234",
      "medicalRecordNumber": "MRN000987",
      "transferDate": "2026-07-12",
      "transferTime": "14:30",
      "transferType": "ADM",
      "statusCode": "ACTIVE",
      "stationDescription": "MEDICAL SURGICAL",
      "benefitNumber": "BN123",
      "planCode": "PPO",
      "coverageTelephone": "555-123-4567"
    }
  ]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record CensusRow(
    String level6Code,
    String accountNumber,
    String medicalRecordNumber,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String statusCode,
    String stationDescription,
    String benefitNumber,
    String planCode,
    String coverageTelephone
) {}

public record CensusResponse(
    String runId,
    ZonedDateTime censusDateTime,
    String levelPath,
    Summary summary,
    List<CensusRow> rows
) {}

public record Summary(
    int totalCandidates,
    int excludedPreAdmission,
    int excludedVoided,
    int excludedOutpatient,
    int finalCensusCount
) {}
```

---

## (15) Performance Considerations

### N+1 Risk Analysis

Legacy HABADTE reads HAPTRFR and then, per‑record, calls status and benefit lookups and performs XML writes. If naively implemented with one query per record for TXPNSTN and TXPBNFIT, a classic N+1 query problem will arise.

To mitigate:
- Use **set‑based joins** to pull status and benefits in bulk.
- Avoid per‑record repository calls; rely on single queries with joins and caching when necessary.

### Recommended Batch Approach

```sql
SELECT
    h.*, s.*, b.*
FROM dbo.HAPTRFR h
LEFT JOIN dbo.TXPNSTN s
    ON s.XFNLV6 = h.AFLVL6
   AND s.XFNSST = h.StatusCode
LEFT JOIN dbo.TXPBNFIT b
    ON b.XFBUBN = h.BenefitNumber
   AND b.XFBPLN = h.PlanCode
WHERE h.AFLVL6 = @level6Code
  AND h.AFTRDT <= @censusDate
  AND (h.AFTRDT < @censusDate OR h.AFTRTM <= @censusTime)
  AND h.FileIndicator <> 0
  AND h.VoidedFlag <> 'V'
  AND h.IOIndicator <> 'O';
```

---

## (16) Business Rules Reference Summary

All relevant rules for HABADTE and its key utilities:

| Rule ID | Description                                                                                                    |
|--------|----------------------------------------------------------------------------------------------------------------|
| BR-017 | Exclude pre-admissions (file indicator = 0) from active census.                                               |
| BR-018 | Exclude voided accounts (flag = 'V') from active census.                                                      |
| BR-019 | Exclude outpatient accounts (I/O indicator = 'O') from census.                                               |
| BR-001 | Skip text centering when the input field is blank.                                                            |
| BR-002 | Skip text centering when text is already left-aligned at position 1.                                          |
| BR-003 | Reject dates before 1800.                                                                                    |
| BR-004 | Reject dates beyond 2100.                                                                                    |
| BR-005 | Enforce month >= 01.                                                                                          |
| BR-006 | Enforce month <= 12.                                                                                          |
| BR-007 | Enforce day >= 01.                                                                                            |
| BR-008 | When VDD > DYS(VMM), branch to EXIT (date overflow rule).                                                     |
| BR-009 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-010 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-011 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-012 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-013 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-014 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-015 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-016 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-020 | HXXAPPPRF reads from HXPAPPPRF to retrieve configuration/reference data.                                      |

---

## (17) Edge Cases to Implement

The migration must implement explicit handling for the following scenarios.

| Scenario                              | Expected Behavior                                                              |
|--------------------------------------|-------------------------------------------------------------------------------|
| Null/zero date in HAPTRFR            | Treat `00000000` as `null`; record may be excluded or flagged for review.    |
| Invalid date range (before 1800)     | Reject record per BR-003; do not include in census.                           |
| Invalid date range (after 2100)      | Reject record per BR-004; do not include in census.                           |
| Month outside 1–12                   | Reject per BR-005/BR-006; log validation error.                               |
| Day <= 0                             | Reject per BR-007; log validation error.                                      |
| Not-found TXPNSTN status             | Leave status fields null; mark row as incomplete and log warning.            |
| Not-found TXPBNFIT benefit           | Default to generic/self-pay benefit; log warning.                             |
| Deleted/obsolete benefit/status rows | Exclude from joins; treat as not-found.                                       |
| XML ID collision                     | Retry ID generation; on repeated failure, abort run and log critical error.   |
| HXFXMLH update failure               | Mark run as failed; do not present partial results to callers.                |
| Empty result set after filters       | Return empty `rows` with summary counts; no error, but log informational event.|
| Preference not configured in tables  | Fall back to default behaviour; highlight configuration gap in logs.         |
