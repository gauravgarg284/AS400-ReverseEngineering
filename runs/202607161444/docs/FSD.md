# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

HABADTE implements the “Active Patients by Date and Time” census report, selecting inpatient accounts active at a specified census date/time and organisational scope, enriching them with transfer history, nursing station, and room class/leave information, and emitting both XML and (legacy) printed outputs. The modern system will expose this functionality as a secured REST API and persist results in SQL Server tables equivalent to the AS400 DDS-based structures.

## 1. Functional Scope

In-scope filter rules for the migration (as per approved HABADTE rules):

- **BR-017 – Exclude Pre-Admitted Patients**: A patient with file indicator = 0 is a pre-admission and must be excluded from the active census.
- **BR-018 – Exclude Voided Accounts**: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census.
- **BR-019 – Exclude Outpatients**: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.

These three rules define the core inclusion boundary for which accounts are considered active in the census.

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter           | AS400 Field / Concept    | Type        | Description                                                                 |
|---------------------|--------------------------|------------|-----------------------------------------------------------------------------|
| `censusDate`        | Report date (CCYYMMDD)   | `LocalDate`| Date at which census is evaluated. Must be between 1800-01-01 and 2100-12-31. |
| `censusTime`        | Report time (HHMM)       | `LocalTime`| Time-of-day at which census snapshot is taken.                              |
| `orgLevel`          | Organisational level 1–6 | `int`      | Level of hierarchy used to filter patients. Must be within valid range.     |
| `orgCode`           | Organisational code      | `String`   | Code identifying the organisation at the selected level.                    |
| `includeLeave`      | Derived from XFDMAP      | `boolean`  | Whether to include patients on hospital/therapeutic leave.                  |
| `userId`            | XML user key (XMDUSR)    | `String`   | User identifier written to XML header/detail records.                       |
| `sequenceSeed`      | XML sequence (XMDSEQ)    | `long`     | Starting sequence number for XML records.                                   |

Spring Boot security:
- Authenticate using OAuth2/OpenID Connect.
- Authorize access with RBAC roles (e.g., `ROLE_CENSUS_VIEW`).
- Audit all API calls with user and parameter context.

### Spring Boot API Design

- Method: `GET`
- Path: `/api/census/active-patients`

Parameters:

| Name          | Type        | Required | Validation                                      |
|---------------|------------|----------|-------------------------------------------------|
| `censusDate`  | `String`   | Yes      | ISO date, convert to `LocalDate`, 1800–2100.    |
| `censusTime`  | `String`   | Yes      | HH:mm, convert to `LocalTime`.                  |
| `orgLevel`    | `Integer`  | Yes      | 1–6; reject invalid levels.                     |
| `orgCode`     | `String`   | Yes      | Non-empty; must exist in level tables.          |
| `includeLeave`| `Boolean`  | No       | Default `true`.                                 |

Layer structure:
- Controller: `CensusController`.
- Service: `ActiveCensusService`.
- Repositories: patient, transfer, room class, station, XML header/detail.
- Utility services: date validation, level lookup, table lookup, XML ID generation.

Response JSON shape:

```json
{
  "metadata": {
    "censusDate": "2026-07-16",
    "censusTime": "14:44",
    "orgLevel": 6,
    "orgCode": "HV0001",
    "orgDescription": "Main Hospital Facility",
    "totalActivePatients": 123,
    "hospitalLeaveCount": 5,
    "therapeuticLeaveCount": 3
  },
  "patients": [
    {
      "accountNumber": "1234567",
      "facilityLevel": "HV0001",
      "patientName": "Doe, Jane",
      "medicalRecordNumber": "MRN000001",
      "admissionDate": "2026-07-01",
      "dischargeDate": null,
      "roomNumber": "402A",
      "roomClassCode": "IC",
      "roomClassDescription": "Intensive Care",
      "nursingStation": "ICU",
      "stationDescription": "Intensive Care Unit",
      "leaveStatus": "NONE"
    }
  ]
}
```

## 3. Business Rules (Summary)

Business rule summary table:

