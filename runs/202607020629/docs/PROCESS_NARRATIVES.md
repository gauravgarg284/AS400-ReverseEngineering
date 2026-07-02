# Process Narratives

## Program: XFXCNTR

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 2 rule(s) with avg confidence 43%.

XFXCNTR controls conditional branching based on a counter-like field `X`, determining when processing should terminate early. It is likely used as a utility routine to enforce bounds on loops or iterative operations.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

- BR-001: When X equals zero, branch to 'EXIT' (confidence 0.47).
- BR-002: When X equals 40, branch to 'EXIT' (confidence 0.40).

### Related Approved Rules (by Domain)

The following approved rules are in the DATA_MAINTENANCE domain and may be conceptually related when XFXCNTR is used alongside other validation utilities:

- BR-003: When VYY is less than 1800, branch to 'EXIT'.
- BR-004: When VYY is greater than 2100, branch to 'EXIT'.
- BR-005: When VMM is less than 01, branch to 'EXIT'.
- BR-006: When VMM is greater than 12, branch to 'EXIT'.
- BR-007: When VDD is less than 01, branch to 'EXIT'.
- BR-008: When VDD is greater than DYS(VMM), branch to 'EXIT'.
- BR-009: When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-010: When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-011: When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-012: When LDAMAP is greater than 9999, branch to 'EXIT'.
- BR-013–BR-016: Indicator-based exit rules in XFXTABL.

### Data Touched

XFXCNTR is a pure logic utility and does not directly access files according to the summarized metadata. It may be called by HABADTE (via dependency edges) but does not itself perform I/O.

## Program: XFXCYMD

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 6 rule(s) with avg confidence 58%.

XFXCYMD validates and interprets calendar date values represented as separate year (VYY), month (VMM), and day (VDD) components. It enforces bounds on each component and uses `DYS(VMM)` to derive the maximum day per month, preventing invalid date constructions.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

Key rules:

- BR-008: When VDD is greater than DYS(VMM), branch to 'EXIT' (confidence 0.68).
- BR-003: When VYY is less than 1800, branch to 'EXIT' (confidence 0.56).
- BR-004: When VYY is greater than 2100, branch to 'EXIT' (confidence 0.56).
- BR-005: When VMM is less than 01, branch to 'EXIT' (confidence 0.56).
- BR-006: When VMM is greater than 12, branch to 'EXIT' (confidence 0.56).

### Related Approved Rules (by Domain)

XFXCYMD shares its domain with XFXCNTR, XFXLDSC, and XFXTABL. All DATA_MAINTENANCE rules BR-001–BR-016 are relevant when considering a consolidated validation library.

### Data Touched

The program does not directly access files based on the summarized dependency edges; it is invoked by HABADTE to validate date components before further processing. It operates on in-memory fields and returns control to the caller.

## Program: XFXLDSC

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 56%.

XFXLDSC enforces valid ranges for level mapping values (LDAMAP), which are used to translate numeric level identifiers into descriptions by consulting HXPLVL1–HXPLVL6 and associated record formats. It ensures level mappings stay within expected bounds.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

Key rules:

- BR-009: When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-010: When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-011: When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-012: When LDAMAP is greater than 9999, branch to 'EXIT'.

### Related Approved Rules (by Domain)

Other DATA_MAINTENANCE rules (BR-001–BR-008, BR-013–BR-016) coexist with LDAMAP-related rules and form a broader validation toolkit.

### Data Touched

Dependency edges show XFXLDSC reading HXFLVL1–HXFLVL6 record formats as part of level description lookups. These are backed by physical files HXPLVL1–HXPLVL6 containing organizational hierarchy data.

## Program: XFXTABL

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 90%.

XFXTABL manages table-driven controls using indicator *IN79. When the indicator is set to "on/active", processing branches to EXIT in multiple points. This encapsulates generic table iteration or lookup logic that stops based on table state.

### Domain and Program Type

- Domain: DATA_MAINTENANCE
- Type: RPGLE

### Business Rules Applied

Key rules:

- BR-013: When *IN79 equals on/active, branch to 'EXIT'.
- BR-014: When *IN79 equals on/active, branch to 'EXIT'.
- BR-015: When *IN79 equals on/active, branch to 'EXIT'.
- BR-016: When *IN79 equals on/active, branch to 'EXIT'.

### Related Approved Rules (by Domain)

All DATA_MAINTENANCE rules apply as a shared rule library context.

### Data Touched

XFXTABL declares and reads table files XFFTABLD, XFFTABL2, XFFTABL3, and XFFTABL4. These are table descriptor files used to drive configurable behavior, but they are not PHI-bearing.

## Program: HABADTE

### Overview

RPGLE program in domain 'Patient Management'. Contains 3 rule(s) with avg confidence 97%.

HABADTE is the primary patient transfer activity extract program. It reads inpatient transfer records, applies strict inclusion/exclusion rules, and enriches data with benefit, station, hierarchy, and XML/printer preferences before output.

### Domain and Program Type

- Domain: PATIENT_MANAGEMENT
- Type: RPGLE

### Business Rules Applied

Key rules:

- BR-018: When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99).
- BR-019: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99).
- BR-017: When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92).

### Related Approved Rules (by Domain)

The PATIENT_MANAGEMENT domain contains only the HABADTE-specific rules BR-017–BR-019 in this extract, all focused on transfer record qualification.

### Data Touched

HABADTE interacts with multiple files and logical files:

- HAPTRFR (PF): primary transfer activity file with PHI fields AFACCT (AccountNumber) and AFMRNO (MRN).
- HXPBNFIT / OXPBNFIT (LF/PF): benefit plan files with PHI field XFBTEL (PhoneNumber).
- HAPIRNK / OAPIRNK (LF/PF over TAPIRNK): rank or risk-related files with PHI field BRKMRN (MRN).
- OMPMAST (PF over TMPMAST): patient master with PHI fields MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN.
- HXPNSTN / OXPNSTN (LF/PF over TXPNSTN): nursing station master, non-PHI but closely tied to patient location.
- HXPLVLn (PFs): organizational hierarchy levels, used via XFXLDSC.
- HXPXMLD/HXPXMLR and ****HXPXML: XML layout and header files for output formatting.
- PRINTER: printer configuration file for routing physical output.

Several of these are flagged as PHI-bearing files in the metadata (`phi_flagged_files`): OXPBNFIT, HXPDICT, OAPIRNK, OMPMAST, HAPTRFR.
