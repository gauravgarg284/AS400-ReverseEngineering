# Business Processing Rules & Functional Specification

## Patient Transfer & XML Export Report (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

## (1) Business Purpose

HABADTE is the primary patient management orchestration program. It reads inpatient transfer records, applies a series of skip rules, normalizes status and level information, and produces XML-based output records for downstream reporting or integration.

The report/process is used by clinical operations and billing support teams to review and transmit patient transfer activity for a given location and time window. It typically runs as part of a batch or scheduled job at end-of-day or on demand when transfer audits are required.

> Core business question: *"For a given unit and time range, which patient transfer records are valid, billable, and should be emitted to downstream systems, and which records must be skipped due to file status, void flags, or outpatient status?"*

Summary outputs produced:
- A sequence of XML header and detail records (HXFXMLH/HXFXMLD) representing validated transfers.
- Counts of processed, skipped, and voided records for audit and reconciliation.
- Status-normalized views of patient transfers using normalized status table XFFNSTN.

---

## (2) Inputs (API Request Parameters)

In the modernized Spring Boot API, HABADTE’s behavior is exposed as a REST endpoint. The following request parameters correspond to AS400 fields and drive the main selection logic.

| Parameter           | AS400 Field / Source      | Type        | Description |
|---------------------|---------------------------|-------------|-------------|
| `unitLevel6`        | AFLVL6 (HAPTRFR)          | integer     | Level-6 unit/location code used to select transfers for a specific ward or department. |
| `accountNumber`     | AFACCT (HAPTRFR)          | long        | Optional account filter; if provided, restricts processing to a single patient account. |
| `transferDateFrom`  | AFTRDT (HAPTRFR)          | LocalDate   | Start of transfer date range (YYYYMMDD in legacy). |
| `transferDateTo`    | AFTRDT (HAPTRFR)          | LocalDate   | End of transfer date range. |
| `transferTimeFrom`  | AFTRTM (HAPTRFR)          | LocalTime   | Start of transfer time window (HHMMSS). |
| `transferTimeTo`    | AFTRTM (HAPTRFR)          | LocalTime   | End of transfer time window. |
| `transferType`      | AFTYPE (HAPTRFR)          | string      | Optional transfer type filter (e.g., admit, discharge, internal move). |
| `includeOutpatients`| In/Out flag (derived from HABADTE logic) | boolean     | Indicates whether outpatient records should be included; default is `false` to mirror BR-019. |
| `runMode`           | HABADTE control field     | string      | Indicates batch vs on-demand mode; affects logging and commit behavior. |

Additional context is obtained from the security framework rather than explicit parameters:

- **Current user identity** is inferred from the Spring Security context (e.g., OAuth2 principal). In legacy, this aligns with user IDs stored in fields such as AFUSER in HAPTRFR or HXFXMLH user stamps.
- **Spring Boot security mapping**:
  - OAuth2/OpenID Connect used to authenticate callers.
  - Role-based access control (RBAC) restricts this API to roles such as `ROLE_TRANSFER_AUDIT` or `ROLE_BILLING_ANALYST`.
  - All PHI access (AFMRNO, MMNAME, MMACCT, etc.) is audited using standard Spring Boot logging plus database audit tables.

---

## (3) Organizational Hierarchy

The HABADTE process is driven by a level-6 location key (AFLVL6 in HAPTRFR) and supported by hierarchical configuration tables HXPLVL1–HXPLVL6. These model enterprise, region, facility, unit, and sub-unit structures.

| Level Number | Name               | Key Size (digits) | AS400 Table |
|--------------|--------------------|-------------------|-------------|
| 1            | Enterprise         | 4                 | HXPLVL1     |
| 2            | Region / Network   | 4                 | HXPLVL2     |
| 3            | Facility / Hospital| 4                 | HXPLVL3     |
| 4            | Campus / Building  | 4                 | HXPLVL4     |
| 5            | Unit Group         | 4                 | HXPLVL5     |
| 6            | Unit / Level-6    | 6                 | HXPLVL6     |

**Business Rule Note (BR-family 009–012):** Report headers and transfer summaries must present location information derived from valid level codes only. Any transfer record with a level code outside the configured ranges in HXPLVL1–6 is rejected by XFXLDSC and does not appear in the output.

