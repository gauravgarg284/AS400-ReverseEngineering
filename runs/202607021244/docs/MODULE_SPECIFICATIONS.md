# MODULE SPECIFICATIONS – HABADTE Run 202607021244

## 1. Component Inventory

### 1.1 Source Members

| Program    | Type     | Subsystem  | Lines | CC  | Risk Band | Orphan |
|-----------|----------|-----------|------|-----|-----------|--------|
| HAPIRNK   | DDS_LF   | HAP       | 13   | N/A | N/A       | no     |
| HAPTRFR   | DDS_PF   | HAP       | 72   | N/A | N/A       | no     |
| HMLMAST5H | DDS_LF   | UNGROUPED | 12   | N/A | N/A       | no     |
| HXLTABLD  | DDS_LF   | HXLT      | 11   | N/A | N/A       | no     |
| HXLTABLP  | DDS_LF   | HXLT      | 12   | N/A | N/A       | no     |
| HXLTABLS  | DDS_LF   | HXLT      | 12   | N/A | N/A       | no     |
| HXPBNFIT  | DDS_LF   | HXP       | 12   | N/A | N/A       | no     |
| HXPDICT   | DDS_PF   | HXP       | 6130 | N/A | N/A       | no     |
| HXPLVL1   | DDS_PF   | HXPL      | 49   | N/A | N/A       | no     |
| HXPLVL2   | DDS_PF   | HXPL      | 52   | N/A | N/A       | no     |
| HXPLVL3   | DDS_PF   | HXPL      | 52   | N/A | N/A       | no     |
| HXPLVL4   | DDS_PF   | HXPL      | 52   | N/A | N/A       | no     |
| HXPLVL5   | DDS_PF   | HXPL      | 55   | N/A | N/A       | no     |
| HXPLVL6   | DDS_PF   | HXPL      | 321  | N/A | N/A       | no     |
| HXPNSTN   | DDS_LF   | HXP       | 12   | N/A | N/A       | no     |
| HXPTABLD  | DDS_PF   | HXP       | 19   | N/A | N/A       | no     |
| HXPXMLD   | DDS_PF   | HXPX      | 19   | N/A | N/A       | no     |
| HXPXMLR   | DDS_PF   | HXPX      | 19   | N/A | N/A       | no     |
| OAPIRNK   | DDS_PF   | UNGROUPED | 80   | N/A | N/A       | no     |
| OMPMAST   | DDS_PF   | UNGROUPED | 310  | N/A | N/A       | no     |
| OXPBNFIT  | DDS_PF   | OXP       | 48   | N/A | N/A       | no     |
| OXPNSTN   | DDS_PF   | OXP       | 65   | N/A | N/A       | no     |
| HXXAPPPRF | SQLRPGLE | HXXA      | 123  | 1   | LOW       | yes    |
| XFXCNTR   | RPGLE    | XFXC      | 49   | 3   | LOW       | yes    |
| XFXCYMD   | RPGLE    | XFXC      | 83   | 7   | LOW       | yes    |
| XFXGETID  | RPGLE    | XFX       | 61   | 1   | LOW       | yes    |
| XFXLDSC   | RPGLE    | XFXL      | 135  | 5   | LOW       | yes    |
| XFXLEAP   | RPGLE    | XFXL      | 61   | 1   | LOW       | yes    |
| XFXMRNROL | RPGLE    | XFX       | 65   | 1   | LOW       | yes    |
| XFXTABL   | RPGLE    | XFX       | 164  | 9   | LOW       | yes    |
| CXXXMLP   | SQLRPGLE | UNGROUPED | 25   | 1   | LOW       | no     |
| HXXAPPPRFP| RPGLE    | HXXA      | 42   | 1   | LOW       | no     |
| HXXCNTRL  | RPGLE    | HXX       | 8    | 1   | LOW       | no     |
| HXXLDA    | RPGLE    | HXXL      | 53   | 1   | LOW       | no     |
| HXXLEVEL  | RPGLE    | HXXL      | 25   | 1   | LOW       | no     |
| HXXXML    | RPGLE    | HXX       | 11   | 1   | LOW       | no     |
| HABADTE   | RPGLE    | HA        | 821  | 152 | HIGH      | yes    |

