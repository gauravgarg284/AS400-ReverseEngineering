# Module Specifications – HABADTE Project

## 1. Component Inventory

The HABADTE project aggregated context does not include a populated `source_members` list; only program interpretations and dependency metadata are available. Based on `program_interpretations`, `dep_edges`, and `dep_hotspots`, the effective component inventory is the set of interpreted programs and key DDS objects.

### 1.1 Program Inventory

| Program        | Type      | Subsystem      | Lines | CC  | Risk Band | Orphan |
|----------------|-----------|----------------|-------|-----|-----------|--------|
| HABADTE        | RPGLE     | PATIENT_MGMT   | N/A   | 152 | HIGH      | yes    |
| XFXCNTR        | RPGLE     | DATA_MAINT     | N/A   | 3   | LOW       | yes    |
| XFXCYMD        | RPGLE     | DATA_MAINT     | N/A   | 7   | LOW       | yes    |
| XFXLDSC        | RPGLE     | DATA_MAINT     | N/A   | 5   | LOW       | yes    |
| XFXTABL        | RPGLE     | DATA_MAINT     | N/A   | 9   | LOW       | yes    |
| XFXGETID       | RPGLE     | DATA_MAINT     | N/A   | 1   | LOW       | yes    |
| XFXLEAP        | RPGLE     | DATA_MAINT     | N/A   | 1   | LOW       | yes    |
| XFXMRNROL      | RPGLE     | DATA_MAINT     | N/A   | 1   | LOW       | yes    |
| HXXAPPPRF      | SQLRPGLE  | DATA_MAINT     | N/A   | 1   | LOW       | yes    |

**Notes**
- Cyclomatic complexity (CC) and risk band are taken from `complexity_per_program`.
- All programs listed appear in the `orphan_programs` array, meaning no caller was detected in the static manifest. However, the dependency edges show HABADTE calling most of the XFX* utilities and HXXAPPPRF, so these are functional participants despite the orphan flag.

### 1.2 Key DDS Objects

From `data_dict_schema.physical_files` and `.logical_files`:

- Physical Files (PF): HAPTRFR, HXPDICT, HXPLVL1–HXPLVL6, HXPTABLD, HXPXMLD, HXPXMLR, OAPIRNK, OMPMAST, OXPBNFIT, OXPNSTN, TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN.
- Logical Files (LF): HAPIRNK, HMLMAST5H, HXLTABLD, HXLTABLP, HXLTABLS, HXPBNFIT, HXPNSTN.

These files are consumed by the programs above via READ/WRITE operations recorded in `dep_edges`.

## 2. Dependency Hotspots

Top-scoring modules from `dep_hotspots`:

| Program     | Score | Fan-in | Fan-out | File Ops | Complexity Band |
|------------|-------|--------|---------|----------|-----------------|
| HABADTE    | 38    | 0      | 13      | 6        | HIGH            |
| XFXLDSC    | 15    | 1      | 0       | 6        | LOW             |
| XFXTABL    | 11    | 1      | 0       | 4        | LOW             |
| XFXCNTR    | 9     | 3      | 0       | 0        | LOW             |
| XFXMRNROL  | 7     | 1      | 2       | 0        | LOW             |
| HXXAPPPRF  | 7     | 1      | 2       | 0        | LOW             |
| XFXGETID   | 7     | 1      | 1       | 1        | LOW             |
| XFXCYMD    | 5     | 1      | 1       | 0        | LOW             |
| HXHAPPPRF  | 3     | 1      | 0       | 0        | LOW             |
| XFXLEAP    | 3     | 1      | 0       | 0        | LOW             |
| HAPIRNK    | 2     | 0      | 0       | 1        | N/A             |
| HXPNSTN    | 2     | 0      | 0       | 1        | N/A             |
| HXLTABLD   | 2     | 0      | 0       | 1        | N/A             |
| HMLMAST5H  | 2     | 0      | 0       | 1        | N/A             |
| HXLTABLS   | 2     | 0      | 0       | 1        | N/A             |
| HXPBNFIT   | 2     | 0      | 0       | 1        | N/A             |
| HXLTABLP   | 2     | 0      | 0       | 1        | N/A             |

