# Low-Level Design (LLD) – HABADTE AS400 Application

## 1. Architecture Overview

### 1.1 Technology Stack and Member Types

The HABADTE application runs on IBM i / AS400 using classic RPG and DDS-based data definitions. The aggregated inventory shows the following member type distribution:

- **DDS_LF (7 members)** – Logical files providing alternate key paths and projections over physical files (e.g., HAPIRNK, HMLMAST5H, HXLTABLD/LP/LS, HXPBNFIT, HXPNSTN).
- **DDS_PF (15 members)** – Physical files storing core business data for transfers, patient masters, dictionary/code tables, level hierarchies, XML metadata, benefits, and institutions (e.g., HAPTRFR, HXPDICT, HXPLVL1–6, HXPTABLD, HXPXMLD/R, OAPIRNK, OMPMAST, OXPBNFIT, OXPNSTN).
- **RPGLE (13 members)** – ILE RPG programs implementing business logic, validation routines, lookups, and utilities (e.g., HABADTE, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML).
- **SQLRPGLE (2 members)** – RPG programs with embedded SQL, primarily used for application profile and XML integration (HXXAPPPRF, CXXXMLP).

### 1.2 Call Graph Summary

At the center of the architecture is **HABADTE (RPGLE)**, a high-complexity patient management driver (cyclomatic complexity 152, HIGH band). Its primary interactions include:

- Calls to utility/validation programs:
  - **XFXMRNROL** for MRN rollover processing.
  - **XFXCNTR** multiple times for counter/control logic.
  - **XFXLDSC** for level description resolution.
  - **XFXCYMD** for calendar date validation.
  - **XFXGETID** for ID retrieval based on XML header records.
  - **XFXTABL** for table/code lookups.

- Copy/include dependencies (configuration and XML integration):
  - **HXXLDA** – logical data area configuration.
  - **HXXLEVEL** – level/privilege metadata.
  - **HXXXML** – XML configuration routines.
  - **CXXXMLP** – XML-related SQLRPGLE copy.
  - **CXXXMLC** – missing copybook for XML, referenced but not present.

File-level dependencies include reads/writes to patient transfer PFs, institution codes, and XML header/detail work files. Lower-level utilities (XFXCYMD, XFXLEAP, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL) form a service layer around HABADTE, and DDS logical files such as HAPIRNK and HMLMAST5H model alternate access paths to transfer and master data.

Design patterns observed:

- **Monolithic orchestrator with shared utilities:** HABADTE centralizes workflow steps while delegating specialized tasks (date validation, table lookup, MRN rollover) to dedicated programs.
- **Table-driven configuration:** HXPTABLD and its logical variants (HXLTABLD/LP/LS) hold configuration or code tables. XFXTABL provides a canonical access path to these tables.
- **Hierarchical level modeling:** HXPLVL1–6 PFs capture level hierarchies, with XFXLDSC reading and interpreting them for display or rule evaluation.
- **XML-based integration boundary:** HXPXMLD/R and HXFXML* files, plus CXXXMLP/CXXXMLC and HXXXML, form a thin XML layer for outbound or persisted messages.

## 2. Database Schema

### 2.1 Physical Files (PF)

Each PF has a record format, unique key properties, and PHI-sensitive fields.

#### HAPTRFR (PF)
- **Record format:** HAFTRFR
- **Unique key:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

HAPTRFR appears to be a patient transfer file keyed by level, account, transfer date/time, and type. The presence of MRN and account number makes it a PHI-bearing table that must be carefully controlled.

#### HXPDICT (PF)
- **Record format:** HXFDICT
- **Unique:** false
- **Total fields:** 2705
- **PHI fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HXPDICT is a large dictionary/reference file containing a broad set of PHI attributes: multiple MRN fields, account and room numbers, phone numbers, names, and dates of birth. It likely centralizes patient master/encounter metadata.

#### HXPLVL1–HXPLVL6 (PFs)

