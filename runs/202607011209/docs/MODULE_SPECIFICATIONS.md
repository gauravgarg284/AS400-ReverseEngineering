# Module Specifications – HABADTE Project

## 1. Component Inventory

| Program | Type    | Subsystem | Lines | CC | Risk Band | Orphan |
|--------|---------|-----------|-------|----|-----------|--------|
| HAPIRNK   | DDS_LF   | HAP        | 13   | N/A | N/A | yes |
| HAPTRFR   | DDS_PF   | HAP        | 72   | N/A | N/A | no |
| HMLMAST5H | DDS_LF   | UNGROUPED  | 12   | N/A | N/A | no |
| HXLTABLD  | DDS_LF   | HXLT       | 11   | N/A | N/A | no |
| HXLTABLP  | DDS_LF   | HXLT       | 12   | N/A | N/A | no |
| HXLTABLS  | DDS_LF   | HXLT       | 12   | N/A | N/A | yes |
| HXPBNFIT  | DDS_LF   | HXP        | 12   | N/A | N/A | yes |
| HXPDICT   | DDS_PF   | HXP        | 6130 | N/A | N/A | no |
| HXPLVL1   | DDS_PF   | HXPL       | 49   | N/A | N/A | no |
| HXPLVL2   | DDS_PF   | HXPL       | 52   | N/A | N/A | no |
| HXPLVL3   | DDS_PF   | HXPL       | 52   | N/A | N/A | no |
| HXPLVL4   | DDS_PF   | HXPL       | 52   | N/A | N/A | no |
| HXPLVL5   | DDS_PF   | HXPL       | 55   | N/A | N/A | no |
| HXPLVL6   | DDS_PF   | HXPL       | 321  | N/A | N/A | no |
| HXPNSTN   | DDS_LF   | HXP        | 12   | N/A | N/A | yes |
| HXPTABLD  | DDS_PF   | HXP        | 19   | N/A | N/A | no |
| HXPXMLD   | DDS_PF   | HXPX       | 19   | N/A | N/A | no |
| HXPXMLR   | DDS_PF   | HXPX       | 19   | N/A | N/A | no |
| OAPIRNK   | DDS_PF   | UNGROUPED  | 80   | N/A | N/A | no |
| OMPMAST   | DDS_PF   | UNGROUPED  | 310  | N/A | N/A | no |
| OXPBNFIT  | DDS_PF   | OXP        | 48   | N/A | N/A | no |
| OXPNSTN   | DDS_PF   | OXP        | 65   | N/A | N/A | no |
| HXXAPPPRF | SQLRPGLE | HXXA       | 123  | 1  | LOW | yes |
| XFXCNTR   | RPGLE    | XFXC       | 49   | 3  | LOW | yes |
| XFXCYMD   | RPGLE    | XFXC       | 83   | 7  | LOW | yes |
| XFXGETID  | RPGLE    | XFX        | 61   | 1  | LOW | yes |
| XFXLDSC   | RPGLE    | XFXL       | 135  | 5  | LOW | no |
| XFXLEAP   | RPGLE    | XFXL       | 61   | 1  | LOW | yes |
| XFXMRNROL | RPGLE    | XFX        | 65   | 1  | LOW | yes |
| XFXTABL   | RPGLE    | XFX        | 164  | 9  | LOW | yes |
| CXXXMLP   | SQLRPGLE | UNGROUPED  | 25   | 1  | LOW | no |
| HXXAPPPRFP| RPGLE    | HXXA       | 42   | 1  | LOW | no |
| HXXCNTRL  | RPGLE    | HXX        | 8    | 1  | LOW | no |
| HXXLDA    | RPGLE    | HXXL       | 53   | 1  | LOW | no |
| HXXLEVEL  | RPGLE    | HXXL       | 25   | 1  | LOW | no |
| HXXXML    | RPGLE    | HXX        | 11   | 1  | LOW | no |
| HABADTE   | RPGLE    | HA         | 821  | 152| HIGH | yes |

## 2. Dependency Hotspots

Programs with highest structural and operational complexity:

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

HABADTE stands out as the central orchestration program for patient management, driving calls to multiple utility and lookup modules and performing file IO across transfer, stay, and XML logging structures.

## 3. Call Graph Summary

Key dependency edges (program-to-program, copybook, and file relationships):

