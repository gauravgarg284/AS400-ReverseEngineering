# Process Narratives - HABADTE Project

This document describes the business processes implemented by each interpreted program in the HABADTE project, linking narrative text, key rules, and data access patterns.

## XFXCNTR

### Overview

RPGLE program in domain "Data Maintenance". Contains 2 rule(s) with average confidence 43%. XFXCNTR is a small utility used to center text within a fixed-width field, typically for report headings and labels.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

- **BR-001:** When X equals zero, branch to 'EXIT' (confidence 0.47).
- **BR-002:** When X equals 40, branch to 'EXIT' (confidence 0.40).

These rules ensure that centering only occurs when the current position X is within valid bounds (1–39). At X=0 or X=40, the routine exits, preventing misaligned or overflow output.

### Related Approved Rules by Domain

In the Data Maintenance domain, XFXCNTR contributes to presentation-level integrity. Its rules work alongside other utilities like XFXCYMD and XFXLDSC, which manage dates and level mappings respectively.

### Data Touched

XFXCNTR operates purely on in-memory fields and does not read or write database files. As such, it does not interact with PHI-flagged files (OMPMAST, HXPDICT, OXPBNFIT, HAPTRFR, OAPIRNK) and has no direct PHI exposure.

---

## XFXCYMD

### Overview

RPGLE program in domain "Data Maintenance". Contains 6 rule(s) with average confidence 58%. XFXCYMD validates and formats dates, supporting conversion from numeric CCYYMMDD values to display or comparison formats.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

Key rules:
- **BR-008:** When VDD is greater than DYS(VMM), branch to 'EXIT' (confidence 0.68).
- **BR-003:** When VYY is less than 1800, branch to 'EXIT' (confidence 0.56).
- **BR-004:** When VYY is greater than 2100, branch to 'EXIT' (confidence 0.56).
- **BR-005:** When VMM is less than 01, branch to 'EXIT' (confidence 0.56).
- **BR-006:** When VMM is greater than 12, branch to 'EXIT' (confidence 0.56).

Together these rules establish a strict validity window for dates used by downstream programs. Invalid dates cause an immediate exit from the conversion routine, preventing incorrect census or transfer computations.

### Related Approved Rules by Domain

These date rules are reused conceptually by HABADTE when validating censusDate and admit/transfer dates. They complement level mapping and configuration rules in other utilities to maintain overall data quality.

### Data Touched

XFXCYMD works on numeric date fields derived from various files but does not directly read from PF/LF objects. PHI fields such as dates of birth in HXPDICT (WBDATE) and admit dates in TMPMAST/OMPMAST feed into the routine, but the program itself only manipulates numeric values and returns validated results.

---

## XFXLDSC

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 56%. XFXLDSC is responsible for translating corporate level codes into human-readable descriptions using HXPLVL1–HXPLVL6.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

Key rules:
- **BR-009–BR-011:** When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-012:** When LDAMAP is greater than 9999, branch to 'EXIT' (confidence 0.56).

These rules ensure that lookup keys remain within valid ranges for level tables. If mapping codes are outside expected bounds, XFXLDSC exits rather than returning misleading descriptions.

### Related Approved Rules by Domain

Level mapping rules work with table configuration rules in XFXTABL to provide accurate textual descriptions used in report headers and XML metadata. In HABADTE, these descriptions appear at the top of the census output.

### Data Touched

XFXLDSC declares and reads from:
- **HXPLVL1–HXPLVL6** physical files (via HXFLVL1–HXFLVL6 record formats).

These files are not PHI-flagged; they store corporate hierarchy definitions rather than patient data. Therefore, XFXLDSC operates on non-PHI configuration data.

---

## XFXTABL

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 90%. XFXTABL implements table-driven configuration lookups, used for room class descriptions, leave flags, and other coded values.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

Key rules:
- **BR-013–BR-016:** When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).

These rules control termination of table processing based on a control indicator. When *IN79 is active, the routine exits, signalling that no further lookups should be performed (e.g., configuration disabled or already satisfied).

### Related Approved Rules by Domain

XFXTABL’s rules support HABADTE’s enrichment of room class and benefit information. They act in concert with level mapping and profile rules to determine which configuration entries are applied to each census record.

### Data Touched

XFXTABL reads from:
- **HXPTABLD** (XFFTABLD) and possibly related table variants XFFTABL2–4.
- Logical files **HXLTABLD/HXLTABLP/HXLTABLS** built over HXPTABLD.

These tables are configuration-oriented and not PHI-flagged. However, their outputs (e.g., leave flags) influence how PHI-bearing records from OMPMAST and HAPTRFR are interpreted in reports.

---

## HABADTE

### Overview

RPGLE program in domain "Patient Management". Contains 3 rule(s) with average confidence 97%. HABADTE is the main inpatient census driver that reads patient master and transfer data, applies inclusion/exclusion rules, enriches records, and outputs report and XML details.

### Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT
- **Type:** RPGLE batch/report program

### Business Rules Applied

Key rules:
- **BR-018:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99).
- **BR-019:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99).
- **BR-017:** When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92).

These rules define the active inpatient population: only non-voided, inpatient records with an active file indicator are counted.

### Related Approved Rules by Domain

HABADTE depends on:
- **Date rules (XFXCYMD)** for validating censusDate.
- **Level mapping (XFXLDSC)** for header descriptions.
- **Table lookups (XFXTABL)** for room class and leave flags.
- **Profile access (HXXAPPPRF/XFXMRNROL)** for MRN roll-up mode.

