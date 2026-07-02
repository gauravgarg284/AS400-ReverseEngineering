# Module Specifications – HABADTE AS400 Reverse Engineering

## 1. Component Inventory

| Program      | Type     | Subsystem | Lines | CC  | Risk Band | Orphan |
|-------------|----------|-----------|-------|-----|-----------|--------|
| HAPIRNK     | DDS_LF   | HAP       | 13    | N/A | N/A       | No     |
| HAPTRFR     | DDS_PF   | HAP       | 72    | N/A | N/A       | No     |
| HMLMAST5H   | DDS_LF   | UNGROUPED | 12    | N/A | N/A       | No     |
| HXLTABLD    | DDS_LF   | HXLT      | 11    | N/A | N/A       | No     |
| HXLTABLP    | DDS_LF   | HXLT      | 12    | N/A | N/A       | No     |
| HXLTABLS    | DDS_LF   | HXLT      | 12    | N/A | N/A       | Yes    |
| HXPBNFIT    | DDS_LF   | HXP       | 12    | N/A | N/A       | Yes    |
| HXPDICT     | DDS_PF   | HXP       | 6130  | N/A | N/A       | No     |
| HXPLVL1     | DDS_PF   | HXPL      | 49    | N/A | N/A       | No     |
| HXPLVL2     | DDS_PF   | HXPL      | 52    | N/A | N/A       | No     |
| HXPLVL3     | DDS_PF   | HXPL      | 52    | N/A | N/A       | No     |
| HXPLVL4     | DDS_PF   | HXPL      | 52    | N/A | N/A       | No     |
| HXPLVL5     | DDS_PF   | HXPL      | 55    | N/A | N/A       | No     |
| HXPLVL6     | DDS_PF   | HXPL      | 321   | N/A | N/A       | No     |
| HXPNSTN     | DDS_LF   | HXP       | 12    | N/A | N/A       | Yes    |
| HXPTABLD    | DDS_PF   | HXP       | 19    | N/A | N/A       | No     |
| HXPXMLD     | DDS_PF   | HXPX      | 19    | N/A | N/A       | No     |
| HXPXMLR     | DDS_PF   | HXPX      | 19    | N/A | N/A       | No     |
| OAPIRNK     | DDS_PF   | UNGROUPED | 80    | N/A | N/A       | No     |
| OMPMAST     | DDS_PF   | UNGROUPED | 310   | N/A | N/A       | No     |
| OXPBNFIT    | DDS_PF   | OXP       | 48    | N/A | N/A       | Yes    |
| OXPNSTN     | DDS_PF   | OXP       | 65    | N/A | N/A       | Yes    |
| HXXAPPPRF   | SQLRPGLE | HXXA      | 123   | 1   | LOW       | Yes    |
| XFXCNTR     | RPGLE    | XFXC      | 49    | 3   | LOW       | Yes    |
| XFXCYMD     | RPGLE    | XFXC      | 83    | 7   | LOW       | Yes    |
| XFXGETID    | RPGLE    | XFX       | 61    | 1   | LOW       | Yes    |
| XFXLDSC     | RPGLE    | XFXL      | 135   | 5   | LOW       | Yes    |
| XFXLEAP     | RPGLE    | XFXL      | 61    | 1   | LOW       | Yes    |
| XFXMRNROL   | RPGLE    | XFX       | 65    | 1   | LOW       | Yes    |
| XFXTABL     | RPGLE    | XFX       | 164   | 9   | LOW       | Yes    |
| CXXXMLP     | SQLRPGLE | UNGROUPED | 25    | 1   | LOW       | No     |
| HXXAPPPRFP  | RPGLE    | HXXA      | 42    | 1   | LOW       | No     |
| HXXCNTRL    | RPGLE    | HXX       | 8     | 1   | LOW       | No     |
| HXXLDA      | RPGLE    | HXXL      | 53    | 1   | LOW       | No     |
| HXXLEVEL    | RPGLE    | HXXL      | 25    | 1   | LOW       | No     |
| HXXXML      | RPGLE    | HXX       | 11    | 1   | LOW       | No     |
| HABADTE     | RPGLE    | HA        | 821   | 152 | HIGH      | Yes    |

## 2. Dependency Hotspots

Programs with highest combined call and file-operation complexity.

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

## 3. Call Graph Summary

High-level view of calls, copy relationships, physical-file relationships, and file operations.

