# MODULE SPECIFICATIONS – HABADTE System

Run ID: 202607161154  
Project: HABADTE

---

## Section 1 – Component Inventory

The table below lists all source members discovered in the HABADTE application, enriched with cyclomatic complexity, risk band and orphan status.

| Program | Type | Subsystem | Lines | CC | Risk Band | Orphan |
|--------|------|-----------|-------|----|-----------|--------|
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
| TAPIRNK | DDS_PF | UNGROUPED | 167 | N/A | N/A | no |
| TMPMAST | DDS_PF | UNGROUPED | 688 | N/A | N/A | no |
| TXPBNFIT | DDS_PF | TXP | 164 | N/A | N/A | no |
| TXPNSTN | DDS_PF | TXP | 116 | N/A | N/A | no |
| HXXAPPPRF | SQLRPGLE | HXXA | 123 | 1 | LOW | no |
| XFXCNTR | RPGLE | XFXC | 49 | 3 | LOW | no |
| XFXCYMD | RPGLE | XFXC | 83 | 7 | LOW | no |
| XFXGETID | RPGLE | XFX | 61 | 1 | LOW | no |
| XFXLDSC | RPGLE | XFXL | 135 | 5 | LOW | no |
| XFXLEAP | RPGLE | XFXL | 61 | 1 | LOW | no |
| XFXMRNROL | RPGLE | XFX | 65 | 1 | LOW | no |
| XFXTABL | RPGLE | XFX | 164 | 9 | LOW | no |
| CXXXMLP | SQLRPGLE | UNGROUPED | 25 | 1 | LOW | no |
| HXXAPPPRFP | RPGLE | HXXA | 42 | 1 | LOW | no |
| HXXCNTRL | RPGLE | HXX | 8 | 1 | LOW | no |
| HXXLDA | RPGLE | HXXL | 53 | 1 | LOW | no |
| HXXLEVEL | RPGLE | HXXL | 25 | 1 | LOW | no |
| HXXXML | RPGLE | HXX | 11 | 1 | LOW | no |
| HABADTE | RPGLE | HA | 821 | 152 | HIGH | no |

Notes:
- Complexity values and bands come from the aggregated complexity_per_program data; DDS files do not carry CC metrics, so they are shown as "N/A".
- Although the aggregator lists several programs as "orphan_programs", dependency edges show that every program is invoked or referenced, so the Orphan column is "no" for all members.

---

## Section 2 – Dependency Hotspots

Programs with the highest dependency scores (fan-in/fan-out and file operations) indicate architectural hotspots.

| Program | Score | Fan-in | Fan-out | File Ops |
|---------|-------|--------|---------|----------|
| HABADTE | 38 | 0 | 13 | 6 |
| XFXLDSC | 15 | 1 | 0 | 6 |
| XFXTABL | 11 | 1 | 0 | 4 |
| XFXCNTR | 9 | 3 | 0 | 0 |
| XFXGETID | 7 | 1 | 1 | 1 |
| HXXAPPPRF | 7 | 1 | 2 | 0 |
| XFXMRNROL | 7 | 1 | 2 | 0 |
| XFXCYMD | 5 | 1 | 1 | 0 |
| HXHAPPPRF | 3 | 1 | 0 | 0 |
| XFXLEAP | 3 | 1 | 0 | 0 |
| HMLMAST5H | 2 | 0 | 0 | 1 |
| HXLTABLP | 2 | 0 | 0 | 1 |
| HXLTABLD | 2 | 0 | 0 | 1 |
| HXLTABLS | 2 | 0 | 0 | 1 |
| HAPIRNK | 2 | 0 | 0 | 1 |
| HXPBNFIT | 2 | 0 | 0 | 1 |
| HXPNSTN | 2 | 0 | 0 | 1 |

Observations:
- HABADTE is the primary hotspot, orchestrating patient census processing with many calls into utility programs and multiple file operations.
- XFXLDSC and XFXTABL are data-access hotspots, mediating organizational hierarchy lookups and table-driven dictionary resolutions.
- XFXCNTR is a pure utility hotspot (text centering) reused by HABADTE with no file IO.

---

## Section 3 – Call Graph Summary

This section summarizes the call/copy/reference edges from the dependency graph.

| Caller (f) | Callee (t) | Relationship |
|------------|------------|--------------|
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

This call graph shows HABADTE as the central orchestrator calling utility programs and interacting with key data structures, while XFX* and HXX* programs encapsulate reusable services.

---

