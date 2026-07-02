# Low-Level Design (LLD) – HABADTE AS400 Application (Run 202607021244)

## 1. Architecture Overview

### 1.1 Technology Stack and Component Types

The HABADTE application is a classic AS400 system built around DDS-defined files and RPG-based business logic. The aggregated inventory reveals the following distribution of component types:

- **DDS logical files (DDS_LF):** 7
- **DDS physical files (DDS_PF):** 15
- **RPGLE programs:** 13
- **SQLRPGLE programs:** 2

This stack indicates:

- Persistent data modeled as DDS PFs (e.g., HAPTRFR, HXPDICT, HXPLVL1–6, HXPTABLD, HXPXMLD/R, OMPMAST, OXPBNFIT, OXPNSTN), with logical views for alternative keys and domain-specific perspectives.
- Business workflows and data maintenance rules implemented primarily in RPGLE (HABADTE, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML).
- SQLRPGLE modules (HXXAPPPRF, CXXXMLP) used for profile management and XML-related operations.

### 1.2 Call Graph Summary

The dependency edges outline a clear orchestration pattern:

- **Core orchestration:**
  - HABADTE → XFXMRNROL, XFXCNTR (multiple invocations), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL (CALL)
- **Helper relationships:**
  - XFXCYMD → XFXLEAP (CALL) for date/leap-year calculations.
  - XFXMRNROL → HXHAPPPRF (missing), HXXAPPPRF (CALL), bridging MRN roll logic and profile processing.
- **Copybook/includes:**
  - HXXAPPPRF → HXXCNTRL, HXXAPPPRFP (COPY)
  - XFXGETID → HXXLDA (COPY)
  - HABADTE → HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC (COPY)

File relationships complement these calls:

- PFILE_OF edges from logical to physical files (HAPIRNK→TAPIRNK, HMLMAST5H→TMPMAST, HXLTABLD/P/S→HXPTABLD, HXPBNFIT→TXPBNFIT, HXPNSTN→TXPNSTN).
- READ/WRITE/UPDATE edges from programs to files (e.g., HABADTE → HAPTRFR, XFFNSTN, HXFXMLH/D; XFXLDSC → HXFLVL1–6; XFXTABL → XFFTABLD/XFFTABL2–4; XFXGETID → HXFXMLR).

### 1.3 Design Patterns

The architecture exhibits several structural patterns:

- **Controller/Coordinator pattern:** HABADTE acts as a controller, orchestrating calls to specialised modules for date validation (XFXCYMD/XFXLEAP), level description (XFXLDSC), table-driven decisions (XFXTABL), MRN roll and profile logic (XFXMRNROL/HXXAPPPRF), and ID resolution (XFXGETID).
- **Table-driven business rules:** XFXTABL centralises behaviour driven by table contents (XFFTABLD and related files), allowing configuration changes without code modifications.
- **Layered data access:** XFXLDSC and XFXGETID encapsulate access to level and XML files, providing clear separation between business logic and raw DDS structures.
- **Profile abstraction via SQL:** HXXAPPPRF uses SQLRPGLE to access profile tables such as HXPAPPPRF/HXPAPPL6 (per interpretation details), enabling more flexible querying over DDS data.

## 2. Database Schema

### 2.1 Physical Files (PF)

Each PF subsection summarises record format, uniqueness, key fields, field count, and PHI-sensitive fields.

#### HAPTRFR
- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

HAPTRFR appears to store transfer records tied to patient accounts and MRNs, keyed by level, account, transfer date/time, and type.

#### HXPDICT
- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** (none declared)
- **Total fields:** 2705
- **PHI fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HXPDICT is a large dictionary/master file, likely consolidating patient identifiers, contact details, room assignments, and other reference data.

#### HXPLVL1–HXPLVL6
- **HXPLVL1** – record format HXFLVL1, unique key HX1NUM, 36 fields, no PHI.
- **HXPLVL2** – HXFLVL2, unique key HX2NUM, 39 fields, no PHI.
- **HXPLVL3** – HXFLVL3, unique key HX3NUM, 39 fields, no PHI.
- **HXPLVL4** – HXFLVL4, unique key HX4NUM, 39 fields, no PHI.
- **HXPLVL5** – HXFLVL5, unique key HX5NUM, 42 fields, no PHI.
- **HXPLVL6** – HXFLVL6, unique key HX6NUM, 155 fields, no PHI.

These level files represent multi-tier configuration structures, likely modelling benefit levels, service tiers, or hierarchical codes consumed by XFXLDSC.

