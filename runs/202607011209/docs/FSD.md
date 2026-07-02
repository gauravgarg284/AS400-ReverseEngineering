# Business Processing Rules and Functional Specification

## HABADTE Patient Management Extract (HABADTE)

### For Java / Spring Boot / SQL Server Migration

---

## (1) Business Purpose

The HABADTE program filters and prepares inpatient account records for downstream XML generation and printing. It reads transfer and benefit data, applies business rules to exclude voided or outpatient records, and writes consolidated XML output. The migration target is a Spring Boot service that exposes equivalent filtering logic via a REST API backed by SQL Server tables.

---

## (2) Inputs / API Request Parameters

The legacy program operates primarily in batch mode over AS400 physical/logical files. For the migration, we expose a RESTful API with request parameters that map to key AS400 fields.

### Proposed REST Endpoint

`GET /api/habadte/inpatient-transfers`

### Request Parameters

| Parameter                 | AS400 Field / Concept        | Type        | Description |
|---------------------------|------------------------------|------------|-------------|
| `accountNumber`          | `AFACCT` (HAPTRFR)           | string     | Patient account number to filter on. If omitted, all eligible accounts are processed. |
| `medicalRecordNumber`    | `AFMRNO` / `MMMRNO`          | string     | Medical record number; used for correlation across HAPTRFR and OMPMAST. |
| `fromTransferDate`       | `AFTRDT`                     | date       | Lower bound for transfer date. |
| `toTransferDate`         | `AFTRDT`                     | date       | Upper bound for transfer date. |
| `inpatientOnly`          | `-INPATIENT/OUTPATIENT FLAG` | boolean    | When true, excludes outpatient records (BR-019). |
| `includeVoided`          | `-FLAG INDICATOR`            | boolean    | When false, excludes voided records (BR-018). |
| `fileIndicatorRequired`  | `-FILE INDICATOR`            | boolean    | When true, requires file indicator > 0 (BR-017). |

#### Spring Boot Security Mapping Note

All endpoints exposing PHI-bearing fields (account number, MRN, names, SSN, phone) must:

- Be protected by OAuth2/OpenID Connect with patient-management specific scopes.
- Enforce role-based access (e.g., `ROLE_PATIENT_MGMT_VIEW`) at the controller level.
- Log all access to PHI fields for audit purposes.

---

## (3) Organizational Hierarchy

The extracted logic operates on patient-level records without explicit organizational hierarchy (e.g., facility, region, business unit) encoded in the analyzed members. Any such hierarchy likely resides in missing reference tables or upstream systems.

**Organizational hierarchy: Not applicable in the analyzed HABADTE subset.**

---

## (4) Primary Data Source

The primary transactional data source is the transfer file `HAPTRFR`.

### SQL Server Table Mapping

- AS400 PF: `HAPTRFR` (record format `HAFTRFR`)
- Target SQL Server table: `PatientTransfer`

### Access Pattern

HABADTE declares and reads `HAPTRFR` and related logical/physical files to drive the main processing loop. The equivalent SQL query selects transfers by account, date and type.

### Key Fields

From `data_dict_schema.physical_files`:

- `AFLVL6` – Level 6 key (e.g., facility/plan level)
- `AFACCT` – Account number (PHI: AccountNumber)
- `AFTRDT` – Transfer date
- `AFTRTM` – Transfer time
- `AFTYPE` – Transfer type code

### Example SQL Server SELECT

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFTRDT,
    AFTRTM,
    AFTYPE
FROM PatientTransfer
WHERE AFTRDT BETWEEN @fromTransferDate AND @toTransferDate
  AND (@accountNumber IS NULL OR AFACCT = @accountNumber)
ORDER BY AFTRDT ASC, AFTRTM ASC;
```

The ORDER BY mirrors the chronological processing implied by `AFTRDT` and `AFTRTM` keys.

---

## (5) Inclusion and Exclusion Rules

Business rules are drawn from `approved_rules` and applied primarily by HABADTE and its utility programs.

### Rule BR-017 – File Indicator Required

- **Description**: When `-FILE INDICATOR` equals zero, branch to `SKIP`.
- **Source Program**: HABADTE (PATIENT_MANAGEMENT)

**Pseudocode IF**

```pseudo
IF fileIndicator = 0
    SKIP record
