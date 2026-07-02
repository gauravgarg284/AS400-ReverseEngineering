# Business Processing Rules & Functional Specification

## HABADTE Patient Management Report (HABADTE) / ### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE  
**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

---

### (1) Business Purpose

The HABADTE program drives a patient management batch process that reads transaction and configuration files, filters records based on status and patient type, and produces XML messages and notifications for downstream systems.

It is used by patient accounting operations to generate outbound messages for qualifying inpatient accounts while excluding voided and inapplicable records.

> Core business question: "Which patient transactions are eligible for downstream processing and messaging based on account status, void flags, and inpatient/outpatient classification?"

Summary outputs:
- XML header and detail records in `HXFXMLH` and `HXFXMLD` representing patient transaction payloads.
- Notification or statement references via `XFFNSTN`.
- Internal counters and flags indicating how many records were processed or skipped.

---

### (2) Inputs (API Request Parameters)

In the modernized API, HABADTE becomes a service endpoint that accepts request parameters mapping to existing AS400 selection logic.

| Parameter Name       | AS400 Field / Concept      | Type      | Description |
|----------------------|----------------------------|-----------|-------------|
| level6               | AFLVL6                     | integer   | Organisational level (e.g., facility/ward) used to segment transactions. |
| accountNumber        | AFACCT                     | string    | Patient account number used as a primary key in HAPTRFR and OMPMAST. |
| transactionDateFrom  | AFTRDT (lower bound)       | date      | Earliest transaction date to consider. |
| transactionDateTo    | AFTRDT (upper bound)       | date      | Latest transaction date to consider. |
| inpatientOnly        | -INPATIENT/OUTPATIENT FLAG | boolean   | If true, restrict output to inpatient records (BR-019). |
| includeVoided        | -FLAG INDICATOR            | boolean   | If false, skip voided records (BR-018). |
| fileStatusFilter     | -FILE INDICATOR            | integer   | File status threshold to include records; zero indicates skip (BR-017). |

Additional implicit input:
- **Current user identity** – carried in an authentication token and mapped to XML user fields like `XMDUSR` and `XMRUSR` for routing and auditing.

Security mapping in Spring Boot:
- Use OAuth2/JWT for authentication, with RBAC roles such as `ROLE_PATIENT_MGMT` controlling access.
- Log PHI field access (MRN, account numbers, names, SSN) for audit compliance.
- Enforce field-level redaction for non-privileged roles in response payloads.

---

### (3) Organizational Hierarchy

HABADTE uses a multi-level hierarchy encoded in level configuration files `HXPLVL1`–`HXPLVL6` and their DDS formats (`HXFLVL1`–`HXFLVL6`). Level 6 is prominent via fields like `AFLVL6`, `MMPLV6`, and `BRKLV6`.

| Level Number | Name / Concept         | Key Size (digits) | AS400 Table |
|-------------|------------------------|-------------------|-------------|
| 1           | Top-level facility     | 4                 | HXPLVL1     |
| 2           | Region / cluster       | 4                 | HXPLVL2     |
| 3           | Hospital / site        | 4                 | HXPLVL3     |
| 4           | Service line           | 4                 | HXPLVL4     |
| 5           | Department / unit      | 4                 | HXPLVL5     |
| 6           | Ward / room grouping   | 6                 | HXPLVL6     |

BR Note: Report headers and XML payloads should incorporate level descriptors (from `XFXLDSC` lookups) to identify facility and unit context for each transaction and notification.

---

### (4) Patient Data Source (PRIMARY_ENTITY Data Source)

#### 4.1 Data Access Pattern

The primary transactional data source for HABADTE is the physical file **HAPTRFR**:
- Record format: `HAFTRFR`
- Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

In the AS400 implementation, HAPTRFR is read using keyed access over level, account, date and time, iterating records in chronological order per account.

