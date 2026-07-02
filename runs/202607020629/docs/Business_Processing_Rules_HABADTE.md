# Business Processing Rules & Functional Specification

## HABADTE Transfer Activity Detail Extract (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

(1) Business Purpose

The HABADTE program produces a detailed extract of patient transfer activity for inpatient stays. It is used by patient accounting and clinical operations teams to understand movement between service locations and to feed downstream reporting and billing processes.

> Core business question: "For a given patient account and level, what inpatient transfer events should be considered valid and included in the activity extract?"

The report produces a row per qualifying transfer, along with summary totals of valid transfers, skipped/voided records (file indicator, void flag, outpatient flag), and counts by inpatient/outpatient status.

(2) Inputs (API Request Parameters)

The Java/Spring Boot API that replaces HABADTE should accept the following request parameters. These are derived from the transfer and level keys used in the DDS and narratives for HABADTE and related files.

| Parameter            | AS400 Field | Type        | Description |
|----------------------|------------|------------|-------------|
| accountNumber        | AFACCT     | String(12) | Patient account number at level 6 used to select transfer records from HAPTRFR. |
| level6Identifier     | AFLVL6     | String(6)  | Level-6 organizational or patient grouping key used in HAPTRFR and related files. |
| transferFromDate     | AFTRDT     | LocalDate  | Inclusive start date for transfer selection; maps to transfer date field AFTRDT (DECIMAL 8,0). |
| transferToDate       | AFTRDT     | LocalDate  | Inclusive end date for transfer selection; maps to AFTRDT; applied as between condition. |
| inpatientOnly        | AFTYPE / INOUT flag | Boolean | When true, restricts output to inpatient transfers using -INPATIENT/OUTPATIENT FLAG rule. |
| includeVoided        | FLAG indicator | Boolean | When false, skip records with void/voided flag based on BR-018. |
| fileIndicatorFilter  | FILE indicator | Integer   | Optional explicit filter for file indicator; default logic uses BR-017 to skip zero. |

The current AS400 implementation also implicitly uses the current user identity and security context (job user profile) to derive printer destination and XML header values. In the Spring Boot implementation, this should be mapped to OAuth2 authentication with RBAC roles (e.g. ROLE_BILLING_ANALYST, ROLE_CLINICAL_ANALYST) and PHI audit logging so that each extract request is tied to a user principal and purpose-of-use.

(3) Organizational Hierarchy

HABADTE operates on level-based patient identifiers that are part of an organizational/payer hierarchy, using level-6 and plan/station keys.

| Level Number | Name           | Key Size (digits) | AS400 Table |
|-------------|----------------|-------------------|-------------|
| 1           | Enterprise     | HX1NUM            | HXPLVL1     |
| 2           | Network        | HX2NUM            | HXPLVL2     |
| 3           | Region         | HX3NUM            | HXPLVL3     |
| 4           | Facility Group | HX4NUM            | HXPLVL4     |
| 5           | Facility       | HX5NUM            | HXPLVL5     |
| 6           | Patient Level  | HX6NUM / AFLVL6   | HXPLVL6 / HAPTRFR |

Business rule note: HABADTE uses the level-6 key (AFLVL6) to select transfers within the patient account hierarchy. The report header should display the organization name at level 6 (patient-level grouping) and may optionally show higher levels when resolved via the HXPLVLn tables.

(4) Patient Transfer Data Source

#### 4.1 Data Access Pattern

The primary data source for HABADTE is the HAPTRFR physical file, which contains one record per transfer event for a patient account at a given level.

- Primary file: HAPTRFR (record format HAFTRFR)
- Access pattern: keyed read by AFLVL6 (level 6) and AFACCT (account), with additional filtering by AFTRDT (date) and AFTYPE (inpatient/outpatient).
- Sort order: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE — this ensures chronological ordering within account and separates transfer types.

Equivalent SQL Server SELECT:

