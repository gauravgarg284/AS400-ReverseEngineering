# PROCESS NARRATIVES – HABADTE Suite

Run ID: 202607021710  
Project: HABADTE AS400 admission and data maintenance suite

This document narrates the business processes implemented by each analyzed program, linking them to approved business rules and data structures.

---

## XFXCNTR

### Overview

XFXCNTR is an RPGLE program in the Data Maintenance domain. It encapsulates counter-based branching logic used by other programs to control loop execution and early exits. The program contains 2 business rules with an average confidence of 43%.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE utility

### Business Rules Applied

Key rules from `interpretations_detail`:

- BR-001 – "When X equals zero, branch to 'EXIT'" (confidence 0.47)
- BR-002 – "When X equals 40, branch to 'EXIT'" (confidence 0.40)

These rules are used to terminate processing when a counter reaches specific thresholds, such as the beginning (0) or a configured maximum (40).

### Related Approved Rules by Domain

From the approved rules list for DATA_MAINTENANCE:
- BR-001, BR-002 (XFXCNTR)
- BR-003–BR-008 (XFXCYMD)
- BR-009–BR-012 (XFXLDSC)
- BR-013–BR-016 (XFXTABL)
- BR-020 (HXXAPPPRF)

XFXCNTR’s rules work alongside date and mapping rules to ensure that loops and counters used in maintenance processes do not run beyond valid ranges.

### Data Touched

XFXCNTR does not directly read or write PHI-bearing files; it operates on internal counters.

Relevant files indirectly in the same domain:
- HXPTABLD / HXLTABLD / HXLTABLP / HXLTABLS – dictionary tables.
- HXPLVL1–HXPLVL6 – level tables.

PHI-flagged files in the suite (not directly touched by XFXCNTR): HAPTRFR, HXPDICT, OXPBNFIT, OAPIRNK, OMPMAST.

---

## XFXCYMD

### Overview

XFXCYMD is an RPGLE program in the Data Maintenance domain that validates calendar dates used across the system. It contains 6 rules with an average confidence of 58%, and it calls XFXLEAP to handle leap-year logic.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE utility

### Business Rules Applied

Key rules:
- BR-008 – "When VDD is greater than DYS(VMM), branch to 'EXIT'" (confidence 0.68)
- BR-003 – "When VYY is less than 1800, branch to 'EXIT'" (confidence 0.56)
- BR-004 – "When VYY is greater than 2100, branch to 'EXIT'" (confidence 0.56)
- BR-005 – "When VMM is less than 01, branch to 'EXIT'" (confidence 0.56)
- BR-006 – "When VMM is greater than 12, branch to 'EXIT'" (confidence 0.56)
- BR-007 – "When VDD is less than 01, branch to 'EXIT'" (confidence 0.56)

These rules ensure that the year, month and day values supplied to admission and reporting programs are within valid bounds and consistent with the actual number of days in a month.

### Related Approved Rules by Domain

- Date validation rules (BR-003–BR-008) work with counter rules (BR-001–BR-002) and mapping rules (BR-009–BR-012) to maintain data integrity.

### Data Touched

XFXCYMD operates on date parameters and does not directly access tables from the data dictionary. However, its validation is critical for fields such as:
- HAPTRFR.AFTRDT – transfer date.
- HXPDICT.WBDATE – birth date.

PHI involvement: WBDATE in HXPDICT is tagged as DateOfBirth, so accurate date validation ensures clean PHI data.

---

## XFXLDSC

### Overview

XFXLDSC is an RPGLE program in the Data Maintenance domain that reads level tables to produce descriptive mappings for numeric level codes. It contains 4 rules with an average confidence of 56% and performs multiple file reads across HXPLVL1–HXPLVL6 and their runtime formats.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE workflow service

### Business Rules Applied

Key rules:
- BR-009 – "When LDAMAP is greater than 99, branch to 'EXIT'" (confidence 0.56)
- BR-010 – "When LDAMAP is greater than 99, branch to 'EXIT'" (confidence 0.56)
- BR-011 – "When LDAMAP is greater than 99, branch to 'EXIT'" (confidence 0.56)
- BR-012 – "When LDAMAP is greater than 9999, branch to 'EXIT'" (confidence 0.56)

These rules act as guardrails, preventing invalid mapping codes from being used when traversing level tables.

### Related Approved Rules by Domain

XFXLDSC’s mapping rules support HABADTE and other programs that rely on level-based configuration (e.g., benefit levels, plan statuses).

### Data Touched

Files declared and read:
- HXPLVL1–HXPLVL6 (physical level tables).
- HXFLVL1–HXFLVL6 (runtime formats).

No PHI fields are directly stored in these configuration tables; they are strictly structural.

---

## XFXTABL

### Overview

XFXTABL is an RPGLE program in the Data Maintenance domain that handles dictionary table lookups. It contains 4 rules with an average confidence of 90% and reads multiple dictionary formats.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE workflow service

### Business Rules Applied

Key rules:
- BR-013 – "When *IN79 equals on/active, branch to 'EXIT'" (confidence 0.9)
- BR-014 – "When *IN79 equals on/active, branch to 'EXIT'" (confidence 0.9)
- BR-015 – "When *IN79 equals on/active, branch to 'EXIT'" (confidence 0.9)
- BR-016 – "When *IN79 equals on/active, branch to 'EXIT'" (confidence 0.9)

These rules use indicator *IN79 to detect an active condition and short-circuit dictionary operations when appropriate.

### Related Approved Rules by Domain

XFXTABL’s high-confidence rules complement XFXLDSC’s mapping logic by guarding dictionary operations based on active flags.