ENDIF
```

**SQL WHERE Fragment**

```sql
AND fileIndicator <> 0
```

### Rule BR-018 – Voided Records Excluded

- **Description**: When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.
- **Source Program**: HABADTE

**Pseudocode IF**

```pseudo
IF flagIndicator IN ('VOID', 'VOIDED')
    SKIP record
ENDIF
```

**SQL WHERE Fragment**

```sql
AND flagIndicator NOT IN ('VOID', 'VOIDED')
```

### Rule BR-019 – Outpatient Records Excluded

- **Description**: When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.
- **Source Program**: HABADTE

**Pseudocode IF**

```pseudo
IF inOutFlag = 'OUTPATIENT'
    SKIP record
ENDIF
```

**SQL WHERE Fragment**

```sql
AND inOutFlag <> 'OUTPATIENT'
```

### Combined Summary WHERE Clause

```sql
WHERE AFTRDT BETWEEN @fromTransferDate AND @toTransferDate
  AND (@accountNumber IS NULL OR AFACCT = @accountNumber)
  -- Ensure record is physically present/active
  AND fileIndicator <> 0
  -- Exclude voided/voided-out records
  AND flagIndicator NOT IN ('VOID', 'VOIDED')
  -- Restrict to inpatient records only
  AND inOutFlag <> 'OUTPATIENT'
```

Rules BR-001–BR-016 from XFXCNTR, XFXCYMD, XFXLDSC and XFXTABL contribute additional exclusion behavior around counters, date ranges and table lookups. Those are enforced in utility layers (see Sections 7 and 9).

---

## (6) Data Enrichment Steps

Secondary lookups and transformations derived from `data_lineage`.

### Enrichment 1 – Level Descriptions via XFXLDSC

- **Source**: `HXPLVL1`–`HXPLVL6` physical files and `HXFLVL1`–`HXFLVL6` read operations.
- **Program**: `XFXLDSC` (DATA_MAINTENANCE)

HABADTE calls XFXLDSC, which:

- Declares level tables `HXPLVL1`–`HXPLVL6`.
- Reads formatted records `HXFLVL1`–`HXFLVL6`.

**Key Fields**

- Level keys: `HX1NUM` … `HX6NUM` (unique per level).

**SELECT Query**

```sql
SELECT HX1NUM, HX2NUM, HX3NUM, HX4NUM, HX5NUM, HX6NUM
FROM LevelHierarchy
WHERE HX6NUM = @afLvl6;
```

**Delete-Flag Logic / Derived Flags**

No explicit delete flags are visible in the compact schema; derived flags are driven by mapping limits enforced by LDAMAP (BR-009–BR-012). In the migrated design, LDAMAP checks can be implemented as `CHECK` constraints or validation methods.

### Enrichment 2 – Table Lookups via XFXTABL

- **Source**: `HXPTABLD`, `HXLTABLD`, `HXLTABLP`, `HXLTABLS`, `XFFTABLD`, `XFFTABL2`, `XFFTABL3`, `XFFTABL4`.
- **Program**: `XFXTABL`

XFXTABL reads multiple table variants to resolve descriptive codes.

**Example SELECT for table-dictionary lookup**

```sql
SELECT XFDTCD, XFDECD
FROM TableDictionary
WHERE XFDTCD = @codeType
ORDER BY XFDTCD;
```

Indicator *IN79 rules (BR-013–BR-016) are used to stop processing once a desired mapping is found.

### Enrichment 3 – ID Retrieval via XFXGETID

- **Source**: `HXPXMLR`, `HXFXMLR` (DECLARE/READ by XFXGETID).

XFXGETID reads XML-related reference records to derive patient or document IDs used in HABADTE outputs.

```sql
SELECT XMRUSR, XMRSEQ, XMRID
FROM XmlReference
WHERE XMRUSR = @userId
  AND XMRSEQ = @sequence;