```sql
SELECT
  AFLVL6,
  AFACCT,
  AFTRDT,
  AFTRTM,
  AFTYPE,
  AFMRNO
FROM dbo.HAPTRFR
WHERE AFLVL6 = :level6Identifier
  AND AFACCT = :accountNumber
  AND AFTRDT BETWEEN :transferFromDate AND :transferToDate
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

#### 4.2 Key Fields

| SQL Column | AS400 Field | Type        | Description |
|-----------|-------------|------------|-------------|
| AFLVL6    | AFLVL6      | CHAR(6)    | Level-6 key defining the patient or contract grouping for transfers; part of primary key and organizational hierarchy. |
| AFACCT    | AFACCT      | CHAR(12)   | Patient account number; PHI-bearing account identifier used as part of the primary key. |
| AFTRDT    | AFTRDT      | DECIMAL(8,0) | Transfer date in YYYYMMDD numeric format. |
| AFTRTM    | AFTRTM      | DECIMAL(4,0) | Transfer time in HHMM numeric format. |
| AFTYPE    | AFTYPE      | CHAR(1)    | Inpatient/outpatient indicator or transfer type code used to apply BR-019. |
| AFMRNO    | AFMRNO      | CHAR(10)   | Medical record number (MRN) of the patient; PHI field included in output for traceability. |

(5) Inclusion and Exclusion Rules

HABADTE applies three primary business rules to determine whether a transfer record is included in the output. These rules are captured as BR-017, BR-018, and BR-019 and are reinforced in the program narrative.

##### BR-017 – File Indicator Zero → Skip

Description: If the internal file indicator field for a transfer record equals zero (meaning the record is marked as unavailable or logically deleted), the record is skipped and not included in the extract.

Pseudocode:

```pseudo
IF FileIndicator = 0 THEN
  SKIP_RECORD;
ELSE
  PROCESS_RECORD;
ENDIF;
```

SQL WHERE clause fragment:

```sql
-- BR-017: Skip records with file indicator = 0
WHERE FileIndicator <> 0
```

##### BR-018 – Voided Flag → Skip

Description: If the record's flag indicator denotes a voided or cancelled transfer (void/voided), the record is excluded from the extract. This ensures that only active transfers are reported.

Pseudocode:

```pseudo
IF FlagIndicator IN ('VOID', 'VO') THEN
  SKIP_RECORD;
ELSE
  PROCESS_RECORD;
ENDIF;
```

SQL WHERE clause fragment:

```sql
-- BR-018: Exclude voided transfers
AND FlagIndicator NOT IN ('VOID', 'VO')
```

##### BR-019 – Outpatient Records → Skip

Description: If the inpatient/outpatient flag indicates outpatient status, the record is skipped. HABADTE focuses on inpatient transfer activity; outpatient movements are not reported in this extract.

Pseudocode:

```pseudo
IF InOutFlag = 'O' /* outpatient */ THEN
  SKIP_RECORD;
ELSE
  PROCESS_RECORD; -- inpatient and any other qualifying types
ENDIF;
```

SQL WHERE clause fragment:

```sql
-- BR-019: Include only inpatient transfers
AND InOutFlag <> 'O'
```

##### Summary SQL WHERE Clause

```sql
WHERE AFLVL6 = :level6Identifier
  AND AFACCT = :accountNumber
  AND AFTRDT BETWEEN :transferFromDate AND :transferToDate
  -- BR-017: skip records with file indicator = 0
  AND FileIndicator <> 0
  -- BR-018: exclude voided transfers
  AND FlagIndicator NOT IN ('VOID', 'VO')
  -- BR-019: include only inpatient transfers
  AND InOutFlag <> 'O'
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

(6) Benefit Plan Enrichment – Coverage and Contact Details

HABADTE enriches each transfer record with benefit plan information using the HXPBNFIT logical file over TXPBNFIT and the OXPBNFIT physical file.

- Business concept: Patient Benefit Plan and Contact Details
- Primary objects: HXPBNFIT (LF) over TXPBNFIT (PF), OXPBNFIT (PF)
- Key fields: XFBUBN (benefit number), XFBPLN (plan code), XFBTEL (contact phone number)

BR reference: derived enrichment rule (implicit) – for each transfer, lookup the active benefit plan to obtain coverage attributes and a contact telephone number (XFBTEL).

Algorithm steps:

1. Derive benefit plan key (benefit number and plan code) from the patient account and level.
2. Read HXPBNFIT using key (XFBUBN, XFBPLN).
3. If multiple benefit records exist, select the current/primary plan based on internal status or effective dates.
4. Populate enrichment fields: plan description, coverage flags, contact telephone (XFBTEL).
5. Attach the enriched benefit data to the transfer output row.

SQL SELECT equivalent:

