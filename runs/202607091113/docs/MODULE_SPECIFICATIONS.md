# Module Specifications – HABADTE Project

## Section 1 – Component Inventory

This section lists all source members discovered in the HABADTE project with basic sizing and risk indicators.

| Program | Type     | Subsystem | Lines | CC  | Risk Band | Orphan |
|---------|----------|-----------|-------|-----|-----------|--------|
| HAPIRNK | DDS_LF   | HAP       | 13    | N/A | N/A       | no     |
| HAPTRFR | DDS_PF   | HAP       | 72    | N/A | N/A       | no     |
| HMLMAST5H | DDS_LF | UNGROUPED | 12    | N/A | N/A       | no     |
| HXLTABLD | DDS_LF  | HXLT      | 11    | N/A | N/A       | no     |
| HXLTABLP | DDS_LF  | HXLT      | 12    | N/A | N/A       | no     |
| HXLTABLS | DDS_LF  | HXLT      | 12    | N/A | N/A       | no     |
| HXPBNFIT | DDS_LF  | HXP       | 12    | N/A | N/A       | no     |
| HXPDICT | DDS_PF   | HXP       | 6130  | N/A | N/A       | no     |
| HXPLVL1 | DDS_PF   | HXPL      | 49    | N/A | N/A       | no     |
| HXPLVL2 | DDS_PF   | HXPL      | 52    | N/A | N/A       | no     |
| HXPLVL3 | DDS_PF   | HXPL      | 52    | N/A | N/A       | no     |
| HXPLVL4 | DDS_PF   | HXPL      | 52    | N/A | N/A       | no     |
| HXPLVL5 | DDS_PF   | HXPL      | 55    | N/A | N/A       | no     |
| HXPLVL6 | DDS_PF   | HXPL      | 321   | N/A | N/A       | no     |
| HXPNSTN | DDS_LF   | HXP       | 12    | N/A | N/A       | no     |
| HXPTABLD | DDS_PF  | HXP       | 19    | N/A | N/A       | no     |
| HXPXMLD | DDS_PF   | HXPX      | 19    | N/A | N/A       | no     |
| HXPXMLR | DDS_PF   | HXPX      | 19    | N/A | N/A       | no     |
| OAPIRNK | DDS_PF   | UNGROUPED | 80    | N/A | N/A       | no     |
| OMPMAST | DDS_PF   | UNGROUPED | 310   | N/A | N/A       | no     |
| OXPBNFIT | DDS_PF  | OXP       | 48    | N/A | N/A       | no     |
| OXPNSTN | DDS_PF   | OXP       | 65    | N/A | N/A       | no     |
| TAPIRNK | DDS_PF   | UNGROUPED | 167   | N/A | N/A       | no     |
| TMPMAST | DDS_PF   | UNGROUPED | 688   | N/A | N/A       | no     |
| TXPBNFIT | DDS_PF  | TXP       | 164   | N/A | N/A       | no     |
| TXPNSTN | DDS_PF   | TXP       | 116   | N/A | N/A       | no     |
| HXXAPPPRF | SQLRPGLE | HXXA    | 123   | 1   | LOW       | yes    |
| XFXCNTR | RPGLE    | XFXC      | 49    | 3   | LOW       | yes    |
| XFXCYMD | RPGLE    | XFXC      | 83    | 7   | LOW       | yes    |
| XFXGETID | RPGLE   | XFX       | 61    | 1   | LOW       | yes    |
| XFXLDSC | RPGLE    | XFXL      | 135   | 5   | LOW       | yes    |
| XFXLEAP | RPGLE    | XFXL      | 61    | 1   | LOW       | yes    |
| XFXMRNROL | RPGLE  | XFX       | 65    | 1   | LOW       | yes    |
| XFXTABL | RPGLE    | XFX       | 164   | 9   | LOW       | yes    |
| CXXXMLP | SQLRPGLE | UNGROUPED | 25    | 1   | LOW       | yes    |
| HXXAPPPRFP | RPGLE | HXXA      | 42    | 1   | LOW       | yes    |
| HXXCNTRL | RPGLE   | HXX       | 8     | 1   | LOW       | yes    |
| HXXLDA | RPGLE     | HXXL      | 53    | 1   | LOW       | yes    |
| HXXLEVEL | RPGLE   | HXXL      | 25    | 1   | LOW       | yes    |
| HXXXML | RPGLE     | HXX       | 11    | 1   | LOW       | yes    |
| HABADTE | RPGLE    | HA        | 821   | 152 | HIGH      | yes    |

Notes:
- CC and Risk Band values are derived from the complexity_per_program section of the summary when available. Non-program members show "N/A".
- Orphan = "yes" when the program appears in the orphan_programs list.

## Section 2 – Dependency Hotspots

Programs with the highest combined complexity and dependency scores.

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
| XFXLEAP   | 3     | 1      | 0       | 0        |
| HXHAPPPRF | 3     | 1      | 0       | 0        |
| HMLMAST5H | 2     | 0      | 0       | 1        |
| HXLTABLD  | 2     | 0      | 0       | 1        |
| HXPNSTN   | 2     | 0      | 0       | 1        |
| HAPIRNK   | 2     | 0      | 0       | 1        |
| HXLTABLS  | 2     | 0      | 0       | 1        |
| HXLTABLP  | 2     | 0      | 0       | 1        |
| HXPBNFIT  | 2     | 0      | 0       | 1        |

