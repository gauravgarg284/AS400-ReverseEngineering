# PROCESS NARRATIVES – HABADTE Run 202607021244

This document provides business-oriented narratives for each interpreted program in the HABADTE AS400 application, linking technical behaviour to business domains and highlighting applied rules and data usage.

---

## Program: XFXCNTR

### Overview

RPGLE program in domain "Data Maintenance". Contains 2 rule(s) with average confidence 43%. XFXCNTR acts as a control and counter routine used by other modules (e.g., HABADTE) to determine when to exit loops or stop processing batches.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

Key rules:

- **BR-001:** "When X equals zero, branch to 'EXIT'" (confidence 0.47)
- **BR-002:** "When X equals 40, branch to 'EXIT'" (confidence 0.4)

These rules ensure that:
- Processing does not continue when the counter indicates no work remaining (X=0).
- Iterations or items are capped at 40, protecting the system from running overly long batches or exceeding configured limits.

### Related Approved Rules (Same Domain)

From the DATA_MAINTENANCE domain:
- XFXCYMD date rules (BR-003–BR-008).
- XFXLDSC mapping rules (BR-009–BR-012).
- XFXTABL indicator rules (BR-013–BR-016).
- HXXAPPPRF profile rule (BR-020).

Together, these rules form a consistent data-maintenance framework: counters, dates, mappings, indicators, and profiles.

### Data Touched

XFXCNTR operates primarily on internal variables rather than files. It does not directly read PF/LF objects, but its decisions influence how other programs (especially HABADTE) process records from:
- **HAPTRFR** (patient transfers)
- **HXPLVL1–6** (level configuration)
- **HXPTABLD** and related logical files

No PHI-bearing fields are directly accessed by XFXCNTR.

---

## Program: XFXCYMD

### Overview

RPGLE program in domain "Data Maintenance". Contains 6 rule(s) with average confidence 58%. XFXCYMD is responsible for validating composite dates (year, month, day) used throughout the application.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

Key rules:

- **BR-008:** "When VDD is greater than DYS(VMM), branch to 'EXIT'" (confidence 0.68)
- **BR-003:** "When VYY is less than 1800, branch to 'EXIT'" (0.56)
- **BR-004:** "When VYY is greater than 2100, branch to 'EXIT'" (0.56)
- **BR-005:** "When VMM is less than 01, branch to 'EXIT'" (0.56)
- **BR-006:** "When VMM is greater than 12, branch to 'EXIT'" (0.56)
- **BR-007:** "When VDD is less than 01, branch to 'EXIT'" (0.56)

Business meaning:
- Enforces that all business processes operate on realistic calendar dates (1800–2100, months 1–12, valid days per month).
- Prevents erroneous ageing, scheduling, and reporting based on invalid dates.

### Related Approved Rules

Shared with the DATA_MAINTENANCE domain, XFXCYMD’s rules complement:
- Counter control in XFXCNTR (BR-001–BR-002).
- Level mapping limits in XFXLDSC (BR-009–BR-012).
- Table indicator exits in XFXTABL (BR-013–BR-016).

### Data Touched

XFXCYMD works on date components rather than table data. Its validation influences how HABADTE and other modules treat date fields in:
- **HAPTRFR** (AFTRDT, AFTRTM)
- **HXPDICT** (WBDATE)

Though these fields may contain PHI (e.g., date of birth), XFXCYMD itself applies generic validation without interpreting PHI semantics.

---

## Program: XFXLDSC

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 56%. XFXLDSC implements level description mapping, using configuration files to convert level codes into human-readable descriptions used in reports and XML outputs.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE service

### Business Rules Applied

Key rules:

- **BR-009, BR-010, BR-011:** "When LDAMAP is greater than 99, branch to 'EXIT'" (0.56 each)
- **BR-012:** "When LDAMAP is greater than 9999, branch to 'EXIT'" (0.56)

Business meaning:
- Mapping codes above configured thresholds are treated as invalid or unconfigured, preventing misleading level descriptions.
- Ensures that only supported hierarchy codes drive patient and benefit reporting.

### Related Approved Rules

XFXLDSC interacts conceptually with:
- XFXTABL (table mappings), which also enforces constraints via indicator *IN79.
- HABADTE, which depends on valid level descriptions to generate accurate XML.

### Data Touched

XFXLDSC declares and reads level configuration files:
- **HXPLVL1–HXPLVL6** (PFs with keys HX1NUM–HX6NUM).
- **HXFLVL1–HXFLVL6** (read operations captured in dep_edges).