#### HXPTABLD
- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI fields:** none

HXPTABLD is a compact table file, probably holding code/description mappings and used as a base for several logical views.

#### HXPXMLD and HXPXMLR
- **HXPXMLD** – record format HXFXMLD, unique key [XMDUSR, XMDSEQ, XMDSQ2], total fields 4, no PHI.
- **HXPXMLR** – record format HXFXMLR, unique key [XMRUSR, XMRSEQ, XMRID], total fields 4, no PHI.

These PFs store XML header/detail control and are central to XML processing for HABADTE and XFXGETID.

#### OAPIRNK
- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI fields:** BRKMRN (MRN)

#### OMPMAST
- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI fields:** MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)

OMPMAST is a core patient master file with multi-level account and identification data.

#### OXPBNFIT
- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI fields:** XFBTEL (PhoneNumber)

#### OXPNSTN
- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI fields:** none

### 2.2 Logical Files (LF)

Logical files provide alternative keys and filtered views over the PF schemas.

#### HAPIRNK
- **PFILE:** TAPIRNK (missing PF, PFILE_OF from HAPIRNK → TAPIRNK)
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** none

This LF likely exposes the same record layout as OAPIRNK/TAPIRNK, keyed by level, account, and sequence, for inquiry or processing routines.

#### HMLMAST5H
- **PFILE:** TMPMAST (missing PF)
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit:** none

Provides a view over patient master records keyed by posting station and admission date/time.

#### HXLTABLD / HXLTABLP / HXLTABLS
- **PFILE:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:**
  - HXLTABLD – XFDTCD, XFDMAP
  - HXLTABLP – XFDTCD, XFDLDS
  - HXLTABLS – XFDTCD, XFDSDS
- **Select/Omit:** none

These three LFs offer distinct indexing paths into the same table data, supporting different mapping dimensions (map codes, load descriptions, subset descriptions).

#### HXPBNFIT
- **PFILE:** TXPBNFIT (missing PF)
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/Omit:** none

#### HXPNSTN
- **PFILE:** TXPNSTN (missing PF)
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/Omit:** none

## 3. Status and Type Reference Data

Business rules contain implicit status/type codes that should be treated as enumerations in a modernised system.

From the approved rules and interpretation details:

- In **XFXCNTR**, conditions such as “When X equals zero” and “When X equals 40” represent counter or code values that trigger exits.
- In **XFXCYMD**, comparisons on year (VYY), month (VMM), and day (VDD) boundaries essentially enforce date validity rules, indirectly referencing ranges of allowable date components.
- In **XFXTABL**, conditions on indicator *IN79 (on/active) reflect UI or process flags that govern exit behaviour.
- In **HABADTE**, flagged indicators such as “FILE INDICATOR equals zero,” “FLAG INDICATOR equals void/voided,” and “INPATIENT/OUTPATIENT FLAG equals outpatient” embody status semantics that should be reified as typed status fields.

There is no explicit, centralised status-code table exposed in the schema beyond HXPTABLD and XFFTABL*, which already provide table-driven mappings. Additional dedicated status/type tables are not identified; any further code sets should be derived from the business rule corpus. If no other status tables are discovered, **“None identified.”** beyond these rule-derived codes applies.

## 4. Stored Procedure Logic Mappings

Although the system primarily uses RPG CALLs rather than SQL stored procedures, we can map call relationships by caller:

### HABADTE
- **Callees (CALL):** XFXMRNROL, XFXCNTR (multiple), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL
- **Call Characteristics:**
  - High fan-out (13 edges) with a mix of control, date validation, level description, table-driven logic, and MRN roll/profile operations.
  - Multiple calls to XFXCNTR suggest a reusable control/counter routine used across different phases of the admission/date processing workflow.

### XFXCYMD
- **Callees:** XFXLEAP (CALL)
- Responsible for composite date validation, delegating leap-year logic to XFXLEAP.

### XFXMRNROL
- **Callees:** HXHAPPPRF (missing), HXXAPPPRF (CALL)
- Acts as a bridge between MRN roll logic and application profile handlers. Missing HXHAPPPRF reduces visibility into one branch of this mapping.

### XFXGETID
- **Callees:** none via CALL
- Uses copybook HXXLDA and reads HXFXMLR, functioning as an ID derivation routine from XML data.

### HXXAPPPRF
- **Callees:** HXXCNTRL, HXXAPPPRFP (COPY)
- SQLRPGLE profile program including control and profile processing logic modules.

