# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE system generates an XML-based patient transfer report by reading transfer records from HAPTRFR, enriching them with station and organisational hierarchy information, applying inclusion/exclusion rules for voided and outpatient records, and writing output to XML header and detail tables. The target implementation will expose this behaviour as a secured REST API backed by SQL Server while preserving the original business semantics.

## 1. Functional Scope

IN SCOPE business rules (filter logic):
- BR-017: When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter         | AS400 Field / Context | Type        | Description |
|-------------------|-----------------------|-------------|-------------|
| orgLevel6         | AFLVL6                | String(6)   | Level-6 organisational code (e.g., facility or unit). |
| accountNumber     | AFACCT                | String      | Patient account identifier; optional filter when constraining to a single account. |
| fromTransferDate  | AFTRDT                | LocalDate   | Start date for transfer selection (inclusive). |
| toTransferDate    | AFTRDT                | LocalDate   | End date for transfer selection (inclusive). |
| fromTransferTime  | AFTRTM                | LocalTime   | Optional start time within the fromTransferDate. |
| toTransferTime    | AFTRTM                | LocalTime   | Optional end time within the toTransferDate. |
| transferType      | AFTYPE                | String(1)   | Optional transfer type filter (e.g., admission, discharge). |
| inpatientOnly     | INPATIENT_FLAG        | Boolean     | Whether to include only inpatient records. |
| includeVoided     | FLAG_INDICATOR        | Boolean     | Whether to include records marked as voided. |

- The current user principal is supplied via OAuth2 / OpenID Connect and propagated through Spring Security.
- Role-based access control (RBAC) is enforced so that only users with PHI-authorised roles can call the endpoint.
- All calls are logged with a PHI audit trail (user, timestamp, parameters, record counts).

### Spring Boot API Design

- Method: `GET`
- Path: `/api/patient-transfers`

| Parameter         | Type        | Required | Validation |
|-------------------|------------|----------|------------|
| orgLevel6         | String      | Yes      | Not blank; matches known level-6 pattern. |
| accountNumber     | String      | No       | Optional; length and format checks. |
| fromTransferDate  | LocalDate   | Yes      | Must be <= toTransferDate. |
| toTransferDate    | LocalDate   | Yes      | Must be >= fromTransferDate. |
| fromTransferTime  | LocalTime   | No       | Optional; if provided, must be before toTransferTime when same date. |
| toTransferTime    | LocalTime   | No       | Optional. |
| inpatientOnly     | Boolean     | No       | Defaults to true. |
| includeVoided     | Boolean     | No       | Defaults to false. |

Layering:
- Controller: `PatientTransferController`
- Service: `PatientTransferService`
- Repository: `HaptrfrRepository`, `StationRepository`, `HierarchyRepository`, `CodeTableRepository`

Sample response JSON:

```json
{
  "orgLevel": {
    "level6Code": "123456",
    "level6Name": "Unit A",
    "hierarchyPath": ["Region", "Hospital", "Campus", "Building", "Wing", "Unit A"]
  },
  "filters": {
    "fromDate": "2024-01-01",
    "toDate": "2024-01-31",
    "inpatientOnly": true
  },
  "counters": {
    "processedRecords": 120,
    "skippedFileZero": 2,
    "skippedVoided": 3,
    "skippedOutpatient": 15
  },
  "transfers": [
    {
      "accountNumber": "ACCT123",
      "mrn": "MRN001",
      "transferDateTime": "2024-01-02T10:15:00",
      "transferType": "A",
      "station": {
        "code": "ST01",
        "name": "Ward 1"
      }
    }
  ]
}
```

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

## 4. Data Model

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| HAPTRFR     | HAPTRFR          | Core transfer records. |
| HXPLVL1–6   | HXPLVL1–HXPLVL6  | Organisational hierarchy levels. |
| HXPTABLD    | HXPTABLD         | Generic code table definitions. |
| HXPXMLD     | HXPXMLD          | XML detail records. |
| HXPXMLR     | HXPXMLR          | XML header records. |
| OXPNSTN     | OXPNSTN          | Station master data. |
| HAPIRNK     | HAPIRNK          | Rank view over TAPIRNK. |
| HMLMAST5H   | HMLMAST5H        | Master data view over TMPMAST. |

## 5. Acceptance Criteria

| Scenario                       | Expected Behavior |
|--------------------------------|-------------------|
| FILE_INDICATOR = 0            | Do not output XML for this record; increment skippedFileZero. |
| FLAG_INDICATOR = 'VOID'       | Do not output XML; increment skippedVoided. |
| INPATIENT_OUTPATIENT_FLAG = 'O'| Exclude outpatient records; increment skippedOutpatient. |
| Station not found             | Output record with default "Unknown Station" label. |
| Hierarchy level invalid       | Apply BR-009–BR-012; use generic labels or omit hierarchy. |
| No records in date range      | Produce header with zero counters and no detail records. |
| Voided-only dataset           | All records skipped; counters reflect skippedVoided count. |

> Full detail: see Business_Processing_Rules_HABADTE.md
