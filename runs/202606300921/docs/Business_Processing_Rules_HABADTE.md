# Business Processing Rules and Functional Specification

## HABADTE (HABADTE) 
### For Java / Spring Boot / SQL Server Migration

## 1. Business Purpose

The HABADTE batch program processes patient account records to determine which should be included in downstream reporting and XML-based interfaces. It evaluates file indicators, void flags, and inpatient/outpatient status, enriches data from master, benefit, institution, and dictionary tables, and produces XML and printer output. The primary users are back-office operations and clinical finance teams, and the process typically runs as a scheduled batch job.

## 2. Inputs / API Request Parameters

In the migrated Spring Boot service, HABADTE will be exposed as a REST endpoint. Typical inputs include run context and optional filters.

| Parameter              | AS400 Field | Type        | Description |
|------------------------|------------|-------------|-------------|
| runDate                | *JOBDATE   | LocalDate   | Date for which the batch is executed. |
| institutionCode        | INSTCD     | String      | Optional filter limiting processing to a specific institution (OXPNSTN). |
| inpatientOnly          | MMIORO     | Boolean     | When true, only inpatients are included (maps to MMIO RO = 'I' vs 'O'). |
| maxRecords             | N/A        | Integer     | Optional cap on number of records processed in a single run. |
| preferenceProfileId    | PREFID     | String      | Preference profile key used by HXXAPPPRF. |

Spring Boot security mapping:

- Endpoint protected by role-based access (e.g., ROLE_HABADTE_BATCH).
- Access restricted to service accounts; PHI fields returned only to authorized clients.

## 3. Organizational Hierarchy

HABADTE uses HXPLVL1–HXPLVL6 to model a six-level organizational hierarchy.

| Level | Name            | Key Size | Table   |
|-------|-----------------|----------|---------|
| 1     | Enterprise      | 10       | HXPLVL1 |
| 2     | Region          | 10       | HXPLVL2 |
| 3     | Network         | 10       | HXPLVL3 |
| 4     | Facility Group  | 10       | HXPLVL4 |
| 5     | Facility        | 10       | HXPLVL5 |
| 6     | Department/Unit | 10       | HXPLVL6 |

## 4. Primary Data Source

Primary SQL Server table: HABADTE_PatientMaster (migration of OMPMAST).

Access pattern:

- Read-only scan or keyed access using MRN/account.
- Joined to rank, benefit, institution, and dictionary tables for enrichment.

Example SQL Server SELECT with ORDER BY:

```sql
SELECT
    p.MMMRNO,
    p.MMACCT,
    p.MMNAME,
    p.MMPSSN,
    p.MMPFIL,
    p.MMPFLG,
    p.MMIO RO,
    r.RANK,
    b.BenefitCode,
    i.InstitutionName
FROM HABADTE_PatientMaster p
LEFT JOIN HABADTE_AccountRank r ON r.Account = p.MMACCT
LEFT JOIN HABADTE_Benefit b ON b.Account = p.MMACCT
LEFT JOIN HABADTE_Institution i ON i.InstitutionCode = p.InstitutionCode
WHERE p.RunDate = @runDate
ORDER BY i.InstitutionCode, p.MMMRNO;
```

Key fields table:

| Field  | Description |
|--------|-------------|
| MMMRNO | Medical Record Number (primary clinical key). |
| MMACCT | Account number (financial key). |
| MMPFIL | File indicator controlling inclusion (BR-017). |
| MMPFLG | Void flag (BR-018). |
| MMIORO | Inpatient/Outpatient flag (BR-019). |

## 5. Inclusion and Exclusion Rules

### 5.1 BR-017 — File Indicator Validation

- BR-ID: BR-017
- Description: Exclude records where the file indicator is zero.
- Pseudocode IF:

```pseudo
IF MMPFIL = 0
    SKIP record
ENDIF
```

- SQL WHERE fragment:

```sql
MMPFIL <> 0
```

### 5.2 BR-018 — Void Flag Filter

- BR-ID: BR-018
- Description: Exclude records marked as void.
- Pseudocode IF:

```pseudo
IF MMPFLG = 'V'
    SKIP record
ENDIF
```

- SQL WHERE fragment:

```sql
MMPFLG <> 'V'
```

### 5.3 BR-019 — Inpatient/Outpatient Filter

- BR-ID: BR-019
- Description: Exclude outpatient records when HABADTE processes inpatient-only data.
- Pseudocode IF:

```pseudo
IF MMIORO = 'O'
    SKIP record
ENDIF
```

- SQL WHERE fragment:

```sql
MMIORO <> 'O'
```

### 5.4 Combined Summary WHERE Clause

```sql
WHERE
    p.MMPFIL <> 0      -- BR-017: only records with valid file indicator
AND p.MMPFLG <> 'V'    -- BR-018: exclude voided records
AND p.MMIO RO <> 'O'   -- BR-019: exclude outpatients in this batch
```

## 6. Data Enrichment Steps

