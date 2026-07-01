# Module Specifications

## 1. Component Inventory

_No source members were provided in the aggregated context. This section reflects an empty source member inventory for run 202607011209._

| Program | Type | Subsystem | Lines | CC | Risk Band | Orphan |
|--------|------|-----------|-------|----|-----------|--------|

## 2. Dependency Hotspots

The following programs have been identified as dependency hotspots based on fan-in/fan-out and file operation counts.

| Program | Score | Fan In | Fan Out | File Ops |
|---------|-------|--------|---------|----------|
| HABADTE | 38 | 0 | 13 | 6 |
| XFXLDSC | 15 | 1 | 0 | 6 |
| XFXTABL | 11 | 1 | 0 | 4 |
| XFXCNTR | 9 | 3 | 0 | 0 |
| XFXMRNROL | 7 | 1 | 2 | 0 |
| XFXGETID | 7 | 1 | 1 | 1 |
| HXXAPPPRF | 7 | 1 | 2 | 0 |
| XFXCYMD | 5 | 1 | 1 | 0 |
| HXHAPPPRF | 3 | 1 | 0 | 0 |
| XFXLEAP | 3 | 1 | 0 | 0 |
| HXLTABLS | 2 | 0 | 0 | 1 |
| HMLMAST5H | 2 | 0 | 0 | 1 |
| HXPNSTN | 2 | 0 | 0 | 1 |
| HAPIRNK | 2 | 0 | 0 | 1 |
| HXLTABLD | 2 | 0 | 0 | 1 |
| HXPBNFIT | 2 | 0 | 0 | 1 |
| HXLTABLP | 2 | 0 | 0 | 1 |

## 3. Call Graph Summary

This section summarizes the call and file dependency graph as extracted from the AS400 codebase.

| From (Caller) | To (Callee/Target) | Dependency Type |
|---------------|--------------------|-----------------|
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

## 4. Business Rules by Program

Business rules are grouped by source program, using the approved rules extracted from the legacy code.

### Program: XFXCNTR

- **BR-001**: When X equals zero, branch to 'EXIT'.
- **BR-002**: When X equals 40, branch to 'EXIT'.

### Program: XFXCYMD

- **BR-003**: When VYY is less than 1800, branch to 'EXIT'.
- **BR-004**: When VYY is greater than 2100, branch to 'EXIT'.
- **BR-005**: When VMM is less than 01, branch to 'EXIT'.
- **BR-006**: When VMM is greater than 12, branch to 'EXIT'.
- **BR-007**: When VDD is less than 01, branch to 'EXIT'.
- **BR-008**: When VDD is greater than DYS(VMM), branch to 'EXIT'.

### Program: XFXLDSC

- **BR-009**: When LDAMAP is greater than 99, branch to 'EXIT'.
- **BR-010**: When LDAMAP is greater than 99, branch to 'EXIT'.
- **BR-011**: When LDAMAP is greater than 99, branch to 'EXIT'.
- **BR-012**: When LDAMAP is greater than 9999, branch to 'EXIT'.

### Program: XFXTABL

- **BR-013**: When *IN79 equals on/active, branch to 'EXIT'.
- **BR-014**: When *IN79 equals on/active, branch to 'EXIT'.
- **BR-015**: When *IN79 equals on/active, branch to 'EXIT'.
- **BR-016**: When *IN79 equals on/active, branch to 'EXIT'.

### Program: HABADTE

- **BR-017**: When -FILE INDICATOR equals zero, branch to 'SKIP'.
- **BR-018**: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- **BR-019**: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

## 5. Missing Dependencies

_No high or medium gaps were identified in the aggregated context for this run._

| Name | Type | Impact | Referenced By |
|------|------|--------|---------------|

## 6. Tech Debt Summary

_No tech debt summary details were available in the aggregated context. All tech debt metrics are effectively zero or not recorded for this run._
