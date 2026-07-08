# PROCESS NARRATIVES – HABADTE Application

## XFXCNTR – Field Formatting Utility

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Contains 2 rule(s) with average confidence 43%. XFXCNTR centralizes field-formatting exits used by multiple flows to enforce basic data quality constraints on 40-character input fields.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

Key rules for XFXCNTR:

- **BR-001** – Field formatting: exit if all blank (40 chars) – confidence 0.47.
- **BR-002** – Field formatting: exit if first char non-blank – confidence 0.40.

XFXCNTR is invoked by HABADTE to decide whether free-text or code fields should be processed. All-blank fields cause an immediate exit, while a non-blank first character can signal special formatting behavior.

### Related Approved Rules

All rules from XFXCNTR appear in the approved_rules list:

- BR-001 (DATA_MAINTENANCE, source_program = XFXCNTR)
- BR-002 (DATA_MAINTENANCE, source_program = XFXCNTR)

### Data Touched

XFXCNTR operates primarily on in-memory fields passed from callers and does not directly read or write DDS files, based on the dependency graph. Data it influences includes:

- Patient-related text fields in HABADTE such as descriptions and comments.
- Code fields whose formatting affects lookups in dictionary tables (HXPTABLD and its logical files).

No PHI-bearing files (HAPTRFR, OXPBNFIT, OAPIRNK, OMPMAST, HXPDICT) are directly accessed by XFXCNTR; PHI exposure is controlled by its callers.

---

## XFXCYMD – Date Validation Service

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Contains 6 rule(s) with average confidence 58%. XFXCYMD is the centralized date validation routine used by HABADTE and other flows to ensure that all dates are calendar-valid and within reasonable ranges.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

Key rules for XFXCYMD:

- **BR-008** – When VDD is greater than DYS(VMM), branch to 'EXIT' – confidence 0.68.
- **BR-003** – Date validation: reject year < 1800 (historical minimum) – confidence 0.56.
- **BR-004** – Date validation: reject year > 2100 (forecast maximum) – confidence 0.56.
- **BR-005** – Date validation: reject month < 01 (calendar constraint) – confidence 0.56.
- **BR-006** – Date validation: reject month > 12 (calendar constraint) – confidence 0.56.

These rules collectively ensure that any dates passed into XFXCYMD are constrained to realistic calendar values. The VDD vs DYS(VMM) rule specifically protects against impossible day-of-month combinations.

### Related Approved Rules

- BR-003–BR-008 (DATA_MAINTENANCE, source_program = XFXCYMD)

### Data Touched

XFXCYMD works on date fields passed from callers (e.g., transfer dates from HAPTRFR) and does not directly access physical files. It influences how HABADTE and other programs interpret date values stored in:

- **HAPTRFR** – AFTRDT (transfer date)
- **HXPDICT** – WBDATE (date of birth)

PHI exposure occurs indirectly through these fields, which store patient-related dates; however, XFXCYMD itself does not read PHI files independently.

---

## XFXLDSC – Level Lookup Service

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Contains 4 rule(s) with average confidence 56%. XFXLDSC resolves hierarchical location codes, validating level keys and returning descriptive names for levels 1–6.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

Key rules for XFXLDSC:

- **BR-009–BR-012** – Level lookup: reject if level code exceeds valid range – confidence 0.56 each.

These rules ensure that configuration data in level tables is enforced consistently and that invalid level codes are rejected before they can drive reporting or routing decisions.

### Related Approved Rules

- BR-009, BR-010, BR-011, BR-012 (DATA_MAINTENANCE, source_program = XFXLDSC)

### Data Touched

XFXLDSC reads from level configuration files:

- **HXPLVL1–HXPLVL6** – hierarchy configuration (no PHI fields).
- It also reads **HXFLVL1–HXFLVL6** via dependency edges (logical or alternate file names for the same structures).

