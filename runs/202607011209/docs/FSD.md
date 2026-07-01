# Business Processing Rules and Functional Specification

## HABADTE (HABADTE) / ### For Java / Spring Boot / SQL Server Migration

### (1) Business Purpose

The HABADTE module is a patient management reporting and extract program that assembles patient transfer and benefit information from multiple AS400 physical and logical files. It focuses on account- and MRN-based activity, drawing from transactional transfer records and master patient data. The program is intended for downstream clinical, financial, or reporting consumers that require a consolidated view of patient-related events.

### (2) Inputs / API Request Parameters

| API Parameter | AS400 Field / Concept | Type | Description |
|---------------|-----------------------|------|-------------|
| accountNumber | AFACCT / MMACCT | string | Patient account number used to scope transfer and master records. |
| mrn | AFMRNO / MMMRNO / MRN variations | string | Medical Record Number identifying the patient across files. |
| inpatientFlag | -INPATIENT/OUTPATIENT FLAG | string | Indicates inpatient vs outpatient records; used to skip outpatient in some flows. |
| fileIndicator | -FILE INDICATOR | integer | Call-level or record-level indicator controlling whether a record should be processed. |
| voidFlag | -FLAG INDICATOR | string | Indicates if a record is void/voided and should be skipped. |

Note: In Spring Boot, these inputs will be exposed via a secure REST endpoint. Authentication and authorization should leverage Spring Security with role-based access control. Sensitive identifiers such as MRN, account, and SSN must never be exposed without proper masking and role checks.

### (3) Organizational Hierarchy

The existing AS400 programs and files do not explicitly encode an organizational hierarchy in the analyzed rules or schema. Any hierarchy (e.g., facility → department → unit) would require additional SME validation.

_Not applicable._

### (4) Primary Data Source

The primary physical files and their key structures are:

- **HAPTRFR** (record format HAFTRFR)
  - Purpose: Transfer/transaction file for patient-related activity.
  - Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
  - PHI: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

- **OMPMAST** (record format HMFMAST)
  - Purpose: Master patient/account file.
  - Key fields: `MMPLV6`, `MMACCT`
  - PHI: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`

Example SQL Server SELECT pattern for the primary transfer stream:

```sql
SELECT
    t.AFLVL6,
    t.AFACCT,
    t.AFTRDT,
    t.AFTRTM,
    t.AFTYPE,
    t.AFMRNO,
    m.MMNAME,
    m.MMMRNO,
    m.MMPLV6,
    m.MMACCT
FROM dbo.HAPTRFR AS t
LEFT JOIN dbo.OMPMAST AS m
    ON m.MMPLV6 = t.AFLVL6
   AND m.MMACCT = t.AFACCT
WHERE 1 = 1
ORDER BY t.AFLVL6, t.AFACCT, t.AFTRDT, t.AFTRTM, t.AFTYPE;
```

Key fields table:

| Table | Key Fields | Unique | Notes |
|-------|-----------|--------|-------|
| HAPTRFR | AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE | Yes | Ordered by location, account, date, time, and type. |
| OMPMAST | MMPLV6, MMACCT | Yes | Master record per location and account. |

### (5) Inclusion and Exclusion Rules

The following rules represent filters that determine which records are processed or skipped. They are derived from the approved rules associated with HABADTE.

#### BR-017: Skip when file indicator equals zero

- **Description**: When -FILE INDICATOR equals zero, branch to 'SKIP'.
- **Pseudocode IF**:

```pseudo
IF FILE_INDICATOR = 0
    GO TO SKIP_RECORD
ENDIF
```

- **SQL WHERE Equivalent (filtering to processed records)**:

```sql
WHERE FILE_INDICATOR <> 0
```

#### BR-018: Skip void or voided records

- **Description**: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- **Pseudocode IF**:

```pseudo
IF FLAG_INDICATOR IN ('VOID', 'VOIDED')
    GO TO SKIP_RECORD
ENDIF
```

- **SQL WHERE Equivalent**:

```sql
WHERE FLAG_INDICATOR NOT IN ('VOID', 'VOIDED')
```

#### BR-019: Skip outpatient records

- **Description**: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.
- **Pseudocode IF**:

```pseudo
IF IO_FLAG = 'O'  // Outpatient
    GO TO SKIP_RECORD