HABADTE is the primary hotspot due to heavy fan-out and multiple file operations. XFXLDSC and XFXTABL are data-access services for level hierarchy and generic code tables.

## 3. Call Graph Summary

Derived from `dep_edges`:

### 3.1 Program-to-Program Dependencies

| Caller      | Callee      | Type  |
|-------------|-------------|-------|
| XFXCYMD     | XFXLEAP     | CALL  |
| XFXMRNROL   | HXHAPPPRF   | CALL  |
| XFXMRNROL   | HXXAPPPRF   | CALL  |
| HABADTE     | XFXMRNROL   | CALL  |
| HABADTE     | XFXCNTR     | CALL  |
| HABADTE     | XFXLDSC     | CALL  |
| HABADTE     | XFXCNTR     | CALL  |
| HABADTE     | XFXCNTR     | CALL  |
| HABADTE     | XFXCYMD     | CALL  |
| HABADTE     | XFXGETID    | CALL  |
| HABADTE     | XFXTABL     | CALL  |
| HXXAPPPRF   | HXXCNTRL    | COPY  |
| HXXAPPPRF   | HXXAPPPRFP  | COPY  |
| XFXGETID    | HXXLDA      | COPY  |
| HABADTE     | HXXLDA      | COPY  |
| HABADTE     | HXXLEVEL    | COPY  |
| HABADTE     | HXXXML      | COPY  |
| HABADTE     | CXXXMLP     | COPY  |
| HABADTE     | CXXXMLC     | COPY  |

### 3.2 File Dependencies

| Program   | File/Object | Operation |
|-----------|-------------|-----------|
| HAPIRNK   | TAPIRNK     | PFILE_OF  |
| HMLMAST5H | TMPMAST     | PFILE_OF  |
| HXLTABLD  | HXPTABLD    | PFILE_OF  |
| HXLTABLP  | HXPTABLD    | PFILE_OF  |
| HXLTABLS  | HXPTABLD    | PFILE_OF  |
| HXPBNFIT  | TXPBNFIT    | PFILE_OF  |
| HXPNSTN   | TXPNSTN     | PFILE_OF  |
| XFXGETID  | HXFXMLR     | READ      |
| XFXLDSC   | HXFLVL1     | READ      |
| XFXLDSC   | HXFLVL2     | READ      |
| XFXLDSC   | HXFLVL3     | READ      |
| XFXLDSC   | HXFLVL4     | READ      |
| XFXLDSC   | HXFLVL5     | READ      |
| XFXLDSC   | HXFLVL6     | READ      |
| XFXTABL   | XFFTABLD    | READ      |
| XFXTABL   | XFFTABL2    | READ      |
| XFXTABL   | XFFTABL3    | READ      |
| XFXTABL   | XFFTABL4    | READ      |
| HABADTE   | HAPTRFR     | READ      |
| HABADTE   | XFFNSTN     | READ      |
| HABADTE   | HXFXMLH     | READ      |
| HABADTE   | HXFXMLH     | UPDATE    |
| HABADTE   | HXFXMLH     | WRITE     |
| HABADTE   | HXFXMLD     | WRITE     |

## 4. Business Rules by Program

Business rules are sourced from `approved_rules` and grouped by `source_program`.

### 4.1 HABADTE (PATIENT_MANAGEMENT)

