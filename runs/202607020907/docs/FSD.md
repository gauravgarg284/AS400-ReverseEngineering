# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

HABADTE is the central patient management driver that reads transfer records from HAPTRFR, enriches them with organizational level and dictionary data, applies MRN rollover profiles, and writes XML header/detail records representing validated inpatient transfer events. The migrated system will expose this functionality as a REST API while preserving existing inclusion/exclusion rules and PHI-handling semantics.

---

## 1. Functional Scope

The functional scope of this migration centers around three key HABADTE rules:

- **BR-017** – When -FILE INDICATOR equals zero, branch to 'SKIP'.
- **BR-018** – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- **BR-019** – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

These rules define which transfer records are in scope for processing (valid, non-voided inpatient records) and must be fully preserved in the Java implementation.

---

## 2. API Contract

### Inputs (API Request Parameters)

| Parameter | AS400 Field / Concept | Type | Description |
|-----------|------------------------|------|-------------|
| accountLevel6 | AFLVL6 | int | Institution or level identifier used to partition transfer records. |
| accountNumber | AFACCT | long | Patient account number used as primary key in HAPTRFR and OMPMAST. |
| inpatientFlag | -INPATIENT/OUTPATIENT FLAG | string | Indicates whether the patient is inpatient or outpatient; only inpatient is processed. |
| fileIndicator | -FILE INDICATOR | int | File status indicator; 0 means no valid record and must be skipped (BR-017). |
| voidFlag | -FLAG INDICATOR | string | Transfer void status; "void" or "voided" records are skipped (BR-018). |
| userId | XMRUSR / XMDUSR | string | User identifier for XML header/detail keys. |
| sequenceNumber | XMRSEQ / XMDSEQ | long | Sequence value for XML message grouping. |

Spring Boot security mapping:
- OAuth2/OIDC for authentication.
- Role-based access control (RBAC) ensuring only users with `ROLE_PATIENT_MGMT` can invoke the transfer XML endpoint.
- PHI access is audited; every call is logged with user, accountNumber, and timestamp.

### Spring Boot API Design

- **Method:** `GET`  
- **Path:** `/api/patient-transfers/{accountNumber}`

Parameters:

| Name | Type | Required | Validation |
|------|------|----------|-----------|
| accountNumber | long | yes | Must be positive; must exist in OMPMAST. |
| accountLevel6 | int | yes | Must match a valid HXPLVL6.HX6NUM. |
| inpatientFlag | string | no | Defaults to "I" (inpatient); must be `I` or `O`. |

Layer structure:
- **Controller:** `PatientTransferController` – validates inputs and delegates to service.  
- **Service:** `PatientTransferService` – orchestrates HABADTE flow (validation, enrichment, XML generation).  
- **Repositories:** PF-backed repositories for HAPTRFR, HXPLVL1–6, HXPTABLD/HXLTABL*, OMPMAST, OXPBNFIT, OXPNSTN, HXPXMLR, HXPXMLD.

Response JSON shape:

```json
{
  "batchId": "XML202607020907-0001",
  "accountNumber": 123456789,
  "facilityLevel6": 999999,
  "transfers": [
    {
      "transferDate": "2026-07-02",
      "transferTime": "14:30",
      "transferType": "A",
      "unitDescription": "Cardiology Ward",
      "institutionStatus": "ACTIVE",
      "benefitCode": "PLAN-A",
      "mrn": "MRN00012345"
    }
  ],
  "summary": {
    "totalProcessedTransfers": 10,
    "totalSkippedVoid": 1,
    "totalSkippedOutpatient": 2,
    "dateValidationFailures": 0
  }
}
```

Java DTOs:

```java
public record TransferDto(
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String unitDescription,
    String institutionStatus,
    String benefitCode,
    String mrn
) {}

public record TransferBatchDto(
    String batchId,
    long accountNumber,
    int facilityLevel6,
    List<TransferDto> transfers,
    int totalProcessedTransfers,
    int totalSkippedVoid,
    int totalSkippedOutpatient,
    int dateValidationFailures
) {}
```

---

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'. |
| BR-003 | When VYY is less than 1800, branch to 'EXIT'. |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'. |
| BR-005 | When VMM is less than 01, branch to 'EXIT'. |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'. |
| BR-007 | When VDD is less than 01, branch to 'EXIT'. |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |

---

## 4. Data Model

| AS400 Object | SQL Server Table | Purpose |
|--------------|------------------|---------|
| HAPTRFR | dbo.HAPTRFR | Patient transfer records keyed by level, account, date/time, type. |
| HXPDICT | dbo.HXPDICT | Central dictionary/reference file with MRNs, account numbers, room and contact info. |
| HXPLVL1 | dbo.HXPLVL1 | Level-1 configuration (global level hierarchy). |
| HXPLVL2 | dbo.HXPLVL2 | Level-2 configuration (region/cluster). |
| HXPLVL3 | dbo.HXPLVL3 | Level-3 configuration (facility group). |
| HXPLVL4 | dbo.HXPLVL4 | Level-4 configuration (facility). |
| HXPLVL5 | dbo.HXPLVL5 | Level-5 configuration (department/unit). |
| HXPLVL6 | dbo.HXPLVL6 | Level-6 configuration (service/bed-level details). |
| HXPTABLD | dbo.HXPTABLD | Table dictionary codes. |
| HXPXMLD | dbo.HXPXMLD | XML detail records for transfer messaging. |
| HXPXMLR | dbo.HXPXMLR | XML header identifier records. |
| OAPIRNK | dbo.OAPIRNK | Break/rank records with MRN. |
| OMPMAST | dbo.OMPMAST | Patient master records with MRN, account, name, SSN. |
| OXPBNFIT | dbo.OXPBNFIT | Benefit records keyed by benefit and plan. |
| OXPNSTN | dbo.OXPNSTN | Institution status records keyed by level and status. |

---

## 5. Acceptance Criteria

| Scenario | Expected Behavior |
|----------|-------------------|
| File indicator is zero | Do not process the record; log as invalid file status (BR-017). |
| Void flag is void/voided | Skip record; increment `skippedVoidTransfers` (BR-018). |
| Inpatient/outpatient flag indicates outpatient | Skip record; increment `skippedOutpatientTransfers` (BR-019). |
| Date fields outside 1800–2100 range | Treat as invalid date; increment `dateValidationFailures`; skip record (BR-003, BR-004). |
| Month not in 1–12 or day not in valid range | Treat as invalid date; increment `dateValidationFailures`; skip record (BR-005–BR-008). |
| LDAMAP mapping code exceeds allowed ranges | Do not enrich level descriptions; use default or code-only representation (BR-009–BR-012). |
| Indicator *IN79 is active during dictionary lookup | Stop further table lookups for that record; rely on existing data (BR-013–BR-016). |
| MRN profile not found | Use AFMRNO as-is; mark record for potential manual review. |
| XML header/detail write fails | Log error, mark batch as partial failure, return error status to API caller. |
| No transfer records after filtering | Return an empty `transfers` array with summary counts set to 0.

> Full detail: see Business_Processing_Rules_HABADTE.md
