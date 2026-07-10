# PROCESS_NARRATIVES – HABADTE Application

This document provides narrative descriptions of each interpreted program, based on `interpretations_detail` and cross-referenced approved business rules.

---

## XFXCNTR

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain. It implements low-level counter and field-formatting logic, ensuring that control fields used for sequencing or identifiers follow strict formatting rules. The program is invoked by other components when a field is expected to be blank prior to assignment.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

From `key_rules`:

- **BR-001:** Field formatting: exit if all blank (40 chars) – confidence 0.47.
- **BR-002:** Field formatting: exit if first char non-blank – confidence 0.40.

### Related Approved Rules (by Domain)

All rules for XFXCNTR come from the DATA_MAINTENANCE domain and are already listed above. No HABADTE-specific PHI rules apply directly in this program.

### Data Touched

XFXCNTR is a pure logic utility in the available metadata; it does not read or write PHI-bearing files directly. Instead, it operates on fields passed in by callers (such as HABADTE) and decides whether to proceed or exit based on formatting.

---

## XFXCYMD

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain. XFXCYMD centralizes calendar and date validation rules, checking ranges for years, months, and days, and enforcing overall calendar correctness (including month-day combinations via leap-year logic). It is called from HABADTE and potentially other flows whenever a legacy date in numeric format needs validation.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

From `key_rules`:

- **BR-008:** When VDD is greater than DYS(VMM), branch to 'EXIT' – confidence 0.68.
- **BR-003:** Date validation: reject year < 1800 (historical minimum) – confidence 0.56.
- **BR-004:** Date validation: reject year > 2100 (forecast maximum) – confidence 0.56.
- **BR-005:** Date validation: reject month < 01 (calendar constraint) – confidence 0.56.
- **BR-006:** Date validation: reject month > 12 (calendar constraint) – confidence 0.56.

### Related Approved Rules (by Domain)

All above rules are approved in the DATA_MAINTENANCE domain. They are used by HABADTE indirectly whenever transfer dates are validated before inclusion in XML output.

### Data Touched

The program operates purely on incoming date fields and does not directly access persistent files. It depends on the utility program **XFXLEAP** (via CALL) to determine leap-year behavior.

---

## XFXLDSC

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain. XFXLDSC performs hierarchical level lookups against the HXPLVL1–HXPLVL6 tables, providing descriptions and metadata for level codes used across the application. It is a key enrichment step in HABADTE when building the final XML representation of patient transfers.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

From `key_rules`:

- **BR-009:** Level lookup: reject if level code exceeds valid range – confidence 0.56.
- **BR-010:** Level lookup: reject if level code exceeds valid range – confidence 0.56.
- **BR-011:** Level lookup: reject if level code exceeds valid range – confidence 0.56.
- **BR-012:** Level lookup: reject if level code exceeds valid range – confidence 0.56.

These rules correspond to multiple guard points in the lookup process, ensuring that different aspects or segments of the level code are validated before use.

### Related Approved Rules (by Domain)

Because the entire rule set comes from the DATA_MAINTENANCE domain, XFXLDSC is a generic utility that can be reused across multiple business processes, not just HABADTE.

### Data Touched

Relevant files from the data dictionary:

- **HXPLVL1–HXPLVL6** (PFs): key fields HX1NUM–HX6NUM, containing hierarchical level/plan data.  
- Logical/read formats: HXFLVL1–HXFLVL6 are read by XFXLDSC according to the dependency map.

None of these files contain PHI fields; they are configuration/reference data.

---

## XFXTABL

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain. XFXTABL implements a generic table lookup mechanism over HXPTABLD and its associated logical views HXLTABLD, HXLTABLP, and HXLTABLS. It retrieves descriptive values or control settings based on table and entry codes. HABADTE uses XFXTABL to resolve configuration-driven behaviors.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

From `key_rules`:

- **BR-013:** When *IN79 equals on/active, branch to 'EXIT' – confidence 0.90.
- **BR-014:** When *IN79 equals on/active, branch to 'EXIT' – confidence 0.90.
- **BR-015:** When *IN79 equals on/active, branch to 'EXIT' – confidence 0.90.
- **BR-016:** When *IN79 equals on/active, branch to 'EXIT' – confidence 0.90.

Collectively, these rules implement a control indicator that can short-circuit one or more table lookups when a configured flag is active.

### Related Approved Rules (by Domain)

All rules belong to DATA_MAINTENANCE and are shared across any consumer that uses XFXTABL. They do not directly interact with PHI but may indirectly affect which PHI-bearing records are processed by enabling or disabling certain workflows.

### Data Touched

From `data_dict_schema` and lineage:

- **HXPTABLD** (PF): XFFTABLD, generic dictionary table.  
- **HXLTABLD**, **HXLTABLP**, **HXLTABLS** (LFs): logical views over HXPTABLD used to access the table by different key combinations.

No PHI fields are present in these tables.

---

## HABADTE

**Overview**  
RPGLE program in the PATIENT_MANAGEMENT domain. HABADTE is the main batch driver responsible for processing patient transfer records. It reads the HAPTRFR file, applies inclusion and exclusion rules, calls multiple utility programs for validation and enrichment, and writes XML header and detail records for external consumers.

- **Domain:** PATIENT_MANAGEMENT  
- **Program Type:** RPGLE

### Business Rules Applied

From `key_rules`:

- **BR-018:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP' – confidence 0.99.
- **BR-019:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' – confidence 0.99.
- **BR-017:** When -FILE INDICATOR equals zero, branch to 'SKIP' – confidence 0.92.

