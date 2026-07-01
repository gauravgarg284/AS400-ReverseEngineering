# Process Narratives

This document summarizes the interpreted business processes for each key program, including applicable rules and data touchpoints.

## Program: XFXCNTR

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 2 rule(s) with avg confidence 43%.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

- **BR-001**: When X equals zero, branch to 'EXIT' (confidence 0.47).
- **BR-002**: When X equals 40, branch to 'EXIT' (confidence 0.40).

### Related Approved Rules (by Domain)

All approved rules in DATA_MAINTENANCE domain:

- BR-001 to BR-016 (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL)

### Data Touched

Based on dependency edges, XFXCNTR is a called utility without direct file I/O listed in the dependencies. It participates in patient-centric flows via HABADTE but does not itself read PHI-bearing files.

## Program: XFXCYMD

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 6 rule(s) with avg confidence 58%.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

- **BR-008**: When VDD is greater than DYS(VMM), branch to 'EXIT' (confidence 0.68).
- **BR-003**: When VYY is less than 1800, branch to 'EXIT' (confidence 0.56).
- **BR-004**: When VYY is greater than 2100, branch to 'EXIT' (confidence 0.56).
- **BR-005**: When VMM is less than 01, branch to 'EXIT' (confidence 0.56).
- **BR-006**: When VMM is greater than 12, branch to 'EXIT' (confidence 0.56).

### Related Approved Rules (by Domain)

- BR-001 to BR-016 (DATA_MAINTENANCE).

### Data Touched

XFXCYMD is invoked from HABADTE for date validation and conversion. It indirectly impacts how dates from PHI-bearing files such as HAPTRFR and OMPMAST are validated, but it does not directly read those files.

## Program: XFXLDSC

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 56%.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

- **BR-009**: When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-010**: When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-011**: When LDAMAP is greater than 99, branch to 'EXIT' (confidence 0.56).
- **BR-012**: When LDAMAP is greater than 9999, branch to 'EXIT' (confidence 0.56).

### Related Approved Rules (by Domain)

- BR-001 to BR-016 (DATA_MAINTENANCE).

### Data Touched

From the dependency graph, XFXLDSC reads level files HXFLVL1-6. These files are not directly flagged as PHI-bearing but may be used to resolve levels and descriptors referenced by HABADTE.

## Program: XFXTABL

### Overview

RPGLE program in domain 'Data Maintenance'. Contains 4 rule(s) with avg confidence 90%.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Program Type**: RPGLE

### Business Rules Applied

- **BR-013**: When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).
- **BR-014**: When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).
- **BR-015**: When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).
- **BR-016**: When *IN79 equals on/active, branch to 'EXIT' (confidence 0.90).

### Related Approved Rules (by Domain)

- BR-001 to BR-016 (DATA_MAINTENANCE).

### Data Touched

XFXTABL interacts with table/dictionary structures such as XFFTABLD and related logical files (HXLTABLD, HXLTABLP, HXLTABLS). These are code tables without direct PHI, but they may reference codes used in PHI-bearing contexts.

## Program: HABADTE

### Overview

RPGLE program in domain 'Patient Management'. Contains 3 rule(s) with avg confidence 97%.

### Domain and Program Type

- **Domain**: PATIENT_MANAGEMENT
- **Program Type**: RPGLE

### Business Rules Applied

- **BR-018**: When -FLAG INDICATOR equals void/voided, branch to 'SKIP' (confidence 0.99).
- **BR-019**: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' (confidence 0.99).
- **BR-017**: When -FILE INDICATOR equals zero, branch to 'SKIP' (confidence 0.92).

### Related Approved Rules (by Domain)

- BR-017, BR-018, BR-019 (PATIENT_MANAGEMENT).

### Data Touched

HABADTE is a high-complexity program (CC=152, HIGH band) and a major dependency hotspot. It calls and reads:

- **HAPTRFR** (PHI-bearing)
  - Fields with PHI: AFACCT (AccountNumber), AFMRNO (MRN)
- **OMPMAST** (via HMLMAST5H / TMPMAST; PHI-bearing)
  - Fields with PHI: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OAPIRNK** / HAPIRNK (PHI-bearing)
  - Fields with PHI: BRKMRN (MRN)
- **OXPBNFIT** / HXPBNFIT (PHI-bearing)
  - Fields with PHI: XFBTEL (PhoneNumber)
- **HXPDICT** (PHI-bearing)
  - Fields with PHI: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HABADTE orchestrates these data sources, applying skip rules to control which patient records are included in downstream XML or report outputs.
