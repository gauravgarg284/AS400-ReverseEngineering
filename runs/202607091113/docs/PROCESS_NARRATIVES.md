# Process Narratives – HABADTE Project

This document provides business-focused narratives for each significant program in the HABADTE codebase. Each narrative summarizes the program’s purpose, domain, type, applied business rules, and data touchpoints.

---

## Program: HABADTE

**Overview**  
HABADTE is a high-complexity RPGLE batch program in the PATIENT_MANAGEMENT domain. It orchestrates the end-to-end processing of transfer records, evaluating which transfers should be exported, enriching them with level, benefit, and station information, and generating XML header and detail records. It relies heavily on reusable utilities for date validation, level lookups, table-driven mappings, identifier management, and MRN roll operations.

**Domain:** PATIENT_MANAGEMENT  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

From interpretations_detail and approved_rules:

- BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'.
- BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

These rules ensure that only active, inpatient transfer records are included in the outbound XML feed.

**Related Approved Rules (Domain PATIENT_MANAGEMENT)**

- BR-017 – File indicator skip rule.
- BR-018 – Voided transfer skip rule.
- BR-019 – Outpatient transfer skip rule.

**Data Touched**

Using data_dict_schema and lineage:

- **HAPTRFR** – Primary PF for transfer records (PHI: AFACCT, AFMRNO).  
- **HXPLVL1–HXPLVL6** – Level master data used indirectly via XFXLDSC.  
- **HXPTABLD / HXLTABLD / HXLTABLP / HXLTABLS** – Table dictionary and logical files accessed via XFXTABL.  
- **TXPBNFIT / HXPBNFIT** – Benefit plans (PHI: XFBTEL via OXPBNFIT).  
- **TXPNSTN / HXPNSTN** – Station definitions.  
- **HXPXMLD / HXFXMLD** – XML detail output.  
- **HXPXMLR / HXFXMLR** – XML identifier/header storage.  
- ******HXPXML** – Missing XML-related file referenced in gaps.  
- **PRINTER** – Missing printer file used for report output.

PHI-sensitive fields from HAPTRFR, HXPDICT, OAPIRNK, OMPMAST, and OXPBNFIT are indirectly handled through this process.

---

## Program: XFXCNTR

**Overview**  
XFXCNTR is an RPGLE utility in the DATA_MAINTENANCE domain responsible for field formatting and counter handling. It is called multiple times by HABADTE to validate or format 40-character fields, ensuring that blank or already-initialized values are handled correctly before further processing.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- BR-001 – Field formatting: exit if all blank (40 chars).  
- BR-002 – Field formatting: exit if first char non-blank.

These rules prevent redundant formatting operations and ensure that only appropriate input is processed, reducing the risk of misaligned fields in reports and XML payloads.

**Related Approved Rules (Domain DATA_MAINTENANCE)**

- BR-001, BR-002 – Field formatting.  
- General data maintenance rules (BR-003–BR-008, BR-009–BR-016, BR-020) may interact indirectly when XFXCNTR is combined with other utilities.

**Data Touched**

XFXCNTR focuses on in-memory fields rather than direct file I/O and does not have explicit file operations in the dependency graph. Its effect is visible in how formatted values are written by HABADTE and other callers.

---

## Program: XFXCYMD

**Overview**  
XFXCYMD is an RPGLE utility in the DATA_MAINTENANCE domain dedicated to date validation and normalization. It validates year, month, and day components, enforcing historical and future bounds and correct day counts per month. It delegates leap-year-specific logic to XFXLEAP.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- BR-003 – Reject year < 1800 (historical minimum).  
- BR-004 – Reject year > 2100 (forecast maximum).  
- BR-005 – Reject month < 01 (calendar constraint).  
- BR-006 – Reject month > 12 (calendar constraint).  
- BR-007 – Reject day < 01 (calendar constraint).  
- BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT'.

These rules collectively ensure that only valid calendar dates pass through. HABADTE uses XFXCYMD to make sure transfer dates do not introduce invalid or future-dated anomalies.

**Related Approved Rules (Domain DATA_MAINTENANCE)**

- BR-003–BR-008 – Date range and calendar integrity rules.