Sort order (derived from keys and usage):
- **Primary sort**: AFLVL6 (ascending) – group by organisational level.
- **Secondary sort**: AFACCT (ascending) – within level, group by account.
- **Tertiary sort**: AFTRDT, AFTRTM (ascending) – chronological transaction history.
- **Quaternary sort**: AFTYPE – transaction type ordering within the same timestamp.

Equivalent SQL Server SELECT:

```sql
SELECT
    AFLVL6,
    AFACCT,
    AFTRDT,
    AFTRTM,
    AFTYPE,
    AFMRNO,
    /* other non-PHI transaction columns */
FROM dbo.HAPTRFR
WHERE AFLVL6 = @level6
  AND AFTRDT BETWEEN @transactionDateFrom AND @transactionDateTo
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

#### 4.2 Key Fields

Key fields table for the primary file:

| SQL Column   | AS400 Field | Type    | Description |
|--------------|-------------|---------|-------------|
| level6       | AFLVL6      | INT     | Organisational level used as partitioning key. |
| account_no   | AFACCT      | VARCHAR | Patient account number (PHI). |
| trans_date   | AFTRDT      | INT/DATE| Transaction date, stored as packed decimal in AS400. |
| trans_time   | AFTRTM      | INT/TIME| Transaction time, stored as packed decimal. |
| trans_type   | AFTYPE      | CHAR    | Transaction type code. |

---

### (5) Inclusion and Exclusion Rules

Inclusion/exclusion rules are driven by HABADTE's key rules (BR-017–BR-019) and applied per transaction.

#### BR-017 – File Status Filter

When the internal **file indicator** equals zero, the record is skipped and not processed further.

Pseudocode:

```pseudo
IF fileIndicator = 0 THEN
    SKIP_RECORD();
ENDIF;
```

SQL WHERE fragment:

```sql
WHERE fileIndicator <> 0
```

#### BR-018 – Void Flag Filter

When the **void flag** equals "void" or "voided", the record is considered cancelled and is excluded.

Pseudocode:

```pseudo
IF flagIndicator IN ('VOID', 'VOIDED') THEN
    SKIP_RECORD();
ENDIF;
```

SQL WHERE fragment:

```sql
AND flagIndicator NOT IN ('VOID', 'VOIDED')
```

#### BR-019 – Inpatient vs Outpatient Filter

When the **inpatient/outpatient flag** equals "outpatient", HABADTE skips the record for this process.

Pseudocode:

```pseudo
IF patientType = 'OUTPATIENT' THEN
    SKIP_RECORD();
ENDIF;
```

SQL WHERE fragment:

```sql
AND patientType <> 'OUTPATIENT'
```

These rules combine to define the eligible record set.

#### Summary SQL WHERE Clause

```sql
WHERE fileIndicator <> 0              -- BR-017
  AND flagIndicator NOT IN ('VOID', 'VOIDED')  -- BR-018
  AND patientType <> 'OUTPATIENT'     -- BR-019
ORDER BY AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE;
```

---

### (6) Level Description Enrichment (First Enrichment Step)

HABADTE calls **XFXLDSC**, which enriches transactions with hierarchical level descriptions by reading configuration files `HXPLVL1`–`HXPLVL6` and their corresponding `HXFLVL*` formats.

BRs involved: BR-009–BR-012.

Algorithm steps:
1. Given a level code (e.g., AFLVL6), map it through LDAMAP (mapping code).
2. Validate that LDAMAP is within allowed ranges:
   - BR-009/010/011: LDAMAP <= 99 for certain contexts.
   - BR-012: LDAMAP <= 9999 for expanded mappings.
3. Read records from `HXFLVL1`–`HXFLVL6` to retrieve textual descriptions (e.g., facility, region, unit, ward).
4. Attach these descriptions to the transaction record for reporting and XML payload.

Pseudocode:

```pseudo
IF LDAMAP > 9999 THEN
    EXIT;  // BR-012
