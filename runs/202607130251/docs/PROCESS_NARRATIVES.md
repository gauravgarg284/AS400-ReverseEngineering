# Process Narratives – HABADTE Application

This document provides business‑oriented narratives for each interpreted program in the HABADTE subsystem. Narratives draw on `interpretations_detail`, approved rules, data dictionary schemas, and PHI registry information.

---

## Program: HABADTE

### Overview

HABADTE is an RPGLE program in the **PATIENT_MANAGEMENT** domain. It is the main inpatient census engine, orchestrating reads from transfer and status tables, applying strict inclusion and exclusion rules, enriching records with organisational and benefit data, and generating XML and printer output. The program has high cyclomatic complexity (152) and calls multiple utility programs to perform date validation, text formatting, organisational lookups, MRN roll‑over, and configuration retrieval.

### Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT
- **Type:** RPGLE

### Business Rules Applied

From `interpretations_detail["HABADTE"].key_rules`:

- **BR-018** – "Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census." (confidence 0.99)
- **BR-019** – "Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded." (confidence 0.99)
- **BR-017** – "Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census." (confidence 0.92)

These rules collectively define which accounts qualify as active inpatients at the census snapshot.

### Related Approved Rules (Same Domain)

In addition to its own rules, HABADTE depends on utilities whose rules affect the process:

- **Date validation (XFXCYMD)** – BR‑003 to BR‑008 ensure that census dates are valid and within supported ranges.
- **Organisational lookups (XFXLDSC)** – BR‑009 to BR‑012 ensure that invalid level codes do not produce misleading descriptions.
- **Table‑driven exits (XFXTABL)** – BR‑013 to BR‑016 provide configurable stop conditions.

### Data Touched

From `data_dict_schema` and PHI registry:

- **HAPTRFR** – transfer records with PHI fields `AFACCT` (AccountNumber) and `AFMRNO` (MRN). This is the primary census source.
- **HXPNSTN/TXPNSTN** – status and nursing station reference data.
- **HXPBNFIT/TXPBNFIT** – benefit plan data with PHI field `XFBTEL` (PhoneNumber).
- **HXPTABLD** – table‑driven configuration.
- **HXPLVL1–HXPLVL6** – organisational hierarchy.
- **HXPXMLD/HXPXMLR/HXFXMLH/HXFXMLD** – XML header and detail storage.
- **PRINTER** (missing) – spool or print file for human‑readable census reports.

Because HABADTE touches multiple PHI‑bearing files, it must be heavily audited and protected in any modern implementation.

---

## Program: XFXCNTR

### Overview

XFXCNTR is an RPGLE utility program in the **DATA_MAINTENANCE** domain. It implements text‑centering logic used by HABADTE and potentially other reporting routines. The narrative indicates that it contains two rules with an average confidence of 43%, focusing on skipping unnecessary centering operations.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Key rules:

- **BR-001** – "Text centering — skip processing if the entire input field is blank (nothing to center)." (confidence 0.47)
- **BR-002** – "Text centering — skip processing if the text already starts at the first position (already left-aligned)." (confidence 0.4)

These rules ensure that centering logic is applied only when necessary, preserving existing formatting and avoiding redundant processing.

### Related Approved Rules (Same Domain)

- XFXCYMD date rules (BR‑003–BR‑008) and XFXTABL table rules (BR‑013–BR‑016) are part of the same DATA_MAINTENANCE domain and are coordinated by HABADTE, but XFXCNTR itself deals exclusively with text formatting.

### Data Touched

XFXCNTR operates on in‑memory strings and does not directly read or write any DDS files. It therefore does not directly interact with PHI fields; however, it may format PHI‑containing strings (names, MRNs, account numbers) which must be handled carefully in user interfaces and output.

---

## Program: XFXCYMD

### Overview

XFXCYMD is an RPGLE program in the **DATA_MAINTENANCE** domain that performs date validation and normalisation. The narrative notes six rules with an average confidence of 58%. It computes valid date boundaries and uses XFXLEAP to determine the number of days in a given month.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

From `interpretations_detail["XFXCYMD"].key_rules` and approved rules:

- **BR-003** – Reject dates before 1800.
- **BR-004** – Reject dates beyond 2100.
- **BR-005** – Month must be January (01) or later.
- **BR-006** – Month must be December (12) or earlier.
- **BR-007** – Day must be 01 or later.
- **BR-008** – When `VDD` is greater than `DYS(VMM)`, branch to EXIT (overflow guard).

These rules protect all date computations in the system from invalid values, ensuring robust age calculations, scheduling logic, and historical analysis.

### Related Approved Rules (Same Domain)

- HABADTE relies on XFXCYMD to validate census dates before processing.
- XFXLEAP, though not containing explicit rules, supports BR‑008 by returning the maximum days per month.

### Data Touched

