# Process Narratives

Project: HABADTE

This document summarizes the interpreted business processes for each RPG program, using extracted narratives and key rules, and relating them to approved business rules and data structures.

## Program: XFXCNTR

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 2 rule(s) with avg confidence 43%.

XFXCNTR appears to be a counter or range enforcement utility used to manage numeric control values for other processes. It implements simple boundary checks to determine when processing should be allowed to continue.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Program Type: RPGLE

### Business Rules Applied (Key Rules)

- **BR-001** – When X equals zero, branch to 'EXIT' (confidence 0.47).
- **BR-002** – When X equals 40, branch to 'EXIT' (confidence 0.40).

These rules ensure that the controlled value remains within an acceptable range and that boundary conditions are handled explicitly.

### Related Approved Rules

From the global approved rules list for domain DATA_MAINTENANCE:
- BR-001 (XFXCNTR)
- BR-002 (XFXCNTR)

### Data Touched

XFXCNTR operates primarily on in-memory variables and counters. It does not directly reference PHI-bearing files in the extracted context. Any data persisted or retrieved would be configuration-oriented rather than patient-specific.

---

## Program: XFXCYMD

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 6 rule(s) with avg confidence 58%.

XFXCYMD is a calendar/date validation utility. It validates year, month, and day components, ensuring they fall within supported ranges and correspond to valid calendar dates before downstream processing uses them.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Program Type: RPGLE

### Business Rules Applied (Key Rules)

- **BR-008** – When VDD is greater than DYS(VMM), branch to 'EXIT' (confidence 0.68).
- **BR-003** – When VYY is less than 1800, branch to 'EXIT' (confidence 0.56).
- **BR-004** – When VYY is greater than 2100, branch to 'EXIT' (confidence 0.56).
- **BR-005** – When VMM is less than 01, branch to 'EXIT' (confidence 0.56).
- **BR-006** – When VMM is greater than 12, branch to 'EXIT' (confidence 0.56).

Together these rules prevent invalid dates from being accepted, reducing the risk of corrupting time-series logic in patient and financial processes.

### Related Approved Rules

- BR-003–BR-008 (XFXCYMD) in domain DATA_MAINTENANCE.

### Data Touched

XFXCYMD works on date components provided as parameters or local fields. It does not touch PHI-bearing files directly; its role is to sanitize date values used by higher-level programs such as HABADTE.

---

## Program: XFXLDSC

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 56%.

XFXLDSC appears to manage load descriptions and mapping codes, enforcing upper bounds on mapping identifiers that may be used to link hierarchy levels or configuration records.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Program Type: RPGLE

### Business Rules Applied (Key Rules)

- **BR-009** – When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-010** – When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-011** – When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-012** – When LDAMAP is greater than 9999, branch to 'EXIT' (confidence 0.56).

These rules ensure that mapping codes remain within an expected numeric range, guarding against configuration errors.

### Related Approved Rules

- BR-009–BR-012 (XFXLDSC) in domain DATA_MAINTENANCE.

### Data Touched

XFXLDSC interacts with hierarchy level files **HXPLVL1–HXPLVL6** and may use the HXPTABLD/HXLTABLD logical views to resolve descriptions. These files are non-PHI and serve purely structural roles.

---

## Program: XFXTABL

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 90%.

XFXTABL is a table-driven configuration engine. It uses indicator *IN79 to determine whether processing should be short-circuited, likely based on whether a configuration segment has already been processed or is active.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Program Type: RPGLE

### Business Rules Applied (Key Rules)

- **BR-013** – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).
- **BR-014** – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).
- **BR-015** – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).
- **BR-016** – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).

These rules enable efficient control flow by exiting as soon as a particular configuration state is active.

### Related Approved Rules

- BR-013–BR-016 (XFXTABL) in domain DATA_MAINTENANCE.

### Data Touched

XFXTABL reads from table/data dictionary structures such as **HXPTABLD** and its logical views **HXLTABLD**, **HXLTABLP**, and **HXLTABLS**, along with their record format XFFTABLD and related files. These tables are configuration-oriented and do not contain PHI.

---

## Program: HABADTE

### Overview

RPGLE program in domain 'Patient Management'. Contains 3 rule(s) with avg confidence 97%.

HABADTE is a high-complexity patient management program responsible for generating a detailed extract of inpatient transfer activity. It coordinates multiple data sources, including the HAPTRFR transfer file, benefit plan lookups, patient status tables, and optional printer/XML output structures.

### Domain and Program Type

- Domain: PATIENT_MANAGEMENT
- Program Type: RPGLE

### Business Rules Applied (Key Rules)

- **BR-018** – When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99).
- **BR-019** – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99).
- **BR-017** – When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92).

These rules collectively determine which transfer records are eligible for inclusion in the inpatient extract. They protect data quality by excluding invalid, voided, and outpatient records.

### Related Approved Rules

From the approved rules list for PATIENT_MANAGEMENT:
- BR-017 (HABADTE) – file indicator skip.
- BR-018 (HABADTE) – voided flag skip.
- BR-019 (HABADTE) – outpatient flag skip.

### Data Touched

HABADTE interacts with multiple PHI-bearing and structural files:

- **HAPTRFR** (physical file)
  - Key fields: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE.
  - PHI fields: AFACCT (AccountNumber), AFMRNO (MRN).

- **HXPBNFIT / OXPBNFIT** (benefit plan files)
  - Key fields: XFBUBN, XFBPLN.
  - PHI field: XFBTEL (PhoneNumber).

- **HXPNSTN / OXPNSTN** (patient status files)
  - Key fields: XFNLV6, XFNSST.
  - Provide status codes and descriptions, non-PHI.

- **HXPXMLD / HXFXMLD / HXFXMLH / ****HXPXML** (XML and printer-related files)
  - Used for XML output and printer routing; structural, not directly PHI.

- **HMLMAST5H / TMPMAST / OMPMAST** (patient master context)
  - PHI fields: MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN).

- **HAPIRNK / TAPIRNK / OAPIRNK** (risk ranking)
  - PHI field: BRKMRN (MRN) in OAPIRNK.

PHI-flagged files in the broader system (from the data dictionary schema and PHI registry) include:
- OXPBNFIT
- HXPDICT
- OAPIRNK
- OMPMAST
- HAPTRFR

HABADTE must be implemented with strict PHI access control and auditing when migrated.

---

## Summary

The interpreted programs form a layered architecture:
- **Data Maintenance utilities** (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL) validate core data (counters, dates, mapping codes, configuration indicators) and operate on non-PHI configuration tables.
- **Patient Management core** (HABADTE) orchestrates multiple data sources, applies high-confidence inclusion/exclusion rules for inpatient transfers, and performs enrichment using benefit and status files while touching PHI-bearing fields.

These narratives should guide the migration team in preserving business intent, ensuring that validation utilities remain reusable services and that patient-facing processes like HABADTE enforce the same quality and safety constraints in the new Java / Spring Boot / SQL Server implementation.