ENDIF;

// For specific mapping types, enforce stricter limit 99
IF contextRequiresStrictMap AND LDAMAP > 99 THEN
    EXIT;  // BR-009–BR-011
ENDIF;

levelDesc1 = readLevelDesc(HXFLVL1, level1Code);
levelDesc2 = readLevelDesc(HXFLVL2, level2Code);
...
levelDesc6 = readLevelDesc(HXFLVL6, level6Code);

attachLevelDescriptionsToRecord(...);
```

SQL SELECT equivalent (simplified):

```sql
SELECT
    l1.HX1NUM, l1.HX1DSC,
    l2.HX2NUM, l2.HX2DSC,
    l3.HX3NUM, l3.HX3DSC,
    l4.HX4NUM, l4.HX4DSC,
    l5.HX5NUM, l5.HX5DSC,
    l6.HX6NUM, l6.HX6DSC
FROM dbo.HXFLVL1 l1
JOIN dbo.HXFLVL2 l2 ON l2.parentKey = l1.HX1NUM
JOIN dbo.HXFLVL3 l3 ON l3.parentKey = l2.HX2NUM
JOIN dbo.HXFLVL4 l4 ON l4.parentKey = l3.HX3NUM
JOIN dbo.HXFLVL5 l5 ON l5.parentKey = l4.HX4NUM
JOIN dbo.HXFLVL6 l6 ON l6.parentKey = l5.HX5NUM
WHERE @LDAMAP BETWEEN @mapMin AND @mapMax;
```

Edge cases:
- Not-found codes: if a level code does not exist, default descriptions like "UNKNOWN LEVEL" should be returned; the record remains but flagged.
- Delete-flag logic: if configuration rows include active/disabled flags, disabled rows should not be used.
- Derived flags: configuration can drive boolean flags such as `isCriticalUnit`.

---

### (7) Table Dictionary Enrichment (Second Enrichment Step)

XFXTABL enriches records using the table dictionary **HXPTABLD** and its logical views **HXLTABLD/HXLTABLP/HXLTABLS**.

BRs involved: BR-013–BR-016.

Algorithm steps:
1. Determine the table code (XFDTCD) and detail code (XFDECD) relevant to the transaction.
2. Use indicator *IN79 to control whether table lookup should proceed.
3. Read `HXPTABLD` via logical views to obtain descriptions and mapping options.
4. Attach resolved code/description pairs and additional table-driven parameters to the transaction.

Pseudocode:

```pseudo
IF indicator79 = 'ON' THEN
    EXIT; // BR-013–BR-016: skip lookup if indicator is active
ENDIF;

record = SELECT *
         FROM HXPTABLD
         WHERE XFDTCD = :dataCode
           AND XFDECD = :detailCode;

IF recordFound THEN
    applyTableMapping(record);
ELSE
    flagMissingTableEntry(dataCode, detailCode);
ENDIF;
```

SQL SELECT equivalent:

```sql
SELECT XFDTCD, XFDECD, XFDMAP, XFDLDS, XFDSDS
FROM dbo.HXPTABLD
WHERE XFDTCD = @dataCode
  AND XFDECD = @detailCode;
```

Edge cases:
- Not-found: log missing table codes for configuration remediation.
- Delete-flag: if `XFDLDS` indicates a deleted/inactive row, treat as not found.
- Derived flags: map table values to booleans or enums in the Java domain model.

---

### (8) XML Routing Enrichment (Third Enrichment Step)

XFXGETID reads XML routing information from **HXPXMLR/HXFXMLR** to enrich outbound messages.

Algorithm steps:
1. For each eligible transaction, derive a routing key (user + sequence + route ID).
2. Read `HXFXMLR` to retrieve routing information (destination system, queue, correlation IDs).
3. Use this routing data to populate fields in `HXFXMLD` and `HXFXMLH`.

Pseudocode:

```pseudo
routing = SELECT *
          FROM HXFXMLR
          WHERE XMRUSR = :currentUser
            AND XMRSEQ = :seq
            AND XMRID  = :routeId;