No PHI fields are flagged in these level files. They act purely as configuration and do not directly expose patient identifiers.

---

## Program: XFXTABL

### Overview

RPGLE program in domain "Data Maintenance". Contains 4 rule(s) with average confidence 90%. XFXTABL is a table-driven control program that uses configuration indicators and mapping tables to influence processing paths.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE service

### Business Rules Applied

Key rules:

- **BR-013–BR-016:** "When *IN79 equals on/active, branch to 'EXIT'" (confidence 0.9 each)

Business meaning:
- Indicator *IN79 acts as a master switch, defined by table contents or context, that short-circuits processing when certain conditions are met.
- This supports feature toggles, exclusion scenarios, or safety stops based on configuration rather than hard-coded logic.

### Related Approved Rules

The table indicator rules align with:
- Counter-based exits in XFXCNTR.
- Level mapping bounds in XFXLDSC.

Together they provide a configuration-driven safety net for data maintenance workflows.

### Data Touched

XFXTABL works with:
- **HXPTABLD** (PF, record format XFFTABLD, keys XFDTCD/XFDECD).
- Logical files **HXLTABLD**, **HXLTABLP**, **HXLTABLS** providing alternate index paths.
- Additional table files: **XFFTABLD**, **XFFTABL2**, **XFFTABL3**, **XFFTABL4**.

No PHI fields are present in these table structures; they store codes and descriptions used to interpret other business data.

---

## Program: HABADTE

### Overview

RPGLE program in domain "Patient Management". Contains 3 rule(s) with average confidence 97%. HABADTE is the main orchestration module coordinating patient transfer processing, enrichment, and XML export. It has high cyclomatic complexity (152) and the highest dependency hotspot score (38), indicating its central role.

### Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT
- **Type:** RPGLE controller

### Business Rules Applied

Key rules:

- **BR-018:** "When -FLAG INDICATOR equals void/voided, branch to 'SKIP'" (0.99)
- **BR-019:** "When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'" (0.99)
- **BR-017:** "When -FILE INDICATOR equals zero, branch to 'SKIP'" (0.92)

Business meaning:
- Ensures that only active, non-voided inpatient transfers are included in processing.
- Aligns exported data with clinical and financial reality; voided or outpatient entries are excluded from certain reports.

### Related Approved Rules

HABADTE depends on and orchestrates other DATA_MAINTENANCE rules:
- Date validation (XFXCYMD: BR-003–BR-008).
- Level mapping (XFXLDSC: BR-009–BR-012).
- Table-driven control (XFXTABL: BR-013–BR-016).
- Counter control (XFXCNTR: BR-001–BR-002).

This makes HABADTE the integration point where patient-management logic meets configuration and control rules.

### Data Touched

HABADTE interacts with multiple files and interfaces:

- **HAPTRFR** (PF): patient transfer records. PHI fields include AFACCT (AccountNumber) and AFMRNO (MRN).
- **HMLMAST5H** (LF over TMPMAST): logical view on patient master data.
- **HXPBNFIT** (LF over TXPBNFIT): benefit-related data.
- **HAPIRNK** (LF over TAPIRNK): inquiry records.
- **HXPNSTN** (LF over TXPNSTN) and **XFFNSTN**: station/status data.
- **HXPXMLD / HXPXMLR / HXFXMLH / HXFXMLD**: XML detail, ID, and header output files.
- External or missing interfaces: ****HXPXML, PRINTER (medium-impact gaps) and copybook CXXXMLC (HIGH impact).

PHI-bearing files referenced include:
- **OMPMAST** (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN).
- **HXPDICT** (multiple MRN, account, phone, name, room fields).
- **OXPBNFIT** (XFBTEL phone number).
- **OAPIRNK** (BRKMRN MRN).

HABADTE must therefore be treated as a PHI-sensitive service in the modernised environment, with strong access controls and audit logging.

---

## Program: HXXAPPPRF

### Overview

SQLRPGLE program in domain "Data Maintenance". Contains 3 rule(s) with average confidence 65%. HXXAPPPRF is an application profile handler that uses SQL to read preference tables affecting behaviour in other modules (e.g., XFXMRNROL and HABADTE indirectly).

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** SQLRPGLE service

### Business Rules Applied

Key rules (from interpretations_detail):

- **BR-020:** "SQL program accesses table 'HXPAPPPRF'" (0.65)
- **BR-021:** SQL program accesses table "HXPAPPL6" (0.65)
- **BR-022:** SQL program accesses table 'HXPAPPPRF'" (0.65)