```sql
SELECT
  b.XFBUBN,
  b.XFBPLN,
  b.XFBTEL,
  b.PlanDesc,
  b.CoverageCode
FROM dbo.HAPTRFR t
JOIN dbo.HXPBNFIT b
  ON b.AccountNumber = t.AFACCT
 AND b.Level6 = t.AFLVL6
WHERE t.AFLVL6 = :level6Identifier
  AND t.AFACCT = :accountNumber;
```

Edge cases:

- Not found: if no benefit plan is found, default coverage indicators to "UNKNOWN" and leave contact phone blank; log a warning for data quality monitoring.
- Delete flag: if the benefit record is marked deleted or inactive, treat as not-found for the purposes of enrichment.
- Derived flags: compute boolean flags such as `hasActiveCoverage` and `isPrimaryPlan` in the service layer based on native codes.

(7) Level Hierarchy Enrichment – Organizational Labels

HABADTE uses level hierarchy files (HXPLVL1–HXPLVL6 and their DDS record formats) through XFXLDSC to enrich records with human-readable level descriptions.

- Business concept: Organizational Level Descriptions
- Primary objects: HXPLVL1–HXPLVL6 physical files, HXFLVL1–HXFLVL6 record formats.

Algorithm steps:

1. For each transfer, obtain level keys (HX1NUM…HX6NUM) from account context.
2. For each level file HXPLVLn, read the record by its key to obtain a name/description (e.g. enterprise, network, facility).
3. Attach these descriptions as part of the header or row-level metadata.

SQL SELECT equivalent:

```sql
SELECT
  l1.HX1NUM, l1.Name AS Level1Name,
  l2.HX2NUM, l2.Name AS Level2Name,
  l3.HX3NUM, l3.Name AS Level3Name,
  l4.HX4NUM, l4.Name AS Level4Name,
  l5.HX5NUM, l5.Name AS Level5Name,
  l6.HX6NUM, l6.Name AS Level6Name
FROM dbo.LevelContext c
LEFT JOIN dbo.HXPLVL1 l1 ON l1.HX1NUM = c.HX1NUM
LEFT JOIN dbo.HXPLVL2 l2 ON l2.HX2NUM = c.HX2NUM
LEFT JOIN dbo.HXPLVL3 l3 ON l3.HX3NUM = c.HX3NUM
LEFT JOIN dbo.HXPLVL4 l4 ON l4.HX4NUM = c.HX4NUM
LEFT JOIN dbo.HXPLVL5 l5 ON l5.HX5NUM = c.HX5NUM
LEFT JOIN dbo.HXPLVL6 l6 ON l6.HX6NUM = c.HX6NUM
WHERE c.AFACCT = :accountNumber
  AND c.AFLVL6 = :level6Identifier;
```

Edge cases:

- Missing level record: if a specific level description is not found, fall back to displaying the numeric key only and flag the missing description in logs.
- Partial hierarchy: some accounts may have only higher-level keys; ensure the UI is resilient to missing lower levels.

(8) Station Enrichment – Nursing Station and Location

HABADTE reads station information from the HXPNSTN logical file over TXPNSTN and the OXPNSTN physical file using the XFNLV6 and XFNSST keys.

- Business concept: Nursing Station / Patient Location
- Primary objects: HXPNSTN (LF) over TXPNSTN (PF), OXPNSTN (PF).

Algorithm steps:

1. For each transfer, derive station key (level6, station code) from context.
2. Lookup station record in HXPNSTN / OXPNSTN to obtain station name, type, and any capacity attributes.
3. Attach station description to the transfer row to indicate where the patient moved from/to.

SQL SELECT equivalent:

```sql
SELECT
  s.XFNLV6,
  s.XFNSST,
  s.StationName,
  s.StationType
FROM dbo.HAPTRFR t
JOIN dbo.HXPNSTN s
  ON s.XFNLV6 = t.AFLVL6
 AND s.XFNSST = t.StationCode
WHERE t.AFLVL6 = :level6Identifier
  AND t.AFACCT = :accountNumber;
```

Edge cases:

- Not-found station: fall back to displaying the raw station code; mark the record for data cleanup.
- Obsolete stations: apply logic to ignore stations marked closed/obsolete in the station master.

(9) Counting Rules

HABADTE maintains several counters based on the three key rules BR-017, BR-018, and BR-019.