XFXCYMD operates primarily on numeric/date parameters rather than database tables. While it does not directly read PHI fields, it is indirectly applied to date fields in PHI‑bearing tables such as HAPTRFR and HXPDICT. Invalid dates in these tables will be caught by XFXCYMD’s validation logic.

---

## Program: XFXLDSC

### Overview

XFXLDSC is an RPGLE program in the **DATA_MAINTENANCE** domain that retrieves organisational level descriptions from HXPLVL1–HXPLVL6. It reads both the DDS PFs (HXPLVLx) and their record formats (HXFLVLx). The narrative indicates four rules with an average confidence of 56%, all focused on handling out‑of‑range level codes.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Key rules:

- **BR-009 / BR-010 / BR-011 / BR-012** – "Organizational level lookup — the level code is out of the valid range; return no description."

These rules ensure that invalid organisational codes do not produce misleading descriptions in reports or APIs. When a level code is out of range, XFXLDSC returns an empty description and allows calling programs to treat the situation as a configuration error or data issue.

### Related Approved Rules (Same Domain)

- XFXTABL’s table‑driven exit rules may influence whether certain organisational lookups proceed or terminate.

### Data Touched

From `data_dict_schema`:

- **HXPLVL1–HXPLVL6** – organisational level PFs with keys HX1NUM–HX6NUM.
- **HXFLVL1–HXFLVL6** – record formats read by XFXLDSC.

No PHI fields are present in these PFs, but their outputs are used to contextualise PHI‑bearing data (e.g., patient accounts by organisational level) in HABADTE’s output.

---

## Program: XFXTABL

### Overview

XFXTABL is an RPGLE program in the **DATA_MAINTENANCE** domain that implements table‑driven logic using HXPTABLD and related dictionary tables (XFFTABLD, XFFTABL2–4). It centralises configurable rules, flags, and mapping entries that drive behaviours in HABADTE and other programs.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

From `interpretations_detail["XFXTABL"].key_rules`:

- **BR-013 / BR-014 / BR-015 / BR-016** – "When *IN79 equals on/active, branch to 'EXIT'."

These indicators act as a configuration‑driven kill‑switch for certain table processing loops. When *IN79 is active, XFXTABL exits early, preventing further dictionary evaluation.

### Related Approved Rules (Same Domain)

- XFXCYMD and XFXLDSC may rely on table entries loaded or governed by XFXTABL.

### Data Touched

From `data_dict_schema` and dep_edges:

- **HXPTABLD** – PF with record format XFFTABLD, keys XFDTCD/XFDECD.
- **XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4** – additional dictionary tables read by XFXTABL.

These tables do not contain PHI fields but drive logic applied to PHI‑bearing entities.

---

## Program: HXXAPPPRF

### Overview

HXXAPPPRF is a SQLRPGLE program in the **DATA_MAINTENANCE** domain. It reads configuration and reference data from application profile tables (e.g., HXPAPPPRF, HXPAPPL6) and includes copybooks HXXCNTRL and HXXAPPPRFP. HABADTE and XFXMRNROL call HXXAPPPRF to obtain configuration settings affecting census and MRN handling.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** SQLRPGLE

### Business Rules Applied

From `interpretations_detail["HXXAPPPRF"].key_rules` and approved rules:

- **BR-020** – "Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed during processing." (confidence 0.65)

Additional non‑approved but documented rules mention HXPAPPL6 and repeated accesses to HXPAPPPRF. Overall, HXXAPPPRF encapsulates how configuration tables are read and interpreted.

### Related Approved Rules (Same Domain)

- HABADTE and XFXMRNROL depend on configuration values loaded by HXXAPPPRF to govern behaviour such as which level codes, statuses, or benefits are considered valid.

### Data Touched

While HXPAPPPRF and HXPAPPL6 are referenced, detailed PF metadata is not present in the compact schema. These tables are assumed to hold configuration (non‑PHI). However, misconfiguration can indirectly impact PHI workflows (e.g., exposing records from unintended organisational units).

---

## Program: XFXGETID

### Overview

XFXGETID is an RPGLE utility in the **DATA_MAINTENANCE** domain with cyclomatic complexity 1. It is used by HABADTE to manage XML record identifiers. The narrative notes that it copies HXXLDA and reads HXFXMLR, linking logical data areas to XML response records.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

No explicit key_rules are listed for XFXGETID, but its functional role is clear:
- Generate or retrieve stable XML IDs for census records.
- Ensure mapping between internal account identifiers and external XML message identifiers.

### Related Approved Rules

- HABADTE’s filtering rules and XFXTABL’s configuration rules determine which records XFXGETID operates on.

### Data Touched

From dep_edges and data lineage:

- **HXXLDA** – logical data area copybook containing layout/ID context.
- **HXFXMLR/HXPXMLR** – XML record storage PFs, with keys XMRUSR/XMRSEQ/XMRID.

These files do not directly contain PHI, but their IDs may correspond to PHI‑bearing records in other tables.

---

## Program: XFXLEAP

### Overview

