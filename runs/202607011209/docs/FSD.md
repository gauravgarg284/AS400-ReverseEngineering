# Business Processing Rules & Functional Specification

## HABADTE Inpatient Transfer Batch (HABADTE) /
### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

### (1) Business Purpose

HABADTE is a batch-oriented patient management program that processes inpatient transfer records and generates XML-based output for downstream systems. It reads transfer events, filters out voided or non-applicable records, enriches them with institution and configuration data, and emits structured messages.

The program is typically run as part of an overnight or scheduled batch cycle used by patient administration and integration teams. It answers the core business question:

> "For a given period and facility, which inpatient transfer events are active and valid, and how should they be represented for downstream consumption?"

Key outputs include:
- A stream of XML detail records written to HXFXMLD for each qualifying transfer.
- Updated XML header/control records in HXFXMLH summarizing batch execution.
- Implicit counts and flags of processed vs skipped records based on file, void, and inpatient/outpatient indicators.

### (2) Inputs (API Request Parameters)

In the legacy batch, HABADTE reads records directly from physical and logical files. In a modern API, these inputs become request parameters.

| Parameter | AS400 Field / Source | Type | Description |
|-----------|----------------------|------|-------------|
| facilityLevel6 | HAPTRFR.AFLVL6 | string | Level-6 facility or organization code identifying the institution/site. |
| accountNumber | HAPTRFR.AFACCT | string | Patient account number used to group transfers; PHI (AccountNumber). |
| transferDateFrom | HAPTRFR.AFTRDT | date | Start of transfer date range to process. |
| transferDateTo | HAPTRFR.AFTRDT | date | End of transfer date range to process. |
| transferTimeFrom | HAPTRFR.AFTRTM | time | Start of time window for transfers on each day. |
| transferTimeTo | HAPTRFR.AFTRTM | time | End of time window for transfers on each day. |
| transferType | HAPTRFR.AFTYPE | string | Transfer type filter (e.g., admission, move, discharge). |
| inpatientOnly | HABADTE.-INPATIENT/OUTPATIENT FLAG | boolean | When true, restricts to inpatient records; modeled from BR-019. |
| fileIndicatorRequired | HABADTE.-FILE INDICATOR | boolean | Controls skipping of records when file indicator is zero; modeled from BR-017. |
| includeVoided | HABADTE.-FLAG INDICATOR | boolean | When false, excludes void/voided records per BR-018. |

In addition, the current authenticated user identity (operator ID) is obtained from the security context rather than a file. For Spring Boot:
- OAuth2/JWT is used to propagate user identity and roles.
- RBAC ensures only users with patient-management privileges can invoke this batch endpoint.
- PHI access is audited using Spring Security filters and centralized logging.

### (3) Organizational Hierarchy

HABADTE relies on level-6 facility codes (AFLVL6, XFNLV6, MMPLV6) to link transfers, stations, and member master data. This can be modeled as a simple two-level hierarchy.

| Level | Name | Key Size (digits) | AS400 Table |
|-------|------|-------------------|-------------|
| 1 | Facility Group | 2–3 | OXPNSTN (XFNLV6) |
| 2 | Facility / Station | 4–6 | OXPNSTN (XFNSST), HAPTRFR (AFLVL6), OMPMAST (MMPLV6) |

**Business Rule Note:**  
Report headers and XML batches are grouped by level-6 facility (AFLVL6). Header segments in HXFXMLH include facility identification derived from these codes.

### (4) Inpatient Transfer Data Source

The primary entity for HABADTE is the inpatient transfer record.

#### 4.1 Data Access Pattern

Legacy pattern:
- Primary file: **HAPTRFR** (PF) record format HAFTRFR.
- Access type: keyed reads over AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE.
- Sort order: by facility (AFLVL6), then account (AFACCT), then chronological transfer date/time (AFTRDT/AFTRTM), then type (AFTYPE).

Modern SQL Server equivalent:

```sql
SELECT  AFLVL6,
        AFACCT,
        AFTRDT,
        AFTRTM,
        AFTYPE,
        AFMRNO,
        /* other non-PHI fields */
FROM    HAPTRFR
WHERE   AFLVL6 = :facilityLevel6
  AND   AFACCT = :accountNumber
  AND   AFTRDT BETWEEN :transferDateFrom AND :transferDateTo
  AND   AFTRTM BETWEEN :transferTimeFrom AND :transferTimeTo
  AND   AFTYPE = :transferType
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

#### 4.2 Key Fields

| SQL Column | AS400 Field | Type | Description |
|------------|------------|------|-------------|
| facility_level6 | AFLVL6 | CHAR | Level-6 facility or organizational code. |
| account_number | AFACCT | CHAR | Patient account number; must be treated as PHI. |
| transfer_date | AFTRDT | DECIMAL(8,0) | Date of the transfer (YYYYMMDD). |
| transfer_time | AFTRTM | DECIMAL(4,0) | Time of the transfer (HHMM). |
| transfer_type | AFTYPE | CHAR | Transfer type or event category. |

### (5) Inclusion and Exclusion Rules

Inclusion/exclusion logic combines batch filters with the three HABADTE rules from approved_rules.

#### BR-017 – File Indicator Skip

**Description**  
Records with a file indicator equal to zero are considered non-file or inactive and are skipped from processing.

**Pseudocode**

```pseudo
IF fileIndicator = 0 THEN
    SKIP_RECORD()
    CONTINUE
ENDIF
```

**SQL WHERE Fragment**

```sql
/* BR-017: Exclude records with file indicator = 0 */
AND file_indicator <> 0
```

#### BR-018 – Void/Void Flag Skip

**Description**  
Records flagged as void or voided are excluded to prevent cancelled or reversed transfers from appearing in downstream outputs.

**Pseudocode**

```pseudo
IF flag_indicator IN ('VOID', 'VOIDED') THEN
    SKIP_RECORD()
    CONTINUE
ENDIF
```

**SQL WHERE Fragment**

```sql
/* BR-018: Exclude voided transfers */
AND flag_indicator NOT IN ('VOID', 'VOIDED')
```

#### BR-019 – Outpatient Flag Skip

**Description**  
Records with an inpatient/outpatient flag set to outpatient are skipped; the batch focuses on inpatient transfers only.

**Pseudocode**

```pseudo
IF in_out_flag = 'OUTPATIENT' THEN
    SKIP_RECORD()
    CONTINUE
ENDIF
```

**SQL WHERE Fragment**

```sql
/* BR-019: Only inpatient records */
AND in_out_flag <> 'OUTPATIENT'
```

#### Summary SQL WHERE Clause

```sql
WHERE AFLVL6 = :facilityLevel6                      -- primary facility filter
  AND AFACCT = :accountNumber                       -- account-level selection
  AND AFTRDT BETWEEN :transferDateFrom AND :transferDateTo
  AND AFTRTM BETWEEN :transferTimeFrom AND :transferTimeTo
  AND AFTYPE = :transferType
  AND file_indicator <> 0                           -- BR-017
  AND flag_indicator NOT IN ('VOID', 'VOIDED')      -- BR-018
  AND in_out_flag <> 'OUTPATIENT'                   -- BR-019
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

### (6) Institution and Station Enrichment

HABADTE enriches transfer records with institution/station metadata via the OXPNSTN/HXPNSTN lineage.

**Data Lineage**  
- `TXPNSTN -> HXPNSTN (LF) -> HABADTE (DECLARE)`
- `OXPNSTN (PF) -> HABADTE (READ via XFFNSTN)`

#### Algorithm

1. For each transfer record, derive level-6 facility code (AFLVL6).
2. Read institution station record from OXPNSTN keyed by (XFNLV6 = AFLVL6, XFNSST = stationCode).
3. If not found, fall back to HXPNSTN logical view over TXPNSTN using the same keys.
4. Use institution descriptors (name, class, location) to populate XML header and detail fields.

#### SQL SELECT Equivalent

```sql
SELECT  s.XFNLV6,
        s.XFNSST,
        s.station_name,
        s.station_class,
        s.location_code
FROM    OXPNSTN AS s
WHERE   s.XFNLV6 = :facilityLevel6
  AND   s.XFNSST = :stationCode;
```

#### Edge Cases

- **Not found:** if no station record exists, mark the transfer as having an unknown station and populate default descriptors.
- **Delete-flag logic:** if the station record includes a logical delete flag, skip enrichment and log a configuration issue.
- **Derived flags:** derive flags such as "isCriticalCare" based on station_class.