Since these files contain configuration only, there is no direct PHI exposure. However, the accuracy of level mappings affects how PHI-bearing patient records (e.g., in HAPTRFR and OMPMAST) are grouped and reported by location.

---

## XFXTABL – Dictionary and Table Lookup Service

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Contains 4 rule(s) with average confidence 90%. XFXTABL is responsible for dictionary-based lookups that translate codes into descriptions using a combination of HXPTABLD and dictionary tables.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

Key rules for XFXTABL:

- **BR-013–BR-016** – When *IN79 equals on/active, branch to 'EXIT' – confidence 0.90 each.

These rules use indicator *IN79 to control when table lookup logic should terminate early, preventing unnecessary processing when certain conditions are met.

### Related Approved Rules

- BR-013, BR-014, BR-015, BR-016 (DATA_MAINTENANCE, source_program = XFXTABL)

### Data Touched

XFXTABL reads from dictionary-related structures:

- **HXPTABLD** – physical dictionary table.
- **HXLTABLD, HXLTABLP, HXLTABLS** – logical views providing different key combinations and descriptions.
- **XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4** – dictionary tables referenced for additional lookups.

No PHI fields are flagged in these dictionary tables. XFXTABL’s outputs, however, are used by HABADTE to enrich transfer and patient records with human-readable labels.

---

## HABADTE – Patient Management Orchestrator

**Overview**

RPGLE program in domain `PATIENT_MANAGEMENT`. Contains 3 rule(s) with average confidence 97%. HABADTE is the central orchestrator for patient transfer processing, coordinating field formatting, date validation, level and dictionary lookups, and XML output generation.

**Domain:** PATIENT_MANAGEMENT  
**Program Type:** RPGLE

### Business Rules Applied

Key rules for HABADTE:

- **BR-018** – When -FLAG INDICATOR equals void/voided, branch to 'SKIP' – confidence 0.99.
- **BR-019** – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' – confidence 0.99.
- **BR-017** – When -FILE INDICATOR equals zero, branch to 'SKIP' – confidence 0.92.

These rules control which transfer records are eligible for processing and which are excluded, focusing on active, non-void, inpatient encounters only.

### Related Approved Rules

From the approved_rules list:

- BR-017, BR-018, BR-019 (PATIENT_MANAGEMENT, source_program = HABADTE)

### Data Touched

HABADTE directly interacts with several files, including PHI-bearing ones:

- **HAPTRFR (PF)** – patient transfer transactions; PHI fields AFACCT (AccountNumber) and AFMRNO (MRN).
- **HAPIRNK (LF over TAPIRNK)** – rank-related views into transfer tables.
- **HMLMAST5H (LF over TMPMAST)** – master patient data keyed by status and admit date/time.
- **HXPNSTN (LF over TXPNSTN)** – normalized status lookups.
- **HXPBNFIT (LF over TXPBNFIT)** – benefit plan data with contact phone (XFBTEL) flagged as PHI.
- **HXFXMLH/HXFXMLD (PFs)** – XML header and detail files for outbound messaging.
- **PRINTER (FILE)** – printer or spool file for report output.
- **"****HXPXML" (FILE)** – XML-related file used for special or temporary payloads.

PHI-flagged files accessed in the HABADTE path:

- HAPTRFR – AFACCT, AFMRNO.
- OMPMAST – MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN (via related flows and lookups through HMLMAST5H/TMPMAST).
- HXPDICT – CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME (used via dictionary and cross-reference flows).
- OXPBNFIT – XFBTEL.

HABADTE orchestrates these data sources to produce XML output, applying the skip rules to reduce PHI exposure to only relevant transfers.

---

## HXXAPPPRF – Application Profile SQL Program

**Overview**

SQLRPGLE program in domain `DATA_MAINTENANCE`. Contains 3 rule(s) with average confidence 65%. HXXAPPPRF manages application-level profile data accessed via SQL, supporting MRN rollover and configuration flows.

