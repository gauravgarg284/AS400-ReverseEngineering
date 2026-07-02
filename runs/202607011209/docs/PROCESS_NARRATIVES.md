# Process Narratives – HABADTE Application

## Program XFXCNTR

### Overview

RPGLE program in domain "Data Maintenance". Contains 2 rule(s) with average confidence 43%. XFXCNTR manages numeric counters or control values and enforces boundaries before downstream logic executes.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

- BR-001 (confidence 0.47): When X equals zero, branch to `EXIT`.
- BR-002 (confidence 0.40): When X equals 40, branch to `EXIT`.

XFXCNTR ensures that counter values outside the active processing range (0 or 40) terminate the current operation.

### Related Approved Rules by Domain

Within DATA_MAINTENANCE, the following rules are relevant to XFXCNTR:

- BR-001–BR-008 from XFXCNTR and XFXCYMD for counter and date validity.
- BR-009–BR-012 from XFXLDSC for mapping bounds.
- BR-013–BR-016 from XFXTABL for indicator-driven table termination.

### Data Touched

XFXCNTR does not directly read physical files in the compact lineage but operates on in-memory counters. It participates in flows orchestrated by HABADTE, which touches:

- `HAPTRFR` – transfer data (PHI fields: AFACCT, AFMRNO).
- `OMPMAST` – patient master (PHI fields: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN).


## Program XFXCYMD

### Overview

RPGLE program in domain "Data Maintenance". Contains 6 rule(s) with average confidence 58%. XFXCYMD validates calendar dates composed of year (VYY), month (VMM) and day (VDD).

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

From key_rules and approved_rules:

- BR-003: VYY < 1800 → `EXIT`.
- BR-004: VYY > 2100 → `EXIT`.
- BR-005: VMM < 01 → `EXIT`.
- BR-006: VMM > 12 → `EXIT`.
- BR-007: VDD < 01 → `EXIT`.
- BR-008: VDD > DYS(VMM) → `EXIT`.

Together, these rules assert that only plausible Gregorian dates are passed to downstream systems.

### Related Approved Rules by Domain

DATA_MAINTENANCE rules relevant to XFXCYMD include:

- BR-001–BR-008 for counter and date bounds.
- BR-009–BR-012 controlling level mapping.
- BR-013–BR-016 for table-driven lookups.

### Data Touched

XFXCYMD itself uses scalar fields for date components and does not directly access PF/LF objects. It is called by HABADTE to validate date parameters before performing transfer queries on:

- `HAPTRFR` (keys: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE; PHI: AFACCT, AFMRNO).


## Program XFXLDSC

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 56%. XFXLDSC manages level description mappings across several hierarchical files.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

- BR-009–BR-011: LDAMAP > 99 → `EXIT`.
- BR-012: LDAMAP > 9999 → `EXIT`.

These rules enforce upper limits on mapping identifiers, ensuring only supported level codes are used.

### Related Approved Rules by Domain

All DATA_MAINTENANCE rules interact conceptually, but XFXLDSC is primarily tied to BR-009–BR-012.

### Data Touched

From `data_lineage_compact.flow_paths` and `dep_edges`:

- Declares `HXPLVL1`–`HXPLVL6`.
- Reads `HXFLVL1`–`HXFLVL6`.

These level tables track hierarchical attributes (e.g., organizational or plan levels). None of their fields are PHI-bearing according to `phi_flagged_fields`.


## Program XFXTABL

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 90%. XFXTABL performs table-based lookups and uses indicator logic for early termination.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

- BR-013–BR-016: When *IN79 equals on/active, branch to `EXIT`.

These rules ensure that once a required mapping is found or an error condition arises, table processing stops.

### Related Approved Rules by Domain

- BR-001–BR-016 collectively shape data-maintenance utilities. XFXTABL provides dictionary-style lookups that complement XFXCYMD and XFXLDSC.

### Data Touched

From `dep_edges` and `data_lineage`:

- Declares `HXPTABLD`, `HXLTABLD`, `HXLTABLP`, `HXLTABLS`.
- Reads `XFFTABLD`, `XFFTABL2`, `XFFTABL3`, `XFFTABL4`.

These tables carry non-PHI configuration codes and descriptions.


## Program HABADTE

### Overview