IF routingFound THEN
    applyRoutingToXmlHeaders(routing);
ELSE
    flagRoutingMissing(currentUser, seq, routeId);
ENDIF;
```

Edge cases:
- Not-found routing: either drop the message or send to a default error destination.
- Delete-flag logic: if routing entries include active flags, only active routes should be used.
- Derived flags: e.g., `isAsync` vs `isSync` messaging mode based on route configuration.

---

### (9) Counting Rules

Counting rules are mainly embedded in XFXCNTR and HABADTE.

| Counter Name         | Incremented When                         | Description |
|----------------------|-------------------------------------------|-------------|
| totalRecordsRead     | Each read from HAPTRFR succeeds           | Total number of transactions inspected. |
| totalRecordsSkipped  | BR-017/BR-018/BR-019 cause SKIP           | Number of records excluded by business rules. |
| totalXmlMessages     | XML header/detail successfully written    | Number of messages generated for downstream systems. |
| invalidDateCount     | XFXCYMD flags dates outside valid ranges  | Transactions with invalid dates. |

Relationship and meaning:
- `totalRecordsRead = totalRecordsSkipped + totalXmlMessages + otherOutcomeCounts` ensures accounting consistency.
- `invalidDateCount` should be monitored as a data quality metric.

---

### (10) Output Data Structure

HABADTE produces an XML-based output plus internal counters.

#### 10.1 Header Fields

Header information is modeled on `HXFXMLH` (not fully detailed in schema but inferred from usage):

| Field          | Source           | Description                         |
|----------------|------------------|-------------------------------------|
| headerId       | sequence/counter | Unique header identifier.           |
| createdBy      | current user     | Who generated the batch.            |
| createdDate    | system date      | Batch run date.                     |
| levelContext   | level descriptions| Summary of organisational hierarchy.|

#### 10.2 Detail Row Fields

XML detail records in `HXFXMLD` contain per-transaction attributes:

| Field          | Source File | Description |
|----------------|-------------|-------------|
| xmdUsr         | HXPXMLD     | User context for the message. |
| xmdSeq         | HXPXMLD     | Batch sequence number. |
| xmdSq2         | HXPXMLD     | Secondary sequence. |
| xmddta         | HXPXMLD     | Serialized transaction payload, including account, MRN, dates and codes. |

#### 10.3 Footer/Summary Fields

Footer information summarises counts:

| Field              | Description |
|--------------------|-------------|
| totalRecordsRead   | Total input records examined. |
| totalRecordsSkipped| Skipped by exclusion rules. |
| totalXmlMessages   | Messages successfully created. |
| invalidDateCount   | Transactions with invalid dates. |

#### 10.4 Sort Order

Primary sort order in the output is derived from HAPTRFR keys and level hierarchy:
- Within each batch, detail rows are sorted by AFLVL6, AFACCT, AFTRDT, AFTRTM.

---

### (11) Complete Processing Flow (Step-by-Step)

```pseudo
STEP 1: Init
    - Load configuration and copybook structures (HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC).
    - Initialize counters: totalRecordsRead, totalRecordsSkipped, totalXmlMessages, invalidDateCount.

STEP 2: Preferences/Lookup
    - Load level configuration from HXPLVL1–HXPLVL6 via XFXLDSC.
    - Load table dictionary entries from HXPTABLD via XFXTABL.
    - Load XML routing preferences from HXPXMLR/HXFXMLR via XFXGETID.

STEP 3: Context
    - Determine current user identity and security context.
    - Set inpatientOnly, includeVoided, fileStatusFilter from configuration or API inputs.

