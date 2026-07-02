# Module Specifications – HABADTE AS400 Portfolio

## 1. Component Inventory

The table below lists all source members discovered in the HABADTE portfolio, combining structural metadata with complexity and orphan status.

| Program | Type   | Subsystem | Lines | CC | Risk Band | Orphan |
|--------|--------|-----------|-------|----|-----------|--------|
| HAPIRNK | DDS_LF | HAP | 13 | N/A | N/A | no |
| HAPTRFR | DDS_PF | HAP | 72 | N/A | N/A | no |
| HMLMAST5H | DDS_LF | UNGROUPED | 12 | N/A | N/A | no |
| HXLTABLD | DDS_LF | HXLT | 11 | N/A | N/A | no |
| HXLTABLP | DDS_LF | HXLT | 12 | N/A | N/A | no |
| HXLTABLS | DDS_LF | HXLT | 12 | N/A | N/A | no |
| HXPBNFIT | DDS_LF | HXP | 12 | N/A | N/A | no |
| HXPDICT | DDS_PF | HXP | 6130 | N/A | N/A | no |
| HXPLVL1 | DDS_PF | HXPL | 49 | N/A | N/A | no |
| HXPLVL2 | DDS_PF | HXPL | 52 | N/A | N/A | no |
| HXPLVL3 | DDS_PF | HXPL | 52 | N/A | N/A | no |
| HXPLVL4 | DDS_PF | HXPL | 52 | N/A | N/A | no |
| HXPLVL5 | DDS_PF | HXPL | 55 | N/A | N/A | no |
| HXPLVL6 | DDS_PF | HXPL | 321 | N/A | N/A | no |
| HXPNSTN | DDS_LF | HXP | 12 | N/A | N/A | no |
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
- "Orphan" is based on the orphan_programs list; members marked "yes" have no inbound CALL edges in the extracted graph, even though some (e.g., HABADTE) are top-level drivers.
- Complexity values (CC and band) are only available for programs; DDS members have "N/A".

## 2. Dependency Hotspots

This section highlights programs with the highest combined structural score (fan-in, fan-out, file operations), which drive most of the runtime interactions.

| Program | Score | Fan-In | Fan-Out | File Ops |
|--------|-------|--------|---------|----------|
| HABADTE | 38 | 0 | 13 | 6 |
| XFXLDSC | 15 | 1 | 0 | 6 |
| XFXTABL | 11 | 1 | 0 | 4 |
| XFXCNTR | 9 | 3 | 0 | 0 |
| XFXMRNROL | 7 | 1 | 2 | 0 |
| HXXAPPPRF | 7 | 1 | 2 | 0 |
| XFXGETID | 7 | 1 | 1 | 1 |
| XFXCYMD | 5 | 1 | 1 | 0 |
| XFXLEAP | 3 | 1 | 0 | 0 |
| HXHAPPPRF | 3 | 1 | 0 | 0 |
| HAPIRNK | 2 | 0 | 0 | 1 |
| HXLTABLD | 2 | 0 | 0 | 1 |
| HXPNSTN | 2 | 0 | 0 | 1 |
| HXLTABLP | 2 | 0 | 0 | 1 |
| HMLMAST5H | 2 | 0 | 0 | 1 |
| HXLTABLS | 2 | 0 | 0 | 1 |
| HXPBNFIT | 2 | 0 | 0 | 1 |

Interpretation:
- HABADTE is the primary orchestration module and the most complex component.
- XFXLDSC and XFXTABL are central lookup services over the level tables and dictionary tables.
- XFXCNTR, XFXMRNROL, HXXAPPPRF, XFXGETID, and XFXCYMD form a shared service layer handling counters, MRN rollover, application profiles, ID assignment, and date validation.

## 3. Call Graph Summary

The table below summarizes key dependency edges, showing callers, callees, and edge type (CALL, COPY, PFILE_OF, READ, WRITE, UPDATE).