HABADTE is the primary orchestration hotspot, with high complexity and significant fan-out.

## Section 3 – Call Graph Summary

This section summarizes the call and copy relationships between programs and files.

| From      | To         | Type |
|-----------|------------|------|
| XFXCYMD   | XFXLEAP    | CALL |
| XFXMRNROL | HXHAPPPRF  | CALL |
| XFXMRNROL | HXXAPPPRF  | CALL |
| HABADTE   | XFXMRNROL  | CALL |
| HABADTE   | XFXCNTR    | CALL |
| HABADTE   | XFXLDSC    | CALL |
| HABADTE   | XFXCNTR    | CALL |
| HABADTE   | XFXCNTR    | CALL |
| HABADTE   | XFXCYMD    | CALL |
| HABADTE   | XFXGETID   | CALL |
| HABADTE   | XFXTABL    | CALL |
| HXXAPPPRF | HXXCNTRL   | COPY |
| HXXAPPPRF | HXXAPPPRFP | COPY |
| XFXGETID  | HXXLDA     | COPY |
| HABADTE   | HXXLDA     | COPY |
| HABADTE   | HXXLEVEL   | COPY |
| HABADTE   | HXXXML     | COPY |
| HABADTE   | CXXXMLP    | COPY |
| HABADTE   | CXXXMLC    | COPY |
| HAPIRNK   | TAPIRNK    | PFILE_OF |
| HMLMAST5H | TMPMAST    | PFILE_OF |
| HXLTABLD  | HXPTABLD   | PFILE_OF |
| HXLTABLP  | HXPTABLD   | PFILE_OF |
| HXLTABLS  | HXPTABLD   | PFILE_OF |
| HXPBNFIT  | TXPBNFIT   | PFILE_OF |
| HXPNSTN   | TXPNSTN    | PFILE_OF |
| XFXGETID  | HXFXMLR    | READ |
| XFXLDSC   | HXFLVL1    | READ |
| XFXLDSC   | HXFLVL2    | READ |
| XFXLDSC   | HXFLVL3    | READ |
| XFXLDSC   | HXFLVL4    | READ |
| XFXLDSC   | HXFLVL5    | READ |
| XFXLDSC   | HXFLVL6    | READ |
| XFXTABL   | XFFTABLD   | READ |
| XFXTABL   | XFFTABL2   | READ |
| XFXTABL   | XFFTABL3   | READ |
| XFXTABL   | XFFTABL4   | READ |
| HABADTE   | HAPTRFR    | READ |
| HABADTE   | XFFNSTN    | READ |
| HABADTE   | HXFXMLH    | READ |
| HABADTE   | HXFXMLH    | UPDATE |
| HABADTE   | HXFXMLH    | WRITE |
| HABADTE   | HXFXMLD    | WRITE |

## Section 4 – Business Rules by Program

This section groups approved business rules by their source program.

### XFXCNTR

- BR-001 – Field formatting: exit if all blank (40 chars)
- BR-002 – Field formatting: exit if first char non-blank

### XFXCYMD

- BR-003 – Date validation: reject year < 1800 (historical minimum)
- BR-004 – Date validation: reject year > 2100 (forecast maximum)
- BR-005 – Date validation: reject month < 01 (calendar constraint)
- BR-006 – Date validation: reject month > 12 (calendar constraint)
- BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT'

### XFXLDSC

- BR-009 – Level lookup: reject if level code exceeds valid range
- BR-010 – Level lookup: reject if level code exceeds valid range
- BR-011 – Level lookup: reject if level code exceeds valid range
- BR-012 – Level lookup: reject if level code exceeds valid range

### XFXTABL

- BR-013 – When *IN79 equals on/active, branch to 'EXIT'
- BR-014 – When *IN79 equals on/active, branch to 'EXIT'
- BR-015 – When *IN79 equals on/active, branch to 'EXIT'
- BR-016 – When *IN79 equals on/active, branch to 'EXIT'

### HABADTE

- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'

### HXXAPPPRF

- BR-020 – SQL program accesses table 'HXPAPPPRF'

## Section 5 – Missing Dependencies

High and medium impact gaps identified during dependency analysis.

| Name      | Type     | Impact | Referenced By |
|-----------|----------|--------|---------------|
| CXXXMLC   | COPYBOOK | HIGH   | HABADTE       |
| HXHAPPPRF | PROGRAM  | MEDIUM | XFXMRNROL     |
| ****HXPXML | FILE    | MEDIUM | HABADTE       |
| PRINTER   | FILE     | MEDIUM | HABADTE       |

These missing components represent unresolved dependencies that must be addressed or replaced during modernization.

## Section 6 – Tech Debt Summary

Overall technical debt summary for the HABADTE project.

- Total findings: 4
- Total remediation hours (estimated): 26.9
- Findings by severity:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

Most risk is concentrated in the HABADTE program due to its high cyclomatic complexity and extensive fan-out.
