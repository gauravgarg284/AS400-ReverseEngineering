# Process Narratives

This document provides business-focused narratives for key programs in the HABADTE project, based on semantic interpretations and approved business rules.

---

## Program: XFXCNTR

### Overview

RPGLE program in domain "Data Maintenance". Contains 2 rule(s) with average confidence 43%. XFXCNTR manages counter-based control flow, determining when iterative processing should stop based on the value of a shared counter variable.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

- **BR-001:** When X equals zero, branch to 'EXIT' (confidence 0.47).
- **BR-002:** When X equals 40, branch to 'EXIT' (confidence 0.4).

### Related Approved Rules (by Domain)

Within the DATA_MAINTENANCE domain, XFXCNTR participates alongside XFXCYMD, XFXLDSC, and XFXTABL. All of these programs implement guardrails for data quality and control flow. XFXCNTR specifically provides iteration bounds for shared processing loops used by other maintenance routines.

### Data Touched

XFXCNTR itself does not directly manipulate PHI-bearing files, but it influences how many records from files like HAPTRFR or OMPMAST are processed by higher-level orchestration programs such as HABADTE.

---

## Program: XFXCYMD

### Overview

RPGLE program in domain "Data Maintenance". Contains 6 rule(s) with average confidence 58%. XFXCYMD validates date components (year, month, and day) to ensure that encounter and transaction dates are within accepted ranges before they are used in patient management logic.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

- **BR-003:** When VYY is less than 1800, branch to 'EXIT' (confidence 0.56).
- **BR-004:** When VYY is greater than 2100, branch to 'EXIT' (confidence 0.56).
- **BR-005:** When VMM is less than 01, branch to 'EXIT' (confidence 0.56).
- **BR-006:** When VMM is greater than 12, branch to 'EXIT' (confidence 0.56).
- **BR-007:** When VDD is less than 01, branch to 'EXIT' (confidence 0.56).
- **BR-008:** When VDD is greater than DYS(VMM), branch to 'EXIT' (confidence 0.68).

### Related Approved Rules (by Domain)

In the DATA_MAINTENANCE domain, XFXCYMD complements XFXCNTR and XFXLDSC by providing input validation for date fields, which are critical for time-based reporting, billing windows, and patient stay calculations.

### Data Touched

XFXCYMD is invoked by HABADTE and other orchestration programs when they need to validate date fields in transfer records (HAPTRFR) and master records (OMPMAST). Although it does not directly own PHI fields, it protects downstream use of date-of-birth (WBDATE in HXPDICT) and encounter dates from being invalid.

---

## Program: XFXLDSC

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 56%. XFXLDSC manages mapping logic based on LDAMAP values, ensuring that lookups into level hierarchy and mapping tables remain within supported ranges.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

- **BR-009:** When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-010:** When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-011:** When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-012:** When LDAMAP is greater than 9999, branch to 'EXIT' (confidence 0.56).

### Related Approved Rules (by Domain)

XFXLDSC works together with XFXTABL to ensure that table-driven configuration applies only to valid mapping keys. It provides bounds checking to prevent the system from using unsupported level mappings when interpreting hierarchical account structures.

### Data Touched

XFXLDSC reads level files HXPLVL1–HXPLVL6 (`HXFLVL1`–`HXFLVL6` record formats) to interpret hierarchy depth. These files do not contain PHI, but they are used to locate PHI-bearing records such as account and MRN fields in HAPTRFR and OMPMAST.

---

## Program: XFXTABL

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 90%. XFXTABL controls table-driven configuration using indicator *IN79 as an on/off switch for portions of the maintenance logic.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

- **BR-013:** When *IN79 equals on/active, branch to 'EXIT' (confidence 0.9).
- **BR-014:** When *IN79 equals on/active, branch to 'EXIT' (confidence 0.9).
- **BR-015:** When *IN79 equals on/active, branch to 'EXIT' (confidence 0.9).
- **BR-016:** When *IN79 equals on/active, branch to 'EXIT' (confidence 0.9).

### Related Approved Rules (by Domain)

XFXTABL shares the DATA_MAINTENANCE domain with XFXCNTR, XFXCYMD, and XFXLDSC. Together, they implement the configuration backbone for HABADTE and other orchestration programs. XFXTABL ensures that when a global indicator (*IN79) is active, table-based logic shuts down cleanly.

### Data Touched

XFXTABL reads configuration from `HXPTABLD` and logical files `HXLTABLD`, `HXLTABLP`, and `HXLTABLS`. These tables do not contain PHI themselves, but they direct how PHI-bearing records in HAPTRFR, OMPMAST, and OXPBNFIT are processed.

---

## Program: HABADTE

### Overview

RPGLE program in domain "Patient Management". Contains 3 rule(s) with average confidence 97%. HABADTE is the main orchestration program that processes patient accounts and encounters, determining which records are included in XML output and downstream reporting based on file indicators, void flags, and inpatient/outpatient status.

### Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT
- **Type:** RPGLE

### Business Rules Applied

- **BR-017:** When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92).
- **BR-018:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99).
- **BR-019:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99).

### Related Approved Rules (by Domain)

Within PATIENT_MANAGEMENT, HABADTE consumes the output of DATA_MAINTENANCE utilities (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL) and applies its own inclusion/exclusion rules. It ensures that only active, non-void inpatient encounters progress to XML generation and reporting.

### Data Touched

HABADTE interacts with several PHI-bearing files and related logical files:

- **HAPTRFR (HAFTRFR)** – Transfer records with account and MRN:
  - PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN).
- **OMPMAST (HMFMAST)** – Patient master records:
  - PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`.
- **OXPBNFIT (XFFBNFIT)** – Benefit configuration:
  - PHI field: `XFBTEL` (PhoneNumber).
- **OAPIRNK (HBFIRNK)** – Rank/benefit linkage:
  - PHI field: `BRKMRN` (MRN).

HABADTE declares and reads additional files for XML handling and printer output:

- `HXPXMLD`, `HXPXMLR`, and `HXFXMLH`/`HXFXMLD` for XML header/detail.
- Logical files `HXPBNFIT`, `HXPNSTN`, `HAPIRNK`, and `HMLMAST5H` for PFILE_OF relationships.

Because HABADTE touches multiple PHI-bearing fields (`AFACCT`, `AFMRNO`, `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `BRKMRN`, and phone numbers), any migrated implementation must enforce strict access controls and auditing consistent with healthcare privacy regulations.
