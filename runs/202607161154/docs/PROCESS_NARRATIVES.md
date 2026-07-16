# PROCESS NARRATIVES – HABADTE System

Run ID: 202607161154  
Project: HABADTE

This document provides business-focused narratives for each interpreted program, linking technical behaviour to domain concepts and approved business rules.

---

## Program: XFXCNTR

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXCNTR is a text-centering utility used by other programs to format headings and labels. It inspects an input field and decides whether centering logic should be applied. The goal is to avoid unnecessary processing and preserve existing formatting when appropriate.

### Business Rules Applied

Key rules (from interpretations_detail):
- **BR-001** – Text centering — skip processing if the entire input field is blank (nothing to center).
- **BR-002** – Text centering — skip processing if the text already starts at the first position (already left-aligned).

### Narrative

When a caller passes a text field to XFXCNTR, the program first checks whether the field is completely blank. If so, it returns immediately without altering the field, ensuring that blank headings remain blank instead of being padded with spaces or other characters.

If the field contains text, XFXCNTR then checks the starting position. If the text already begins at the first character (i.e., is left-aligned), the program again returns without applying centering, avoiding cosmetic changes that might disrupt alignment in printed or on-screen reports.

Only when the field contains text that is not left-aligned does XFXCNTR compute a new starting position (typically the midpoint of the available width) and reposition the text to appear centered. This behaviour standardises visual formatting across reports while respecting pre-aligned fields.

### Related Approved Rules

All approved rules for XFXCNTR are purely formatting-related and do not involve PHI or patient data. They support readability of headers in HABADTE and other reporting programs.

### Data Touched

XFXCNTR does not read or write database files directly. It operates entirely in memory on character fields passed by callers, so it has no direct interaction with PHI-bearing tables.

---

## Program: XFXCYMD

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXCYMD is a date validation utility. It enforces bounds on year, month and day components and delegates leap-year-specific logic to XFXLEAP. This ensures that dates used in reporting and calculations are structurally valid and within supported ranges.

### Business Rules Applied

Key rules:
- **BR-003** – Reject dates before 1800.
- **BR-004** – Reject dates beyond 2100.
- **BR-005** – Month must be January (01) or later.
- **BR-006** – Month must be December (12) or earlier.
- **BR-007** – Day must be 01 or later.
- **BR-008** – When VDD is greater than DYS(VMM), branch to 'EXIT'.

### Narrative

For any date value provided by a calling program (such as HABADTE), XFXCYMD splits the numeric representation into year, month and day components. It first checks whether the year lies between 1800 and 2100. If the year is outside this range, the date is rejected and processing branches to an exit path, preventing the use of unrealistic historical or future dates.

Next, the utility validates the month, ensuring it lies between 1 and 12. Months below 1 or above 12 cause immediate rejection. The day component is similarly checked to be at least 1; days of 0 or negative values are treated as invalid.

For certain calculations, XFXCYMD compares derived values (VDD and DYS(VMM)). When the difference exceeds a defined threshold, the program branches to EXIT, signalling that the date combination is not acceptable or that the calculation would produce nonsensical results.

When the date passes all basic validations, XFXCYMD invokes XFXLEAP to handle leap-year-specific validation (e.g., February 29 on leap years). This separation keeps leap-year logic small and focused while leaving general validation in XFXCYMD.

### Related Approved Rules

All rules for XFXCYMD are structural and range validations. They do not reference patient identifiers directly but are essential for HABADTE’s census date comparisons (e.g., admission vs discharge vs census date).

### Data Touched

XFXCYMD operates on date fields supplied by callers. It does not access physical or logical files directly, so it has no direct PHI exposure. Its impact on PHI is indirect: by validating dates, it ensures that PHI-bearing records are associated with valid timelines.

---

## Program: XFXLDSC

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXLDSC performs organisational level description lookups. It reads hierarchical tables HXPLVL1–HXPLVL6 and their corresponding HXFLVL* record formats to translate numeric level codes into human-readable descriptions used in report headers and filters.

### Business Rules Applied

Key rules:
- **BR-009–BR-012** – For each organisational level, if the level code is out of the valid range, return no description.

### Narrative

When HABADTE or another caller requests a description for a particular level (e.g., facility, ward, or nursing unit), XFXLDSC identifies the appropriate table based on the level number (HXPLVL1 for level 1, up to HXPLVL6 for level 6). It then attempts to read the record matching the supplied level code.

If the level code is valid and present, XFXLDSC returns the textual description (such as a facility name or unit name). This description is used in report headers, filters and user interfaces.

If the level code is missing or outside the configured range, the program deliberately returns no description rather than a default or guessed value. This forces calling programs to recognise that they are dealing with an unknown organisational code, which may indicate configuration gaps or data quality issues.

### Related Approved Rules

BR-009–BR-012 capture the "no description for invalid code" behaviour across multiple hierarchy levels, ensuring consistency. These rules support HABADTE’s organisational filter BR-020 and contribute to data minimisation by ensuring that only properly scoped organisational entities are displayed.

### Data Touched