| Counter Name             | Incremented When                            | Description |
|--------------------------|---------------------------------------------|-------------|
| totalRecordsRead         | For every record read from HAPTRFR.         | Tracks total volume of transfer records scanned. |
| totalValidTransfers      | When a record passes BR-017, BR-018, BR-019.| Number of inpatient transfer records included in the extract. |
| skippedFileIndicatorZero | When FileIndicator = 0 (BR-017).            | Measures records excluded because the underlying file indicates invalid/removed state. |
| skippedVoided            | When FlagIndicator denotes voided (BR-018). | Measures cancelled/voided transfers not present in output. |
| skippedOutpatient        | When InOutFlag = 'O' (BR-019).              | Measures outpatient movements filtered out. |

Relationship and business meaning:

- `totalValidTransfers` + `skippedFileIndicatorZero` + `skippedVoided` + `skippedOutpatient` should equal `totalRecordsRead`, ensuring accounting completeness.
- High `skippedVoided` counts may indicate workflow or data-entry issues causing frequent cancellations.

(10) Output Data Structure

HABADTE writes an extract structure, conceptually equivalent to a header/detail/footer layout.

#### 10.1 Header Fields

| Field            | Source              | Description |
|------------------|---------------------|-------------|
| reportRunDate    | System date         | Date the extract was generated. |
| accountNumber    | HAPTRFR.AFACCT      | Patient account identifier. |
| level6Identifier | HAPTRFR.AFLVL6      | Level-6 key for the patient/account. |
| facilityName     | HXPLVL5.Name        | Facility description from level hierarchy enrichment. |
| patientName      | OMPMAST.MMNAME      | Patient name from master file (when joined). |

#### 10.2 Detail Row Fields

| Field           | Source         | Description |
|-----------------|----------------|-------------|
| transferDate    | HAPTRFR.AFTRDT | Date of transfer event (YYYYMMDD). |
| transferTime    | HAPTRFR.AFTRTM | Time of transfer (HHMM). |
| transferType    | HAPTRFR.AFTYPE | Indicates inpatient/outpatient and type of movement. |
| mrn             | HAPTRFR.AFMRNO | Medical record number (MRN) for the patient. |
| stationCode     | Station key    | Nursing station or location code from HXPNSTN/OXPNSTN. |
| stationName     | HXPNSTN.Name   | Human-readable station description. |
| benefitPlan     | HXPBNFIT.PlanDesc | Benefit plan description. |
| benefitPhone    | HXPBNFIT.XFBTEL | Contact phone number for benefit plan (PHI). |

#### 10.3 Footer/Summary Fields

| Field                   | Source        | Description |
|-------------------------|---------------|-------------|
| totalRecordsRead        | Counter       | Total records read from HAPTRFR. |
| totalValidTransfers     | Counter       | Records included in output after applying BR-017–BR-019. |
| skippedFileIndicatorZero| Counter       | Records excluded due to file indicator zero. |
| skippedVoided           | Counter       | Records excluded due to void flag. |
| skippedOutpatient       | Counter       | Records excluded due to outpatient flag. |

#### 10.4 Sort Order

Primary sort order is by `transferDate`, `transferTime`, and `transferType`, grouped by `accountNumber` and `level6Identifier`. This matches the keyed sequence of AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE on HAPTRFR.