**Data Touched**

XFXCYMD operates on date fields passed from callers (e.g., HAPTRFR date fields via HABADTE) and calls **XFXLEAP** to compute month-day maxima for leap years. It has no direct file I/O in the dependency graph.

---

## Program: XFXLDSC

**Overview**  
XFXLDSC is an RPGLE program in the DATA_MAINTENANCE domain that performs level description lookups. It reads from a suite of level master files (HXPLVL1–HXPLVL6) to translate numeric level codes into descriptive attributes that feed reports and XML structures.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- BR-009 – Level lookup: reject if level code exceeds valid range.  
- BR-010 – Level lookup: reject if level code exceeds valid range.  
- BR-011 – Level lookup: reject if level code exceeds valid range.  
- BR-012 – Level lookup: reject if level code exceeds valid range.

These rules ensure that level codes used to segment patients or accounts fall within configured ranges and correspond to valid entries in the level master files.

**Related Approved Rules (Domain DATA_MAINTENANCE)**

- BR-009–BR-012 – Level code validation rules.

**Data Touched**

From data_dict_schema and lineage:

- **HXPLVL1–HXPLVL6** – PFs defining level metadata.  
- **HXFLVL1–HXFLVL6** – Read operations from logical or physical formats, as indicated by dependency edges.

XFXLDSC is a key part of the enrichment pipeline used by HABADTE.

---

## Program: XFXTABL

**Overview**  
XFXTABL is an RPGLE table lookup utility in the DATA_MAINTENANCE domain. It centralizes table-driven behavior, reading code dictionaries and mapping values to descriptions or control flags. HABADTE calls XFXTABL to resolve status codes and configuration values that influence export behavior.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- BR-013 – When *IN79 equals on/active, branch to 'EXIT'.  
- BR-014 – When *IN79 equals on/active, branch to 'EXIT'.  
- BR-015 – When *IN79 equals on/active, branch to 'EXIT'.  
- BR-016 – When *IN79 equals on/active, branch to 'EXIT'.

These rules drive an indicator-based early-exit strategy for table lookups, allowing XFXTABL to stop processing when certain configuration conditions are met.

**Related Approved Rules (Domain DATA_MAINTENANCE)**

- BR-013–BR-016 – Indicator-driven early exits.

**Data Touched**

From data_dict_schema and lineage:

- **HXPTABLD** – PF representing the core table dictionary.  
- **HXLTABLD, HXLTABLP, HXLTABLS** – LFs providing different access paths to HXPTABLD.  
- **XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4** – Underlying table formats read by XFXTABL.

---

## Program: XFXGETID

**Overview**  
XFXGETID is an RPGLE utility in the DATA_MAINTENANCE domain that manages XML identifiers. It reads from HXFXMLR to retrieve or generate identifiers used in HXFXMLD and HXFXMLH, ensuring that each XML message has a consistent user, sequence, and ID.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- No explicit key_rules listed in interpretations_detail, but the program is responsible for ID assignment and collision avoidance.

**Related Approved Rules (Domain DATA_MAINTENANCE)**

- BR-020 – Application profile table access (through HXXAPPPRF) is conceptually related to configuration but not directly in XFXGETID.

**Data Touched**

- **HXPXMLR / HXFXMLR** – PF storing XML request/identifier records.  
- Copybook **HXXLDA** – Declared both in XFXGETID and HABADTE, providing shared data area definitions.

---

## Program: XFXLEAP

**Overview**  
XFXLEAP is a minimal RPGLE program in the DATA_MAINTENANCE domain that determines leap-year characteristics. It is called by XFXCYMD to compute the number of days in a particular month and year.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- No explicit rules listed, but its logic underpins BR-008 by providing accurate day counts for each month, especially February.

**Data Touched**

XFXLEAP operates entirely on in-memory values and has no direct file I/O.

---

## Program: XFXMRNROL

**Overview**  
XFXMRNROL is an RPGLE program in the DATA_MAINTENANCE domain responsible for MRN (Medical Record Number) roll operations. It orchestrates calls to HXHAPPPRF (missing program) and HXXAPPPRF to apply application-specific profile rules when rolling or reassigning MRNs.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- No explicit key_rules captured, but MRN roll logic is implied by its name and call targets.