The approved rules in Data Maintenance domain thus underpin HABADTE’s core patient-management logic.

### Data Touched

HABADTE reads and writes:
- **HAPTRFR** (HAFTRFR) – transfer history, PHI fields AFACCT (AccountNumber), AFMRNO (MRN).
- **HMLMAST5H/TMPMAST/OMPMAST** – patient master data including MMACCT (AccountNumber), MMMRNO/MMMMRN (MRN), MMNAME (PatientName), MMPSSN (SSN).
- **HXPNSTN/OXPNSTN** – nursing station definitions.
- **HXPTABLD/HXLTABLD/P/S** – configuration tables.
- **HXPXMLD/HXPXMLR** – XML header/detail records.

PHI-flagged files involved:
- **OMPMAST** – primary patient master (AccountNumber, MRN, PatientName, SSN).
- **HXPDICT** – dictionary with additional MRN, account, and contact details.
- **OXPBNFIT** – benefit plans including phone numbers.
- **HAPTRFR** – transfer history with account and MRN.
- **OAPIRNK** – PF with MRN for breaking sequences.

HABADTE orchestrates PHI across these structures, applying strict inclusion rules to limit exposure to active inpatient accounts.

---

## HXXAPPPRF

### Overview

SQLRPGLE program in domain "Data Maintenance". Contains 3 rule(s) with average confidence 65%. HXXAPPPRF reads application profile tables to determine MRN roll-up behaviour and other configuration used by XFXMRNROL and HABADTE.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** SQLRPGLE configuration/profile reader

### Business Rules Applied

Key rules include:
- **BR-020:** SQL program accesses table 'HXPAPPPRF' (confidence 0.65).
- Additional profile access rules (BR-021, BR-022) reference tables like HXPAPPL6.

These rules collectively indicate that HXXAPPPRF must successfully read profile records in order for MRN roll-up mode to be determined.

### Related Approved Rules by Domain

HXXAPPPRF is downstream of XFXMRNROL and upstream of HABADTE’s grouping logic. Its configuration decisions influence whether census is organised by MRN or account, affecting report layout and totals.

### Data Touched

HXXAPPPRF accesses:
- **HXPAPPPRF**, **HXPAPPL6** – application profile tables (not individually detailed in compact schema).

These tables store configuration flags, not PHI. However, misconfiguration could indirectly alter how PHI-bearing records are grouped and presented.

---

## XFXGETID

### Overview

RPGLE program in domain "Data Maintenance". Cyclomatic complexity: 1. XFXGETID retrieves XML identifier fragments, supporting HABADTE’s XML output.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

No explicit BR-IDs are listed in key_rules for XFXGETID, but it participates in XML id generation and retrieval.

### Related Approved Rules by Domain

XFXGETID works in conjunction with HABADTE’s XML-writing subroutines and the configuration provided by CXXXMLP/CXXXMLC copybooks.

### Data Touched

XFXGETID reads from:
- **HXPXMLR/HXFXMLR** – XML header record file.

These structures do not contain PHI fields in the compact schema, serving primarily as XML metadata stores.

---

## XFXLEAP

### Overview

RPGLE program in domain "Data Maintenance". Cyclomatic complexity: 1. XFXLEAP calculates leap-year properties used by XFXCYMD.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

No explicit BR-IDs, but its logic supports BR-008 by determining days-in-month for February.

### Data Touched

XFXLEAP operates on in-memory numeric year values without direct PF/LF access or PHI exposure.

---

## XFXMRNROL

### Overview

RPGLE program in domain "Data Maintenance". Calls programs HXHAPPPRF and HXXAPPPRF. Cyclomatic complexity: 1. XFXMRNROL determines MRN roll-up mode, controlling whether HABADTE groups accounts by MRN or account number.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE configuration utility

### Business Rules Applied

No direct BR-IDs are listed, but XFXMRNROL’s primary business rule is:
- Read application profile via HXHAPPPRF/HXXAPPPRF.
- Set roll-up mode flag (e.g., BY_MRN vs BY_ACCOUNT) used by HABADTE.

### Data Touched

XFXMRNROL touches:
- **HXHAPPPRF** (missing program) and **HXXAPPPRF** – profile programs.

These programs in turn access non-PHI profile tables.

---

## Cross-Program Narrative Summary

The HABADTE project is structured as a central patient-management report (HABADTE) orchestrating several Data Maintenance utilities:

- **XFXCNTR** – presentation centering.
- **XFXCYMD/XFXLEAP** – date validation and conversion.
- **XFXLDSC** – corporate level description lookup.
- **XFXTABL** – table-driven configuration for room class and leave flags.
- **XFXGETID** – XML identifier retrieval.
- **XFXMRNROL/HXXAPPPRF** – MRN roll-up configuration.

Data access centres on PHI-bearing files (OMPMAST, HXPDICT, OXPBNFIT, HAPTRFR, OAPIRNK) with logical files (HMLMAST5H, HAPIRNK, HXPBNFIT, HXPNSTN) providing keyed views. The approved rules ensure data quality (dates, mappings), enforce clinical scope (inpatient vs outpatient), and control configuration-driven behaviours. Modernization efforts must preserve these narratives and rule linkages to maintain business correctness and compliance.
