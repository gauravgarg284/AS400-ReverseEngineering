# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE system generates an "Active Accounts by Admit Date and Time" inpatient census, producing both printer output and XML payloads. It filters out voided and outpatient records, enriches data with nursing station names, room class configuration, and transfer history, and calculates facility-level totals for active inpatients and leave accounts. The modern implementation will expose this logic via a REST API, backed by SQL Server, while preserving all inclusion/exclusion and enrichment rules.

## 1. Functional Scope

IN SCOPE business rules (record filters):
- **BR-017** – When file indicator equals zero, skip the record from census.
- **BR-018** – When flag indicator equals void/voided, skip the record from census.
- **BR-019** – When inpatient/outpatient flag indicates outpatient, skip the record from census.

These three rules define the primary inclusion/exclusion criteria for the inpatient census.

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter Name      | AS400 Field / Concept | Type      | Description |
|---------------------|-----------------------|-----------|------------|
| facilityLevel6      | MMPLV6 / AFLVL6       | string    | Corporate level-6 code identifying the facility or hospital for which the census is generated. |
| censusDate          | AFTRDT / MMADDT       | date      | Census date (CCYYMMDD) representing the desired admit date snapshot. |
| censusTime          | AFTRTM / MMADTM       | time      | Census time (HHMM) representing the cutoff time within the census date. |
| nursingStationCode  | XFNSST / MMPNST       | string    | Optional nursing station filter; when provided, limits the census to a specific station. |
| includeLeave        | indicator             | boolean   | Flag indicating whether patients on hospital/therapeutic leave should be included in counts. |
| mrnRollupMode       | profile flag          | string    | Mode returned by MRN roll-up profile (e.g., "BY_MRN" vs "BY_ACCOUNT"); controls grouping and header labels. |
| roomClassCode       | XFDTCD                | string    | Optional room class code filter used by table lookup for descriptions. |
| requestUserId       | LDAUSR/XMDUSR/XMRUSR | string    | Current user identity taken from LDA and propagated into XML records and audit trails. |

Notes:
- Current user identity is sourced from the LDA (HXXLDA) and written into HXPXMLD/HXPXMLR via fields XMDUSR/XMRUSR.
- Spring Boot security must enforce OAuth2/JWT with RBAC for patient-management roles.

### Spring Boot API Design

- **Method:** `GET`
- **Path:** `/api/census/active-accounts`

Parameters:

| Name              | Type    | Required | Validation |
|-------------------|---------|----------|-----------|
| facilityLevel6    | string  | yes      | Must match known HXPLVL6 code. |
| censusDate        | string  | yes      | ISO date; converted to DECIMAL(8,0) with XFXCYMD rules. |
| censusTime        | string  | yes      | HH:mm; must be valid time. |
| nursingStationCode| string  | no       | If present, must exist in NursingStation table. |
| includeLeave      | boolean | no       | Default false. |

Layer structure:
- Controller: `CensusController`
- Service: `CensusService`
- Repositories: `PatientMasterRepository`, `TransferHistoryRepository`, `NursingStationRepository`, `ConfigTableRepository`, `BenefitPlanRepository`

Response JSON shape:

```json
{
  "facility": "General Hospital",
  "censusDate": "2024-03-31",
  "censusTime": "06:00",
  "mrnRollupMode": "BY_ACCOUNT",
  "totals": {
    "active": 125,
    "leave": 7,
    "beds": 125
  },
  "records": [
    {
      "accountNumber": "123456789012",
      "mrn": "MRN00000001",
      "patientName": "DOE, JANE",
      "nursingStation": "3EAST",
      "stationName": "3 East Medical",
      "roomNumber": "301A",
      "admitDate": "2024-03-29",
      "admitTime": "14:32",
      "roomClass": "SEMIPRIV",
      "roomClassDescription": "Semi-Private",
      "onLeave": false
    }
  ]
}
```

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017  | When file indicator equals zero, skip the record from census. |
| BR-018  | When flag indicator equals void/voided, skip the record from census. |
| BR-019  | When inpatient/outpatient flag indicates outpatient, skip the record from census. |
| BR-003  | Validate year: when VYY < 1800, exit date conversion. |
| BR-004  | Validate year: when VYY > 2100, exit date conversion. |
| BR-005  | Validate month: when VMM < 01, exit date conversion. |
| BR-006  | Validate month: when VMM > 12, exit date conversion. |
| BR-007  | Validate day: when VDD < 01, exit date conversion. |
| BR-008  | Validate day: when VDD > DYS(VMM), exit date conversion. |
| BR-009–BR-012 | Validate level mapping (LDAMAP) ranges for facility descriptions. |
| BR-013–BR-016 | When *IN79 indicator is on/active, exit table lookup processing. |
| BR-020 | SQL program HXXAPPPRF accesses table HXPAPPPRF for profile configuration. |

## 4. Data Model

### SQL Server Table Mapping

| AS400 Object | SQL Server Table      | Purpose |
|-------------|-----------------------|---------|
| TMPMAST/OMPMAST | dbo.PatientMaster  | Core patient account and demographic data. |
| HAPTRFR      | dbo.TransferHistory   | Room and bed transfer events per account. |
| HXPLVL1–6    | dbo.CorporateLevels   | Corporate hierarchy (levels 1–6). |
| HXPTABLD     | dbo.ConfigTable       | Generic configuration table for codes and descriptions. |
| HXLTABLD/P/S | dbo.ConfigTableViews  | Alternate keys and projections over configuration data. |
| OXPBNFIT     | dbo.BenefitPlans      | Benefit plan definitions and contact phone numbers. |
| OXPNSTN      | dbo.NursingStation    | Station definitions per facility. |
| HXPXMLD      | dbo.XmlDetail         | XML detail payload per user/run. |
| HXPXMLR      | dbo.XmlHeader         | XML header and id sequences. |

## 5. Acceptance Criteria

| Scenario                              | Expected Behavior |
|---------------------------------------|-------------------|
| Null/zero census date                 | Reject request; return validation error; no data processed. |
| Invalid date (month/day/year out of range) | Use XFXCYMD-equivalent validation; return error or skip record depending on context. |
| Null/zero census time                 | Treat as 00:00 or reject based on configuration; document decision. |
| Not-found nursing station lookup      | Print station code; log configuration gap; do not fail request. |
| Not-found room class mapping          | Use generic description; set leave_flag false; log gap. |
| Transfer record not found             | Use room from master record; still count record as active. |
| Delete/void flags on master record    | Apply BR-018; skip from census. |
| Outpatient flag on record             | Apply BR-019; skip from census. |
| Empty result set                      | Return response with totals = 0 and no records; report header still printed. |
| Preferences not configured (missing profile records) | Default mrnRollupMode to BY_ACCOUNT; mark response with warning; still produce census.

> Full detail: see Business_Processing_Rules_HABADTE.md