Files:
- **HXPLVL1–HXPLVL6** (physical files) and **HXFLVL1–HXFLVL6** (record formats) – organisational hierarchy.

These tables do not carry PHI; they provide structural context. XFXLDSC thus operates only on non-PHI reference data.

---

## Program: XFXTABL

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXTABL is a dictionary and status lookup program. It reads HXPTABLD and associated DDS views to resolve data type codes, mapping codes, logical descriptions and status descriptions. It also uses an indicator (*IN79) as a control flag to exit processing when certain conditions are met.

### Business Rules Applied

Key rules:
- **BR-013–BR-016** – When *IN79 equals on/active, branch to EXIT.

### Narrative

XFXTABL is invoked when callers need to translate compact codes into human-readable descriptions or behaviours. For example, HABADTE relies on XFXTABL to resolve room class codes and mapping fields that drive leave flags.

The program iterates over dictionary records in XFFTABLD and related tables. During processing, an internal indicator (*IN79) reflects whether a particular condition has been met—such as finding a terminating entry, encountering a configuration flag, or reaching an upper bound in a loop.

When *IN79 becomes "on" or "active", XFXTABL immediately branches to an EXIT path, stopping further processing. This pattern prevents redundant lookups and enforces business constraints encoded in the dictionary tables.

### Related Approved Rules

BR-013–BR-016 demonstrate that the exit-on-indicator pattern appears multiple times within the program, reinforcing its importance in dictionary-driven logic.

### Data Touched

Files:
- **HXPTABLD** and logical files **HXLTABLD/HXLTABLP/HXLTABLS** – dictionary and status tables.

These tables themselves do not carry PHI fields in the compact schema; they represent configuration and mapping data.

---

## Program: HXXAPPPRF

**Domain:** DATA_MAINTENANCE  
**Program Type:** SQLRPGLE

### Overview

HXXAPPPRF provides table-driven application preferences. It uses embedded SQL to read configuration tables such as HXPAPPPRF and HXPAPPL6, returning preference values that influence behaviour in programs like XFXMRNROL and HABADTE.

### Business Rules Applied

Key rules:
- **BR-030** – Reads from HXPAPPPRF to retrieve configuration or reference data.
- **BR-031** – Reads from HXPAPPL6 for level-specific configuration.
- **BR-032** – Additional reads from HXPAPPPRF for processing.

### Narrative

When HABADTE needs to determine how to display patient identifiers or apply facility-specific options, it calls XFXMRNROL, which in turn calls HXXAPPPRF (and the missing HXHAPPPRF). HXXAPPPRF runs SQL queries against preference tables to obtain values such as "display MRN vs account", default facilities, or override behaviours for specific organisations.

These preferences are table-driven rather than hard-coded, allowing administrators to adjust behaviour without changing program source. The SQLRPGLE implementation indicates that HXXAPPPRF is tightly coupled to database schema and is a natural candidate for migration into a configuration service.

### Related Approved Rules

The three rules highlight reliance on both a main preference table and a level-specific variant, supporting multi-level configuration.

### Data Touched

Files (inferred from rules):
- **HXPAPPPRF**, **HXPAPPL6** – preference tables.

Compact PHI data does not explicitly list PHI fields in these tables, but the preferences affect how PHI (MRN/account) is displayed. Therefore, HXXAPPPRF indirectly influences PHI exposure.

---

## Program: XFXGETID

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXGETID is an ID retrieval utility. It declares XML reference file HXPXMLR and reads HXFXMLR to obtain identifiers used as batch and sequence keys in HABADTE’s XML output.

### Business Rules Applied

No explicit BR-IDs are recorded in key_rules for XFXGETID; its behaviour is structural rather than rule-rich.

### Narrative

Before HABADTE writes XML header and detail records, it needs a batch identifier and possibly per-record sequence IDs. XFXGETID provides these by reading from HXFXMLR, which stores user-based sequences and IDs.

The program likely performs a read, increments a sequence if necessary, and returns the new value to HABADTE. This helps ensure that each census run has a unique batch ID and that detail records are uniquely addressable.

### Related Approved Rules

None directly; XFXGETID’s importance is architectural, supporting identity and traceability of XML outputs.

### Data Touched

Files:
- **HXPXMLR/HXFXMLR** – XML reference table.

These tables do not carry PHI in compact schema and primarily store control identifiers.

---

## Program: XFXLEAP

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXLEAP is a small utility that supports leap-year logic for date validation. It is called by XFXCYMD to ensure that February 29 is only accepted on leap years.

### Business Rules Applied

No explicit BR-IDs; behaviour is derived from its narrative and its narrow responsibility.

### Narrative

Upon receiving a year and month/day combination from XFXCYMD, XFXLEAP determines whether the year is a leap year and whether the day component is valid for that month. For example, February 29 is permitted only when the year is divisible by 4 and not by 100, unless divisible by 400.

If a date falls outside leap-year rules, XFXLEAP signals back to XFXCYMD to reject the date. This protects the system from subtle date errors that could impact length-of-stay calculations, billing periods, and census alignment.

### Data Touched

