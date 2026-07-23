# Business Processing Rules & Functional Specification

## Active Patients by Date and Time (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

The HABADTE process produces an **Active Patients by Date and Time** census, showing all inpatient accounts that are active at a specified census date/time for a given organisational scope (facility, hospital, unit, etc.). The report is used by nursing administration, bed control, and hospital operations to understand current occupancy, leave status, and distribution by nursing station and room class.

> Core business question: *“Which patients are actively admitted at this facility (or organisational level) at the requested census date and time, and what are their current room, station, and leave status?”*

Key outputs:
- Per-patient detail lines including account, MRN, patient name (in production), admission/discharge dates, room, nursing station, and room class.
- Summary totals by leave status (hospital leave, therapeutic leave) and by other classification dimensions.
- XML-based structured output suitable for downstream reporting and integration.

---

## (2) Inputs (API Request Parameters)

The modern API will replace the HABADTE batch invocation. Parameters are derived from the HABADTE rules and the level hierarchy logic:

| Parameter           | AS400 Field / Concept    | Type        | Description                                                                 |
|---------------------|--------------------------|------------|-----------------------------------------------------------------------------|
| `censusDate`        | Report date (CCYYMMDD)   | `LocalDate`| Date at which census is evaluated. Must be between 1800-01-01 and 2100-12-31 (see BR-003/004). |
| `censusTime`        | Report time (HHMM)       | `LocalTime`| Time-of-day at which census snapshot is taken. Used with transfer history to determine current room. |
| `orgLevel`          | Organisational level 1–6 | `int`      | Level of hierarchy used to filter patients (see BR-020 and XFXLDSC rules). Must be within valid range; invalid codes are rejected (BR-009/012). |
| `orgCode`           | Level code               | `String`   | Code identifying the organisation at the selected level (e.g., facility code at level 6). Patients whose org code at this level does not match are excluded. |
| `includeLeave`      | Derived from XFDMAP      | `boolean`  | Whether to include patients on hospital/therapeutic leave. If false, patients with leave flags (from XFDMAP position 10) are excluded from detail. |
| `userId`            | XML user key (XMDUSR)    | `String`   | User identifier written to XML header/detail records, used for sequencing and auditing. |
| `sequenceSeed`      | XML sequence (XMDSEQ)    | `long`     | Starting sequence number for XML records. Increments per detail row and section. |

Additional implicit input:
- **Current user identity and security context**: In the AS400 system, the job and user profile govern access to PHI-bearing files (HAPTRFR, OMPMAST, HXPDICT). In the modern API, the caller identity must be captured from the security token.

**Spring Boot security mapping note**
- Use OAuth2/OpenID Connect for authentication; map user roles to RBAC permissions controlling access to PHI (e.g., `ROLE_CENSUS_VIEW`, `ROLE_ADMIN`).
- Implement audit logging for all calls, capturing user ID, org scope, census date/time, and result size.
- Enforce field-level security where appropriate, minimizing exposure of identifiers in responses.

---

## (3) Organizational Hierarchy

The HABADTE report uses the corporate level hierarchy stored in the HXPLVL1–HXPLVL6 tables via the XFXLDSC service. Each level has its own numeric key field:

| Level Number | Name (Conceptual)      | Key Size (digits) | AS400 Table | Notes                                      |
|--------------|------------------------|-------------------|-------------|--------------------------------------------|
| 1            | Corporate System       | 3 (example HX1NUM)| HXPLVL1     | Top-level corporate identifier.           |
| 2            | Region / Network       | 3 (HX2NUM)        | HXPLVL2     | Groups facilities into larger regions.    |
| 3            | Division / Cluster     | 3 (HX3NUM)        | HXPLVL3     | Intermediate organisational cluster.      |
| 4            | Hospital Group         | 3 (HX4NUM)        | HXPLVL4     | Collections of hospitals or facilities.   |
| 5            | Hospital / Campus      | 3 (HX5NUM)        | HXPLVL5     | Single hospital or campus.                |
| 6            | Facility / Unit Level  | 3 (HX6NUM)        | HXPLVL6     | Detailed facility/unit. Used heavily by HABADTE. |

**Business Rule Note (Report Header)**
- HABADTE uses XFXLDSC to resolve textual descriptions for the selected org level and org code. These descriptions are centered in report headings using XFXCNTR.
- The report header includes the organisation name and the census date/time, forming a context string such as “Active Patients by Date and Time – Facility XYZ – 2026-07-16 14:44”.