---

## (4) Patient Data Source

The primary entity is the **patient transfer** record, driven by the physical file **HAPTRFR**.

### 4.1 Data Access Pattern

Legacy HABADTE reads HAPTRFR with a composite key:
- AFLVL6 (level-6 location)
- AFACCT (patient account)
- AFTRDT (transfer date)
- AFTRTM (transfer time)
- AFTYPE (transfer type)

The program processes transfers sequentially for a given unit and date/time window, applying skip rules before enriching with status and level information.

Access pattern:
- Primary file: **HAPTRFR**
- Mode: **Keyed sequential** over the composite key, constrained by date/time range.
- Sort order: AFLVL6, AFTRDT, AFTRTM, AFACCT, AFTYPE.

SQL Server equivalent SELECT (core query):

```sql
SELECT
    AFLVL6       AS UnitLevel6,
    AFACCT       AS AccountNumber,
    AFTRDT       AS TransferDate,
    AFTRTM       AS TransferTime,
    AFTYPE       AS TransferType,
    AFMRNO       AS MRN,
    AFUSER       AS EnteredBy,
    AFSTAT       AS TransferStatus
FROM dbo.PatientTransfer AS pt
WHERE pt.AFLVL6 = @unitLevel6
  AND pt.AFTRDT BETWEEN @transferDateFrom AND @transferDateTo
  AND pt.AFTRTM BETWEEN @transferTimeFrom AND @transferTimeTo
  AND (@accountNumber IS NULL OR pt.AFACCT = @accountNumber)
  AND (@transferType IS NULL OR pt.AFTYPE = @transferType)
ORDER BY pt.AFTRDT, pt.AFTRTM, pt.AFACCT, pt.AFTYPE;
```

### 4.2 Key Fields

Key fields for the primary data source in SQL Server and their AS400 origins:

| SQL Column         | AS400 Field | Type        | Description |
|--------------------|------------|-------------|-------------|
| `UnitLevel6`       | AFLVL6     | DEC(6,0)    | Level-6 unit/location code. |
| `AccountNumber`    | AFACCT     | DEC(10,0)   | Patient billing account number. |
| `TransferDate`     | AFTRDT     | DEC(8,0)    | Transfer date in YYYYMMDD format. |
| `TransferTime`     | AFTRTM     | DEC(6,0)    | Transfer time in HHMMSS format. |
| `TransferType`     | AFTYPE     | CHAR(1)     | Transfer type code (admit, discharge, internal move). |
| `MRN`              | AFMRNO     | CHAR(12)    | Medical Record Number associated with the transfer. |

---

## (5) Inclusion and Exclusion Rules

This section consolidates HABADTE’s high-confidence skip logic and supporting rules from its domain narrative.

### BR-017 – Skip When File Indicator Equals Zero

**Description**

If the internal FILE INDICATOR is zero (no active records or file not ready), HABADTE branches to `SKIP` for the current record or iteration. This avoids processing incomplete or non-existent file contexts.

**Pseudocode**

```pseudo
IF FileIndicator = 0 THEN
    GOTO SKIP_RECORD
END IF
```

**SQL WHERE Clause Fragment**

```sql
-- BR-017: Skip records when file indicator is zero
WHERE FileIndicator <> 0
```

### BR-018 – Skip When Flag Indicator Equals Void/VoidED

**Description**

If the FLAG INDICATOR is set to "void" or "voided", the transfer record is considered logically deleted or excluded and HABADTE branches to `SKIP`. These records do not contribute to XML output or downstream counts.

**Pseudocode**

```pseudo
IF FlagIndicator IN ('VOID', 'VOIDED') THEN
    GOTO SKIP_RECORD
END IF
```

**SQL WHERE Clause Fragment**

```sql
-- BR-018: Exclude voided records
AND FlagIndicator NOT IN ('VOID', 'VOIDED')
```

### BR-019 – Skip When Inpatient/Outpatient Flag Equals Outpatient

**Description**

If the INPATIENT/OUTPATIENT FLAG indicates outpatient, HABADTE branches to `SKIP`. The process focuses on inpatient transfers for bed management and occupancy reporting; outpatient encounters are out of scope.