```

### Enrichment 4 – Benefit and Status Data

From data lineage:

- `TXPBNFIT` via `HXPBNFIT` (PFILE_OF)
- `TXPNSTN` via `HXPNSTN` (PFILE_OF)

These logical files provide benefit plan and status information.

```sql
SELECT XFBUBN, XFBPLN, XFBTEL
FROM BenefitPlan
WHERE XFBUBN = @benefitId;
```

```sql
SELECT XFNLV6, XFNSST
FROM PatientStatus
WHERE XFNLV6 = @lvl6Key;
```

---

## (7) Counting and Aggregation Rules

Counting behavior is mostly implicit in the HABADTE narrative and key rules.

From `interpretations_detail`:

- HABADTE: "Contains 3 rule(s) with avg confidence 97%." Rules BR-017–BR-019 define which records are included.

### Inferred Counters

| Counter Name                | Based On              | Description |
|----------------------------|----------------------|-------------|
| `totalRecordsRead`         | All HAPTRFR rows     | Number of records scanned from the transfer file. |
| `totalRecordsSkippedFile`  | BR-017               | Records skipped due to missing file indicator. |
| `totalRecordsSkippedVoid`  | BR-018               | Records skipped because flagged as void/voided. |
| `totalRecordsSkippedOutpt` | BR-019               | Records skipped because flagged outpatient. |
| `totalRecordsIncluded`     | BR-017–BR-019        | Records that pass all filters and are written to XML. |

These counters should be explicitly modeled in the service implementation for observability.

---

## (8) Output Data Structure

HABADTE writes XML header and detail records via `HXFXMLH` (read/update/write) and `HXFXMLD` (write).

### Header Fields (HXFXMLH)

In absence of full DDS, we infer typical header contents:

- Document/control identifiers (from XFXGETID and XML reference tables).
- Account-level identifiers (`AFACCT`, `AFMRNO`, possibly `MMACCT`, `MMMRNO`).
- Date/time fields (`AFTRDT`, `AFTRTM`, system date/time).

### Detail Fields (HXFXMLD)

Detail records include per-transfer or per-benefit data, e.g.:

- Transfer-specific attributes (`AFTYPE`).
- Benefit/plan information from `OXPBNFIT` / `HXPBNFIT` (`XFBUBN`, `XFBPLN`, `XFBTEL`).
- Status fields from `HXPNSTN`.

### Summary Fields

Summaries are likely embedded as counters or totals in XML attributes (e.g., number of qualified transfers, total voided). These map directly to the counters in Section 7.

### Sort Order

Records are sorted chronologically by transfer date/time and potentially grouped by account:

- Primary sort: `AFTRDT`, `AFTRTM`.
- Secondary sort: `AFACCT`.

---

## (9) Complete Processing Flow

Mapped to Spring Boot layers using program narratives.

### STEP 1 – Init (Controller Layer)

- Receive HTTP request at `/api/habadte/inpatient-transfers`.
- Validate query parameters (dates, flags).
- Initialize aggregation counters and logging context.

### STEP 2 – Preferences (Service Layer)

- Apply default behavior: `inpatientOnly=true`, `includeVoided=false`, `fileIndicatorRequired=true`.
- Load any configuration for LDAMAP maximums and table lookup preferences (XFXLDSC, XFXTABL).

### STEP 3 – Context Assembly (Service Layer)

- Use XFXGETID-equivalent service to resolve document IDs and XML context based on user/session.
- Join patient master (`OMPMAST`) and transfer (`PatientTransfer`) records using `AFACCT` and `AFMRNO` / `MMMRNO`.

### STEP 4 – Query (Repository Layer)

- Execute SQL SELECT over `PatientTransfer` and related benefit/status tables with WHERE clause from Section 5.
- Stream results to avoid loading the entire set in memory.

### STEP 5 – Per-Record Enrichment (Service Layer)

For each qualifying row:

1. Enforce counter/date rules using XFXCNTR, XFXCYMD and XFXLDSC equivalents.
2. Call table-lookup service modeled on XFXTABL to resolve descriptive codes.
3. Enrich with benefit and status data (from BenefitPlan and PatientStatus).
4. Produce XML header/detail representations.

### STEP 6 – Assemble Response (Controller/Service Layer)

- Aggregate XML fragments or map them to DTOs.
- Return a JSON payload with header and detail structures or a generated XML document, depending on client expectations.

---

## (10) Data Type Conversions

AS400 data types must be mapped to Java and SQL Server types.

Examples inferred from fields:

- `AFTRDT` (numeric or packed date) → `java.time.LocalDate` / `DATE` in SQL Server.
- `AFTRTM` (HHMM or HHMMSS) → `java.time.LocalTime` / `TIME` in SQL Server.
- MRN fields (`AFMRNO`, `MMMRNO`, etc.) → `VARCHAR(20)` / `String`.
- Account numbers (`AFACCT`, `MMACCT`, `HVACCT`) → `VARCHAR(20)` with not-null constraints.
- Indicator fields (`fileIndicator`, `flagIndicator`, `inOutFlag`) → `CHAR(1)` or small enumerations mapped to Java `enum`.

Where numeric representations encode dates (as seen in VYY/VMM/VDD for XFXCYMD), conversion utilities should reconstruct `LocalDate` only if all components pass BR-003–BR-008 validations.

---

## (11) SQL Server Table Mapping

Mapping AS400 objects to SQL Server tables using `data_dict_schema`.

### Core Tables

| AS400 Object | Type    | SQL Server Table    | Key Fields                                    |
|-------------|--------|----------------------|-----------------------------------------------|
| HAPTRFR     | PF     | PatientTransfer      | `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE` |
| OMPMAST     | PF     | PatientMaster        | `MMPLV6`, `MMACCT`                            |
| OAPIRNK     | PF     | BreakRank            | `BRKLV6`, `BRKACC`, `BRKSEQ`                  |
| OXPBNFIT    | PF     | BenefitPlan          | `XFBUBN`, `XFBPLN`                            |
| HXPDICT     | PF     | PatientDictionary    | composite keys (none declared as unique)      |

### Covering Index Suggestions

- `IX_PatientTransfer_AccountDate` on (`AFACCT`, `AFTRDT`, `AFTRTM`).
- `IX_PatientMaster_Account` on (`MMACCT`).
- `IX_BenefitPlan_Plan` on (`XFBUBN`, `XFBPLN`).
- `IX_PatientDictionary_MRN` on (`CCMRNO`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `XMDMRN`).

These indexes support the primary filter and join scenarios of HABADTE.

---

## (12) Spring Boot API Design

### Controller

```java
@RestController
@RequestMapping("/api/habadte")
public class HabadteController {