### 1.2 Inventory Notes

- Total members: 37
- Total lines of code: 9,153
- DDS_LF: 7, DDS_PF: 15, RPGLE: 13, SQLRPGLE: 2
- HABADTE is the primary patient-management controller with the highest complexity.

## 2. Dependency Hotspots

| Program    | Score | Fan-In | Fan-Out | File Ops |
|-----------|-------|--------|---------|----------|
| HABADTE   | 38    | 0      | 13      | 6        |
| XFXLDSC   | 15    | 1      | 0       | 6        |
| XFXTABL   | 11    | 1      | 0       | 4        |
| XFXCNTR   | 9     | 3      | 0       | 0        |
| XFXMRNROL | 7     | 1      | 2       | 0        |
| HXXAPPPRF | 7     | 1      | 2       | 0        |
| XFXGETID  | 7     | 1      | 1       | 1        |
| XFXCYMD   | 5     | 1      | 1       | 0        |
| XFXLEAP   | 3     | 1      | 0       | 0        |
| HXHAPPPRF | 3     | 1      | 0       | 0        |
| HMLMAST5H | 2     | 0      | 0       | 1        |
| HXLTABLS  | 2     | 0      | 0       | 1        |
| HXPNSTN   | 2     | 0      | 0       | 1        |
| HXPBNFIT  | 2     | 0      | 0       | 1        |
| HAPIRNK   | 2     | 0      | 0       | 1        |
| HXLTABLD  | 2     | 0      | 0       | 1        |
| HXLTABLP  | 2     | 0      | 0       | 1        |

HABADTE sits at the centre of the call graph, orchestrating patient transfers, level description lookups, table-driven business logic, MRN/profile resolution, and XML output.

## 3. Call Graph Summary

| Caller      | Callee     | Relationship |
|------------|-----------|-------------|
| XFXCYMD    | XFXLEAP   | CALL        |
| XFXMRNROL  | HXHAPPPRF | CALL        |
| XFXMRNROL  | HXXAPPPRF | CALL        |
| HABADTE    | XFXMRNROL | CALL        |
| HABADTE    | XFXCNTR   | CALL        |
| HABADTE    | XFXLDSC   | CALL        |
| HABADTE    | XFXCNTR   | CALL        |
| HABADTE    | XFXCNTR   | CALL        |
| HABADTE    | XFXCYMD   | CALL        |
| HABADTE    | XFXGETID  | CALL        |
| HABADTE    | XFXTABL   | CALL        |
| HXXAPPPRF  | HXXCNTRL  | COPY        |
| HXXAPPPRF  | HXXAPPPRFP| COPY        |
| XFXGETID   | HXXLDA    | COPY        |
| HABADTE    | HXXLDA    | COPY        |
| HABADTE    | HXXLEVEL  | COPY        |
| HABADTE    | HXXXML    | COPY        |
| HABADTE    | CXXXMLP   | COPY        |
| HABADTE    | CXXXMLC   | COPY        |
| HAPIRNK    | TAPIRNK   | PFILE_OF    |
| HMLMAST5H  | TMPMAST   | PFILE_OF    |
| HXLTABLD   | HXPTABLD  | PFILE_OF    |
| HXLTABLP   | HXPTABLD  | PFILE_OF    |
| HXLTABLS   | HXPTABLD  | PFILE_OF    |
| HXPBNFIT   | TXPBNFIT  | PFILE_OF    |
| HXPNSTN    | TXPNSTN   | PFILE_OF    |
| XFXGETID   | HXFXMLR   | READ        |
| XFXLDSC    | HXFLVL1   | READ        |
| XFXLDSC    | HXFLVL2   | READ        |
| XFXLDSC    | HXFLVL3   | READ        |
| XFXLDSC    | HXFLVL4   | READ        |
| XFXLDSC    | HXFLVL5   | READ        |
| XFXLDSC    | HXFLVL6   | READ        |
| XFXTABL    | XFFTABLD  | READ        |
| XFXTABL    | XFFTABL2  | READ        |
| XFXTABL    | XFFTABL3  | READ        |
| XFXTABL    | XFFTABL4  | READ        |
| HABADTE    | HAPTRFR   | READ        |
| HABADTE    | XFFNSTN   | READ        |
| HABADTE    | HXFXMLH   | READ        |
| HABADTE    | HXFXMLH   | UPDATE      |
| HABADTE    | HXFXMLH   | WRITE       |
| HABADTE    | HXFXMLD   | WRITE       |

