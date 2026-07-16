# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE program implements an inpatient census workflow that selects transfer records from HAPTRFR, applies strict inclusion/exclusion rules (pre‑admission, voided accounts, outpatients), enriches records with organisational, status, and benefit data, and finally produces XML and printer output. The modernized solution must reproduce this behaviour as a secure, auditable REST API backed by SQL Server tables.

## 1. Functional Scope

IN SCOPE business rules:

- **BR-017** – Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census.
- **BR-018** – Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census.
- **BR-019** – Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.

## 2. API Contract

(From Business Processing Rules document, section (2) Inputs and section (14) Spring Boot API Design.)

### Inputs (API Request Parameters)

| Parameter            | AS400 Field / Concept | Type        | Description                                                        |
|----------------------|-----------------------|------------|--------------------------------------------------------------------|
| `censusDate`         | AFTRDT                | `LocalDate`| Transfer date used to select eligible census records.              |
| `censusTime`         | AFTRTM                | `LocalTime`| Transfer time used with date to determine current status.          |
| `level6Code`         | AFLVL6                | `String`   | Organisational level‑6 code identifying the unit or facility.      |
| `includeVoided`      | Voided flag           | `boolean`  | If `true`, includes voided accounts; default is `false` (BR‑018).  |
| `includeOutpatient`  | I/O indicator         | `boolean`  | If `true`, includes outpatient accounts; default `false` (BR‑019). |
| `includePreAdmission`| File indicator        | `boolean`  | If `true`, includes pre‑admission (indicator=0); default `false`.  |
| `maxRecords`         | Internal counter limit| `int`      | Soft cap on the number of records returned (performance safety).   |

Spring Boot security mapping notes:
- Use OAuth2 / OpenID Connect for authentication, with roles mapped to organisational functions (e.g., `ROLE_CENSUS_ADMIN`, `ROLE_BED_MGMT`).
- Enforce RBAC on endpoints that expose PHI from HAPTRFR, OMPMAST, and HXPDICT.
- Persist audit events for every request that touches PHI‑bearing PFs.

### Spring Boot API Design

**Endpoint:** `GET /api/census`

| Parameter            | Type        | Required | Validation                                   |
|----------------------|------------|---------:|----------------------------------------------|
| `censusDate`         | `LocalDate`| yes      | Must be between 1800-01-01 and 2100-12-31.   |
| `censusTime`         | `LocalTime`| yes      | Must be a valid time.                        |
| `level6Code`         | `String`   | yes      | Non-empty; must match existing HXPLVL6 code. |
| `includeVoided`      | `boolean`  | no       | Defaults to `false`.                         |
| `includeOutpatient`  | `boolean`  | no       | Defaults to `false`.                         |
| `includePreAdmission`| `boolean`  | no       | Defaults to `false`.                         |
| `maxRecords`         | `int`      | no       | Range 1–10000; default 1000.                 |

Layer structure:
- Controller: `CensusController`
- Service: `CensusService` (+ enrichment services)
- Repositories: `HaptrfrRepository`, `TxpnstnRepository`, `TxpbnfitRepository`, `HxplvlRepository`, `XmlHeaderRepository`, `XmlDetailRepository`.

Response JSON shape:

```json
{
  "runId": "202607130251-001",
  "censusDateTime": "2026-07-13T02:51:00Z",
  "levelPath": "System > Region > Facility > Ward > Station",
  "summary": {
    "totalCandidates": 120,
    "excludedPreAdmission": 5,
    "excludedVoided": 3,
    "excludedOutpatient": 12,
    "finalCensusCount": 100
  },
  "rows": [
    {
      "level6Code": "123456",
      "accountNumber": "A00001234",
      "medicalRecordNumber": "MRN000987",
      "transferDate": "2026-07-12",
      "transferTime": "14:30",
      "transferType": "ADM",
      "statusCode": "ACTIVE",
      "stationDescription": "MEDICAL SURGICAL",
      "benefitNumber": "BN123",
      "planCode": "PPO",
      "coverageTelephone": "555-123-4567"
    }
  ]
}
```