Business meaning:
- Retrieves application and user profile preferences that govern how data-maintenance routines behave.
- May control thresholds, flags, or feature toggles used by XFXMRNROL and HABADTE.

### Related Approved Rules

HXXAPPPRF is called from XFXMRNROL, which sits on the boundary between patient management and data-maintenance logic. Its rules affect MRN roll behaviour and possibly patient identity handling.

### Data Touched

HXXAPPPRF accesses profile tables:
- **HXPAPPPRF** – application profile records.
- **HXPAPPL6** – level-specific profile data.

These tables are not flagged as PHI-bearing but may reference patient or account identifiers indirectly. Care should be taken to treat profile data as sensitive configuration.

---

## Program: XFXGETID

### Overview

RPGLE program in domain "Data Maintenance". Cyclomatic complexity: 1. XFXGETID is a simple ID retrieval routine that resolves XML-related IDs used by HABADTE when writing XML output.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

No explicit business rules are listed in `key_rules` for XFXGETID, but its function is implied by data lineage.

Business meaning:
- Retrieves or computes XML sequence/identity values for the current user or processing context.
- Ensures XML outputs have unique and traceable identifiers.

### Related Approved Rules

XFXGETID supports HABADTE’s patient-management rules by enabling XML export but does not itself enforce inclusion/exclusion conditions.

### Data Touched

- Declares **HXPXMLR** and reads **HXFXMLR** (READ edge).
- Contributes IDs used in **HXFXMLH** and **HXFXMLD** written by HABADTE.

No PHI fields are directly flagged in these XML control files, but they can carry identifiers that reference PHI-bearing records indirectly.

---

## Program: XFXLEAP

### Overview

RPGLE program in domain "Data Maintenance". Cyclomatic complexity: 1. XFXLEAP is a leap-year utility called by XFXCYMD.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE utility

### Business Rules Applied

No explicit rules are listed, but the program likely implements leap-year logic used by XFXCYMD when evaluating DYS(VMM).

Business meaning:
- Ensures that February and other months have correct day counts depending on the year, which is critical for accurate scheduling and reporting.

### Related Approved Rules

Used indirectly by XFXCYMD’s BR-008 (day > DYS(VMM)).

### Data Touched

Works purely on date components; no direct file IO or PHI exposure.

---

## Program: XFXMRNROL

### Overview

RPGLE program in domain "Data Maintenance". Calls programs HXHAPPPRF and HXXAPPPRF. Cyclomatic complexity: 1. XFXMRNROL appears to manage MRN roll or reassignment logic, coordinating with profile handlers.

### Domain and Program Type

- **Domain:** DATA_MAINTENANCE
- **Type:** RPGLE integration module

### Business Rules Applied

No key_rules are explicitly listed, but behaviour is inferred from its calls:
- Uses **HXHAPPPRF** (missing program) and **HXXAPPPRF** (SQL profile handler) to determine how MRNs or patient identifiers should be handled under various profiles.

Business meaning:
- Mediates between application profile rules and patient identity management.
- May control when MRNs are rolled, reassigned, or flagged.

### Related Approved Rules

Relies on HXXAPPPRF’s profile rules (BR-020–BR-022). Its outcomes influence HABADTE’s patient-management process.

### Data Touched

Data access is indirect via called programs; XFXMRNROL itself is not recorded as reading PHI-bearing files, but it affects MRN handling semantics.

---

## Cross-Program Narrative: HABADTE Workflow

Bringing together the above programs:

1. **HABADTE** pulls transfer records from HAPTRFR, applies BR-017–BR-019, and orchestrates enrichment and XML export.
2. **XFXCYMD** and **XFXLEAP** ensure that all date fields used in this process are valid within 1800–2100 and respect days-per-month rules.
3. **XFXLDSC** and **HXPLVL*/HXFLVL*** provide level hierarchy descriptions, allowing transfers to be contextualised within organisational or plan structures.
4. **XFXTABL** and **HXPTABLD/HXLTABL*** supply table-driven mappings and configuration indicators that can short-circuit processing when required.
5. **XFXCNTR** enforces loop bounds and processing limits.
6. **XFXMRNROL** and **HXXAPPPRF** integrate profile rules that may affect MRN and patient identity handling.
7. **XFXGETID** and XML files (**HXPXMLR/HXPXMLD/HXFXMLH/HXFXMLD**) enable the final XML export used by downstream systems.

PHI-sensitive data is concentrated in patient and benefits files (HAPTRFR, HXPDICT, OMPMAST, OXPBNFIT, OAPIRNK). All modernised services built from these narratives must apply strict access control, masking, and auditing in line with regulatory requirements.
