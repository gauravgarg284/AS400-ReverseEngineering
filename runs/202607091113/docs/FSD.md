# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

HABADTE is a batch-oriented transfer processing program in the PATIENT_MANAGEMENT domain. It reads transfer records from HAPTRFR, applies a set of high-confidence business rules to filter and classify them, enriches each record with level, benefit, station, and table-driven attributes, and then writes XML header and detail records for downstream systems. The modern implementation will expose this functionality as a secured Spring Boot service that writes to SQL Server tables mirroring the HXFXMLH/HXFXMLD structures.

## 1. Functional Scope

IN SCOPE business rules (filter rules for HABADTE):

- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

These rules determine which transfers are eligible for XML export.

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter | AS400 Field / Concept | Type | Description |
|-----------|------------------------|------|-------------|
| runDate | AFTRDT (HAPTRFR) | date | Processing date to limit transfer selection. |
| runTimeFrom | AFTRTM (HAPTRFR) | time | Lower bound for transfer time in the batch window. |
| runTimeTo | AFTRTM (HAPTRFR) | time | Upper bound for transfer time in the batch window. |
| level6 | AFLVL6 (HAPTRFR) | string | Level-6 key segment used to filter transfers (e.g., plan/network level). |
| accountNumber | AFACCT (HAPTRFR) | string | Optional filter to restrict processing to a single account. |
| inpatientOnly | INP/OUT flag | boolean | When true, restricts output to inpatient transfers only. |
| includeVoided | FLAG indicator | boolean | When false, void/voided transfers are excluded. |
| maxRecords | derived | integer | Optional limit on number of transfers processed per run. |

Additional implicit inputs:

- Current user identity from the security context, mapped to XML header fields and audit logs.

Security mapping for the modernized service:

- REST endpoint secured with OAuth2/JWT.
- RBAC to ensure only authorized service accounts can trigger the batch.
- PHI access and batch statistics written to an audit trail.

### Spring Boot API Design

- Method: `POST`
- Path: `/api/batch/habadte/transfers`

| Parameter | Type | Required | Validation |
|-----------|------|----------|-----------|
| runDate | LocalDate | yes | Must be >= 1900-01-01; cannot be beyond 2100-12-31. |
| runTimeFrom | LocalTime | yes | Must be <= runTimeTo. |
| runTimeTo | LocalTime | yes | Must be >= runTimeFrom. |
| level6 | String | no | Trimmed; max length as per AFLVL6. |
| accountNumber | String | no | Trimmed; must satisfy account format. |
| inpatientOnly | Boolean | no | Default true. |
| includeVoided | Boolean | no | Default false. |
| maxRecords | Integer | no | Positive; upper bound enforced (e.g., 100000). |

Response JSON shape:

```json
{
  "runId": "202607091113",
  "runDate": "2026-07-09",
  "parameters": {
    "level6": "...",
    "accountNumber": "...",
    "inpatientOnly": true,
    "includeVoided": false
  },
  "counters": {
    "totalTransfersRead": 1000,
    "totalProcessed": 750,
    "skippedFileIndicator": 50,
    "skippedVoided": 100,
    "skippedOutpatient": 100
  },
  "status": "COMPLETED",
  "xmlBatchId": "XMR123456",
  "messages": [
    "XML generation completed.",
    "Printer output written."
  ]
}
```

Layer structure:

- Controller: `HabadteBatchController`
- Service: `HabadteBatchService`
- Repositories: `HaptrfrRepository`, `HxplvlRepository`, `HxptabldRepository`, `TxpbnfitRepository`, `TxpnstnRepository`, `HxxmlRepository`

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |
| BR-001 | Field formatting: exit if all blank (40 chars). |
| BR-002 | Field formatting: exit if first char non-blank. |
| BR-003 | Date validation: reject year < 1800 (historical minimum). |
| BR-004 | Date validation: reject year > 2100 (forecast maximum). |
| BR-005 | Date validation: reject month < 01 (calendar constraint). |
| BR-006 | Date validation: reject month > 12 (calendar constraint). |
| BR-007 | Date validation: reject day < 01 (calendar constraint). |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |
| BR-009 | Level lookup: reject if level code exceeds valid range. |
| BR-010 | Level lookup: reject if level code exceeds valid range. |
| BR-011 | Level lookup: reject if level code exceeds valid range. |
| BR-012 | Level lookup: reject if level code exceeds valid range. |
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-020 | SQL program accesses table 'HXPAPPPRF'. |

## 4. Data Model

### SQL Server Table Mapping

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR | dbo.HAPTRFR | Primary transfer data source. |
| HXPLVL1 | dbo.HXPLVL1 | Level 1 master data. |
| HXPLVL2 | dbo.HXPLVL2 | Level 2 master data. |
| HXPLVL3 | dbo.HXPLVL3 | Level 3 master data. |
| HXPLVL4 | dbo.HXPLVL4 | Level 4 master data. |
| HXPLVL5 | dbo.HXPLVL5 | Level 5 master data. |
| HXPLVL6 | dbo.HXPLVL6 | Level 6 master data. |
| HXPTABLD | dbo.HXPTABLD | Table dictionary for code mappings. |
| HXPBNFIT/TXPBNFIT | dbo.TXPBNFIT | Benefit plan definitions. |
| HXPNSTN/TXPNSTN | dbo.TXPNSTN | Station/state definitions. |
| HXPXMLD/HXFXMLD | dbo.HXFXMLD | XML detail output. |
| HXPXMLR/HXFXMLR | dbo.HXFXMLR | XML request/identifier store. |

Suggested indexes:

```sql
CREATE INDEX IX_HAPTRFR_RunWindow
ON dbo.HAPTRFR (AFTRDT, AFTRTM, AFLVL6, AFACCT, AFTYPE);

CREATE INDEX IX_HXPLVL6_Key
ON dbo.HXPLVL6 (HX6NUM);

CREATE INDEX IX_HXPTABLD_Code
ON dbo.HXPTABLD (XFDTCD, XFDECD);

CREATE INDEX IX_TXPBNFIT_Key
ON dbo.TXPBNFIT (XFBUBN, XFBPLN);

CREATE INDEX IX_TXPNSTN_Key
ON dbo.TXPNSTN (XFNLV6, XFNSST);

CREATE INDEX IX_HXFXMLD_UserSeq
ON dbo.HXFXMLD (XMDUSR, XMDSEQ, XMDSQ2);
```

## 5. Acceptance Criteria

| Scenario | Expected Behavior |
|----------|-------------------|
| File indicator = 0 | Apply BR-017; skip record and increment `skippedFileIndicator`. |
| Flag indicator = void/voided | Apply BR-018; skip record and increment `skippedVoided`. |
| In/out flag = outpatient | Apply BR-019; skip record and increment `skippedOutpatient`. |
| Invalid level code | Apply BR-009..BR-012; mark record as invalid and either skip or route to error bucket based on configuration. |
| Missing benefit record | Write record with default benefit description or skip; log warning. |
| Missing station record | Write record with default station description or skip; log warning. |
| XML ID collision | Regenerate or increment XML ID via XFXGETID before writing HXFXMLD. |
| HXFXMLD write failure | Roll back header state or mark batch as FAILED; report via API response. |
| Date field = 0 | Treat as null date in Java and SQL Server; do not fail the batch. |
| Time field = 0 | Treat as null time; avoid populating invalid times in output. |
| No transfers found | Return COMPLETED status with zero counts; no XML written. |
| Preferences not configured | Fail batch with configuration error; log details and expose error message in API.

> Full detail: see Business_Processing_Rules_HABADTE.md
