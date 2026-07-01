# Low-Level Design (LLD)

PIPELINE_RUN-ID: 202607010802

## Section 1 - Architecture Overview

### 1.1 Technology Stack

| Layer          | Technology / Artifact               | Notes                                         |
|----------------|-------------------------------------|-----------------------------------------------|
| Platform       | IBM i / AS400                       | Legacy midrange platform                      |
| Language       | RPGLE, SQLRPGLE                     | Core business logic                           |
| Data Storage   | DB2 for i (PF/LF, DDS-based)        | Physical and logical files                    |
| Batch Control  | RPGLE utilities (`XFX*`)            | Shared validation and control routines        |
| Integration    | XML (via `CXXXMLC`, `****HXPXML`)   | XML mappings for external interfaces          |
| Output         | Printer files (`PRINTER`, spools)   | Report/spool-based outbound interfaces        |

### 1.2 Component Interaction

- **Utility Programs (`XFX*`)**
  - `XFXCNTR`, `XFXCYMD`, `XFXLDSC`, `XFXTABL`, `XFXGETID`, `XFXLEAP`, `XFXMRNROL` provide reusable services for counters, dates, mapping IDs, and table lookups.
  - These are invoked by domain-specific programs (e.g., `HABADTE`, `HAPIRNK`) as shared subroutine libraries.

- **Domain Programs**
  - `HABADTE` orchestrates patient admission/date edits, applying validation rules and routing logic based on indicators and status flags.
  - Other domain programs (referenced via gaps) handle ranking (`HAPIRNK`), patient master maintenance (`HMLMAST5H`), benefits (`HXPBNFIT`), and status (`HXPNSTN`).

- **Data Layer**
  - Core PFs (`OXPBNFIT`, `OMPMAST`, `HAPTRFR`, `HXPDICT`, `OAPIRNK`) store PHI and transactional data.
  - LFs provide alternate key paths and selection/omission rules.

### 1.3 Design Patterns

- **Call Patterns**
  - Utility programs are likely invoked via `CALL`/`EXSR` from domain programs, with shared parameter conventions (e.g., date fields `VYY`, `VMM`, `VDD`).
  - SQLRPGLE program `HXXAPPPRF` likely acts as a service wrapper around SQL tables.

- **Data Coupling**
  - Programs are tightly coupled to PF/LF structures via DDS-defined record formats and REF() dependencies.
  - PHI files are accessed directly from RPG programs, with limited abstraction.

- **Error Handling**
  - Business rules such as "branch to 'EXIT'" or "branch to 'SKIP'" indicate structured exits on validation failures.
  - Indicators (e.g., `*IN79`, `-FILE INDICATOR`, `-FLAG INDICATOR`) drive control flow and error branches.

- **Initialization**
  - Utilities like `XFXCNTR` and `XFXGETID` suggest standardized initialization of counters and IDs.

- **Framework**
  - The combination of `framework_annotations.json` (not detailed here) and `XFX*` utilities indicates an implicit framework for validation and control, though not formalized as an object-oriented layer.

---

## Section 2 - Database Schema

This section summarizes PF and LF structures using `data_dictionary.json`. Only PHI-related PFs are elaborated explicitly; additional PFs/LFs exist in the full dictionary.

### 2.1 Physical File: OMPMAST (Patient Master)

**Record Format:** (inferred) OMPMASTR

**Unique Key:** Likely MRN or patient ID.

**Key Fields (inferred):**
- PATID / MRN
- FACILITY CODE
- EFFECTIVE DATE

**PHI Sensitive:** Yes (listed in `phi_flagged_files`).

**Fields (sample/inferred):**

| # | Column Name      | DB2 Type | Length | Dec | Nullable | PK | FK | Description                       | PHI Category         |
|---|------------------|----------|--------|-----|----------|----|----|-----------------------------------|----------------------|
| 1 | PATID            | CHAR     | 10     | 0   | No       | Yes| No | Patient identifier (MRN)          | Identifier           |
| 2 | LNAME            | CHAR     | 30     | 0   | No       | No | No | Patient last name                 | Direct identifier    |
| 3 | FNAME            | CHAR     | 20     | 0   | Yes      | No | No | Patient first name                | Direct identifier    |
| 4 | DOB              | DEC      | 8      | 0   | Yes      | No | No | Date of birth (YYYYMMDD)          | Demographic          |
| 5 | GENDER           | CHAR     | 1      | 0   | Yes      | No | No | Gender code                       | Demographic          |
| 6 | ADDR1            | CHAR     | 40     | 0   | Yes      | No | No | Address line 1                    | Demographic          |
| 7 | CITY             | CHAR     | 25     | 0   | Yes      | No | No | City                              | Demographic          |
| 8 | STATE            | CHAR     | 2      | 0   | Yes      | No | No | State                             | Demographic          |
| 9 | ZIP              | CHAR     | 10     | 0   | Yes      | No | No | Postal code                       | Demographic          |

