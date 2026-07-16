# Business Processing Rules & Functional Specification

## Patient Census by Nursing Station (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

HABADTE produces a patient census report by nursing station and organisational level for a given census date. It filters the patient population to show only active inpatients, enriches each record with room, nursing station and preference-driven identifiers, and writes XML-based output records for downstream systems.

> Core business question: "Which patients are actively admitted to a given organisation and nursing station on the census date, and what are their current identifiers, room, and status?"

Summary outputs include:
- A structured list of active inpatients by nursing station and organisational level.
- Header and detail records written to XML tables (HXFXMLH/HXFXMLD) for interface consumption.
- Running totals of patient counts used for census and occupancy statistics.

---

## (2) Inputs (API Request Parameters)

The modernized API will accept the following request parameters, mapped from HABADTE’s report inputs and filters:

| Parameter | AS400 Field / Concept | Type | Description |
|-----------|------------------------|------|-------------|
| censusDate | Requested census date (used with admission/discharge dates) | LocalDate | Date for which active patients are reported. |
| orgLevel | Organisational level number (1–6) | int | Hierarchy level at which the filter is applied. |
| orgCode | Organisational code at the chosen level | String | Code identifying facility/ward/unit; must match patient org at that level. |
| nursingStation | MMPNST (from HMLMAST5H/TMPMAST) | String | Nursing station code used to constrain the census scope. |
| facilityId | Facility identifier used in preference lookups | String | Key into application preference tables (HXXAPPPRF/HXHAPPPRF). |
| identifierMode | Preference for MRN vs account display | String ("MRN"/"ACCOUNT") | May be derived from application preferences, but can be overridden by caller. |

Additional implicit inputs:
- **Current user identity:** Derived from the security context (e.g., OAuth2 principal). This corresponds to legacy job/user information used when writing to HXFXMLH/HXFXMLD.

Spring Boot security mapping:
- Use **OAuth2/OpenID Connect** for authentication.
- Apply **RBAC** so only authorized clinical staff and administrators can access census data.
- Implement **PHI audit logging** for every request that reads patient identifiers, room, and other PHI-bearing fields.

---

## (3) Organizational Hierarchy

HABADTE uses the HXPLVL1–HXPLVL6 tables (and their corresponding HXFLVL* record formats) via XFXLDSC to resolve organisational level descriptions.

| Level Number | Name (Conceptual) | Key Size / Digits | AS400 Table |
|--------------|-------------------|-------------------|-------------|
| 1 | Health System / Region | HX1NUM | HXPLVL1 / HXFLVL1 |
| 2 | Facility / Hospital | HX2NUM | HXPLVL2 / HXFLVL2 |
| 3 | Campus / Building | HX3NUM | HXPLVL3 / HXFLVL3 |
| 4 | Service Line / Department | HX4NUM | HXPLVL4 / HXFLVL4 |
| 5 | Ward / Unit Group | HX5NUM | HXPLVL5 / HXFLVL5 |
| 6 | Nursing Unit / Bed Group | HX6NUM | HXPLVL6 / HXFLVL6 |

Business rule note:
- Report headers include the chosen organisational level description and code. For example, if orgLevel=6, the header will show the nursing unit name and code resolved via XFXLDSC from HXPLVL6/HXFLVL6.

---

## (4) Patient Data Source

### 4.1 Data Access Pattern

Primary entity: **Patient admission/master records**

Primary physical file: **TMPMAST**, with logical view **HMLMAST5H** over TMPMAST keyed by:
- MMPNST – nursing station
- MMADDT – admission date
- MMADTM – admission time

HABADTE declares and reads **HMLMAST5H** to iterate patients admitted to a given nursing station, then applies additional filters (pre-admission, voided, outpatient, census date, organisational level).

Data access pattern:
- **Primary read:** Sequential scan over HMLMAST5H, constrained by MMPNST (nursingStation) and possibly date range conditions around censusDate.
- **Sort order:** Records are naturally ordered by nursing station, then admission date/time. The report may use this order directly or apply secondary sorting for output.

