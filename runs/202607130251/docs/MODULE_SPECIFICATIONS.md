# Module Specifications – HABADTE AS400 Application

## Section 1 – Component Inventory

The HABADTE subsystem comprises all source members listed below. Complexity metrics are taken from the aggregated analysis; programs without explicit complexity entries are marked as **N/A**. Orphan status is based on the `orphan_programs` list in the aggregated context.

| Program     | Type      | Subsystem  | Lines | CC  | Risk Band | Orphan |
|------------|-----------|-----------|------:|----:|-----------|--------|
| HAPIRNK    | DDS_LF    | HAP       | 13   | N/A | N/A       | no     |
| HAPTRFR    | DDS_PF    | HAP       | 72   | N/A | N/A       | no     |
| HMLMAST5H  | DDS_LF    | UNGROUPED | 12   | N/A | N/A       | no     |
| HXLTABLD   | DDS_LF    | HXLT      | 11   | N/A | N/A       | no     |
| HXLTABLP   | DDS_LF    | HXLT      | 12   | N/A | N/A       | no     |
| HXLTABLS   | DDS_LF    | HXLT      | 12   | N/A | N/A       | no     |
| HXPBNFIT   | DDS_LF    | HXP       | 12   | N/A | N/A       | no     |
| HXPDICT    | DDS_PF    | HXP       | 6130 | N/A | N/A       | no     |
| HXPLVL1    | DDS_PF    | HXPL      | 49   | N/A | N/A       | no     |
| HXPLVL2    | DDS_PF    | HXPL      | 52   | N/A | N/A       | no     |
| HXPLVL3    | DDS_PF    | HXPL      | 52   | N/A | N/A       | no     |
| HXPLVL4    | DDS_PF    | HXPL      | 52   | N/A | N/A       | no     |
| HXPLVL5    | DDS_PF    | HXPL      | 55   | N/A | N/A       | no     |
| HXPLVL6    | DDS_PF    | HXPL      | 321  | N/A | N/A       | no     |
| HXPNSTN    | DDS_LF    | HXP       | 12   | N/A | N/A       | no     |
| HXPTABLD   | DDS_PF    | HXP       | 19   | N/A | N/A       | no     |
| HXPXMLD    | DDS_PF    | HXPX      | 19   | N/A | N/A       | no     |
| HXPXMLR    | DDS_PF    | HXPX      | 19   | N/A | N/A       | no     |
| OAPIRNK    | DDS_PF    | UNGROUPED | 80   | N/A | N/A       | no     |
| OMPMAST    | DDS_PF    | UNGROUPED | 310  | N/A | N/A       | no     |
| OXPBNFIT   | DDS_PF    | OXP       | 48   | N/A | N/A       | no     |
| OXPNSTN    | DDS_PF    | OXP       | 65   | N/A | N/A       | no     |
| TAPIRNK    | DDS_PF    | UNGROUPED | 167  | N/A | N/A       | no     |
| TMPMAST    | DDS_PF    | UNGROUPED | 688  | N/A | N/A       | no     |
| TXPBNFIT   | DDS_PF    | TXP       | 164  | N/A | N/A       | no     |
| TXPNSTN    | DDS_PF    | TXP       | 116  | N/A | N/A       | no     |
| HXXAPPPRF  | SQLRPGLE  | HXXA      | 123  | 1   | LOW       | yes    |
| XFXCNTR    | RPGLE     | XFXC      | 49   | 3   | LOW       | yes    |
| XFXCYMD    | RPGLE     | XFXC      | 83   | 7   | LOW       | yes    |
| XFXGETID   | RPGLE     | XFX       | 61   | 1   | LOW       | yes    |
| XFXLDSC    | RPGLE     | XFXL      | 135  | 5   | LOW       | yes    |
| XFXLEAP    | RPGLE     | XFXL      | 61   | 1   | LOW       | yes    |
| XFXMRNROL  | RPGLE     | XFX       | 65   | 1   | LOW       | yes    |
| XFXTABL    | RPGLE     | XFX       | 164  | 9   | LOW       | yes    |
| CXXXMLP    | SQLRPGLE  | UNGROUPED | 25   | 1   | LOW       | no     |
| HXXAPPPRFP | RPGLE     | HXXA      | 42   | 1   | LOW       | no     |
| HXXCNTRL   | RPGLE     | HXX       | 8    | 1   | LOW       | no     |
| HXXLDA     | RPGLE     | HXXL      | 53   | 1   | LOW       | no     |
| HXXLEVEL   | RPGLE     | HXXL      | 25   | 1   | LOW       | no     |
| HXXXML     | RPGLE     | HXX       | 11   | 1   | LOW       | no     |
| HABADTE    | RPGLE     | HA        | 821  | 152 | HIGH      | yes    |