---

## (4) Patient Data Source

HABADTE’s primary entity is the **inpatient account**, keyed by facility-level code and account number. While the compact context does not explicitly list OMPMAST in HABADTE’s declared files, the patient master concept is present in the data dictionary. The **primary data source** for active census evaluation is the **patient master / transfer combination**, with the *transfer history (HAPTRFR)* providing the current room assignment.

### 4.1 Data Access Pattern

Main access:
- **Transfer History (HAPTRFR)** PF with record format HAFTRFR and key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`.
- HABADTE declares HAPTRFR and reads it (dep_edges: `HABADTE -> HAPTRFR READ`).

Likely access pattern (based on BR-023 and key structure):
- Primary key: facility level 6 (`AFLVL6`) and account (`AFACCT`).
- Records are ordered by transfer date (`AFTRDT`) and time (`AFTRTM`) to identify the most recent transfer at or before the census date/time.
- Transfer type (`AFTYPE`) differentiates room moves, leave entries, and discharges.

Canonical SQL Server equivalent:

```sql
SELECT TOP 1
    t.AFACCT      AS accountNumber,
    t.AFLVL6      AS facilityLevel,
    t.AFTRDT      AS transferDate,
    t.AFTRTM      AS transferTime,
    t.AFTYPE      AS transferType,
    t.AFNRMC      AS roomClassCode,
    t.AFROOM      AS roomNumber
FROM dbo.HAPTRFR t
WHERE t.AFLVL6 = @facilityLevel
  AND t.AFACCT = @accountNumber
  AND (t.AFTRDT < @censusDate
       OR (t.AFTRDT = @censusDate AND t.AFTRTM <= @censusTime))
ORDER BY t.AFTRDT DESC, t.AFTRTM DESC;
```

### 4.2 Key Fields

Derived from data_dict_schema for HAPTRFR:

| SQL Column        | AS400 Field | Type        | Description                                      |
|-------------------|-------------|------------|--------------------------------------------------|
| `facilityLevel`   | AFLVL6      | INT        | Facility or unit code (level 6).                 |
| `accountNumber`   | AFACCT      | BIGINT     | Inpatient account identifier (PHI – AccountNumber). |
| `transferDate`    | AFTRDT      | INT / DATE | Transfer date (DECIMAL 8,0 representing CCYYMMDD). |
| `transferTime`    | AFTRTM      | INT / TIME | Transfer time (DECIMAL 6,0 HHMMSS or 4-digit HHMM). |
| `transferType`    | AFTYPE      | CHAR(1)    | Type of transfer (move, leave, discharge).       |

These fields drive BR-023 (room retrieval) and BR-025 (leave flag via room class code).

---

## (5) Inclusion and Exclusion Rules

Inclusion/exclusion is primarily driven by HABADTE’s business rules:

### BR-017 – Exclude Pre-Admitted Patients

**Description**  
A patient with file indicator = 0 is a pre-admission, meaning the account is not yet formally admitted. Such records must be excluded from the active census.

**Pseudocode**
```pseudo
if patient.fileIndicator = 0 then
    excludeFromCensus(patient)
end if
```

**SQL WHERE Clause fragment**
```sql
-- BR-017: exclude pre-admissions
AND patient.fileIndicator <> 0
```

---

### BR-018 – Exclude Voided Accounts

**Description**  
Accounts marked as Voided (flag = 'V') are cancelled and must never appear on the active census.

**Pseudocode**
```pseudo
if patient.voidFlag = 'V' then
    excludeFromCensus(patient)
end if
```

**SQL WHERE Clause fragment**
```sql
-- BR-018: exclude voided accounts
AND patient.voidFlag <> 'V'
```

---

### BR-019 – Exclude Outpatients

**Description**  
Only inpatients appear in this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.

**Pseudocode**
```pseudo
if patient.ioIndicator = 'O' then
    excludeFromCensus(patient)
end if
```

**SQL WHERE Clause fragment**
```sql
-- BR-019: exclude outpatients
AND patient.ioIndicator <> 'O'
```

---

### BR-020 – Organisational Level Filter

**Description**  
Each patient record belongs to one or more organisational levels (1–6). The report accepts an org level and org code. If the patient’s code at that level does not match the requested org code, the record is excluded.

**Pseudocode**
```pseudo
levelCode = getLevelCode(patient, orgLevel)
if levelCode <> requestedOrgCode then
    excludeFromCensus(patient)
