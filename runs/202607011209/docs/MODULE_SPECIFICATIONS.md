# Module Specifications – HABADTE AS400 Application

## Section 1 – Component Inventory

| Program       | Type     | Subsystem | Lines | CC  | Risk Band | Orphan |
|--------------|---------|-----------|-------|-----|-----------|--------|
| HAPIRNK      | DDS_LF  | HAP       | 13    | N/A | N/A       | No     |
| HAPTRFR      | DDS_PF  | HAP       | 72    | N/A | N/A       | No     |
| HMLMAST5H    | DDS_LF  | UNGROUPED | 12    | N/A | N/A       | No     |
| HXLTABLD     | DDS_LF  | HXLT      | 11    | N/A | N/A       | No     |
| HXLTABLP     | DDS_LF  | HXLT      | 12    | N/A | N/A       | No     |
| HXLTABLS     | DDS_LF  | HXLT      | 12    | N/A | N/A       | No     |
| HXPBNFIT     | DDS_LF  | HXP       | 12    | N/A | N/A       | No     |
| HXPDICT      | DDS_PF  | HXP       | 6130  | N/A | N/A       | No     |
| HXPLVL1      | DDS_PF  | HXPL      | 49    | N/A | N/A       | No     |
| HXPLVL2      | DDS_PF  | HXPL      | 52    | N/A | N/A       | No     |
| HXPLVL3      | DDS_PF  | HXPL      | 52    | N/A | N/A       | No     |
| HXPLVL4      | DDS_PF  | HXPL      | 52    | N/A | N/A       | No     |
| HXPLVL5      | DDS_PF  | HXPL      | 55    | N/A | N/A       | No     |
| HXPLVL6      | DDS_PF  | HXPL      | 321   | N/A | N/A       | No     |
| HXPNSTN      | DDS_LF  | HXP       | 12    | N/A | N/A       | No     |
| HXPTABLD     | DDS_PF  | HXP       | 19    | N/A | N/A       | No     |
| HXPXMLD      | DDS_PF  | HXPX      | 19    | N/A | N/A       | No     |
| HXPXMLR      | DDS_PF  | HXPX      | 19    | N/A | N/A       | No     |
| OAPIRNK      | DDS_PF  | UNGROUPED | 80    | N/A | N/A       | No     |
| OMPMAST      | DDS_PF  | UNGROUPED | 310   | N/A | N/A       | No     |
| OXPBNFIT     | DDS_PF  | OXP       | 48    | N/A | N/A       | No     |
| OXPNSTN      | DDS_PF  | OXP       | 65    | N/A | N/A       | No     |
| HXXAPPPRF    | SQLRPGLE| HXXA      | 123   | 1   | LOW       | Yes    |
| XFXCNTR      | RPGLE   | XFXC      | 49    | 3   | LOW       | Yes    |
| XFXCYMD      | RPGLE   | XFXC      | 83    | 7   | LOW       | Yes    |
| XFXGETID     | RPGLE   | XFX       | 61    | 1   | LOW       | Yes    |
| XFXLDSC      | RPGLE   | XFXL      | 135   | 5   | LOW       | No     |
| XFXLEAP      | RPGLE   | XFXL      | 61    | 1   | LOW       | Yes    |
| XFXMRNROL    | RPGLE   | XFX       | 65    | 1   | LOW       | Yes    |
| XFXTABL      | RPGLE   | XFX       | 164   | 9   | LOW       | Yes    |
| CXXXMLP      | SQLRPGLE| UNGROUPED | 25    | 1   | LOW       | Yes    |
| HXXAPPPRFP   | RPGLE   | HXXA      | 42    | 1   | LOW       | Yes    |
| HXXCNTRL     | RPGLE   | HXX       | 8     | 1   | LOW       | Yes    |
| HXXLDA       | RPGLE   | HXXL      | 53    | 1   | LOW       | Yes    |
| HXXLEVEL     | RPGLE   | HXXL      | 25    | 1   | LOW       | Yes    |
| HXXXML       | RPGLE   | HXX       | 11    | 1   | LOW       | Yes    |
| HABADTE      | RPGLE   | HA        | 821   | 152 | HIGH      | Yes    |

