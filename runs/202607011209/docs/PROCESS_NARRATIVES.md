# Process Narratives – HABADTE Project

This document describes program-level narratives derived from semantic interpretations and approved business rules.

---

## XFXCNTR

### Overview

XFXCNTR is an RPGLE utility program in the DATA_MAINTENANCE domain. It implements simple counter-based logic used to control loop execution and iteration limits. The interpretation shows it contains two primary rules with relatively low confidence, indicating straightforward but important guard conditions.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

### Business Rules Applied

From `interpretations_detail` for XFXCNTR:

- BR-001 – "When X equals zero, branch to 'EXIT'" (confidence 0.47).
- BR-002 – "When X equals 40, branch to 'EXIT'" (confidence 0.4).

These rules bound the internal counter `X` and determine when loops should terminate.

### Related Approved Rules

Within the DATA_MAINTENANCE domain, XFXCNTR is related to a wider set of validation rules:

- Date validation rules from XFXCYMD (BR-003–BR-008).
- Map validation rules from XFXLDSC (BR-009–BR-012).
- Indicator-based guards from XFXTABL (BR-013–BR-016).

Together, these rules ensure that utility routines enforce strict boundaries on data and control flow.

### Data Touched

XFXCNTR does not directly access PHI-tagged files in the aggregated lineage; its logic is internal to the program and acts as a control mechanism rather than a data access layer. As such, it should be implemented as a stateless utility service in modernization, with no direct PHI exposure.

---

## XFXCYMD

### Overview

XFXCYMD is an RPGLE program in the DATA_MAINTENANCE domain that validates calendar dates. It enforces acceptable ranges for year, month, and day values before downstream logic operates on them. With six rules and an average confidence of 58%, it provides robust but slightly inferred behavior.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

### Business Rules Applied

Key rules from `interpretations_detail`:

- BR-008 – "When VDD is greater than DYS(VMM), branch to 'EXIT'" (confidence 0.68).
- BR-003 – "When VYY is less than 1800, branch to 'EXIT'" (confidence 0.56).
- BR-004 – "When VYY is greater than 2100, branch to 'EXIT'" (confidence 0.56).
- BR-005 – "When VMM is less than 01, branch to 'EXIT'" (confidence 0.56).
- BR-006 – "When VMM is greater than 12, branch to 'EXIT'" (confidence 0.56).

These rules ensure that only valid Gregorian dates are passed to caller programs.

### Related Approved Rules

All XFXCYMD rules are part of the DATA_MAINTENANCE domain and are conceptually linked to XFXLEAP (for leap-year support) and HABADTE’s date-dependent logic. HABADTE relies on these validations to avoid processing transfers with invalid dates.

### Data Touched

XFXCYMD primarily manipulates in-memory date components and does not read or write database files directly. Its effects are indirect: by rejecting invalid dates, it prevents invalid AFTRDT/AFTRTM combinations from influencing transfer processing.

---

## XFXLDSC

### Overview

XFXLDSC is an RPGLE program in the DATA_MAINTENANCE domain that builds level descriptors using multi-level configuration tables. It has four rules controlling an LDAMAP value, with an average confidence of 56%. The program participates in enriching records with hierarchical descriptors.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

### Business Rules Applied

From `interpretations_detail`:

- BR-009 – "When LDAMAP is greater than 99, branch to 'EXIT'".
- BR-010 – "When LDAMAP is greater than 99, branch to 'EXIT'".
- BR-011 – "When LDAMAP is greater than 99, branch to 'EXIT'".
- BR-012 – "When LDAMAP is greater than 9999, branch to 'EXIT'".

These rules bound mapping identifiers and prevent the program from working with undefined descriptor maps.

### Related Approved Rules

XFXLDSC’s rules interact conceptually with XFXTABL (table lookups) and HABADTE’s enrichment logic:

- Map limits ensure that enrichment uses valid configuration records from HXPLVL1–HXPLVL6.
- In HABADTE, invalid LDAMAP values result in skipped descriptor enrichment or whole-record skips.

### Data Touched

Derived from `data_dict_schema` and `data_lineage`:

- HXPLVL1–HXPLVL6 physical files provide hierarchical configuration data.
- HXFLVL1–HXFLVL6 record formats are read by XFXLDSC.

None of these level configuration files contain PHI-tagged fields, so XFXLDSC operates exclusively on non-sensitive configuration data.

---

## XFXTABL

### Overview

XFXTABL is an RPGLE program in the DATA_MAINTENANCE domain that performs generic table lookups against dictionary files. It has four high-confidence rules related to indicator *IN79, acting as a global guard for lookup behavior.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

### Business Rules Applied

Key rules from `interpretations_detail`:

- BR-013 – "When *IN79 equals on/active, branch to 'EXIT'".
- BR-014 – "When *IN79 equals on/active, branch to 'EXIT'".
- BR-015 – "When *IN79 equals on/active, branch to 'EXIT'".
- BR-016 – "When *IN79 equals on/active, branch to 'EXIT'".

These rules ensure that when the global indicator is on, certain lookup paths are disabled.

### Related Approved Rules

The XFXTABL guards complement threshold checks in XFXLDSC and date validations in XFXCYMD. Collectively, they protect the system against misconfigurations in code tables and date ranges.

### Data Touched

From `data_dict_schema` and `data_lineage`:

- HXPTABLD (PF) with record format XFFTABLD, and its logical projections HXLTABLD, HXLTABLP, HXLTABLS.
- XFXTABL reads XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4.

These dictionary tables are non-PHI and store reference codes and descriptions. In modernization, XFXTABL’s behavior maps to service-based dictionary lookups.

---

## HABADTE

### Overview

HABADTE is a high-complexity RPGLE program in the PATIENT_MANAGEMENT domain. It acts as the central batch driver for inpatient transfer processing. The interpretation reports three high-confidence rules and a narrative indicating that it coordinates multiple utility programs and file operations.

### Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT
- **Program Type:** RPGLE

### Business Rules Applied

From `interpretations_detail`:

- BR-018 – "When -FLAG INDICATOR equals void/voided, branch to 'SKIP'" (confidence 0.99).
- BR-019 – "When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'" (confidence 0.99).
- BR-017 – "When -FILE INDICATOR equals zero, branch to 'SKIP'" (confidence 0.92).

These rules shape the core inclusion/exclusion criteria for the batch.

### Related Approved Rules

HABADTE also depends on rules from utility programs:

- Date validation rules in XFXCYMD (BR-003–BR-008) to ensure valid transfer dates.
- Descriptor map rules in XFXLDSC (BR-009–BR-012) for level configuration.
- Table guard rules in XFXTABL (BR-013–BR-016) controlling dictionary lookups.
- Counter rules in XFXCNTR (BR-001–BR-002) for loop termination.

Together, these programs form a rule network that controls data quality, configuration integrity, and batch flow.

### Data Touched

From `data_dict_schema` and PHI registry:

- **HAPTRFR** – transfer records (AFACCT, AFMRNO flagged as AccountNumber and MRN). PHI-sensitive.
- **OXPNSTN/HXPNSTN lineage** – institution/station data, non-PHI.
- **HXPBNFIT/OXPBNFIT lineage** – benefit data (XFBTEL flagged as PhoneNumber; PHI).
- **OMPMAST** – member master (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN; PHI).
- **HXPDICT** – large dictionary/mapping file with multiple PHI fields.
- **HXPXMLH/HXPXMLD** – XML header and detail records (carry PHI fields from transfers and masters).

HABADTE orchestrates reading of PHI-bearing files and writing of PHI-bearing XML outputs. This makes it a focal point for security and compliance in modernization. Access to HABADTE’s modern equivalents must be strictly controlled and audited.

---

# Summary

The narratives above show a clear separation of concerns:

- Utility programs in DATA_MAINTENANCE (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL) enforce validation and configuration rules without directly handling PHI.
- HABADTE, in PATIENT_MANAGEMENT, applies these utility rules to PHI-bearing transfer and master data, deciding which records become part of inpatient integration feeds.

In modernization, these programs should be re-expressed as discrete services and modules, preserving rule semantics while introducing stronger security and observability around HABADTE’s PHI processing.
