# Business Processing Rules & Functional Specification

## Active Inpatient Census by Admit Date and Time (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

### (1) Business Purpose

HABADTE produces an "Active Accounts by Admit Date and Time" census report and a corresponding XML payload that summarises current inpatients per nursing station and organisational level. The report is used daily by patient flow, nursing management, and bed control teams to understand who is actively admitted, where they are located, and how many beds are occupied.

> Core business question: "For the selected facility and census date/time, which inpatient accounts are currently active, where are they located, and what are the aggregate counts by unit and room class?"

Summary outputs include:
- Per-account detail rows listing account number, room/bed, nursing station, admit date/time, and indicators.
- Station-level and facility-level totals of active inpatients.
- Breakdowns by room class and leave status (e.g., hospital/therapeutic leave).
- XML representation of the same data for downstream systems or archiving.

---

### (2) Inputs (API Request Parameters)

The HABADTE program reads a combination of LDA (Local Data Area) fields and request parameters that correspond to corporate level codes, census date/time, and user context. In the modernized API, these will be explicit request parameters.

| Parameter Name      | AS400 Field / Concept | Type      | Description |
|---------------------|-----------------------|-----------|------------|
| facilityLevel6      | MMPLV6 / AFLVL6       | string    | Corporate level-6 code identifying the facility or hospital for which the census is generated. |
| censusDate          | AFTRDT / MMADDT       | date      | Census date (CCYYMMDD) representing the desired admit date snapshot. |
| censusTime          | AFTRTM / MMADTM       | time      | Census time (HHMM) representing the cutoff time within the census date. |
| nursingStationCode  | XFNSST / MMPNST       | string    | Optional nursing station filter; when provided, limits the census to a specific station. |
| includeLeave        | indicator             | boolean   | Flag indicating whether patients on hospital/therapeutic leave should be included in counts. |
| mrnRollupMode       | profile flag          | string    | Mode returned by MRN roll-up profile (e.g., "BY_MRN" vs "BY_ACCOUNT"); controls grouping and header labels. |
| roomClassCode       | XFDTCD                | string    | Optional room class code filter used by table lookup for descriptions. |
| requestUserId       | LDAUSR/XMDUSR/XMRUSR | string    | Current user identity taken from LDA and propagated into XML records and audit trails. |

Notes:
- **Current user identity** is sourced from the LDA (HXXLDA) and written into HXPXMLD/HXPXMLR via fields XMDUSR/XMRUSR. This identity must be preserved in the modern system for auditability.
- **Spring Boot security mapping:** Requests must be authenticated via OAuth2/JWT with RBAC enforcing patient-management roles. Access to PHI-bearing endpoints (e.g., census detail including account and MRN data) must be logged with an audit trail linking `requestUserId` to each data fetch.

---

### (3) Organizational Hierarchy

HABADTE relies on a corporate hierarchy stored in HXPLVL1–HXPLVL6. Level 6 (HXPLVL6) is used most prominently for facility identification.

| Level Number | Name             | Key Size (digits) | AS400 Table |
|-------------|------------------|-------------------|-------------|
| 1           | Corporate Entity | 6                 | HXPLVL1     |
| 2           | Region           | 6                 | HXPLVL2     |
| 3           | Network          | 6                 | HXPLVL3     |
| 4           | System           | 6                 | HXPLVL4     |
| 5           | Division         | 6                 | HXPLVL5     |
| 6           | Facility/Hospital| 6                 | HXPLVL6     |

BR Note (report header format): HABADTE uses XFXLDSC to translate incoming level codes (from LDA and parameters) into descriptions. The report header prints the facility description and corresponding level codes; if any LDAMAP value used to resolve levels exceeds allowed ranges (see rules in XFXLDSC), the description is not printed and the process exits.

---

### (4) Patient Data Source

The primary entity is **inpatient account**, driven by admit date/time and nursing station. While the compact schema shows transfer file HAPTRFR and master file OMPMAST, the main sequential read pattern in HABADTE uses the logical file HMLMAST5H over TMPMAST keyed by nursing station and admit date/time.