## Section 4 – Business Rules by Program

Business rules are grouped below by their source program using the approved_rules list.

### XFXCNTR

Text centering utility rules:

- **BR-001** – Text centering — skip processing if the entire input field is blank (nothing to center).
- **BR-002** – Text centering — skip processing if the text already starts at the first position (already left-aligned).

### XFXCYMD

Date validation and range enforcement:

- **BR-003** – Date validation — reject dates before 1800: the system does not support historical records predating 1800.
- **BR-004** – Date validation — reject dates beyond 2100: the system does not support forecast dates past the year 2100.
- **BR-005** – Date validation — month must be January (01) or later: a month value below 01 is not a valid calendar month.
- **BR-006** – Date validation — month must be December (12) or earlier: a month value above 12 is not a valid calendar month.
- **BR-007** – Date validation — day must be 01 or later: a day value of 0 or less is not a valid calendar day.
- **BR-008** – When VDD is greater than DYS(VMM), branch to 'EXIT'.

### XFXLDSC

Organizational level description lookups:

- **BR-009** – Organizational level lookup — the level code is out of the valid range; return no description.
- **BR-010** – Organizational level lookup — the level code is out of the valid range; return no description.
- **BR-011** – Organizational level lookup — the level code is out of the valid range; return no description.
- **BR-012** – Organizational level lookup — the level code is out of the valid range; return no description.

### XFXTABL

Table-driven dictionary and status behavior:

- **BR-013** – When *IN79 equals on/active, branch to 'EXIT'.
- **BR-014** – When *IN79 equals on/active, branch to 'EXIT'.
- **BR-015** – When *IN79 equals on/active, branch to 'EXIT'.
- **BR-016** – When *IN79 equals on/active, branch to 'EXIT'.

### HABADTE

Patient census and filtering rules:

- **BR-017** – Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census.
- **BR-018** – Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census.
- **BR-019** – Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.
- **BR-020** – Organisational Level Filter: Each patient record must belong to the requested organisation. The report accepts an org level (1–6) and org code from the caller. If the patient's org code at that level does not match, the record is excluded.

(Additional HABADTE rules such as BR-021 and BR-022 are documented elsewhere in the BRD but are beyond the subset listed in approved_rules in this context.)

### HXXAPPPRF

Application preference access:

- **BR-030** – Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed during processing.
- **BR-031** – Database table access — HXXAPPPRF reads from the 'HXPAPPL6' table to retrieve configuration or reference data needed during processing.
- **BR-032** – Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed during processing.

---

## Section 5 – Missing Dependencies

High and medium impact gaps represent referenced components that were not found in the harvested source set.

| Name | Type | Impact | Referenced By |
|------|------|--------|---------------|
| CXXXMLC | COPYBOOK | HIGH | HABADTE |
| HXHAPPPRF | PROGRAM | MEDIUM | XFXMRNROL |
| ****HXPXML | FILE | MEDIUM | HABADTE |
| PRINTER | FILE | MEDIUM | HABADTE |

Impact analysis:
- **CXXXMLC**: High-impact copybook used by HABADTE alongside XML-related copy members. Likely defines XML header/control structures required for census export. Missing this copybook complicates reconstruction of XML payloads in the target system.
- **HXHAPPPRF**: Medium-impact program called by XFXMRNROL. Appears to provide application or facility-specific preferences influencing MRN/account display. Its absence introduces ambiguity into identifier preference logic.
- ******HXPXML**: Medium-impact file name suggesting an XML configuration or routing table. Referenced by HABADTE, probably as part of XML export or transformation logic.
- **PRINTER**: Medium-impact file referenced by HABADTE, likely controlling printer routing or report output destinations. Missing this definition hinders accurate modeling of print behavior.

---

## Section 6 – Tech Debt Summary

Tech debt is summarized at application level.

- **Total findings:** 4
- **Estimated remediation effort:** 26.9 hours
- **By severity:**
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

Interpretation:
- The single HIGH-severity finding is likely associated with HABADTE’s very high cyclomatic complexity (152, HIGH band) and its role as a dependency hotspot.
- MEDIUM findings probably correspond to missing or poorly isolated interfaces (CXXXMLC, HXHAPPPRF, ****HXPXML, PRINTER) and schema coupling in dictionary/hierarchy programs.
- No LOW-severity findings are recorded, indicating that debt is concentrated in a small number of critical components.

This module specification document provides a consolidated view of components, dependencies, rules and known gaps to guide modernization and risk assessment.