| Caller (f) | Callee/Target (t) | Type |
|-----------|--------------------|------|
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

This graph shows HABADTE at the center, calling utilities, copying configuration, and interacting with patient transfer and XML files. Logical-file relationships (PFILE_OF) indicate DDS-based indexing structures.

## 4. Business Rules by Program

Approved business rules are grouped by their source program.

### 4.1 XFXCNTR

RPGLE program in the Data Maintenance domain responsible for counter-based branching.

- **BR-001** – "When X equals zero, branch to 'EXIT'"  
  This rule terminates processing when the main counter X is zero, typically meaning there are no items to process or a loop has completed.

- **BR-002** – "When X equals 40, branch to 'EXIT'"  
  This rule uses a ceiling of 40 iterations, enforcing an upper limit on the number of processed records or attempts.

### 4.2 XFXCYMD

RPGLE program in Data Maintenance that validates calendar dates.

- **BR-003** – "When VYY is less than 1800, branch to 'EXIT'"  
- **BR-004** – "When VYY is greater than 2100, branch to 'EXIT'"  
- **BR-005** – "When VMM is less than 01, branch to 'EXIT'"  
- **BR-006** – "When VMM is greater than 12, branch to 'EXIT'"  
- **BR-007** – "When VDD is less than 01, branch to 'EXIT'"  
- **BR-008** – "When VDD is greater than DYS(VMM), branch to 'EXIT'"  

Collectively, these rules ensure that year, month, and day values fall within reasonable ranges and respect the number of days in each month.

### 4.3 XFXLDSC

RPGLE program in Data Maintenance that interprets level mappings.

- **BR-009** – "When LDAMAP is greater than 99, branch to 'EXIT'"  
- **BR-010** – "When LDAMAP is greater than 99, branch to 'EXIT'"  
- **BR-011** – "When LDAMAP is greater than 99, branch to 'EXIT'"  
- **BR-012** – "When LDAMAP is greater than 9999, branch to 'EXIT'"  

These rules constrain LDAMAP mapping codes to acceptable ranges, preventing invalid level mappings from being used.

### 4.4 XFXTABL

RPGLE program in Data Maintenance that handles table-based control using indicator *IN79.

- **BR-013** – "When *IN79 equals on/active, branch to 'EXIT'"  
- **BR-014** – "When *IN79 equals on/active, branch to 'EXIT'"  
- **BR-015** – "When *IN79 equals on/active, branch to 'EXIT'"  
- **BR-016** – "When *IN79 equals on/active, branch to 'EXIT'"  

These rules standardize the use of indicator 79 as a control flag for terminating lookups or signaling completion.

### 4.5 HABADTE

RPGLE program in the Patient Management domain serving as the main driver.

- **BR-017** – "When -FILE INDICATOR equals zero, branch to 'SKIP'"  
- **BR-018** – "When -FLAG INDICATOR equals void/voided, branch to 'SKIP'"  
- **BR-019** – "When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'"  

These rules govern which records are eligible for processing: only non-voided inpatient records with valid file indicators proceed through the HABADTE workflow.

## 5. Missing Dependencies

Gaps represent referenced-but-missing artifacts that must be resolved during modernization.

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

Modernization teams must obtain or reconstruct these components from the AS400 environment, particularly CXXXMLC and the missing base PFs (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN), before attempting automated code conversion.

## 6. Tech Debt Summary

The aggregated tech debt metrics for the HABADTE portfolio are:

- **Total findings:** 0  
- **Total remediation hours:** 0.0  
- **By severity:**  
  - HIGH: 0  
  - MEDIUM: 0  
  - LOW: 0  

Although no formal tech debt items are recorded, the structural analysis highlights:
- A highly complex main program (HABADTE) that would benefit from modularization.
- Multiple missing dependencies that represent architectural risk rather than traditional tech debt.
- Heavy reliance on DDS logical files for indexing, which should be mapped to relational indexes or views in the target platform.