#### 4.1 Data Access Pattern

AS400 pattern:
- Declare LF **HMLMAST5H** (PFILE TMPMAST) keyed by `MMPNST`, `MMADDT`, `MMADTM`.
- Set key fields from request parameters: `keyStation = nursingStationCode`, `keyDate = censusDate`, `keyTime = censusTime`.
- Perform keyed read starting at the specified station/date/time and iterate while records remain for the selected facility.
- For each record, optionally chain to **HXPNSTN** (nursing station definitions) and **HXPTABLD/HXLTABL* via XFXTABL** for room class enrichment.

SQL Server equivalent:

```sql
SELECT
    p.MMPLV6      AS facility_level6,
    p.MMACCT      AS account_number,
    p.MMMRNO      AS mrn,
    p.MMNAME      AS patient_name,
    p.MMPNST      AS nursing_station,
    p.MMADDT      AS admit_date,
    p.MMADTM      AS admit_time,
    p.room_class,
    p.leave_flag
FROM dbo.PatientMaster p
WHERE p.MMADDT = @censusDate
  AND p.MMADTM <= @censusTime
  AND (@nursingStationCode IS NULL OR p.MMPNST = @nursingStationCode)
  AND p.MMPLV6 = @facilityLevel6
ORDER BY p.MMPNST, p.MMADDT, p.MMADTM;
```

Here `dbo.PatientMaster` maps to the OMPMAST/TMPMAST concept, potentially merged in modernization.

#### 4.2 Key Fields

Key fields for the primary logical/physical files:

| SQL Column       | AS400 Field | Type      | Description |
|------------------|-------------|-----------|------------|
| facility_level6  | MMPLV6      | numeric   | Level-6 code representing the facility. |
| account_number   | MMACCT      | numeric   | Patient account number; unique within facility. |
| nursing_station  | MMPNST      | char      | Nursing station code. |
| admit_date       | MMADDT      | decimal   | Admit date (CCYYMMDD). |
| admit_time       | MMADTM      | decimal   | Admit time (HHMM). |

These correspond logically to keys used by HMLMAST5H and OMPMAST.

---

### (5) Inclusion and Exclusion Rules

Section 5 aggregates all HABADTE-specific filter logic from approved rules and key_rules.

#### BR-017 – File Indicator Must Be Active

When the internal file indicator flag equals zero (no active data), the record is skipped.

- **Description:** HABADTE only processes records where the file indicator is non-zero, meaning the data is present and active in the underlying files.

Pseudocode:

```pseudo
if fileIndicator = 0 then
    goto SKIP_RECORD; // BR-017
endif;
```

SQL WHERE fragment:

```sql
WHERE file_indicator <> 0
```

#### BR-018 – Voided Records Are Excluded

When the flag indicator equals `void` or `voided`, the record is skipped.

- **Description:** Voided accounts must not appear in the active census; their presence would distort counts and could expose stale or cancelled PHI.

Pseudocode:

```pseudo
if flag_indicator in ('VOID', 'VOIDED') then
    goto SKIP_RECORD; // BR-018
endif;
```

SQL WHERE fragment:

```sql
WHERE flag_indicator NOT IN ('VOID', 'VOIDED')
```

#### BR-019 – Outpatient Records Are Excluded

When the inpatient/outpatient flag indicates outpatient status, the record is skipped.

- **Description:** HABADTE is an inpatient census; outpatient visits are excluded even if they have similar account structures.

Pseudocode:

```pseudo
if ip_op_flag = 'O' then
    goto SKIP_RECORD; // BR-019
endif;
```

SQL WHERE fragment:

```sql
WHERE ip_op_flag <> 'O'
```

#### Summary SQL WHERE Clause

Combining BR-017–BR-019:

```sql
WHERE file_indicator <> 0          -- BR-017
  AND flag_indicator NOT IN ('VOID', 'VOIDED')  -- BR-018
  AND ip_op_flag <> 'O'            -- BR-019
ORDER BY nursing_station, admit_date, admit_time;
```