**Pseudocode**

```pseudo
IF InOutFlag = 'OUTPATIENT' THEN
    GOTO SKIP_RECORD
END IF
```

**SQL WHERE Clause Fragment**

```sql
-- BR-019: Limit to inpatient transfers
AND InOutFlag <> 'OUTPATIENT'
```

### Summary SQL WHERE Clause

Combining BR-017, BR-018, and BR-019, the primary selection query becomes:

```sql
SELECT ...
FROM dbo.PatientTransfer AS pt
WHERE pt.AFLVL6 = @unitLevel6
  AND pt.AFTRDT BETWEEN @transferDateFrom AND @transferDateTo
  AND pt.AFTRTM BETWEEN @transferTimeFrom AND @transferTimeTo
  AND (@accountNumber IS NULL OR pt.AFACCT = @accountNumber)
  AND (@transferType IS NULL OR pt.AFTYPE = @transferType)
  -- BR-017: File must be active
  AND pt.FileIndicator <> 0
  -- BR-018: Exclude voided records
  AND pt.FlagIndicator NOT IN ('VOID', 'VOIDED')
  -- BR-019: Inpatient only
  AND pt.InOutFlag <> 'OUTPATIENT'
ORDER BY pt.AFTRDT, pt.AFTRTM, pt.AFACCT, pt.AFTYPE;
```

---

## (6) Level and Status Enrichment

The main enrichment steps are driven by **XFXLDSC** and **XFXTABL**, which read level and status/dictionary tables.

### Section 6 – Level Hierarchy Lookup (XFXLDSC)

**Business Concept:** Resolve a transfer’s organizational hierarchy (enterprise → region → facility → unit) by validating level codes against HXPLVL1–HXPLVL6.

**Relevant BR IDs:** BR-009, BR-010, BR-011, BR-012 (level code validity).

**Algorithm Steps**

1. Receive level-6 code from HAPTRFR (AFLVL6).
2. Derive related level keys (HX1NUM–HX6NUM) according to configuration (hierarchical breakdown stored in HXPLVL1–HXPLVL6).
3. For each level file HXPLVL1–HXPLVL6, perform a keyed READ using the derived number.
4. If any level code is missing or outside valid range, apply the corresponding BR-009–BR-012 rule: reject or mark the record as having invalid configuration.
5. If all levels are valid, attach descriptive names and status flags to the in-memory transfer context.

**SQL SELECT Equivalent**

```sql
SELECT
    l1.HX1NUM, l1.HX1DESC,
    l2.HX2NUM, l2.HX2DESC,
    l3.HX3NUM, l3.HX3DESC,
    l4.HX4NUM, l4.HX4DESC,
    l5.HX5NUM, l5.HX5DESC,
    l6.HX6NUM, l6.HX6DESC
FROM dbo.Level1 AS l1
JOIN dbo.Level2 AS l2 ON l2.HX2NUM = @level2Num
JOIN dbo.Level3 AS l3 ON l3.HX3NUM = @level3Num
JOIN dbo.Level4 AS l4 ON l4.HX4NUM = @level4Num
JOIN dbo.Level5 AS l5 ON l5.HX5NUM = @level5Num
JOIN dbo.Level6 AS l6 ON l6.HX6NUM = @level6Num;
```

**Edge Cases**

- **Not-found level code:** Reject the record from output, log a configuration error, and optionally send an alert to configuration management.
- **Inactive level:** If a status field on any level indicates inactive, still process the record but include a warning flag in the XML output.
- **Derived flags:** A flag such as `IsUnitMigrated` can be derived from specific level attributes to drive migration-specific behavior.

---

## (7) Status Normalization Enrichment (HXPNSTN / XFFNSTN)

**Business Concept:** Normalize transfer status codes using the HXP status tables and logical file HXPNSTN, which maps level-6 and status codes to descriptive statuses.

**Algorithm Steps**

1. From each transfer record, derive status key (XFNLV6, XFNSST) based on AFLVL6 and a transfer status code.
2. Read from HXPNSTN (logical over TXPNSTN) using the composite key.
3. If found, attach normalized status description and active flag to the record.
4. If not found, treat the transfer as having an unknown status and either skip or mark for manual review depending on configuration.