Equivalent SQL Server SELECT:

```sql
SELECT
    p.MMPNST      AS nursing_station,
    p.MMADDT      AS admission_date,
    p.MMADTM      AS admission_time,
    p.MMACCT      AS account_number,
    p.MMMRNO      AS mrn,
    p.MMNAME      AS patient_name,
    p.MMPSSN      AS ssn,
    p.MMCENS      AS census_indicator -- inferred field name
FROM PatientMaster p
WHERE p.MMPNST = :nursingStation
ORDER BY p.MMPNST, p.MMADDT, p.MMADTM;
```

### 4.2 Key Fields

Key fields for the primary entity (modelled on TMPMAST/HMLMAST5H):

| SQL Column | AS400 Field | Type | Description |
|-----------|-------------|------|-------------|
| nursing_station | MMPNST | VARCHAR | Nursing station code; primary partition for census. |
| admission_date | MMADDT | DATE (from DECIMAL 8,0) | Date patient was admitted. |
| admission_time | MMADTM | TIME (from DECIMAL 4,0 HHMM) | Time of admission. |
| org_level6 | MMPLV6 | VARCHAR | Organisational level 6 code (ward/unit). |
| account_number | MMACCT | VARCHAR | Patient account number (PHI). |
| mrn | MMMRNO / MMMMRN | VARCHAR | Medical Record Number (PHI). |

---

## (5) Inclusion and Exclusion Rules

This section captures HABADTE’s primary filter rules, using BR-017–BR-022 and the organisational filter BR-020.

### BR-017 – Exclude Pre-Admitted Patients

Patients with **file indicator = 0** are considered pre-admission; they are not yet formally admitted and must be excluded from the active census.

**Description:**
- Pre-admission records represent tentative or scheduled admissions.
- The census should show only active, admitted patients.

**Pseudocode:**

```pseudo
IF fileIndicator = 0 THEN
    EXCLUDE record FROM census
ENDIF
```

**SQL WHERE fragment:**

```sql
AND file_indicator <> 0
```

### BR-018 – Exclude Voided Accounts

Accounts marked as **Voided (flag = 'V')** are cancelled and must never appear on the active census.

**Pseudocode:**

```pseudo
IF account_status_flag = 'V' THEN
    EXCLUDE record
ENDIF
```

**SQL WHERE fragment:**

```sql
AND account_status_flag <> 'V'
```

### BR-019 – Exclude Outpatients

Only inpatients appear on the census. Accounts with **I/O indicator = 'O' (Outpatient)** are excluded.

**Pseudocode:**

```pseudo
IF io_indicator = 'O' THEN
    EXCLUDE record
ENDIF
```

**SQL WHERE fragment:**

```sql
AND io_indicator <> 'O'
```

### BR-020 – Organisational Level Filter

Each patient record must belong to the requested organisation. The report accepts an org level (1–6) and org code.

**Description:**
- For the chosen level (e.g., 6=unit), HABADTE compares the patient’s level code to the requested orgCode.
- If it does not match, the record is excluded.

**Pseudocode:**

```pseudo
SELECT patient_org_code
FROM levels_table
WHERE level = :orgLevel AND account = :account_number;

IF patient_org_code <> :orgCode THEN
    EXCLUDE record
ENDIF
```

**SQL WHERE fragment:**

```sql
AND patient_org_code_at_level = :orgCode
```

### BR-021 – Census Date – Admission Date Check

A patient admitted **after** the census date is not yet active on that date.

**Pseudocode:**

```pseudo
IF admission_date > :censusDate THEN
    EXCLUDE record
ENDIF
```

**SQL WHERE fragment:**

```sql
AND admission_date <= :censusDate
```

### BR-022 – Census Date – Discharge Date Check

A patient discharged **on or before** the census date is no longer active.
- Discharge date = 0 means "not yet discharged" and is always considered active (subject to other filters).

**Pseudocode:**