---

### (6) Nursing Station Enrichment (Station Name Lookup)

This section describes the first enrichment step, which resolves nursing station codes into human-friendly names.

- **Business Concept:** "Nursing Station Lookup" – mapping `MMPNST` codes to descriptive station names.
- **Data Sources:**
  - Logical file **HXPNSTN** over PF TXPNSTN (keys XFNLV6, XFNSST).
  - PF **OXPNSTN** (XFFNSTN) as the underlying table in the compact schema.

Algorithm steps:
1. From the primary record, take `facilityLevel6` (MMPLV6) and `nursingStationCode` (MMPNST).
2. Construct key: `XFNLV6 = facilityLevel6`, `XFNSST = nursingStationCode`.
3. Chain to HXPNSTN (or XFFNSTN) using the key.
4. If record is found, retrieve station description fields (e.g., long/short names) and store into `station_name`.
5. If not found, set `station_name = nursingStationCode` and flag an enrichment warning.

SQL SELECT equivalent:

```sql
SELECT n.station_name
FROM dbo.NursingStation n
WHERE n.facility_level6 = @facilityLevel6
  AND n.station_code = @nursingStationCode;
```

Edge cases:
- **Not-found handling:** If no station row exists, the report prints the code as-is and may increment a "missing station definition" counter.
- **Delete-flag logic:** If the station row has a delete or inactive flag, the program may still print the code but should not show descriptive name, to avoid misrepresenting closed units.
- **Derived flags:** Some station attributes might mark specialty units (e.g., ICU, pediatrics). These could be used later for speciality-level breakdowns.

---

### (7) Room Class Lookup (Benefit/Dictionary Enrichment)

Second enrichment step focuses on room class and benefit-related codes.

- **Business Concept:** "Room Class and Benefit Lookup" – mapping generic table codes to descriptions and leave flags.
- **Data Sources:**
  - PF **HXPTABLD** (XFFTABLD) and logical files **HXLTABLD/HXLTABLP/HXLTABLS**.
  - PF **OXPBNFIT** (XFFBNFIT) and LF **HXPBNFIT** over TXPBNFIT.
  - Utility program **XFXTABL** to read configuration tables.

Algorithm steps:
1. From the primary record (room/bed and account), derive a room class code (e.g., from HXPDICT or TMPMAST fields).
2. Call XFXTABL with `dataTypeCode = 'ROOM'` and `detailCode = roomClassCode`.
3. Inside XFXTABL:
   - Read from XFFTABLD and related tables using keys XFDTCD/XFDECD.
   - Apply BR-013–BR-016 (indicator *IN79) to determine whether to exit early.
4. If a mapping row is found, set: `room_class_description`, `leave_account_flag`, other derived flags.
5. Optionally enrich benefits information via HXPBNFIT/OXPBNFIT and contact phone (XFBTEL).

SQL SELECT examples:

```sql
SELECT t.long_description, t.short_description, t.leave_flag
FROM dbo.RoomClassTable t
WHERE t.data_type_code = 'ROOM'
  AND t.detail_code = @roomClassCode;
```

Edge cases:
- **Not-found:** Print room class code as-is and mark record for configuration review.
- **Invalid mapping:** If LDAMAP or table mapping keys are out of range, XFXTABL exits and HABADTE should default to a generic description.

---

### (8) Transfer History Enrichment

Although the main census is driven by master data, HABADTE accesses **HAPTRFR** to understand recent transfers.

- **Business Concept:** "Room Transfer History" – linking active accounts to last transfer events.
- **Data Source:**
  - PF **HAPTRFR** (HAFTRFR) keyed by AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE.

Algorithm steps:
1. For each active account from the primary query, construct key fields:
   - AFLVL6 = facilityLevel6
   - AFACCT = account_number
2. Read transfers where AFTRDT/AFTRTM are prior to or equal to censusDate/censusTime.
3. Choose the latest transfer record (max AFTRDT, AFTRTM) to determine current bed/room.
4. Overlay master record’s room/bed fields with transfer-derived values.

