# Business Processing Rules and Functional Specification

## HABADTE Patient Management (HABADTE)

### For Java / Spring Boot / SQL Server Migration

---

## (1) Business Purpose

The HABADTE program orchestrates patient-account level processing for inpatient and outpatient encounters. It reads transfer, master, benefit, and status records to determine whether each encounter should be included in downstream XML output and reporting. The logic primarily serves hospital billing and patient management teams, and it is executed in batch mode for a set of patient accounts and admission episodes.

---

## (2) Inputs / API Request Parameters

In a modern Java / Spring Boot implementation, the HABADTE batch logic would be exposed as a REST API or scheduled job. Key request parameters map to AS400 file keys and indicators.

| Parameter Name                 | AS400 Field / Concept     | Type        | Description |
|--------------------------------|---------------------------|------------|-------------|
| accountLevel6                  | AFLVL6 / MMPLV6 / XFNLV6 | String(6)  | Level-6 account key used across transfer, master, and status files. |
| accountNumber                  | AFACCT / MMACCT          | String(10) | Core financial account number used for billing and patient master lookup. |
| inpatientOutpatientFlag        | -INPATIENT/OUTPATIENT     | String(1)  | Flag indicating whether the encounter is inpatient (`I`) or outpatient (`O`). |
| fileIndicator                  | -FILE INDICATOR           | Integer    | Numeric flag indicating whether the current record/file is active and should be processed. |
| voidFlag                       | -FLAG INDICATOR           | String(1)  | Flag indicating if the record has been voided or canceled. |
| benefitPlanCode                | XFBPLN                    | String     | Benefit plan identifier used in benefit lookup (OXPBNFIT/HXPBNFIT). |
| benefitBundleNumber           | XFBUBN                    | String     | Benefit bundle or grouping key. |
| printerDestination            | PRINTER                   | String     | Logical printer or output destination for any generated reports. |
| xmlUserId                     | XMDUSR / XMRUSR           | String     | User context for XML header/detail records (HXPXMLD/HXPXMLR). |

Spring Boot security mapping:

- All endpoints handling PHI-bearing fields (`AFMRNO`, `HVACCT`, `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, etc.) must be protected by Spring Security with role-based access (e.g., `ROLE_BILLING`, `ROLE_PATIENT_ADMIN`).
- Request parameters containing MRN, account, or SSN values must be validated and logged using audit trails without storing raw values in logs.

---

## (3) Organizational Hierarchy

The HABADTE program operates at the level of patient accounts and encounters rather than organizational hierarchy.

Organizational hierarchy is therefore not explicitly modeled in the source and is **not applicable** as a separate dimension in this specification. Any organizational reporting (e.g., by facility or department) would be derived from master data in other systems.

---

## (4) Primary Data Source

The primary data sources for HABADTE’s processing are the following physical files:

1. **HAPTRFR (HAFTRFR)** – Transfer records with account and MRN PHI.
   - Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
   - PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

2. **OMPMAST (HMFMAST)** – Patient master records.
   - Key fields: `MMPLV6`, `MMACCT`
   - PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`

3. **OXPBNFIT (XFFBNFIT)** – Benefit configuration.
   - Key fields: `XFBUBN`, `XFBPLN`
   - PHI fields: `XFBTEL` (PhoneNumber)

4. **HXPNSTN (XFFNSTN)** – Level-6 status/notation records.
   - Key fields: `XFNLV6`, `XFNSST`

### Access Pattern

The HABADTE orchestration reads and declares these files via RPGLE:

- Declare cursors or record formats for each PF and related LF.
- For each eligible account or encounter, read `HAPTRFR`, `OMPMAST`, `OXPBNFIT`, and `HXPNSTN` records to determine inclusion or exclusion.

### Example SQL Server SELECT Pattern

```sql
SELECT  t.AFLVL6,
        t.AFACCT,
        t.AFTRDT,
        t.AFTRTM,
        t.AFTYPE,
        m.MMMRNO,
        m.MMNAME,
        m.MMPSSN,
        b.XFBUBN,
        b.XFBPLN,
        s.XFNSST
FROM    dbo.HAPTRFR t
JOIN    dbo.OMPMAST m
        ON m.MMPLV6 = t.AFLVL6
       AND m.MMACCT = t.AFACCT
LEFT JOIN dbo.OXPBNFIT b
        ON b.XFBUBN = t.AFLVL6
       AND b.XFBPLN = m.MMPLN
LEFT JOIN dbo.OXPNSTN s
        ON s.XFNLV6 = t.AFLVL6
ORDER BY t.AFLVL6, t.AFACCT, t.AFTRDT, t.AFTRTM;
```

### Key Fields Table

| SQL Server Table | Key Fields                          |
|------------------|-------------------------------------|
| HAPTRFR          | AFLVL6, AFACCT, AFTRDT, AFTRTM     |
| OMPMAST          | MMPLV6, MMACCT                     |
| OXPBNFIT         | XFBUBN, XFBPLN                     |
| OXPNSTN          | XFNLV6, XFNSST                     |

---

## (5) Inclusion and Exclusion Rules

Business rules controlling whether a record is processed or skipped are derived directly from HABADTE and its supporting utility programs.

### BR-017 – Skip when file indicator is zero

**Description:** Records with a zero file indicator are considered inactive and must be skipped.

**Pseudocode (IF):**

```pseudo
IF fileIndicator = 0
    GOTO SKIP_RECORD
ENDIF
```

**SQL WHERE fragment:**

```sql
WHERE fileIndicator <> 0
```

### BR-018 – Skip when flag indicates voided record

**Description:** Records marked as void or voided are excluded from XML output and downstream processing.

**Pseudocode (IF):**

```pseudo
IF voidFlag IN ('V', 'VOID')
    GOTO SKIP_RECORD
ENDIF
```

**SQL WHERE fragment:**

```sql
WHERE voidFlag NOT IN ('V', 'VOID')
```

### BR-019 – Skip outpatient encounters for this flow

**Description:** When the inpatient/outpatient flag indicates outpatient, the record is skipped for this specific flow (which focuses on inpatient admission discharge and transfer).

**Pseudocode (IF):**

```pseudo
IF inOutFlag = 'O'
    GOTO SKIP_RECORD
ENDIF
```

**SQL WHERE fragment:**

```sql
WHERE inOutFlag <> 'O'
```

### Date validation utility rules (XFXCYMD)

These rules ensure that date components are within valid ranges before the HABADTE orchestration uses them.

- **BR-003 / BR-004 – Year bounds**

  ```pseudo
  IF VYY < 1800 OR VYY > 2100
      GOTO EXIT_INVALID_DATE
  ENDIF
  ```

  ```sql
  WHERE VYY BETWEEN 1800 AND 2100
  ```

- **BR-005 / BR-006 – Month bounds**

  ```pseudo
  IF VMM < 1 OR VMM > 12
      GOTO EXIT_INVALID_DATE
  ENDIF
  ```

  ```sql
  WHERE VMM BETWEEN 1 AND 12
  ```

- **BR-007 / BR-008 – Day bounds**

  ```pseudo
  IF VDD < 1 OR VDD > DYS(VMM)
      GOTO EXIT_INVALID_DATE
  ENDIF
  ```

  ```sql
  WHERE VDD BETWEEN 1 AND DYS(VMM)
  ```

### Counter and control rules (XFXCNTR, XFXLDSC, XFXTABL)

- **BR-001 / BR-002 – Counter exit thresholds**

  ```pseudo
  IF X = 0 OR X = 40
      GOTO EXIT_COUNTER_PROCESS
  ENDIF
  ```

- **BR-009–BR-012 – LDAMAP bounds**

  ```pseudo
  IF LDAMAP > 9999
      GOTO EXIT_MAP_PROCESS
  ELSEIF LDAMAP > 99
      GOTO EXIT_MAP_PROCESS
  ENDIF
  ```

- **BR-013–BR-016 – Table indicator exit**

  ```pseudo
  IF *IN79 = *ON
      GOTO EXIT_TABLE_PROCESS
  ENDIF
  ```

These utility rules protect the HABADTE orchestration from invalid control values and mapping configurations.

### Summary WHERE Clause

Combining the HABADTE-specific inclusion/exclusion rules into a single SQL WHERE:

```sql
WHERE fileIndicator <> 0          -- BR-017: require active file
  AND voidFlag NOT IN ('V','VOID') -- BR-018: exclude voided records
  AND inOutFlag <> 'O'            -- BR-019: restrict to inpatient
```

---

## (6) Data Enrichment Steps

Data enrichment involves secondary lookups and XML-related processing described in `data_lineage.json`.

### Benefit enrichment via OXPBNFIT / HXPBNFIT

- **Purpose:** Resolve patient benefit details (plan and bundle) for each account.
- **Key fields:** `XFBUBN`, `XFBPLN`

**Example SELECT:**

```sql
SELECT  XFBUBN,
        XFBPLN,
        XFBTEL
FROM    dbo.OXPBNFIT
WHERE   XFBUBN = @accountLevel6
  AND   XFBPLN = @benefitPlanCode
  AND   deleteFlag = 0;
```

Derived flags:

- `hasBenefit = (row found)`
- `contactPhone = XFBTEL`

### Status enrichment via OXPNSTN / HXPNSTN

- **Purpose:** Determine current status of the level-6 account (e.g., active, discharged).
- **Key fields:** `XFNLV6`, `XFNSST`

**Example SELECT:**

```sql
SELECT  XFNLV6,
        XFNSST
FROM    dbo.OXPNSTN
WHERE   XFNLV6 = @accountLevel6
  AND   deleteFlag = 0;
```

Derived flags:

- `isActive = (XFNSST = 'A')`

### XML header and detail enrichment via HXPXMLD / HXPXMLR

According to data_lineage, HABADTE declares and writes `HXFXMLD` (detail) and reads `HXFXMLH`/`HXPXMLD`/`HXPXMLR` for XML orchestration.

**Key fields:**

- Header: `XMRUSR`, `XMRSEQ`, `XMRID`
- Detail: `XMDUSR`, `XMDSEQ`, `XMDSQ2`

**Example SELECT/INSERT:**

```sql
-- Read existing header
SELECT XMRUSR, XMRSEQ, XMRID
FROM   dbo.HXPXMLR
WHERE  XMRUSR = @xmlUserId
  AND  XMRSEQ = @xmlSeq;

-- Insert new detail
INSERT INTO dbo.HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2, payload)
VALUES (@xmlUserId, @xmlSeq, @detailSeq, @jsonPayload);
```

Delete-flag logic (general pattern):

- Each PF/LF should have a `deleteFlag` or equivalent indicator mapped to a boolean column in SQL Server.
- Enrichment queries must include `WHERE deleteFlag = 0` to avoid processing retired records.

---

## (7) Counting and Aggregation Rules

Counting logic is inferred from the key rules in utility programs:

- **Encounter counters (XFXCNTR)**
  - Counter `X` is incremented per processed record.
  - If `X = 0` or `X = 40`, processing branches to `EXIT`, representing thresholds for minimum/maximum iterations.

- **Valid date counts (XFXCYMD)**
  - Only dates that pass year, month, and day bounds contribute to aggregates such as "valid encounter days".

- **Mapping counts (XFXLDSC)**
  - `LDAMAP` ranges control whether mappings are counted as valid or out-of-range.

Sample aggregation in SQL:

```sql
SELECT  COUNT(*) AS totalProcessed,
        SUM(CASE WHEN inOutFlag = 'I' THEN 1 ELSE 0 END) AS inpatientCount,
        SUM(CASE WHEN inOutFlag = 'O' THEN 1 ELSE 0 END) AS outpatientSkipped,
        SUM(CASE WHEN voidFlag IN ('V','VOID') THEN 1 ELSE 0 END) AS voidedCount
FROM    dbo.HabAccountFlow
WHERE   fileIndicator <> 0;
```

---

## (8) Output Data Structure

The target output is a combination of XML records and a JSON/REST response representing patient account flows.

### Header Fields

- `xmlUserId` (XMRUSR/XMDUSR)
- `xmlSequence` (XMRSEQ/XMDSEQ)
- `xmlDetailSeq` (XMDSQ2)

### Detail Fields (per account/encounter)

Derived from `HAPTRFR`, `OMPMAST`, `OXPBNFIT`, and `OXPNSTN`:

- `accountLevel6` (AFLVL6/MMPLV6/XFNLV6)
- `accountNumber` (AFACCT/MMACCT)
- `mrn` (AFMRNO/MMMRNO/MMMRN)
- `patientName` (MMNAME)
- `ssn` (MMPSSN)
- `transferDate` (AFTRDT)
- `transferTime` (AFTRTM)
- `transferType` (AFTYPE)
- `benefitPlanCode` (XFBPLN)
- `benefitBundleNumber` (XFBUBN)
- `statusCode` (XFNSST)

### Summary Fields

Aggregated counts per run:

- `totalProcessed`
- `totalSkippedFileIndicator`
- `totalSkippedVoid`
- `totalSkippedOutpatient`

Sort order:

- Primary: `accountLevel6`
- Secondary: `accountNumber`
- Tertiary: `transferDate`, `transferTime`

---

## (9) Complete Processing Flow

Using narratives from `interpretations_detail`, the HABADTE flow can be mapped to Spring Boot layers.

1. **STEP 1 – Init (Controller/Batch Launcher)**
   - Spring Boot batch job or REST controller validates request parameters (account range, run options).
   - HABADTE-equivalent service initializes counters (XFXCNTR) and mappings (XFXLDSC).

2. **STEP 2 – Preferences (Configuration Load)**
   - Load mapping tables and control flags using `XFXTABL` and related DDS files (`HXPTABLD`, `HXLTABLD`, `HXLTABLP`, `HXLTABLS`).
   - Configuration is cached at application startup.

3. **STEP 3 – Context (Patient and Account Context)**
   - For each account, read patient master (`OMPMAST`) and transfer (`HAPTRFR`) records.
   - Validate dates via `XFXCYMD` to ensure only valid encounter windows are processed.

4. **STEP 4 – Query (Benefit and Status Lookup)**
   - Perform lookups to `OXPBNFIT`/`OXPNSTN` using the benefit and status keys.
   - Apply inclusion/exclusion rules BR-017–BR-019 to filter records.

5. **STEP 5 – Per-Record Enrichment (XML and Derived Fields)**
   - Build XML header/detail using `HXPXMLD`/`HXPXMLR` and write via `HXFXMLD`/`HXFXMLH` equivalents.
   - Enrich responses with PHI fields where permitted, masked or tokenized per compliance requirements.

6. **STEP 6 – Assemble Response (Return or Persist)**
   - Aggregate results into response JSON or persist to SQL Server tables.
   - Return summary statistics and detailed records to the caller.

---

## (10) Data Type Conversions

Typical AS400-to-Java/SQL conversions in this system:

- **Dates (numeric YYYYMMDD)**
  - RPG fields like `AFTRDT` or `WBDATE` may be stored as numeric or zoned decimals.
  - Java type: `LocalDate`
  - SQL type: `DATE`

- **Times (HHMM or HHMMSS)**
  - RPG fields like `AFTRTM` may be numeric.
  - Java type: `LocalTime`
  - SQL type: `TIME(0)` or `TIME(0)` with appropriate precision.

- **Level-6 account keys (AFLVL6/MMPLV6/XFNLV6)**
  - Stored as character or packed numeric in AS400.
  - Java type: `String`
  - SQL type: `VARCHAR(6)`.

- **Account numbers (AFACCT/MMACCT/HVACCT/IHACCT)**
  - Java type: `String`
  - SQL type: `VARCHAR(10)`.

- **MRN fields (AFMRNO/CCMRNO/HXGMRN/IMGMRN/MMMRNO/MMMRN/XMDMRN)**
  - Java type: `String`
  - SQL type: `VARCHAR(15)` with masking rules.

- **Telephone numbers (XFBTEL)**
  - Java type: `String`
  - SQL type: `VARCHAR(20)`.

- **SSN (MMPSSN)**
  - Java type: `String`
  - SQL type: `CHAR(9)` with strict encryption.

---

## (11) SQL Server Table Mapping

AS400 objects and their proposed SQL Server tables:

| AS400 Object | Type    | SQL Server Table | Notes |
|-------------|---------|------------------|-------|
| HAPTRFR     | PF      | dbo.HAPTRFR      | Transfer records keyed by AFLVL6+AFACCT+AFTRDT+AFTRTM. |
| OMPMAST     | PF      | dbo.OMPMAST      | Patient master keyed by MMPLV6+MMACCT. |
| OXPBNFIT    | PF      | dbo.OXPBNFIT     | Benefit configuration keyed by XFBUBN+XFBPLN. |
| OXPNSTN     | PF      | dbo.OXPNSTN      | Status/notation keyed by XFNLV6+XFNSST. |
| HXPXMLD     | PF      | dbo.HXPXMLD      | XML detail records keyed by XMDUSR+XMDSEQ+XMDSQ2. |
| HXPXMLR     | PF      | dbo.HXPXMLR      | XML header records keyed by XMRUSR+XMRSEQ+XMRID. |

### Suggested Covering Indexes

- On `dbo.HAPTRFR`:

  ```sql
  CREATE INDEX IX_HAPTRFR_AccountDateTime
  ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM);
  ```

- On `dbo.OMPMAST`:

  ```sql
  CREATE INDEX IX_OMPMAST_AccountLevel
  ON dbo.OMPMAST (MMPLV6, MMACCT);
  ```

- On `dbo.OXPBNFIT`:

  ```sql
  CREATE INDEX IX_OXPBNFIT_BundlePlan
  ON dbo.OXPBNFIT (XFBUBN, XFBPLN);
  ```

---

## (12) Spring Boot API Design

### REST Endpoint

```http
POST /api/habadte/process
Content-Type: application/json
Authorization: Bearer <token>
```

Request JSON sample:

```json
{
  "accountLevel6": "123456",
  "accountNumber": "0001234567",
  "inpatientOutpatientFlag": "I",
  "fileIndicator": 1,
  "voidFlag": " ",
  "benefitPlanCode": "PLN1",
  "benefitBundleNumber": "BN1",
  "xmlUserId": "HABUSER1"
}
```

### Layered Design

- **Controller:** `HabadteController` – validates input, triggers service.
- **Service:** `HabadteService` – encapsulates business rules BR-017–BR-019 and orchestrates enrichment.
- **Repository:** `HaptrfrRepository`, `OmpmastRepository`, `OxpbnfitRepository`, `OxpnstnRepository` – Spring Data JPA repositories mapping to SQL Server tables.

Entity sketch (JPA):

```java
@Entity
@Table(name = "HAPTRFR")
public class TransferRecord {
    @Id
    private Long id; // synthetic key

    private String aflvl6;
    private String afacct;
    private LocalDate aftrdt;
    private LocalTime aftrtm;
    private String aftype;
}
```

DTO sketch:

```java
public class HabadteResponseItem {
    private String accountLevel6;
    private String accountNumber;
    private String mrn;
    private String patientName;
    private String statusCode;
    private String benefitPlanCode;
    private boolean included;
}
```

---

## (13) Performance Considerations

- **N+1 query risk:**
  - The legacy code reads benefit and status records per account, which can become multiple queries per row in a naïve JPA implementation.
  - Mitigation: Use bulk joins (as shown in the SELECT pattern) or batch fetch with appropriate `JOIN` strategies.

- **Large dictionary file (HXPDICT):**
  - Although HABADTE does not directly iterate over the 2705-field `HXPDICT`, downstream services may.
  - Mitigation: Only load required columns via projections; avoid `SELECT *`.

- **High complexity hotspot (HABADTE):**
  - Cyclomatic complexity 152 indicates many branches. Care should be taken to refactor into smaller service methods instead of a monolithic method.

---

## (14) Business Rules Reference Summary

| Rule ID | Description |
|--------|-------------|
| BR-001 | When X equals zero, branch to 'EXIT'. |
| BR-002 | When X equals 40, branch to 'EXIT'. |
| BR-003 | When VYY is less than 1800, branch to 'EXIT'. |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'. |
| BR-005 | When VMM is less than 01, branch to 'EXIT'. |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'. |
| BR-007 | When VDD is less than 01, branch to 'EXIT'. |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'. |
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

---

## (15) Edge Cases

Edge cases are inferred from the rule set and data structures:

- **Null or zero indicators:**
  - `fileIndicator = 0` leads to skipping the record (BR-017).
  - Missing or null flags should be treated as non-void and non-outpatient, but logged for data quality review.

- **Outpatient records:**
  - Outpatient encounters are skipped in this flow (BR-019) but may require alternative processing; ensure they are not silently dropped in the migrated system.

- **Voided records:**
  - Records marked voided are excluded (BR-018); implement a separate audit trail for these to support reconciliation.

- **Invalid dates:**
  - Years outside 1800–2100, months outside 1–12, and days outside 1–`DYS(VMM)` cause exit from date routines.
  - In Java, these should raise validation errors rather than corrupt date objects.

- **Missing enrichment records:**
  - If benefit or status records are not found, the system should default `hasBenefit = false` and `isActive = null` and continue processing, while logging warnings.

- **Empty result sets:**
  - When no records satisfy the WHERE clause, the API should return an empty list with `totalProcessed = 0` rather than an error.