**SQL SELECT Equivalent**

```sql
SELECT
    s.XFNLV6 AS UnitLevel6,
    s.XFNSST AS StatusCode,
    s.XFNDSC AS StatusDescription,
    s.XFNACT AS IsActive
FROM dbo.NormalizedStatus AS s
WHERE s.XFNLV6 = @unitLevel6
  AND s.XFNSST = @statusCode;
```

**Edge Cases**

- **Not-found normalized status:** Flag record as `StatusUnknown` but still emit with original raw status.
- **Inactive status:** Include the record but mark as `StatusInactive` for downstream audit.

---

## (8) Dictionary and Benefit Enrichment (HXPTABLD / HXPBNFIT)

Where HABADTE requires descriptive labels or benefit details, it leverages dictionary and benefit tables via logical files HXLTABLD/LP/LS and HXPBNFIT.

**Business Concept:** Attach human-readable descriptions and benefit plan metadata to each transfer record.

**Algorithm Steps**

1. Based on transfer type and flags, derive dictionary keys (XFDTCD, XFDECD, XFDMAP, XFDLDS, XFDSDS).
2. Use XFXTABL to read dictionary tables XFFTABLD/2/3/4 and logical views HXLTABLD/LP/LS.
3. Load description fields (logical, screen, print descriptions) into the transfer context.
4. If benefit-related fields are present, read HXPBNFIT (logical over TXPBNFIT) keyed by XFBUBN/XFBPLN and attach contact phone (XFBTEL) and plan metadata.

**SQL SELECT Equivalent – Dictionary**

```sql
SELECT
    d.XFDTCD,
    d.XFDECD,
    d.XFDMAP,
    d.XFDLDS,
    d.XFDSDS
FROM dbo.Dictionary AS d
WHERE d.XFDTCD = @dataTypeCode
  AND d.XFDECD = @detailCode;
```

**SQL SELECT Equivalent – Benefit Plan**

```sql
SELECT
    b.XFBUBN AS BenefitNumber,
    b.XFBPLN AS PlanCode,
    b.XFBTEL AS ContactPhone,
    b.XFBSTAT AS Status
FROM dbo.BenefitPlan AS b
WHERE b.XFBUBN = @benefitNumber
  AND b.XFBPLN = @planCode;
```

**Edge Cases**

- **Missing dictionary entry:** Use raw code as description and flag record for configuration review.
- **Missing benefit plan:** Emit record without benefit enrichment but log an exception.
- **PHI-related fields (XFBTEL):** Ensure phone numbers are masked or omitted in non-privileged views.

---

## (9) Counting Rules

Counts are derived from key rules in HABADTE’s narrative.

| Counter Name           | Incremented When                                      | Description |
|------------------------|--------------------------------------------------------|-------------|
| `processedCount`       | Record passes all BR-017/018/019 checks               | Number of inpatient transfer records successfully processed. |
| `skippedFileIndicator` | BR-017 triggers (FileIndicator = 0)                    | Number of records skipped due to inactive file context. |
| `skippedVoidFlag`      | BR-018 triggers (FlagIndicator void/voided)           | Number of records skipped because they are logically voided. |
| `skippedOutpatient`    | BR-019 triggers (InOutFlag = OUTPATIENT)              | Number of outpatient records skipped to keep report inpatient-only. |

**Relationship Notes**

- The sum of all counters equals the total number of candidate transfer records read.
- Skipped counters collectively represent exclusions; processedCount is the primary business output.

**Business Meaning**

These counters drive reconciliation and audit, proving that all eligible transfers were either processed or explicitly skipped according to clear business rules.

---

## (10) Output Data Structure

Legacy HABADTE writes XML header and detail records to HXFXMLH and HXFXMLD. In the modern system, these become JSON or XML responses.

### 10.1 Header Fields

| Field Name          | Source          | Description |
|---------------------|-----------------|-------------|
| `runId`             | Batch context   | Unique identifier for the execution run. |
| `unitLevel6`        | AFLVL6          | Unit/location for which transfers are reported. |
| `dateRange`         | AFTRDT          | Date range of transfers. |
| `generatedAt`       | System clock    | Date/time the report was generated. |
| `processedCount`    | Counter         | Number of processed transfers. |
| `skippedCount`      | Derived         | Sum of skipped counters. |

