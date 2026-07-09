# Module Specifications – HABADTE Project

## Section 1 – Component Inventory

| Program | Type | Subsystem | Lines | CC | Risk Band | Orphan |
|--------|------|-----------|-------|----|-----------|--------|
| HAPIRNK | DDS_LF | HAP | 13 | N/A | N/A | yes |
| HAPTRFR | DDS_PF | HAP | 72 | N/A | N/A | yes |
| HMLMAST5H | DDS_LF | UNGROUPED | 12 | N/A | N/A | yes |
| HXLTABLD | DDS_LF | HXLT | 11 | N/A | N/A | yes |
| HXLTABLP | DDS_LF | HXLT | 12 | N/A | N/A | yes |
| HXLTABLS | DDS_LF | HXLT | 12 | N/A | N/A | yes |
| HXPBNFIT | DDS_LF | HXP | 12 | N/A | N/A | yes |
| HXPDICT | DDS_PF | HXP | 6130 | N/A | N/A | yes |
| HXPLVL1 | DDS_PF | HXPL | 49 | N/A | N/A | yes |
| HXPLVL2 | DDS_PF | HXPL | 52 | N/A | N/A | yes |
| HXPLVL3 | DDS_PF | HXPL | 52 | N/A | N/A | yes |
| HXPLVL4 | DDS_PF | HXPL | 52 | N/A | N/A | yes |
| HXPLVL5 | DDS_PF | HXPL | 55 | N/A | N/A | yes |
| HXPLVL6 | DDS_PF | HXPL | 321 | N/A | N/A | yes |
| HXPNSTN | DDS_LF | HXP | 12 | N/A | N/A | yes |
| HXPTABLD | DDS_PF | HXP | 19 | N/A | N/A | yes |
| HXPXMLD | DDS_PF | HXPX | 19 | N/A | N/A | yes |
| HXPXMLR | DDS_PF | HXPX | 19 | N/A | N/A | yes |
| OAPIRNK | DDS_PF | UNGROUPED | 80 | N/A | N/A | yes |
| OMPMAST | DDS_PF | UNGROUPED | 310 | N/A | N/A | yes |
| OXPBNFIT | DDS_PF | OXP | 48 | N/A | N/A | yes |
| OXPNSTN | DDS_PF | OXP | 65 | N/A | N/A | yes |
| TAPIRNK | DDS_PF | UNGROUPED | 167 | N/A | N/A | yes |
| TMPMAST | DDS_PF | UNGROUPED | 688 | N/A | N/A | yes |
| TXPBNFIT | DDS_PF | TXP | 164 | N/A | N/A | yes |
| TXPNSTN | DDS_PF | TXP | 116 | N/A | N/A | yes |
| HXXAPPPRF | SQLRPGLE | HXXA | 123 | 1 | LOW | yes |
| XFXCNTR | RPGLE | XFXC | 49 | 3 | LOW | yes |
| XFXCYMD | RPGLE | XFXC | 83 | 7 | LOW | yes |
| XFXGETID | RPGLE | XFX | 61 | 1 | LOW | yes |
| XFXLDSC | RPGLE | XFXL | 135 | 5 | LOW | yes |
| XFXLEAP | RPGLE | XFXL | 61 | 1 | LOW | yes |
| XFXMRNROL | RPGLE | XFX | 65 | 1 | LOW | yes |
| XFXTABL | RPGLE | XFX | 164 | 9 | LOW | yes |
| CXXXMLP | SQLRPGLE | UNGROUPED | 25 | 1 | LOW | yes |
| HXXAPPPRFP | RPGLE | HXXA | 42 | 1 | LOW | yes |
| HXXCNTRL | RPGLE | HXX | 8 | 1 | LOW | yes |
| HXXLDA | RPGLE | HXXL | 53 | 1 | LOW | yes |
| HXXLEVEL | RPGLE | HXXL | 25 | 1 | LOW | yes |
| HXXXML | RPGLE | HXX | 11 | 1 | LOW | yes |
| HABADTE | RPGLE | HA | 821 | 152 | HIGH | yes |

> Note: Cyclomatic complexity (CC) and risk band values are populated where available from the aggregated complexity metrics; DDS files do not have CC metrics in this run.

## Section 2 – Dependency Hotspots

The following table lists programs identified as dependency hotspots based on fan-in, fan-out, and file operations.

| Program | Hotspot Score | Fan-in | Fan-out | File Ops |
|---------|---------------|--------|---------|----------|
| HABADTE | 38 | 0 | 13 | 6 |
| XFXLDSC | 15 | 1 | 0 | 6 |
| XFXTABL | 11 | 1 | 0 | 4 |
| XFXCNTR | 9 | 3 | 0 | 0 |
| XFXMRNROL | 7 | 1 | 2 | 0 |
| XFXGETID | 7 | 1 | 1 | 1 |
| HXXAPPPRF | 7 | 1 | 2 | 0 |
| XFXCYMD | 5 | 1 | 1 | 0 |
| XFXLEAP | 3 | 1 | 0 | 0 |
| HXHAPPPRF | 3 | 1 | 0 | 0 |
| HMLMAST5H | 2 | 0 | 0 | 1 |
| HXLTABLD | 2 | 0 | 0 | 1 |
| HXPNSTN | 2 | 0 | 0 | 1 |
| HAPIRNK | 2 | 0 | 0 | 1 |
| HXLTABLS | 2 | 0 | 0 | 1 |
| HXLTABLP | 2 | 0 | 0 | 1 |
| HXPBNFIT | 2 | 0 | 0 | 1 |