| BR-ID  | Rule                                                                                                                                    |
|--------|-----------------------------------------------------------------------------------------------------------------------------------------|
| BR-017 | Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census. |
| BR-018 | Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census.       |
| BR-019 | Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.               |
| BR-020 | Organisational Level Filter: Each patient record must belong to the requested organisation. The report accepts an org level (1–6) and org code from the caller. If the patient's org code at that level does not match, the record is excluded. |
| BR-021 | Census Date — Admission Date Check: A patient admitted after the census date is not yet active on that date. If the patient's admission date is later than the requested census date, the patient is excluded from the report. |
| BR-022 | Census Date — Discharge Date Check: A patient who has already been discharged on or before the census date is no longer active. If discharge date is set (non-zero) and is on or before the census date, the patient is excluded. A discharge date of zero means the patient has not been discharged and is always considered active (subject to other filters). |
| BR-023 | Room Number Retrieval from Transfer History: The patient's current room number is not stored directly on the patient record. Instead, it is taken from the most recent entry in the ATD Transfer History file within the relevant date range. |
| BR-024 | Room Class Description Lookup: The 2-character room class code (AFNRMC) retrieved from the transfer history is resolved to a human-readable description using the system reference table (HXPDICT/HXPTABLD). |
| BR-025 | Hospital / Therapeutic Leave Flag: When the room class description is looked up, the HCS mapping field (XFDMAP) is also examined. If the 10th character of XFDMAP is 'H', the patient is on Hospital Leave; if it is 'T', the patient is on Therapeutic Leave. |

### 4.2 XFXCNTR (DATA_MAINTENANCE)

| BR-ID  | Rule                                                                                                                                   |
|--------|-----------------------------------------------------------------------------------------------------------------------------------------|
| BR-001 | Text Centering — Blank String Optimization (UI & Reporting Workflows): Before attempting to center a string for screen display or report formatting, the system checks if the input is completely blank. If the string contains no non-space characters, the centering logic is skipped entirely. |
| BR-002 | Text Centering — Full String Optimization (UI & Reporting Workflows): When formatting text for UI screens or reports, the system checks if the string is already occupying the maximum field width. If the first character is non-blank, the text is considered already centered, and the routine exits. |

### 4.3 XFXCYMD (DATA_MAINTENANCE)

| BR-ID  | Rule                                                                                                              |
|--------|-------------------------------------------------------------------------------------------------------------------|
| BR-003 | Date validation: reject year < 1800 (historical minimum).                                                        |
| BR-004 | Date validation: reject year > 2100 (forecast maximum).                                                          |
| BR-005 | Date validation: reject month < 01 (calendar constraint).                                                        |
| BR-006 | Date validation: reject month > 12 (calendar constraint).                                                        |
| BR-007 | Date validation: reject day < 01 (calendar constraint).                                                          |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT' (day component exceeds maximum days in month).               |

### 4.4 XFXLDSC (DATA_MAINTENANCE)

| BR-ID  | Rule                                                          |
|--------|----------------------------------------------------------------|
| BR-009 | Level lookup: reject if level code exceeds valid range.       |
| BR-012 | Level lookup: reject if level code exceeds valid range.       |

### 4.5 XFXTABL (DATA_MAINTENANCE)

| BR-ID  | Rule                                                          |
|--------|----------------------------------------------------------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'.                |

Other XFXTABL rules (BR-014–BR-016) are noted in interpretations_detail but not included in the approved_rules subset.

## 5. Missing Dependencies

The `high_medium_gaps` list in aggregated_context is empty for this run, indicating no unresolved gap records at the summary layer. All currently known dependencies are resolved via the dep_edges and data_dict_schema.

## 6. Tech Debt Summary

The `tech_debt_summary` object is empty in the aggregated context, indicating no consolidated tech debt metrics were generated by Agent-2/Agent-3 for this run. However, complexity and hotspot data imply:

- HABADTE (cc=152, HIGH band) is a high-maintenance module with many branches.
- Utility modules (XFX*) have LOW complexity but are widely reused, making them important to stabilize.

For modernization planning, HABADTE should be prioritized for refactoring and thorough testing, while XFX* utilities should be extracted into shared services with clear contracts.