## 3. Business Rules (Summary)

| Rule ID | Description                                                                                                    |
|--------|----------------------------------------------------------------------------------------------------------------|
| BR-017 | Exclude pre-admissions (file indicator = 0) from active census.                                               |
| BR-018 | Exclude voided accounts (flag = 'V') from active census.                                                      |
| BR-019 | Exclude outpatient accounts (I/O indicator = 'O') from census.                                               |
| BR-001 | Skip text centering when the input field is blank.                                                            |
| BR-002 | Skip text centering when text is already left-aligned at position 1.                                          |
| BR-003 | Reject dates before 1800.                                                                                    |
| BR-004 | Reject dates beyond 2100.                                                                                    |
| BR-005 | Enforce month >= 01.                                                                                          |
| BR-006 | Enforce month <= 12.                                                                                          |
| BR-007 | Enforce day >= 01.                                                                                            |
| BR-008 | When VDD > DYS(VMM), branch to EXIT (date overflow rule).                                                     |
| BR-009 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-010 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-011 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-012 | Organisational level lookup: out-of-range level code returns no description.                                  |
| BR-013 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-014 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-015 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-016 | When indicator *IN79 is on/active, branch to EXIT in table-driven logic.                                      |
| BR-020 | HXXAPPPRF reads from HXPAPPPRF to retrieve configuration/reference data.                                      |

## 4. Data Model

SQL Server table mapping:

| AS400 Object | SQL Server Table | Purpose                                              |
|-------------|------------------|------------------------------------------------------|
| HAPTRFR     | dbo.HAPTRFR      | Transfer records / base inpatient census source.     |
| TXPNSTN     | dbo.TXPNSTN      | Status and nursing station reference data.          |
| HXPNSTN     | dbo.HXPNSTN      | Logical view over TXPNSTN (optional).               |
| TXPBNFIT    | dbo.TXPBNFIT     | Benefit plan base table.                            |
| HXPBNFIT    | dbo.HXPBNFIT     | Logical view over TXPBNFIT (optional).              |
| HXPTABLD    | dbo.HXPTABLD     | Table‑driven configuration dictionary.              |
| HXPLVL1–6   | dbo.HXPLVL1–6    | Organisational level hierarchy tables.              |
| HXPXMLD     | dbo.HXPXMLD      | XML detail segments storage.                         |
| HXPXMLR     | dbo.HXPXMLR      | XML record identifiers / response records.          |
| HXFXMLH     | dbo.HXFXMLH      | XML header / run metadata.                          |
| HXFXMLD     | dbo.HXFXMLD      | XML detail segments (mirror of HXPXMLD).           |

## 5. Acceptance Criteria

Edge cases to implement in the modern solution:

| Scenario                              | Expected Behavior                                                              |
|--------------------------------------|-------------------------------------------------------------------------------|
| Null/zero date in HAPTRFR            | Treat `00000000` as `null`; record may be excluded or flagged for review.    |
| Invalid date range (before 1800)     | Reject record per BR-003; do not include in census.                           |
| Invalid date range (after 2100)      | Reject record per BR-004; do not include in census.                           |
| Month outside 1–12                   | Reject per BR-005/BR-006; log validation error.                               |
| Day <= 0                             | Reject per BR-007; log validation error.                                      |
| Not-found TXPNSTN status             | Leave status fields null; mark row as incomplete and log warning.            |
| Not-found TXPBNFIT benefit           | Default to generic/self-pay benefit; log warning.                             |
| Deleted/obsolete benefit/status rows | Exclude from joins; treat as not-found.                                       |
| XML ID collision                     | Retry ID generation; on repeated failure, abort run and log critical error.   |
| HXFXMLH update failure               | Mark run as failed; do not present partial results to callers.                |
| Empty result set after filters       | Return empty `rows` with summary counts; no error, but log informational event.|
| Preference not configured in tables  | Fall back to default behaviour; highlight configuration gap in logs.         |

> Full detail: see Business_Processing_Rules_HABADTE.md