These hotspots should be prioritized for detailed testing and risk assessment during modernization, especially HABADTE as the primary orchestrator.

## Section 3 – Call Graph Summary

| Caller | Callee | Dependency Type |
|--------|--------|-----------------|
| XFXCYMD | XFXLEAP | CALL |
| XFXMRNROL | HXHAPPPRF | CALL |
| XFXMRNROL | HXXAPPPRF | CALL |
| HABADTE | XFXMRNROL | CALL |
| HABADTE | XFXCNTR | CALL |
| HABADTE | XFXLDSC | CALL |
| HABADTE | XFXCNTR | CALL |
| HABADTE | XFXCNTR | CALL |
| HABADTE | XFXCYMD | CALL |
| HABADTE | XFXGETID | CALL |
| HABADTE | XFXTABL | CALL |
| HXXAPPPRF | HXXCNTRL | COPY |
| HXXAPPPRF | HXXAPPPRFP | COPY |
| XFXGETID | HXXLDA | COPY |
| HABADTE | HXXLDA | COPY |
| HABADTE | HXXLEVEL | COPY |
| HABADTE | HXXXML | COPY |
| HABADTE | CXXXMLP | COPY |
| HABADTE | CXXXMLC | COPY |
| HAPIRNK | TAPIRNK | PFILE_OF |
| HMLMAST5H | TMPMAST | PFILE_OF |
| HXLTABLD | HXPTABLD | PFILE_OF |
| HXLTABLP | HXPTABLD | PFILE_OF |
| HXLTABLS | HXPTABLD | PFILE_OF |
| HXPBNFIT | TXPBNFIT | PFILE_OF |
| HXPNSTN | TXPNSTN | PFILE_OF |
| XFXGETID | HXFXMLR | READ |
| XFXLDSC | HXFLVL1 | READ |
| XFXLDSC | HXFLVL2 | READ |
| XFXLDSC | HXFLVL3 | READ |
| XFXLDSC | HXFLVL4 | READ |
| XFXLDSC | HXFLVL5 | READ |
| XFXLDSC | HXFLVL6 | READ |
| XFXTABL | XFFTABLD | READ |
| XFXTABL | XFFTABL2 | READ |
| XFXTABL | XFFTABL3 | READ |
| XFXTABL | XFFTABL4 | READ |
| HABADTE | HAPTRFR | READ |
| HABADTE | XFFNSTN | READ |
| HABADTE | HXFXMLH | READ |
| HABADTE | HXFXMLH | UPDATE |
| HABADTE | HXFXMLH | WRITE |
| HABADTE | HXFXMLD | WRITE |

This call graph shows HABADTE as the central orchestrator calling multiple utility programs and performing XML and file-based operations.

## Section 4 – Business Rules by Program

### HABADTE

- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

### HXXAPPPRF

- BR-020 – SQL program accesses table 'HXPAPPPRF'.

### XFXCNTR

- BR-001 – Field formatting: exit if all blank (40 chars).
- BR-002 – Field formatting: exit if first char non-blank.

### XFXCYMD

- BR-003 – Date validation: reject year < 1800 (historical minimum).
- BR-004 – Date validation: reject year > 2100 (forecast maximum).
- BR-005 – Date validation: reject month < 01 (calendar constraint).
- BR-006 – Date validation: reject month > 12 (calendar constraint).
- BR-007 – Date validation: reject day < 01 (calendar constraint).
- BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT'.

### XFXLDSC

- BR-009 – Level lookup: reject if level code exceeds valid range.
- BR-010 – Level lookup: reject if level code exceeds valid range.
- BR-011 – Level lookup: reject if level code exceeds valid range.
- BR-012 – Level lookup: reject if level code exceeds valid range.

### XFXTABL

- BR-013 – When *IN79 equals on/active, branch to 'EXIT'.
- BR-014 – When *IN79 equals on/active, branch to 'EXIT'.
- BR-015 – When *IN79 equals on/active, branch to 'EXIT'.
- BR-016 – When *IN79 equals on/active, branch to 'EXIT'.

## Section 5 – Missing Dependencies

| Name | Type | Impact | Referenced By |
|------|------|--------|---------------|
| CXXXMLC | COPYBOOK | HIGH | HABADTE |
| HXHAPPPRF | PROGRAM | MEDIUM | XFXMRNROL |
| ****HXPXML | FILE | MEDIUM | HABADTE |
| PRINTER | FILE | MEDIUM | HABADTE |

These gaps represent components referenced in the code but not present in the harvested source set. They must be resolved or reconstructed during modernization.

## Section 6 – Technical Debt Summary

Overall technical debt metrics from the aggregated analysis:

- Total findings: 4
- Total remediation hours (estimated): 26.9
- Findings by severity:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

The high-severity item corresponds to missing core artifacts and the high complexity of HABADTE, while medium items are associated with external dependencies and PHI-heavy structures. Addressing these findings should be part of the modernization backlog to reduce risk and improve maintainability.
