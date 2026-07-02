# MODULE SPECIFICATIONS – HABADTE Project

## 1. Component Inventory

The table below lists all source members discovered in the HABADTE codebase, enriched with cyclomatic complexity, risk band, and orphan status from the aggregated context.

| Program    | Type     | Subsystem | Lines | CC  | Risk Band | Orphan |
|-----------|----------|-----------|-------|-----|-----------|--------|
| HAPIRNK   | DDS_LF   | HAP       | 13    | N/A | N/A       | no     |
| HAPTRFR   | DDS_PF   | HAP       | 72    | N/A | N/A       | no     |
| HMLMAST5H | DDS_LF   | UNGROUPED | 12    | N/A | N/A       | no     |
| HXLTABLD  | DDS_LF   | HXLT      | 11    | N/A | N/A       | no     |
| HXLTABLP  | DDS_LF   | HXLT      | 12    | N/A | N/A       | no     |
| HXLTABLS  | DDS_LF   | HXLT      | 12    | N/A | N/A       | no     |
| HXPBNFIT  | DDS_LF   | HXP       | 12    | N/A | N/A       | no     |
| HXPDICT   | DDS_PF   | HXP       | 6130  | N/A | N/A       | no     |
| HXPLVL1   | DDS_PF   | HXPL      | 49    | N/A | N/A       | no     |
| HXPLVL2   | DDS_PF   | HXPL      | 52    | N/A | N/A       | no     |
| HXPLVL3   | DDS_PF   | HXPL      | 52    | N/A | N/A       | no     |
| HXPLVL4   | DDS_PF   | HXPL      | 52    | N/A | N/A       | no     |
| HXPLVL5   | DDS_PF   | HXPL      | 55    | N/A | N/A       | no     |
| HXPLVL6   | DDS_PF   | HXPL      | 321   | N/A | N/A       | no     |
| HXPNSTN   | DDS_LF   | HXP       | 12    | N/A | N/A       | no     |
| HXPTABLD  | DDS_PF   | HXP       | 19    | N/A | N/A       | no     |
| HXPXMLD   | DDS_PF   | HXPX      | 19    | N/A | N/A       | no     |
| HXPXMLR   | DDS_PF   | HXPX      | 19    | N/A | N/A       | no     |
| OAPIRNK   | DDS_PF   | UNGROUPED | 80    | N/A | N/A       | no     |
| OMPMAST   | DDS_PF   | UNGROUPED | 310   | N/A | N/A       | no     |
| OXPBNFIT  | DDS_PF   | OXP       | 48    | N/A | N/A       | no     |
| OXPNSTN   | DDS_PF   | OXP       | 65    | N/A | N/A       | no     |
| HXXAPPPRF | SQLRPGLE | HXXA      | 123   | 1   | LOW       | yes    |
| XFXCNTR   | RPGLE    | XFXC      | 49    | 3   | LOW       | yes    |
| XFXCYMD   | RPGLE    | XFXC      | 83    | 7   | LOW       | yes    |
| XFXGETID  | RPGLE    | XFX       | 61    | 1   | LOW       | yes    |
| XFXLDSC   | RPGLE    | XFXL      | 135   | 5   | LOW       | yes    |
| XFXLEAP   | RPGLE    | XFXL      | 61    | 1   | LOW       | yes    |
| XFXMRNROL | RPGLE    | XFX       | 65    | 1   | LOW       | yes    |
| XFXTABL   | RPGLE    | XFX       | 164   | 9   | LOW       | yes    |
| CXXXMLP   | SQLRPGLE | UNGROUPED | 25    | 1   | LOW       | no     |
| HXXAPPPRFP| RPGLE    | HXXA      | 42    | 1   | LOW       | no     |
| HXXCNTRL  | RPGLE    | HXX       | 8     | 1   | LOW       | no     |
| HXXLDA    | RPGLE    | HXXL      | 53    | 1   | LOW       | no     |
| HXXLEVEL  | RPGLE    | HXXL      | 25    | 1   | LOW       | yes    |
| HXXXML    | RPGLE    | HXX       | 11    | 1   | LOW       | no     |
| HABADTE   | RPGLE    | HA        | 821   | 152 | HIGH      | yes    |

Notes:
- Orphan = yes where the member appears in `orphan_programs` (no callers in the extracted graph).
- CC and Risk Band are populated from `complexity_per_program`; DDS members do not have complexity scores, so "N/A" is used.

## 2. Dependency Hotspots

Dependency hotspots highlight structurally important programs based on fan‑in, fan‑out, and file operations.

| Program    | Score | Fan-In | Fan-Out | File Ops |
|------------|-------|--------|---------|----------|
| HABADTE    | 38    | 0      | 13      | 6        |
| XFXLDSC    | 15    | 1      | 0       | 6        |
| XFXTABL    | 11    | 1      | 0       | 4        |
| XFXCNTR    | 9     | 3      | 0       | 0        |
| XFXGETID   | 7     | 1      | 1       | 1        |
| HXXAPPPRF  | 7     | 1      | 2       | 0        |
| XFXMRNROL  | 7     | 1      | 2       | 0        |
| XFXCYMD    | 5     | 1      | 1       | 0        |
| XFXLEAP    | 3     | 1      | 0       | 0        |
| HXHAPPPRF  | 3     | 1      | 0       | 0        |
| HXPBNFIT   | 2     | 0      | 0       | 1        |
| HXPNSTN    | 2     | 0      | 0       | 1        |
| HXLTABLS   | 2     | 0      | 0       | 1        |
| HMLMAST5H  | 2     | 0      | 0       | 1        |
| HXLTABLP   | 2     | 0      | 0       | 1        |
| HAPIRNK    | 2     | 0      | 0       | 1        |
| HXLTABLD   | 2     | 0      | 0       | 1        |