**Domain:** DATA_MAINTENANCE  
**Program Type:** SQLRPGLE

### Business Rules Applied

Key rules for HXXAPPPRF include:

- **BR-020** – SQL program accesses table 'HXPAPPPRF' – confidence 0.65.
- BR-021, BR-022 – additional access to related tables (HXPAPPL6, HXPAPPPRF) as noted in interpretations_detail.

These rules indicate that application profiles and level-related configurations are stored in SQL tables and must be queried correctly.

### Related Approved Rules

From approved_rules:

- BR-020 (DATA_MAINTENANCE, source_program = HXXAPPPRF)

### Data Touched

HXXAPPPRF interacts with SQL tables:

- **HXPAPPPRF** – application profiles (not flagged as PHI in aggregated context).
- **HXPAPPL6** – level-related profile table (also not flagged as PHI).

Although these tables are not explicitly PHI-tagged, their configuration affects flows that operate on PHI-bearing data, such as MRN rollover and access controls.

---

## XFXGETID – Identifier Service

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Cyclomatic complexity: 1. XFXGETID provides minimal logic to retrieve identifiers from XML record files.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

No specific BR-IDs are listed as key_rules for XFXGETID. Its behavior is defined structurally by data access rather than explicit business rules.

### Related Approved Rules

No direct approved_rules entries reference XFXGETID.

### Data Touched

XFXGETID reads from:

- **HXPXMLR/HXFXMLR** – XML record files used to resolve identifiers and maintain linkage between XML messages and underlying patient data.

No PHI fields are flagged in these files; they primarily contain XML metadata such as user, sequence, and identifier values.

---

## XFXLEAP – Leap-Year Helper

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Cyclomatic complexity: 1. XFXLEAP evaluates leap-year behavior and supports XFXCYMD’s date validation logic.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

No explicit BR-IDs are associated with XFXLEAP in the interpretations_detail. Its logic is tightly coupled to BR-008, ensuring that month-length calculations are correct when validating dates.

### Related Approved Rules

Indirectly supports BR-008 (XFXCYMD) by providing leap-year calculations.

### Data Touched

XFXLEAP works on numeric date fragments passed from XFXCYMD and does not access DDS files directly.

---

## XFXMRNROL – MRN Rollover Service

**Overview**

RPGLE program in domain `DATA_MAINTENANCE`. Calls programs HXHAPPPRF and HXXAPPPRF. Cyclomatic complexity: 1. XFXMRNROL coordinates Medical Record Number (MRN) rollover operations, ensuring that MRN changes propagate to application profile structures.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE

### Business Rules Applied

No key_rules are listed for XFXMRNROL in interpretations_detail; its primary behavior is orchestrating calls to downstream profile programs.

### Related Approved Rules

Indirectly associated with BR-020 (HXXAPPPRF’s SQL access to HXPAPPPRF) as part of MRN rollover flows.

### Data Touched

XFXMRNROL calls:

- **HXHAPPPRF** – missing program (MEDIUM impact) expected to handle application-level MRN updates.
- **HXXAPPPRF** – SQL profile program for MRN and application configuration.

MRN fields in PHI-bearing files (e.g., MMMRNO, MMMMRN in OMPMAST; CCMRNO in HXPDICT) are indirectly affected by MRN rollover operations, even though XFXMRNROL itself does not directly read DDS PHI tables.

---

## HXXAPPPRFP – Application Profile Copy Member

**Overview**

RPGLE program/copy member in subsystem `HXXA`. Cyclomatic complexity: 1. HXXAPPPRFP is included by HXXAPPPRF and defines reusable structures for application profile processing.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE (copy)

### Business Rules Applied

No explicit BR-IDs are associated with HXXAPPPRFP; it contributes structural fields and constants used by HXXAPPPRF.

### Data Touched

HXXAPPPRFP interacts with the same SQL tables as HXXAPPPRF via embedded structures and does not directly access DDS files.