end if
```

**SQL WHERE Clause fragment**
```sql
-- BR-020: organisational level filter
AND get_level_code(patient, @orgLevel) = @orgCode
```

Implementation of `get_level_code` is derived from XFXLDSC and level tables HXPLVL1–HXPLVL6.

---

### BR-021 – Admission Date Check

**Description**  
A patient admitted after the census date is not yet active at the census snapshot. If the admission date is later than the census date, the patient is excluded.

**Pseudocode**
```pseudo
if patient.admissionDate > censusDate then
    excludeFromCensus(patient)
end if
```

**SQL WHERE Clause fragment**
```sql
-- BR-021: admission date must be on or before census date
AND patient.admissionDate <= @censusDate
```

---

### BR-022 – Discharge Date Check

**Description**  
A patient discharged on or before the census date is no longer active. If the discharge date is non-zero and is on or before the census date, the patient is excluded. A zero discharge date means the patient is not yet discharged.

**Pseudocode**
```pseudo
if patient.dischargeDate <> 0 and patient.dischargeDate <= censusDate then
    excludeFromCensus(patient)
end if
```

**SQL WHERE Clause fragment**
```sql
-- BR-022: discharge date handling
AND (
    patient.dischargeDate = 0
    OR patient.dischargeDate > @censusDate
)
```

---

### BR-023 – Room Number Retrieval from Transfer History

**Description**  
The current room number is taken from the most recent transfer history record rather than directly from the patient record.

**Pseudocode**
```pseudo
latestTransfer = findLatestTransfer(HAPTRFR, patient.accountNumber, censusDate, censusTime)
patient.roomNumber = latestTransfer.roomNumber
patient.roomClassCode = latestTransfer.roomClassCode
```

**SQL WHERE Clause fragment (subquery)**
```sql
-- BR-023: latest transfer join
OUTER APPLY (
    SELECT TOP 1
        t.AFROOM      AS roomNumber,
        t.AFNRMC      AS roomClassCode,
        t.AFTRDT      AS transferDate,
        t.AFTRTM      AS transferTime
    FROM dbo.HAPTRFR t
    WHERE t.AFLVL6 = patient.facilityLevel
      AND t.AFACCT = patient.accountNumber
      AND (t.AFTRDT < @censusDate
           OR (t.AFTRDT = @censusDate AND t.AFTRTM <= @censusTime))
    ORDER BY t.AFTRDT DESC, t.AFTRTM DESC
) tr
```

---

### BR-024 – Room Class Description Lookup

**Description**  
The room class code (e.g., AFNRMC) is resolved via reference tables (HXPTABLD and its LFs) to a description used in the report.

**Pseudocode**
```pseudo
code = latestTransfer.roomClassCode
mapping = lookupMapping(HXPTABLD, code)
patient.roomClassDescription = mapping.description
```

**SQL WHERE Clause fragment (join)**
```sql
-- BR-024: room class lookup
LEFT JOIN dbo.HXPTABLD rc
  ON rc.XFDTCD = @roomClassDataCode
 AND rc.XFDECD = tr.roomClassCode
```

---

### BR-025 – Hospital / Therapeutic Leave Flag

**Description**  
Using the mapping field XFDMAP from the table lookup, HABADTE determines if the patient is on hospital or therapeutic leave based on the 10th character.

**Pseudocode**
```pseudo
flags = mapping.XFDMAP
if substring(flags, 10, 1) = 'H' then
    patient.leaveStatus = 'HOSPITAL_LEAVE'
elseif substring(flags, 10, 1) = 'T' then
    patient.leaveStatus = 'THERAPEUTIC_LEAVE'
else
    patient.leaveStatus = 'NONE'
end if
```

**SQL fragment**
```sql
-- BR-025: derive leave flag from XFDMAP
CASE SUBSTRING(rc.XFDMAP, 10, 1)
    WHEN 'H' THEN 'HOSPITAL_LEAVE'
    WHEN 'T' THEN 'THERAPEUTIC_LEAVE'
    ELSE 'NONE'
END AS leaveStatus
```

### Summary SQL WHERE Clause

```sql
WHERE 1 = 1
  -- BR-017: exclude pre-admissions
  AND patient.fileIndicator <> 0
  -- BR-018: exclude voided accounts
  AND patient.voidFlag <> 'V'
  -- BR-019: exclude outpatients
  AND patient.ioIndicator <> 'O'
  -- BR-020: organisational filter
  AND get_level_code(patient, @orgLevel) = @orgCode
  -- BR-021: admission must be on/before census
  AND patient.admissionDate <= @censusDate
  -- BR-022: discharge later than census or not set
  AND (
      patient.dischargeDate = 0
      OR patient.dischargeDate > @censusDate
  )