This summary shows HABADTE as the orchestrating caller, integrating patient transfer file access (HAPTRFR), station/level information (XFFNSTN, HXFLVL*), table-driven logic (XFFTABL*), and XML file handling (HXFXMLH/D).

## 4. Business Rules by Program

### HABADTE (PATIENT_MANAGEMENT)

| Rule ID | Rule Text                                                        |
|---------|------------------------------------------------------------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'              |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'       |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' |

These rules implement core inclusion/exclusion logic for patient transfer records, ensuring that only non-voided, inpatient records with active file indicators are processed.

### XFXCNTR (DATA_MAINTENANCE)

| Rule ID | Rule Text                                       |
|---------|-------------------------------------------------|
| BR-001  | When X equals zero, branch to 'EXIT'           |
| BR-002  | When X equals 40, branch to 'EXIT'            |

XFXCNTR implements counter-based control flow, signalling exits when control values reach boundary conditions.

### XFXCYMD (DATA_MAINTENANCE)

| Rule ID | Rule Text                                                          |
|---------|--------------------------------------------------------------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'                      |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'                   |
| BR-005  | When VMM is less than 01, branch to 'EXIT'                        |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'                     |
| BR-007  | When VDD is less than 01, branch to 'EXIT'                        |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'               |

XFXCYMD validates composite date components (year, month, day) before proceeding, enforcing an 1800–2100 year range and valid month/day combinations.

### XFXLDSC (DATA_MAINTENANCE)

| Rule ID | Rule Text                                       |
|---------|-------------------------------------------------|
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'   |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'   |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'   |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT' |

XFXLDSC enforces bounds on mapping values used in level description logic, preventing invalid map codes from being processed.

### XFXTABL (DATA_MAINTENANCE)

| Rule ID | Rule Text                                            |
|---------|------------------------------------------------------|
| BR-013  | When *IN79 equals on/active, branch to 'EXIT'       |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT'       |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT'       |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT'       |

XFXTABL uses indicator *IN79 to control exit behaviour when table-driven conditions are met.

### HXXAPPPRF (DATA_MAINTENANCE)

| Rule ID | Rule Text                                   |
|---------|---------------------------------------------|
| BR-020  | SQL program accesses table 'HXPAPPPRF'     |

HXXAPPPRF encapsulates SQL-based profile access to HXPAPPPRF (and related tables per interpretations), providing data-maintenance rules around application profiles.

## 5. Missing Dependencies

| Name      | Type     | Impact | Referenced By |
|----------|----------|--------|--------------|
| CXXXMLC  | COPYBOOK | HIGH   | HABADTE      |
| HXHAPPPRF| PROGRAM  | MEDIUM | XFXMRNROL    |
| TAPIRNK  | FILE     | HIGH   | HAPIRNK      |
| TMPMAST  | FILE     | HIGH   | HMLMAST5H    |
| TXPBNFIT | FILE     | HIGH   | HXPBNFIT     |
| TXPNSTN  | FILE     | HIGH   | HXPNSTN      |
| ****HXPXML| FILE    | MEDIUM | HABADTE      |
| PRINTER  | FILE     | MEDIUM | HABADTE      |

These missing components represent unresolved PFs and copybooks that are essential for fully understanding PF/LF relationships, XML layouts, and profile or printing behaviour.

## 6. Tech Debt Summary

- Total findings: 4
- Estimated remediation effort: 26.9 hours
- Severity distribution:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

The primary concentration of technical debt is around HABADTE’s high complexity and its reliance on missing external components (PFs TAPIRNK/TMPMAST/TXPBNFIT/TXPNSTN and COPYBOOK CXXXMLC), which must be addressed during modernisation.