**Related Approved Rules (Domain DATA_MAINTENANCE)**

- BR-020 – SQL program accesses table 'HXPAPPPRF', relevant via HXXAPPPRF.

**Data Touched**

- **HXHAPPPRF** – Missing program, likely encapsulating application-specific profile logic.  
- **HXXAPPPRF** – SQLRPGLE program providing DB-backed profiles.  
- PHI-containing PFs (HXPDICT, OMPMAST) are indirectly affected when MRNs are rolled.

---

## Program: HXXAPPPRF

**Overview**  
HXXAPPPRF is an SQLRPGLE program in the DATA_MAINTENANCE domain that accesses the HXPAPPPRF table (and related application profile tables). It supplies configuration and profile information to programs such as XFXMRNROL, influencing how MRNs and other identifiers are managed.

**Domain:** DATA_MAINTENANCE  
**Program Type:** SQLRPGLE

**Business Rules Applied (Key Rules)**

- BR-020 – SQL program accesses table 'HXPAPPPRF'.  
- Additional rules (BR-021, BR-022) referenced in interpretations_detail indicate access to HXPAPPL6 and repeated HXPAPPPRF access.

**Data Touched**

- **HXPAPPPRF** – Application profile table.  
- **HXPAPPL6** – Level-specific application profile table (from interpretations).  
- Copybooks **HXXCNTRL** and **HXXAPPPRFP** – Provide control and profile structures.

---

## Program: HXXLDA

**Overview**  
HXXLDA is an RPGLE program representing a load-area or data-area definition used by both HABADTE and XFXGETID. It provides shared structures for XML identifiers and other control fields.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- No explicit rules; its primary role is structural.

**Data Touched**

- Shared data area fields used in XML identifier management and batch control.

---

## Program: HXXLEVEL

**Overview**  
HXXLEVEL is an RPGLE program in the DATA_MAINTENANCE domain that complements level-based processing, likely providing helper routines or transformations for level codes used by XFXLDSC and HABADTE.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- No explicit rules listed; it supports level processing infrastructure.

**Data Touched**

- Level-related data structures, consistent with HXPLVLx PFs.

---

## Program: HXXXML

**Overview**  
HXXXML is an RPGLE program in the DATA_MAINTENANCE domain that supports XML formatting or header/footer routines. It is copied into HABADTE and likely defines XML structures and default segments.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

**Business Rules Applied (Key Rules)**

- No explicit rules captured; behavior inferred from its role as an XML helper.

**Data Touched**

- XML layout and metadata fields that appear in HXFXMLH/HXFXMLD.

---

## Program: CXXXMLP

**Overview**  
CXXXMLP is an SQLRPGLE program that participates in XML processing for HABADTE. It is copied into HABADTE and, together with the missing CXXXMLC copybook, likely encapsulates SQL-based XML preparation logic.

**Domain:** DATA_MAINTENANCE  
**Program Type:** SQLRPGLE

**Business Rules Applied (Key Rules)**

- Not explicitly captured, but implied to manage XML-related queries and transformations.

**Data Touched**

- XML-related PFs such as HXPXMLD/HXPXMLR, as well as configuration tables.

---

## Program: HAPIRNK / TAPIRNK

**Overview**  
HAPIRNK is a logical file over TAPIRNK that provides ranked or sequenced access paths for rank-related data. HABADTE declares HAPIRNK as part of its data lineage, using it to cross-reference transfer records with ranking or ordering information.

**Domain:** DATA_MAINTENANCE (data structure)  
**Program Type:** DDS_LF over DDS_PF

**Business Rules Applied (Key Rules)**

- No explicit rules; behavior is derived from its usage in HABADTE.

**Data Touched**

- **TAPIRNK** – PF with 43 fields used to store rank/sequence data.  
- PHI: not directly flagged; however, rank records may reference account or MRN fields stored elsewhere.

---

## Program: HMLMAST5H / TMPMAST

**Overview**  
HMLMAST5H is a logical file over TMPMAST that provides a specific access path to patient or member master data, keyed by plan/state and admission date/time. HABADTE declares HMLMAST5H in its lineage, suggesting it may use this view for certain master-data checks or enrichment.