ORDER BY patient.nursingStation, patient.roomNumber, patient.accountNumber;
```

---

## (6) Transfer History Enrichment

The first major enrichment step is the **transfer history lookup** from HAPTRFR.

### BR-ID and Algorithm Steps

Relevant rule: **BR-023** (room retrieval).

Algorithm:
1. For each patient candidate passing filters BR-017–BR-022, determine the facility level (AFLVL6/MMPLV6) and account number (AFACCT/MMACCT).
2. Query HAPTRFR for transfer records matching facility and account.
3. Restrict to transfers with date/time less than or equal to the census date/time.
4. Order transfers descending by date/time and take the top record.
5. Use that record’s room number and room class code as the current room attributes for the census.

### SQL SELECT Equivalent

```sql
SELECT TOP 1
    t.AFROOM      AS roomNumber,
    t.AFNRMC      AS roomClassCode,
    t.AFTRDT      AS transferDate,
    t.AFTRTM      AS transferTime,
    t.AFTYPE      AS transferType
FROM dbo.HAPTRFR t
WHERE t.AFLVL6 = @facilityLevel
  AND t.AFACCT = @accountNumber
  AND (t.AFTRDT < @censusDate
       OR (t.AFTRDT = @censusDate AND t.AFTRTM <= @censusTime))
ORDER BY t.AFTRDT DESC, t.AFTRTM DESC;
```

### Edge Cases

- **No transfer history found**: The patient may be newly admitted without transfer entries; treat room fields as null and optionally include in a “no room assigned” bucket.
- **Transfer after census time**: Exclude transfers after census to prevent leakage of future state; only consider records up to census timestamp.
- **Discharge transfer**: If the latest transfer record indicates discharge (AFTYPE), this should already have been filtered out by BR-022; if not, exclude or mark as inactive.

---

## (7) Nursing Station Enrichment

The second enrichment step is **nursing station lookup** via HXPNSTN / XFFNSTN.

### BR-ID and Algorithm Steps

Indirectly tied to BR-023/024 and organisational presentation.

Algorithm:
1. Take facility level and nursing station code for the patient’s current room.
2. Query HXPNSTN (LF over TXPNSTN) keyed by `XFNLV6` (level 6) and `XFNSST` (station code).
3. Retrieve the station description (e.g., `XFNDES`) for inclusion in report headings and detail lines.

### SQL SELECT Equivalent

```sql
SELECT s.XFNDES AS stationDescription
FROM dbo.OXPNSTN s
WHERE s.XFNLV6 = @facilityLevel
  AND s.XFNSST = @stationCode;
```

### Edge Cases

- **Station not found**: Display a fallback such as “Unknown Station @stationCode” and flag for data quality review.
- **Inactive station**: If the table includes active/inactive flags, filter inactive stations and treat patients as assigned to an obsolete station category.

---

## (8) Room Class / Leave Enrichment

The third enrichment is **room class description and leave status** via HXPTABLD, HXLTABLD, and XFXTABL.

Algorithm:
1. Take the room class code from the latest transfer record (AFNRMC).
2. Call the table lookup service (XFXTABL) using HXPTABLD/HXLTABLD/HXLTABLP/HXLTABLS.
3. Retrieve the descriptive text and mapping flags (XFDMAP).
4. Use XFDMAP’s 10th character to derive leave status per BR-025.

SQL SELECT Equivalent:

```sql
SELECT
    rc.XFDLDS AS roomClassLongDescription,
    rc.XFDSDS AS roomClassShortDescription,
    rc.XFDMAP AS mappingFlags
FROM dbo.HXPTABLD rc
WHERE rc.XFDTCD = @roomClassDataCode
  AND rc.XFDECD = @roomClassCode;