STEP 4: Query
    - Read HAPTRFR records keyed by AFLVL6, AFACCT, AFTRDT, AFTRTM.
    - For each record:
        - totalRecordsRead++.
        - Apply inclusion/exclusion rules BR-017, BR-018, BR-019.
        - If any rule triggers SKIP, totalRecordsSkipped++ and continue.

STEP 5: Per-Record Enrichment
    5a: Level Enrichment (XFXLDSC)
        - Validate LDAMAP and load level descriptions.
    5b: Table Dictionary Enrichment (XFXTABL)
        - Use indicator *IN79 to decide whether to apply table mappings.
    5c: Date Validation (XFXCYMD/XFXLEAP)
        - Validate transaction date; if invalid, increment invalidDateCount and flag.
    5d: XML Routing (XFXGETID)
        - Determine routing and destination for XML messages.

STEP 6: Assemble Response
    - Build XML header (HXFXMLH) with batch metadata.
    - Build XML detail rows (HXFXMLD) for each enriched transaction.
    - Write header and detail records; increment totalXmlMessages.
    - Commit or rollback based on overall processing outcomes.
```

---

### (12) Data Type Conversions

#### 12.1 Date Fields

AS400 dates such as `AFTRDT` and `WBDATE` are stored as packed decimal (e.g., DECIMAL(8,0) or DECIMAL(7,0)). Conversion rules:
- Interpret them as `YYYYMMDD`.
- Value `0` or `00000000` is treated as `NULL`.

Java/Spring Boot mapping:

```java
LocalDate fromPackedDate(long packed) {
    if (packed == 0L) return null;
    int year = (int)(packed / 10000);
    int month = (int)((packed / 100) % 100);
    int day = (int)(packed % 100);
    return LocalDate.of(year, month, day);
}
```

#### 12.2 Time Fields

Times such as `AFTRTM` use DECIMAL(4,0) in `HHMM` format.

```java
LocalTime fromPackedTime(int packed) {
    if (packed == 0) return null;
    int hour = packed / 100;
    int minute = packed % 100;
    return LocalTime.of(hour, minute);
}
```

#### 12.3 Packed Decimal Keys

Keys like `AFLVL6`, `HX6NUM` are stored as packed decimals but should be represented as numeric types in Java and SQL Server.

```java
long fromPackedKey(BigDecimal packed) {
    return packed.longValue();
}
```

#### 12.4 String Trimming

Fixed-length right-padded strings (e.g., account numbers, names) must be trimmed.

```java
String normalizeString(String raw) {
    return raw == null ? null : raw.trim();
}
```

---

### (13) SQL Server Table Mapping

AS400 objects map to SQL Server tables as follows:

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR      | dbo.HAPTRFR      | Transaction history per account and level. |
| HXPDICT      | dbo.HXPDICT      | Central patient dictionary and reference data. |
| HXPLVL1–6    | dbo.HXPLVL1–6    | Multi-level organisational configuration. |
| HXPTABLD     | dbo.HXPTABLD     | Table dictionary for code/description mappings. |
| HXPXMLD      | dbo.HXPXMLD      | XML detail segments. |
| HXPXMLR      | dbo.HXPXMLR      | XML routing definitions. |
| OAPIRNK      | dbo.OAPIRNK      | Patient ranking data. |
| OMPMAST      | dbo.OMPMAST      | Patient master records. |
| OXPBNFIT     | dbo.OXPBNFIT     | Benefits data. |
| OXPNSTN      | dbo.OXPNSTN      | Notification/statement status. |

#### Suggested SQL Server Indexes

```sql
-- Primary query over transaction file
CREATE INDEX IX_HAPTRFR_LevelAcctDateTime
    ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE);

