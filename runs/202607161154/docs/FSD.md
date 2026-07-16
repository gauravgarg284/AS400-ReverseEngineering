# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE program produces a nursing-station and organisation-level patient census for a requested date, excluding pre-admissions, voided accounts and outpatients, and enriching each qualifying patient with room, station, and preference-driven identifiers. It writes XML header and detail records representing the census, along with summary counts, for consumption by downstream systems and modern APIs.

---

## 1. Functional Scope

In-scope filter rules (key business rules for census selection):
- **BR-017** – Exclude Pre-Admitted Patients: file indicator = 0 records are excluded from active census.
- **BR-018** – Exclude Voided Accounts: accounts with flag = 'V' are cancelled and excluded.
- **BR-019** – Exclude Outpatients: accounts with I/O indicator = 'O' are excluded.

These rules, together with census date and organisational filters, define the functional scope of which patients are eligible to appear in the census output.

---

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter | AS400 Field / Concept | Type | Description |
|-----------|------------------------|------|-------------|
| censusDate | Requested census date (used with admission/discharge dates) | LocalDate | Date for which active patients are reported. |
| orgLevel | Organisational level number (1–6) | int | Hierarchy level at which the filter is applied. |
| orgCode | Organisational code at the chosen level | String | Code identifying facility/ward/unit; must match patient org at that level. |
| nursingStation | MMPNST (from HMLMAST5H/TMPMAST) | String | Nursing station code used to constrain the census scope. |
| facilityId | Facility identifier used in preference lookups | String | Key into application preference tables (HXXAPPPRF/HXHAPPPRF). |
| identifierMode | Preference for MRN vs account display | String ("MRN"/"ACCOUNT") | May be derived from application preferences, but can be overridden by caller. |

Security:
- OAuth2/OpenID Connect for authentication.
- RBAC for authorization.
- PHI audit logging for all census retrieval requests.

### Spring Boot API Design

**Method:** `GET`  
**Path:** `/api/v1/census`

Parameters:

| Name | Type | Required | Validation |
|------|------|----------|-----------|
| censusDate | String (ISO date) | Yes | Must be a valid date; not in the future beyond business rules. |
| orgLevel | int | Yes | 1–6 only. |
| orgCode | String | Yes | Non-empty; must match a known organisational code. |
| nursingStation | String | Yes | Non-empty; length within allowed range. |
| facilityId | String | No | If omitted, derived from user’s default facility. |
| identifierMode | String | No | Must be "MRN" or "ACCOUNT"; if omitted, derived from preferences. |

Layer structure:
- Controller: `CensusController` – validates and maps HTTP requests.
- Service: `CensusService` – implements business flow and rules.
- Repositories: `PatientRepository`, `TransferHistoryRepository`, `DictionaryRepository`, `NursingStationRepository`, `XmlBatchRepository`.

Response JSON shape:

```json
{
  "batchId": "CNS202607161154",
  "censusDate": "2026-07-16",
  "orgLevel": 6,
  "orgCode": "WARD-01",
  "nursingStation": "NS01",
  "totals": {
    "totalPatients": 42,
    "inpatientCount": 42,
    "leaveCount": 3
  },
  "patients": [
    {
      "accountNumber": "00012345",
      "mrn": "MRN000987",
      "patientName": "DOE, JANE",
      "admissionDate": "2026-07-10",
      "dischargeDate": null,
      "nursingStation": "NS01",
      "stationName": "Nursing Station 01",
      "roomNumber": "201A",
      "roomClass": "General Ward",
      "leaveFlag": "H"
    }
  ]
}
```

Java entities/DTOs:

```java
public record CensusRequest(
    LocalDate censusDate,
    int orgLevel,
    String orgCode,
    String nursingStation,
    String facilityId,
    String identifierMode
) {}

public record PatientCensusRow(
    String accountNumber,
    String mrn,
    String patientName,
    LocalDate admissionDate,
    LocalDate dischargeDate,
    String nursingStation,
    String stationName,
    String roomNumber,
    String roomClass,
    String leaveFlag
) {}

public record CensusResponse(
    String batchId,
    LocalDate censusDate,
    int orgLevel,
    String orgCode,
    String nursingStation,
    int totalPatients,
    int inpatientCount,
    int leaveCount,
    List<PatientCensusRow> patients
) {}
```