```

Edge Cases:
- **Code not found**: Mark room class as “Unknown” and leaveStatus “NONE”; log for data correction.
- **MappingFlags shorter than 10 characters**: Treat as no leave flag.

---

## (9) Counting Rules

HABADTE’s key_rules emphasize filters rather than explicit counters, but the following counters should be implemented:

| Counter Name          | Incremented When                                              | Description                                   |
|-----------------------|---------------------------------------------------------------|-----------------------------------------------|
| `totalActivePatients` | A patient passes BR-017–BR-022 and is included in detail.     | Total number of active inpatient accounts.    |
| `hospitalLeaveCount`  | Derived leaveStatus = 'HOSPITAL_LEAVE'.                      | Number of patients on hospital leave.         |
| `therapeuticLeaveCount`| Derived leaveStatus = 'THERAPEUTIC_LEAVE'.                  | Number of patients on therapeutic leave.      |
| `noRoomAssignedCount` | Patient eligible but no transfer history or missing room.    | Patients without an assigned room.            |

Relationship and business meaning:
- Counts must reconcile with operational expectations and may be cross-checked against bed management systems.
- Leave counts are used by nursing administration to manage occupancy and staff allocation.

---

## (10) Output Data Structure

HABADTE produces both XML and (legacy) printed output. In the modern system, focus is on structured JSON/XML.

### 10.1 Header Fields

Derived from XML header tables (HXPXMLR/HXFXMLH) and level descriptions.

| Field                | Source           | Description                                   |
|----------------------|------------------|-----------------------------------------------|
| `reportId`           | XMRID / HXFXMLH  | Unique report identifier for this run.        |
| `userId`             | XMRUSR           | User who triggered the census.                |
| `sequenceStart`      | XMRSEQ           | Initial sequence number.                      |
| `censusDate`         | Input parameter  | Date of census snapshot.                      |
| `censusTime`         | Input parameter  | Time of census snapshot.                      |
| `orgLevel`           | Input parameter  | Organisational level used for filter.         |
| `orgCode`            | Input parameter  | Organisational code.                          |
| `orgDescription`     | XFXLDSC result   | Text description of org scope.                |

### 10.2 Detail Row Fields

Using HAPTRFR, patient master, and enrichment:

| Field                   | Source                     | Description                                      |
|-------------------------|----------------------------|--------------------------------------------------|
| `accountNumber`         | HAPTRFR.AFACCT             | Inpatient account identifier (PHI).             |
| `facilityLevel`         | HAPTRFR.AFLVL6             | Facility level code (6).                        |
| `patientName`           | OMPMAST.MMNAME             | Patient name (PHI – in production).             |
| `medicalRecordNumber`   | OMPMAST.MMMRNO/MMMMRN      | MRN (PHI).                                      |
| `admissionDate`         | Patient master field       | Admission date.                                 |
| `dischargeDate`         | Patient master field       | Discharge date (0 if not discharged).          |
| `roomNumber`            | latest transfer.AFROOM     | Current room number from transfer history.     |
| `roomClassCode`         | latest transfer.AFNRMC     | Room class code.                               |
| `roomClassDescription`  | HXPTABLD.XFDLDS/XFDSDS     | Resolved description.                          |
| `nursingStation`        | HXPNSTN.XFNSST             | Station code.                                  |
| `stationDescription`    | HXPNSTN.XFNDES             | Station description.                           |
| `leaveStatus`           | Derived from XFDMAP        | None/Hospital/Therapeutic.                     |

### 10.3 Footer/Summary Fields

| Field                   | Source                    | Description                                   |
|-------------------------|---------------------------|-----------------------------------------------|
| `totalActivePatients`   | Counter                   | Count of active patients.                     |
| `hospitalLeaveCount`    | Counter                   | Count of hospital leave patients.             |
| `therapeuticLeaveCount`| Counter                   | Count of therapeutic leave patients.          |
| `generationTimestamp`   | System clock              | Timestamp of report generation.               |

### 10.4 Sort Order

Primary sort order in the report:
- Nursing station ascending.
- Room number ascending within station.
- Account number ascending within room.

SQL ORDER BY:

```sql
ORDER BY stationDescription, roomNumber, accountNumber;
```

---

## (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
  - Parse input parameters: censusDate, censusTime, orgLevel, orgCode, includeLeave, userId, sequenceSeed.
  - Validate censusDate using XFXCYMD rules (BR-003–BR-008).
  - Prepare XML header record in HXFXMLH/HXPXMLR with userId, censusDate, org context.

STEP 2: Preferences / Lookup
  - Use XFXMRNROL to determine MRN rollup behavior via HXHAPPPRF/HXXAPPPRF.
  - Use XFXLDSC to resolve orgLevel/orgCode description from HXPLVL1–HXPLVL6.
  - Use XFXGETID to obtain XML fragment IDs and static content for headings.

STEP 3: Context
  - Load level 6 facility information from HXPLVL6 for the target org scope.
  - Initialize counters: totalActivePatients, hospitalLeaveCount, therapeuticLeaveCount, noRoomAssignedCount.

STEP 4: Query (Patient Selection)
  - Query patient master (conceptually OMPMAST) for candidate accounts within the org scope.
  - Apply WHERE filters BR-017–BR-022:
    - Exclude pre-admissions (fileIndicator = 0).
    - Exclude voided accounts (voidFlag = 'V').
    - Exclude outpatients (ioIndicator = 'O').
    - Filter by organisational level (get_level_code = orgCode).
    - Exclude patients admitted after censusDate.
    - Exclude patients discharged on/before censusDate.

  Example SQL:

  SELECT p.*
  FROM dbo.OMPMAST p
  WHERE p.fileIndicator <> 0
    AND p.voidFlag <> 'V'
    AND p.ioIndicator <> 'O'
    AND get_level_code(p, @orgLevel) = @orgCode
    AND p.admissionDate <= @censusDate
    AND (p.dischargeDate = 0 OR p.dischargeDate > @censusDate);

STEP 5: Per-Record Enrichment
  For each patient in result set:

  STEP 5a: Transfer History
    - Lookup latest transfer in HAPTRFR (BR-023) using censusDate/censusTime.
    - If found, set roomNumber and roomClassCode; otherwise increment noRoomAssignedCount.

  STEP 5b: Nursing Station
    - Resolve stationDescription via HXPNSTN/XFFNSTN using facilityLevel and station code.

  STEP 5c: Room Class / Leave
    - Resolve room class description via HXPTABLD/HXLTABLD/HXLTABLP/HXLTABLS and XFXTABL.
    - Inspect XFDMAP position 10 to derive leaveStatus (BR-025).
    - If leaveStatus is HOSPITAL_LEAVE or THERAPEUTIC_LEAVE, increment respective counters.

  STEP 5d: Counters
    - If patient passes all filters and enrichments, increment totalActivePatients.

STEP 6: Assemble Response
  - Construct XML detail records in HXPXMLD for each patient, with fields from Step 4–5.
  - Write XML header/footer records via HXFXMLH/HXPXMLR using XFXGETID-defined fragments.
  - Produce JSON response from same data set for API consumers.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields

AS400 uses DECIMAL fields like DECIMAL(8,0) for dates (CCYYMMDD). Conversion:
- `0` → `null` (no date).
- Non-zero: parse into `LocalDate`.

```java
LocalDate toLocalDate(int yyyymmdd) {
  if (yyyymmdd == 0) return null;
  int year = yyyymmdd / 10000;
  int month = (yyyymmdd / 100) % 100;
  int day = yyyymmdd % 100;
  return LocalDate.of(year, month, day);
}
```

### 12.2 Time Fields

Times stored as DECIMAL(4,0) or DECIMAL(6,0) HHMM/HHMMSS. Conversion:

```java
LocalTime toLocalTime(int hhmm) {
  int hour = hhmm / 100;
  int minute = hhmm % 100;
  return LocalTime.of(hour, minute);
}
```

### 12.3 Packed Decimal Keys

Account numbers and sequence fields (e.g., AFACCT, XMDSEQ) become numeric types:
- DECIMAL(6,0) → `long` or `int`.

### 12.4 String Trimming

Fixed-length CHAR fields are right-padded with spaces; trim in Java:

```java
String clean(String s) {
  return s == null ? null : s.trim();
}
```

---

## (13) SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose                                                |
|--------------|------------------|--------------------------------------------------------|
| HAPTRFR      | dbo.HAPTRFR      | Transfer history for room assignment and movement.    |
| OMPMAST      | dbo.OMPMAST      | Inpatient master records (demographics & account).    |
| HXPTABLD     | dbo.HXPTABLD     | Generic code and description table (room classes etc.). |
| HXPLVL1      | dbo.HXPLVL1      | Corporate level 1 hierarchy.                          |
| HXPLVL2      | dbo.HXPLVL2      | Level 2 hierarchy.                                    |
| HXPLVL3      | dbo.HXPLVL3      | Level 3 hierarchy.                                    |
| HXPLVL4      | dbo.HXPLVL4      | Level 4 hierarchy.                                    |
| HXPLVL5      | dbo.HXPLVL5      | Level 5 hierarchy.                                    |
| HXPLVL6      | dbo.HXPLVL6      | Level 6 hierarchy (facility-level).                   |
| HXPNSTN      | dbo.OXPNSTN      | Nursing station reference data.                       |
| HXPBNFIT     | dbo.HXPBNFIT     | Benefit reference table (used tangentially by HABADTE). |
| HXPXMLD      | dbo.HXPXMLD      | XML detail records generated by HABADTE.              |
| HXPXMLR      | dbo.HXPXMLR      | XML report header records.                            |

### Suggested SQL Server Indexes

```sql
-- Primary query: select active patients by org scope and dates
CREATE INDEX IX_OMPMAST_Org_Admit_Discharge
ON dbo.OMPMAST (orgLevel6Code, admissionDate, dischargeDate, ioIndicator, voidFlag, fileIndicator);

