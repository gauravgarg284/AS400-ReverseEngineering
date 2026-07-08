# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

HABADTE is the central patient transfer management program. It reads transfer records from HAPTRFR, applies file/void/outpatient skip rules, enriches data with level hierarchy, status normalization, and dictionary/benefit details, then writes XML header and detail records for downstream reporting and integration. The modernized system will expose this behavior via a secure REST API backed by SQL Server, preserving all core inclusion/exclusion logic and enrichment semantics.

## 1. Functional Scope

The following filter rules are in scope and must be fully implemented:

- **BR-017** – When FILE INDICATOR equals zero, branch to `SKIP` (exclude inactive file records).
- **BR-018** – When FLAG INDICATOR equals void/voided, branch to `SKIP` (exclude logically voided transfers).
- **BR-019** – When INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP` (limit report to inpatient records).

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter           | AS400 Field / Source      | Type        | Description |
|---------------------|---------------------------|-------------|-------------|
| `unitLevel6`        | AFLVL6 (HAPTRFR)          | integer     | Level-6 unit/location code used to select transfers for a specific ward or department. |
| `accountNumber`     | AFACCT (HAPTRFR)          | long        | Optional account filter; if provided, restricts processing to a single patient account. |
| `transferDateFrom`  | AFTRDT (HAPTRFR)          | LocalDate   | Start of transfer date range (YYYYMMDD in legacy). |
| `transferDateTo`    | AFTRDT (HAPTRFR)          | LocalDate   | End of transfer date range. |
| `transferTimeFrom`  | AFTRTM (HAPTRFR)          | LocalTime   | Start of transfer time window (HHMMSS). |
| `transferTimeTo`    | AFTRTM (HAPTRFR)          | LocalTime   | End of transfer time window. |
| `transferType`      | AFTYPE (HAPTRFR)          | string      | Optional transfer type filter (e.g., admit, discharge, internal move). |
| `includeOutpatients`| In/Out flag (derived)     | boolean     | Indicates whether outpatient records should be included; default is `false` to mirror BR-019. |

**Spring Boot security mapping:**
- OAuth2/OpenID Connect for authentication.
- RBAC limiting access to transfer audit/billing roles.
- PHI access audited at the service layer.

### Spring Boot API Design

**Endpoint:**

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

**Layer Structure**

- Controller: `TransferReportController`
- Service: `TransferReportService`
- Repositories: `PatientTransferRepository`, `LevelRepository`, `StatusRepository`, `DictionaryRepository`, `BenefitPlanRepository`

**Response JSON Shape**

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

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017  | When FILE INDICATOR equals zero, branch to `SKIP` (exclude inactive file records). |
| BR-018  | When FLAG INDICATOR equals void/voided, branch to `SKIP` (exclude logically voided transfers). |
| BR-019  | When INPATIENT/OUTPATIENT FLAG equals outpatient, branch to `SKIP` (limit report to inpatient records). |

Supporting rules relevant to HABADTE’s flow:

| Rule ID | Description |
|---------|-------------|
| BR-009–012 | Level lookup: reject if level code exceeds valid range (XFXLDSC). |
| BR-013–016 | When *IN79 equals on/active, branch to `EXIT` for table lookups (XFXTABL). |

## 4. Data Model

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

## 5. Acceptance Criteria

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

> Full detail: see Business_Processing_Rules_HABADTE.md