ENDIF
```

- **SQL WHERE Equivalent**:

```sql
WHERE IO_FLAG <> 'O'
```

#### Combined Summary WHERE Clause

```sql
WHERE 1 = 1
  AND FILE_INDICATOR <> 0          -- BR-017: ignore uninitialized/no-op records
  AND FLAG_INDICATOR NOT IN ('VOID', 'VOIDED')  -- BR-018: ignore voided rows
  AND IO_FLAG <> 'O'               -- BR-019: ignore outpatient-only transactions
```

### (6) Data Enrichment Steps

The HABADTE program performs read and declare operations that indicate secondary lookups and enrichments.

#### Enrichment 1: Transfer file to master patient file

- **Description**: For each transfer in HAPTRFR, look up the corresponding master patient record in OMPMAST.
- **Key Fields**: `AFLVL6` → `MMPLV6`, `AFACCT` → `MMACCT`.
- **SQL Query**:

```sql
SELECT m.*
FROM dbo.OMPMAST AS m
WHERE m.MMPLV6 = @AFLVL6
  AND m.MMACCT = @AFACCT
  AND ISNULL(m.DELETE_FLAG, ' ') <> 'D';
```

- **Delete-Flag Logic**: Records marked as deleted (DELETE_FLAG = 'D') should be excluded from enrichment.
- **Derived Flags**: A derived flag `HAS_MASTER` can indicate whether a matching master record was found.

#### Enrichment 2: Network station / location lookup

- **Description**: HABADTE references HXPNSTN and XFFNSTN to derive station/location descriptors.
- **Key Fields**: `XFNLV6`, `XFNSST`.
- **SQL Query**:

```sql
SELECT n.*
FROM dbo.OXPNSTN AS n
WHERE n.XFNLV6 = @Level6
  AND n.XFNSST = @StationCode;
```

- **Derived Flags**: A derived `STATION_ACTIVE` flag could be set if the record is not obsolete.

#### Enrichment 3: Benefit dictionary lookup

- **Description**: HABADTE references HXPBNFIT (TXPBNFIT / OXPBNFIT) to resolve benefit or plan codes.
- **Key Fields**: `XFBUBN`, `XFBPLN`.
- **SQL Query**:

```sql
SELECT b.*
FROM dbo.OXPBNFIT AS b
WHERE b.XFBUBN = @BenefitNumber
  AND b.XFBPLN = @PlanCode;