-- Transfer history enrichment
CREATE INDEX IX_HAPTRFR_Facility_Account_DateTime
ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM);

-- Room class lookup
CREATE INDEX IX_HXPTABLD_DataCode_RoomClass
ON dbo.HXPTABLD (XFDTCD, XFDECD);

-- Nursing station lookup
CREATE INDEX IX_OXPNSTN_Facility_Station
ON dbo.OXPNSTN (XFNLV6, XFNSST);

-- XML detail retrieval
CREATE INDEX IX_HXPXMLD_User_Seq
ON dbo.HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2);

-- XML report header retrieval
CREATE INDEX IX_HXPXMLR_User_Seq_Id
ON dbo.HXPXMLR (XMRUSR, XMRSEQ, XMRID);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

- Method: `GET`
- Path: `/api/census/active-patients`

Parameters:

| Name          | Type        | Required | Validation                                      |
|---------------|------------|----------|-------------------------------------------------|
| `censusDate`  | `String`   | Yes      | ISO date, convert to `LocalDate`, 1800–2100.    |
| `censusTime`  | `String`   | Yes      | HH:mm, convert to `LocalTime`.                  |
| `orgLevel`    | `Integer`  | Yes      | 1–6; reject invalid levels (BR-009/012).        |
| `orgCode`     | `String`   | Yes      | Non-empty; must exist in level tables.          |
| `includeLeave`| `Boolean`  | No       | Default `true`.                                 |