In a modern architecture, these call relationships can be transformed into service method invocations or API calls, preserving the grouping observed above.

## 5. Service Class Method Reference

We classify hotspot programs into service classes based on their scores:

- **BatchService (high score):**
  - **HABADTE (score 38):** Core batch or orchestrated process handling patient admission/date, XML persistence, and integration with MRN/profile services.

- **WorkflowService (medium score):**
  - **XFXLDSC (score 15):** Level description workflow service, controlling multi-file reads of HXPLVL* and HXFLVL*.
  - **XFXTABL (score 11):** Table lookup workflow service, driving configuration-based branching.
  - **XFXCNTR (score 9):** Counter/control workflow, invoked from HABADTE to drive control flow.
  - **XFXMRNROL, HXXAPPPRF, XFXGETID (scores 7):** MRN roll/profile and ID workflow services integrating patient and profile tables.

- **UtilityService (low score):**
  - **XFXCYMD (score 5):** Utility for date validation with leap-year calculation delegated to XFXLEAP.
  - **XFXLEAP, HXHAPPPRF (score 3), HMLMAST5H, HXLTABLS, HXPNSTN, HXPBNFIT, HAPIRNK, HXLTABLD, HXLTABLP (scores 2):** Smaller utilities and logical views providing focused operations or access paths.

This classification assists in mapping AS400 components to service-oriented constructs during modernisation.

## 6. External Interfaces

High-impact gaps in the graph represent missing interfaces that nonetheless shape system behaviour:

- **Files TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN** act as inbound data sources for logical files HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN. Their absence in the inventory indicates external PFs in other libraries or environments.
- **CXXXMLC (COPYBOOK)** is an inbound XML layout definition included by HABADTE. It governs how XML records are structured for HXFXMLD/HXFXMLH and possibly external systems consuming these XML files.
- ******HXPXML and PRINTER** references from HABADTE suggest XML configuration and printing interfaces. They likely map to external files or devices controlling outbound report or document generation.

No explicit outbound SPOOL interfaces are identified beyond the PRINTER reference; therefore:

- **No outbound SPOOL interfaces identified.**

Any modern integration approach should treat TAPIRNK/TMPMAST/TXPBNFIT/TXPNSTN and XML/print targets as external data sources/sinks with well-defined contracts.

## 7. Performance and Security Notes

### 7.1 Complexity Hotspots

Cyclomatic complexity metrics highlight:

- **HABADTE – cc 152, band HIGH:** This program contains extensive branching, flags, and indicator-driven logic for patient management. It is the primary performance and maintainability risk. Refactoring into smaller service modules (e.g., admission validation service, XML persistence service, MRN/profile coordination service) is strongly recommended.
- All other measured programs (HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, CXXXMLP, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML) have **LOW** complexity bands (cc between 1 and 9). These are structurally simpler and suitable for direct translation or encapsulation as microservices or library functions.

### 7.2 PHI Field Exposure

PHI-tagged fields appear primarily in patient and benefit master files:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN (MRN); XFBTEL (PhoneNumber); XCNAME, ENNAME (PatientName); HXRMNO, XFRMNO (RoomNumber); HVACCT, IHACCT (AccountNumber); WBDATE (DateOfBirth)
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO, MMMMRN (MRN); MMACCT (AccountNumber); MMNAME (PatientName); MMPSSN (SSN)
- **OXPBNFIT:** XFBTEL (PhoneNumber)

Exposure paths are currently classified as **ISOLATED**, meaning no direct multi-hop propagation beyond their origin files has been detected in the compact lineage. Nonetheless, HABADTE’s high complexity and XML write operations (HXFXMLH/D) indicate that PHI may be transformed into XML outputs, requiring careful review of field-level mapping within HABADTE and related copybooks.

### 7.3 Tech Debt Summary

The aggregated tech debt metrics show:

- **Total findings:** 4
- **Total remediation hours:** 26.9
- **By severity:** 1 HIGH, 3 MEDIUM, 0 LOW

Given the structural analysis, likely high-severity items include:

- HABADTE’s extreme complexity and central role.
- Missing PFs and copybooks affecting completeness of data lineage and dependency validation.

Medium-severity items probably relate to profile asymmetry (missing HXHAPPPRF), XML/print integration gaps, and older DDS design patterns that complicate schema evolution.

Addressing the high-severity issue (refactoring HABADTE, clarifying missing PFs/copybooks) will significantly reduce risk and unlock modernisation options such as API-based services and event-driven XML generation.