```pseudo
IF discharge_date <> 0 AND discharge_date <= :censusDate THEN
    EXCLUDE record
ENDIF
```

**SQL WHERE fragment:**

```sql
AND (discharge_date = 0 OR discharge_date > :censusDate)
```

### Summary SQL WHERE Clause

```sql
WHERE p.MMPNST = :nursingStation
  AND p.file_indicator <> 0                 -- BR-017: pre-admission excluded
  AND p.account_status_flag <> 'V'         -- BR-018: voided accounts excluded
  AND p.io_indicator <> 'O'                -- BR-019: outpatients excluded
  AND patient_org_code_at_level = :orgCode -- BR-020: organisational filter
  AND p.admission_date <= :censusDate      -- BR-021: census vs admission
  AND (p.discharge_date = 0
       OR p.discharge_date > :censusDate)  -- BR-022: census vs discharge
ORDER BY p.MMPNST, p.MMADDT, p.MMADTM;
```

---

## (6) Room Number Retrieval from Transfer History (First Enrichment Step)

This enrichment corresponds to BR-023 and data lineage showing HABADTE declaring **HAPTRFR**.

**Business concept:** The patient’s current room number is derived from transfer history, not directly from the patient master.

**Algorithm (BR-023, inferred):**
1. For each patient in the primary result set, query **HAPTRFR** (transfer history) by:
   - AFLVL6 = patient’s organisational level 6.
   - AFACCT = patient account number.
2. Filter transfer records to those **on or before the census date**.
3. Sort by **AFTRDT (transfer date)** and **AFTRTM (transfer time)** ascending.
4. Take the **last** qualifying record as the current location.
5. Extract the room number (e.g., AFNRMC or related room field) and nursing station from the transfer record.
6. Attach these to the patient census row.

**SQL SELECT equivalent:**

```sql
SELECT TOP 1
    t.AFNRMC      AS current_room_class,
    t.AFROOM      AS current_room_number,
    t.AFLVL6      AS level6,
    t.AFACCT      AS account_number,
    t.AFTRDT      AS transfer_date,
    t.AFTRTM      AS transfer_time
FROM TransferHistory t
WHERE t.AFLVL6 = :patientLevel6
  AND t.AFACCT = :accountNumber
  AND t.AFTRDT <= :censusDate
ORDER BY t.AFTRDT DESC, t.AFTRTM DESC;
```

**Edge cases:**
- **No transfer history found:** Leave room fields blank or mark as "Unknown"; patient is still counted if other filters pass.
- **Multiple transfers on same date/time:** Choose the record with highest sequence or latest timestamp if present.
- **Delete-flag logic:** If HAPTRFR contains logical delete flags, ignore deleted records in the enrichment query.

---

## (7) Room Class Description Lookup (Second Enrichment Step)

This enrichment corresponds to BR-024 and use of HXPTABLD/HXLTABLD/HXLTABLP/HXLTABLS via XFXTABL.

**Business concept:** The room class code (e.g., AFNRMC) from transfer history is resolved to a descriptive class and mapping using dictionary tables.

**Algorithm (BR-024):**
1. Take the room class code from HAPTRFR (AFNRMC or similar).
2. Using XFXTABL, query **HXPTABLD** via logical file **HXLTABLD** with keys:
   - XFDTCD = data type code representing "room class".
   - XFDMAP = AFNRMC (room class code).
3. Retrieve status/logical description fields from related logical files:
   - **HXLTABLP** for logical description (XFDLDS).
   - **HXLTABLS** for status description (XFDSDS).
4. Attach descriptive fields to the census row (e.g., "Single Room", "ICU", "Therapeutic Leave").

**SQL SELECT equivalent:**

```sql
SELECT
    d.XFDMAP    AS room_class_code,
    d.XFDESC    AS room_class_desc,
    d.XFSTATUS  AS room_status
FROM DictionaryRoomClass d
WHERE d.XFDTCD = :roomClassDataType
  AND d.XFDMAP = :roomClassCode;
```