### 14.2 Layer Structure

- **Controller**: `CensusController` exposing `/api/census/active-patients`.
- **Service**: `ActiveCensusService` implementing main flow (Steps 1–6).
- **Repository**: `PatientRepository`, `TransferRepository`, `RoomClassRepository`, `StationRepository`, `XmlHeaderRepository`, `XmlDetailRepository` using Spring Data JPA.
- **Utility Services**: `DateValidationService` (XFXCYMD), `LevelLookupService` (XFXLDSC), `TableLookupService` (XFXTABL), `XmlIdService` (XFXGETID).

### 14.3 Response JSON Shape

```json
{
  "metadata": {
    "censusDate": "2026-07-16",
    "censusTime": "14:44",
    "orgLevel": 6,
    "orgCode": "HV0001",
    "orgDescription": "Main Hospital Facility",
    "totalActivePatients": 123,
    "hospitalLeaveCount": 5,
    "therapeuticLeaveCount": 3
  },
  "patients": [
    {
      "accountNumber": "1234567",
      "facilityLevel": "HV0001",
      "patientName": "Doe, Jane",
      "medicalRecordNumber": "MRN000001",
      "admissionDate": "2026-07-01",
      "dischargeDate": null,
      "roomNumber": "402A",
      "roomClassCode": "IC",
      "roomClassDescription": "Intensive Care",
      "nursingStation": "ICU",
      "stationDescription": "Intensive Care Unit",
      "leaveStatus": "NONE"
    }
  ]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record ActiveCensusMetadata(
    LocalDate censusDate,
    LocalTime censusTime,
    int orgLevel,
    String orgCode,
    String orgDescription,
    int totalActivePatients,
    int hospitalLeaveCount,
    int therapeuticLeaveCount
) {}

public record ActiveCensusPatient(
    long accountNumber,
    String facilityLevel,
    String patientName,
    String medicalRecordNumber,
    LocalDate admissionDate,
    LocalDate dischargeDate,
    String roomNumber,
    String roomClassCode,
    String roomClassDescription,
    String nursingStation,
    String stationDescription,
    String leaveStatus
) {}

public record ActiveCensusResponse(
    ActiveCensusMetadata metadata,
    List<ActiveCensusPatient> patients
) {}
```

