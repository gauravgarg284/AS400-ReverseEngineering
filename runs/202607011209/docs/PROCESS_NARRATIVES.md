# Process Narratives – HABADTE Project

This document provides program-level business narratives for key components in the HABADTE AS400 ecosystem.
Each section describes the program’s role, domain, applied rules, and data touchpoints to guide Java / Spring Boot / SQL Server modernization.

---

## Program: XFXCNTR

### Overview

RPGLE program `XFXCNTR` operates in the **Data Maintenance** domain.
It is responsible for counter-based control of iteration, ensuring that loops process only a defined number of records and terminate at specific boundary values.
With two primary business rules and an average confidence of 43%, it provides guardrails around batch counters used by higher-level flows such as HABADTE.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied (Key Rules)

From interpretations detail:

- **BR-001:** When X equals zero, branch to `EXIT` (confidence 0.47).
- **BR-002:** When X equals 40, branch to `EXIT` (confidence 0.40).

These rules act as lower- and upper-bound checks on loop counters, preventing underflow (no items) and overflow (too many items).

### Related Approved Rules by Domain

All DATA_MAINTENANCE rules relate to XFXCNTR indirectly:

- BR-001–BR-016 (date validation, mapping constraints, table indicator checks) form the broader environment in which XFXCNTR’s counters are consumed.

### Data Touched

XFXCNTR itself deals with control values rather than persistent data.
It is invoked from HABADTE (per dependency edges) to manage iteration over data read from:

- **OMPMAST** (patient encounter master; PHI-bearing).
- **HAPTRFR** (transfer file with account and MRN PHI fields AFACCT, AFMRNO).

Because counters influence how many records from these PHI-bearing sources are processed, migration must preserve its exit semantics.

---

## Program: XFXCYMD

### Overview

RPGLE program `XFXCYMD` is a **Data Maintenance** component dedicated to calendar date validation.
It checks year, month, and day fields against hard-coded boundaries and derived month lengths before downstream modules accept them.
With six rules and an average confidence of 58%, it is the primary gatekeeper for date correctness in the HABADTE ecosystem.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Key rules from interpretations detail:

- **BR-008:** When `VDD` is greater than `DYS(VMM)`, branch to `EXIT` (confidence 0.68).
- **BR-003:** Year (`VYY`) less than 1800 → `EXIT` (0.56).
- **BR-004:** Year (`VYY`) greater than 2100 → `EXIT` (0.56).
- **BR-005:** Month (`VMM`) less than 01 → `EXIT` (0.56).
- **BR-006:** Month (`VMM`) greater than 12 → `EXIT` (0.56).

Together these rules define an allowable calendar range and prevent malformed dates from entering patient workflows.

### Related Approved Rules by Domain

Other DATA_MAINTENANCE rules provide complementary constraints:

- BR-001–BR-002 (counter exits) influence the iteration of date checks.
- BR-009–BR-012 (mapping constraints) ensure date-related layouts remain within bounds.
- BR-013–BR-016 (indicator shutoff) can stop table-driven date processing when disabled.

### Data Touched

XFXCYMD works primarily with in-memory date fields and does not directly read PHI-bearing files.
However, it is called from HABADTE, which operates over data from:

- **OMPMAST** – patient encounters (MMMRNO, MMNAME, MMACCT, MMPSSN, MMMMRN).
- **HAPTRFR** – transfer records (AFACCT, AFMRNO PHI fields).

Therefore, its validation logic must be preserved exactly to maintain reporting and billing integrity over historical and future dates.

---

## Program: XFXLDSC

### Overview

`XFXLDSC` is a **Data Maintenance** RPGLE program that manages layout descriptors associated with hierarchy levels.
It uses mapping codes (`LDAMAP`) to drive lookup and validation of level-based structures from HXPLVL1–HXPLVL6.
With four rules and an average confidence of 56%, the program enforces strict ranges on mapping identifiers.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Key rules:

- **BR-009, BR-010, BR-011:** When `LDAMAP` > 99 → `EXIT` (confidence 0.56 each).
- **BR-012:** When `LDAMAP` > 9999 → `EXIT` (0.56).

These rules ensure mapping codes stay within supported bounds, preventing invalid hierarchy references from propagating.

### Related Approved Rules by Domain

XFXLDSC participates with:

- XFXCYMD’s date rules (BR-003–BR-008) to ensure that both layout and dates are valid.
- XFXTABL’s indicator rules (BR-013–BR-016), which may disable layout-driven table processing altogether.

### Data Touched

From the dependency and lineage information:

- Declares and reads **HXPLVL1–HXPLVL6** physical files (hierarchy levels).
- Reads **HXFLVL1–HXFLVL6** formats for level data.

These files do not expose explicit PHI in the summarized schema but are tightly coupled with patient-facing structures via level keys.
Preserving XFXLDSC’s mapping rules is essential to maintain valid hierarchical relationships in the migrated schema.

---

## Program: XFXTABL

### Overview