| From Program | To Object  | Edge Type   |
|--------------|-----------|-------------|
| XFXCYMD      | XFXLEAP   | CALL        |
| XFXMRNROL    | HXHAPPPRF | CALL        |
| XFXMRNROL    | HXXAPPPRF | CALL        |
| HABADTE      | XFXMRNROL | CALL        |
| HABADTE      | XFXCNTR   | CALL        |
| HABADTE      | XFXLDSC   | CALL        |
| HABADTE      | XFXCNTR   | CALL        |
| HABADTE      | XFXCNTR   | CALL        |
| HABADTE      | XFXCYMD   | CALL        |
| HABADTE      | XFXGETID  | CALL        |
| HABADTE      | XFXTABL   | CALL        |
| HXXAPPPRF    | HXXCNTRL  | COPY        |
| HXXAPPPRF    | HXXAPPPRFP| COPY        |
| XFXGETID     | HXXLDA    | COPY        |
| HABADTE      | HXXLDA    | COPY        |
| HABADTE      | HXXLEVEL  | COPY        |
| HABADTE      | HXXXML    | COPY        |
| HABADTE      | CXXXMLP   | COPY        |
| HABADTE      | CXXXMLC   | COPY        |
| HAPIRNK      | TAPIRNK   | PFILE_OF    |
| HMLMAST5H    | TMPMAST   | PFILE_OF    |
| HXLTABLD     | HXPTABLD  | PFILE_OF    |
| HXLTABLP     | HXPTABLD  | PFILE_OF    |
| HXLTABLS     | HXPTABLD  | PFILE_OF    |
| HXPBNFIT     | TXPBNFIT  | PFILE_OF    |
| HXPNSTN      | TXPNSTN   | PFILE_OF    |
| XFXGETID     | HXFXMLR   | READ        |
| XFXLDSC      | HXFLVL1   | READ        |
| XFXLDSC      | HXFLVL2   | READ        |
| XFXLDSC      | HXFLVL3   | READ        |
| XFXLDSC      | HXFLVL4   | READ        |
| XFXLDSC      | HXFLVL5   | READ        |
| XFXLDSC      | HXFLVL6   | READ        |
| XFXTABL      | XFFTABLD  | READ        |
| XFXTABL      | XFFTABL2  | READ        |
| XFXTABL      | XFFTABL3  | READ        |
| XFXTABL      | XFFTABL4  | READ        |
| HABADTE      | HAPTRFR   | READ        |
| HABADTE      | XFFNSTN   | READ        |
| HABADTE      | HXFXMLH   | READ        |
| HABADTE      | HXFXMLH   | UPDATE      |
| HABADTE      | HXFXMLH   | WRITE       |
| HABADTE      | HXFXMLD   | WRITE       |

## 4. Business Rules by Program

### Program XFXCNTR (Domain: DATA_MAINTENANCE)

- BR-001 – When X equals zero, branch to `EXIT`.
- BR-002 – When X equals 40, branch to `EXIT`.

These rules define exit conditions based on counter values, preventing further processing in boundary cases.

### Program XFXCYMD (Domain: DATA_MAINTENANCE)

- BR-003 – When `VYY` is less than 1800, branch to `EXIT`.
- BR-004 – When `VYY` is greater than 2100, branch to `EXIT`.
- BR-005 – When `VMM` is less than 01, branch to `EXIT`.
- BR-006 – When `VMM` is greater than 12, branch to `EXIT`.
- BR-007 – When `VDD` is less than 01, branch to `EXIT`.
- BR-008 – When `VDD` is greater than `DYS(VMM)`, branch to `EXIT`.

Collectively these rules enforce calendar date validity and prevent nonsensical year, month, and day combinations from entering downstream flows.

### Program XFXLDSC (Domain: DATA_MAINTENANCE)

- BR-009 – When `LDAMAP` is greater than 99, branch to `EXIT`.
- BR-010 – When `LDAMAP` is greater than 99, branch to `EXIT`.
- BR-011 – When `LDAMAP` is greater than 99, branch to `EXIT`.
- BR-012 – When `LDAMAP` is greater than 9999, branch to `EXIT`.

This module enforces upper bounds on mapping codes, protecting downstream logic from unsupported layout or mapping identifiers.

### Program XFXTABL (Domain: DATA_MAINTENANCE)

- BR-013 – When `*IN79` equals on/active, branch to `EXIT`.
- BR-014 – When `*IN79` equals on/active, branch to `EXIT`.
- BR-015 – When `*IN79` equals on/active, branch to `EXIT`.
- BR-016 – When `*IN79` equals on/active, branch to `EXIT`.

These rules tie a single indicator state (`*IN79`) to termination of processing, likely signalling a configuration or control flag that disables table maintenance.

### Program HABADTE (Domain: PATIENT_MANAGEMENT)

- BR-017 – When `-FILE INDICATOR` equals zero, branch to `SKIP`.
- BR-018 – When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.
- BR-019 – When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.

The HABADTE engine filters out records based on file status, void flags, and encounter type. Only non-voided inpatient records with valid file indicators progress into patient-account level processing.

## 5. Missing Dependencies

High- and medium-impact components referenced by the scanned source but not present in the export.

| Name      | Type     | Impact | Referenced By |
|-----------|----------|--------|---------------|
| CXXXMLC   | COPYBOOK | HIGH   | HABADTE       |
| HXHAPPPRF | PROGRAM  | MEDIUM | XFXMRNROL     |
| TAPIRNK   | FILE     | HIGH   | HAPIRNK       |
| TMPMAST   | FILE     | HIGH   | HMLMAST5H     |
| TXPBNFIT  | FILE     | HIGH   | HXPBNFIT      |
| TXPNSTN   | FILE     | HIGH   | HXPNSTN       |
| ****HXPXML| FILE     | MEDIUM | HABADTE       |
| PRINTER   | FILE     | MEDIUM | HABADTE       |

These gaps correspond to physical files, copybooks, or programs that are required for full behavior but not available in the analyzed snapshot. Migration teams must either locate these artifacts or design compensating logic.

## 6. Tech Debt Summary

Global technical-debt indicators derived from static analysis.

- Total findings: **0**
- Estimated remediation hours: **0.0**
- Findings by severity:
  - HIGH: 0
  - MEDIUM: 0
  - LOW: 0

Despite the absence of recorded tech-debt findings, the high cyclomatic complexity of HABADTE (CC 152, HIGH band) should be treated as a structural risk during modernization. Refactoring this module into smaller Spring Boot services and shared utilities is recommended.