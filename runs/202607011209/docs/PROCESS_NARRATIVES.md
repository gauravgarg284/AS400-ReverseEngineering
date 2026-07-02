# Process Narratives – HABADTE Project

This document provides program-level narratives synthesised from the semantic interpretations and approved business rules for run `202607011209`.

---

## Program: XFXCNTR

**Domain:** DATA_MAINTENANCE  
**Type:** RPGLE

### Overview

XFXCNTR is a small utility program in the Data Maintenance domain that manages a numeric counter or loop index. The interpretation summary describes it as "RPGLE program in domain 'Data Maintenance'. Contains 2 rule(s) with avg confidence 43%." Its primary role is to control iteration through records or configuration entries, terminating processing when the counter reaches boundary values.

### Business Rules Applied

Key rules (from interpretations_detail and approved_rules):

- **BR-001:** When X equals zero, branch to 'EXIT' (confidence 0.47).
- **BR-002:** When X equals 40, branch to 'EXIT' (confidence 0.40).

These rules ensure that counter-driven loops start and stop at defined boundaries, typically from 1 to 39.

### Related Approved Rules (DATA_MAINTENANCE)

Within the Data Maintenance domain, XFXCNTR works alongside rules from date and mapping utilities (XFXCYMD, XFXLDSC, XFXTABL), although it does not directly manipulate PHI. Its behaviour impacts higher-level programs like HABADTE, which invoke counter logic for iteration.

### Data Touched

XFXCNTR does not declare PF/LF files in the compact schema; its data is largely internal numeric variables. It is called by HABADTE (as shown in dep_edges), indicating its values contribute to control flow in patient-management processing but not directly to persisted data.

---

## Program: XFXCYMD

**Domain:** DATA_MAINTENANCE  
**Type:** RPGLE

### Overview

XFXCYMD is a calendar validation utility. The narrative states: "RPGLE program in domain 'Data Maintenance'. Contains 6 rule(s) with avg confidence 58%." It validates year (`VYY`), month (`VMM`), and day (`VDD`) components to ensure dates fall within a supported range and are structurally valid.

### Business Rules Applied

Key rules:

- **BR-003:** When VYY is less than 1800, branch to 'EXIT'.
- **BR-004:** When VYY is greater than 2100, branch to 'EXIT'.
- **BR-005:** When VMM is less than 01, branch to 'EXIT'.
- **BR-006:** When VMM is greater than 12, branch to 'EXIT'.
- **BR-007:** When VDD is less than 01, branch to 'EXIT'.
- **BR-008:** When VDD is greater than DYS(VMM), branch to 'EXIT'.

Together, these rules enforce a closed range of valid dates and ensure the day-of-month fits the month (including leap-year handling via the `DYS(VMM)` function).

### Related Approved Rules (DATA_MAINTENANCE)

XFXCYMD complements XFXCNTR and XFXLDSC by providing temporal validation, which is critical for programs like HABADTE that work with transfer dates (`AFTRDT`) and XML logging timestamps. Any invalid date encountered will be rejected or sanitized before use in downstream queries.

### Data Touched

XFXCYMD operates on in-memory date components and does not directly read or write PF/LF files. However, its logic is applied wherever DECIMAL(8,0) date fields (such as those in `HAPTRFR` and `OMPMAST`) are interpreted.

---

## Program: XFXLDSC

**Domain:** DATA_MAINTENANCE  
**Type:** RPGLE

### Overview

XFXLDSC is a level-description utility. Its narrative: "RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 56%." It validates and maps level codes used to navigate hierarchical station files (HXPLVL1–HXPLVL6) for hospital facilities.

### Business Rules Applied

Key rules:

- **BR-009:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **BR-010:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **BR-011:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **BR-012:** When LDAMAP is greater than 9999, branch to 'EXIT'.

These rules restrict mapping codes to configured ranges (two-digit and four-digit bands), preventing invalid level mappings from driving file reads.

### Related Approved Rules (DATA_MAINTENANCE)

XFXLDSC is closely associated with station-level PFs (`HXPLVL1–6`) accessed by HABADTE via XFXLDSC and related dependencies. Correct enforcement of LDAMAP thresholds ensures that hospital hierarchy lookups remain within configured boundaries, reducing risk of retrieving incorrect facility/unit data.

### Data Touched

