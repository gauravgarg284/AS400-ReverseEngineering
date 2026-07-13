# Process Narratives – HABADTE Project

## XFXCNTR

**Overview:**  
RPGLE program in the Data Maintenance domain that applies field-formatting rules to input strings. It focuses on early-exit conditions based on whether fields are blank or already populated.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

- BR-001: Field formatting: exit if all blank (40 chars) – prevents processing of completely blank fields.
- BR-002: Field formatting: exit if first char non-blank – avoids overwriting already populated fields.

### Related Approved Rules

All rules for this program are already captured above.

### Data Touched

XFXCNTR operates on in-memory fields and does not directly access DDS files. No PHI-bearing files are involved.

---

## XFXCYMD

**Overview:**  
RPGLE program in the Data Maintenance domain that validates dates, including ranges for year, month, and day, and maximum day per month using a leap-year-aware check.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

- BR-008: When VDD is greater than DYS(VMM), branch to 'EXIT'.
- BR-003: Date validation: reject year < 1800 (historical minimum).
- BR-004: Date validation: reject year > 2100 (forecast maximum).
- BR-005: Date validation: reject month < 01 (calendar constraint).
- BR-006: Date validation: reject month > 12 (calendar constraint).

### Related Approved Rules

All rules above are sourced directly from XFXCYMD.

### Data Touched

XFXCYMD validates date fields passed in from callers but does not directly read or write database files. It relies on XFXLEAP for leap-year calculations.

---

## XFXLDSC

**Overview:**  
RPGLE program in the Data Maintenance domain that resolves level descriptions for a multi-level organisational hierarchy. It reads level files HXPLVL1–HXPLVL6 and associated HXFLVL* records to obtain names and attributes.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

- BR-009: Level lookup: reject if level code exceeds valid range.
- BR-010: Level lookup: reject if level code exceeds valid range.
- BR-011: Level lookup: reject if level code exceeds valid range.
- BR-012: Level lookup: reject if level code exceeds valid range.

### Related Approved Rules

These four rules fully describe the constraints for level validation.

### Data Touched

- HXPLVL1–HXPLVL6 physical files (hierarchy levels).  
- HXFLVL1–HXFLVL6 record formats (read operations).  
No PHI-bearing fields are flagged in these files.

---

## XFXTABL

**Overview:**  
RPGLE program in the Data Maintenance domain that implements generic table lookup logic for code translation, using HXPTABLD and related tables.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

- BR-013: When *IN79 equals on/active, branch to 'EXIT'.
- BR-014: When *IN79 equals on/active, branch to 'EXIT'.
- BR-015: When *IN79 equals on/active, branch to 'EXIT'.
- BR-016: When *IN79 equals on/active, branch to 'EXIT'.

### Related Approved Rules

All above rules originate from XFXTABL and govern early-exit behaviour when control indicators are set.

### Data Touched

- HXPTABLD (PF) via XFFTABLD format.  
- HXLTABLD, HXLTABLP, HXLTABLS (LFs) providing different key sequences.  
No PHI-bearing fields are identified in these table definitions.

---

## HABADTE

**Overview:**  
RPGLE program in the Patient Management domain that orchestrates patient transfer processing and XML output generation. It applies filters to exclude invalid, voided, or outpatient transfers and coordinates calls to utilities for hierarchy and station enrichment.

- **Domain:** PATIENT_MANAGEMENT  
- **Program Type:** RPGLE

### Business Rules Applied

- BR-018: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.
- BR-017: When -FILE INDICATOR equals zero, branch to 'SKIP'.

### Related Approved Rules

These rules define the inclusion/exclusion criteria for HABADTE's transfer selection.

### Data Touched

- HAPTRFR (PF) – transfer records with PHI fields AFACCT (AccountNumber) and AFMRNO (MRN).  
- OXPNSTN / HXPNSTN (PF/LF) – station master and logical view.  
- HXPBNFIT / TXPBNFIT – benefit-related tables (non-PHI for this process).  
- HAPIRNK / TAPIRNK – rank tables (may carry MRN via OAPIRNK).  
- HMLMAST5H / TMPMAST – master data view and base.  
- HXPXMLD, HXPXMLR, HXFXMLH, HXFXMLD – XML header and detail files.  
- PRINTER (external) – printer or spool-file definition (missing in source).  

PHI-sensitive data is read from HAPTRFR and potentially from linked master tables, requiring strong access controls.

---

## HXXAPPPRF

**Overview:**  
SQLRPGLE program in the Data Maintenance domain that manages application profile data using embedded SQL. It accesses HXPAPPPRF and related tables to update or read profile records.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** SQLRPGLE

### Business Rules Applied

- BR-020: SQL program accesses table 'HXPAPPPRF'.

### Related Approved Rules

Additional rules BR-021 and BR-022 (from interpretations) indicate related accesses but are not listed as approved_rules.

### Data Touched

- HXPAPPPRF (table referenced in BR-020).  
PHI flags are not explicitly associated with these tables in the compact schema, but profiles may include user or application identifiers.

---

## XFXGETID

**Overview:**  
RPGLE program in the Data Maintenance domain that retrieves identifiers for XML records, acting as a utility to read ID information from XML header files.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

No explicit approved_rules beyond structural behaviour; the program focuses on retrieving IDs.

### Related Approved Rules

None.

### Data Touched

- HXPXMLR (PF) – declared in the program.  
- HXFXMLR – read to obtain XML header identifiers.  
No PHI-bearing fields are flagged for these files.

---

## XFXLEAP

**Overview:**  
RPGLE program in the Data Maintenance domain that performs leap-year calculations, supporting date validation logic in XFXCYMD.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

No direct approved_rules; the program provides supporting calculations used by XFXCYMD.

### Data Touched

No direct DDS file access; computations are in-memory.

---

## XFXMRNROL

**Overview:**  
RPGLE program in the Data Maintenance domain that coordinates MRN roll logic, calling both legacy and SQL-based profile programs.

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### Business Rules Applied

No explicit approved_rules; behaviour is inferred from call graph and interpretations.

### Data Touched

- HXHAPPPRF (PROGRAM, missing) – called for legacy profile updates.  
- HXXAPPPRF – called for SQL-based profile updates.  
Actual PF/LF usage is encapsulated in those called programs.