- **HXPLVL1 (HXFLVL1)** – key: HX1NUM, unique, 36 fields, no PHI fields.
- **HXPLVL2 (HXFLVL2)** – key: HX2NUM, unique, 39 fields, no PHI.
- **HXPLVL3 (HXFLVL3)** – key: HX3NUM, unique, 39 fields, no PHI.
- **HXPLVL4 (HXFLVL4)** – key: HX4NUM, unique, 39 fields, no PHI.
- **HXPLVL5 (HXFLVL5)** – key: HX5NUM, unique, 42 fields, no PHI.
- **HXPLVL6 (HXFLVL6)** – key: HX6NUM, unique, 155 fields, no PHI.

These PFs store multi-level configuration or classification data (such as product levels, institution hierarchies, or status ladders). XFXLDSC reads these levels for description mapping.

#### HXPTABLD (PF)
- **Record format:** XFFTABLD
- **Unique:** false
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7

HXPTABLD stores table codes and descriptions. It is the base PF for multiple LFs (HXLTABLD/LP/LS), enabling different keyed views over the same dictionary.

#### HXPXMLD and HXPXMLR (PFs)

- **HXPXMLD (HXFXMLD)** – unique key: XMDUSR, XMDSEQ, XMDSQ2; total fields: 4.
- **HXPXMLR (HXFXMLR)** – unique key: XMRUSR, XMRSEQ, XMRID; total fields: 4.

These are XML-related detail and header files keyed by user and sequence identifiers. XFXGETID interacts with HXFXMLR to read XML header records when assigning identifiers.

#### OAPIRNK (PF)
- **Record format:** HBFIRNK
- **Unique key:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI fields:** BRKMRN (MRN)

OAPIRNK appears to contain break or rank information keyed on level, account, and sequence, with MRN as a sensitive field.

#### OMPMAST (PF)
- **Record format:** HMFMAST
- **Unique key:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI fields:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN

OMPMAST is a core patient master-like file, holding multiple identifiers, names, account numbers, and social security numbers. It is a high-sensitivity file.

#### OXPBNFIT (PF)
- **Record format:** XFFBNFIT
- **Unique key:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI fields:** XFBTEL (PhoneNumber)

OXPBNFIT contains benefit information keyed by benefit and plan codes, with phone numbers representing contact or provider details.

#### OXPNSTN (PF)
- **Record format:** XFFNSTN
- **Unique key:** XFNLV6, XFNSST
- **Total fields:** 23

OXPNSTN likely models institution status or location attributes without direct PHI.

### 2.2 Logical Files (LF)

#### HAPIRNK (LF)
- **Base PF:** TAPIRNK (missing in source set)
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/omit:** none

HAPIRNK provides a logical index over TAPIRNK for transfer ranking or break records. TAPIRNK must be obtained from the host system during modernization.

#### HMLMAST5H (LF)
- **Base PF:** TMPMAST (missing)
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM

HMLMAST5H exposes patient master data keyed by posting state and add date/time. It is a historical or audit-oriented view.

#### HXLTABLD / HXLTABLP / HXLTABLS (LFs)
- **Base PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:**
  - HXLTABLD: XFDTCD, XFDMAP
  - HXLTABLP: XFDTCD, XFDLDS
  - HXLTABLS: XFDTCD, XFDSDS

These LFs represent different layouts of the same table dictionary, giving code-to-map, code-to-long description, and code-to-short description access paths.

#### HXPBNFIT (LF)
- **Base PF:** TXPBNFIT (missing)
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN

Logical view over benefits data, keyed by benefit and plan identifiers.

#### HXPNSTN (LF)
- **Base PF:** TXPNSTN (missing)
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST

View over institution status data, keyed by level and status.

## 3. Status and Type Reference Data

Approved business rules in the DATA_MAINTENANCE domain provide insight into status and control codes:

- **XFXCNTR:**
  - "When X equals zero, branch to 'EXIT'" (BR-001).
  - "When X equals 40, branch to 'EXIT'" (BR-002).
  These rules show that numeric counters or status fields use specific cutoff values (0, 40) to terminate processing.

