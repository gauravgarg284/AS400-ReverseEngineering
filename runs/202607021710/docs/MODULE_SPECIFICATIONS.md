# MODULE SPECIFICATIONS – HABADTE Suite

Run ID: 202607021710  
Project: HABADTE AS400 admission and data maintenance suite

---

## 1. Component Inventory

This inventory lists all analyzed source members from the HABADTE codebase, enriched with complexity and orphan status.

| Program | Type    | Subsystem | Lines | CC | Risk Band | Orphan |
|---------|---------|-----------|-------|----|-----------|--------|
| HAPIRNK    | DDS_LF   | HAP        | 13  | N/A | N/A   | yes |
| HAPTRFR    | DDS_PF   | HAP        | 72  | N/A | N/A   | yes |
| HMLMAST5H  | DDS_LF   | UNGROUPED  | 12  | N/A | N/A   | yes |
| HXLTABLD   | DDS_LF   | HXLT       | 11  | N/A | N/A   | yes |
| HXLTABLP   | DDS_LF   | HXLT       | 12  | N/A | N/A   | yes |
| HXLTABLS   | DDS_LF   | HXLT       | 12  | N/A | N/A   | yes |
| HXPBNFIT   | DDS_LF   | HXP        | 12  | N/A | N/A   | yes |
| HXPDICT    | DDS_PF   | HXP        | 6130| N/A | N/A   | yes |
| HXPLVL1    | DDS_PF   | HXPL       | 49  | N/A | N/A   | yes |
| HXPLVL2    | DDS_PF   | HXPL       | 52  | N/A | N/A   | yes |
| HXPLVL3    | DDS_PF   | HXPL       | 52  | N/A | N/A   | yes |
| HXPLVL4    | DDS_PF   | HXPL       | 52  | N/A | N/A   | yes |
| HXPLVL5    | DDS_PF   | HXPL       | 55  | N/A | N/A   | yes |
| HXPLVL6    | DDS_PF   | HXPL       | 321 | N/A | N/A   | yes |
| HXPNSTN    | DDS_LF   | HXP        | 12  | N/A | N/A   | yes |
| HXPTABLD   | DDS_PF   | HXP        | 19  | N/A | N/A   | yes |
| HXPXMLD    | DDS_PF   | HXPX       | 19  | N/A | N/A   | yes |
| HXPXMLR    | DDS_PF   | HXPX       | 19  | N/A | N/A   | yes |
| OAPIRNK    | DDS_PF   | UNGROUPED  | 80  | N/A | N/A   | yes |
| OMPMAST    | DDS_PF   | UNGROUPED  | 310 | N/A | N/A   | yes |
| OXPBNFIT   | DDS_PF   | OXP        | 48  | N/A | N/A   | yes |
| OXPNSTN    | DDS_PF   | OXP        | 65  | N/A | N/A   | yes |
| HXXAPPPRF  | SQLRPGLE | HXXA       | 123 | 1  | LOW   | yes |
| XFXCNTR    | RPGLE    | XFXC       | 49  | 3  | LOW   | yes |
| XFXCYMD    | RPGLE    | XFXC       | 83  | 7  | LOW   | yes |
| XFXGETID   | RPGLE    | XFX        | 61  | 1  | LOW   | yes |
| XFXLDSC    | RPGLE    | XFXL       | 135 | 5  | LOW   | yes |
| XFXLEAP    | RPGLE    | XFXL       | 61  | 1  | LOW   | yes |
| XFXMRNROL  | RPGLE    | XFX        | 65  | 1  | LOW   | yes |
| XFXTABL    | RPGLE    | XFX        | 164 | 9  | LOW   | yes |
| CXXXMLP    | SQLRPGLE | UNGROUPED  | 25  | 1  | LOW   | yes |
| HXXAPPPRFP | RPGLE    | HXXA       | 42  | 1  | LOW   | yes |
| HXXCNTRL   | RPGLE    | HXX        | 8   | 1  | LOW   | yes |
| HXXLDA     | RPGLE    | HXXL       | 53  | 1  | LOW   | yes |
| HXXLEVEL   | RPGLE    | HXXL       | 25  | 1  | LOW   | yes |
| HXXXML     | RPGLE    | HXX        | 11  | 1  | LOW   | yes |
| HABADTE    | RPGLE    | HA         | 821 | 152| HIGH  | yes |