| Rule ID | Description                                                                                                                     |
|---------|---------------------------------------------------------------------------------------------------------------------------------|
| BR-017  | Exclude Pre-Admitted Patients (fileIndicator = 0).                                                                             |
| BR-018  | Exclude Voided Accounts (voidFlag = 'V').                                                                                       |
| BR-019  | Exclude Outpatients (ioIndicator = 'O').                                                                                        |
| BR-020  | Organisational Level Filter: patient must belong to requested org scope at selected level.                                     |
| BR-021  | Admission Date Check: admissionDate must be on or before censusDate.                                                            |
| BR-022  | Discharge Date Check: dischargeDate is zero or after censusDate; else exclude patient.                                          |
| BR-023  | Room Number Retrieval: use latest transfer history entry at or before censusDate/time.                                          |
| BR-024  | Room Class Description Lookup via HXPTABLD/HXLTABL* and XFXTABL.                                                                |
| BR-025  | Hospital/Therapeutic Leave Flag: use 10th char of XFDMAP to derive leave status.                                               |

## 4. Data Model

SQL Server table mappings:

| AS400 Object | SQL Server Table | Purpose                                                |
|--------------|------------------|--------------------------------------------------------|
| HAPTRFR      | dbo.HAPTRFR      | Transfer history for room assignment and movement.    |
| OMPMAST      | dbo.OMPMAST      | Inpatient master records (demographics & account).    |
| HXPTABLD     | dbo.HXPTABLD     | Generic code and description table (room classes etc.). |
| HXPLVL1      | dbo.HXPLVL1      | Corporate level 1 hierarchy.                          |
| HXPLVL2      | dbo.HXPLVL2      | Level 2 hierarchy.                                    |
| HXPLVL3      | dbo.HXPLVL3      | Level 3 hierarchy.                                    |
| HXPLVL4      | dbo.HXPLVL4      | Level 4 hierarchy.                                    |
| HXPLVL5      | dbo.HXPLVL5      | Level 5 hierarchy.                                    |
| HXPLVL6      | dbo.HXPLVL6      | Level 6 hierarchy (facility-level).                   |
| HXPNSTN      | dbo.OXPNSTN      | Nursing station reference data.                       |
| HXPBNFIT     | dbo.HXPBNFIT     | Benefit reference table (used tangentially).          |
| HXPXMLD      | dbo.HXPXMLD      | XML detail records generated by HABADTE.              |
| HXPXMLR      | dbo.HXPXMLR      | XML report header records.                            |

Suggested indexes:

```sql
CREATE INDEX IX_OMPMAST_Org_Admit_Discharge
ON dbo.OMPMAST (orgLevel6Code, admissionDate, dischargeDate, ioIndicator, voidFlag, fileIndicator);

CREATE INDEX IX_HAPTRFR_Facility_Account_DateTime
ON dbo.HAPTRFR (AFLVL6, AFACCT, AFTRDT, AFTRTM);

CREATE INDEX IX_HXPTABLD_DataCode_RoomClass
ON dbo.HXPTABLD (XFDTCD, XFDECD);

CREATE INDEX IX_OXPNSTN_Facility_Station
ON dbo.OXPNSTN (XFNLV6, XFNSST);

CREATE INDEX IX_HXPXMLD_User_Seq
ON dbo.HXPXMLD (XMDUSR, XMDSEQ, XMDSQ2);

CREATE INDEX IX_HXPXMLR_User_Seq_Id
ON dbo.HXPXMLR (XMRUSR, XMRSEQ, XMRID);
```

## 5. Acceptance Criteria

Edge cases to implement:

| Scenario                                        | Expected Behavior                                                                                 |
|-------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Null/zero admission date                        | Treat as invalid; exclude or flag for data quality review.                                       |
| Null/zero discharge date                        | Interpret as “not discharged”; patient remains eligible subject to other filters.                  |
| Transfer history not found                      | Include patient with null room/station fields; increment `noRoomAssignedCount`.                   |
| Transfer after census time only                 | Ignore transfers after census; use latest transfer at or before census time.                      |
| Room class code not found in HXPTABLD           | Set roomClassDescription to “Unknown”; set leaveStatus to “NONE”; log data issue.                |
| XFDMAP shorter than 10 characters               | Do not derive leave status; treat as “NONE”.                                                      |
| Nursing station not found in HXPNSTN           | Display fallback “Unknown Station @code”; include patient but flag record.                       |
| Patient on hospital/therapeutic leave          | Include patient in census (if `includeLeave = true`) and increment respective leave counters.     |
| Empty result set after filters                  | Return empty `patients` list and metadata with counts = 0; write XML header/footer only.          |
| Preference not configured for MRN rollup       | Use default rollup logic; log configuration gap but do not fail the census run.                   |

> Full detail: see Business_Processing_Rules_HABADTE.md