### (7) Benefit Plan Enrichment

HABADTE also references benefit data via HXPBNFIT/OXPBNFIT.

**Data Lineage**  
- `TXPBNFIT -> HXPBNFIT (LF) -> HABADTE (DECLARE)`
- `OXPBNFIT (PF)` provides additional benefit attributes.

#### Algorithm

1. From each transfer record, identify benefit number and plan (XFBUBN, XFBPLN) from related dictionaries.
2. Lookup benefit details through HXPBNFIT (logical file over TXPBNFIT).
3. Use OXPBNFIT for enriched data such as phone contacts and coverage limits.
4. Populate XML detail fields with benefit descriptions and contact information.

#### SQL SELECT Equivalent

```sql
SELECT  b.XFBUBN,
        b.XFBPLN,
        b.benefit_desc,
        b.coverage_type,
        b.XFBTEL
FROM    OXPBNFIT AS b
WHERE   b.XFBUBN = :benefitNumber
  AND   b.XFBPLN = :benefitPlan;
```

#### Edge Cases

- **Not found:** if benefit data is missing, mark the XML segment as having "UNKNOWN BENEFIT" but still emit the transfer.
- **Delete flag:** if benefits are marked inactive, suppress optional benefit descriptors.
- **PHI:** treat XFBTEL as PHI; do not expose raw phone numbers to unauthorized consumers.

### (8) Level Descriptor Enrichment

XFXLDSC enriches records with multi-level descriptors using HXPLVL1–HXPLVL6 and their READ lineage.

#### Algorithm

1. For each level code present in the transfer (e.g., L1–L6), call XFXLDSC.
2. XFXLDSC declares and reads HXPLVL1–HXPLVL6 to retrieve descriptor text and classification.
3. Apply the LDAMAP boundary rules (BR-009–BR-012) to ensure map values are within allowed ranges.
4. Assemble a composite descriptor string attached to each XML record.

#### SQL SELECT Example

```sql
SELECT  l1.HX1NUM, l1.level1_desc,
        l2.HX2NUM, l2.level2_desc,
        l3.HX3NUM, l3.level3_desc,
        l4.HX4NUM, l4.level4_desc,
        l5.HX5NUM, l5.level5_desc,
        l6.HX6NUM, l6.level6_desc
FROM    HXPLVL1 AS l1
JOIN    HXPLVL2 AS l2 ON l2.parent_lvl1 = l1.HX1NUM
JOIN    HXPLVL3 AS l3 ON l3.parent_lvl2 = l2.HX2NUM
JOIN    HXPLVL4 AS l4 ON l4.parent_lvl3 = l3.HX3NUM
JOIN    HXPLVL5 AS l5 ON l5.parent_lvl4 = l4.HX4NUM
JOIN    HXPLVL6 AS l6 ON l6.parent_lvl5 = l5.HX5NUM
WHERE   l6.HX6NUM = :level6Code;
```

#### Edge Cases

- Map values above configured thresholds cause the record to be skipped per BR-009–BR-012.
- Missing level records produce partial descriptors but the transfer may still be emitted.

### (9) Counting Rules

Counters in HABADTE and XFXCNTR track processed vs skipped records and control loop termination.

| Counter Name | Incremented When | Description |
|--------------|------------------|-------------|
| totalRecordsRead | Each record read from HAPTRFR | Raw count of all transfer records examined. |
| totalRecordsSkippedFile | fileIndicator = 0 | Records skipped due to file suppression (BR-017). |
| totalRecordsSkippedVoid | flag_indicator IN ('VOID','VOIDED') | Records skipped due to void/voided status (BR-018). |
| totalRecordsSkippedOutpatient | in_out_flag = 'OUTPATIENT' | Records skipped because they are outpatient (BR-019). |
| totalRecordsEmitted | A record passes all filters and is written to HXFXMLD | Count of XML detail records emitted.
| counterX | internal loop variable X used by XFXCNTR | Controls loop termination based on BR-001/BR-002.

Relationship and Business Meaning:
- `totalRecordsRead = totalRecordsSkipped* + totalRecordsEmitted`.  
- High skipped counts may indicate configuration issues (e.g., many void or outpatient transfers being included in the batch window).