Notes:
- HABADTE is the central orchestrator with very high cyclomatic complexity (152) and the largest line count.
- Utility programs (XFX* and HXX* series) are generally low‑complexity and reusable across workflows.

## Section 2 – Dependency Hotspots

Top dependency hotspots as reported in `dep_hotspots`:

| Program    | Score | Fan-In | Fan-Out | File Ops |
|-----------|------:|-------:|--------:|---------:|
| HABADTE   | 38    | 0      | 13      | 6        |
| XFXLDSC   | 15    | 1      | 0       | 6        |
| XFXTABL   | 11    | 1      | 0       | 4        |
| XFXCNTR   | 9     | 3      | 0       | 0        |
| XFXMRNROL | 7     | 1      | 2       | 0        |
| HXXAPPPRF | 7     | 1      | 2       | 0        |
| XFXGETID  | 7     | 1      | 1       | 1        |
| XFXCYMD   | 5     | 1      | 1       | 0        |
| HXHAPPPRF | 3     | 1      | 0       | 0        |
| XFXLEAP   | 3     | 1      | 0       | 0        |
| HAPIRNK   | 2     | 0      | 0       | 1        |
| HXPNSTN   | 2     | 0      | 0       | 1        |
| HXLTABLD  | 2     | 0      | 0       | 1        |
| HMLMAST5H | 2     | 0      | 0       | 1        |
| HXLTABLS  | 2     | 0      | 0       | 1        |
| HXPBNFIT  | 2     | 0      | 0       | 1        |
| HXLTABLP  | 2     | 0      | 0       | 1        |

Interpretation:
- HABADTE has the highest score and fan‑out, confirming its role as the main batch/report engine.
- XFXLDSC and XFXTABL are important configuration and lookup services with multiple file operations.
- XFXCNTR is a heavily reused utility (high fan‑in) for text centering.

## Section 3 – Call Graph Summary

Key call, copy, and file‑relationship edges:

| Caller      | Callee     | Dependency Type |
|------------|------------|-----------------|
| XFXCYMD    | XFXLEAP    | CALL            |
| XFXMRNROL  | HXHAPPPRF  | CALL            |
| XFXMRNROL  | HXXAPPPRF  | CALL            |
| HABADTE    | XFXMRNROL  | CALL            |
| HABADTE    | XFXCNTR    | CALL            |
| HABADTE    | XFXLDSC    | CALL            |
| HABADTE    | XFXCNTR    | CALL            |
| HABADTE    | XFXCNTR    | CALL            |
| HABADTE    | XFXCYMD    | CALL            |
| HABADTE    | XFXGETID   | CALL            |
| HABADTE    | XFXTABL    | CALL            |
| HXXAPPPRF  | HXXCNTRL   | COPY            |
| HXXAPPPRF  | HXXAPPPRFP | COPY            |
| XFXGETID   | HXXLDA     | COPY            |
| HABADTE    | HXXLDA     | COPY            |
| HABADTE    | HXXLEVEL   | COPY            |
| HABADTE    | HXXXML     | COPY            |
| HABADTE    | CXXXMLP    | COPY            |
| HABADTE    | CXXXMLC    | COPY            |
| HAPIRNK    | TAPIRNK    | PFILE_OF        |
| HMLMAST5H  | TMPMAST    | PFILE_OF        |
| HXLTABLD   | HXPTABLD   | PFILE_OF        |
| HXLTABLP   | HXPTABLD   | PFILE_OF        |
| HXLTABLS   | HXPTABLD   | PFILE_OF        |
| HXPBNFIT   | TXPBNFIT   | PFILE_OF        |
| HXPNSTN    | TXPNSTN    | PFILE_OF        |
| XFXGETID   | HXFXMLR    | READ            |
| XFXLDSC    | HXFLVL1    | READ            |
| XFXLDSC    | HXFLVL2    | READ            |
| XFXLDSC    | HXFLVL3    | READ            |
| XFXLDSC    | HXFLVL4    | READ            |
| XFXLDSC    | HXFLVL5    | READ            |
| XFXLDSC    | HXFLVL6    | READ            |
| XFXTABL    | XFFTABLD   | READ            |
| XFXTABL    | XFFTABL2   | READ            |
| XFXTABL    | XFFTABL3   | READ            |
| XFXTABL    | XFFTABL4   | READ            |
| HABADTE    | HAPTRFR    | READ            |
| HABADTE    | XFFNSTN    | READ            |
| HABADTE    | HXFXMLH    | READ            |
| HABADTE    | HXFXMLH    | UPDATE          |
| HABADTE    | HXFXMLH    | WRITE           |
| HABADTE    | HXFXMLD    | WRITE           |

