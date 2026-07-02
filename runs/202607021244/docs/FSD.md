# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE program implements a patient transfer and XML export workflow that reads transfer records, applies inclusion/exclusion filters, enriches records from configuration tables, and writes XML header and detail records used by downstream systems. This FSD distils the essential business behaviour and API contract required to reproduce the AS400 process in a modern Java / Spring Boot / SQL Server stack.

## 1. Functional Scope

The following filter rules are IN SCOPE for this migration:

- **BR-017** – Skip records where file indicator = 0.
- **BR-018** – Skip records where flag indicator denotes void/voided status.
- **BR-019** – Skip records where inpatient/outpatient flag indicates outpatient.

These rules together define the eligible subset of patient transfer records that the service will process and export.

## 2. API Contract

### 2.1 Inputs (API Request Parameters)

From Business_Processing_Rules_HABADTE.md section (2):

| Parameter                    | AS400 Field / Source | Type        | Description |
|-----------------------------|-----------------------|------------|-------------|
| runDate                     | VYY/VMM/VDD (XFXCYMD) | string (YYYY-MM-DD) | Processing date used to validate transfer date fields before querying. |
| inpatientOnly               | -INPATIENT/OUTPATIENT FLAG (HABADTE) | boolean     | When true, include only inpatient transfers; enforced by BR-019. |
| includeVoided               | -FLAG INDICATOR (HABADTE)           | boolean     | When false, skip records marked void/voided; enforced by BR-018. |
| requireActiveFileIndicator  | -FILE INDICATOR (HABADTE)           | boolean     | When true, skip records with file indicator = 0; enforced by BR-017. |
| levelMapCode                | LDAMAP (XFXLDSC)                    | integer     | Optional mapping code constraint; values above configured thresholds cause exits (BR-009–BR-012). |
| tableIndicator79            | *IN79 (XFXTABL)                     | boolean     | Internal flag reflecting table-driven conditions; when active, triggers exits (BR-013–BR-016). |

Security mapping notes:
- Use OAuth2/OIDC with RBAC scopes (`patient.transfer.read`, `patient.transfer.export`).
- Log all PHI access for AFACCT, AFMRNO, MMACCT, MRN fields.

### 2.2 Spring Boot API Design

From Business_Processing_Rules_HABADTE.md section (14):

- HTTP method: `GET`
- Path: `/api/patient-transfers/export`

Parameters:

| Name            | Type    | Required | Validation                   |
|----------------|--------|---------|------------------------------|
| runDate        | String | Yes     | ISO-8601 date, not in future.|
| inpatientOnly  | Boolean| No      | Default `true`.              |
| includeVoided  | Boolean| No      | Default `false`.             |
| levelMapCode   | Integer| No      | Must be >=0 and <=9999.      |

Layer structure:
- Controller: `PatientTransferExportController`
- Service: `PatientTransferExportService`
- Repositories: `HaptrfrRepository`, `LevelConfigRepository`, `TableMappingRepository`, `XmlRepository`

Response JSON shape:

```json
{
  "runDate": "2026-07-02",
  "processedCount": 120,
  "skippedCount": 15,
  "errorCount": 3,
  "transfers": [
    {
      "transferKey": {
        "level6": 6,
        "account": "A12345",
        "date": "2026-07-01",
        "time": "14:30",
        "type": "ADMIT"
      },
      "patientMrn": "MRN0001",
      "inpatient": true,
      "voided": false,
      "levelDesc": "L1-L2-L3-L4-L5-L6",
      "mappingDesc": "BedTransfer",
      "xmlDetailId": "XMLD-000001"
    }
  ]
}
```

Java entity/DTO sketch:

```java
public record TransferKey(
    int level6,
    String account,
    LocalDate date,
    LocalTime time,
    String type
) {}

public record PatientTransferDto(
    TransferKey transferKey,
    String patientMrn,
    boolean inpatient,
    boolean voided,
    String levelDesc,
    String mappingDesc,
    String xmlDetailId
) {}

public record PatientTransferExportResponse(
    LocalDate runDate,
    int processedCount,
    int skippedCount,
    int errorCount,
    List<PatientTransferDto> transfers
) {}
```

## 3. Business Rules (Summary)

From Business_Processing_Rules_HABADTE.md section (16):

