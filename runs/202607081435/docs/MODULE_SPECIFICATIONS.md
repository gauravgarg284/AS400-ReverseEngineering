# MODULE SPECIFICATIONS – HABADTE

## 1. Component Inventory

The HABADTE application consists of 41 source members across RPG, SQLRPGLE, and DDS. The table below summarizes every member using the aggregated source metadata.

| Program | Type    | Subsystem | Lines | CC  | Risk Band | Orphan |
|---------|---------|-----------|-------|-----|-----------|--------|
| HAPIRNK   | DDS_LF    | HAP       | 13    | N/A | N/A       | yes |
| HAPTRFR   | DDS_PF    | HAP       | 72    | N/A | N/A       | yes |
| HMLMAST5H | DDS_LF    | UNGROUPED | 12    | N/A | N/A       | yes |
| HXLTABLD  | DDS_LF    | HXLT      | 11    | N/A | N/A       | yes |
| HXLTABLP  | DDS_LF    | HXLT      | 12    | N/A | N/A       | yes |
| HXLTABLS  | DDS_LF    | HXLT      | 12    | N/A | N/A       | yes |
| HXPBNFIT  | DDS_LF    | HXP       | 12    | N/A | N/A       | yes |
| HXPDICT   | DDS_PF    | HXP       | 6130  | N/A | N/A       | yes |
| HXPLVL1   | DDS_PF    | HXPL      | 49    | N/A | N/A       | yes |
| HXPLVL2   | DDS_PF    | HXPL      | 52    | N/A | N/A       | yes |
| HXPLVL3   | DDS_PF    | HXPL      | 52    | N/A | N/A       | yes |
| HXPLVL4   | DDS_PF    | HXPL      | 52    | N/A | N/A       | yes |
| HXPLVL5   | DDS_PF    | HXPL      | 55    | N/A | N/A       | yes |
| HXPLVL6   | DDS_PF    | HXPL      | 321   | N/A | N/A       | yes |
| HXPNSTN   | DDS_LF    | HXP       | 12    | N/A | N/A       | yes |
| HXPTABLD  | DDS_PF    | HXP       | 19    | N/A | N/A       | yes |
| HXPXMLD   | DDS_PF    | HXPX      | 19    | N/A | N/A       | yes |
| HXPXMLR   | DDS_PF    | HXPX      | 19    | N/A | N/A       | yes |
| OAPIRNK   | DDS_PF    | UNGROUPED | 80    | N/A | N/A       | yes |
| OMPMAST   | DDS_PF    | UNGROUPED | 310   | N/A | N/A       | yes |
| OXPBNFIT  | DDS_PF    | OXP       | 48    | N/A | N/A       | yes |
| OXPNSTN   | DDS_PF    | OXP       | 65    | N/A | N/A       | yes |
| TAPIRNK   | DDS_PF    | UNGROUPED | 167   | N/A | N/A       | yes |
| TMPMAST   | DDS_PF    | UNGROUPED | 688   | N/A | N/A       | yes |
| TXPBNFIT  | DDS_PF    | TXP       | 164   | N/A | N/A       | yes |
| TXPNSTN   | DDS_PF    | TXP       | 116   | N/A | N/A       | yes |
| HXXAPPPRF | SQLRPGLE  | HXXA      | 123   | 1   | LOW       | yes |
| XFXCNTR   | RPGLE     | XFXC      | 49    | 3   | LOW       | yes |
| XFXCYMD   | RPGLE     | XFXC      | 83    | 7   | LOW       | yes |
| XFXGETID  | RPGLE     | XFX       | 61    | 1   | LOW       | yes |
| XFXLDSC   | RPGLE     | XFXL      | 135   | 5   | LOW       | yes |
| XFXLEAP   | RPGLE     | XFXL      | 61    | 1   | LOW       | yes |
| XFXMRNROL | RPGLE     | XFX       | 65    | 1   | LOW       | yes |
| XFXTABL   | RPGLE     | XFX       | 164   | 9   | LOW       | yes |
| CXXXMLP   | SQLRPGLE  | UNGROUPED | 25    | 1   | LOW       | yes |
| HXXAPPPRFP| RPGLE     | HXXA      | 42    | 1   | LOW       | yes |
| HXXCNTRL  | RPGLE     | HXX       | 8     | 1   | LOW       | yes |
| HXXLDA    | RPGLE     | HXXL      | 53    | 1   | LOW       | yes |
| HXXLEVEL  | RPGLE     | HXXL      | 25    | 1   | LOW       | yes |
| HXXXML    | RPGLE     | HXX       | 11    | 1   | LOW       | yes |
| HABADTE   | RPGLE     | HA        | 821   | 152 | HIGH      | yes |