### (10) Output Data Structure

HABADTE writes XML header and detail records to HXFXMLH and HXFXMLD.

#### 10.1 Header Fields (HXFXMLH)

| Field | Description |
|-------|-------------|
| batch_id | Unique batch identifier for this run. |
| facility_level6 | Level-6 facility code summarizing the batch. |
| run_timestamp | Date-time of batch execution. |
| total_records_emitted | Count of XML detail records written. |
| total_records_skipped | Aggregate of skipped records by reason. |

#### 10.2 Detail Row Fields (HXFXMLD)

| Field | Description |
|-------|-------------|
| batch_id | Links detail to header. |
| account_number | AFACCT for the transfer; PHI. |
| mrn | AFMRNO or OMPMAST.MMMRNO; PHI. |
| transfer_datetime | Combined AFTRDT/AFTRTM. |
| transfer_type | AFTYPE value. |
| station_code | Derived from XFNSST. |
| station_descriptor | Enriched descriptor from OXPNSTN/HXPNSTN. |
| level_descriptors | Composite descriptor from HXPLVL* tables. |
| benefit_summary | Derived from OXPBNFIT/HXPBNFIT. |

#### 10.3 Footer/Summary Fields

| Field | Description |
|-------|-------------|
| total_records_read | Count from HAPTRFR. |
| total_records_emitted | Matches header field. |
| total_records_skipped_file | Per BR-017. |
| total_records_skipped_void | Per BR-018. |
| total_records_skipped_outpatient | Per BR-019. |

#### 10.4 Sort Order

Detail records in HXFXMLD are sorted by:
1. facility_level6 (AFLVL6)  
2. account_number (AFACCT)  
3. transfer_datetime (AFTRDT/AFTRTM)  
4. transfer_type (AFTYPE)

### (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
    - Read runtime parameters (facilityLevel6, date/time range, transferType).
    - Initialize counters: totalRecordsRead, totalRecordsEmitted, skipped counters.
    - Open SQL connections to HAPTRFR, OXPNSTN, OXPBNFIT, HXPLVL*, HXFXMLH, HXFXMLD.

STEP 2: Preferences / Lookup Initialization
    - Preload station and benefit dictionaries where possible.
    - Prepare level descriptor caches using HXPLVL1–HXPLVL6.
    - Initialize XML header record in HXFXMLH with batch_id and facility.

STEP 3: Context Setup
    - Determine authenticated user from Spring Security context.
    - Validate user roles for patient management and PHI access.
    - Log batch start event with user, facility, and parameters.

STEP 4: Query Transfers
    - Execute main SELECT on HAPTRFR using filters and ORDER BY.

STEP 5: Per-Record Enrichment
    FOR each row in HAPTRFR result set DO
        totalRecordsRead++

        5a: Apply HABADTE filters
            - If file_indicator = 0 (BR-017) THEN
                totalRecordsSkippedFile++
                CONTINUE
            - If flag_indicator IN ('VOID','VOIDED') (BR-018) THEN
                totalRecordsSkippedVoid++
                CONTINUE
            - If in_out_flag = 'OUTPATIENT' (BR-019) THEN
                totalRecordsSkippedOutpatient++
                CONTINUE

        5b: Station enrichment
            - Lookup station in OXPNSTN using facility and station_code.
            - If not found, attempt HXPNSTN logical view.
            - Derive station_descriptor and flags.

        5c: Benefit enrichment
            - Identify benefitNumber/benefitPlan.
            - Lookup benefit in OXPBNFIT/HXPBNFIT.
            - Derive benefit_summary and ensure PHI restrictions on phone.

        5d: Level descriptor enrichment
            - Call descriptor service (modern equivalent of XFXLDSC).
            - Validate LDAMAP using BR-009–BR-012.
            - Build level_descriptors string.

        5e: XML detail creation
            - Build HXFXMLD row with header linkage, transfer, station, benefit, descriptors.
            - Write HXFXMLD record.
            - totalRecordsEmitted++
    END FOR

STEP 6: Assemble Response / Finalize
    - Update HXFXMLH with summary counts and completion timestamp.
    - Commit all changes.
    - Return API response containing batch_id, counts, and optional preview of emitted details.
