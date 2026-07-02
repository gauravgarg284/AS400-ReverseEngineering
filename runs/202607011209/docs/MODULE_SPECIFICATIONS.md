# Module Specifications – HABADTE Project

## 1. Component Inventory

The table below lists all harvested source members with structural and risk metadata derived from the aggregated context.

| Program | Type | Subsystem | Lines | CC | Risk Band | Orphan |
|--------|------|-----------|-------|----|-----------|--------|
| HAPIRNK | DDS_LF | HAP | 13 | N/A | N/A | yes |
| HAPTRFR | DDS_PF | HAP | 72 | N/A | N/A | no |
| HMLMAST5H | DDS_LF | UNGROUPED | 12 | N/A | N/A | yes |
| HXLTABLD | DDS_LF | HXLT | 11 | N/A | N/A | yes |
| HXLTABLP | DDS_LF | HXLT | 12 | N/A | N/A | yes |
| HXLTABLS | DDS_LF | HXLT | 12 | N/A | N/A | yes |
| HXPBNFIT | DDS_LF | HXP | 12 | N/A | N/A | yes |
| HXPDICT | DDS_PF | HXP | 6130 | N/A | N/A | no |
| HXPLVL1 | DDS_PF | HXPL | 49 | N/A | N/A | no |
| HXPLVL2 | DDS_PF | HXPL | 52 | N/A | N/A | no |
| HXPLVL3 | DDS_PF | HXPL | 52 | N/A | N/A | no |
| HXPLVL4 | DDS_PF | HXPL | 52 | N/A | N/A | no |
| HXPLVL5 | DDS_PF | HXPL | 55 | N/A | N/A | no |
| HXPLVL6 | DDS_PF | HXPL | 321 | N/A | N/A | no |
| HXPNSTN | DDS_LF | HXP | 12 | N/A | N/A | yes |
| HXPTABLD | DDS_PF | HXP | 19 | N/A | N/A | no |
| HXPXMLD | DDS_PF | HXPX | 19 | N/A | N/A | no |
| HXPXMLR | DDS_PF | HXPX | 19 | N/A | N/A | no |
| OAPIRNK | DDS_PF | UNGROUPED | 80 | N/A | N/A | no |
| OMPMAST | DDS_PF | UNGROUPED | 310 | N/A | N/A | no |
| OXPBNFIT | DDS_PF | OXP | 48 | N/A | N/A | no |
| OXPNSTN | DDS_PF | OXP | 65 | N/A | N/A | no |
| HXXAPPPRF | SQLRPGLE | HXXA | 123 | 1 | LOW | yes |
| XFXCNTR | RPGLE | XFXC | 49 | 3 | LOW | yes |
| XFXCYMD | RPGLE | XFXC | 83 | 7 | LOW | yes |
| XFXGETID | RPGLE | XFX | 61 | 1 | LOW | yes |
| XFXLDSC | RPGLE | XFXL | 135 | 5 | LOW | yes |
| XFXLEAP | RPGLE | XFXL | 61 | 1 | LOW | yes |
| XFXMRNROL | RPGLE | XFX | 65 | 1 | LOW | yes |
| XFXTABL | RPGLE | XFX | 164 | 9 | LOW | yes |
| CXXXMLP | SQLRPGLE | UNGROUPED | 25 | 1 | LOW | no |
| HXXAPPPRFP | RPGLE | HXXA | 42 | 1 | LOW | no |
| HXXCNTRL | RPGLE | HXX | 8 | 1 | LOW | no |
| HXXLDA | RPGLE | HXXL | 53 | 1 | LOW | no |
| HXXLEVEL | RPGLE | HXXL | 25 | 1 | LOW | no |
| HXXXML | RPGLE | HXX | 11 | 1 | LOW | no |
| HABADTE | RPGLE | HA | 821 | 152 | HIGH | yes |

Notes:
- "Orphan" is marked "yes" where the program appears in the orphan_programs list.
- Cyclomatic complexity (CC) and risk band are included only for executable programs.

## 2. Dependency Hotspots

Programs with the highest structural complexity and dependency fan-out.

| Program | Score | Fan-In | Fan-Out | File Ops |
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

HABADTE is the dominant hotspot and should be treated as the primary batch/integration module during modernization.

## 3. Call Graph Summary

The following table summarizes direct dependencies between members.

| Caller (f) | Callee/File (t) | Dependency Type |
|------------|-----------------|-----------------|
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

These edges drive the sequencing of batch processing, table lookups, and XML creation.

## 4. Business Rules by Program

Business rules extracted from approved_rules, grouped by source program.

### Program: XFXCNTR

- BR-001 – When X equals zero, branch to 'EXIT'.
- BR-002 – When X equals 40, branch to 'EXIT'.

These rules define boundary conditions for a numeric counter; values outside the range (1–39) terminate processing.

### Program: XFXCYMD

- BR-003 – When VYY is less than 1800, branch to 'EXIT'.
- BR-004 – When VYY is greater than 2100, branch to 'EXIT'.
- BR-005 – When VMM is less than 01, branch to 'EXIT'.
- BR-006 – When VMM is greater than 12, branch to 'EXIT'.
- BR-007 – When VDD is less than 01, branch to 'EXIT'.
- BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT'.

XFXCYMD enforces calendar validity on year, month, and day components before downstream batch logic executes.

### Program: XFXLDSC

- BR-009 – When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-010 – When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-011 – When LDAMAP is greater than 99, branch to 'EXIT'.
- BR-012 – When LDAMAP is greater than 9999, branch to 'EXIT'.

These rules prevent processing of descriptor maps outside allowed numeric ranges, ensuring table lookups remain within configured limits.

### Program: XFXTABL

- BR-013 – When *IN79 equals on/active, branch to 'EXIT'.
- BR-014 – When *IN79 equals on/active, branch to 'EXIT'.
- BR-015 – When *IN79 equals on/active, branch to 'EXIT'.
- BR-016 – When *IN79 equals on/active, branch to 'EXIT'.

XFXTABL uses indicator *IN79 to guard execution segments. When the indicator is on, certain branches terminate early, effectively disabling specific table lookup paths.

### Program: HABADTE

- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

HABADTE skips records that are file-suppressed, voided, or flagged as outpatient, ensuring that only active inpatient transfer records are included in the batch run.

## 5. Missing Dependencies

Missing components identified as high or medium impact gaps.

| Name | Type | Impact | Referenced By |
|------|------|--------|---------------|
| CXXXMLC | COPYBOOK | HIGH | HABADTE |
| HXHAPPPRF | PROGRAM | MEDIUM | XFXMRNROL |
| TAPIRNK | FILE | HIGH | HAPIRNK |
| TMPMAST | FILE | HIGH | HMLMAST5H |
| TXPBNFIT | FILE | HIGH | HXPBNFIT |
| TXPNSTN | FILE | HIGH | HXPNSTN |
| ****HXPXML | FILE | MEDIUM | HABADTE |
| PRINTER | FILE | MEDIUM | HABADTE |

These gaps must be addressed before a fully faithful behavioral reconstruction can be achieved. In migration, they will drive interface and feed design.

## 6. Tech Debt Summary

From the aggregated tech_debt_summary:

- Total findings: 0
- Total remediation hours: 0.0
- Findings by severity:
  - HIGH: 0
  - MEDIUM: 0
  - LOW: 0

Although no automated findings were raised, structural complexity in HABADTE and missing core files/copybooks represent implicit technical debt and modernization risk.
