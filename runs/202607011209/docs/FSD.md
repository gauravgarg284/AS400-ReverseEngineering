# Business Processing Rules and Functional Specification

## HABADTE Patient Management Engine (HABADTE) /
### For Java / Spring Boot / SQL Server Migration

---

## 1. Business Purpose

The HABADTE patient management engine processes inpatient encounter records and related benefit and account data to prepare them for downstream billing and reporting.
It evaluates control indicators, void flags, and inpatient/outpatient status to determine which records are eligible for further processing.
Execution is typically triggered as part of nightly or batch workflows that consolidate transactional data into patient-level structures.

---

## 2. Inputs / API Request Parameters

For the migrated service, HABADTE behavior will be exposed as a Spring Boot API that receives a combination of patient, account, and encounter identifiers.
Key inputs are derived from the AS400 physical and logical files.

### Request Parameters

| Parameter Name              | AS400 Field / Source     | Type        | Description |
|----------------------------|--------------------------|------------|-------------|
| patientMrn                 | OMPMAST.MMMRNO           | STRING(MRN) | Master medical record number used to locate the patient master row. |
| accountNumber              | HAPTRFR.AFACCT           | STRING      | Financial account identifier linked to transfer records. |
| encounterAccount           | OMPMAST.MMACCT           | STRING      | Encounter-level account number, often shared across stay segments. |
| benefitPlanCode            | OXPBNFIT.XFBPLN          | STRING      | Benefit plan identifier from benefit table. |
| benefitBundleNumber        | OXPBNFIT.XFBUBN          | STRING      | Benefit bundle number grouping plan options. |
| level6Location             | OXPBNFIT.XFBTEL          | STRING      | Telephone/contact or location attribute for benefits. |
| inpatientLevel6Key         | HXPLVL6.HX6NUM           | STRING      | Level-6 hierarchy key representing highest-level grouping of flows. |
| inpatientStatusCode        | HXPNSTN.XFNSST           | STRING      | Inpatient status code used to classify hospital stay state. |
| controlIndicator           | Request-derived flag      | STRING      | Control flag mapped from HABADTE `-FILE INDICATOR` semantics. |
| voidFlag                   | Request-derived flag      | STRING      | Void indicator mapped from HABADTE `-FLAG INDICATOR`. |
| inpatientOutpatientFlag    | Request-derived flag      | STRING      | Encounter type mapped from HABADTE `-INPATIENT/OUTPATIENT FLAG`. |

The API must enforce Spring Security (e.g., OAuth2/JWT) and link the caller’s identity to permitted MRNs and account ranges.
Authorization checks should execute before any data access, especially for PHI-bearing sources such as OMPMAST, HAPTRFR, HXPDICT, and OAPIRNK.

---

## 3. Organizational Hierarchy

The source system primarily encodes technical hierarchies (levels 1–6 in HXPLVL1–HXPLVL6) rather than organizational departments.
These level files represent nested structures such as facility → service line → product, but concrete business department names are not visible in the summarized metadata.

For the purposes of migration, organizational hierarchy can be treated as **not explicitly encoded** and derived externally from master data.

| Level | Name / Concept          | Key Size | Backing Table |
|-------|-------------------------|---------|---------------|
| 1     | Hierarchy Level 1      | HX1NUM  | HXPLVL1       |
| 2     | Hierarchy Level 2      | HX2NUM  | HXPLVL2       |
| 3     | Hierarchy Level 3      | HX3NUM  | HXPLVL3       |
| 4     | Hierarchy Level 4      | HX4NUM  | HXPLVL4       |
| 5     | Hierarchy Level 5      | HX5NUM  | HXPLVL5       |
| 6     | Hierarchy Level 6      | HX6NUM  | HXPLVL6       |

---

## 4. Primary Data Source

The HABADTE engine is centered on inpatient patient-management data.
Two physical files are especially important:

- **OMPMAST (HMFMAST format)** – patient master / encounter master, with PHI (MMMRNO, MMNAME, MMACCT, MMPSSN, MMMMRN).
- **HAPTRFR (HAFTRFR format)** – transfer records across levels, keyed by account and transfer date/time.