### Data Touched

Files read:
- XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4 via HXPTABLD and logical files HXLTABLD, HXLTABLP, HXLTABLS.

These dictionary tables are not PHI-bearing; they contain configuration codes and descriptors.

---

## HABADTE

### Overview

HABADTE is the main RPGLE program in the Patient Management domain. It drives the admission screening process, calling multiple utilities and reading/writing several PHI-bearing files. It contains 3 rules with an average confidence of 97% and has the highest complexity score in the suite (cc = 152).

### Domain and Program Type

- Domain: PATIENT_MANAGEMENT
- Type: RPGLE main driver

### Business Rules Applied

Key rules:
- BR-018 – "When -FLAG INDICATOR equals void/voided, branch to 'SKIP'" (confidence 0.99)
- BR-019 – "When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'" (confidence 0.99)
- BR-017 – "When -FILE INDICATOR equals zero, branch to 'SKIP'" (confidence 0.92)

These rules form the core admission filter, ensuring that only active, non-void, inpatient records are processed.

### Related Approved Rules by Domain

HABADTE relies on and orchestrates Data Maintenance rules from:
- XFXCYMD (BR-003–BR-008) for date validation.
- XFXLDSC (BR-009–BR-012) for level mappings.
- XFXTABL (BR-013–BR-016) for dictionary checks.
- XFXCNTR (BR-001–BR-002) for loop control.

### Data Touched

HABADTE interacts with:
- HAPTRFR (transfer records; PHI fields AFACCT, AFMRNO).
- XFFNSTN / OXPNSTN / HXPNSTN (plan status tables).
- HMLMAST5H (historical master via TMPMAST).
- HXPBNFIT (benefit logical file over TXPBNFIT).
- HAPIRNK (logical file over TAPIRNK).
- HXFXMLH/HXFXMLD (XML header/detail workfiles) via HXPXMLD and HXPXMLR.

PHI involvement:
- AFACCT, AFMRNO (HAPTRFR) – account number and MRN.
- MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN (OMPMAST) – patient master identifiers and sensitive data.
- CCMRNO, XCNAME, WBDATE, ENNAME, HVACCT (HXPDICT) – MRNs, names, birth dates and accounts.

---

## HXXAPPPRF

### Overview

HXXAPPPRF is an SQLRPGLE program in the Data Maintenance domain that manages application profiles through direct SQL access. It is called by XFXMRNROL and uses copybooks HXXCNTRL and HXXAPPPRFP.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: SQLRPGLE service

### Business Rules Applied

Key rules:
- BR-020 – "SQL program accesses table 'HXPAPPPRF'" (confidence 0.65)
- Additional rules (BR-021, BR-022) – access to tables like 'HXPAPPL6'.

These rules indicate that application profiles are centralized in SQL tables, which control how MRN roll-ups and admission profiles behave.

### Related Approved Rules by Domain

HXXAPPPRF participates in broader profile maintenance, supporting HABADTE indirectly via XFXMRNROL.

### Data Touched

Tables and copybooks:
- HXPAPPPRF, HXPAPPL6 (application profile tables).
- HXXCNTRL, HXXAPPPRFP (control and layout copybooks).

PHI: Profile tables may reference MRN or account identifiers; however, no specific PHI-tagged fields are listed in compact schema.

---

## XFXGETID

### Overview

XFXGETID is an RPGLE program in the Data Maintenance domain with cyclomatic complexity 1. It retrieves identifiers from XML response records and uses copybook HXXLDA.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE utility

### Business Rules Applied

No explicit key_rules are listed, indicating simple logic (e.g., fetch ID, check for null/empty).

### Data Touched

- HXPXMLR / HXFXMLR (XML response workfile; keys XMRUSR, XMRSEQ, XMRID).

PHI: No PHI fields are flagged on HXPXMLR/HXFXMLR; they hold technical identifiers and response codes.

---

## XFXLEAP

### Overview

XFXLEAP is an RPGLE program in the Data Maintenance domain with cyclomatic complexity 1. It provides leap-year logic for XFXCYMD, determining the number of days in February.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE utility

### Business Rules Applied

No explicit key_rules beyond its use inside XFXCYMD; its function is implied by date rules that reference DYS(VMM).

### Data Touched

No direct table access; it computes values based on year input.

---

## XFXMRNROL

### Overview

XFXMRNROL is an RPGLE program in the Data Maintenance domain that coordinates MRN roll-ups and application profile updates. It calls HXHAPPPRF (missing) and HXXAPPPRF.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE workflow service

### Business Rules Applied

No explicit key_rules are listed, but the narrative notes its role in coordinating MRN roll-up and profile updates.

### Data Touched

- HXHAPPPRF (missing program) – likely handles application profile history.
- HXXAPPPRF – SQL-based profile maintenance.

PHI: MRN fields from OMPMAST and HXPDICT are likely involved indirectly through profile tables.

---

## Cross-Program Narrative Summary

The HABADTE suite combines a central patient management driver (HABADTE) with multiple data maintenance utilities (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXLEAP, XFXMRNROL, HXXAPPPRF). HABADTE uses PHI-bearing transfer and master files to assemble admission encounters, while utilities validate dates, control counters, map levels and perform dictionary lookups.

Approved business rules define strict boundaries for valid dates, mapping codes, and active indicators, while HABADTE’s rules focus on filtering out inappropriate encounters (inactive, voided, outpatient). Together, these programs implement a robust admission screening process that can be faithfully migrated to a Java/Spring Boot/SQL Server stack with clear separation of concerns and strong controls over PHI.