XFXLEAP does not touch any database files; it operates purely on date parameters.

---

## Program: XFXMRNROL

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Overview

XFXMRNROL orchestrates patient identifier preferences, deciding whether MRN or account numbers are used in outputs. It calls HXHAPPPRF (missing) and HXXAPPPRF to read preference tables.

### Business Rules Applied

No explicit BR-IDs in key_rules; behaviour is inferred from function and dependencies.

### Narrative

When HABADTE needs to display an identifier for a patient, it delegates the decision to XFXMRNROL. This program reads configuration via HXHAPPPRF/HXXAPPPRF to determine:
- Whether MRN should be the primary identifier for the facility.
- Whether account numbers should be used instead, for billing-oriented views.
- Any facility-level overrides or exceptions.

Based on these preferences, XFXMRNROL selects the correct field from patient master or dictionary tables (e.g., MMMRNO vs MMACCT) and returns it to HABADTE. This ensures consistent identifier display across reports and supports PHI governance by allowing facilities to choose the least-sensitive identifier where appropriate.

### Data Touched

Indirectly touches PHI because it influences MRN vs account usage. Direct file access is through preference tables (HXPAPPPRF/HXPAPPL6) via HXHAPPPRF/HXXAPPPRF.

---

## Program: HABADTE

**Domain:** PATIENT_MANAGEMENT  
**Program Type:** RPGLE

### Overview

HABADTE is the core patient census driver. It reads patient admissions via TMPMAST/HMLMAST5H, applies multiple inclusion and exclusion rules, enriches records with room, station and preference-based identifiers, and writes XML header/detail records and summary counts.

### Key Business Rules Applied

From interpretations_detail and approved_rules:
- **BR-017** – Exclude pre-admitted patients (file indicator = 0).
- **BR-018** – Exclude voided accounts (flag = 'V').
- **BR-019** – Exclude outpatients (I/O indicator = 'O').
- **BR-020** – Organisational level filter for orgLevel/orgCode.
- **BR-021** – Census date vs admission date check.
- **BR-022** – Census date vs discharge date check.
- **BR-023** – Room number retrieval from transfer history.
- **BR-024** – Room class description lookup via dictionary tables.
- **BR-025** – Hospital/therapeutic leave flag from mapping field.
- **BR-026** – Nursing station name resolution.
- **BR-027** – MRN vs account display choice.
- **BR-028** – Application preference lookup with facility override.
- **BR-029** – Patient counting (three running totals).

### Narrative

At the start of processing, HABADTE determines the census context: census date, organisational level and code, nursing station, and user/facility preferences. It calls XFXMRNROL/HXXAPPPRF to resolve whether MRN or account numbers should be presented.

It then iterates through patient master records via the HMLMAST5H logical file, which presents admissions keyed by nursing station and date/time. For each patient, it applies a series of filters:
- Pre-admission records (file indicator = 0) are dropped.
- Voided accounts (flag = 'V') are removed.
- Outpatient records (I/O indicator = 'O') are excluded to keep the census inpatient-only.
- Organisational filters ensure that only patients belonging to the requested facility/unit appear.
- Admissions after the census date are excluded; discharges on or before the census date remove patients from the active set.

For patients who pass all filters, HABADTE performs enrichment. It reads HAPTRFR (transfer history) to determine current room and location by selecting the latest transfer up to the census date. It uses XFXTABL and dictionary tables to resolve room class descriptions and leave flags. Nursing station names are retrieved from HXPNSTN/TXPNSTN based on level and station codes.

Throughout processing, HABADTE maintains three counters capturing total active patients, inpatients and leave cases. These counters are written alongside XML header and detail records to HXFXMLH/HXFXMLD, establishing a complete, machine-readable representation of the census. The results can be printed or consumed by downstream systems.

### Related Approved Rules

HABADTE concentrates most patient-management rules in the system, and its logic drives the main BRD and backlog for modernisation.

### Data Touched

Relevant files from data_dict_schema and phi_flagged_fields:
- **TMPMAST/OMPMAST** – patient master (PHI: MRN, account, name, SSN).
- **HAPTRFR** – transfer history (PHI: AFACCT, AFMRNO).
- **HXPTABLD/HXLTABL* ** – dictionary tables for room class and status.
- **TXPNSTN/HXPNSTN** – nursing station reference.
- **HXPXMLR/HXPXMLD/HXFXMLR/HXFXMLD** – XML header/detail tables.

The program is a primary PHI consumer; all modernised implementations must enforce strong access controls and auditing.

---

## Program: Summary – PHI and Process Context

Across all interpreted programs, HABADTE is the central orchestrator for patient census. XFX* utilities provide reusable services for text formatting, date validation, organisational lookup, dictionary mapping, ID generation and identifier preferences. HXXAPPPRF and the missing HXHAPPPRF root configuration in database tables.

PHI is concentrated in patient master and transfer history files, with HABADTE responsible for enforcing business rules that determine which patients appear in the census and how their identifiers are displayed. Utilities and preference services shape supporting behaviour but do not directly manipulate PHI beyond influencing identifier selection.