SQL example:

```sql
SELECT TOP 1
    t.AFTRDT AS last_transfer_date,
    t.AFTRTM AS last_transfer_time,
    t.room_number
FROM dbo.TransferHistory t
WHERE t.AFLVL6 = @facilityLevel6
  AND t.AFACCT = @accountNumber
  AND (t.AFTRDT < @censusDate
       OR (t.AFTRDT = @censusDate AND t.AFTRTM <= @censusTime))
ORDER BY t.AFTRDT DESC, t.AFTRTM DESC;
```

Edge cases:
- No transfer record: use room from master record.
- Transfers after census: ignore; census snapshot is as-of specified date/time.

---

### (9) Counting Rules

Counts are driven by key_rules from HABADTE and enrichment flags.

| Counter Name        | Incremented When                                         | Description |
|---------------------|----------------------------------------------------------|------------|
| totalActive         | Record passes BR-017, BR-018, BR-019 filters             | Number of active inpatient accounts at census. |
| totalLeave          | Record is active but has leave_flag = true               | Number of accounts on hospital/therapeutic leave. |
| totalInpatientBeds  | Active records grouped by room/bed and station          | Total occupied beds per facility. |

Relationships and meaning:
- `totalActive` is the base census count; `totalLeave` is a subset of `totalActive`.
- `totalInpatientBeds` may equal `totalActive` unless multi-bed accounts or double occupancy are allowed.

---

### (10) Output Data Structure

HABADTE produces both printed output and XML records.

#### 10.1 Header Fields

| Field           | Source        | Description |
|----------------|---------------|------------|
| facility_name  | HXPLVL6/HXXLEVEL/XFXLDSC | Facility description used in report heading. |
| census_date    | request param | Date printed in header. |
| census_time    | request param | Time printed in header. |
| mrn_rollupMode | XFXMRNROL/HXXAPPPRF | Indicates grouping (by MRN or account). |
| user_id        | HXXLDA/XMDUSR/XMRUSR | User who generated the report. |

#### 10.2 Detail Row Fields

| Field            | Source File   | Description |
|------------------|--------------|------------|
| account_number   | TMPMAST/OMPMAST | Unique account identifier. |
| mrn              | TMPMAST/OMPMAST | Medical Record Number. |
| patient_name     | TMPMAST/OMPMAST/HXPDICT | Patient name for display. |
| nursing_station  | TMPMAST/HMLMAST5H/HXPNSTN | Station code for the bed. |
| station_name     | HXPNSTN      | Human-friendly station description. |
| room_number      | TMPMAST/HXPDICT/HAPTRFR | Room/bed location. |
| admit_date       | TMPMAST      | Date of admission. |
| admit_time       | TMPMAST      | Time of admission. |
| room_class       | HXPTABLD/XFXTABL | Room class code. |
| room_class_desc  | HXLTABLP/HXLTABLS/XFXTABL | Description of room class. |
| leave_flag       | HXPTABLD/XFXTABL | Indicates hospital/therapeutic leave. |

#### 10.3 Footer/Summary Fields

| Field             | Source     | Description |
|-------------------|-----------|------------|
| totalActive       | HABADTE   | Count of active inpatients. |
| totalLeave        | HABADTE   | Count of leave accounts. |
| totalBeds         | HABADTE   | Total occupied beds. |
| reportRunDateTime | system    | Timestamp of report generation. |

#### 10.4 Sort Order

Primary sort order:
- Nursing station (ascending)
- Admit date (ascending)
- Admit time (ascending)

Within each station, secondary sort by room number may be applied.

---

### (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
  - Load LDA values (facilityLevel6, userId, default preferences) from HXXLDA.
  - Read MRN roll-up configuration via XFXMRNROL, which calls HXHAPPPRF and HXXAPPPRF.
  - Set mrnRollupMode based on profile tables (HXPAPPPRF/HXPAPPL6).