> Note: Orphan column is based on the aggregated orphan_programs list and does not imply the component is unused in production; it simply has no callers inside the harvested codebase.

## 2. Dependency Hotspots

The following table lists programs identified as dependency hotspots, combining fan-in, fan-out, and file operation counts.

| Program   | Hotspot Score | Fan-In | Fan-Out | File Ops |
|-----------|---------------|--------|---------|----------|
| HABADTE   | 38            | 0      | 13      | 6        |
| XFXLDSC   | 15            | 1      | 0       | 6        |
| XFXTABL   | 11            | 1      | 0       | 4        |
| XFXCNTR   | 9             | 3      | 0       | 0        |
| XFXGETID  | 7             | 1      | 1       | 1        |
| XFXMRNROL | 7             | 1      | 2       | 0        |
| HXXAPPPRF | 7             | 1      | 2       | 0        |
| XFXCYMD   | 5             | 1      | 1       | 0        |
| XFXLEAP   | 3             | 1      | 0       | 0        |
| HXHAPPPRF | 3             | 1      | 0       | 0        |
| HMLMAST5H | 2             | 0      | 0       | 1        |
| HXLTABLS  | 2             | 0      | 0       | 1        |
| HXPNSTN   | 2             | 0      | 0       | 1        |
| HAPIRNK   | 2             | 0      | 0       | 1        |
| HXPBNFIT  | 2             | 0      | 0       | 1        |
| HXLTABLP  | 2             | 0      | 0       | 1        |
| HXLTABLD  | 2             | 0      | 0       | 1        |

## 3. Call Graph Summary

This section summarizes the main dependency edges between programs and files.

| From (Caller) | To (Callee/Target) | Edge Type |
|---------------|--------------------|-----------|
| XFXCYMD       | XFXLEAP            | CALL      |
| XFXMRNROL     | HXHAPPPRF          | CALL      |
| XFXMRNROL     | HXXAPPPRF          | CALL      |
| HABADTE       | XFXMRNROL          | CALL      |
| HABADTE       | XFXCNTR            | CALL      |
| HABADTE       | XFXLDSC            | CALL      |
| HABADTE       | XFXCNTR            | CALL      |
| HABADTE       | XFXCNTR            | CALL      |
| HABADTE       | XFXCYMD            | CALL      |
| HABADTE       | XFXGETID           | CALL      |
| HABADTE       | XFXTABL            | CALL      |
| HXXAPPPRF     | HXXCNTRL           | COPY      |
| HXXAPPPRF     | HXXAPPPRFP         | COPY      |
| XFXGETID      | HXXLDA             | COPY      |
| HABADTE       | HXXLDA             | COPY      |
| HABADTE       | HXXLEVEL           | COPY      |
| HABADTE       | HXXXML             | COPY      |
| HABADTE       | CXXXMLP            | COPY      |
| HABADTE       | CXXXMLC            | COPY      |
| HAPIRNK       | TAPIRNK            | PFILE_OF  |
| HMLMAST5H     | TMPMAST            | PFILE_OF  |
| HXLTABLD      | HXPTABLD           | PFILE_OF  |
| HXLTABLP      | HXPTABLD           | PFILE_OF  |
| HXLTABLS      | HXPTABLD           | PFILE_OF  |
| HXPBNFIT      | TXPBNFIT           | PFILE_OF  |
| HXPNSTN       | TXPNSTN            | PFILE_OF  |
| XFXGETID      | HXFXMLR            | READ      |
| XFXLDSC       | HXFLVL1            | READ      |
| XFXLDSC       | HXFLVL2            | READ      |
| XFXLDSC       | HXFLVL3            | READ      |
| XFXLDSC       | HXFLVL4            | READ      |
| XFXLDSC       | HXFLVL5            | READ      |
| XFXLDSC       | HXFLVL6            | READ      |
| XFXTABL       | XFFTABLD           | READ      |
| XFXTABL       | XFFTABL2           | READ      |
| XFXTABL       | XFFTABL3           | READ      |
| XFXTABL       | XFFTABL4           | READ      |
| HABADTE       | HAPTRFR            | READ      |
| HABADTE       | XFFNSTN            | READ      |
| HABADTE       | HXFXMLH            | READ      |
| HABADTE       | HXFXMLH            | UPDATE    |
| HABADTE       | HXFXMLH            | WRITE     |
| HABADTE       | HXFXMLD            | WRITE     |