-- Level configuration lookups
CREATE INDEX IX_HXFLVL1_Key ON dbo.HXFLVL1 (HX1NUM);
CREATE INDEX IX_HXFLVL2_Key ON dbo.HXFLVL2 (HX2NUM);
CREATE INDEX IX_HXFLVL3_Key ON dbo.HXFLVL3 (HX3NUM);
CREATE INDEX IX_HXFLVL4_Key ON dbo.HXFLVL4 (HX4NUM);
CREATE INDEX IX_HXFLVL5_Key ON dbo.HXFLVL5 (HX5NUM);
CREATE INDEX IX_HXFLVL6_Key ON dbo.HXFLVL6 (HX6NUM);

-- Table dictionary
CREATE INDEX IX_HXPTABLD_DataDetail ON dbo.HXPTABLD (XFDTCD, XFDECD);

-- XML routing
CREATE INDEX IX_HXFXMLR_UserSeqId ON dbo.HXFXMLR (XMRUSR, XMRSEQ, XMRID);

-- Patient master and ranking
CREATE INDEX IX_OMPMAST_LevelAcct ON dbo.OMPMAST (MMPLV6, MMACCT);
CREATE INDEX IX_OAPIRNK_LevelAcctSeq ON dbo.OAPIRNK (BRKLV6, BRKACC, BRKSEQ);

-- Benefits and notifications
CREATE INDEX IX_OXPBNFIT_BenefitPlan ON dbo.OXPBNFIT (XFBUBN, XFBPLN);
CREATE INDEX IX_OXPNSTN_LevelStatus ON dbo.OXPNSTN (XFNLV6, XFNSST);
```

---

### (14) Spring Boot API Design

#### 14.1 Recommended REST Endpoint

- Method: `POST`
- Path: `/api/patient-management/habadte/run`

Parameters:

| Name                | Type      | Required | Validation |
|---------------------|-----------|----------|------------|
| level6              | integer   | yes      | > 0 |
| accountNumber       | string    | no       | length 1–12 |
| transactionDateFrom | date      | yes      | not null |
| transactionDateTo   | date      | yes      | >= transactionDateFrom |
| inpatientOnly       | boolean   | no       | default true |
| includeVoided       | boolean   | no       | default false |
| fileStatusFilter    | integer   | no       | >= 0 |

#### 14.2 Layer Structure

- **Controller**: `HabadteController` – handles REST requests, validates parameters.
- **Service**: `HabadteService` – implements business rules BR-017–BR-019 and orchestrates enrichment.
- **Repositories**:
  - `HaptrfrRepository` – transaction data access.
  - `LevelConfigRepository` – HXPLVL1–6.
  - `TableDictionaryRepository` – HXPTABLD.
  - `XmlRoutingRepository` – HXFXMLR/HXPXMLR.

#### 14.3 Response JSON Shape

```json
{
  "runId": "202607020629",
  "level6": 123456,
  "criteria": {
    "inpatientOnly": true,
    "includeVoided": false,
    "fileStatusFilter": 1
  },
  "summary": {
    "totalRecordsRead": 1000,
    "totalRecordsSkipped": 250,
    "totalXmlMessages": 750,
    "invalidDateCount": 10
  },
  "messages": [
    {
      "accountNumber": "000123456789",
      "mrn": "MRN00000001",
      "patientType": "INPATIENT",
      "levelContext": {
        "level1": "Region A",
        "level6": "Ward 12A"
      },
      "xmlRouteId": "ROUTE-01",
      "payload": "<xml>...</xml>"
    }
  ]
}
```

#### 14.4 Java Entity/DTO Sketch

```java
public record HabadteRequest(
    int level6,
    String accountNumber,
    LocalDate transactionDateFrom,
    LocalDate transactionDateTo,
    boolean inpatientOnly,
    boolean includeVoided,
    Integer fileStatusFilter
) {}

public record HabadteSummary(
    long totalRecordsRead,
    long totalRecordsSkipped,
    long totalXmlMessages,
    long invalidDateCount
) {}

public record HabadteMessage(
    String accountNumber,
    String mrn,
    String patientType,
    LevelContext levelContext,
    String xmlRouteId,
    String payload
) {}