STEP 2: Preferences / Lookup
  - Use XFXLDSC to resolve facility level descriptions from HXPLVL1–HXPLVL6.
  - Build header strings (facility_name, date/time) and center them via XFXCNTR.
  - Pre-load table mappings for room class and leave flags via XFXTABL over HXPTABLD/HXLTABLD/P/S.

STEP 3: Context
  - Accept request parameters for censusDate, censusTime, nursingStationCode, includeLeave.
  - Validate censusDate using XFXCYMD (enforcing BR-003–BR-008).
  - Set user_id from LDA and propagate to XML header/detail structures (HXFXMLH/HXFXMLD).

STEP 4: Query
  - Build primary query over patient master data (TMPMAST/OMPMAST via HMLMAST5H).
  - Apply inclusion/exclusion rules:
      * fileIndicator <> 0 (BR-017)
      * flag_indicator NOT IN ('VOID', 'VOIDED') (BR-018)
      * ip_op_flag <> 'O' (BR-019)
  - For each candidate record, construct account and facility keys.

STEP 4 SQL (conceptual):
  SELECT ... FROM PatientMaster
   WHERE facility_level6 = @facilityLevel6
     AND admit_date = @censusDate
     AND admit_time <= @censusTime
     AND file_indicator <> 0
     AND flag_indicator NOT IN ('VOID', 'VOIDED')
     AND ip_op_flag <> 'O';

STEP 5: Per-Record Enrichment
  STEP 5a: Nursing Station Enrichment
    - Chain to HXPNSTN/XFFNSTN using facilityLevel6 + nursingStationCode.
    - Retrieve station_name; handle not found by printing code.

  STEP 5b: Room Class & Leave Enrichment
    - Call XFXTABL with roomClassCode to fetch descriptions and leave flags.
    - Apply XFXTABL rules (BR-013–BR-016) to determine whether to continue.

  STEP 5c: Transfer History Enrichment
    - Query HAPTRFR for latest transfer per account.
    - Overlay room_number and related fields from transfer record.

  STEP 5d: MRN Roll-up Adjustments
    - Based on mrnRollupMode, group output either by account or MRN.
    - Adjust headings and totals accordingly.

STEP 6: Assemble Response
  - Write print lines for each enriched record to PRINTER output.
  - Construct XML header (HXFXMLH) and detail (HXFXMLD) records using CXXXMLP/CXXXMLC templates.
  - Persist XML via HXPXMLD/HXPXMLR tables.
  - Compute totals (totalActive, totalLeave, totalBeds) and write summary lines.
  - Return structured JSON response to the API caller along with an XML payload reference.
```

---

### (12) Data Type Conversions

#### 12.1 Date Fields

- AS400 representation: DECIMAL(8,0) `CCYYMMDD`.
- Java representation: `java.time.LocalDate`.
- Conversion rule: `0` → `null`; otherwise parse string.

```java
LocalDate toLocalDate(int ccyymmdd) {
  if (ccyymmdd == 0) return null;
  String s = String.format("%08d", ccyymmdd);
  int year = Integer.parseInt(s.substring(0, 4));
  int month = Integer.parseInt(s.substring(4, 6));
  int day = Integer.parseInt(s.substring(6, 8));
  return LocalDate.of(year, month, day);
}
```

#### 12.2 Time Fields

- AS400 representation: DECIMAL(4,0) or DECIMAL(6,0) `HHMM` or `HHMMSS`.
- Java representation: `java.time.LocalTime`.

```java
LocalTime toLocalTime(int hhmm) {
  if (hhmm == 0) return null;
  int hour = hhmm / 100;
  int minute = hhmm % 100;
  return LocalTime.of(hour, minute);
}
```

#### 12.3 Packed Decimal Keys

- AS400: DECIMAL(6,0) etc.
- Java: `long` or `BigDecimal` depending on range.

```java
long toLong(int packed) {
  return packed; // direct mapping for non-negative keys
}
```

#### 12.4 String Trimming

- AS400: fixed-length, right-padded with spaces.
- Java: `String.trim()` before display or persistence.

```java
String normalize(String s) {
  return s == null ? null : s.trim();
}
```

---

### (13) SQL Server Table Mapping

| AS400 Object | SQL Server Table      | Purpose |
|-------------|-----------------------|---------|
| TMPMAST/OMPMAST | dbo.PatientMaster  | Core patient account and demographic data. |
| HAPTRFR      | dbo.TransferHistory   | Room and bed transfer events per account. |
| HXPLVL1–6    | dbo.CorporateLevels   | Corporate hierarchy (levels 1–6). |
| HXPTABLD     | dbo.ConfigTable       | Generic configuration table for codes and descriptions. |
| HXLTABLD/P/S | dbo.ConfigTableViews  | Alternate keys and projections over configuration data. |
| OXPBNFIT     | dbo.BenefitPlans      | Benefit plan definitions and contact phone numbers. |
| OXPNSTN      | dbo.NursingStation    | Station definitions per facility. |
| HXPXMLD      | dbo.XmlDetail         | XML detail payload per user/run. |
| HXPXMLR      | dbo.XmlHeader         | XML header and id sequences. |

#### Suggested SQL Server Indexes

```sql
-- Primary census query
CREATE INDEX IX_PatientMaster_Census
ON dbo.PatientMaster (MMPLV6, MMADDT, MMADTM, MMPNST);