RPGLE program in domain "Patient Management". Contains 3 rule(s) with average confidence 97%. HABADTE is the central orchestration module that reads patient transfer data, applies inclusion/exclusion rules and generates XML output for downstream systems.

### Domain and Program Type

- Domain: PATIENT_MANAGEMENT
- Type: RPGLE

### Business Rules Applied

Key rules from `interpretations_detail.key_rules` and `approved_rules`:

- BR-018 (confidence 0.99): When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.
- BR-019 (confidence 0.99): When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.
- BR-017 (confidence 0.92): When `-FILE INDICATOR` equals zero, branch to `SKIP`.

Effectively, HABADTE:

1. Skips records without a valid file indicator.
2. Skips voided transfers.
3. Skips outpatient records for this inpatient-oriented flow.

### Related Approved Rules by Domain

In the PATIENT_MANAGEMENT domain, HABADTE is associated with:

- BR-017 – file indicator requirement.
- BR-018 – voided transaction exclusion.
- BR-019 – outpatient exclusion.

It also leverages DATA_MAINTENANCE rules indirectly via calls to XFXCNTR, XFXCYMD, XFXLDSC and XFXTABL.

### Data Touched

Based on `dep_edges`, `data_dict_schema` and `phi_flagged_files`:

- Reads `HAPTRFR` (PF, subsystem HAP; PHI fields: AFACCT – AccountNumber, AFMRNO – MRN).
- Declares `HMLMAST5H`, `HAPIRNK`, `HXPBNFIT`, `HXPNSTN`, `****HXPXML`, `HXPXMLD`, `PRINTER`.
- Reads and writes XML-related files `HXFXMLH` and `HXFXMLD` (not fully defined in schema but present in dependencies).
- Interacts indirectly with PHI-bearing files:
  - `OMPMAST` (PHI: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN).
  - `OAPIRNK` (PHI: BRKMRN).
  - `OXPBNFIT` (PHI: XFBTEL).
  - `HXPDICT` (PHI: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME).

Together, these flows mean HABADTE must be treated as a high-sensitivity component in the modernization effort, with strict security and audit controls.


## Program HXXAPPPRF

### Overview

SQLRPGLE program in domain HXXA (interpreted as an application profile or configuration domain). Contains low cyclomatic complexity but participates in profile copy operations.

### Domain and Program Type

- Domain: DATA_MAINTENANCE (related via XFXMRNROL call graph)
- Type: SQLRPGLE

### Business Rules Applied

HXXAPPPRF does not have explicit approved_rules entries but is referenced by XFXMRNROL as a profile provider. Its logic likely involves:

- Loading profile preferences for MRN rollover or patient-context behavior.

### Related Approved Rules by Domain

Indirectly tied to PATIENT_MANAGEMENT rules via HABADTE → XFXMRNROL → HXXAPPPRF chain.

### Data Touched

From `dep_edges`:

- Copies `HXXCNTRL` and `HXXAPPPRFP`, which define control blocks and additional profile routines.

No PHI fields are directly associated with this program in `phi_flagged_fields`.


## Program XFXMRNROL

### Overview

RPGLE program in domain "Data Maintenance" that handles MRN (medical record number) rollover or mapping. Cyclomatic complexity is low but it calls into external profile programs.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

No distinct approved_rules are attributed directly to XFXMRNROL, but its behavior is inferred from dependencies:

- Calls `HXHAPPPRF` and `HXXAPPPRF` for profile-based decisions.

### Related Approved Rules by Domain

The PATIENT_MANAGEMENT rules (BR-017–BR-019) depend on proper MRN handling to ensure correct patient identification.

### Data Touched

From `dep_edges` and lineage:

- Uses MRN-related fields from PHI registry (e.g., CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN, MMMRNO).

These interactions mean XFXMRNROL participates in PHI flows.


## Narrative Summary

Across the HABADTE application:

- **Utility programs (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL)** enforce technical validity (counters, dates, mappings, table lookups).
- **Core orchestrator (HABADTE)** applies patient-management rules to filter records and generate XML, touching multiple PHI-bearing files.
- **Profile and MRN mapping programs (XFXMRNROL, HXXAPPPRF, HXHAPPPRF)** refine patient context and identification.

These narratives provide the basis for designing Spring Boot services that preserve business behavior, enforce data quality and maintain compliance with PHI-handling requirements.