HABADTE is the main orchestration program, while the XFX* utilities act as shared services for date validation, level description, table lookups, counters, and ID resolution.

## 3. Call Graph Summary

Call, copy, and file‑relationship edges capture how programs interact.

| Caller     | Callee    | Relationship Type |
|------------|-----------|-------------------|
| XFXCYMD    | XFXLEAP   | CALL              |
| XFXMRNROL  | HXHAPPPRF | CALL              |
| XFXMRNROL  | HXXAPPPRF | CALL              |
| HABADTE    | XFXMRNROL | CALL              |
| HABADTE    | XFXCNTR   | CALL              |
| HABADTE    | XFXLDSC   | CALL              |
| HABADTE    | XFXCNTR   | CALL              |
| HABADTE    | XFXCNTR   | CALL              |
| HABADTE    | XFXCYMD   | CALL              |
| HABADTE    | XFXGETID  | CALL              |
| HABADTE    | XFXTABL   | CALL              |
| HXXAPPPRF  | HXXCNTRL  | COPY              |
| HXXAPPPRF  | HXXAPPPRFP| COPY              |
| XFXGETID   | HXXLDA    | COPY              |
| HABADTE    | HXXLDA    | COPY              |
| HABADTE    | HXXLEVEL  | COPY              |
| HABADTE    | HXXXML    | COPY              |
| HABADTE    | CXXXMLP   | COPY              |
| HABADTE    | CXXXMLC   | COPY              |
| HAPIRNK    | TAPIRNK   | PFILE_OF          |
| HMLMAST5H  | TMPMAST   | PFILE_OF          |
| HXLTABLD   | HXPTABLD  | PFILE_OF          |
| HXLTABLP   | HXPTABLD  | PFILE_OF          |
| HXLTABLS   | HXPTABLD  | PFILE_OF          |
| HXPBNFIT   | TXPBNFIT  | PFILE_OF          |
| HXPNSTN    | TXPNSTN   | PFILE_OF          |
| XFXGETID   | HXFXMLR   | READ              |
| XFXLDSC    | HXFLVL1   | READ              |
| XFXLDSC    | HXFLVL2   | READ              |
| XFXLDSC    | HXFLVL3   | READ              |
| XFXLDSC    | HXFLVL4   | READ              |
| XFXLDSC    | HXFLVL5   | READ              |
| XFXLDSC    | HXFLVL6   | READ              |
| XFXTABL    | XFFTABLD  | READ              |
| XFXTABL    | XFFTABL2  | READ              |
| XFXTABL    | XFFTABL3  | READ              |
| XFXTABL    | XFFTABL4  | READ              |
| HABADTE    | HAPTRFR   | READ              |
| HABADTE    | XFFNSTN   | READ              |
| HABADTE    | HXFXMLH   | READ              |
| HABADTE    | HXFXMLH   | UPDATE            |
| HABADTE    | HXFXMLH   | WRITE             |
| HABADTE    | HXFXMLD   | WRITE             |

This call graph confirms HABADTE as the primary entry point and shows how configuration and dictionary files are accessed through XFX* utilities.

## 4. Business Rules by Program

Business rules are grouped by their `source_program` and referenced by BR-ID.

### XFXCNTR

- BR-001 – When X equals zero, branch to "EXIT".
- BR-002 – When X equals 40, branch to "EXIT".

These rules implement counter‑based exit conditions, used to terminate loops or processing after threshold checks.

### XFXCYMD

- BR-003 – When VYY is less than 1800, branch to "EXIT".
- BR-004 – When VYY is greater than 2100, branch to "EXIT".
- BR-005 – When VMM is less than 01, branch to "EXIT".
- BR-006 – When VMM is greater than 12, branch to "EXIT".
- BR-007 – When VDD is less than 01, branch to "EXIT".
- BR-008 – When VDD is greater than DYS(VMM), branch to "EXIT".

Together these rules enforce valid calendar dates before further processing.

### XFXLDSC

- BR-009 – When LDAMAP is greater than 99, branch to "EXIT".
- BR-010 – When LDAMAP is greater than 99, branch to "EXIT".
- BR-011 – When LDAMAP is greater than 99, branch to "EXIT".
- BR-012 – When LDAMAP is greater than 9999, branch to "EXIT".

These rules constrain allowed mapping codes, protecting against misconfigured level mappings.

### XFXTABL

- BR-013 – When *IN79 equals on/active, branch to "EXIT".
- BR-014 – When *IN79 equals on/active, branch to "EXIT".
- BR-015 – When *IN79 equals on/active, branch to "EXIT".
- BR-016 – When *IN79 equals on/active, branch to "EXIT".

These rules represent table‑driven exit conditions controlled by indicator *IN79.

### HABADTE

- BR-017 – When -FILE INDICATOR equals zero, branch to "SKIP".
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to "SKIP".
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to "SKIP".

In HABADTE these rules implement record‑level inclusion/exclusion logic based on file status, void flags, and patient type.

## 5. Missing Dependencies

Critical and medium‑impact gaps must be resolved or explicitly modelled during modernization.

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

These missing members affect XML payload layout, MRN roll‑up logic, ranking and master history views, benefits and notification files, and external output devices.

## 6. Tech Debt Summary

Overall tech‑debt metrics from automated analysis:

- Total findings: **0**
- Total remediation hours: **0.0**
- By severity:
  - HIGH: 0
  - MEDIUM: 0
  - LOW: 0

Although formal tech‑debt flags are zero, practical complexity and missing dependencies indicate structural risk, particularly in HABADTE and its XML and data‑feed integrations.
