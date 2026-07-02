# PROCESS NARRATIVES – HABADTE Project

This document describes the business process narratives for key programs in the HABADTE codebase, using the semantic interpretations and approved rules.

## XFXCNTR – Counter Exit Logic

### Overview

XFXCNTR is an RPGLE program in the **DATA_MAINTENANCE** domain. Its narrative describes a small utility that manages counter-based exit conditions, with 2 rules and an average confidence of 43%. It is called multiple times by HABADTE and acts as a reusable decision point for loop and threshold control.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

Key rules (from interpretations_detail):
- **BR-001** – When X equals zero, branch to "EXIT" (confidence 0.47).
- **BR-002** – When X equals 40, branch to "EXIT" (confidence 0.40).

Related approved_rules in the DATA_MAINTENANCE domain:
- BR-001 and BR-002 as above.

The program compares a counter value and decides whether to terminate processing. A value of 0 short-circuits further work, and 40 represents a maximum threshold.

### Data Touched

XFXCNTR operates purely on in-memory counters and does not directly read or write any DDS files. It participates in the HABADTE process as a control utility rather than a data access component.

PHI involvement: none, since no PHI-bearing files are accessed.

---

## XFXCYMD – Calendar Date Validation

### Overview

XFXCYMD is an RPGLE program in the **DATA_MAINTENANCE** domain. Its narrative indicates it contains 6 rules with an average confidence of 58%. It validates year, month, and day components of dates before they are used elsewhere in the system.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

Key rules (from interpretations_detail):
- **BR-008** – When VDD is greater than DYS(VMM), branch to "EXIT" (confidence 0.68).
- **BR-003** – When VYY is less than 1800, branch to "EXIT" (confidence 0.56).
- **BR-004** – When VYY is greater than 2100, branch to "EXIT" (confidence 0.56).
- **BR-005** – When VMM is less than 01, branch to "EXIT" (confidence 0.56).
- **BR-006** – When VMM is greater than 12, branch to "EXIT" (confidence 0.56).

Related approved_rules:
- BR-003–BR-008, all in DATA_MAINTENANCE.

XFXCYMD applies these rules to enforce valid calendar dates, and calls XFXLEAP to determine the number of days in a given month, accounting for leap years.

### Data Touched

XFXCYMD works on date values passed from callers and does not directly access files. However, the dates it validates are subsequently used in HAPTRFR (transaction date AFTRDT) and HXPDICT (birth date WBDATE) for patient-related processing.

PHI involvement: indirect, since date fields like WBDATE (DateOfBirth) are validated but not directly read by this program.

---

## XFXLDSC – Level Description Mapping

### Overview

XFXLDSC is an RPGLE program in the **DATA_MAINTENANCE** domain. Its narrative summarises 4 rules with an average confidence of 56%. It maps level codes to human-readable descriptions using the HXPLVL1–HXPLVL6 configuration hierarchy and enforces constraints on mapping codes.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

Key rules (from interpretations_detail):
- **BR-009** – When LDAMAP is greater than 99, branch to "EXIT" (confidence 0.56).
- **BR-010** – When LDAMAP is greater than 99, branch to "EXIT" (confidence 0.56).
- **BR-011** – When LDAMAP is greater than 99, branch to "EXIT" (confidence 0.56).
- **BR-012** – When LDAMAP is greater than 9999, branch to "EXIT" (confidence 0.56).

Related approved_rules:
- BR-009–BR-012.

Together, these rules ensure that mapping codes stay within valid ranges, preventing misconfiguration that could corrupt level descriptions.

### Data Touched

XFXLDSC reads from level configuration files:
- HXPLVL1–HXPLVL6 (physical files) and their corresponding HXFLVL* record formats.

These files store hierarchical organisational data (facility, region, unit, ward). No PHI fields are flagged in these structures.

PHI involvement: none—level configuration is non-PHI metadata.

---

## XFXTABL – Table Dictionary Lookup

### Overview

XFXTABL is an RPGLE program in the **DATA_MAINTENANCE** domain. Its narrative identifies 4 rules with an average confidence of 90%. It performs table lookups against the HXPTABLD dictionary and related logical files to transform codes into descriptive values and parameters.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