### Primary SQL Server Table

For migration, OMPMAST should become a primary SQL Server table:

- Table name: `PatientEncounterMaster`
- Access pattern: keyed lookup by MRN and account, with secondary indexes on encounter dates.

#### Key Fields

| Column Name   | Source Field | Type      | Notes |
|---------------|-------------|-----------|-------|
| mrn           | MMMRNO      | VARCHAR   | Primary patient MRN. |
| accountNumber | MMACCT      | VARCHAR   | Financial account number. |
| patientName   | MMNAME      | VARCHAR   | Patient name, used for display only (never as a join key). |
| ssn           | MMPSSN      | VARCHAR   | Social security number (PHI). |
| encounterMrn  | MMMMRN      | VARCHAR   | Alternate MRN variant, if present. |

#### Example SQL Server SELECT

```sql
SELECT
    mrn,
    accountNumber,
    patientName,
    ssn,
    encounterMrn
FROM PatientEncounterMaster
WHERE mrn = @mrn
  AND accountNumber = @accountNumber
ORDER BY encounterMrn ASC;
```

For HABADTE, transfer data in HAPTRFR can be modeled as `PatientTransfer` with composite key:

| Column       | Source Field | Type      |
|-------------|-------------|-----------|
| level6Key   | AFLVL6      | VARCHAR   |
| account     | AFACCT      | VARCHAR   |
| transferDate| AFTRDT      | DATE      |
| transferTime| AFTRTM      | TIME      |
| transferType| AFTYPE      | VARCHAR   |

---

## 5. Inclusion and Exclusion Rules

Rules are derived from the approved business rule catalog.
Each rule is represented with its BR-ID, description, pseudocode, and SQL translation.

### 5.1 File Indicator-Based Exclusion (BR-017)

- **BR-ID:** BR-017
- **Source Program:** HABADTE
- **Description:** When `-FILE INDICATOR` equals zero, branch to `SKIP`.

**Pseudocode:**

```pseudo
IF fileIndicator = 0
    SKIP current record
END IF
```

**SQL WHERE fragment:**

```sql
AND fileIndicator <> 0
```

### 5.2 Void Flag Exclusion (BR-018)

- **BR-ID:** BR-018
- **Source Program:** HABADTE
- **Description:** When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.

**Pseudocode:**

```pseudo
IF flagIndicator IN ('VOID', 'VOIDED')
    SKIP current record
END IF
```

**SQL WHERE fragment:**

```sql
AND flagIndicator NOT IN ('VOID', 'VOIDED')
```

### 5.3 Outpatient Exclusion (BR-019)

- **BR-ID:** BR-019
- **Source Program:** HABADTE
- **Description:** When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.

**Pseudocode:**

```pseudo
IF inpatientOutpatientFlag = 'OUTPATIENT'
    SKIP current record
END IF
```

**SQL WHERE fragment:**

```sql
AND inpatientOutpatientFlag <> 'OUTPATIENT'
```

### 5.4 Counter-Based Exit Logic (BR-001, BR-002)

- **BR-001:** When `X` equals 0, branch to `EXIT`.
- **BR-002:** When `X` equals 40, branch to `EXIT`.

These rules in XFXCNTR gate iteration based on a numeric counter and are used by HABADTE during batch loops.

**Pseudocode:**

```pseudo
IF counterX = 0 OR counterX = 40
    EXIT loop
END IF
```

**SQL WHERE fragment (for limiting iteration-bound result sets):**

```sql
AND counterX NOT IN (0, 40)
```

### 5.5 Date Validity Exits (BR-003 – BR-008)

XFXCYMD validates date fields before HABADTE consumes them.

- BR-003/004: Year range [1800, 2100]
- BR-005/006: Month range [01, 12]
- BR-007/008: Day range [01, DYS(month)]

**Pseudocode:**

```pseudo
IF year < 1800 OR year > 2100
    EXIT
END IF
IF month < 1 OR month > 12
    EXIT
END IF
IF day < 1 OR day > daysInMonth(month)
    EXIT
END IF
```

**SQL WHERE fragment:**