| From       | To         | Type      |
|------------|------------|-----------|
| XFXCYMD    | XFXLEAP    | CALL      |
| XFXMRNROL  | HXHAPPPRF  | CALL      |
| XFXMRNROL  | HXXAPPPRF  | CALL      |
| HABADTE    | XFXMRNROL  | CALL      |
| HABADTE    | XFXCNTR    | CALL      |
| HABADTE    | XFXLDSC    | CALL      |
| HABADTE    | XFXCNTR    | CALL      |
| HABADTE    | XFXCNTR    | CALL      |
| HABADTE    | XFXCYMD    | CALL      |
| HABADTE    | XFXGETID   | CALL      |
| HABADTE    | XFXTABL    | CALL      |
| HXXAPPPRF  | HXXCNTRL   | COPY      |
| HXXAPPPRF  | HXXAPPPRFP | COPY      |
| XFXGETID   | HXXLDA     | COPY      |
| HABADTE    | HXXLDA     | COPY      |
| HABADTE    | HXXLEVEL   | COPY      |
| HABADTE    | HXXXML     | COPY      |
| HABADTE    | CXXXMLP    | COPY      |
| HABADTE    | CXXXMLC    | COPY      |
| HAPIRNK    | TAPIRNK    | PFILE_OF  |
| HMLMAST5H  | TMPMAST    | PFILE_OF  |
| HXLTABLD   | HXPTABLD   | PFILE_OF  |
| HXLTABLP   | HXPTABLD   | PFILE_OF  |
| HXLTABLS   | HXPTABLD   | PFILE_OF  |
| HXPBNFIT   | TXPBNFIT   | PFILE_OF  |
| HXPNSTN    | TXPNSTN    | PFILE_OF  |
| XFXGETID   | HXFXMLR    | READ      |
| XFXLDSC    | HXFLVL1    | READ      |
| XFXLDSC    | HXFLVL2    | READ      |
| XFXLDSC    | HXFLVL3    | READ      |
| XFXLDSC    | HXFLVL4    | READ      |
| XFXLDSC    | HXFLVL5    | READ      |
| XFXLDSC    | HXFLVL6    | READ      |
| XFXTABL    | XFFTABLD   | READ      |
| XFXTABL    | XFFTABL2   | READ      |
| XFXTABL    | XFFTABL3   | READ      |
| XFXTABL    | XFFTABL4   | READ      |
| HABADTE    | HAPTRFR    | READ      |
| HABADTE    | XFFNSTN    | READ      |
| HABADTE    | HXFXMLH    | READ      |
| HABADTE    | HXFXMLH    | UPDATE    |
| HABADTE    | HXFXMLH    | WRITE     |
| HABADTE    | HXFXMLD    | WRITE     |

This call graph shows HABADTE at the center of the patient-management flow, delegating ID and level lookups to XFXGETID and XFXLDSC, table-driven logic to XFXTABL, and date/counter utilities to XFXCYMD and XFXCNTR.

## 4. Business Rules by Program

### HABADTE (PATIENT_MANAGEMENT)

| Rule ID | Rule Text | Confidence | PHI in Path |
|---------|-----------|------------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP' | 0.92 | false |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP' | 0.99 | false |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' | 0.99 | false |

These rules control which transfer records are considered valid inpatient activity for inclusion in downstream reporting and XML export.

### XFXCNTR (DATA_MAINTENANCE)

| Rule ID | Rule Text | Confidence | PHI in Path |
|---------|-----------|------------|-------------|
| BR-001 | When X equals zero, branch to 'EXIT' | 0.47 | false |
| BR-002 | When X equals 40, branch to 'EXIT' | 0.40 | false |

XFXCNTR appears to manage a numeric counter or iteration index, terminating processing when the counter reaches specific boundary values (0 or 40).

### XFXCYMD (DATA_MAINTENANCE)

| Rule ID | Rule Text | Confidence | PHI in Path |
|---------|-----------|------------|-------------|
| BR-003 | When VYY is less than 1800, branch to 'EXIT' | 0.56 | false |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT' | 0.56 | false |
| BR-005 | When VMM is less than 01, branch to 'EXIT' | 0.56 | false |
| BR-006 | When VMM is greater than 12, branch to 'EXIT' | 0.56 | false |
| BR-007 | When VDD is less than 01, branch to 'EXIT' | 0.56 | false |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT' | 0.68 | false |

XFXCYMD enforces calendar validity around year, month, and day values, looping out of processing when dates fall outside the supported range or exceed the days in a given month.

### XFXLDSC (DATA_MAINTENANCE)

| Rule ID | Rule Text | Confidence | PHI in Path |
|---------|-----------|------------|-------------|
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'   | 0.56 | false |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'   | 0.56 | false |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'   | 0.56 | false |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT' | 0.56 | false |

XFXLDSC uses LDAMAP thresholds to constrain lookup or mapping keys, protecting downstream file reads on HXPLVL* tables from invalid mapping values.

### XFXTABL (DATA_MAINTENANCE)

| Rule ID | Rule Text | Confidence | PHI in Path |
|---------|-----------|------------|-------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT' | 0.90 | false |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT' | 0.90 | false |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT' | 0.90 | false |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT' | 0.90 | false |

XFXTABL applies a panel indicator (*IN79) to short-circuit table-driven logic when a certain UI or control condition is active, preventing unnecessary table traversals.

## 5. Missing Dependencies (High/Medium Gaps)

These represent copybooks or files referenced by compiled members but missing from the source set.

| Name      | Type      | Impact  | Referenced By |
|-----------|-----------|---------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH    | HABADTE       |
| HXHAPPPRF | PROGRAM   | MEDIUM  | XFXMRNROL     |
| TAPIRNK   | FILE      | HIGH    | HAPIRNK       |
| TMPMAST   | FILE      | HIGH    | HMLMAST5H     |
| TXPBNFIT  | FILE      | HIGH    | HXPBNFIT      |
| TXPNSTN   | FILE      | HIGH    | HXPNSTN       |
| ****HXPXML| FILE      | MEDIUM  | HABADTE       |
| PRINTER   | FILE      | MEDIUM  | HABADTE       |

High-impact gaps are predominantly in physical files that underpin logical files for patient benefits (TXPBNFIT), station information (TXPNSTN), and master patient records (TMPMAST), as well as an XML-related file used by HABADTE. These will need remediation or replacement definitions during migration.

## 6. Tech Debt Summary

The aggregated tech debt summary for this run indicates no formal findings were recorded:

- **Total findings:** 0
- **Estimated remediation hours:** 0.0
- **By severity:**
  - HIGH: 0
  - MEDIUM: 0
  - LOW: 0

Despite the absence of explicit tech-debt items, the high cyclomatic complexity of HABADTE (152, HIGH band) and its fan-out of 13 calls suggest an architectural refactor is advisable during modernization (separating patient transfer selection, enrichment, and XML export into distinct services).