```

- **Sensitive Data**: `XFBTEL` is a phone number and should be masked in downstream outputs.

### (7) Counting and Aggregation Rules

The interpretations for HABADTE highlight control rules around skipping records rather than complex aggregation; however, typical counters inferred from the control flow include:

| Counter Name | Description | Increment Condition |
|--------------|-------------|---------------------|
| totalRecordsRead | Total records read from HAPTRFR. | On every read of HAPTRFR. |
| totalProcessed | Records passing all skip rules. | After BR-017, BR-018, BR-019 all evaluate to "process". |
| totalSkippedByFileIndicator | Records skipped because FILE_INDICATOR = 0. | When BR-017 triggers. |
| totalSkippedVoid | Records skipped as void/voided. | When BR-018 triggers. |
| totalSkippedOutpatient | Records skipped as outpatient. | When BR-019 triggers. |

### (8) Output Data Structure

HABADTE writes to XML-related structures (HXFXMLH, HXFXMLD) and potentially to printed output.

- **Header (HXFXMLH)**: Contains batch-level metadata (e.g., run identifier, timestamp, facility code).
- **Detail (HXFXMLD)**: Contains row-level patient activity details derived from HAPTRFR and enriched files.

Representative fields (derived from the data dictionary schema):

- From **HAPTRFR**: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`, `AFMRNO`.
- From **OMPMAST**: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`.
- From **OAPIRNK / HAPIRNK**: `BRKLV6`, `BRKACC`, `BRKSEQ`, `BRKMRN`.
- From **OXPBNFIT**: `XFBUBN`, `XFBPLN`, `XFBTEL` (phone, PHI).

Sort order in the output is typically aligned with the primary key of HAPTRFR: level, account, date, time, type.

### (9) Complete Processing Flow

The HABADTE program can be mapped to a modern Spring Boot layered architecture as follows:

1. **STEP 1 – Init (Controller)**
   - Receive REST request with account, MRN, and optional filters.
   - Validate required fields and authenticate the caller.

2. **STEP 2 – Preferences (Service)**
   - Load application-level preferences or flags (e.g., whether to include outpatient records) from configuration.

3. **STEP 3 – Context (Service)**
   - Construct a processing context containing account, MRN, facility, and run identifiers.
   - Initialize counters and logging.

4. **STEP 4 – Query (Repository)**
   - Execute the primary HAPTRFR/OMPMAST query with inclusion/exclusion WHERE conditions.
   - Stream results to avoid loading entire datasets into memory.

5. **STEP 5 – Per-Record Enrichment (Service)**
   - For each base record, apply skip rules BR-017, BR-018, BR-019.
   - For records that pass, perform enrichment lookups to station, benefit, and master patient tables.
   - Apply masking to sensitive fields before constructing DTOs.

6. **STEP 6 – Assemble Response (Service/Controller)**
   - Map enriched entities to XML or JSON structures equivalent to HXFXMLH/HXFXMLD.
   - Return a paginated or streamed response to the caller.

### (10) Data Type Conversions

Typical AS400-to-Java type conversions:

| AS400 Type | Example | Java / SQL Server Type | Notes |
|-----------|---------|------------------------|-------|
| DECIMAL(8,0) | Date as YYYYMMDD | `LocalDate` / `DATE` | Convert via string parsing and validation using XFXCYMD rules. |
| DECIMAL(6,0) | Time as HHMMSS | `LocalTime` / `TIME` | Convert to `LocalTime` with zero padding. |
| CHAR(1) | I/O flag | `String` / `CHAR(1)` | Map to enums in Java. |
| NUMERIC(1,0) | Indicator fields | `Integer` / `TINYINT` | Use boolean in Java where appropriate. |

### (11) SQL Server Table Mapping

Representative table mappings for HABADTE:

| AS400 Object | SQL Server Table | Notes |
|--------------|------------------|-------|
| HAPTRFR | dbo.HAPTRFR | Primary transfer fact table. |
| OMPMAST | dbo.OMPMAST | Master patient/account dimension. |
| OAPIRNK | dbo.OAPIRNK | Rank/priority file with MRN. |
| OXPBNFIT | dbo.OXPBNFIT | Benefit/plan configuration with phone numbers. |
| HXPLVL1-6 | dbo.HXPLVL1-6 | Ancillary level tables. |
| HXPTABLD | dbo.HXPTABLD | Code table for dictionary values. |

Suggested covering indexes:

- `IX_HAPTRFR_AFACCT_TRDT_TRTM` on `(AFLVL6, AFACCT, AFTRDT, AFTRTM)`
- `IX_OMPMAST_MMPLV6_MMACCT` on `(MMPLV6, MMACCT)`
- `IX_OAPIRNK_BRKLV6_BRKACC` on `(BRKLV6, BRKACC)`

### (12) Spring Boot API Design

- **Endpoint**: `GET /api/patient-transfers`
- **Controller**: `PatientTransferController`
- **Service**: `PatientTransferService`
- **Repository**: `HaptrfrRepository`, `OmpmastRepository`, `BenefitRepository`, `StationRepository`

Sample JSON response:

```json
{
  "accountNumber": "1234567890",
  "mrn": "MRN12345",
  "facilityLevel": "000001",
  "transfers": [
    {
      "transferDate": "2024-06-01",
      "transferTime": "13:45:00",
      "type": "ADMIT",
      "station": "MED-01",
      "status": "ACTIVE",
      "void": false,
      "inpatient": true
    }
  ]
}
```

### (13) Performance Considerations

- The legacy program performs multiple per-record lookups (e.g., to master, station, benefit tables), creating N+1 access patterns.
- In Spring Boot, favor set-based JOIN queries and batch fetching over individual per-record calls.
- Use streaming or pagination for large result sets.

### (14) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

### (15) Edge Cases

- **Null / Zero Indicators**: When FILE_INDICATOR is null or zero, treat the record as non-processable and skip.
- **Not Found Master Record**: If a matching OMPMAST record is not found, proceed with transfer-level data only and mark a flag `masterMissing`.
- **Delete Flags**: Exclude records with delete flags to avoid resurrecting obsolete data.
- **Empty Result Sets**: If no transfers are found, return an empty array with HTTP 200 and an explanatory message instead of treating it as an error.