```sql
AND year BETWEEN 1800 AND 2100
AND month BETWEEN 1 AND 12
AND day BETWEEN 1 AND daysInMonth(month)
```

### 5.6 Mapping Constraints (BR-009 – BR-012)

XFXLDSC restricts `LDAMAP` values, controlling valid mapping codes.

**Pseudocode:**

```pseudo
IF ldamap > 9999
    EXIT
ELSE IF ldamap > 99
    EXIT
END IF
```

**SQL WHERE fragment:**

```sql
AND ldamap <= 9999
AND ldamap <= 99
```

### 5.7 Indicator-Driven Table Exit (BR-013 – BR-016)

XFXTABL uses `*IN79` to switch off table processing.

**Pseudocode:**

```pseudo
IF in79 = 'ON'
    EXIT table maintenance
END IF
```

**SQL WHERE fragment:**

```sql
AND in79 <> 'ON'
```

### 5.8 Combined Summary WHERE Clause

Aggregating the inclusion criteria for HABADTE’s patient-selection logic:

```sql
WHERE fileIndicator <> 0                        -- BR-017
  AND flagIndicator NOT IN ('VOID', 'VOIDED')   -- BR-018
  AND inpatientOutpatientFlag <> 'OUTPATIENT'   -- BR-019
  AND year BETWEEN 1800 AND 2100                -- BR-003, BR-004
  AND month BETWEEN 1 AND 12                    -- BR-005, BR-006
  AND day BETWEEN 1 AND daysInMonth(month)      -- BR-007, BR-008
  AND ldamap <= 9999                            -- BR-012
  AND in79 <> 'ON';                             -- BR-013–BR-016
```

---

## 6. Data Enrichment Steps

Data enrichment is primarily driven by lookups across HXPLVL* hierarchy tables and benefit/status tables as captured in data lineage.

### 6.1 Level Hierarchy Enrichment

- **Source:** XFXLDSC
- **Purpose:** Load level descriptors from HXPLVL1–HXPLVL6 based on a mapping code (`LDAMAP`).
- **Key Fields:** HX1NUM, HX2NUM, HX3NUM, HX4NUM, HX5NUM, HX6NUM.

**Sample SELECT:**

```sql
SELECT
    l1.levelCode, l2.levelCode, l3.levelCode,
    l4.levelCode, l5.levelCode, l6.levelCode
FROM Level1 l1
JOIN Level2 l2 ON l2.parentLevel1Id = l1.id
JOIN Level3 l3 ON l3.parentLevel2Id = l2.id
JOIN Level4 l4 ON l4.parentLevel3Id = l3.id
JOIN Level5 l5 ON l5.parentLevel4Id = l4.id
JOIN Level6 l6 ON l6.parentLevel5Id = l5.id
WHERE l6.levelKey = @hx6num
  AND l6.deletedFlag = 0;
```

Derived flags can identify incomplete hierarchies (e.g., missing level segments) and set a `hierarchyIncompleteFlag` indicator.

### 6.2 Benefit Plan Enrichment

- **Source:** OXPBNFIT / HXPBNFIT
- **Purpose:** Attach benefit details to patient encounters based on `XFBUBN` and `XFBPLN`.

**Sample SELECT:**

```sql
SELECT
    benefitBundleNumber,
    benefitPlanCode,
    contactTelephone,
    deletedFlag
FROM BenefitPlan
WHERE benefitBundleNumber = @bundle
  AND benefitPlanCode      = @plan
  AND deletedFlag = 0;
```

Derived flags:

- `hasActiveBenefits` – true if a non-deleted row exists.
- `missingBenefits` – true if no matching plan can be found.

### 6.3 Status Enrichment

- **Source:** HXPNSTN
- **Purpose:** Translate status codes (e.g., `XFNSST`) into human-readable statuses.

**Sample SELECT:**

```sql
SELECT statusCode, statusDescription
FROM InpatientStatus
WHERE level6Key   = @level6Key
  AND statusCode  = @statusCode
  AND deletedFlag = 0;
```

Derived flags:

- `isDischarged`
- `isAdmitted`
- `isPending` based on status description.

---

## 7. Counting and Aggregation Rules

