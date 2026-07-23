# Process Narratives – HABADTE Project

This document describes the business process narrative for each interpreted program in the HABADTE application, using the semantic interpretations and approved business rules. It focuses on how each module contributes to the overall Active Patients by Date and Time census.

---

## XFXCNTR – Text Centering Utility

### Overview

RPGLE program in domain **Data Maintenance**. Contains 2 rule(s) with average confidence 43%.

XFXCNTR is a small utility used to center text strings for display on screens and printed or XML reports. It optimizes centering behavior by detecting blank or fully occupied strings, skipping unnecessary processing when centering is either meaningless or impossible.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

Key rules:

- **BR-001** – Text Centering — Blank String Optimization: If the input string contains no non-space characters, centering is skipped entirely.
- **BR-002** – Text Centering — Full String Optimization: If the string already occupies the maximum field width and starts at position 1, it is considered centered and the routine exits.

These rules ensure that headings and labels are processed efficiently and displayed correctly.

### Related Approved Rules (Same Domain)

Other Data Maintenance rules provide context but are implemented in different modules:

- BR-003–BR-008 (date validation in XFXCYMD).
- BR-009, BR-012 (level code validation in XFXLDSC).
- BR-013 (indicator-based exit in XFXTABL).

### Data Touched

XFXCNTR operates purely on in-memory strings and does not access PF/LF tables. No PHI-bearing files are directly involved.

Relevant files (from overall project context for reporting outputs):
- HXPXMLD / HXPXMLR: XML detail and header files that eventually store centered headings.
- PRINTER (device file): used by HABADTE for legacy spool output where centered headings appear.

PHI-related files (HXPDICT, OAPIRNK, OMPMAST, HAPTRFR, OXPBNFIT) are not directly touched by XFXCNTR.

---

## XFXCYMD – Date Validation and Conversion

### Overview

RPGLE program in domain **Data Maintenance**. Contains 6 rule(s) with average confidence 58%.

XFXCYMD performs validation and conversion of calendar dates, ensuring that dates used in admission, discharge, and census logic fall within acceptable ranges and respect month/day boundaries, including leap-year behavior through calls to XFXLEAP.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

Key rules:

- **BR-003** – Reject year < 1800.
- **BR-004** – Reject year > 2100.
- **BR-005** – Reject month < 01.
- **BR-006** – Reject month > 12.
- **BR-007** – Reject day < 01.
- **BR-008** – Reject day greater than maximum days in month (branch to EXIT).

These rules standardize date values used across the system, including HABADTE’s censusDate parameter.

### Related Approved Rules (Same Domain)

- BR-001, BR-002 (text centering in XFXCNTR).
- BR-009, BR-012 (level code validation in XFXLDSC).
- BR-013 (indicator exit in XFXTABL).

### Data Touched

XFXCYMD works on numeric date values; no tables are read directly. Indirectly influences:
- HAPTRFR: transfer dates used when intersecting with censusDate.
- OMPMAST: admission/discharge dates used in HABADTE filters.

PHI-bearing fields such as AFACCT, AFMRNO, MMMRNO, MMACCT are not accessed by XFXCYMD itself but rely on valid dates ensured by this utility.

---

## XFXLDSC – Organisational Level Description Lookup

### Overview

RPGLE program in domain **Data Maintenance**. Contains 4 rule(s) with average confidence 56%.

XFXLDSC resolves textual descriptions for organisational levels (1–6) used in reporting and filtering, rejecting invalid level codes and reading from hierarchical tables HXPLVL1–HXPLVL6.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

Key rules:

- **BR-009** – Level lookup: reject if level code exceeds valid range.
- **BR-012** – Variant of level code range validation.

These rules ensure that HABADTE and other reports use valid organisational scope and can display human-readable descriptions for hospitals, units, and facilities.

### Related Approved Rules

Within Patient Management, HABADTE’s BR-020 relies on valid level codes when applying organisational filters.

### Data Touched

From data_dict_schema and dep_edges:
- HXPLVL1–HXPLVL6 (PFs): hierarchical level tables, unique keys HX1NUM–HX6NUM.
- HXFLVL1–HXFLVL6 record formats: read operations logged in dep_edges.

No PHI fields are present in these tables according to the compact schema; XFXLDSC operates purely on structural codes and descriptions.

---

## XFXTABL – Generic Code Table Lookup

### Overview

RPGLE program in domain **Data Maintenance**. Contains 4 rule(s) with average confidence 90%.