    private final HabadteService service;

    @GetMapping("/inpatient-transfers")
    @PreAuthorize("hasRole('ROLE_PATIENT_MGMT_VIEW')")
    public ResponseEntity<List<TransferDto>> getInpatientTransfers(
            @RequestParam(required = false) String accountNumber,
            @RequestParam(required = false) String medicalRecordNumber,
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fromTransferDate,
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate toTransferDate) {
        List<TransferDto> result = service.getInpatientTransfers(accountNumber, medicalRecordNumber,
                fromTransferDate, toTransferDate);
        return ResponseEntity.ok(result);
    }
}
```

### Service Skeleton

```java
@Service
public class HabadteService {

    private final TransferRepository transferRepository;

    public List<TransferDto> getInpatientTransfers(String accountNumber, String mrn,
                                                   LocalDate fromDate, LocalDate toDate) {
        List<Transfer> transfers = transferRepository
                .findQualifiedTransfers(accountNumber, mrn, fromDate, toDate);
        // apply BR-017, BR-018, BR-019 and enrichment steps
        return transfers.stream()
                .filter(this::passesIndicators)
                .map(this::toDto)
                .toList();
    }

    private boolean passesIndicators(Transfer t) {
        return t.getFileIndicator() != 0
            && !t.getFlagIndicator().isVoided()
            && !t.getInOutFlag().isOutpatient();
    }
}
```

### Repository Example

```java
public interface TransferRepository extends JpaRepository<Transfer, Long> {