**Edge cases:**
- **No dictionary entry found:** Display the raw code and mark description as "Unmapped"; log a warning for configuration gaps.
- **Inactive/obsolete status:** If status indicates inactive, consider whether patient should be flagged or excluded depending on business policy.

---

## (8) Nursing Station Name Resolution & Leave Flag (Third Enrichment Step)

This section combines BR-025 and BR-026 using HXPNSTN/HXPBNFIT.

**Business concepts:**
- Nursing station names are looked up from station tables.
- Hospital/therapeutic leave flags are derived via mapping fields (e.g., XFDMAP) from dictionary/benefit tables.

**Algorithm (BR-026 – Nursing Station Name):**
1. From transfer history or patient master, obtain nursing station code (e.g., MMPNST or equivalent).
2. Query **HXPNSTN** (logical nursing station file over TXPNSTN) with keys:
   - XFNLV6 = patient’s level 6.
   - XFNSST = nursing station code.
3. Retrieve station name and type and attach to census row.

**Algorithm (BR-025 – Hospital/Therapeutic Leave Flag):**
1. When resolving room class via dictionary, also inspect mapping field (e.g., XFDMAP) indicating leave status.
2. Interpret mapping into a boolean or enumerated flag:
   - H = Hospital stay.
   - T = Therapeutic leave.
3. Attach this flag to the census row for downstream reporting.

**SQL SELECT example:**

```sql
SELECT
    s.XFNSST    AS station_code,
    s.XFNDESC   AS station_name,
    s.XFNTYPE   AS station_type
FROM NursingStation s
WHERE s.XFNLV6 = :patientLevel6
  AND s.XFNSST = :nursingStation;
```

Edge cases:
- Station not found: display station code only and mark name as "Unknown".
- Leave flag implies patient is temporarily absent: patient may still be counted depending on census policy; logic should be aligned with clinical requirements.

---

## (9) Counting Rules

BR-029 identifies three running totals maintained as each qualifying patient is processed.

| Counter Name | Incremented When | Description |
|--------------|------------------|-------------|
| totalPatients | After all inclusion/exclusion rules pass for a record | Total number of active patients on the census date. |
| inpatientCount | When io_indicator <> 'O' and patient qualifies | Number of inpatients (should equal totalPatients for this report, but separated for clarity). |
| leaveCount | When hospital/therapeutic leave flag indicates patient is on leave | Number of patients currently on leave (therapeutic or hospital). |

Relationship and business meaning:
- `totalPatients` forms the basic census count used by administration.
- `inpatientCount` can be used for sanity checks or future expansion to include other modalities.
- `leaveCount` supports occupancy and staffing decisions, indicating beds temporarily vacated.

---

## (10) Output Data Structure

HABADTE writes XML header and detail records to **HXFXMLH/HXFXMLD** via DDS files HXPXMLR/HXPXMLD.

### 10.1 Header Fields

Header (HXFXMLH) conceptually contains:

| Field | Description |
|-------|-------------|
| batch_id | Unique identifier for census run (from HXFXMLR/XMRID). |
| census_date | Date of census. |
| org_level | Selected organisational level. |
| org_code | Selected org code. |
| nursing_station | Nursing station filter. |
| requested_by | User or application requesting the report. |

### 10.2 Detail Row Fields

Detail (HXFXMLD) conceptually contains:

| Field | Description |
|-------|-------------|
| batch_id | Links detail to header record. |
| sequence | Sequence within the batch (XMDSEQ/XMDSQ2). |
| account_number | Patient account number (from TMPMAST/OMPMAST/HAPTRFR). |
| mrn | Patient MRN. |
| patient_name | Patient name. |
| admission_date | Admission date. |
| discharge_date | Discharge date (if any). |
| nursing_station | Nursing station code. |
| station_name | Nursing station description (from HXPNSTN). |
| room_number | Current room number (from HAPTRFR/HXPDICT). |
| room_class | Room class description. |
| leave_flag | Hospital/therapeutic leave indicator. |