Key rules (from interpretations_detail):
- **BR-013** – When *IN79 equals on/active, branch to "EXIT" (confidence 0.9).
- **BR-014** – When *IN79 equals on/active, branch to "EXIT" (confidence 0.9).
- **BR-015** – When *IN79 equals on/active, branch to "EXIT" (confidence 0.9).
- **BR-016** – When *IN79 equals on/active, branch to "EXIT" (confidence 0.9).

Related approved_rules:
- BR-013–BR-016.

XFXTABL uses indicator *IN79 as a gate: when active, table lookup logic is bypassed and the program exits early. This allows configuration to disable particular transformations without code changes.

### Data Touched

XFXTABL accesses:
- HXPTABLD (physical file) and possibly logical views HXLTABLD, HXLTABLP, HXLTABLS.

These table files store non-PHI configuration codes and descriptions.

PHI involvement: none—HXPTABLD does not contain PHI fields.

---

## HABADTE – Patient Management Orchestrator

### Overview

HABADTE is an RPGLE program in the **PATIENT_MANAGEMENT** domain. Its narrative notes 3 high-confidence rules with an average confidence of 97%. It acts as the main orchestrator, reading transaction data, applying inclusion/exclusion criteria, and coordinating enrichment and XML output.

### Domain and Program Type

- **Domain**: PATIENT_MANAGEMENT
- **Program Type**: RPGLE

### Business Rules Applied

Key rules (from interpretations_detail):
- **BR-018** – When -FLAG INDICATOR equals void/voided, branch to "SKIP" (confidence 0.99).
- **BR-019** – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to "SKIP" (confidence 0.99).
- **BR-017** – When -FILE INDICATOR equals zero, branch to "SKIP" (confidence 0.92).

Related approved_rules for the PATIENT_MANAGEMENT domain:
- BR-017, BR-018, BR-019.

These rules determine which transactions are eligible for processing. HABADTE:
1. Reads transaction records from HAPTRFR.
2. Applies BR-017 to skip records with inactive file indicators.
3. Applies BR-018 to skip voided records.
4. Applies BR-019 to skip outpatient records when the batch is targeted at inpatients.
5. For remaining records, calls XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, and XFXCNTR to perform date validation, level mapping, table lookup, routing, and counter checks.
6. Writes XML headers and details to HXFXMLH/HXFXMLD and may interact with notification data via XFFNSTN.

### Data Touched

HABADTE interacts with multiple data structures:
- **PHI-bearing files** (from phi_flagged_files):
  - HAPTRFR – transaction file with AFACCT (AccountNumber) and AFMRNO (MRN).
  - HXPDICT – dictionary with MRNs, names, phone numbers, accounts, and birthdates.
  - OAPIRNK – ranking file with BRKMRN (MRN).
  - OMPMAST – patient master with MRN, account, name, SSN.
  - OXPBNFIT – benefits file with phone numbers.
- **Non-PHI configuration**:
  - HXPLVL1–HXPLVL6 – level configuration.
  - HXPTABLD – table dictionary.
  - HXPXMLD/HXPXMLR/HXFXMLH/HXFXMLD – XML detail and routing.
  - HXPNSTN/XFFNSTN – statement/notification status.

PHI involvement is substantial: HABADTE sits on top of patient transaction and master data, and its outputs include patient identifiers and account information in XML messages.

---

## XFXGETID – XML Routing and Identifier Utility

### Overview

Although not explicitly described in interpretations_detail beyond domain/type, XFXGETID is an RPGLE utility in the XFX subsystem, used by HABADTE to resolve XML routing identifiers.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE (utility context)
- **Program Type**: RPGLE

### Business Rules Applied

XFXGETID does not have standalone BR-IDs in the approved_rules list but participates indirectly in HABADTE’s routing logic.

Typical behavior inferred from data_lineage:
- Declares and reads from HXPXMLR and HXFXMLR.
- Resolves user/sequence-based identifiers for XML messages.

### Data Touched

- HXPXMLR/HXFXMLR – XML route definitions, no PHI flagged.

PHI involvement: none directly; routing metadata is non-PHI, although it controls distribution of PHI-containing messages.

---

## Summary

Across these programs, the HABADTE application separates concerns into:
- **Control and validation utilities** (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID).
- **Core business orchestrator** (HABADTE) operating over PHI-rich transaction and master data.

Understanding these narratives provides a foundation for migrating business logic into Java/Spring Boot services while preserving rule behavior and protecting sensitive data.