    @Query("SELECT t FROM Transfer t " +
           "WHERE t.transferDate BETWEEN :fromDate AND :toDate " +
           "AND (:accountNumber IS NULL OR t.accountNumber = :accountNumber)" )
    List<Transfer> findQualifiedTransfers(@Param("accountNumber") String accountNumber,
                                          @Param("mrn") String mrn,
                                          @Param("fromDate") LocalDate fromDate,
                                          @Param("toDate") LocalDate toDate);
}
```

### Response JSON Sample

```json
[
  {
    "accountNumber": "123456",
    "medicalRecordNumber": "MRN001",
    "transferDate": "2024-01-15",
    "transferTime": "13:45:00",
    "transferType": "ADMIT",
    "benefitPlan": "PLN01",
    "phone": "555-123-4567",
    "status": "ACTIVE"
  }
]
```

DTO and entity fields are sourced from `PatientTransfer`, `PatientMaster`, `BenefitPlan` and XML context tables.

---

## (13) Performance Considerations

The legacy design performs per-record enrichment and multiple table lookups. In a naive migration, this leads to N+1 query patterns.

Risks identified:

- For each transfer row, separate queries to benefit, status and dictionary tables.
- Repeated table lookups in XFXTABL-equivalent services.

Mitigations:

- Use JOINs in the base query to bring in required data in a single roundtrip.
- Apply batching for reference data; cache table-dictionary entries in memory.
- Rely on covering indexes (Section 11) to keep join operations efficient.

Example optimized query:

```sql
SELECT t.*, m.*, b.*, s.*
FROM PatientTransfer t
JOIN PatientMaster m ON m.MMACCT = t.AFACCT
LEFT JOIN BenefitPlan b ON b.XFBUBN = t.BenefitId
LEFT JOIN PatientStatus s ON s.XFNLV6 = t.AFLVL6;
```

---

## (14) Business Rules Reference Summary

| Rule ID | Description                                                            | Domain             | Source Program |
|---------|------------------------------------------------------------------------|--------------------|----------------|
| BR-001  | When X equals zero, branch to `EXIT`.                                  | DATA_MAINTENANCE   | XFXCNTR        |
| BR-002  | When X equals 40, branch to `EXIT`.                                    | DATA_MAINTENANCE   | XFXCNTR        |
| BR-003  | When VYY is less than 1800, branch to `EXIT`.                          | DATA_MAINTENANCE   | XFXCYMD        |
| BR-004  | When VYY is greater than 2100, branch to `EXIT`.                       | DATA_MAINTENANCE   | XFXCYMD        |
| BR-005  | When VMM is less than 01, branch to `EXIT`.                            | DATA_MAINTENANCE   | XFXCYMD        |
| BR-006  | When VMM is greater than 12, branch to `EXIT`.                         | DATA_MAINTENANCE   | XFXCYMD        |
| BR-007  | When VDD is less than 01, branch to `EXIT`.                            | DATA_MAINTENANCE   | XFXCYMD        |
| BR-008  | When VDD is greater than DYS(VMM), branch to `EXIT`.                   | DATA_MAINTENANCE   | XFXCYMD        |
| BR-009  | When LDAMAP is greater than 99, branch to `EXIT`.                      | DATA_MAINTENANCE   | XFXLDSC        |
| BR-010  | When LDAMAP is greater than 99, branch to `EXIT`.                      | DATA_MAINTENANCE   | XFXLDSC        |
| BR-011  | When LDAMAP is greater than 99, branch to `EXIT`.                      | DATA_MAINTENANCE   | XFXLDSC        |
| BR-012  | When LDAMAP is greater than 9999, branch to `EXIT`.                    | DATA_MAINTENANCE   | XFXLDSC        |
| BR-013  | When *IN79 equals on/active, branch to `EXIT`.                         | DATA_MAINTENANCE   | XFXTABL        |
| BR-014  | When *IN79 equals on/active, branch to `EXIT`.                         | DATA_MAINTENANCE   | XFXTABL        |
| BR-015  | When *IN79 equals on/active, branch to `EXIT`.                         | DATA_MAINTENANCE   | XFXTABL        |
| BR-016  | When *IN79 equals on/active, branch to `EXIT`.                         | DATA_MAINTENANCE   | XFXTABL        |
| BR-017  | When -FILE INDICATOR equals zero, branch to `SKIP`.                    | PATIENT_MANAGEMENT | HABADTE        |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to `SKIP`.             | PATIENT_MANAGEMENT | HABADTE        |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP`.   | PATIENT_MANAGEMENT | HABADTE        |

---

## (15) Edge Cases

Edge cases inferred from key rules and data structures:

- **Missing File Indicator (BR-017)**: Records with `fileIndicator = 0` are skipped. Migration must treat null or zero indicators equivalently and avoid including such records.
- **Voided Records (BR-018)**: Records flagged as void/voided are excluded. The flag column should be modeled as an enumeration; unrecognized values require defensive handling (e.g., default to excluded or log for review).
- **Outpatient Records (BR-019)**: Outpatient records are skipped when the endpoint is defined for inpatient flows. Future endpoints might explicitly include outpatient flows with a different rule set.
- **Invalid Dates (BR-003–BR-008)**: Any date whose year, month or day falls outside valid ranges causes early exit from validation logic. Migrated services must reject such input or treat corresponding records as invalid.
- **LDAMAP Overflow (BR-009–BR-012)**: Overly large mapping codes are not processed; downstream logic must not assume LDAMAP beyond configured thresholds.
- **Empty Result Sets**: If all records are skipped by the rules, the service should return an empty list with HTTP 200 and clearly indicate that no records met inclusion criteria.
- **Null PHI Fields**: MRN, account, name, SSN and phone fields may be null or missing. When performing joins, use left joins and explicit null checks to avoid unintended exclusions.