### 10.2 Detail Row Fields

| Field Name       | Source         | Description |
|------------------|----------------|-------------|
| `accountNumber`  | AFACCT         | Patient account associated with the transfer. |
| `mrn`            | AFMRNO         | Medical Record Number. |
| `transferDate`   | AFTRDT         | Date of transfer. |
| `transferTime`   | AFTRTM         | Time of transfer. |
| `transferType`   | AFTYPE         | Transfer type. |
| `normalizedStatus`| HXPNSTN/XFFNSTN| Normalized status description. |
| `levelHierarchy` | HXPLVL1–6      | Expanded location information. |
| `dictionaryLabels`| HXPTABLD/XFFTABLD| Human-readable descriptions for codes. |

### 10.3 Footer/Summary Fields

| Field Name           | Description |
|----------------------|-------------|
| `totalRecordsRead`   | Total records read from PatientTransfer. |
| `processedCount`     | Same as header processedCount. |
| `skippedFileIndicator`| Number skipped due to BR-017. |
| `skippedVoidFlag`    | Number skipped due to BR-018. |
| `skippedOutpatient`  | Number skipped due to BR-019. |

### 10.4 Sort Order

Detail rows are sorted by:

1. `transferDate`
2. `transferTime`
3. `accountNumber`
4. `transferType`

---

## (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1 – Init
  - Load configuration from Level tables (HXPLVL1–6).
  - Initialize counters: processedCount, skippedFileIndicator, skippedVoidFlag, skippedOutpatient.
  - Open primary file PatientTransfer (HAPTRFR) and status tables (HXPNSTN/XFFNSTN).

STEP 2 – Preferences / Lookup
  - For the requested unitLevel6, pre-load level hierarchy via XFXLDSC.
  - Pre-load dictionary entries and benefit plans via XFXTABL and HXPBNFIT for common codes.

STEP 3 – Context
  - Determine runMode (batch or on-demand).
  - Capture current user identity for audit stamps.

STEP 4 – Query
  - Execute primary SELECT on PatientTransfer (see Section 4) with BR-017/018/019 filters.

STEP 5 – Per-Record Enrichment
  For each record returned:
    5a – Validate level codes with XFXLDSC.
    5b – Normalize status via HXPNSTN/XFFNSTN.
    5c – Attach dictionary labels and benefit metadata via XFXTABL/HXPBNFIT.
    5d – Populate XML detail structure (HXFXMLD) with enriched fields.
    5e – Update counters based on rule outcomes.

STEP 6 – Assemble Response
  - Write XML header (HXFXMLH) including counters and run metadata.
  - Commit XML header/detail records.
  - Return JSON/XML response from Spring Boot API mirroring legacy output.
```

---

## (12) Data Type Conversions

### 12.1 Date Fields

Legacy date fields (e.g., AFTRDT, WBDATE) are DECIMAL(8,0) in YYYYMMDD form.

Conversion rule:
- `0` value → `null` (unknown date).
- Otherwise, parse as `LocalDate` using `YYYYMMDD` pattern.

### 12.2 Time Fields

Legacy time fields (e.g., AFTRTM) are DECIMAL(6,0) in HHMMSS or DECIMAL(4,0) in HHMM form.

Conversion rule:
- `0` value → `null` (unknown time).
- Map HHMMSS/HHMM to `LocalTime`.

### 12.3 Packed Decimal Keys

Packed decimal keys (e.g., AFLVL6, AFACCT) become numeric types:
- Convert to `int` or `long` depending on length.
- Preserve leading zeros only where they are part of a display code.

### 12.4 String Trimming

Fixed-length CHAR fields are right-padded with spaces.

Conversion rule:
- Apply `.trim()` to remove trailing spaces.
- Preserve internal spaces (e.g., in patient names).

---

## (13) SQL Server Table Mapping

| AS400 Object   | SQL Server Table       | Purpose |
|----------------|------------------------|---------|
| HAPTRFR        | dbo.PatientTransfer    | Stores patient transfer transactions keyed by unit, account, date/time, type. |
| HXPLVL1–6      | dbo.Level1–dbo.Level6  | Hierarchical location configuration. |
| HXPTABLD       | dbo.Dictionary         | Core dictionary table for code/description mappings. |
| HXLTABLD/LP/LS | dbo.DictionaryViews    | Specialized views for mapping, logical, and screen descriptions. |
| TXPBNFIT       | dbo.BenefitPlanBase    | Base table for benefit plans. |
| HXPBNFIT       | dbo.BenefitPlan        | Keyed view for benefit plans with phone contact. |
| TXPNSTN        | dbo.NormalizedStatusBase | Base status table. |
| HXPNSTN        | dbo.NormalizedStatus   | Keyed status lookup by unit and status code. |
| HXFXMLH/D      | dbo.XmlHeader / dbo.XmlDetail | XML header/detail representation of transfer data. |
| HXPDICT        | dbo.PatientDictionary  | Wide cross-reference table containing MRN, account, and other PHI. |

### Suggested SQL Server Indexes

```sql
-- Primary key for patient transfers
CREATE UNIQUE INDEX IX_PatientTransfer_Primary
ON dbo.PatientTransfer (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE);