(11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
  - Load runtime parameters: accountNumber, level6Identifier, date range, flags.
  - Initialise counters: totalRecordsRead, totalValidTransfers, skipped*.
  - Resolve organizational hierarchy using HXPLVL1–HXPLVL6 for level keys.

STEP 2: Preferences/Lookup
  - Resolve benefit plan mappings via HXPBNFIT/OXPBNFIT.
  - Resolve station mappings via HXPNSTN/OXPNSTN.
  - Load any XML layout or printer preferences via HXPXMLD and PRINTER.

STEP 3: Context
  - Build execution context containing user identity, facility and level descriptions, plan details.

STEP 4: Query
  - Perform primary query against HAPTRFR as described in Section 4.
  - SQL example:

    SELECT *
    FROM dbo.HAPTRFR
    WHERE AFLVL6 = :level6Identifier
      AND AFACCT = :accountNumber
      AND AFTRDT BETWEEN :fromDate AND :toDate
    ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;

STEP 5: Per-Record Enrichment
  5a. Apply inclusion/exclusion rules (BR-017–BR-019):
      - If FileIndicator = 0 → increment skippedFileIndicatorZero, continue.
      - If FlagIndicator in ('VOID','VO') → increment skippedVoided, continue.
      - If InOutFlag = 'O' → increment skippedOutpatient, continue.
  5b. Benefit enrichment via HXPBNFIT/OXPBNFIT (Section 6).
  5c. Level hierarchy enrichment via HXPLVLn (Section 7).
  5d. Station enrichment via HXPNSTN/OXPNSTN (Section 8).

STEP 6: Assemble Response
  - Map enriched record fields to output DTO.
  - Aggregate counters into footer summary.
  - Write extract rows to XML/print format or REST response payload.
```

(12) Data Type Conversions

#### 12.1 Date Fields

- AS400 representation: DECIMAL(8,0) in YYYYMMDD.
- Java: `java.time.LocalDate`.
- Conversion: parse integer to string, then `LocalDate.parse` with `DateTimeFormatter.BASIC_ISO_DATE`.
- Special rule: value 0 represents null/blank date and should be mapped to `null`.

#### 12.2 Time Fields

- AS400 representation: DECIMAL(4,0) as HHMM.
- Java: `java.time.LocalTime`.
- Conversion: split into hours and minutes, e.g. 930 → 09:30.

#### 12.3 Packed Decimal Keys

- Key fields such as AFLVL6, AFACCT may be stored as packed or zoned decimals.
- Java: map to `Long` or `String` depending on leading zeros requirements.
- Conversion: use BigDecimal to read and then convert to `long` or formatted string preserving leading zeros.

#### 12.4 String Trimming

- Fixed-length CHAR fields are right-padded with spaces.
- In Java, trim on both ends for display fields; preserve significant internal spaces for names.

(13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR      | dbo.HAPTRFR      | Primary transfer activity table capturing patient account movements. |
| HXPBNFIT     | dbo.HXPBNFIT     | Benefit plan and contact lookup for enrichment. |
| HXPNSTN      | dbo.HXPNSTN      | Nursing station and location master for station enrichment. |
| HXPLVL1      | dbo.HXPLVL1      | Level 1 organizational hierarchy. |
| HXPLVL2      | dbo.HXPLVL2      | Level 2 organizational hierarchy. |
| HXPLVL3      | dbo.HXPLVL3      | Level 3 organizational hierarchy. |
| HXPLVL4      | dbo.HXPLVL4      | Level 4 organizational hierarchy. |
| HXPLVL5      | dbo.HXPLVL5      | Level 5 organizational hierarchy. |
| HXPLVL6      | dbo.HXPLVL6      | Level 6 patient/account hierarchy. |
| OXPBNFIT     | dbo.OXPBNFIT     | Physical benefit plan storage (if separate from HXPBNFIT). |
| OXPNSTN      | dbo.OXPNSTN      | Physical station master (if separate from HXPNSTN). |

### Suggested SQL Server Indexes

```sql
-- Primary query index on HAPTRFR
CREATE INDEX IX_HAPTRFR_AccountLevelDate
ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE);

-- Benefit lookup index
CREATE INDEX IX_HXPBNFIT_AccountLevel
ON dbo.HXPBNFIT (AccountNumber, Level6);

-- Station lookup index
CREATE INDEX IX_HXPNSTN_LevelStation
ON dbo.HXPNSTN (XFNLV6, XFNSST);