From the interpretations detail, HABADTE’s key rules aggregate record-level filters into batch-level metrics.
Typical counters include:

| Counter Name          | Based On Rule(s)              | Description |
|----------------------|------------------------------|-------------|
| totalRecordsRead     | All records before filters    | Count of all rows read from primary tables. |
| totalRecordsSkipped  | BR-017, BR-018, BR-019        | Number of records skipped due to file indicator, void flag, or outpatient status. |
| totalInpatientKept   | Opposite of BR-019            | Number of inpatient encounters retained. |
| totalInvalidDates    | BR-003–BR-008                 | Count of rows failing date-validity checks. |
| totalInvalidMappings | BR-009–BR-012                 | Count of rows with out-of-range mapping codes. |

These aggregations are surfaced as summary fields in the migrated service, enabling operational monitoring.

---

## 8. Output Data Structure

The HABADTE processing flow produces structured outputs that combine patient-level, account-level, and benefit/status enrichment results.

### Header Fields (Encounter-Level)

Derived from OMPMAST and hierarchy tables:

| Field              | Source           | Type    |
|--------------------|------------------|---------|
| headerMrn          | OMPMAST.MMMRNO   | STRING  |
| headerAccount      | OMPMAST.MMACCT   | STRING  |
| headerPatientName  | OMPMAST.MMNAME   | STRING  |
| headerEncounterMrn | OMPMAST.MMMMRN   | STRING  |
| headerLevel6Key    | HXPLVL6.HX6NUM   | STRING  |

### Detail Fields (Transfer and Status Rows)

| Field                | Source                        | Type   |
|----------------------|-------------------------------|--------|
| detailTransferDate   | HAPTRFR.AFTRDT                | DATE   |
| detailTransferTime   | HAPTRFR.AFTRTM                | TIME   |
| detailTransferType   | HAPTRFR.AFTYPE                | STRING |
| detailInpatientStatus| HXPNSTN.XFNSST                | STRING |
| detailBenefitPlan    | OXPBNFIT.XFBPLN               | STRING |
| detailBenefitBundle  | OXPBNFIT.XFBUBN               | STRING |

### Summary Fields (Aggregations)

| Field                 | Derived From       |
|-----------------------|--------------------|
| totalRecordsRead      | Counting rules     |
| totalRecordsSkipped   | BR-017–BR-019      |
| totalInpatientKept    | BR-019 inverse     |
| totalInvalidDates     | BR-003–BR-008      |
| totalInvalidMappings  | BR-009–BR-012      |

Output collections should be sorted by `headerMrn, headerAccount, detailTransferDate, detailTransferTime` to reproduce predictable batch ordering.

---

## 9. Complete Processing Flow (Mapped to Spring Boot Layers)

The HABADTE engine can be conceptualized as a layered Spring Boot service.

1. **STEP 1 – Init (Controller Layer)**
   - Validate inbound request payload.
   - Enforce authentication and authorization.
   - Map request parameters to internal MRN/account structures.

2. **STEP 2 – Preferences (Service Layer)**
   - Load configuration flags equivalent to `*IN79` and other control indicators.
   - Evaluate active/disabled states using XFXTABL logic (BR-013–BR-016).

3. **STEP 3 – Context (Service Layer)**
   - Pull patient header from `PatientEncounterMaster` (OMPMAST).
   - Pull related transfer and status context from `PatientTransfer` and `InpatientStatus`.
   - Compute derived metadata such as level hierarchy keys from HXPLVL*.

4. **STEP 4 – Query (Repository Layer)**
   - Execute filtered queries incorporating BR-003–BR-008 (date validity) and BR-009–BR-012 (mapping limits).
   - Apply `fileIndicator`, `flagIndicator`, and `inpatientOutpatientFlag` filters (BR-017–BR-019).

5. **STEP 5 – Per-Record Enrichment (Service Layer)**
   - For each retained record, join benefit plans (OXPBNFIT/HXPBNFIT) and status descriptors (HXPNSTN).
   - Set derived flags such as `hasActiveBenefits` and `isDischarged`.