-- Transfer history lookups
CREATE INDEX IX_TransferHistory_AccountDate
ON dbo.TransferHistory (AFLVL6, AFACCT, AFTRDT, AFTRTM);

-- Nursing station lookups
CREATE INDEX IX_NursingStation_FacilityStation
ON dbo.NursingStation (facility_level6, station_code);

-- Room class table lookups
CREATE INDEX IX_ConfigTable_DataTypeDetail
ON dbo.ConfigTable (data_type_code, detail_code);

-- Benefit plans lookups
CREATE INDEX IX_BenefitPlans_Plan
ON dbo.BenefitPlans (XFBUBN, XFBPLN);
```

---

### (14) Spring Boot API Design

#### 14.1 Recommended REST Endpoint

- **Method:** `GET`
- **Path:** `/api/census/active-accounts`

Parameters:

| Name              | Type    | Required | Validation |
|-------------------|---------|----------|-----------|
| facilityLevel6    | string  | yes      | Must match known HXPLVL6 code. |
| censusDate        | string  | yes      | ISO date; converted to DECIMAL(8,0) with XFXCYMD rules. |
| censusTime        | string  | yes      | HH:mm; must be valid time. |
| nursingStationCode| string  | no       | If present, must exist in NursingStation table. |
| includeLeave      | boolean | no       | Default false. |
|

#### 14.2 Layer Structure

- **Controller:** `CensusController` – handles HTTP requests, validation, and mapping to service.
- **Service:** `CensusService` – implements business rules, orchestrates queries and enrichment.
- **Repository:** `PatientMasterRepository`, `TransferHistoryRepository`, `NursingStationRepository`, `ConfigTableRepository`, `BenefitPlanRepository` – Spring Data repositories encapsulating SQL access.

#### 14.3 Response JSON Shape

```json
{
  "facility": "General Hospital",
  "censusDate": "2024-03-31",
  "censusTime": "06:00",
  "mrnRollupMode": "BY_ACCOUNT",
  "totals": {
    "active": 125,
    "leave": 7,
    "beds": 125
  },
  "records": [
    {
      "accountNumber": "123456789012",
      "mrn": "MRN00000001",
      "patientName": "DOE, JANE",
      "nursingStation": "3EAST",
      "stationName": "3 East Medical",
      "roomNumber": "301A",
      "admitDate": "2024-03-29",
      "admitTime": "14:32",
      "roomClass": "SEMIPRIV",
      "roomClassDescription": "Semi-Private",
      "onLeave": false
    }
  ]
}
```

#### 14.4 Java Entity/DTO Sketch

```java
public record CensusRecord(
    String accountNumber,
    String mrn,
    String patientName,
    String nursingStation,
    String stationName,
    String roomNumber,
    LocalDate admitDate,
    LocalTime admitTime,
    String roomClass,
    String roomClassDescription,
    boolean onLeave
) {}