-- Hierarchy lookup indexes
CREATE INDEX IX_HXPLVL1_Key ON dbo.HXPLVL1 (HX1NUM);
CREATE INDEX IX_HXPLVL2_Key ON dbo.HXPLVL2 (HX2NUM);
CREATE INDEX IX_HXPLVL3_Key ON dbo.HXPLVL3 (HX3NUM);
CREATE INDEX IX_HXPLVL4_Key ON dbo.HXPLVL4 (HX4NUM);
CREATE INDEX IX_HXPLVL5_Key ON dbo.HXPLVL5 (HX5NUM);
CREATE INDEX IX_HXPLVL6_Key ON dbo.HXPLVL6 (HX6NUM);
```

(14) Spring Boot API Design

#### 14.1 Recommended REST Endpoint

- Method: GET
- Path: `/api/patient-transfers`

Parameters:

| Name               | Type        | Required | Validation |
|--------------------|------------|----------|-----------|
| accountNumber      | String      | Yes      | Non-empty, matches account format. |
| level6Identifier   | String      | Yes      | Non-empty, length 6. |
| transferFromDate   | LocalDate   | Yes      | Valid date, not after transferToDate. |
| transferToDate     | LocalDate   | Yes      | Valid date, not before transferFromDate. |
| inpatientOnly      | Boolean     | No       | Defaults to true. |
| includeVoided      | Boolean     | No       | Defaults to false. |

#### 14.2 Layer Structure

- Controller: `TransferActivityController` – exposes REST endpoint, handles request/response mapping.
- Service: `TransferActivityService` – implements business rules BR-017–BR-019 and enrichment logic.
- Repository: `HaptrfrRepository`, `BenefitPlanRepository`, `StationRepository`, `HierarchyRepository` – encapsulate SQL queries.

#### 14.3 Response JSON Shape

```json
{
  "accountNumber": "123456789012",
  "level6Identifier": "LVL006",
  "transfers": [
    {
      "transferDate": "2024-06-01",
      "transferTime": "09:30",
      "transferType": "I",
      "mrn": "MRN123456",
      "stationCode": "STN01",
      "stationName": "Med-Surg East",
      "benefitPlan": "PLAN-A",
      "benefitPhone": "555-123-4567"
    }
  ],
  "summary": {
    "totalRecordsRead": 120,
    "totalValidTransfers": 80,
    "skippedFileIndicatorZero": 5,
    "skippedVoided": 10,
    "skippedOutpatient": 25
  }
}
```

#### 14.4 Java Entity/DTO Sketch

```java
public record TransferActivityRequest(
    String accountNumber,
    String level6Identifier,
    LocalDate transferFromDate,
    LocalDate transferToDate,
    boolean inpatientOnly,
    boolean includeVoided
) {}

public record TransferActivityRow(
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String mrn,
    String stationCode,
    String stationName,
    String benefitPlan,
    String benefitPhone
) {}

public record TransferActivityResponse(
    String accountNumber,
    String level6Identifier,
    List<TransferActivityRow> transfers,
    SummaryCounters summary
) {}

public record SummaryCounters(
    long totalRecordsRead,
    long totalValidTransfers,
    long skippedFileIndicatorZero,
    long skippedVoided,
    long skippedOutpatient
) {}
```

(15) Performance Considerations

The per-record enrichment pattern (benefit plan, hierarchy, station) introduces classic N+1 query risk if implemented naïvely. To avoid this:

- Use set-based queries to prefetch benefit plans, stations, and hierarchy records for all accounts/levels involved in the primary query.
- Cache lookups in memory during request processing using maps keyed by account/level and station code.

Example JOIN-based SQL:

```sql
SELECT
  t.AFLVL6,
  t.AFACCT,
  t.AFTRDT,
  t.AFTRTM,
  t.AFTYPE,
  t.AFMRNO,
  s.StationName,
  b.PlanDesc,
  b.XFBTEL
FROM dbo.HAPTRFR t
LEFT JOIN dbo.HXPNSTN s
  ON s.XFNLV6 = t.AFLVL6
 AND s.XFNSST = t.StationCode
LEFT JOIN dbo.HXPBNFIT b
  ON b.AccountNumber = t.AFACCT
 AND b.Level6 = t.AFLVL6
WHERE t.AFLVL6 = :level6Identifier
  AND t.AFACCT = :accountNumber
  AND t.AFTRDT BETWEEN :fromDate AND :toDate
  AND FileIndicator <> 0
  AND FlagIndicator NOT IN ('VOID','VO')
  AND InOutFlag <> 'O';
```

(16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

(17) Edge Cases to Implement

| Scenario                                 | Expected Behavior |
|------------------------------------------|-------------------|
| File indicator equals zero               | Exclude record from output; increment `skippedFileIndicatorZero`. |
| Flag indicator marks record as voided    | Exclude record; increment `skippedVoided`. |
| Inpatient/outpatient flag indicates 'O'  | Exclude record; increment `skippedOutpatient`. |
| Null or zero date fields (AFTRDT = 0)    | Treat as null; either exclude record or route to error handling depending on business policy. |
| Benefit plan not found for account/level | Include transfer, but leave benefit fields blank and flag in logs. |
| Station record not found                 | Include transfer, using raw station code and flag in logs. |
| Printer or XML preferences missing       | Use system defaults for output layout and destination. |
| Empty result set                         | Return empty `transfers` array with summary counters = 0. |