Notes:
- CC and Risk Band are taken from the complexity_per_program map where available; DDS files do not have cyclomatic complexity.
- "Orphan" reflects the orphan_programs list and indicates no callers in the analyzed graph, even when business usage is known from user stories.

---

## 2. Dependency Hotspots

Programs with the highest dependency scores (fan-in, fan-out, file operations) are shown below.

| Program   | Score | Fan-In | Fan-Out | File Ops |
|-----------|-------|--------|---------|----------|
| HABADTE   | 38    | 0      | 13      | 6        |
| XFXLDSC   | 15    | 1      | 0       | 6        |
| XFXTABL   | 11    | 1      | 0       | 4        |
| XFXCNTR   | 9     | 3      | 0       | 0        |
| HXXAPPPRF | 7     | 1      | 2       | 0        |
| XFXGETID  | 7     | 1      | 1       | 1        |
| XFXMRNROL | 7     | 1      | 2       | 0        |
| XFXCYMD   | 5     | 1      | 1       | 0        |
| XFXLEAP   | 3     | 1      | 0       | 0        |
| HXHAPPPRF | 3     | 1      | 0       | 0        |
| HXLTABLD  | 2     | 0      | 0       | 1        |
| HAPIRNK   | 2     | 0      | 0       | 1        |
| HXLTABLP  | 2     | 0      | 0       | 1        |
| HMLMAST5H | 2     | 0      | 0       | 1        |
| HXPBNFIT  | 2     | 0      | 0       | 1        |
| HXPNSTN   | 2     | 0      | 0       | 1        |
| HXLTABLS  | 2     | 0      | 0       | 1        |

Interpretation:
- HABADTE is the central orchestrator with the highest fan-out and the largest number of file operations.
- XFXLDSC and XFXTABL are intensive data access utilities over level tables and dictionary tables.
- XFXCNTR, XFXMRNROL and HXXAPPPRF are shared routines that are called from multiple places and should be treated as services.

---

## 3. Call Graph Summary

This section summarizes direct dependency edges between components.

| Caller    | Callee     | Edge Type |
|-----------|------------|-----------|
| XFXCYMD   | XFXLEAP    | CALL      |
| XFXMRNROL | HXHAPPPRF  | CALL      |
| XFXMRNROL | HXXAPPPRF  | CALL      |
| HABADTE   | XFXMRNROL  | CALL      |
| HABADTE   | XFXCNTR    | CALL      |
| HABADTE   | XFXLDSC    | CALL      |
| HABADTE   | XFXCNTR    | CALL      |
| HABADTE   | XFXCNTR    | CALL      |
| HABADTE   | XFXCYMD    | CALL      |
| HABADTE   | XFXGETID   | CALL      |
| HABADTE   | XFXTABL    | CALL      |
| HXXAPPPRF | HXXCNTRL   | COPY      |
| HXXAPPPRF | HXXAPPPRFP | COPY      |
| XFXGETID  | HXXLDA     | COPY      |
| HABADTE   | HXXLDA     | COPY      |
| HABADTE   | HXXLEVEL   | COPY      |
| HABADTE   | HXXXML     | COPY      |
| HABADTE   | CXXXMLP    | COPY      |
| HABADTE   | CXXXMLC    | COPY      |
| HAPIRNK   | TAPIRNK    | PFILE_OF  |
| HMLMAST5H | TMPMAST    | PFILE_OF  |
| HXLTABLD  | HXPTABLD   | PFILE_OF  |
| HXLTABLP  | HXPTABLD   | PFILE_OF  |
| HXLTABLS  | HXPTABLD   | PFILE_OF  |
| HXPBNFIT  | TXPBNFIT   | PFILE_OF  |
| HXPNSTN   | TXPNSTN    | PFILE_OF  |
| XFXGETID  | HXFXMLR    | READ      |
| XFXLDSC   | HXFLVL1    | READ      |
| XFXLDSC   | HXFLVL2    | READ      |
| XFXLDSC   | HXFLVL3    | READ      |
| XFXLDSC   | HXFLVL4    | READ      |
| XFXLDSC   | HXFLVL5    | READ      |
| XFXLDSC   | HXFLVL6    | READ      |
| XFXTABL   | XFFTABLD   | READ      |
| XFXTABL   | XFFTABL2   | READ      |
| XFXTABL   | XFFTABL3   | READ      |
| XFXTABL   | XFFTABL4   | READ      |
| HABADTE   | HAPTRFR    | READ      |
| HABADTE   | XFFNSTN    | READ      |
| HABADTE   | HXFXMLH    | READ      |
| HABADTE   | HXFXMLH    | UPDATE    |
| HABADTE   | HXFXMLH    | WRITE     |
| HABADTE   | HXFXMLD    | WRITE     |