-- Supporting index for date-range queries
CREATE INDEX IX_PatientTransfer_DateTime
ON dbo.PatientTransfer (AFTRDT, AFTRTM);

-- Level lookup indexes
CREATE UNIQUE INDEX IX_Level6_Primary
ON dbo.Level6 (HX6NUM);

-- Status normalization
CREATE UNIQUE INDEX IX_NormalizedStatus_Key
ON dbo.NormalizedStatus (XFNLV6, XFNSST);

-- Dictionary lookups
CREATE INDEX IX_Dictionary_TypeDetail
ON dbo.Dictionary (XFDTCD, XFDECD);

-- Benefit plan lookups
CREATE UNIQUE INDEX IX_BenefitPlan_Key
ON dbo.BenefitPlan (XFBUBN, XFBPLN);
```

---

## (14) Spring Boot API Design

### 14.1 Recommended REST Endpoint

**Method & Path:**

`GET /api/transfers/report`

**Parameters**

| Name               | Type        | Required | Validation |
|--------------------|------------|----------|------------|
| `unitLevel6`        | integer     | Yes      | > 0, must exist in Level6. |
| `accountNumber`     | long        | No       | > 0 if provided. |
| `transferDateFrom`  | LocalDate   | Yes      | Valid date, <= `transferDateTo`. |
| `transferDateTo`    | LocalDate   | Yes      | Valid date, >= `transferDateFrom`. |
| `transferTimeFrom`  | LocalTime   | No       | Valid time. |
| `transferTimeTo`    | LocalTime   | No       | Valid time; if provided, >= `transferTimeFrom`. |
| `transferType`      | string      | No       | Must match configured type codes. |
| `includeOutpatients`| boolean     | No       | Default `false`. |

### 14.2 Layer Structure

- **Controller:** `TransferReportController` – handles HTTP requests, validation, and security.
- **Service:** `TransferReportService` – implements selection, skip logic (BR-017/018/019), and orchestration of enrichment services.
- **Repository:** `PatientTransferRepository`, `LevelRepository`, `StatusRepository`, `DictionaryRepository`, `BenefitPlanRepository` – JPA or MyBatis repositories for data access.

### 14.3 Response JSON Shape

```json
{
  "runId": "202607081435",
  "unitLevel6": 123456,
  "dateRange": {
    "from": "2026-07-01",
    "to": "2026-07-01"
  },
  "processedCount": 42,
  "skipped": {
    "fileIndicator": 3,
    "voidFlag": 2,
    "outpatient": 5
  },
  "details": [
    {
      "accountNumber": 1002003000,
      "mrn": "MRN000123456",
      "transferDate": "2026-07-01",
      "transferTime": "14:35:00",
      "transferType": "I",
      "normalizedStatus": "ACTIVE",
      "levelHierarchy": {
        "level1": "Enterprise A",
        "level2": "Region North",
        "level3": "Facility Main",
        "level4": "Building 1",
        "level5": "Unit Group X",
        "level6": "Unit 12A"
      },
      "dictionaryLabels": {
        "transferTypeLabel": "Internal Transfer",
        "statusLabel": "Admitted"
      }
    }
  ]
}
```

### 14.4 Java Entity/DTO Sketch

```java
public record TransferReportRequest(
    int unitLevel6,
    Long accountNumber,
    LocalDate transferDateFrom,
    LocalDate transferDateTo,
    LocalTime transferTimeFrom,
    LocalTime transferTimeTo,
    String transferType,
    boolean includeOutpatients
) {}

