# Process Narratives – HABADTE Project

This document summarizes the interpreted processing narratives for each analyzed program, tying business rules to data access patterns.

## XFXCNTR

**Overview:** RPGLE program in the Data Maintenance domain. It focuses on formatting character fields and deciding when formatting is necessary.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- BR-001 – Field formatting: exit if all blank (40 chars) (confidence 0.47)
- BR-002 – Field formatting: exit if first char non-blank (confidence 0.40)

These rules ensure that only non-empty, unformatted fields are processed, avoiding unnecessary transformations.

## XFXCYMD

**Overview:** RPGLE program in the Data Maintenance domain. It validates calendar dates, checking year, month, and day ranges and using leap-year logic.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT' (confidence 0.68)
- BR-003 – Date validation: reject year < 1800 (confidence 0.56)
- BR-004 – Date validation: reject year > 2100 (confidence 0.56)
- BR-005 – Date validation: reject month < 01 (confidence 0.56)
- BR-006 – Date validation: reject month > 12 (confidence 0.56)

Together, these rules enforce valid calendar dates and prevent impossible or out-of-range values from entering the system.

## XFXLDSC

**Overview:** RPGLE program in the Data Maintenance domain. It performs level lookups across multiple level tables (HXPLVL1–HXPLVL6 and corresponding logical views) and enforces valid level codes.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- BR-009 – Level lookup: reject if level code exceeds valid range (confidence 0.56)
- BR-010 – Level lookup: reject if level code exceeds valid range (confidence 0.56)
- BR-011 – Level lookup: reject if level code exceeds valid range (confidence 0.56)
- BR-012 – Level lookup: reject if level code exceeds valid range (confidence 0.56)

**Data Touched:**
- PFs: HXPLVL1–HXPLVL6
- LFs: HXFLVL1–HXFLVL6 (via READ operations)

These lookups drive table-based configuration for levels and prevent unauthorized level codes from being used.

## XFXTABL

**Overview:** RPGLE program in the Data Maintenance domain. It implements generic table lookups over HXPTABLD and related logical files and uses indicator *IN79 to stop processing when a control condition is met.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- BR-013 – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90)
- BR-014 – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90)
- BR-015 – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90)
- BR-016 – When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90)

**Data Touched:**
- PFs: HXPTABLD, XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4
- LFs: HXLTABLD, HXLTABLP, HXLTABLS

The program centralizes table lookup behavior and uses control indicators to terminate processing once the desired row has been found or a stop condition is met.

## HABADTE

**Overview:** RPGLE program in the Patient Management domain. It is the main controller for patient transfer processing and XML export, orchestrating calls to multiple XFX* utilities and accessing PHI-bearing files.

- **Domain:** PATIENT_MANAGEMENT
- **Program Type:** RPGLE

**Business Rules Applied:**
- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92)
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99)
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99)

These rules define which transactions are eligible for processing, ensuring that non-existent, voided, or outpatient transactions are excluded from the inpatient-focused transfer and export workflow.

**Data Touched:**
- PHI-bearing PFs: HAPTRFR (AFACCT, AFMRNO)
- Other PFs/LFs: HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN, HXPXMLD, ****HXPXML, PRINTER, XFFNSTN, HXFXMLH, HXFXMLD

## HXXAPPPRF

**Overview:** SQLRPGLE program in the Data Maintenance domain. It manages application profile records using SQL and is called from XFXMRNROL.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** SQLRPGLE

**Business Rules Applied:**
- BR-020 – SQL program accesses table 'HXPAPPPRF' (confidence 0.65)

This program ensures that MRN rollover and related processes respect application profiles stored in SQL tables.

## XFXGETID

**Overview:** RPGLE program in the Data Maintenance domain. It retrieves identifiers from XML header tables and may allocate new IDs when needed.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- No specific BR-IDs captured as key rules, but it participates in the XML header retrieval flow.

**Data Touched:**
- PFs: HXPXMLR, HXFXMLR (via DECLARE and READ operations)

## XFXLEAP

**Overview:** RPGLE program in the Data Maintenance domain. It supports leap-year calculations used by XFXCYMD.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- No explicit BR-IDs, but it underpins date validation by providing leap-year logic.

## XFXMRNROL

**Overview:** RPGLE program in the Data Maintenance domain. It coordinates MRN rollover logic and delegates to additional profile programs.

- **Domain:** DATA_MAINTENANCE
- **Program Type:** RPGLE

**Business Rules Applied:**
- No explicit BR-IDs listed, but the program calls HXHAPPPRF and HXXAPPPRF to enforce application profile rules during MRN changes.

Overall, these narratives describe how individual programs contribute to the wider patient management and data maintenance processes, linking rules, data structures, and PHI usage.