6. **STEP 6 – Assemble Response (Controller/Service Layer)**
   - Aggregate detail rows into a response DTO with header, detail, and summary sections.
   - Populate aggregation counters and any warning indicators (e.g., `missingBenefits`).
   - Serialize to JSON and return to the caller.

---

## 10. Data Type Conversions

Key AS400 data representations must be converted to Java/Spring Boot friendly types:

- `DECIMAL(8,0)` representing YYYYMMDD → `LocalDate` via string parsing.
- Year (`VYY`), month (`VMM`), day (`VDD`) separate numeric fields → composed `LocalDate` after validation.
- Time fields (e.g., `AFTRTM` HHMM or HHMMSS) → `LocalTime`.
- Packed decimals for account numbers → `BigDecimal` or `String` depending on usage.
- Single-character indicators (e.g., `*IN79`, void flags) → `boolean` or small `Enum` in Java.

All PHI fields (MRN, SSN, names, phone numbers) must be handled via explicit encryption or tokenization policies where required by organizational standards.

---

## 11. SQL Server Table Mapping

The AS400 objects are mapped to SQL Server tables as follows:

| AS400 Object | Category   | SQL Server Table         | Notes |
|-------------|-----------|-------------------------|-------|
| OMPMAST     | DDS_PF    | PatientEncounterMaster  | Core patient/encounter header. |
| HAPTRFR     | DDS_PF    | PatientTransfer         | Transfer events between levels. |
| OXPBNFIT    | DDS_PF    | BenefitPlan             | Benefit plan catalog. |
| HXPLVL1–6   | DDS_PF    | Level1–Level6           | Hierarchy tables. |
| HXPNSTN     | DDS_LF    | InpatientStatus         | Status mapping per level. |
| OAPIRNK     | DDS_PF    | BreakRank               | Ranking metrics per level/account. |

### Suggested Covering Indexes

- `IX_PatientEncounterMaster_MrnAccount` on `(mrn, accountNumber)`.
- `IX_PatientTransfer_AccountDateTime` on `(account, transferDate, transferTime)`.
- `IX_BenefitPlan_BundlePlan` on `(benefitBundleNumber, benefitPlanCode)`.
- `IX_InpatientStatus_LevelStatus` on `(level6Key, statusCode)`.

These indexes support the main lookups observed in the dependency and lineage data.

---

## 12. Spring Boot API Design

### REST Endpoint

- **Method:** `POST`
- **Path:** `/api/habadte/patient-encounters`
- **Request Body (JSON):**

```json
{
  "patientMrn": "123456",
  "accountNumber": "A0001",
  "benefitPlanCode": "PLN01",
  "benefitBundleNumber": "B001",
  "inpatientOutpatientFlag": "INPATIENT",
  "voidFlag": "ACTIVE",
  "fileIndicator": 1
}
```

### Controller / Service / Repository Layers

- **Controller:** validates input, performs security checks, invokes service.
- **Service:** orchestrates rules BR-017–BR-019 and calls repositories for header, detail, and enrichment.
- **Repositories:** encapsulate SQL Server queries against the mapped tables.

### Response JSON Sample

```json
{
  "header": {
    "mrn": "123456",
    "accountNumber": "A0001",
    "patientName": "Sample Patient",
    "level6Key": "L6-001"
  },
  "details": [
    {
      "transferDate": "2024-06-01",
      "transferTime": "10:30:00",
      "transferType": "ADMIT",
      "statusCode": "INPATIENT",
      "benefitPlanCode": "PLN01",
      "benefitBundleNumber": "B001"
    }
  ],
  "summary": {
    "totalRecordsRead": 25,
    "totalRecordsSkipped": 3,
    "totalInpatientKept": 22,
    "totalInvalidDates": 0,
    "totalInvalidMappings": 0
  }
}
```

Entity and DTO classes should align with SQL Server table schemas described earlier.

---

## 13. Performance Considerations

The original HABADTE flow collaborates with helper programs like XFXCYMD, XFXCNTR, XFXLDSC, and XFXTABL.
These dependencies suggest per-record date validation, mapping checks, and indicator evaluation, which can introduce N+1 query patterns if naively ported.

Recommendations:

- Consolidate validations into single SQL predicates executed within set-based queries.
- Use joins to bring in benefit and status enrichment in one pass (e.g., joining `PatientEncounterMaster` with `BenefitPlan` and `InpatientStatus`).
- Cache reference data (e.g., mapping codes and status descriptions) in-memory or via Redis to avoid repeated lookups.
- Batch process large inputs using streaming and pagination.

---

## 14. Business Rules Reference Summary

| Rule ID | Domain             | Source Program | Description |
|---------|--------------------|----------------|-------------|
| BR-001  | DATA_MAINTENANCE   | XFXCNTR        | When X equals zero, branch to EXIT. |
| BR-002  | DATA_MAINTENANCE   | XFXCNTR        | When X equals 40, branch to EXIT. |
| BR-003  | DATA_MAINTENANCE   | XFXCYMD        | When VYY is less than 1800, branch to EXIT. |
| BR-004  | DATA_MAINTENANCE   | XFXCYMD        | When VYY is greater than 2100, branch to EXIT. |
| BR-005  | DATA_MAINTENANCE   | XFXCYMD        | When VMM is less than 01, branch to EXIT. |
| BR-006  | DATA_MAINTENANCE   | XFXCYMD        | When VMM is greater than 12, branch to EXIT. |
| BR-007  | DATA_MAINTENANCE   | XFXCYMD        | When VDD is less than 01, branch to EXIT. |
| BR-008  | DATA_MAINTENANCE   | XFXCYMD        | When VDD is greater than DYS(VMM), branch to EXIT. |
| BR-009  | DATA_MAINTENANCE   | XFXLDSC        | When LDAMAP is greater than 99, branch to EXIT. |
| BR-010  | DATA_MAINTENANCE   | XFXLDSC        | When LDAMAP is greater than 99, branch to EXIT. |
| BR-011  | DATA_MAINTENANCE   | XFXLDSC        | When LDAMAP is greater than 99, branch to EXIT. |
| BR-012  | DATA_MAINTENANCE   | XFXLDSC        | When LDAMAP is greater than 9999, branch to EXIT. |
| BR-013  | DATA_MAINTENANCE   | XFXTABL        | When *IN79 equals on/active, branch to EXIT. |
| BR-014  | DATA_MAINTENANCE   | XFXTABL        | When *IN79 equals on/active, branch to EXIT. |
| BR-015  | DATA_MAINTENANCE   | XFXTABL        | When *IN79 equals on/active, branch to EXIT. |
| BR-016  | DATA_MAINTENANCE   | XFXTABL        | When *IN79 equals on/active, branch to EXIT. |
| BR-017  | PATIENT_MANAGEMENT | HABADTE        | When -FILE INDICATOR equals zero, branch to SKIP. |
| BR-018  | PATIENT_MANAGEMENT | HABADTE        | When -FLAG INDICATOR equals void/voided, branch to SKIP. |
| BR-019  | PATIENT_MANAGEMENT | HABADTE        | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to SKIP. |

---

## 15. Edge Cases

Edge cases inferred from key rules and data structures:

- **Null or Zero File Indicator (BR-017):** Records with indicator `0` must not be processed; in the API, treat missing or null indicators as invalid and return a validation error.
- **Void or Voided Records (BR-018):** Records marked as void should be excluded; the service should log these occurrences for audit.
- **Outpatient Encounters (BR-019):** If outpatient records are not supported by HABADTE, respond with an explicit message indicating that only inpatient encounters are handled.
- **Invalid Date Combinations (BR-003–BR-008):** If year, month, or day values fail validation, the record should be tagged as `invalidDate` and omitted from the core response while counted in summary metrics.
- **Out-of-Range Mapping Codes (BR-009–BR-012):** If `ldamap` exceeds allowable thresholds, treat mapping as unknown and skip hierarchy enrichment for that record.
- **Missing Benefit/Status Rows:** If benefit or status lookups return no rows, set derived flags (`missingBenefits`, `statusUnknown`) and continue processing without failing the entire batch.
- **Empty Result Sets:** When filters remove all records, return an empty `details` array with summary counts all zero and a 200 OK status, rather than an error.

These behaviors should be codified in unit and integration tests to preserve legacy semantics in the migrated solution.
