# Functional Specification Document (FSD)

## HABADTE - HABADTE Migration to Java / Spring Boot / SQL Server

**Version:** 1.0 DRAFT  **Source:** Reverse-engineered from HABADTE.RPGLE

**Purpose:** Authoritative migration specification for the forward engineering team.

The HABADTE admission screening system evaluates transfer records and related plan/benefit data to decide which encounters enter the inpatient admission workflow. It applies a set of exclusion rules (file indicator, void flag, outpatient flag) and then enriches accepted records with plan status, benefit plan and level mapping information before producing XML-based outputs for downstream systems.

---

## 1. Functional Scope

In-scope filter rules:
- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

These rules must be fully implemented and preserved in the migrated solution.

---

## 2. API Contract

### Original Sections (2) Inputs and (14) Spring Boot API Design

**Inputs (API Request Parameters)**

| Parameter                    | AS400 Field / Concept     | Type        | Description |
|------------------------------|---------------------------|-------------|-------------|
| transferLevel6               | AFLVL6                    | integer     | Level-6 transfer classification used to group transfer records. |
| accountNumber                | AFACCT                    | long        | Patient account number; identifies the financial account associated with the encounter. |
| transferDate                 | AFTRDT                    | string (CYMD) | Transfer date in calendar-YYYYMMDD format; must pass date validation rules. |
| transferTime                 | AFTRTM                    | string (HHMMSS) | Transfer time; combined with transferDate to sequence events. |
| transferType                 | AFTYPE                    | string      | Transfer type code driving specific admission flows. |
| inpatientOutpatientFlag      | domain flag (INPAT/OUTPAT)| string      | Indicates whether the encounter is inpatient or outpatient. |
| fileIndicator                | -FILE INDICATOR           | integer     | Binary/indicator field used to mark records as active (non-zero) or inactive (zero). |
| voidFlag                     | -FLAG INDICATOR           | string      | Flag indicating void/voided status of the record. |
| planStatusLevel6             | XFNLV6                    | integer     | Level-6 code linking to plan status in OXPNSTN/XFFNSTN. |
| planStatusCode               | XFNSST                    | string      | Plan status code; drives status mapping from plan status master. |
| userId                       | XMDUSR/XMRUSR             | string      | User identifier recorded in XML header/detail workfiles. |
| sequenceNumber               | XMDSEQ/XMRSEQ             | integer     | Sequence number tying together XML messages for the same request. |

Security note: OAuth2/RBAC will protect PHI-bearing fields and operations.

**Spring Boot API Design**

- Method: `POST`
- Path: `/api/admissions/screen`

Parameters (request body JSON):

| Name                    | Type      | Required | Validation |
|-------------------------|-----------|----------|-----------|
| accountNumber           | long      | yes      | > 0 |
| transferLevel6          | int       | yes      | 1–999999 |
| fromDate                | string    | yes      | YYYYMMDD, valid date |
| toDate                  | string    | yes      | YYYYMMDD, valid date, >= fromDate |
| inpatientOutpatientFlag | string    | no       | 'INPATIENT' or 'OUTPATIENT' |

Layer structure:
- Controller: `AdmissionScreenController`
- Service: `AdmissionScreenService`
- Repositories: `TransferRepository`, `PlanStatusRepository`, `BenefitPlanRepository`, `PatientMasterRepository`, `XmlDetailRepository`, `XmlHeaderRepository`.

Response JSON shape:

```json
{
  "runId": "202607021710",
  "accountNumber": 1234567890,
  "fromDate": "20260101",
  "toDate": "20260131",
  "summary": {
    "processedTransfers": 120,
    "acceptedAdmissions": 80,
    "skippedFileIndicator": 10,
    "skippedVoidFlag": 20,
    "skippedOutpatient": 10
  },
  "admissions": [
    {
      "accountNumber": 1234567890,
      "transferDate": "20260110",
      "transferTime": "083000",
      "transferType": "TR",
      "planStatusCode": "ACT",
      "benefitPlanCode": "PLN01",
      "level6Code": 100001,
      "statusFlags": ["OK"]
    }
  ]
}
```

Java DTOs:

```java
public record AdmissionRequest(
    long accountNumber,
    int transferLevel6,
    String fromDate,
    String toDate,
    String inpatientOutpatientFlag
) {}

public record AdmissionSummary(
    int processedTransfers,
    int acceptedAdmissions,
    int skippedFileIndicator,
    int skippedVoidFlag,
    int skippedOutpatient
) {}

public record AdmissionRecord(
    long accountNumber,
    LocalDate transferDate,
    LocalTime transferTime,
    String transferType,
    String planStatusCode,
    String benefitPlanCode,
    int level6Code,
    List<String> statusFlags
) {}

public record AdmissionResponse(
    String runId,
    long accountNumber,
    String fromDate,
    String toDate,
    AdmissionSummary summary,
    List<AdmissionRecord> admissions
) {}
```

---

## 3. Business Rules (Summary)

| Rule ID | Description |
|---------|-------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

Supporting rules from enrichment utilities:

| Rule ID | Description |
|---------|-------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'. |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'. |
| BR-005  | When VMM is less than 01, branch to 'EXIT'. |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'. |
| BR-007  | When VDD is less than 01, branch to 'EXIT'. |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'. |
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT'. |
| BR-013  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-020  | SQL program accesses table 'HXPAPPPRF'. |

---

## 4. Data Model

AS400 → SQL Server mapping:

| AS400 Object | SQL Server Table       | Purpose |
|--------------|------------------------|---------|
| HAPTRFR      | dbo.HAPTRFR            | Transfer/admission source records. |
| OMPMAST      | dbo.PatientMaster      | Patient master (MRN, account, demographics). |
| OAPIRNK      | dbo.TransferIndex      | Indexed break/transfer records by level and sequence. |
| OXPBNFIT     | dbo.BenefitPlan        | Benefit plan definitions and coverage details. |
| OXPNSTN      | dbo.PlanStatus         | Plan status master keyed by level and status code. |
| HXPDICT      | dbo.Dictionary         | Reference and dictionary values; PHI-bearing fields. |
| HXPXMLD      | dbo.XmlDetail          | XML detail workfile. |
| HXPXMLR      | dbo.XmlResponse        | XML response workfile. |

Suggested indexes:

```sql
CREATE INDEX IX_HAPTRFR_Main
    ON dbo.HAPTRFR (AFACCT, AFTRDT, AFTRTM, AFLVL6, AFTYPE);

CREATE INDEX IX_PlanStatus_Key
    ON dbo.PlanStatus (XFNLV6, XFNSST);

CREATE INDEX IX_BenefitPlan_Key
    ON dbo.BenefitPlan (XFBUBN, XFBPLN);

CREATE INDEX IX_PatientMaster_AccountMrn
    ON dbo.PatientMaster (MMACCT, MMMRNO);

CREATE INDEX IX_XmlDetail_UserSeq
    ON dbo.XmlDetail (XMDUSR, XMDSEQ, XMDSQ2);
```

---

## 5. Acceptance Criteria

Edge cases and expected behaviors:

| Scenario                                   | Expected Behavior |
|--------------------------------------------|-------------------|
| FILE_INDICATOR = 0                         | Record is skipped; SKIPPED_REASON = 'FILE_INDICATOR_ZERO'. |
| FLAG_INDICATOR = 'VOID' or 'VOIDED'       | Record is skipped; SKIPPED_REASON = 'VOID_RECORD'. |
| INPATIENT_OUTPATIENT_FLAG = 'OUTPATIENT'  | Record is skipped; SKIPPED_REASON = 'OUTPATIENT'. |
| Transfer date AFTRDT = 0                  | Treat as null; record fails date validation and is skipped or flagged as error. |
| Transfer date outside valid range         | VYY < 1800 or > 2100, VMM < 1 or > 12, VDD < 1 or > days-in-month → skip per XFXCYMD rules. |
| LDAMAP > 99 or > 9999                     | Treat as invalid mapping; log configuration error and apply safe defaults; avoid crashing flow. |
| Missing plan status row                   | Mark STATUS_MISSING = true; proceed with admission but flag for downstream review. |
| Missing benefit plan row                  | Mark BENEFIT_MISSING = true; proceed with admission; billing must handle missing benefits. |
| Missing level configuration               | Mark LEVEL_MISSING = true; use default level attributes. |
| Empty result set for account/date range   | Return summary with zero acceptedAdmissions and no admissions array entries. |
| Preference not configured (e.g., no default status)| Fail fast with error response indicating missing configuration; log event for operations. |

> Full detail: see Business_Processing_Rules_HABADTE.md
