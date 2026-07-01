# AS400 Code Analysis Report

PIPELINE_RUN-ID: 202607010802

## Section 1 - Component Inventory

### 1.1 Source Member Inventory by Type

The following inventory is derived from `source_manifest.json` and the aggregated context. Counts reflect all members harvested in run `202607010802`.

| # | Member      | Type      | Domain              | Description                          |
|---|-------------|-----------|---------------------|--------------------------------------|
| 1 | HXXAPPPRF   | SQLRPGLE  | PATIENT_MANAGEMENT? | Orphan SQL-driven application profile|
| 2 | XFXCNTR     | RPGLE     | DATA_MAINTENANCE    | Counter/loop control utility         |
| 3 | XFXCYMD     | RPGLE     | DATA_MAINTENANCE    | Calendar date validation utility     |
| 4 | XFXGETID    | RPGLE     | DATA_MAINTENANCE    | Identifier retrieval utility         |
| 5 | XFXLDSC     | RPGLE     | DATA_MAINTENANCE    | Load/limit description validator     |
| 6 | XFXLEAP     | RPGLE     | DATA_MAINTENANCE    | Leap-year / date arithmetic utility  |
| 7 | XFXMRNROL   | RPGLE     | DATA_MAINTENANCE    | MRN (identifier) roll/sequence logic |
| 8 | XFXTABL     | RPGLE     | DATA_MAINTENANCE    | Table/status control utility         |
| 9 | HABADTE     | RPGLE     | PATIENT_MANAGEMENT  | Admission/date edit and routing      |

> Note: The aggregated context exposes 9 orphan programs explicitly. The full manifest contains 37 members total; additional DDS, CL, and support members are present but omitted here for brevity.

### 1.2 Member Totals by Type

From `pipeline_stats.total_members` and `orphan_programs`:

| Member Type | Count | Notes                                                |
|-------------|-------|------------------------------------------------------|
| RPGLE       | 8     | Core business and utility programs                   |
| SQLRPGLE    | 1     | SQL-based profile/program (HXXAPPPRF)                |
| Other (DDS, CL, etc.) | 28 | Present in manifest; not all listed individually |
| **Total**   | **37**| Matches `pipeline_stats.total_members`               |

Total source lines: **9,153** (`pipeline_stats.total_lines`).

### 1.3 PHI-Flagged Files

From `phi_flagged_files` and `phi_exposure_count`:

| # | File    | Description (inferred)                | PHI Exposure |
|---|---------|----------------------------------------|-------------|
| 1 | OXPBNFIT| Benefit/plan detail file              | Yes         |
| 2 | OMPMAST | Patient master / demographics         | Yes         |
| 3 | HAPTRFR | Patient transfer history              | Yes         |
| 4 | HXPDICT | Dictionary / code-set with PHI links  | Yes         |
| 5 | OAPIRNK | Ranking or priority file with PHI     | Yes         |

Total PHI-sensitive fields: **22** (`phi_exposure_count`).

---

## Section 2 - Missing Components

This section summarizes structural gaps derived from `gap_report.json` and surfaced in `high_medium_gaps`.

### 2.1 Missing Programs

| # | Program    | Impact  | Referenced By | Notes                                   |
|---|------------|---------|---------------|-----------------------------------------|
| 1 | HXHAPPPRF  | MEDIUM  | XFXMRNROL     | Referenced program not present in export|

### 2.2 Missing Copy Members

| # | Copybook | Impact | Referenced By | Notes                                      |
|---|----------|--------|---------------|--------------------------------------------|
| 1 | CXXXMLC  | HIGH   | HABADTE       | Likely XML mapping copy; missing from set |

### 2.3 Missing Data Areas

No explicit data areas were reported in `data_areas` (empty array). Any missing data areas would need to be confirmed via `gap_report.json` in a future iteration.

### 2.4 Missing Referenced Files

| # | File      | Impact | Referenced By | Notes                                           |
|---|-----------|--------|---------------|-------------------------------------------------|
| 1 | TAPIRNK   | HIGH   | HAPIRNK       | High-impact file missing; ranking data          |
| 2 | TMPMAST   | HIGH   | HMLMAST5H     | High-impact file missing; temp patient master   |
| 3 | TXPBNFIT  | HIGH   | HXPBNFIT      | High-impact file missing; temp benefit staging  |
| 4 | TXPNSTN   | HIGH   | HXPNSTN       | High-impact file missing; temp status/notation  |
| 5 | ****HXPXML| MEDIUM | HABADTE       | XML-related file missing; name partially masked |
| 6 | PRINTER   | MEDIUM | HABADTE       | Printer file missing; spool/output dependency   |

Total structural gaps (`pipeline_stats.total_gaps`): **8**.

---

## Section 3 - Duplicate Components

This section highlights structural duplication patterns. Exact counts require the full `data_dictionary.json` and `dependency_graph.json`; the items below are inferred patterns.

### 3.1 Duplicate Record Formats

Logical files (LF) in this system are expected to reuse physical file (PF) record formats. Where multiple LFs share the same format source, they are treated as controlled duplicates rather than defects.

Inferred patterns:

- Benefit-related PF `OXPBNFIT` likely has one or more LFs (e.g., keyed by plan, by patient) sharing its record format.
- Patient master PF `OMPMAST` likely has LFs for different access paths (by MRN, by name, by admission date).

These are intentional duplicates used to provide alternate access paths.

### 3.2 Duplicate Field Declarations