---

## 3. Business Rules (Summary)

Business rules reference summary:

| Rule ID | Description |
|---------|-------------|
| BR-017 | Exclude Pre-Admitted Patients: file indicator = 0 records are excluded from active census. |
| BR-018 | Exclude Voided Accounts: accounts with flag = 'V' are cancelled and excluded. |
| BR-019 | Exclude Outpatients: accounts with I/O indicator = 'O' are excluded. |
| BR-020 | Organisational Level Filter: patient org code at requested level must match orgCode. |
| BR-021 | Census Date – Admission Date Check: patients admitted after censusDate are excluded. |
| BR-022 | Census Date – Discharge Date Check: patients discharged on or before censusDate are excluded; zero date = not discharged. |
| BR-023 | Room Number Retrieval from Transfer History: current room derived from latest transfer up to censusDate. |
| BR-024 | Room Class Description Lookup: room class code resolved via dictionary tables. |
| BR-025 | Hospital / Therapeutic Leave Flag: leave flag derived from room class mapping. |
| BR-026 | Nursing Station Name Resolution: station name looked up from nursing station table. |
| BR-027 | Patient Identifier Display – MRN vs Account Number (preference-driven). |
| BR-028 | Application Preference Lookup with Facility Override. |
| BR-029 | Patient Counting – three running totals maintained during processing. |

---

## 4. Data Model

AS400-to-SQL Server mappings:

| AS400 Object | SQL Server Table | Purpose |
|-------------|------------------|---------|
| TMPMAST / HMLMAST5H | PatientMaster | Primary patient admissions master and view. |
| HAPTRFR | TransferHistory | Patient transfer events and current room resolution. |
| HXPTABLD / HXLTABLD/HXLTABLP/HXLTABLS | DictionaryRoomClass | Room class and status mapping. |
| HXPLVL1–HXPLVL6 | OrgHierarchyLevel1–6 | Organisational hierarchy tables for level codes/descriptions. |
| TXPNSTN / HXPNSTN | NursingStation | Nursing station reference and descriptors. |
| HXPXMLR / HXFXMLR | XmlHeader | XML batch/header metadata. |
| HXPXMLD / HXFXMLD | XmlDetail | XML census detail rows. |

Suggested SQL Server indexes:

```sql
CREATE INDEX IX_PatientMaster_Census
    ON PatientMaster (nursing_station, admission_date, admission_time);

CREATE INDEX IX_TransferHistory_PatientDate
    ON TransferHistory (level6, account_number, transfer_date, transfer_time);

CREATE INDEX IX_DictionaryRoomClass_Code
    ON DictionaryRoomClass (room_class_code);

CREATE INDEX IX_NursingStation_LevelStation
    ON NursingStation (level6, station_code);

CREATE INDEX IX_XmlHeader_Batch
    ON XmlHeader (batch_id);

CREATE INDEX IX_XmlDetail_BatchSeq
    ON XmlDetail (batch_id, sequence);
```

---

## 5. Acceptance Criteria

Edge cases to implement as acceptance criteria:

| Scenario | Expected Behavior |
|----------|-------------------|
| Null/zero admission date | Treat 0 as "no date"; exclude record or route to error handling depending on policy. |
| Null/zero discharge date | Interpret as "not discharged"; patient considered active if other filters pass. |
| Transfer history not found | Keep patient in census but leave room-related fields blank or "Unknown"; log for follow-up. |
| Dictionary room class not found | Use raw code; mark description as "Unmapped"; do not block census. |
| Nursing station not found | Display station code only; mark name as "Unknown". |
| Voided accounts | Never include in output; ensure downstream XML does not contain these accounts. |
| Pre-admission records | Always excluded; verify no such records leak into XML detail. |
| Outpatient records | Always excluded for this census; if other reports handle outpatients, they must use different logic. |
| Empty result set | Return header with zero totals and an empty patients array; do not treat as error. |
| Preference not configured | Fall back to default identifier mode (e.g., MRN) and default facility; log configuration gap. |

> Full detail: see Business_Processing_Rules_HABADTE.md