This summary shows HABADTE orchestrating patient census processing by delegating to utility programs and reading/writing key PFs and XML record files.

## Section 4 – Business Rules by Program

Business rules are grouped by `source_program` from `approved_rules` and enriched with key_rules from `interpretations_detail`.

### HABADTE (PATIENT_MANAGEMENT)

Core census filtering rules:

- **BR-017** – Exclude Pre-Admitted Patients
  - Text: "Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census."
- **BR-018** – Exclude Voided Accounts
  - Text: "Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census."
- **BR-019** – Exclude Outpatients
  - Text: "Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded."

These rules ensure that only active, admitted inpatient accounts participate in the census output.

### XFXCNTR (DATA_MAINTENANCE)

Text‑centering rules:

- **BR-001** – Skip Blank Input
  - Text: "Text centering — skip processing if the entire input field is blank (nothing to center)."
- **BR-002** – Skip Already Left‑Aligned Text
  - Text: "Text centering — skip processing if the text already starts at the first position (already left-aligned)."

### XFXCYMD (DATA_MAINTENANCE)

Date validation rules:

- **BR-003** – Reject Dates Before 1800.
- **BR-004** – Reject Dates Beyond 2100.
- **BR-005** – Month must be January (01) or later.
- **BR-006** – Month must be December (12) or earlier.
- **BR-007** – Day must be 01 or later.
- **BR-008** – When VDD is greater than DYS(VMM), branch to 'EXIT'.

Together these rules enforce valid calendar ranges and guard against impossible or unsupported dates.

### XFXLDSC (DATA_MAINTENANCE)

Organizational level lookup rules (all similar variants):

- **BR-009/010/011/012** – For out‑of‑range level codes, return no description.

These rules ensure that invalid organisational levels do not produce misleading descriptions.

### XFXTABL (DATA_MAINTENANCE)

Table‑driven exit rules:

- **BR-013/014/015/016** – When indicator *IN79 is on/active, branch to 'EXIT'.

This forms a protective mechanism to stop table processing under certain configuration or status conditions.

### HXXAPPPRF (DATA_MAINTENANCE)

Configuration table access rules:

- **BR-020** – Reads from HXPAPPPRF for configuration/reference data.

(Additional rules BR-021 and BR-022 are present in interpretations_detail but outside the approved_rules list and are therefore not repeated here.)

## Section 5 – Missing Dependencies

High and medium impact gaps:

| Name       | Type      | Impact  | Referenced By |
|-----------|-----------|--------:|---------------|
| CXXXMLC   | COPYBOOK  | HIGH    | HABADTE       |
| HXHAPPPRF | PROGRAM   | MEDIUM  | XFXMRNROL     |
| ****HXPXML| FILE      | MEDIUM  | HABADTE       |
| PRINTER   | FILE      | MEDIUM  | HABADTE       |

Impact notes:
- **CXXXMLC** – XML copybook; missing layout or constants critical for HABADTE XML output.
- **HXHAPPPRF** – MRN profile program; its absence obscures part of the MRN roll‑over workflow.
- ******HXPXML** – XML‑related file; definition missing, limiting understanding of XML storage.
- **PRINTER** – printer/spool file; necessary to fully document census report layouts.

## Section 6 – Tech Debt Summary

From `tech_debt_summary`:

- **Total findings:** 4
- **Total remediation hours:** 26.9
- **By severity:**
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

Interpretation:
- A small set of structural or quality issues drive the majority of remediation effort.
- The single HIGH‑severity item is likely associated with HABADTE’s high complexity and/or missing XML/printer components.
- MEDIUM items are expected around MRN roll‑over dependencies and configuration table access.