```

### (12) Data Type Conversions

#### 12.1 Date Fields

AS400 transfer dates use a DECIMAL(8,0) representation (YYYYMMDD).

Conversion:
- If value = 0, treat as null.
- Else, parse into `LocalDate`.

```java
Long rawDate = rs.getLong("AFTRDT");
LocalDate transferDate = (rawDate == 0L)
    ? null
    : LocalDate.parse(rawDate.toString(), DateTimeFormatter.ofPattern("yyyyMMdd"));
```

#### 12.2 Time Fields

Times use DECIMAL(4,0) representation (HHMM).

```java
int rawTime = rs.getInt("AFTRTM");
LocalTime transferTime = (rawTime == 0)
    ? null
    : LocalTime.of(rawTime / 100, rawTime % 100);
```

#### 12.3 Packed Decimal Keys

Numeric keys such as AFLVL6 and account numbers may be stored as DECIMAL without scale. In Java, map to `Long` or `String` depending on leading zeros.

```java
long facilityLevel6 = rs.getLong("AFLVL6");
String accountNumber = rs.getString("AFACCT").trim();
```

#### 12.4 String Trimming

Fixed-length CHAR fields are right-padded with spaces.

```java
String stationCode = rs.getString("XFNSST").trim();
String benefitPlan = rs.getString("XFBPLN").trim();
```

### (13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR | dbo.InpatientTransfer | Core transfer events (inpatient-focused). |
| OXPNSTN | dbo.InstitutionStation | Station and facility metadata. |
| HXPNSTN | dbo.InstitutionStationLegacyView | Legacy-access view over institution data. |
| OXPBNFIT | dbo.BenefitPlan | Benefit definitions and contact information. |
| HXPBNFIT | dbo.BenefitPlanLegacyView | Legacy-access view over benefits. |
| HXPLVL1–HXPLVL6 | dbo.LevelConfig1–6 | Hierarchical level configuration tables. |
| HXPTABLD | dbo.DictionaryTable | Generic dictionary for code translation. |
| HXPXMLH | dbo.XmlTransferHeader | XML header/control records. |
| HXPXMLD | dbo.XmlTransferDetail | XML detail records per transfer. |

#### Suggested SQL Server Indexes

```sql
CREATE INDEX IX_InpatientTransfer_FacilityAccountDateTime
ON dbo.InpatientTransfer (facility_level6, account_number, transfer_date, transfer_time, transfer_type);

CREATE INDEX IX_InstitutionStation_FacilityStation
ON dbo.InstitutionStation (facility_level6, station_code);

CREATE INDEX IX_BenefitPlan_NumberPlan
ON dbo.BenefitPlan (benefit_number, benefit_plan);

CREATE INDEX IX_LevelConfig6_LevelCode
ON dbo.LevelConfig6 (level6_code);

CREATE INDEX IX_XmlTransferDetail_BatchAccountDateTime
ON dbo.XmlTransferDetail (batch_id, account_number, transfer_datetime);
```

### (14) Spring Boot API Design

#### 14.1 Recommended REST Endpoint

- Method: `POST`
- Path: `/api/batch/inpatient-transfers`

| Parameter | Type | Required | Validation |
|-----------|------|----------|-----------|
| facilityLevel6 | String | Yes | Not blank; matches known facility codes. |
| accountNumber | String | No | If present, must match account pattern. |
| transferDateFrom | LocalDate | Yes | Before or equal to transferDateTo. |
| transferDateTo | LocalDate | Yes | Not in the future beyond allowed window. |
| transferTimeFrom | LocalTime | No | Optional; must be < transferTimeTo when both sent. |
| transferTimeTo | LocalTime | No | Optional. |
| transferType | String | No | Must be one of configured transfer types. |
| inpatientOnly | Boolean | No | Default true. |

#### 14.2 Layer Structure

- **Controller:** `InpatientTransferBatchController` – validates request, triggers service.
- **Service:** `InpatientTransferBatchService` – implements steps 1–6, orchestrating repositories and mappers.
- **Repositories:**
  - `InpatientTransferRepository` (HAPTRFR).
  - `InstitutionStationRepository` (OXPNSTN/HXPNSTN).
  - `BenefitPlanRepository` (OXPBNFIT/HXPBNFIT).
  - `LevelConfigRepository` (HXPLVL*).
  - `XmlTransferHeaderRepository`, `XmlTransferDetailRepository` (HXPXMLH/HXPXMLD).

#### 14.3 Response JSON Shape

```json
{
  "batchId": "202607011209-HABADTE",
  "facilityLevel6": "123456",
  "parameters": {
    "transferDateFrom": "2026-07-01",
    "transferDateTo": "2026-07-01",
    "transferType": "ADMIT"
  },
  "counts": {
    "totalRecordsRead": 1500,
    "totalRecordsEmitted": 1200,
    "totalRecordsSkippedFile": 50,
    "totalRecordsSkippedVoid": 100,
    "totalRecordsSkippedOutpatient": 150
  }
}
```

#### 14.4 Java Entity/DTO Sketch

```java
public record InpatientTransfer(
    String facilityLevel6,
    String accountNumber,
    String mrn,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String stationCode
) {}