According to `dep_edges`, XFXLDSC reads from `HXFLVL1`–`HXFLVL6`, which are physical files representing level data. These are not explicitly listed in `data_dict_schema.physical_files` but are clearly part of the hierarchy. No PHI fields are associated with these level files; they carry structural metadata like facility and unit codes.

---

## Program: XFXTABL

**Domain:** DATA_MAINTENANCE  
**Type:** RPGLE

### Overview

XFXTABL is a table-driven configuration utility. The narrative: "RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 90%." It reads configuration from `HXPTABLD` and related logical files, applying a panel indicator (`*IN79`) to determine whether table logic should run.

### Business Rules Applied

Key rules:

- **BR-013:** When *IN79 equals on/active, branch to 'EXIT'.
- **BR-014:** When *IN79 equals on/active, branch to 'EXIT'.
- **BR-015:** When *IN79 equals on/active, branch to 'EXIT'.
- **BR-016:** When *IN79 equals on/active, branch to 'EXIT'.

These repeated rules indicate that multiple code paths rely on the same indicator to suppress table processing when a certain user-interface or configuration state is active.

### Related Approved Rules (DATA_MAINTENANCE)

XFXTABL interacts with `HXPTABLD` and its logical views (`HXLTABLD`, `HXLTABLP`, `HXLTABLS`) as shown in `dep_edges`. Its indicator-driven branching influences both Data Maintenance tools and HABADTE, which calls XFXTABL for table-based lookups.

### Data Touched

From `data_dict_schema.physical_files` and logical files:

- `HXPTABLD` (PF): key fields `XFDTCD`, `XFDECD`; no PHI fields.
- `HXLTABLD`, `HXLTABLP`, `HXLTABLS` (LF): logical views used to segment table definitions.

These tables control mappings like descriptions, status codes, or layout parameters without directly exposing PHI.

---

## Program: HABADTE

**Domain:** PATIENT_MANAGEMENT  
**Type:** RPGLE

### Overview

HABADTE is the central patient-management program in this run. Its narrative: "RPGLE program in domain 'Patient Management'. Contains 3 rule(s) with avg confidence 97%." It orchestrates inpatient transfer selection, enrichment, and XML export. HABADTE has cyclomatic complexity 152 (HIGH band) and is a dependency hotspot with a score of 38, fan-out of 13 program calls, and multiple file operations.

### Business Rules Applied

Key rules:

- **BR-018:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99).
- **BR-019:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99).
- **BR-017:** When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92).

These rules govern whether transfer records and entire accounts are eligible for inclusion in inpatient XML exports and printed reports.

### Related Approved Rules (DATA_MAINTENANCE and PATIENT_MANAGEMENT)

HABADTE relies on Data Maintenance utilities for validation and enrichment:

- Date rules from XFXCYMD (BR-003–BR-008) ensure transfer dates and XML timestamps are valid.
- Counter rules from XFXCNTR (BR-001–BR-002) may drive iteration logic for transfer lists.
- Mapping rules from XFXLDSC and XFXTABL (BR-009–BR-016) control station hierarchy and table-based configuration.

Within PATIENT_MANAGEMENT, HABADTE is the sole program with explicit approved rules, making it the anchor for inpatient transfer business semantics.

### Data Touched

From `data_dict_schema` and PHI flags:

- **Primary PF:** `HAPTRFR` (transfer records) – PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN).
- **Dictionary PF:** `HXPDICT` – PHI fields including MRN, phone (`XFBTEL`), patient names (`XCNAME`, `ENNAME`), room numbers (`HXRMNO`, `XFRMNO`), account numbers (`HVACCT`, `IHACCT`), and DOB (`WBDATE`).
- **Rank PF:** `OAPIRNK` – PHI field `BRKMRN` (MRN).
- **Patient Master PF:** `OMPMAST` – PHI fields `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`.
- **Benefit PF:** `OXPBNFIT` – PHI field `XFBTEL` (PhoneNumber).
- **Logical files:** `HAPIRNK`, `HMLMAST5H`, `HXPBNFIT`, `HXPNSTN` – used for rank, master, benefit, and station enrichment.

HABADTE interacts with these structures via declared files and dependencies (`dep_edges` and `data_lineage`), making it a focal point for PHI handling and compliance.

PHI-bearing files from `phi_flagged_files` relevant to HABADTE include: `OXPBNFIT`, `OMPMAST`, `HAPTRFR`, `HXPDICT`, and `OAPIRNK`. Any migration must enforce strict access controls and auditing around HABADTE’s endpoints.