public record LevelContext(
    String level1,
    String level2,
    String level3,
    String level4,
    String level5,
    String level6
) {}

public record HabadteResponse(
    String runId,
    int level6,
    HabadteRequest criteria,
    HabadteSummary summary,
    List<HabadteMessage> messages
) {}
```

---

### (15) Performance Considerations

The legacy pattern reads HAPTRFR sequentially and performs multiple per-record lookups (levels, tables, XML routing), which risks N+1 query behavior in a naive migration.

Recommended approach:
- Use bulk queries and joins wherever possible.
- Pre-load level configurations and table dictionaries into memory caches.
- Use set-based operations instead of per-record repository calls.

Example SQL with JOINs:

```sql
SELECT
    t.AFLVL6,
    t.AFACCT,
    t.AFTRDT,
    t.AFTRTM,
    t.AFTYPE,
    l6.HX6NUM,
    l6.HX6DSC,
    d.XFDTCD,
    d.XFDECD,
    r.XMRID
FROM dbo.HAPTRFR t
LEFT JOIN dbo.HXFLVL6 l6 ON l6.HX6NUM = t.AFLVL6
LEFT JOIN dbo.HXPTABLD d ON d.XFDTCD = @dataCode AND d.XFDECD = @detailCode
LEFT JOIN dbo.HXFXMLR r ON r.XMRUSR = @user AND r.XMRSEQ = @seq AND r.XMRID = @routeId
WHERE fileIndicator <> 0
  AND flagIndicator NOT IN ('VOID', 'VOIDED')
  AND patientType <> 'OUTPATIENT';
```

---

### (16) Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to "SKIP". |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to "SKIP". |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to "SKIP". |
| BR-003  | When VYY is less than 1800, branch to "EXIT". |
| BR-004  | When VYY is greater than 2100, branch to "EXIT". |
| BR-005  | When VMM is less than 01, branch to "EXIT". |
| BR-006  | When VMM is greater than 12, branch to "EXIT". |
| BR-007  | When VDD is less than 01, branch to "EXIT". |
| BR-008  | When VDD is greater than DYS(VMM), branch to "EXIT". |
| BR-009  | When LDAMAP is greater than 99, branch to "EXIT". |
| BR-010  | When LDAMAP is greater than 99, branch to "EXIT". |
| BR-011  | When LDAMAP is greater than 99, branch to "EXIT". |
| BR-012  | When LDAMAP is greater than 9999, branch to "EXIT". |
| BR-013  | When *IN79 equals on/active, branch to "EXIT". |
| BR-014  | When *IN79 equals on/active, branch to "EXIT". |
| BR-015  | When *IN79 equals on/active, branch to "EXIT". |
| BR-016  | When *IN79 equals on/active, branch to "EXIT". |
| BR-001  | When X equals zero, branch to "EXIT". |
| BR-002  | When X equals 40, branch to "EXIT". |

---

### (17) Edge Cases to Implement

| Scenario                          | Expected Behavior |
|-----------------------------------|-------------------|
| fileIndicator = 0                 | Skip record; do not produce XML; increment totalRecordsSkipped. |
| flagIndicator = VOID/VOIDED      | Skip record as cancelled; do not bill or notify. |
| patientType = OUTPATIENT         | Skip record for this batch; may be handled by a different process. |
| Invalid date in AFTRDT           | Increment invalidDateCount; either skip or route to data quality queue. |
| Level mapping LDAMAP > 9999      | Do not enrich levels; flag configuration error. |
| Table entry missing in HXPTABLD  | Flag configuration gap; use default codes or bypass enrichment. |
| XML routing missing in HXFXMLR   | Route message to error channel or hold queue. |
| No records meet inclusion rules  | Return empty messages array with summary counts set accordingly. |
| Preferences not configured       | Fall back to sensible defaults (e.g., inpatientOnly=true, includeVoided=false) and log configuration warnings. |