_Total members: 37; total lines: 9,153; completeness: 82.2%; gaps: 8._

## Section 2 – Dependency Hotspots

Programs with highest combined call fan-out and file activity.

| Program   | Score | Fan-In | Fan-Out | File Ops |
|-----------|-------|--------|---------|----------|
| HABADTE   | 38    | 0      | 13      | 6        |
| XFXLDSC   | 15    | 1      | 0       | 6        |
| XFXTABL   | 11    | 1      | 0       | 4        |
| XFXCNTR   | 9     | 3      | 0       | 0        |
| XFXMRNROL | 7     | 1      | 2       | 0        |
| XFXGETID  | 7     | 1      | 1       | 1        |
| HXXAPPPRF | 7     | 1      | 2       | 0        |
| XFXCYMD   | 5     | 1      | 1       | 0        |
| HXHAPPPRF | 3     | 1      | 0       | 0        |
| XFXLEAP   | 3     | 1      | 0       | 0        |
| HXLTABLS  | 2     | 0      | 0       | 1        |
| HMLMAST5H | 2     | 0      | 0       | 1        |
| HXPNSTN   | 2     | 0      | 0       | 1        |
| HAPIRNK   | 2     | 0      | 0       | 1        |
| HXLTABLD  | 2     | 0      | 0       | 1        |
| HXPBNFIT  | 2     | 0      | 0       | 1        |
| HXLTABLP  | 2     | 0      | 0       | 1        |

HABADTE is the central orchestrator for patient management and XML output, with many downstream calls and file operations. XFXLDSC and XFXTABL are key lookup and level-discovery utilities.

## Section 3 – Call Graph Summary

Call, copy, file and logical relationships inferred from the dependency edges.

### Program-to-Program Calls

| Caller     | Callee      | Type  |
|------------|-------------|-------|
| XFXCYMD    | XFXLEAP     | CALL  |
| XFXMRNROL  | HXHAPPPRF   | CALL  |
| XFXMRNROL  | HXXAPPPRF   | CALL  |
| HABADTE    | XFXMRNROL   | CALL  |
| HABADTE    | XFXCNTR     | CALL  |
| HABADTE    | XFXLDSC     | CALL  |
| HABADTE    | XFXCNTR     | CALL  |
| HABADTE    | XFXCNTR     | CALL  |
| HABADTE    | XFXCYMD     | CALL  |
| HABADTE    | XFXGETID    | CALL  |
| HABADTE    | XFXTABL     | CALL  |

HABADTE calls multiple XFX* utility programs for counter logic, date validation, ID retrieval and table lookup. XFXMRNROL further delegates to profile programs HXHAPPPRF and HXXAPPPRF.

### Copy Relationships

| Source Program | Copied Member | Type  |
|----------------|---------------|-------|
| HXXAPPPRF      | HXXCNTRL      | COPY  |
| HXXAPPPRF      | HXXAPPPRFP    | COPY  |
| XFXGETID       | HXXLDA        | COPY  |
| HABADTE        | HXXLDA        | COPY  |
| HABADTE        | HXXLEVEL      | COPY  |
| HABADTE        | HXXXML        | COPY  |
| HABADTE        | CXXXMLP       | COPY  |
| HABADTE        | CXXXMLC       | COPY  |

These copies indicate HABADTE and XFXGETID share data area and XML control structures defined in HXX* and CXX* members.

### Logical-to-Physical File Relationships

| Logical File | Physical File | Type      |
|--------------|---------------|-----------|
| HAPIRNK      | TAPIRNK       | PFILE_OF  |
| HMLMAST5H    | TMPMAST       | PFILE_OF  |
| HXLTABLD     | HXPTABLD      | PFILE_OF  |
| HXLTABLP     | HXPTABLD      | PFILE_OF  |
| HXLTABLS     | HXPTABLD      | PFILE_OF  |
| HXPBNFIT     | TXPBNFIT      | PFILE_OF  |
| HXPNSTN      | TXPNSTN       | PFILE_OF  |