`XFXTABL` is a **Data Maintenance** RPGLE program responsible for table-based configuration and control.
It uses the indicator `*IN79` as a global flag to determine whether certain table operations should run or terminate.
Four high-confidence rules make it a critical safety valve for the batch.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Key rules:

- **BR-013–BR-016:** When `*IN79` equals on/active, branch to `EXIT` (confidence 0.90).

Regardless of the internal path, the presence of `*IN79` set to ON stops table processing, acting as a kill-switch for the affected batch operations.

### Related Approved Rules by Domain

XFXTABL interacts conceptually with:

- XFXCNTR (BR-001–BR-002) – counters may be moot when `*IN79` disables processing.
- XFXCYMD and XFXLDSC – their validations only apply if table operations are allowed.

### Data Touched

Per data lineage and dependencies:

- Declares **HXPTABLD**, **HXLTABLD**, **HXLTABLP**, **HXLTABLS** (table-related DDS files).
- Reads **XFFTABLD**, **XFFTABL2**, **XFFTABL3**, **XFFTABL4** logical and physical table formats.

These tables are reference data rather than direct PHI; however, they affect how patient and benefit data are interpreted by HABADTE and related programs.

---

## Program: HABADTE

### Overview

`HABADTE` is an **RPGLE** program in the **Patient Management** domain.
It is the central engine of this project, orchestrating patient encounter selection, filtering, enrichment, and output across multiple AS400 files and helper programs.
According to interpretations detail, HABADTE contains three primary rules with an average confidence of 97%, and has high cyclomatic complexity (CC 152, HIGH risk band).

### Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT
- **Type:** RPGLE

### Business Rules Applied

From interpretations detail:

- **BR-018:** When `-FLAG INDICATOR` equals void/voided, branch to `SKIP` (confidence 0.99).
- **BR-019:** When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP` (0.99).
- **BR-017:** When `-FILE INDICATOR` equals zero, branch to `SKIP` (0.92).

These rules filter out invalid, voided, or outpatient records, ensuring that HABADTE processes only valid inpatient encounters.

### Related Approved Rules by Domain

Although HABADTE’s domain is PATIENT_MANAGEMENT, it relies heavily on DATA_MAINTENANCE rules:

- Counter control (BR-001–BR-002) affects batch iteration.
- Date validation (BR-003–BR-008) guarantees temporal correctness for encounters.
- Mapping constraints (BR-009–BR-012) control hierarchy resolution.
- Table indicator rules (BR-013–BR-016) can globally disable supporting table operations.

Together, these rules create a coherent filter-and-enrich pipeline for inpatient data.

### Data Touched

From summarized schema, dependencies, and lineage:

- **OMPMAST (HMFMAST)** – patient master; PHI fields: MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN).
- **HAPTRFR (HAFTRFR)** – transfer records; PHI fields: AFACCT (AccountNumber), AFMRNO (MRN).
- **HXPDICT (HXFDICT)** – large dictionary table with multiple PHI fields (CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME).
- **OAPIRNK (HBFIRNK)** – break/rank file with PHI field BRKMRN.
- **OXPBNFIT (XFFBNFIT)** – benefit plan with PHI field XFBTEL.
- **HMLMAST5H**, **HXPBNFIT**, **HXPNSTN**, **HAPIRNK**, **HXPXMLD/HXPXMLR**, and the gap file `****HXPXML` and `PRINTER` are also involved via declarations and reads/writes.

HABADTE reads these sources, applies inclusion/exclusion rules, enriches with benefits and status, and writes to **HXFXMLD/HXFXMLH** XML-related files for downstream consumption.
All interactions with PHI fields must be replicated in the migrated solution with appropriate access controls and auditing.

---

## Cross-Program Narrative: End-to-End HABADTE Flow

1. **Initialization:** HABADTE starts by declaring key DDS and logical files (HMLMAST5H, HAPTRFR, HXPBNFIT, HAPIRNK, HXPNSTN, HXPXMLD, PRINTER) and sets up counters (via XFXCNTR) and control indicators.
2. **Configuration Check:** XFXTABL is consulted through `*IN79`. If table configuration indicates processing should be disabled, HABADTE exits early, preserving the legacy kill-switch behavior.
3. **Date and Mapping Validation:** XFXCYMD validates calendar fields; XFXLDSC validates mapping (`LDAMAP`) and hierarchy relationships against HXPLVL*.
4. **Patient Selection:** HABADTE applies BR-017–BR-019 to filter records by file indicator, void status, and inpatient/outpatient flag, focusing solely on non-voided inpatient encounters.
5. **Enrichment:** Benefit and status data are joined from OXPBNFIT/HXPBNFIT and HXPNSTN, while dictionary and rank information may be referenced from HXPDICT and OAPIRNK.
6. **Output Assembly:** Valid encounters are serialized via HXFXMLD/HXFXMLH into XML-like structures for downstream systems, with transfer events and hierarchy information attached.

This narrative should guide the design of the Spring Boot service and database model, ensuring that each step in the legacy flow has a clear modern counterpart.