> Exact column names and lengths should be taken from `data_dictionary.json`; the above is representative.

### 2.2 Physical File: OXPBNFIT (Benefit File)

**Record Format:** (inferred) OXPBNFTR

**Unique Key:** Benefit ID + Plan ID.

**PHI Sensitive:** Yes.

| # | Column Name  | DB2 Type | Length | Dec | Nullable | PK | FK | Description                 | PHI Category |
|---|--------------|----------|--------|-----|----------|----|----|-----------------------------|--------------|
| 1 | PLANID       | CHAR     | 10     | 0   | No       | Yes| No | Plan identifier             | None         |
| 2 | BENEFITID    | CHAR     | 10     | 0   | No       | Yes| No | Benefit identifier          | None         |
| 3 | PATID        | CHAR     | 10     | 0   | Yes      | No | Yes| Link to patient master      | Identifier   |

### 2.3 Physical File: HAPTRFR (Patient Transfer)

**PHI Sensitive:** Yes.

- Stores transfer events between units/facilities.
- Key likely composed of patient ID + transfer sequence.

### 2.4 Physical File: HXPDICT (Dictionary)

**PHI Sensitive:** Yes (due to linkage with PHI-bearing entities).

- Stores code sets and descriptions used by other PFs.

### 2.5 Physical File: OAPIRNK (Ranking)

**PHI Sensitive:** Yes.

- Stores ranking or prioritization data, potentially tied to patients or encounters.

### 2.6 Logical Files (LFs)

Logical files are defined over the PFs listed above. `data_dictionary.json` captures, for each LF:

- **Based On PF:** name of base PF.
- **Format Source:** record format from PF or independent DDS.
- **Key Fields:** ordered list of key columns.
- **Select/Omit:** selection criteria (e.g., status flags, date ranges).

Representative examples (inferred):

1. LF over `OMPMAST` keyed by MRN.
2. LF over `OMPMAST` keyed by last name + first name.
3. LF over `OXPBNFIT` keyed by plan and benefit.

---

## Section 3 - Status and Type Reference Data

Status codes and reference data are typically stored in tables and controlled via literals such as `CABEQ` constants. While the full set is in `data_dictionary.json` and `business_rules.json`, the following patterns are evident:

- **Flag Indicators**
  - `-FLAG INDICATOR` with values like `void/voided` controlling skip logic.
  - `-INPATIENT/OUTPATIENT FLAG` with values like `outpatient`.

- **Control Indicators**
  - `*IN79` used as a control/status indicator in `XFXTABL`.

These codes should be centralized into reference tables in the target design, with explicit enums or domain types.

---

## Section 4 - Stored Procedure Logic Mappings

This section maps high-reuse subroutines (fan-in > 1) to their behavior. `business_rules.json` provides a view into the most significant rules.

### 4.1 High-Reuse Subroutines (Inferred)

| # | Program  | Subroutine (Inferred) | Parameters (Inferred)         | Files Affected | Key Logic Summary                                                   | Status Transition |
|---|----------|-----------------------|--------------------------------|----------------|---------------------------------------------------------------------|-------------------|
| 1 | XFXCNTR  | CNTRCHK               | X (counter)                   | N/A            | If X = 0 or X = 40, branch to `EXIT`                               | N/A               |
| 2 | XFXCYMD  | DATECHK               | VYY, VMM, VDD                 | N/A            | Enforces year/month/day ranges; branches to `EXIT` on invalid date | N/A               |
| 3 | XFXLDSC  | MAPCHK                | LDAMAP                        | N/A            | Enforces LDAMAP thresholds; branches to `EXIT` on overflow         | N/A               |
| 4 | XFXTABL  | TABLCHK               | *IN79, table keys             | N/A            | If *IN79 is on/active, branch to `EXIT`                            | N/A               |
| 5 | HABADTE  | FILECHK               | -FILE INDICATOR               | Patient files  | If file indicator = 0, branch to `SKIP`                            | Status = SKIP     |
| 6 | HABADTE  | FLAGCHK               | -FLAG INDICATOR               | Patient files  | If flag = void/voided, branch to `SKIP`                            | Status = SKIP     |
| 7 | HABADTE  | IOFLAGCHK             | -INP/OUT FLAG                 | Patient files  | If flag = outpatient, branch to `SKIP`                             | Status = SKIP     |