### File Access Operations

| Program   | File/Object | Operation |
|-----------|------------|-----------|
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

HABADTE is responsible for reading transactional transfer data, performing status lookups, and producing XML structures.

## Section 4 – Business Rules by Program

Business rules extracted from RPG logic and grouped by source program.

### Program XFXCNTR (Domain: DATA_MAINTENANCE)

- BR-001: When X equals zero, branch to `EXIT`.
- BR-002: When X equals 40, branch to `EXIT`.

These rules govern boundary conditions for a counter or numeric control field. If the counter is at zero or at a specific terminal value (40), processing exits early.

### Program XFXCYMD (Domain: DATA_MAINTENANCE)

- BR-003: When VYY is less than 1800, branch to `EXIT`.
- BR-004: When VYY is greater than 2100, branch to `EXIT`.
- BR-005: When VMM is less than 01, branch to `EXIT`.
- BR-006: When VMM is greater than 12, branch to `EXIT`.
- BR-007: When VDD is less than 01, branch to `EXIT`.
- BR-008: When VDD is greater than `DYS(VMM)`, branch to `EXIT`.

These rules collectively validate Gregorian calendar dates, ensuring year, month and day are all within valid ranges before further processing.

### Program XFXLDSC (Domain: DATA_MAINTENANCE)

- BR-009: When LDAMAP is greater than 99, branch to `EXIT`.
- BR-010: When LDAMAP is greater than 99, branch to `EXIT`.
- BR-011: When LDAMAP is greater than 99, branch to `EXIT`.
- BR-012: When LDAMAP is greater than 9999, branch to `EXIT`.

LDAMAP appears to be a level or mapping code. These rules enforce upper bounds on mapping identifiers to avoid invalid lookups.

### Program XFXTABL (Domain: DATA_MAINTENANCE)

- BR-013: When *IN79 equals on/active, branch to `EXIT`.
- BR-014: When *IN79 equals on/active, branch to `EXIT`.
- BR-015: When *IN79 equals on/active, branch to `EXIT`.
- BR-016: When *IN79 equals on/active, branch to `EXIT`.

These rules use the indicator *IN79 to control early exit from table processing; when a particular condition flag is active, subsequent table rows are not processed.

### Program HABADTE (Domain: PATIENT_MANAGEMENT)

- BR-017: When `-FILE INDICATOR` equals zero, branch to `SKIP`.
- BR-018: When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.
- BR-019: When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.

HABADTE applies these rules to skip records that are missing required file indicators, voided, or flagged as outpatient when only inpatient records are in scope.

## Section 5 – Missing Dependencies

High and medium-impact components referenced by the code but not present in the harvested source.

| Name       | Type      | Impact | Referenced By |
|------------|-----------|--------|---------------|
| CXXXMLC    | COPYBOOK  | HIGH   | HABADTE       |
| HXHAPPPRF  | PROGRAM   | MEDIUM | XFXMRNROL     |
| TAPIRNK    | FILE      | HIGH   | HAPIRNK       |
| TMPMAST    | FILE      | HIGH   | HMLMAST5H     |
| TXPBNFIT   | FILE      | HIGH   | HXPBNFIT      |
| TXPNSTN    | FILE      | HIGH   | HXPNSTN       |
| ****HXPXML | FILE      | MEDIUM | HABADTE       |
| PRINTER    | FILE      | MEDIUM | HABADTE       |

These gaps affect completeness of printing, XML formatting, and some logical file definitions but do not prevent understanding of the core patient filtering logic.

## Section 6 – Tech Debt Summary

Global tech debt indicators from the semantic analysis.

- Total findings: 0
- Estimated remediation hours: 0.0
- Findings by severity:
  - HIGH: 0
  - MEDIUM: 0
  - LOW: 0

The automated tech-debt profiler did not flag structural issues in this code subset. Risk is instead concentrated in business logic complexity (HABADTE has CC=152) and missing external dependencies.