HABADTE enriches each base record via secondary lookups.

### 6.1 Rank Enrichment (Account Rank)

- BR-ID: (implicit)
- Description: Attach rank/priority from HABADTE_AccountRank.
- Key Fields: MMACCT.
- SELECT query:

```sql
SELECT r.RankCode, r.RankDescription
FROM HABADTE_AccountRank r
WHERE r.Account = @MMACCT
  AND r.DeleteFlag = 0
  AND @runDate BETWEEN r.EffectiveDate AND r.ExpiryDate;
```

- Delete-flag/effective-date logic: Only active rank entries are used.

### 6.2 Benefit Enrichment

- BR-ID: (implicit)
- Description: Attach benefit information for the account.
- Key Fields: MMACCT.

```sql
SELECT b.BenefitCode, b.BenefitDescription, b.XFBTEL
FROM HABADTE_Benefit b
WHERE b.Account = @MMACCT
  AND b.DeleteFlag = 0
  AND @runDate BETWEEN b.EffectiveDate AND b.ExpiryDate;
```

### 6.3 Institution Enrichment

- BR-ID: (implicit)
- Description: Attach institution details.
- Key Fields: InstitutionCode.

```sql
SELECT i.InstitutionName, i.RegionCode
FROM HABADTE_Institution i
WHERE i.InstitutionCode = @InstitutionCode;
```

### 6.4 Dictionary Enrichment (MRN/Phone/Name)

- BR-ID: (implicit)
- Description: Map MRN to cross-reference identifiers and contact details.
- Key Fields: MMMRNO.

```sql
SELECT d.CCMRNO, d.HXRMNO, d.XCNAME, d.XFBTEL
FROM HABADTE_Dictionary d
WHERE d.CCMRNO = @MMMRNO
  AND d.DeleteFlag = 0;
```

Derived flags:

- hasContactPhone = (XFBTEL IS NOT NULL).
- hasCrossMRN = (HXRMNO IS NOT NULL).

## 7. Counting and Aggregation Rules

Counters are implemented via XFXCNTR and HABADTE logic.

| Counter                | Incremented When                             | Description |
|------------------------|-----------------------------------------------|-------------|
| totalRecordsRead       | For each record read from HABADTE_PatientMaster. | Total records scanned. |
| totalRecordsProcessed  | For each record that passes BR-017–BR-019.   | Records included in output. |
| totalVoidedRecords     | For each record with MMPFLG = 'V'.           | Records excluded due to void flag. |
| totalOutpatientRecords | For each record with MMIORO = 'O'.           | Outpatient records excluded. |
| totalInvalidFileInd    | For each record with MMPFIL = 0.             | Records excluded due to invalid file indicator. |

## 8. Output Data Structure

### 8.1 Header Fields

| Field           | Source            | Type      | Description |
|-----------------|-------------------|-----------|-------------|
| runDate         | Input             | LocalDate | Batch run date. |
| institutionCode | Input / OXPNSTN   | String    | Institution being processed. |
| totalRecords    | Aggregation       | Integer   | Total records read. |
| totalProcessed  | Aggregation       | Integer   | Total records processed. |

### 8.2 Detail Row Fields

| Field           | Source                | Type    | Description |
|-----------------|-----------------------|---------|-------------|
| mrn             | OMPMAST.MMMRNO        | String  | Medical Record Number. |
| account         | OMPMAST.MMACCT        | String  | Account number. |
| patientName     | OMPMAST.MMNAME        | String  | Patient name. |
| ssn             | OMPMAST.MMPSSN        | String  | SSN / national ID. |
| rankCode        | AccountRank.RankCode  | String  | Rank/priority code. |
| benefitCode     | Benefit.BenefitCode   | String  | Benefit code. |
| institutionName | InstitutionName       | String  | Institution name. |
| phone           | Dictionary.XFBTEL     | String  | Contact phone number. |
| flags           | HABADTE flags         | String  | Combined status flags. |

### 8.3 Summary/Footer Fields

| Field                | Source        | Type    | Description |
|----------------------|--------------|---------|-------------|
| totalVoidedRecords   | Aggregation  | Integer | Count of voided records. |
| totalOutpatient      | Aggregation  | Integer | Count of outpatient records skipped. |
| totalInvalidFileInd  | Aggregation  | Integer | Count of records with invalid file indicator. |

Sort order:

- Primary: institutionCode.
- Secondary: mrn.

## 9. Complete Processing Flow

Numbered steps aligned with Spring Boot service methods:

1. STEP 1 Init: Validate input parameters, load run context, and initialize counters.
2. STEP 2 Preferences: Call PreferenceService (migration of HXXAPPPRF) to load application preferences and thresholds.
3. STEP 3 Context: Resolve organizational hierarchy via OrgHierarchyService (HXXLEVEL + HXPLVL1–6) and populate context.
4. STEP 4 Query: Execute primary query against HABADTE_PatientMaster and related tables.
5. STEP 5 Per-Record Enrichment: For each record, apply BR-017–BR-019 filters, then enrich data via rank, benefit, institution, and dictionary lookups.
6. STEP 6 Assemble Response: Build XML and/or JSON response payloads and printer/report output; update counters and footer fields.