public record CensusResponse(
    String facility,
    LocalDate censusDate,
    LocalTime censusTime,
    String mrnRollupMode,
    int totalActive,
    int totalLeave,
    int totalBeds,
    List<CensusRecord> records
) {}
```

---

### (15) Performance Considerations

Because HABADTE enriches each record via station lookup, room class table reading, and transfer history, a naive implementation would suffer from N+1 query patterns.

Recommended approach:
- Use **set-based** queries with joins rather than per-record lookups.

Example SQL with LEFT JOINs:

```sql
SELECT
  p.MMACCT,
  p.MMMRNO,
  p.MMNAME,
  p.MMPNST,
  ns.station_name,
  t.room_number,
  p.MMADDT,
  p.MMADTM,
  rc.room_class,
  rc.room_class_description,
  rc.leave_flag
FROM dbo.PatientMaster p
LEFT JOIN dbo.NursingStation ns
  ON ns.facility_level6 = p.MMPLV6
 AND ns.station_code = p.MMPNST
LEFT JOIN dbo.TransferHistory t
  ON t.AFLVL6 = p.MMPLV6
 AND t.AFACCT = p.MMACCT
 AND (t.AFTRDT < @censusDate OR (t.AFTRDT = @censusDate AND t.AFTRTM <= @censusTime))
LEFT JOIN dbo.RoomClassTable rc
  ON rc.detail_code = p.room_class_code
WHERE p.MMPLV6 = @facilityLevel6
  AND p.MMADDT = @censusDate
  AND p.MMADTM <= @censusTime
  AND p.file_indicator <> 0
  AND p.flag_indicator NOT IN ('VOID', 'VOIDED')
  AND p.ip_op_flag <> 'O';
```

This reduces round-trips and uses indexes defined in section 13.

---

### (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When file indicator equals zero, skip the record from census. |
| BR-018  | When flag indicator equals void/voided, skip the record from census. |
| BR-019  | When inpatient/outpatient flag indicates outpatient, skip the record from census. |
| BR-003  | Validate year: when VYY < 1800, exit date conversion. |
| BR-004  | Validate year: when VYY > 2100, exit date conversion. |
| BR-005  | Validate month: when VMM < 01, exit date conversion. |
| BR-006  | Validate month: when VMM > 12, exit date conversion. |
| BR-007  | Validate day: when VDD < 01, exit date conversion. |
| BR-008  | Validate day: when VDD > DYS(VMM), exit date conversion. |
| BR-009–BR-012 | Validate level mapping (LDAMAP) ranges for facility descriptions. |
| BR-013–BR-016 | When *IN79 indicator is on/active, exit table lookup processing. |
| BR-020 | SQL program HXXAPPPRF accesses table HXPAPPPRF for profile configuration. |

---

### (17) Edge Cases to Implement

| Scenario                              | Expected Behavior |
|---------------------------------------|-------------------|
| Null/zero census date                 | Reject request; return validation error; no data processed. |
| Invalid date (month/day/year out of range) | Use XFXCYMD-equivalent validation; return error or skip record depending on context. |
| Null/zero census time                 | Treat as 00:00 or reject based on configuration; document decision. |
| Not-found nursing station lookup      | Print station code; log configuration gap; do not fail request. |
| Not-found room class mapping          | Use generic description; set leave_flag false; log gap. |
| Transfer record not found             | Use room from master record; still count record as active. |
| Delete/void flags on master record    | Apply BR-018; skip from census. |
| Outpatient flag on record             | Apply BR-019; skip from census. |
| Empty result set                      | Return response with totals = 0 and no records; report header still printed. |
| Preferences not configured (missing profile records) | Default mrnRollupMode to BY_ACCOUNT; mark response with warning; still produce census.