### Related Approved Rules (by Domain)

All three rules above are also present in `approved_rules` under the PATIENT_MANAGEMENT domain. HABADTE indirectly depends on additional DATA_MAINTENANCE rules via its calls to XFXCYMD, XFXLDSC, XFXTABL, and XFXCNTR.

### Data Touched

Using `data_dict_schema` and `phi_flagged_files`, HABADTE interacts (directly or via declared files) with:

- **HAPTRFR** (PF, PHI): patient transfer records, with `AFACCT` (AccountNumber) and `AFMRNO` (MRN) marked as PHI.
- **HMLMAST5H** / **TMPMAST**: patient master staging data (no PHI fields flagged in TMPMAST, but related live PHI exists in OMPMAST).
- **HXPBNFIT** / **TXPBNFIT**: benefit data, with live PHI in OXPBNFIT (XFBTEL phone number).
- **HAPIRNK** / **TAPIRNK**: ranking data with live PHI in OAPIRNK (BRKMRN MRN).
- **HXPNSTN** / **TXPNSTN**: status data.
- **HXPXMLD**, **HXPXMLR**, and external HXFXMLH/HXFXMLD: XML header and detail structures (not individually PHI-tagged but expected to carry PHI from source tables).

PHI handling is therefore concentrated in HABADTE’s processing of HAPTRFR and its downstream XML outputs.

---

## HXXAPPPRF

**Overview**  
SQLRPGLE program in the DATA_MAINTENANCE domain. HXXAPPPRF manages application profile data via SQL, reading and updating HXPAPPPRF and related profile tables. It is called from XFXMRNROL, allowing MRN rollover logic to consult profile configuration.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** SQLRPGLE

### Business Rules Applied

From `key_rules`:

- **BR-020:** SQL program accesses table 'HXPAPPPRF' – confidence 0.65.
- **BR-021:** SQL program accesses table 'HXPAPPL6' – confidence 0.65.
- **BR-022:** SQL program accesses table 'HXPAPPPRF' – confidence 0.65.

These rules indicate that HXXAPPPRF is strongly tied to application profile tables, which must be migrated alongside the program’s logic.

### Related Approved Rules (by Domain)

Only BR-020 appears in the approved_rules list; BR-021 and BR-022 are inferred from interpretations_detail and should be validated during design. All belong to DATA_MAINTENANCE.

### Data Touched

The specific tables HXPAPPPRF and HXPAPPL6 are not detailed in `data_dict_schema`, but HXXAPPPRF itself does not directly expose PHI according to available PHI metadata. It may, however, control access to PHI-bearing applications.

---

## XFXGETID

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain with cyclomatic complexity 1. XFXGETID is a simple utility that allocates or retrieves identifiers, particularly for XML header records. It reads from HXFXMLR and uses HXPXMLR to manage sequences.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

No explicit key_rules are listed for XFXGETID, but behavior is inferred from dependencies: it declares HXPXMLR and reads HXFXMLR to manage XML record IDs.

### Related Approved Rules (by Domain)

No dedicated approved_rules entries are tied to XFXGETID; it is a technical utility whose correctness is inferred from integration tests involving HABADTE.

### Data Touched

- **HXPXMLR** (PF): HXFXMLR, XML header records, non-PHI in schema but likely carrying batch context for PHI-bearing payloads.
- **HXFXMLR** (external PF): read for ID allocation according to lineage.

---

## XFXLEAP

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain. XFXLEAP is a tiny utility used by XFXCYMD to determine whether a given year is a leap year. Its cyclomatic complexity is 1, indicating a single decision path.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

No explicit key_rules are cataloged, but the program’s responsibility is to implement standard leap-year logic (e.g., divisible by 4, not by 100 unless also by 400).

### Related Approved Rules (by Domain)

The leap-year behavior supports the calendar constraints in BR-003–BR-008 (XFXCYMD) and should be tested together with that program.

### Data Touched

XFXLEAP operates purely in memory on integer year values and does not touch persistent storage.

---

## XFXMRNROL

**Overview**  
RPGLE program in the DATA_MAINTENANCE domain. XFXMRNROL orchestrates MRN (Medical Record Number) rollover or validation logic and delegates to application profile programs HXHAPPPRF (missing) and HXXAPPPRF. It acts as a routing layer between transfer logic and profile maintenance.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

No explicit key_rules are listed. The program’s behavior is inferred from its dependencies:

- Calls **HXHAPPPRF** – likely a legacy MRN profile handler.
- Calls **HXXAPPPRF** – SQL-based MRN profile handler.

### Related Approved Rules (by Domain)

XFXMRNROL is influenced by the profile access rules in HXXAPPPRF (BR-020–BR-022) but has no direct business rules in the approved list. It should be treated as an integration component.

### Data Touched

Data access is mediated through HXHAPPPRF and HXXAPPPRF; no PHI-specific fields are called out at this level, but MRN handling inherently involves PHI and must be secured.

---

## Summary of PHI-Relevant Processes

Programs that directly process or influence PHI-bearing data include:

- **HABADTE** – reads HAPTRFR and emits XML records carrying account numbers and MRNs.
- **XFXMRNROL / HXXAPPPRF** – manage MRN and profile logic (MRNs are PHI by definition).
- **Supporting files** (via HABADTE): OMPMAST, OAPIRNK, OXPBNFIT, HXPDICT.

All such programs must be included in PHI handling reviews and security design for the modernized platform.