- **XFXCYMD:**
  - Rules such as "When VYY is less than 1800, branch to 'EXIT'" and "When VYY is greater than 2100, branch to 'EXIT'" define valid year ranges.
  - Additional rules constrain month (VMM) and day (VDD) values relative to calendar norms.

- **XFXLDSC:**
  - Multiple rules of the form "When LDAMAP is greater than 99, branch to 'EXIT'" and "... greater than 9999, branch to 'EXIT'" describe limit values for mapping codes.

- **XFXTABL:**
  - Rules like "When *IN79 equals on/active, branch to 'EXIT'" indicate the use of indicator 79 for control/status transitions.

- **HABADTE (PATIENT_MANAGEMENT domain):**
  - "When -FILE INDICATOR equals zero, branch to 'SKIP'".
  - "When -FLAG INDICATOR equals void/voided, branch to 'SKIP'".
  - "When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'".

These rules collectively demonstrate that status and type reference data is encoded using numeric ranges, indicator bits (*INxx), and flag fields. While explicit code lists are stored in HXPTABLD and related tables, the rules make clear how these codes influence control flow.

If any formal status-code catalog exists beyond HXPTABLD, it is not directly surfaced in the aggregated rules; thus, **no additional explicit status-code tables are identified beyond the table dictionary and level PFs.**

## 4. Stored Procedure Logic Mappings

Although the platform does not use stored procedures in the relational database sense, CALL edges between programs effectively define procedural mappings.

### 4.1 HABADTE Call Map

Caller: **HABADTE**
- Callees:
  - XFXMRNROL – MRN rollover procedure.
  - XFXCNTR – counter/control logic (called multiple times).
  - XFXLDSC – level description lookup.
  - XFXCYMD – calendar date validation.
  - XFXGETID – ID retrieval using XML header file.
  - XFXTABL – code table lookup.

HABADTE orchestrates these procedures in sequence, first validating input dates and counters, then mapping institution and level metadata, performing MRN rollover if needed, and finally persisting XML records and transfer entries.

### 4.2 Secondary Caller Maps

Caller: **XFXCYMD**
- Callee: XFXLEAP – specialized leap-year computation.

Caller: **XFXMRNROL**
- Callees:
  - HXHAPPPRF – missing program that likely handles application profile checks.
  - HXXAPPPRF – SQLRPGLE routine that queries profile-related tables.

Caller: **HXXAPPPRF**
- Callees (COPY targets):
  - HXXCNTRL – control structure definitions.
  - HXXAPPPRFP – parameter/control include.

Caller: **XFXGETID**
- Callee (COPY target): HXXLDA – logical data area configuration.
- File operations: READ HXFXMLR.

These mappings show a layered approach: HABADTE calls XFXMRNROL for MRN servicing, which in turn invokes profile routines; XFXCYMD calls XFXLEAP for specialized date handling; XFXGETID bridges configuration (HXXLDA) with XML header reads.

## 5. Service Class Method Reference

Based on hotspot scores, we can classify programs into service classes.

- **BatchService (high score):**
  - HABADTE (score 38) – central batch-style patient management and XML generation service. Handles multiple file operations and complex control paths.

- **WorkflowService (medium scores):**
  - XFXLDSC (score 15) – workflow-level service for interpreting multi-level configuration across HXPLVL1–6.
  - XFXTABL (score 11) – workflow service for code table lookups using HXPTABLD and derivatives.
  - XFXCNTR (score 9) – workflow control service managing counters and exit branches.
  - XFXMRNROL (score 7) – workflow service for MRN rollover sequences.
  - HXXAPPPRF (score 7) – workflow service interfacing with application profiles and SQL-based queries.
  - XFXGETID (score 7) – workflow service for ID assignment.
  - XFXCYMD (score 5) – workflow service for date range validation.

- **UtilityService (lower scores):**
  - XFXLEAP (score 3) – utility method implementing leap-year logic.
  - HXHAPPPRF (score 3) – utility-like external program for profile checks.
  - Logical-file drivers (HAPIRNK, HXLTABLD, HXLTABLP, HXLTABLS, HXPNSTN, HMLMAST5H, HXPBNFIT) – simple wrappers around PFs.