public record XmlTransferDetail(
    String batchId,
    String facilityLevel6,
    String accountNumber,
    String mrn,
    LocalDateTime transferDateTime,
    String transferType,
    String stationCode,
    String stationDescriptor,
    String levelDescriptors,
    String benefitSummary
) {}

public record InpatientTransferBatchResponse(
    String batchId,
    String facilityLevel6,
    Map<String, Object> parameters,
    Map<String, Integer> counts
) {}
```

### (15) Performance Considerations

The legacy design risks N+1 file reads:
- For each transfer, additional reads to OXPNSTN/HXPNSTN, OXPBNFIT/HXPBNFIT, and multiple HXPLVL* tables.

Modern mitigation:
- Use set-based SQL queries with joins rather than per-record lookups.

```sql
SELECT  t.*, s.station_name, b.benefit_desc, l6.level6_desc
FROM    dbo.InpatientTransfer AS t
LEFT JOIN dbo.InstitutionStation AS s
       ON s.facility_level6 = t.facility_level6
      AND s.station_code    = t.station_code
LEFT JOIN dbo.BenefitPlan AS b
       ON b.benefit_number  = t.benefit_number
      AND b.benefit_plan    = t.benefit_plan
LEFT JOIN dbo.LevelConfig6 AS l6
       ON l6.level6_code    = t.level6_code
WHERE   t.facility_level6 = :facilityLevel6
  AND   t.transfer_date BETWEEN :transferDateFrom AND :transferDateTo;
```

Batch-fetch dictionaries into memory (e.g., maps keyed by codes) when data is small and stable, reducing database hits.

### (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017 | Skip records when file indicator equals zero. |
| BR-018 | Skip records when flag indicator equals void/voided. |
| BR-019 | Skip records when inpatient/outpatient flag equals outpatient. |
| BR-009–BR-012 | Enforce LDAMAP map boundaries; skip records with out-of-range map values. |
| BR-001–BR-002 | Control loop termination or counter bounds based on X values. |
| BR-003–BR-008 | Enforce calendar validity (year, month, day) for date-related processing. |
| BR-013–BR-016 | Use *IN79 indicator to disable or early-exit specific table lookup paths.

### (17) Edge Cases to Implement

| Scenario | Expected Behavior |
|----------|-------------------|
| Transfer date = 0 (AFTRDT) | Treat date as null; log data quality issue but allow record if other fields are valid. |
| Transfer time = 0 (AFTRTM) | Treat time as null; assume midnight or leave undefined in XML. |
| fileIndicator = 0 | Skip record per BR-017; increment `totalRecordsSkippedFile`. |
| flag_indicator = 'VOID'/'VOIDED' | Skip record per BR-018; increment `totalRecordsSkippedVoid`. |
| in_out_flag = 'OUTPATIENT' | Skip record per BR-019; increment `totalRecordsSkippedOutpatient`. |
| Station lookup not found | Set stationDescriptor to "UNKNOWN"; do not fail batch. |
| Benefit lookup not found | Set benefitSummary to "UNKNOWN"; still emit transfer. |
| Level descriptor missing or LDAMAP out-of-range | Skip descriptor enrichment or whole record per BR-009–BR-012; log configuration issue. |
| Empty result set from HAPTRFR | Emit header with zero counts; no detail records. |
| Preference or configuration not present | Fall back to safe defaults; do not expose incomplete PHI in outputs.