XFXLEAP is a simple RPGLE helper in the **DATA_MAINTENANCE** domain. It calculates leap‑year and days‑in‑month information for use by XFXCYMD. Cyclomatic complexity is 1.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

No discrete business rules are listed, but functionally it enforces calendar correctness by computing legitimate day counts for each month/year combination.

### Data Touched

No database files are directly accessed. Inputs and outputs are numeric fields representing year and month, used by XFXCYMD in date validation.

---

## Program: XFXMRNROL

### Overview

XFXMRNROL is an RPGLE program in the **DATA_MAINTENANCE** domain that performs MRN roll‑over or translation. It calls HXHAPPPRF (missing) and HXXAPPPRF, indicating that MRN handling is tightly coupled with application profile logic. Cyclomatic complexity is 1, suggesting straightforward control flow.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

No explicit key_rules are present, but inferred behaviours include:
- Determining which MRN to use when multiple MRNs exist (e.g., MMMRNO vs MMMMRN in OMPMAST).
- Updating MRN references in configuration or patient tables according to profile rules.

### Related Approved Rules

- BR‑020 in HXXAPPPRF governs how configuration data is read, influencing MRN roll logic.

### Data Touched

While specific PFs are not listed in the compact schema for XFXMRNROL, its domain implies interaction with PHI fields in patient master tables (e.g., OMPMAST) and possibly HXPDICT.

---

## Program: HXXAPPPRFP

### Overview

HXXAPPPRFP is an RPGLE copy/program associated with HXXAPPPRF in the **DATA_MAINTENANCE** domain. It is included via a COPY relationship and provides layout or helper logic for application profile processing.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

No independent rules are listed, but HXXAPPPRFP contributes to the behaviour described in BR‑020 by defining record formats, constants, or subroutines.

### Data Touched

Uses the same configuration tables as HXXAPPPRF (HXPAPPPRF, HXPAPPL6), with no explicit PHI fields.

---

## Program: HXXCNTRL

### Overview

HXXCNTRL is an RPGLE program in the **DATA_MAINTENANCE** domain included by HXXAPPPRF via COPY. It likely encapsulates control structures, flags, and configuration toggles for application profile processing.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Not separately enumerated, but HXXCNTRL contributes to the configuration management pattern governed by BR‑020.

### Data Touched

Primarily in‑memory control fields or configuration metadata, not directly PHI.

---

## Program: HXXLDA

### Overview

HXXLDA is an RPGLE program in the **DATA_MAINTENANCE** domain that defines logical data area structures used by HABADTE and XFXGETID. It is copied into these programs to provide shared layouts.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

No explicit rules, but HXXLDA ensures consistency of data areas across modules, which is critical for MRN and XML ID handling.

### Data Touched

Logical data areas (LDAs) rather than PFs; these may carry IDs or configuration flags indirectly linked to PHI.

---

## Program: HXXLEVEL

### Overview

HXXLEVEL is an RPGLE helper program in the **DATA_MAINTENANCE** domain, copied into HABADTE. It likely defines constants or structures associated with level codes used in organisational hierarchy.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

No discrete rules; it supports organisational hierarchy processing in HABADTE.

### Data Touched

Level code fields, not PHI directly.

---

## Program: HXXXML

### Overview

HXXXML is an RPGLE program in the **DATA_MAINTENANCE** domain, copied into HABADTE to provide XML layout and helper routines. It underpins the construction of XML structures written to HXPXMLD/HXFXMLD.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE

### Business Rules Applied

Not individually enumerated, but HXXXML contributes to the XML enrichment step described in the HABADTE narrative.

### Data Touched

XML buffer fields and structural templates, not directly PHI; however, XML payloads will wrap PHI fields from HAPTRFR and other PFs.

---

## Program: CXXXMLP

### Overview

CXXXMLP is a SQLRPGLE program in the **DATA_MAINTENANCE** domain, copied into HABADTE. It likely implements parameterised XML processing or persistence routines.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** SQLRPGLE

### Business Rules Applied

No specific rules are listed, but its presence in HABADTE’s COPY list indicates it plays a role in building or managing XML payloads for census records.

### Data Touched

Potentially interacts with HXPXMLD/HXPXMLR tables and related XML structures.

---

## PHI Considerations Across Programs

Programs with direct PHI exposure via PFs:

- **HABADTE** – reads HAPTRFR (AFACCT, AFMRNO), and indirectly touches OMPMAST and HXPDICT through MRN and status/benefit workflows.
- **XFXMRNROL** – inferred to operate on MRNs in patient master records.

Utility and configuration programs (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, HXXAPPPRF, XFXGETID, HXX*) typically operate on non‑PHI or metadata but influence how PHI is processed and exposed.

Any modernization effort must:
- Implement strict access control and auditing around HABADTE and MRN‑related utilities.
- Ensure that text‑formatting and XML helpers do not inadvertently leak PHI beyond authorised contexts.