public record TransferDetail(
    long accountNumber,
    String mrn,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String normalizedStatus,
    LevelHierarchy levelHierarchy,
    Map<String, String> dictionaryLabels
) {}

public record TransferReportResponse(
    String runId,
    int unitLevel6,
    LocalDate fromDate,
    LocalDate toDate,
    int processedCount,
    int skippedFileIndicator,
    int skippedVoidFlag,
    int skippedOutpatient,
    List<TransferDetail> details
) {}
```

---

## (15) Performance Considerations

The legacy pattern reads HAPTRFR and then, for each record, performs multiple lookups:
- Level hierarchy via HXPLVL1–6 (XFXLDSC).
- Status normalization via HXPNSTN/XFFNSTN.
- Dictionary and benefit lookups via HXPTABLD/HXPBNFIT.

This has an inherent **N+1** risk if naively implemented in the modern system.

Recommended approach:
- Use **set-based queries** and JOINs wherever possible.
- Pre-load configuration tables into memory caches for read-heavy workloads.

Example consolidated SQL using JOINs:

```sql
SELECT
    pt.*,
    s.XFNDSC AS StatusDescription,
    b.XFBTEL AS BenefitContactPhone,
    l6.HX6DESC AS UnitDescription
FROM dbo.PatientTransfer AS pt
LEFT JOIN dbo.NormalizedStatus AS s
    ON s.XFNLV6 = pt.AFLVL6
   AND s.XFNSST = pt.StatusCode
LEFT JOIN dbo.BenefitPlan AS b
    ON b.XFBUBN = pt.BenefitNumber
   AND b.XFBPLN = pt.PlanCode
LEFT JOIN dbo.Level6 AS l6
    ON l6.HX6NUM = pt.AFLVL6
WHERE pt.AFLVL6 = @unitLevel6
  AND pt.AFTRDT BETWEEN @transferDateFrom AND @transferDateTo
  AND pt.AFTRTM BETWEEN @transferTimeFrom AND @transferTimeTo;
```

---

## (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When FILE INDICATOR equals zero, branch to `SKIP` (exclude inactive file records). |
| BR-018  | When FLAG INDICATOR equals void/voided, branch to `SKIP` (exclude logically voided transfers). |
| BR-019  | When INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP` (limit report to inpatient records). |

Supporting rules from other programs (used by HABADTE indirectly):

| Rule ID | Description |
|---------|-------------|
| BR-009–012 | Level lookup: reject if level code exceeds valid range (XFXLDSC). |
| BR-013–016 | When *IN79 equals on/active, branch to `EXIT` for table lookups (XFXTABL). |

---

## (17) Edge Cases to Implement

| Scenario                                   | Expected Behavior |
|--------------------------------------------|-------------------|
| FileIndicator = 0                          | Apply BR-017: skip record; increment `skippedFileIndicator`; do not emit in XML/JSON output. |
| FlagIndicator = VOID or VOIDED             | Apply BR-018: skip record; increment `skippedVoidFlag`; do not emit in output. |
| InOutFlag = OUTPATIENT                     | Apply BR-019: skip record; increment `skippedOutpatient`; do not emit in output. |
| Level code not found in HXPLVL1–6         | Reject record from output; log configuration error; optionally raise alert. |
| Status code not found in HXPNSTN/XFFNSTN  | Emit record with raw status; mark as `StatusUnknown` in output. |
| Benefit plan not found in HXPBNFIT        | Emit record without benefit enrichment; log missing configuration. |
| Date fields = 0                            | Treat as null; avoid casting errors; handle as missing date in UI. |
| Time fields = 0                            | Treat as null; handle as missing time. |
| Empty result set for given filters         | Return empty `details` array; header/footer counts are zero; HTTP 200 with no data. |
| Preference / configuration not set        | Use safe defaults (e.g., exclude outpatients); log warning; do not fail the run. |