XFXTABL performs generic table lookups against HXPTABLD and its logical views (HXLTABLD, HXLTABLP, HXLTABLS), returning descriptions and mapping flags for codes such as room classes. It includes indicator-driven early exit behavior for performance and control.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

Key rule:

- **BR-013** – When *IN79 equals on/active, branch to EXIT.

This rule allows calling logic to disable lookups under certain conditions, simplifying error handling or bypassing enrichment when not needed.

In HABADTE, XFXTABL underpins BR-024 (room class description lookup) and BR-025 (leave flag from XFDMAP).

### Related Approved Rules

- BR-024 – Room Class Description Lookup (HABADTE).
- BR-025 – Hospital/Therapeutic Leave Flag (HABADTE).

### Data Touched

Per data_lineage and dep_edges:
- HXPTABLD (PF) via XFFTABLD (record format).
- HXLTABLD, HXLTABLP, HXLTABLS (LFs) for different key sequences.

These tables are non-PHI reference structures.

---

## HABADTE – Active Patients by Date and Time Census

### Overview

RPGLE program in domain **Patient Management**. Contains 13 rule(s) with average confidence 75%.

HABADTE is the main orchestration program that produces the Active Patients by Date and Time census. It filters patient accounts based on admission/discharge status, outpatient/pre-admit flags, and organisational scope; enriches each included patient with transfer-derived room data, nursing station details, and room class/leave status; and emits XML and printed reports.

### Domain and Program Type

- **Domain**: PATIENT_MANAGEMENT
- **Type**: RPGLE

### Business Rules Applied

Key HABADTE rules:

- **BR-017** – Exclude pre-admitted patients (fileIndicator = 0).
- **BR-018** – Exclude voided accounts (voidFlag = 'V').
- **BR-019** – Exclude outpatients (ioIndicator = 'O').
- **BR-020** – Organisational level filter using orgLevel and orgCode.
- **BR-021** – Exclude patients admitted after censusDate.
- **BR-022** – Exclude patients discharged on or before censusDate (non-zero dischargeDate).
- **BR-023** – Room number retrieval from transfer history HAPTRFR.
- **BR-024** – Room class description lookup via HXPDICT/HXPTABLD.
- **BR-025** – Hospital/Therapeutic leave flag derived from XFDMAP mapping field.

These rules define the inclusion set and the enrichment logic for the census.

### Related Approved Rules (Same Domain)

No additional Patient Management rules beyond HABADTE’s set appear in the approved_rules list; HABADTE is the central Patient Management program for this run.

### Data Touched

From data_dict_schema and data_lineage:
- **HAPTRFR** (PF): transfer history, PHI fields AFACCT (AccountNumber), AFMRNO (MRN). HABADTE declares and reads HAPTRFR and uses it for BR-023 enrichment.
- **HMLMAST5H** (LF over TMPMAST): used conceptually for sorted master records by station and admit date/time (DECLARE in data_lineage). Contains patient-related fields.
- **HXPBNFIT** (LF over TXPBNFIT): declared but not central to census rules; holds benefit data.
- **HAPIRNK** (LF over TAPIRNK): declared in HABADTE, likely used for ranking or indexing accounts.
- **HXPNSTN** (LF over TXPNSTN): declared; via XFFNSTN, HABADTE reads nursing station definitions.
- **HXPXMLD/HXPXMLR** and **HXFXMLH**: XML detail and header files used for electronic report output.
- **PRINTER** (device file): used for legacy spool output.

PHI-related files and fields:
- HXPDICT: MRN, account, names, room numbers, phone numbers, DOB.
- OAPIRNK: BRKMRN (MRN).
- OMPMAST: MMMRNO/MMACCT/MMNAME/MMPSSN/MMMMRN.
- OXPBNFIT: XFBTEL (PhoneNumber).

HABADTE interacts with some of these directly (HAPTRFR, station file via HXPNSTN, XML files) and indirectly via lookups (room class mapping through HXPTABLD, potential master data from OMPMAST/HXPDICT), making it the key PHI-exposing process in the project.

---

## HXXAPPPRF – Application Preferences (Profiles)

### Overview

SQLRPGLE program in domain **Data Maintenance**. Contains 3 rule(s) with average confidence 65%.

HXXAPPPRF reads application preference tables to supply configuration data to other programs, notably XFXMRNROL. Preferences may include MRN rollup behavior, facility-specific options, or other control flags.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: SQLRPGLE

### Business Rules Applied (from interpretation)

- Reads from table HXPAPPPRF to retrieve configuration or reference data.
- Reads from table HXPAPPL6 to retrieve level-6-specific configuration.