### 10.3 Footer/Summary Fields

Footer or summary section includes:

| Field | Description |
|-------|-------------|
| total_patients | Total census count. |
| inpatient_count | Inpatient count. |
| leave_count | Leave count. |

### 10.4 Sort Order

Primary sort order:
- Nursing station (ascending).
- Room number (ascending).
- Patient name or MRN (ascending).

---

## (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1 – Init
    - Read request: censusDate, orgLevel, orgCode, nursingStation, facilityId.
    - Resolve application preferences via XFXMRNROL/HXXAPPPRF/HXHAPPPRF:
        * Determine identifierMode (MRN vs ACCOUNT).
        * Load other display preferences.
    - Initialize counters: totalPatients = 0; inpatientCount = 0; leaveCount = 0.
    - Obtain batch_id using XFXGETID from HXFXMLR.

STEP 2 – Preferences / Lookup
    - Use XFXLDSC to resolve orgLevel/orgCode to descriptive name.
    - Prepare dictionary lookups via XFXTABL for room class, status codes, leave flags.

STEP 3 – Context
    - Build report header structure:
        * batch_id, censusDate, orgLevel, orgCode, nursingStation, user.
    - Write header to HXFXMLH.

STEP 4 – Query
    - Select candidate patients from TMPMAST via HMLMAST5H:
        SQL:
        SELECT *
        FROM PatientMaster p
        WHERE p.MMPNST = :nursingStation
          AND p.admission_date <= :censusDate;

STEP 5 – Per-Record Enrichment
    FOR each patient record p IN result:
        5a – Apply Inclusion/Exclusion Filters (BR-017–BR-022, BR-020)
            - If file_indicator = 0 (pre-admission) => skip.
            - If account_status_flag = 'V' (voided) => skip.
            - If io_indicator = 'O' (outpatient) => skip.
            - If org code at orgLevel <> orgCode => skip.
            - If admission_date > censusDate => skip.
            - If discharge_date <> 0 AND discharge_date <= censusDate => skip.

        5b – Enrich from Transfer History (BR-023)
            - Query HAPTRFR by AFLVL6, AFACCT up to censusDate.
            - Determine current room_number and room_class_code.

        5c – Enrich Room Class Description (BR-024)
            - Use XFXTABL with HXLTABLD/HXLTABLP/HXLTABLS to resolve room_class_code
              into room_class_desc and status.

        5d – Enrich Station and Leave Flag (BR-025, BR-026)
            - Query HXPNSTN to resolve nursing_station name.
            - Derive leave_flag from dictionary/benefit mapping fields.

        5e – Counting Rules (BR-029)
            - totalPatients++.
            - inpatientCount++ (since outpatients already excluded).
            - If leave_flag indicates leave => leaveCount++.

        5f – Assemble Detail Row
            - Build XML detail record from enriched fields.
            - Write detail to HXFXMLD.

STEP 6 – Assemble Response
    - Write footer/summary fields (totalPatients, inpatientCount, leaveCount).
    - Commit batch to HXFXMLH/HXFXMLD.
    - Return response to API caller with JSON representation of census data.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields
- AS400 dates stored as **DECIMAL(8,0)** in YYYYMMDD format.
- Conversion to Java:

```java
int rawDate = record.getInt("MMADDT");
LocalDate admissionDate = (rawDate == 0)
    ? null
    : LocalDate.of(rawDate / 10000, (rawDate / 100) % 100, rawDate % 100);
```

- Rule: value 0 = **null** (no date).

### 12.2 Time Fields
- AS400 times stored as **DECIMAL(4,0)** in HHMM format.

```java
int rawTime = record.getInt("MMADTM");
LocalTime admissionTime = LocalTime.of(rawTime / 100, rawTime % 100);
```

### 12.3 Packed Decimal Keys
- Keys like MMACCT, AFLVL6 may be **DECIMAL(6,0)** and should be converted to `long` in Java or numeric columns in SQL Server:

```java
long accountNumber = record.getLong("MMACCT");
```

### 12.4 String Trimming
- Fixed-length, right-padded strings (e.g., patient name, nursing station) must be trimmed:

```java
String patientName = record.getString("MMNAME").trim();
```

---

## (13) SQL Server Table Mapping

Core AS400 objects mapped to SQL Server tables:

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| TMPMAST / HMLMAST5H | PatientMaster | Primary patient admissions master and view. |
| HAPTRFR | TransferHistory | Patient transfer events and current room resolution. |
| HXPTABLD / HXLTABLD/HXLTABLP/HXLTABLS | DictionaryRoomClass | Room class and status mapping. |
| HXPLVL1–HXPLVL6 | OrgHierarchyLevel1–6 | Organisational hierarchy tables for level codes/descriptions. |
| TXPNSTN / HXPNSTN | NursingStation | Nursing station reference and descriptors. |
| HXPXMLR / HXFXMLR | XmlHeader | XML batch/header metadata. |
| HXPXMLD / HXFXMLD | XmlDetail | XML census detail rows. |

### Suggested SQL Server Indexes

```sql
CREATE INDEX IX_PatientMaster_Census
    ON PatientMaster (nursing_station, admission_date, admission_time);

CREATE INDEX IX_TransferHistory_PatientDate
    ON TransferHistory (level6, account_number, transfer_date, transfer_time);

CREATE INDEX IX_DictionaryRoomClass_Code
    ON DictionaryRoomClass (room_class_code);

CREATE INDEX IX_NursingStation_LevelStation
    ON NursingStation (level6, station_code);

CREATE INDEX IX_XmlHeader_Batch
    ON XmlHeader (batch_id);

CREATE INDEX IX_XmlDetail_BatchSeq
    ON XmlDetail (batch_id, sequence);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

**Method:** `GET`  
**Path:** `/api/v1/census`

Parameters:

| Name | Type | Required | Validation |
|------|------|----------|-----------|
| censusDate | String (ISO date) | Yes | Must be a valid date; not in the future beyond business rules. |
| orgLevel | int | Yes | 1–6 only. |
| orgCode | String | Yes | Non-empty; must match a known organisational code. |
| nursingStation | String | Yes | Non-empty; length within allowed range. |
| facilityId | String | No | If omitted, derived from user’s default facility. |
| identifierMode | String | No | Must be "MRN" or "ACCOUNT"; if omitted, derived from preferences. |

### 14.2 Layer Structure

- **Controller:** `CensusController` – validates request, invokes service.
- **Service:** `CensusService` – implements Steps 1–6, orchestrating queries and enrichment.
- **Repository:**
  - `PatientRepository` – wraps PatientMaster/HMLMAST5H.
  - `TransferHistoryRepository` – wraps TransferHistory.
  - `DictionaryRepository` – wraps DictionaryRoomClass and OrgHierarchy.
  - `NursingStationRepository` – wraps NursingStation.
  - `XmlBatchRepository` – wraps XmlHeader/XmlDetail.

### 14.3 Response JSON Shape

```json
{
  "batchId": "CNS202607161154",
  "censusDate": "2026-07-16",
  "orgLevel": 6,
  "orgCode": "WARD-01",
  "nursingStation": "NS01",
  "totals": {
    "totalPatients": 42,
    "inpatientCount": 42,
    "leaveCount": 3
  },
  "patients": [
    {
      "accountNumber": "00012345",
      "mrn": "MRN000987",
      "patientName": "DOE, JANE",
      "admissionDate": "2026-07-10",
      "dischargeDate": null,
      "nursingStation": "NS01",
      "stationName": "Nursing Station 01",
      "roomNumber": "201A",
      "roomClass": "General Ward",
      "leaveFlag": "H"
    }
  ]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record CensusRequest(
    LocalDate censusDate,
    int orgLevel,
    String orgCode,
    String nursingStation,
    String facilityId,
    String identifierMode
) {}

public record PatientCensusRow(
    String accountNumber,
    String mrn,
    String patientName,
    LocalDate admissionDate,
    LocalDate dischargeDate,
    String nursingStation,
    String stationName,
    String roomNumber,
    String roomClass,
    String leaveFlag
) {}

public record CensusResponse(
    String batchId,
    LocalDate censusDate,
    int orgLevel,
    String orgCode,
    String nursingStation,
    int totalPatients,
    int inpatientCount,
    int leaveCount,
    List<PatientCensusRow> patients
) {}
```

---

## (15) Performance Considerations

Per-record enrichment (transfer history lookups, room class resolution, nursing station resolution, leave flag mapping) can cause **N+1 query** issues.

To mitigate:
- Use batch queries or joins instead of per-record calls.

Example SQL JOIN approach:

```sql
SELECT
    p.*, t.*, s.*, d.*
FROM PatientMaster p
LEFT JOIN TransferHistory t
    ON t.level6 = p.org_level6
   AND t.account_number = p.account_number
   AND t.transfer_date <= :censusDate
LEFT JOIN NursingStation s
    ON s.level6 = p.org_level6
   AND s.station_code = p.nursing_station
LEFT JOIN DictionaryRoomClass d
    ON d.room_class_code = t.room_class_code
WHERE -- apply filters BR-017–BR-022, BR-020 here
ORDER BY p.nursing_station, t.room_number, p.patient_name;
```

Additional recommendations:
- Cache dictionary and organisational hierarchy tables in memory.
- Use proper indexes as defined in Section 13.

---

## (16) Business Rules Reference Summary

Business rules for HABADTE and closely related utilities:

| Rule ID | Description |
|---------|-------------|
| BR-017 | Exclude Pre-Admitted Patients: file indicator = 0 records are excluded from active census. |
| BR-018 | Exclude Voided Accounts: accounts with flag = 'V' are cancelled and excluded. |
| BR-019 | Exclude Outpatients: accounts with I/O indicator = 'O' are excluded. |
| BR-020 | Organisational Level Filter: patient org code at requested level must match orgCode. |
| BR-021 | Census Date – Admission Date Check: patients admitted after censusDate are excluded. |
| BR-022 | Census Date – Discharge Date Check: patients discharged on or before censusDate are excluded; zero date = not discharged. |
| BR-023 | Room Number Retrieval from Transfer History: current room derived from latest transfer up to censusDate. |
| BR-024 | Room Class Description Lookup: room class code resolved via dictionary tables. |
| BR-025 | Hospital / Therapeutic Leave Flag: leave flag derived from room class mapping. |
| BR-026 | Nursing Station Name Resolution: station name looked up from nursing station table. |
| BR-027 | Patient Identifier Display – MRN vs Account Number (preference-driven). |
| BR-028 | Application Preference Lookup with Facility Override. |
| BR-029 | Patient Counting – three running totals maintained during processing. |

---

## (17) Edge Cases to Implement

| Scenario | Expected Behavior |
|----------|-------------------|
| Null/zero admission date | Treat 0 as "no date"; exclude record or route to error handling depending on policy. |
| Null/zero discharge date | Interpret as "not discharged"; patient considered active if other filters pass. |
| Transfer history not found | Keep patient in census but leave room-related fields blank or "Unknown"; log for follow-up. |
| Dictionary room class not found | Use raw code; mark description as "Unmapped"; do not block census. |
| Nursing station not found | Display station code only; mark name as "Unknown". |
| Voided accounts | Never include in output; ensure downstream XML does not contain these accounts. |
| Pre-admission records | Always excluded; verify no such records leak into XML detail. |
| Outpatient records | Always excluded for this census; if other reports handle outpatients, they must use different logic. |
| Empty result set | Return header with zero totals and an empty patients array; do not treat as error. |
| Preference not configured | Fall back to default identifier mode (e.g., MRN) and default facility; log configuration gap. |