These edges show HABADTE as the primary orchestrator, delegating work to XFX* utilities and interacting with patient transfer, status, and XML header/detail files.

## 4. Business Rules by Program

Business rules have been grouped by their source program based on the approved_rules list.

### XFXCNTR

RPGLE utility in the DATA_MAINTENANCE domain responsible for field-formatting exits.

- **BR-001** – Field formatting: exit if all blank (40 chars)
- **BR-002** – Field formatting: exit if first char non-blank

These rules ensure that 40-character input fields either contain meaningful content or cause an immediate exit from the validation routine.

### XFXCYMD

RPGLE date validation routine in the DATA_MAINTENANCE domain.

- **BR-003** – Date validation: reject year < 1800 (historical minimum)
- **BR-004** – Date validation: reject year > 2100 (forecast maximum)
- **BR-005** – Date validation: reject month < 01 (calendar constraint)
- **BR-006** – Date validation: reject month > 12 (calendar constraint)
- **BR-007** – Date validation: reject day < 01 (calendar constraint)
- **BR-008** – When VDD is greater than DYS(VMM), branch to 'EXIT'

Collectively, these rules enforce valid calendar ranges and prevent obviously incorrect date values from entering downstream processing.

### XFXLDSC

RPGLE level-lookup program in the DATA_MAINTENANCE domain.

- **BR-009** – Level lookup: reject if level code exceeds valid range
- **BR-010** – Level lookup: reject if level code exceeds valid range
- **BR-011** – Level lookup: reject if level code exceeds valid range
- **BR-012** – Level lookup: reject if level code exceeds valid range

Although the rules are structurally identical, they likely apply to different hierarchical levels (e.g., facility, region, ward), ensuring only configured level codes pass validation.

### XFXTABL

RPGLE table-lookup program in the DATA_MAINTENANCE domain.

- **BR-013** – When *IN79 equals on/active, branch to 'EXIT'
- **BR-014** – When *IN79 equals on/active, branch to 'EXIT'
- **BR-015** – When *IN79 equals on/active, branch to 'EXIT'
- **BR-016** – When *IN79 equals on/active, branch to 'EXIT'

These rules guard table-driven logic based on indicator *IN79. When the indicator is in an "on/active" state, the routine short-circuits by branching to EXIT, preventing certain actions.

### HABADTE

RPGLE orchestration program in the PATIENT_MANAGEMENT domain.

- **BR-017** – When -FILE INDICATOR equals zero, branch to 'SKIP'
- **BR-018** – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'
- **BR-019** – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'

These high-confidence rules drive the core patient management workflow, determining when records should be skipped based on file status, void flags, and inpatient/outpatient flags.

### HXXAPPPRF

SQLRPGLE application-profile program in the DATA_MAINTENANCE domain.

- **BR-020** – SQL program accesses table 'HXPAPPPRF'

This rule confirms that HXXAPPPRF reads from the HXPAPPPRF table as part of its profile-maintenance logic.

## 5. Missing Dependencies

The following components are referenced from the codebase but were not found in the harvested sources.

| Name      | Type      | Impact  | Referenced By |
|-----------|-----------|---------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH    | HABADTE       |
| HXHAPPPRF | PROGRAM   | MEDIUM  | XFXMRNROL     |
| ****HXPXML| FILE      | MEDIUM  | HABADTE       |
| PRINTER   | FILE      | MEDIUM  | HABADTE       |

- **CXXXMLC (COPYBOOK, HIGH)** – Likely contains shared XML structures or constants. Without it, XML-related logic in HABADTE and CXXXMLP cannot be fully validated.
- **HXHAPPPRF (PROGRAM, MEDIUM)** – Called from XFXMRNROL. Represents part of the MRN/application profile workflow and must be recovered or stubbed for end-to-end equivalence.
- **"****HXPXML" (FILE, MEDIUM)** – An XML-related file used by HABADTE, probably for special-purpose payloads or staging.
- **PRINTER (FILE, MEDIUM)** – External printer or spool file used by HABADTE to produce reports or statements.

## 6. Tech Debt Summary

The aggregated tech-debt analysis for the HABADTE application is summarized below.

- **Total findings:** 4
- **Estimated remediation hours:** 26.9
- **By severity:**
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

The single HIGH-severity item is associated with HABADTE’s high cyclomatic complexity (152, HIGH band) and hotspot score. The MEDIUM findings align with missing dependencies (HXHAPPPRF, CXXXMLC, ****HXPXML, PRINTER) and integration gaps. Addressing these items will reduce modernization risk and improve maintainability in the target Java/Spring Boot/SQL Server implementation.