**Domain:** PATIENT_MANAGEMENT (data structure)  
**Program Type:** DDS_LF over DDS_PF

**Business Rules Applied (Key Rules)**

- No explicit rules; it enables ordering and filtering of TMPMAST records.

**Data Touched**

- **TMPMAST** – PF with 181 fields; likely a test or staging equivalent of OMPMAST (which contains PHI fields like MMACCT, MMMRNO, MMNAME, MMPSSN).  
- PHI fields in corresponding production PFs (**OMPMAST**) require careful handling in modernized code.

---

## Program: HXLTABLD / HXLTABLP / HXLTABLS

**Overview**  
These logical files provide alternative keyed views over the HXPTABLD table dictionary, supporting different combinations of code and descriptor fields. They are used indirectly via XFXTABL to implement table-driven behavior in HABADTE and other programs.

**Domain:** DATA_MAINTENANCE (data structure)  
**Program Type:** DDS_LF over DDS_PF

**Business Rules Applied (Key Rules)**

- No standalone rules; they support rules BR-013–BR-016 implemented in XFXTABL.

**Data Touched**

- **HXPTABLD** – PF for table dictionary entries.  
- PHI: none flagged in HXPTABLD.

---

## Program: HXPBNFIT / TXPBNFIT

**Overview**  
HXPBNFIT is a logical file over TXPBNFIT providing access to benefit plan data keyed by UB number and plan. HABADTE declares HXPBNFIT as part of its lineage and uses it to enrich transfers with benefit information.

**Domain:** DATA_MAINTENANCE (benefits)  
**Program Type:** DDS_LF over DDS_PF

**Business Rules Applied (Key Rules)**

- No explicit rules, but benefit data influences eligibility and output content.

**Data Touched**

- **TXPBNFIT** – PF with 12 fields; production PF **OXPBNFIT** contains PHI (XFBTEL).  
- Benefit data may include contact information and coverage attributes that must be secured.

---

## Program: HXPNSTN / TXPNSTN

**Overview**  
HXPNSTN is a logical file over TXPNSTN providing keyed access to station or state records. HABADTE uses HXPNSTN to derive station-related attributes for transfers.

**Domain:** DATA_MAINTENANCE (station/state)  
**Program Type:** DDS_LF over DDS_PF

**Business Rules Applied (Key Rules)**

- No explicit rules recorded; station data contributes to routing and reporting.

**Data Touched**

- **TXPNSTN** – PF with 19 fields defining station/state attributes.  
- PHI: none flagged for TXPNSTN.

---

## Program: OAPIRNK / OMPMAST / OXPBNFIT / OXPNSTN

**Overview**  
These PFs form the production equivalents of TAPIRNK, TMPMAST, TXPBNFIT, and TXPNSTN. While they do not appear as direct inputs to HABADTE in the compact lineage, they store production-grade rank, master, benefit, and station data and contain multiple PHI fields.

**Domain:** PATIENT_MANAGEMENT / DATA_MAINTENANCE  
**Program Type:** DDS_PF

**Business Rules Applied (Key Rules)**

- None directly listed; they act as data sources for other programs.

**Data Touched (PHI Focus)**

- **OAPIRNK** – BRKMRN (MRN).  
- **OMPMAST** – MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN (MRN, account, name, SSN).  
- **OXPBNFIT** – XFBTEL (PhoneNumber).  
- **OXPNSTN** – Station attributes (non-PHI).

These PFs must be mapped carefully to SQL Server with strong access controls and auditing.

---

## Program: HXXCNTRL, HXXAPPPRFP, HXXCNTRL-Related Utilities

**Overview**  
These RPGLE support programs and copy members provide control structures and wrapper logic for application profiles and operational control data. They are included via COPY in HXXAPPPRF and related programs.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE / Copy Members

**Business Rules Applied (Key Rules)**

- No explicit rules; they support higher-level logic in HXXAPPPRF and XFXMRNROL.

**Data Touched**

- Control fields and profile metadata that determine runtime behavior of MRN roll and related operations.