Top 30 subroutines are not all enumerated due to limited rule extraction (19 rules total), but the above represent key reusable behaviors.

---

## Section 5 - Service Class Method Reference

Programs can be grouped into service classes based on their role.

### 5.1 DataService

Programs that primarily manage data validation and persistence:

| Program  | Description                          |
|----------|--------------------------------------|
| XFXCNTR  | Counter validation and control       |
| XFXCYMD  | Date validation utility              |
| XFXLDSC  | Mapping/description validation       |
| XFXGETID | Identifier retrieval utility         |
| XFXLEAP  | Leap-year/date arithmetic            |
| XFXMRNROL| MRN/ID roll management               |

### 5.2 WorkflowService

Programs that orchestrate business workflows:

| Program | Description                             |
|---------|-----------------------------------------|
| HABADTE | Admission/date edit and routing logic   |
| HAPIRNK | Ranking workflow (inferred)             |
| HMLMAST5H | Patient master workflow (inferred)    |

### 5.3 UtilityService

Programs providing cross-cutting utilities:

| Program | Description                             |
|---------|-----------------------------------------|
| XFXTABL | Table/status control utility            |

### 5.4 BatchService

Programs likely invoked in batch or scheduled contexts:

| Program  | Description                            |
|----------|----------------------------------------|
| HXXAPPPRF| SQL-based application/profile service  |
| HXPBNFIT | Benefit batch processing (inferred)    |
| HXPNSTN  | Status/notation batch updates (inferred)|

---

## Section 6 - External Interfaces

### 6.1 Inbound Interfaces (MISSING_SOURCE High-Impact Nodes)

From `high_medium_gaps` and `gap_report.json`:

- `CXXXMLC` (COPYBOOK, HIGH impact)
  - Likely defines XML structures for inbound messages into `HABADTE`.
  - Missing from export; indicates an external XML-based interface.

- `****HXPXML` (FILE, MEDIUM impact)
  - XML-related file referenced by `HABADTE`.
  - Suggests inbound/outbound XML staging area.

### 6.2 Outbound Interfaces (SPOOL Nodes)

- `PRINTER` (FILE, MEDIUM impact)
  - Referenced by `HABADTE`.
  - Represents an outbound spool/printer file for reports or notifications.

- Additional spool files may exist in `dependency_graph.json` but are not explicitly listed in the aggregated context.

---

## Section 7 - Performance and Security Notes

### 7.1 Top 10 Programs by Cyclomatic Complexity

`complexity_scores.json` indicates no programs are classified as critical or high risk:

- `critical_risk_programs = 0`
- `high_risk_programs = 0`

This suggests that, within the analyzed set, complexity is manageable. Detailed per-program scores are available in the raw artifact.

### 7.2 PHI Access Control Summary

From `phi_flagged_files` and `phi_exposure_count`:

- PHI is concentrated in 5 files with 22 PHI fields.
- Programs most likely to access PHI include:
  - `HABADTE` (patient admission and date edits).
  - `HXPBNFIT` (benefit processing).
  - Other patient-related programs referencing `OMPMAST`, `HAPTRFR`, `OXPBNFIT`, `OAPIRNK`.

Recommended controls in the modernized system:

- Centralized authorization checks around PHI PFs.
- Field-level masking for reports generated via `PRINTER`.
- Auditing of accesses originating from high-impact programs.

### 7.3 Tech Debt Totals by Category

From `tech_debt_findings.json` and `pipeline_stats`:

- `total_tech_debt_hours = 0.0`
- `total_td_findings = 0`

This indicates that no explicit tech debt items were recorded in this run. Future iterations may enrich this with:

- Use of indicators instead of structured error handling.
- Tight coupling to PF/LF structures.
- Missing abstraction layers around PHI access.