| Rule ID | Description |
|---------|-------------|
| BR-017  | Skip records where file indicator = 0. |
| BR-018  | Skip records where flag indicator denotes void/voided status. |
| BR-019  | Skip records where inpatient/outpatient flag indicates outpatient. |
| BR-001  | Exit control routine when X = 0. |
| BR-002  | Exit control routine when X = 40. |
| BR-003  | Exit date validation when year < 1800. |
| BR-004  | Exit date validation when year > 2100. |
| BR-005  | Exit date validation when month < 1. |
| BR-006  | Exit date validation when month > 12. |
| BR-007  | Exit date validation when day < 1. |
| BR-008  | Exit date validation when day exceeds days-in-month. |
| BR-009  | Exit level description when LDAMAP > 99. |
| BR-010  | Exit level description when LDAMAP > 99. |
| BR-011  | Exit level description when LDAMAP > 99. |
| BR-012  | Exit level description when LDAMAP > 9999. |
| BR-013  | Exit table routine when indicator *IN79 is ON. |
| BR-014  | Exit table routine when indicator *IN79 is ON. |
| BR-015  | Exit table routine when indicator *IN79 is ON. |
| BR-016  | Exit table routine when indicator *IN79 is ON. |
| BR-020  | SQL profile program accesses table HXPAPPPRF. |

## 4. Data Model

From Business_Processing_Rules_HABADTE.md section (13):

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| HAPTRFR     | dbo.HAPTRFR      | Patient transfer records (primary entity). |
| HXPDICT     | dbo.HXPDICT      | Patient dictionary/master data with PHI fields. |
| HXPLVL1–6   | dbo.HXPLVL1–6    | Level configuration hierarchy. |
| HXPTABLD    | dbo.HXPTABLD     | Code/description mapping table. |
| HXPXMLD     | dbo.HXPXMLD      | XML detail control table. |
| HXPXMLR     | dbo.HXPXMLR      | XML sequence/identity table. |
| OAPIRNK     | dbo.OAPIRNK      | Inquiry records with MRN. |
| OMPMAST     | dbo.OMPMAST      | Patient master with MRN, account, name, SSN. |
| OXPBNFIT    | dbo.OXPBNFIT     | Benefit master records. |
| OXPNSTN     | dbo.OXPNSTN      | Station/status records. |

Suggested SQL Server indexes:

```sql
CREATE INDEX IX_HAPTRFR_RunDate
ON dbo.HAPTRFR (AFTRDT, AFTRTM, AFLVL6, AFACCT, AFTYPE);

CREATE INDEX IX_HXPLVL1_Key ON dbo.HXPLVL1 (HX1NUM);
CREATE INDEX IX_HXPLVL2_Key ON dbo.HXPLVL2 (HX2NUM);
CREATE INDEX IX_HXPLVL3_Key ON dbo.HXPLVL3 (HX3NUM);
CREATE INDEX IX_HXPLVL4_Key ON dbo.HXPLVL4 (HX4NUM);
CREATE INDEX IX_HXPLVL5_Key ON dbo.HXPLVL5 (HX5NUM);
CREATE INDEX IX_HXPLVL6_Key ON dbo.HXPLVL6 (HX6NUM);

CREATE INDEX IX_HXPTABLD_Code
ON dbo.HXPTABLD (XFDTCD, XFDECD);

CREATE INDEX IX_HXPXMLR_UserSeq
ON dbo.HXPXMLR (XMRUSR, XMRSEQ);

CREATE INDEX IX_OMPMAST_Account
ON dbo.OMPMAST (MMACCT);

CREATE INDEX IX_OMPMAST_MRN
ON dbo.OMPMAST (MMMRNO, MMMMRN);
```

## 5. Acceptance Criteria

From Business_Processing_Rules_HABADTE.md section (17):

| Scenario                                | Expected Behavior |
|----------------------------------------|-------------------|
| AFTRDT = 0 or AFTRTM = 0               | Treat as null; exclude record from main report or flag as error. |
| Date components invalid (XFXCYMD rules)| Do not process transfer; increment errorCount. |
| LDAMAP > 99 or > 9999                  | Skip level enrichment; proceed with transfer flagged as `mappingInvalid`. |
| Missing level records in HXPLVL*       | Use partial levelDesc; log warning. |
| *IN79 = ON in table logic              | Skip table enrichment for that record; possibly skip record depending on business decision. |
| No XML ID found in HXPXMLR/HXFXMLR     | Initialise new XML sequence; create header record before details. |
| Failed write to HXFXMLD/HXFXMLH        | Increment errorCount; log error; do not retry automatically. |
| No transfers match inclusion filters   | Return empty `transfers` array with processedCount = 0; HTTP 200 response. |
| Preferences not configured (missing mapping) | Mark records as `requiresManualReview`; do not block processing. |

> Full detail: see Business_Processing_Rules_HABADTE.md