This classification informs how to group functions into services when re-platforming—for example, a `PatientManagementService` (HABADTE), `ReferenceDataService` (XFXTABL, HXPTABLD, HXLTABLD/LP/LS), and `IdentityService` (XFXMRNROL, XFXGETID, OMPMAST/HAPTRFR).

## 6. External Interfaces

High-impact gaps highlight inbound external interfaces:

- **TAPIRNK / HAPIRNK:** Incoming rank or transfer data, likely from feeder systems or batch files, is exposed through the HAPIRNK LF over TAPIRNK.
- **TMPMAST / HMLMAST5H:** External or upstream patient master data (TMPMAST) is consumed via HMLMAST5H logical file.
- **TXPBNFIT / HXPBNFIT and TXPNSTN / HXPNSTN:** External benefit and institution status sources are wrapped in LFs for local use.
- ******HXPXML:** Composite XML work file(s) referenced by HABADTE, representing inbound or staging XML payloads.
- **PRINTER:** Printer file referenced by HABADTE, used as an outbound SPOOL interface for reports or notifications.

The aggregated context does not explicitly identify any additional outbound SPOOL interfaces beyond the PRINTER file reference. Therefore:

- **No outbound SPOOL interfaces identified beyond the HABADTE PRINTER file reference.**

In modernization, XML work files and printer interfaces should be mapped to message queues, REST endpoints, or reporting services.

## 7. Performance and Security Notes

### 7.1 Complexity Hotspots

Cyclomatic complexity per program shows:

- **HABADTE:** cc = 152 (HIGH band) – primary hotspot, with a dense network of conditional branches related to flags (FILE INDICATOR, FLAG INDICATOR, INPATIENT/OUTPATIENT FLAG) and integration steps.
- **XFXTABL:** cc = 9 (LOW band but highest among utilities) – code table service with multiple decision points.
- **XFXCYMD:** cc = 7 – date validation service with multiple range checks.
- **XFXLDSC:** cc = 5 – level description mapping with several mapping-related conditions.
- The remaining utilities (XFXCNTR, XFXGETID, XFXLEAP, XFXMRNROL, HXXAPPPRF, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML, CXXXMLP) have cc = 1–3 and are structurally simple.

Performance risks thus concentrate in HABADTE; refactoring it into smaller modules and employing optimized data access patterns (e.g., reducing repeated lookups to XFXCNTR and XFXTABL) will improve maintainability and execution efficiency.

### 7.2 PHI Fields and Security Posture

PHI-tagged fields span several PFs:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN).
- **HXPDICT:** multiple MRN and account fields, patient names (XCNAME, ENNAME), room numbers, phone numbers, dates of birth.
- **OAPIRNK:** BRKMRN (MRN).
- **OMPMAST:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN.
- **OXPBNFIT:** XFBTEL (PhoneNumber).

Data lineage shows most PHI exposure as "ISOLATED", meaning fields are present but not widely propagated through transformation chains in the extracted scope. However, HABADTE reads HAPTRFR and interacts with XML files, implying potential serialization of PHI into XML for external interfaces.

Security implications:

- All PFs with PHI must be treated as high-sensitivity tables in the target platform, with field-level masking or encryption where appropriate.
- XML-related files (HXPXMLD/R, HXFXMLH/D/R) should be inspected for PHI elements and secured in transit and at rest.
- Printer outputs driven by HABADTE must be governed by strict access controls and auditing, especially if they contain patient-identifiable data.

### 7.3 Tech Debt Summary

The tech_debt_summary indicates:

- **Total findings:** 0
- **Total remediation hours:** 0.0
- **By severity:** HIGH: 0, MEDIUM: 0, LOW: 0

From a static analysis perspective, the codebase does not exhibit formally recorded technical debt issues. However, architectural observations (monolithic HABADTE, missing components, reliance on DDS logical files, and XML/printer interfaces) point to modernization challenges that are not yet captured as tech debt tickets but should be tracked in transformation planning.