While these table names are captured in interpretations, they are not explicitly present in the compact data_dict_schema. The behavior is configuration-driven and affects how MRNs are aggregated or displayed.

### Related Approved Rules

No direct BR-IDs from approved_rules are assigned to HXXAPPPRF, but XFXMRNROL’s MRN rollup behavior depends on it.

### Data Touched

Configuration tables (HXPAPPPRF, HXPAPPL6) – not fully documented; assumed non-PHI settings.

---

## XFXGETID – XML ID and Fragment Service

### Overview

RPGLE program in domain **Data Maintenance**. Cyclomatic complexity: 1.

XFXGETID provides XML element IDs and possibly static text fragments used by HABADTE when writing XML header and detail records. It declares and reads HXPXMLR/HXFXMLR.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

No explicit approved_rules entries, but its narrative indicates simple control logic around reading XML header records.

### Data Touched

From data_lineage and dep_edges:
- HXPXMLR (PF) and HXFXMLR (record format): read for XML header information.

No PHI fields are identified in these tables; they hold report metadata.

---

## XFXLEAP – Leap Year Utility

### Overview

RPGLE program in domain **Data Maintenance**. Cyclomatic complexity: 1.

XFXLEAP implements leap-year logic, used by XFXCYMD to determine the correct number of days in a given month/year combination.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

No direct BR-IDs; its behavior supports BR-008 (day vs month boundary check) in XFXCYMD.

### Data Touched

Works on numeric year/month/day values only; no tables accessed and no PHI involvement.

---

## XFXMRNROL – MRN Rollup Controller

### Overview

RPGLE program in domain **Data Maintenance**. Calls programs HXHAPPPRF and HXXAPPPRF. Cyclomatic complexity: 1.

XFXMRNROL coordinates MRN rollup behavior (e.g., whether multiple MRNs are combined or presented separately) based on application profiles. HABADTE calls XFXMRNROL early in its processing to determine how patient identifiers should be aggregated.

### Domain and Program Type

- **Domain**: DATA_MAINTENANCE
- **Type**: RPGLE

### Business Rules Applied

No approved_rules entries directly, but the narrative emphasizes that it calls:
- HXHAPPPRF – likely a traditional profile program.
- HXXAPPPRF – SQLRPGLE profile reader.

### Related Approved Rules

HABADTE’s MRN-related handling implicitly depends on MRN rollup configuration; although not explicitly expressed as BR-IDs, any change in rollup semantics must be coordinated through XFXMRNROL.

### Data Touched

Underlying profile tables via HXXAPPPRF/HXHAPPPRF; PHI involvement is indirect (MRN behavior, not raw PHI fields).

---

## Cross-Program Narrative – End-to-End Census Flow

Taken together, these programs implement the following high-level process for the Active Patients by Date and Time census:

1. **Configuration and Setup**
   - HXXAPPPRF reads application preferences.
   - XFXMRNROL derives MRN rollup behavior from profiles.
   - XFXLDSC validates organisational level codes and retrieves descriptions from HXPLVL1–HXPLVL6.
   - XFXCYMD validates the census date.

2. **Selection of Candidate Patients (HABADTE)**
   - HABADTE filters out pre-admissions, voided accounts, and outpatients (BR-017–BR-019).
   - HABADTE restricts selection to the requested organisational scope (BR-020).
   - HABADTE enforces admission/discharge date boundaries relative to censusDate (BR-021–BR-022).

3. **Enrichment (HABADTE + XFXTABL + XFXGETID)**
   - HABADTE uses HAPTRFR to retrieve the latest transfer record for each account (BR-023).
   - HABADTE uses HXPNSTN/XFFNSTN to resolve nursing station names.
   - HABADTE calls XFXTABL (HXPTABLD/HXLTABL*) to describe room class and derive leave flags (BR-024, BR-025).
   - XFXGETID uses HXPXMLR/HXFXMLR to supply XML section IDs and static content; HABADTE writes XML detail (HXPXMLD) and header (HXFXMLH) records.
   - XFXCNTR centers headings and facility names for display.

4. **Output and Reporting**
   - HABADTE writes XML records and produces spool output to PRINTER.
   - Downstream consumers receive structured census data for operational decision-making.

PHI is concentrated in patient-related PFs (HAPTRFR, OMPMAST, HXPDICT, OAPIRNK, OXPBNFIT); utilities operate mostly on codes, dates, and configuration, providing a clear separation between infrastructure and patient management logic.