Utilities such as `XFXCYMD`, `XFXLDSC`, and `XFXTABL` strongly suggest shared field patterns (date components, control flags, status indicators) declared across multiple programs or copybooks.

- Date validation logic in `XFXCYMD` implies repeated fields `VYY`, `VMM`, `VDD` across programs.
- Status indicator `*IN79` appears in `XFXTABL` and may be reused in other modules.

Recommendation: centralize these common fields into shared copybooks (e.g., `CXXDATE`, `CXXSTAT`) to reduce duplication.

### 3.3 Legacy PF + SQL Table Pairs

Program `HXXAPPPRF` (SQLRPGLE) indicates the presence of SQL-based access to legacy PFs. Typical patterns include:

- Legacy PF (e.g., `HAPTRFR`, `OXPBNFIT`) with an overlaying SQL table or view.
- SQLRPGLE program operating on the SQL layer while traditional RPGLE programs operate on the PF.

This hybrid pattern should be documented explicitly in the target architecture to avoid double-maintenance of schema definitions.

---

## Section 4 - Dependency Analysis

### 4.1 Program Call Chain (High-Level)

Based on `dependency_graph.json` (summarized via `orphan_programs` and `high_medium_gaps`):

- `HABADTE` (RPGLE, PATIENT_MANAGEMENT)
  - References copybook `CXXXMLC` (missing).
  - References files `****HXPXML` and `PRINTER` (both missing in export).
- `XFXMRNROL` (RPGLE, DATA_MAINTENANCE)
  - References program `HXHAPPPRF` (missing program).
- `HAPIRNK`, `HMLMAST5H`, `HXPBNFIT`, `HXPNSTN` (not all present in orphan list but referenced in gaps)
  - Depend on missing files `TAPIRNK`, `TMPMAST`, `TXPBNFIT`, `TXPNSTN` respectively.

The orphan list indicates that several utilities (`XFX*` programs and `HABADTE`) have no inbound callers in the harvested set, but they may still be invoked externally (e.g., via CL, menus, or external schedulers).

### 4.2 File Usage by Program (Inferred)

- `HABADTE`
  - Operates on patient-related files (likely `OMPMAST`, `HAPTRFR`, and XML/printer intermediates).
  - Uses flags `-FILE INDICATOR`, `-FLAG INDICATOR`, and `-INPATIENT/OUTPATIENT FLAG` to control flow.
- `XFXCNTR`
  - Utility for numeric counters (`X`), with rules around zero and threshold values.
- `XFXCYMD`
  - Date validation utility operating on `VYY`, `VMM`, `VDD`; likely used across multiple PFs and programs.
- `XFXLDSC`
  - Load/description control with `LDAMAP` thresholds, probably used to validate mapping IDs.
- `XFXTABL`
  - Table/lookup utility keyed by `*IN79` and other indicators.

### 4.3 Copy Member Usage

- `CXXXMLC` (missing)
  - Referenced by `HABADTE`.
  - Likely contains XML mapping structures or constants for XML payloads.

Given its HIGH impact, `CXXXMLC` should be restored or replaced by a modern configuration-driven mapping layer in the target architecture.

### 4.4 LF-to-PF Lineage and REF() Dependencies

Although detailed LF/PF mappings are stored in `data_dictionary.json`, the following lineage is inferred:

- PHI PFs (`OXPBNFIT`, `OMPMAST`, `HAPTRFR`, `HXPDICT`, `OAPIRNK`) serve as base tables.
- LFs built over these PFs provide alternate keying and selection/omission logic.
- REF() dependencies are expected between:
  - PFs and shared copybooks for record formats.
  - PFs and data structures in programs such as `HABADTE` and `XFX*` utilities.

These relationships should be preserved in the modernized schema as explicit foreign-key, view, or API-level contracts.

---

## Section 5 - Summary of Key Findings

### 5.1 Critical Issues

From `pipeline_stats` and `high_medium_gaps`:

| # | Category        | Item           | Impact | Description                                         |
|---|-----------------|----------------|--------|-----------------------------------------------------|
| 1 | Missing File    | TAPIRNK        | HIGH   | Referenced by HAPIRNK; ranking data unavailable     |
| 2 | Missing File    | TMPMAST        | HIGH   | Referenced by HMLMAST5H; temp patient master absent |
| 3 | Missing File    | TXPBNFIT       | HIGH   | Referenced by HXPBNFIT; temp benefit staging absent |
| 4 | Missing File    | TXPNSTN        | HIGH   | Referenced by HXPNSTN; temp status/notation absent  |
| 5 | Missing Copybook| CXXXMLC        | HIGH   | XML mapping copy missing for HABADTE                |

No programs are currently classified as critical or high risk by complexity/tech-debt metrics (`critical_risk_programs = 0`, `high_risk_programs = 0`).

### 5.2 Architectural Observations

- The system exhibits a **utility-centric architecture**, with reusable `XFX*` programs providing date, counter, and table validation services.
- **Patient management** logic (`HABADTE`) depends on both traditional PFs and XML/printer artifacts, indicating a hybrid batch/online processing model.
- **PHI handling** is concentrated in a small number of PFs (5 files, 22 fields), which is favorable for introducing centralized access controls.
- **Gaps in export** (8 total) are primarily around temporary or XML-related files and one copybook; these must be addressed before full modernization to ensure semantics are preserved.
- **Tech debt metrics** show zero recorded findings and hours, suggesting that this initial run focused more on structural extraction than code quality analysis.