This call graph shows HABADTE at the center, calling specialized utilities and manipulating XML header/detail workfiles, transfer and status records as part of admission processing.

---

## 4. Business Rules by Program

This section groups approved business rules by their source program.

### Program: XFXCNTR

RPGLE counter-control utility in the Data Maintenance domain.

- BR-001 – When X equals zero, branch to 'EXIT'
- BR-002 – When X equals 40, branch to 'EXIT'

These rules implement threshold-based branching, allowing HABADTE or other callers to exit particular processing paths when counter X reaches specified values (0 or 40).

### Program: XFXCYMD

RPGLE date validation utility in the Data Maintenance domain.

- BR-003 – When VYY is less than 1800, branch to 'EXIT'
- BR-004 – When VYY is greater than 2100, branch to 'EXIT'
- BR-005 – When VMM is less than 01, branch to 'EXIT'
- BR-006 – When VMM is greater than 12, branch to 'EXIT'
- BR-007 – When VDD is less than 01, branch to 'EXIT'
- BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT'

These rules ensure that the admission date components (year, month, day) fall within valid ranges and respect month-specific day limits, with leap-year adjustments handled by XFXLEAP.

### Program: XFXLDSC

RPGLE level-description utility in the Data Maintenance domain.

- BR-009 – When LDAMAP is greater than 99, branch to 'EXIT'
- BR-010 – When LDAMAP is greater than 99, branch to 'EXIT'
- BR-011 – When LDAMAP is greater than 99, branch to 'EXIT'
- BR-012 – When LDAMAP is greater than 9999, branch to 'EXIT'

These rules cap mapping codes (LDAMAP) at specified thresholds, preventing invalid or out-of-range level mappings when traversing HXPLVL1–HXPLVL6 and associated runtime formats.

### Program: XFXTABL

RPGLE dictionary lookup utility in the Data Maintenance domain.

- BR-013 – When *IN79 equals on/active, branch to 'EXIT'
- BR-014 – When *IN79 equals on/active, branch to 'EXIT'
- BR-015 – When *IN79 equals on/active, branch to 'EXIT'
- BR-016 – When *IN79 equals on/active, branch to 'EXIT'

These rules use indicator *IN79 to guard dictionary operations. When the indicator reflects an "on/active" status, processing exits early, likely to avoid duplicate or unnecessary dictionary updates.

### Program: HABADTE

Main RPGLE program in the Patient Management domain.

- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'

These core admission-screening rules skip records that are inactive: missing file indicators, voided flags, or outpatient flags when inpatient workflows are being processed.

### Program: HXXAPPPRF

SQLRPGLE program in the Data Maintenance domain.

- BR-020 – SQL program accesses table 'HXPAPPPRF'

This rule reflects a key data access pattern: the program performs SQL operations against application profile table HXPAPPPRF as part of profile maintenance and MRN roll-up workflows.

---

## 5. Missing Dependencies (High/Medium Gaps)

Missing components referenced by the codebase.

| Name      | Type     | Impact  | Referenced By |
|-----------|----------|---------|---------------|
| CXXXMLC   | COPYBOOK | HIGH    | HABADTE       |
| HXHAPPPRF | PROGRAM  | MEDIUM  | XFXMRNROL     |
| TAPIRNK   | FILE     | HIGH    | HAPIRNK       |
| TMPMAST   | FILE     | HIGH    | HMLMAST5H     |
| TXPBNFIT  | FILE     | HIGH    | HXPBNFIT      |
| TXPNSTN   | FILE     | HIGH    | HXPNSTN       |
| ****HXPXML| FILE     | MEDIUM  | HABADTE       |
| PRINTER   | FILE     | MEDIUM  | HABADTE       |

Impact notes:
- High-impact gaps affect core schema and XML definitions (CXXXMLC, TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN).
- Medium-impact gaps affect auxiliary programs and IO targets (HXHAPPPRF, ****HXPXML, PRINTER).

---

## 6. Tech Debt Summary

Overall technical debt extracted from the analysis pipeline.

- Total findings: **4**
- Estimated remediation effort: **26.9 hours**
- Severity distribution:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

These findings cluster around missing artifacts and high-complexity routines (primarily HABADTE). Addressing them early in modernization will reduce risk and rework.