---

## (15) Performance Considerations

### N+1 Risk

Per-record enrichment steps (transfer lookup, station lookup, room class lookup) can cause N+1 query patterns if implemented naively:
- One query for the patient list.
- Plus one query per patient for HAPTRFR.
- Plus one query per patient for HXPNSTN.
- Plus one query per patient for HXPTABLD.

### Recommended Approach

Use set-based queries and joins:

```sql
SELECT
  p.accountNumber,
  p.facilityLevel,
  p.patientName,
  p.medicalRecordNumber,
  p.admissionDate,
  p.dischargeDate,
  tr.roomNumber,
  tr.roomClassCode,
  s.stationCode,
  s.stationDescription,
  rc.XFDLDS AS roomClassLongDescription,
  rc.XFDSDS AS roomClassShortDescription,
  CASE SUBSTRING(rc.XFDMAP, 10, 1)
    WHEN 'H' THEN 'HOSPITAL_LEAVE'
    WHEN 'T' THEN 'THERAPEUTIC_LEAVE'
    ELSE 'NONE'
  END AS leaveStatus
FROM dbo.OMPMAST p
LEFT JOIN dbo.HAPTRFR tr
  ON tr.AFLVL6 = p.facilityLevel
 AND tr.AFACCT = p.accountNumber
 AND (tr.AFTRDT < @censusDate
   OR (tr.AFTRDT = @censusDate AND tr.AFTRTM <= @censusTime))
LEFT JOIN dbo.OXPNSTN s
  ON s.XFNLV6 = p.facilityLevel
 AND s.XFNSST = tr.stationCode
LEFT JOIN dbo.HXPTABLD rc
  ON rc.XFDTCD = @roomClassDataCode
 AND rc.XFDECD = tr.roomClassCode
WHERE ...; -- Filter rules BR-017–BR-022
```

Batch-fetch or JOIN-based logic reduces database round trips and improves performance in large census runs.

---

## (16) Business Rules Reference Summary

Collated from `approved_rules` and HABADTE key_rules:

| Rule ID | Description                                                                                                                     |
|---------|---------------------------------------------------------------------------------------------------------------------------------|
| BR-017  | Exclude Pre-Admitted Patients (fileIndicator = 0).                                                                             |
| BR-018  | Exclude Voided Accounts (voidFlag = 'V').                                                                                       |
| BR-019  | Exclude Outpatients (ioIndicator = 'O').                                                                                        |
| BR-020  | Organisational Level Filter: patient must belong to requested org scope at selected level.                                     |
| BR-021  | Admission Date Check: admissionDate must be on or before censusDate.                                                            |
| BR-022  | Discharge Date Check: dischargeDate is zero or after censusDate; else exclude patient.                                          |
| BR-023  | Room Number Retrieval: use latest transfer history entry at or before censusDate/time.                                          |
| BR-024  | Room Class Description Lookup via HXPTABLD/HXLTABL* and XFXTABL.                                                                |
| BR-025  | Hospital/Therapeutic Leave Flag: use 10th char of XFDMAP to derive leave status.                                               |

---

## (17) Edge Cases to Implement

| Scenario                                        | Expected Behavior                                                                                 |
|-------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Null/zero admission date                        | Treat as invalid; exclude or flag for data quality review (cannot determine active status).       |
| Null/zero discharge date                        | Interpret as “not discharged”; patient remains eligible subject to other filters.                  |
| Transfer history not found                      | Include patient with null room/station fields; increment `noRoomAssignedCount`.                   |
| Transfer after census time only                 | Ignore transfers after census; use latest transfer at or before census time.                      |
| Room class code not found in HXPTABLD           | Set roomClassDescription to “Unknown”; set leaveStatus to “NONE”; log data issue.                |
| XFDMAP shorter than 10 characters               | Do not derive leave status; treat as “NONE”.                                                      |
| Nursing station not found in HXPNSTN           | Display fallback “Unknown Station @code”; include patient but flag record.                       |
| Patient on hospital/therapeutic leave          | Include patient in census (if `includeLeave = true`) and increment respective leave counters.     |
| Empty result set after filters                  | Return empty `patients` list and metadata with counts = 0; still write XML header/footer.         |
| Preference not configured for MRN rollup       | Use default rollup logic; log configuration gap but do not fail the census run.                   |