## 10. Data Type Conversions

- Date fields: DECIMAL(8,0) representing YYYYMMDD converted to Java `LocalDate` using `DateUtil` (migration of XFXCYMD).
- Time fields: HHMM integer or CHAR(4) converted to `LocalTime`.
- Packed decimal keys: Converted to `Long` or `BIGINT` in SQL Server.
- Strings: Trimmed of trailing spaces; left-padded numeric strings normalized.

## 11. SQL Server Table Mapping

| AS400 Object | SQL Server Table        | Used For |
|--------------|-------------------------|----------|
| OMPMAST      | HABADTE_PatientMaster   | Primary patient/account data. |
| HAPTRFR      | HABADTE_Transfer        | Transfer/transaction details. |
| OAPIRNK      | HABADTE_AccountRank     | Rank/priority information. |
| OXPBNFIT     | HABADTE_Benefit         | Benefit definitions and contact data. |
| OXPNSTN      | HABADTE_Institution     | Institution/location master. |
| HXPDICT      | HABADTE_Dictionary      | MRN/phone/name cross-reference. |
| HXPTABLD     | HABADTE_LookupTableDef  | Table-driven configuration. |

Suggested covering indexes:

- IX_PatientMaster_RunDate_Inst_MRN(MMRunDate, InstitutionCode, MMMRNO).
- IX_AccountRank_Account(Account).
- IX_Benefit_Account(Account).
- IX_Dictionary_MRN(CCMRNO).

## 12. Spring Boot API Design

### 12.1 REST Endpoint

- `GET /api/habadte/batch`

Query parameters:

- `runDate` (required)
- `institutionCode` (optional)
- `inpatientOnly` (optional, default true)
- `maxRecords` (optional)

### 12.2 Layers

- Controller: `HabadteBatchController` — validates inputs and delegates to service.
- Service: `HabadteBatchService` — implements steps 1–6 and calls repositories.
- Repositories: `PatientMasterRepository`, `AccountRankRepository`, `BenefitRepository`, `InstitutionRepository`, `DictionaryRepository`.

### 12.3 Response JSON Sample

```json
{
  "runDate": "2026-06-30",
  "institutionCode": "INST01",
  "totalRecords": 1200,
  "totalProcessed": 850,
  "details": [
    {
      "mrn": "0001234567",
      "account": "ACCT0001",
      "patientName": "DOE, JOHN",
      "rankCode": "R1",
      "benefitCode": "BEN001",
      "institutionName": "General Hospital",
      "phone": "555-123-4567",
      "flags": "IP-ACT"
    }
  ],
  "summary": {
    "totalVoidedRecords": 50,
    "totalOutpatient": 300,
    "totalInvalidFileInd": 0
  }
}
```

### 12.4 Entity/DTO Sketches

- `PatientRecord` entity mapping HABADTE_PatientMaster.
- `HabadteDetailDTO` for per-record output.
- `HabadteBatchResponseDTO` for overall response.

## 13. Performance Considerations

- Avoid N+1 queries: Instead of per-record lookups for rank, benefit, institution, and dictionary, use set-based joins as shown in the primary query.
- Use batch processing and streaming to handle large result sets.
- Cache lookup tables (HXPTABLD-derived) in memory where appropriate.

Recommended JOIN query replacing per-record loops:

```sql
SELECT
    p.*, r.*, b.*, i.*, d.*
FROM HABADTE_PatientMaster p
LEFT JOIN HABADTE_AccountRank r ON r.Account = p.MMACCT
LEFT JOIN HABADTE_Benefit b ON b.Account = p.MMACCT
LEFT JOIN HABADTE_Institution i ON i.InstitutionCode = p.InstitutionCode
LEFT JOIN HABADTE_Dictionary d ON d.CCMRNO = p.MMMRNO
WHERE p.RunDate = @runDate
  AND p.MMPFIL <> 0
  AND p.MMPFLG <> 'V'
  AND p.MMIO RO <> 'O';
```

## 14. Business Rules Reference Summary

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

## 15. Edge Cases

| Scenario                         | Expected Behavior |
|----------------------------------|-------------------|
| MMPFIL = 0                       | Record is skipped; counters updated for invalid file indicator. |
| MMPFLG = 'V'                     | Record is skipped; counters updated for voided record. |
| MMIORO = 'O'                     | Record is skipped; counters updated for outpatient record. |
| MRN not found in Dictionary      | Enrichment fields (XCNAME, XFBTEL, HXRMNO) left null; record still processed. |
| Benefit or rank not found        | Benefit/rank fields left null; record still processed. |
| Institution code not found       | InstitutionName left null; record still processed but flagged for review. |
| Empty result set                 | Response contains header with zero counts and empty details array. |