---

## HXXCNTRL – Control Copy Member

**Overview**

RPGLE program/copy member in subsystem `HXX`. Cyclomatic complexity: 1. HXXCNTRL provides control blocks and configuration settings that HXXAPPPRF uses to steer profile queries and updates.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE (copy)

### Business Rules Applied

No explicit BR-IDs are associated with HXXCNTRL; it encapsulates control flags and parameters.

### Data Touched

Used by HXXAPPPRF to manage how profile data is accessed and updated in HXPAPPPRF.

---

## HXXLDA – Level Data Area Copy Member

**Overview**

RPGLE program in subsystem `HXXL`. Cyclomatic complexity: 1. HXXLDA is copied into HABADTE and XFXGETID, providing shared data area structures for location and level-related configuration.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE (copy)

### Business Rules Applied

No direct BR-IDs; HXXLDA’s fields underpin level and location handling in HABADTE and XFXGETID.

### Data Touched

Indirectly supports access to HXPLVL1–HXPLVL6 and other configuration files.

---

## HXXLEVEL – Level Configuration Helper

**Overview**

RPGLE program in subsystem `HXXL`. Cyclomatic complexity: 1. HXXLEVEL is copied into HABADTE and assists with deriving or validating level-related fields beyond what XFXLDSC provides.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE (copy)

### Business Rules Applied

No explicit BR-IDs; HXXLEVEL provides structural support for level configuration.

### Data Touched

Interacts indirectly with HXPLVL1–HXPLVL6 and level context used in HABADTE’s transfer processing.

---

## HXXXML – XML Layout Helper

**Overview**

RPGLE program in subsystem `HXX`. Cyclomatic complexity: 1. HXXXML is copied into HABADTE and defines common XML layout structures used when writing HXFXMLH and HXFXMLD.

**Domain:** DATA_MAINTENANCE  
**Program Type:** RPGLE (copy)

### Business Rules Applied

No specific BR-IDs; its role is structural, providing field layouts for XML header and detail records.

### Data Touched

Supports writing to:

- **HXFXMLH** – XML header file.
- **HXFXMLD** – XML detail file.

---

## CXXXMLP – XML Processing SQLRPGLE Program

**Overview**

SQLRPGLE program in `UNGROUPED` subsystem. Cyclomatic complexity: 1. CXXXMLP is copied into HABADTE and works alongside XML-related files to handle more complex XML processing or transformation.

**Domain:** DATA_MAINTENANCE  
**Program Type:** SQLRPGLE

### Business Rules Applied

No explicit BR-IDs are listed, but CXXXMLP cooperates with the missing copybook CXXXMLC (HIGH impact) to define XML structures and behavior.

### Data Touched

Likely interacts with:

- **HXFXMLH/D/R** – XML header/detail/record files.
- **"****HXPXML"** – special XML file referenced by HABADTE.

As the exact field structure of CXXXMLC is missing, detailed field-level behavior must be reconstructed during modernization.

---

## Data Lineage Summary

Across all programs:

- Configuration-driven flows originate from **HXPTABLD**, **HXPLVL1–6**, **TXPBNFIT**, **TXPNSTN** and their logical files, supporting lookups in XFXLDSC, XFXTABL, HXPBNFIT, and HXPNSTN.
- XML identifier and payload flows are mediated by **HXPXMLR/HXFXMLR**, **HXFXMLH/D**, and the special file **"****HXPXML"**, with XFXGETID, CXXXMLP, HXXXML, and HABADTE orchestrating the logic.
- PHI exposure paths are isolated to a subset of files (HAPTRFR, OMPMAST, OAPIRNK, OXPBNFIT, HXPDICT); business rules focus on filtering and correct handling rather than direct masking in RPG.

This narrative provides the business-level understanding of how each program participates in the HABADTE ecosystem, guiding the forward engineering team in mapping legacy flows to modern services while preserving functional behavior.
