# Low-Level Design (LLD) – HABADTE AS400 Application

## 1. Architecture Overview

### 1.1 Technology Stack and Member Types

The HABADTE application is implemented on IBM i/AS400 using classic RPG and DDS constructs, with limited use of SQLRPGLE:

- **RPGLE programs (13):** Core procedural logic for patient management, validation, table lookups, and XML processing. Key programs include HABADTE, XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL, and the HXX* utility programs.
- **SQLRPGLE programs (2):** HXXAPPPRF and CXXXMLP provide embedded SQL capabilities, likely for application profile queries and XML payload processing.
- **DDS physical files (19):** Core relational structures for patient master, benefits, levels, status tables, dictionaries, and XML control data (e.g., HAPTRFR, HXPDICT, HXPLVL1-6, HXPTABLD, HXPXMLD/R, OAPIRNK, OMPMAST, OXPBNFIT, OXPNSTN, TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN).
- **DDS logical files (7):** Alternate access paths exposing physical files via different keys: HAPIRNK, HMLMAST5H, HXLTABLD, HXLTABLP, HXLTABLS, HXPBNFIT, HXPNSTN.

This mix indicates a traditional layered design where DDS defines schema and access paths, while RPG and SQLRPGLE encapsulate business logic.

### 1.2 Call Graph Summary

The central orchestration program is **HABADTE (RPGLE, domain PATIENT_MANAGEMENT)**, which coordinates multiple utility and domain services via CALL and COPY edges:

- **HABADTE → XFXCNTR** (three calls): formatting exit routines for 40-character fields (exit if blank, exit if first char non-blank).
- **HABADTE → XFXCYMD → XFXLEAP**: date validation (year/month/day range checks) with leap-year logic delegated to XFXLEAP.
- **HABADTE → XFXLDSC → HXPLVL1-6 (READ):** level lookup service reading hierarchical level files.
- **HABADTE → XFXTABL → XFFTABLD / XFFTABL2-4 (READ):** generic table lookup across multiple dictionary files, controlled by indicator *IN79.
- **HABADTE → XFXGETID → HXFXMLR (READ):** ID retrieval via XML record file.
- **HABADTE → XFXMRNROL → (HXHAPPPRF, HXXAPPPRF):** MRN rollover logic with calls to missing HXHAPPPRF and SQLRPGLE program HXXAPPPRF.

COPY edges bind HABADTE and other programs to shared data areas and layout definitions:

- **HABADTE → HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC** (COPY): shared data areas for level and XML processing, plus missing copybook CXXXMLC.
- **XFXGETID → HXXLDA** (COPY): reuses the same data area as HABADTE for ID-related operations.
- **HXXAPPPRF → HXXCNTRL, HXXAPPPRFP** (COPY): control and profile copy members for the application profile SQLRPGLE routine.

This call graph yields a hub-and-spoke architecture: HABADTE as the hub, XFX* and HXX* programs as spokes encapsulating specific responsibilities.

### 1.3 Design Patterns Observed

- **Service-style Utility Programs:** XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, and XFXMRNROL behave like stateless services, operating on passed parameters and shared DDS data without maintaining persistent state.
- **Configuration-Driven Logic:** Level files HXPLVL1-6 and dictionary tables XFFTABLD/2/3/4 encode configuration and reference data that drive behavior (e.g., level code validity, table lookups) instead of embedding rules directly in code.
- **XML Integration Layer:** HXFXMLH/D/R, HXPXMLD/R, CXXXMLP, and the missing HXPXML file and CXXXMLC copybook form an integration layer that moves patient or transaction data into XML structures for downstream processing.

These patterns are critical inputs when mapping the legacy system to modern microservices or domain-driven design.


## 2. Database Schema

### 2.1 Physical Files (PF)

For each PF, we describe record format, key structure, uniqueness, and PHI sensitivity based on the aggregated schema.

#### HAPTRFR (PF)
- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

This table models patient transfer transactions keyed by level-6 location, account number, transfer date/time, and type. It is PHI-sensitive and participates in HABADTE read flows.

#### HXPDICT (PF)
- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** none
- **Total fields:** 2705
- **PHI fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HXPDICT appears as a large dictionary or cross-reference table containing multiple MRN, account, room, phone, and name fields, serving as a foundational PHI repository.

#### HXPLVL1–6 (PF)

Each level file represents hierarchical configuration:

- **HXPLVL1:** HXFLVL1, key HX1NUM, unique, 36 fields.
- **HXPLVL2:** HXFLVL2, key HX2NUM, unique, 39 fields.
- **HXPLVL3:** HXFLVL3, key HX3NUM, unique, 39 fields.
- **HXPLVL4:** HXFLVL4, key HX4NUM, unique, 39 fields.
- **HXPLVL5:** HXFLVL5, key HX5NUM, unique, 42 fields.
- **HXPLVL6:** HXFLVL6, key HX6NUM, unique, 155 fields.

None of these level files are directly flagged with PHI, indicating they primarily contain configuration and structural metadata (e.g., level codes, descriptions).

#### HXPTABLD (PF)
- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI fields:** none

HXPTABLD is a small dictionary table keyed by data type and detail codes. It serves as the base for multiple logical views (HXLTABLD, HXLTABLP, HXLTABLS).

#### HXPXMLD / HXPXMLR (PF)

- **HXPXMLD:** HXFXMLD, unique on (XMDUSR, XMDSEQ, XMDSQ2), 4 fields, no PHI.
- **HXPXMLR:** HXFXMLR, unique on (XMRUSR, XMRSEQ, XMRID), 4 fields, no PHI.

These files hold XML detail and record metadata, used in ID resolution and XML flow control.

#### OAPIRNK (PF)
- **Record format:** HBFIRNK
- **Unique key:** Yes (BRKLV6, BRKACC, BRKSEQ)
- **Total fields:** 33
- **PHI fields:** BRKMRN (MRN)

#### OMPMAST (PF)
- **Record format:** HMFMAST
- **Unique key:** Yes (MMPLV6, MMACCT)
- **Total fields:** 149
- **PHI fields:** MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)

OMPMAST is the central patient master record table, with extensive PHI.

#### OXPBNFIT (PF)
- **Record format:** XFFBNFIT
- **Unique key:** Yes (XFBUBN, XFBPLN)
- **Total fields:** 34
- **PHI fields:** XFBTEL (PhoneNumber)

#### OXPNSTN (PF)
- **Record format:** XFFNSTN
- **Unique key:** Yes (XFNLV6, XFNSST)
- **Total fields:** 23
- **PHI fields:** none

#### TAPIRNK / TMPMAST / TXPBNFIT / TXPNSTN (PF)

These are “ATE TABLE” PFs with no explicit keys and no PHI flags:

- TAPIRNK: 43 fields.
- TMPMAST: 181 fields.
- TXPBNFIT: 12 fields.
- TXPNSTN: 19 fields.

They act as base tables for logical views, providing broader datasets from which keyed logical files are derived.

### 2.2 Logical Files (LF)

#### HAPIRNK (LF)
- **PFILE:** TAPIRNK
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/omit:** none

Defines a keyed view over TAPIRNK for patient rank data, aligned with OAPIRNK and HAPTRFR via level and account fields.

#### HMLMAST5H (LF)
- **PFILE:** TMPMAST
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/omit:** none

Provides an access path into TMPMAST keyed by patient status and admit date/time, complementing OMPMAST.

#### HXLTABLD / HXLTABLP / HXLTABLS (LFs over HXPTABLD)

- **HXLTABLD:** keys XFDTCD, XFDMAP.
- **HXLTABLP:** keys XFDTCD, XFDLDS.
- **HXLTABLS:** keys XFDTCD, XFDSDS.

These logical files expose different attribute combinations from HXPTABLD, enabling specialized lookup flows (e.g., mapping codes, labels for print, labels for screen).

#### HXPBNFIT (LF)
- **PFILE:** TXPBNFIT
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN

Logical view over benefit base table TXPBNFIT used to drive benefit plan lookups.

#### HXPNSTN (LF)
- **PFILE:** TXPNSTN
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST

Status lookup logical file used together with OXPNSTN and related PFs to normalize status codes.


## 3. Status and Type Reference Data

Status and type codes are primarily implemented via:

- Level files HXPLVL1-6.
- Status tables OXPNSTN and TXPNSTN accessed through HXPNSTN.
- Dictionary tables HXPTABLD and its logical views, plus XFFTABL2-4.

Approved business rules indicate several status-related behaviors, such as level code validity checks in XFXLDSC and indicator-based branching in XFXTABL. However, the rule texts are generic and do not enumerate explicit status or type code values.

Based on the available rule text and schema, **no explicit list of status or type codes is present in the aggregated context**. Any modernization mapping would require reading the actual table contents during data migration.

Therefore, for this document:

- **Status code catalogue:** None identified in this codebase (schema-only view).
- **Type code catalogue:** None identified in this codebase (schema-only view).


## 4. Stored Procedure Logic Mappings

This section maps CALL edges to a stored-procedure-style view of program interaction. Each caller program is listed with its callees and call counts based on the dependency graph.

### 4.1 HABADTE (RPGLE – Patient Management Orchestrator)

Callees:
- XFXMRNROL – 1 call
- XFXCNTR – 3 calls
- XFXLDSC – 1 call
- XFXCYMD – 1 call
- XFXGETID – 1 call
- XFXTABL – 1 call

HABADTE behaves as a master stored procedure driving the patient transaction workflow, delegating specialized logic to the XFX* utilities and MRN rollover routine.

### 4.2 XFXMRNROL (RPGLE – MRN Rollover Service)

Callees:
- HXHAPPPRF – 1 call (missing program)
- HXXAPPPRF – 1 call

XFXMRNROL encapsulates MRN reassignment logic and then triggers application profile updates via HXXAPPPRF.

### 4.3 XFXCYMD (RPGLE – Date Validation Service)

Callees:
- XFXLEAP – 1 call

The date validation routine uses a dedicated leap-year checker, maintaining separation between core validation and leap-year specialization.

### 4.4 HXXAPPPRF (SQLRPGLE – Application Profile Service)

Callees via COPY / embedded constructs:
- HXXCNTRL
- HXXAPPPRFP

Although modeled via COPY edges, these copybooks can be treated as internal helpers supplying control and profile structures to the SQLRPGLE logic.

### 4.5 XFXGETID (RPGLE – Identifier Service)

Callees via data/file operations:
- HXFXMLR (READ)

XFXGETID reads from HXFXMLR to resolve identifiers, acting as a stored procedure over XML record metadata.

Programs without outbound CALL edges (e.g., XFXCNTR, XFXLDSC, XFXTABL) operate more like inline stored procedures invoked by HABADTE, with their main logic defined in the program itself rather than external callees.


## 5. Service Class Method Reference

Using hotspot scores, we classify programs into service categories:

- **BatchService (High score):** long-running or orchestration-centric services.
- **WorkflowService (Medium score):** services orchestrating several file operations with moderate complexity.
- **UtilityService (Low score):** simple lookups or validations.

### 5.1 BatchService

- **HABADTE (score 38, cc=152 HIGH):** Central BatchService orchestrating patient transaction processing, XML generation, and coordination of multiple utility services.

### 5.2 WorkflowService

- **XFXLDSC (score 15, cc=5 LOW, file_ops=6):** WorkflowService performing multi-level lookup across HXPLVL1-6 and OV tables, encapsulating hierarchical configuration resolution.
- **XFXTABL (score 11, cc=9 LOW, file_ops=4):** WorkflowService for table and dictionary lookups, driving branching based on indicator states like *IN79.
- **XFXCNTR (score 9, cc=3 LOW, fan_in=3):** WorkflowService that centralizes field-formatting exits used across HABADTE flows.
- **XFXMRNROL (score 7, cc=1 LOW):** WorkflowService encapsulating MRN rollover steps and delegating to profile services.
- **HXXAPPPRF (score 7, cc=1 LOW):** WorkflowService performing SQL-based application profile updates.

### 5.3 UtilityService

- **XFXGETID (score 7, cc=1 LOW, file_ops=1):** UtilityService for ID retrieval from HXFXMLR.
- **XFXCYMD (score 5, cc=7 LOW, file_ops=0):** UtilityService for date validation.
- **XFXLEAP (score 3, cc=1 LOW):** UtilityService for leap-year determinations.
- **HXHAPPPRF (score 3, cc=unknown, missing program):** Intended UtilityService for application profile handling in MRN workflows.
- **HMLMAST5H, HXLTABLS, HXPNSTN, HAPIRNK, HXPBNFIT, HXLTABLP, HXLTABLD** (score 2 each): Utility-style logical file services providing specialized read paths with minimal logic.

This classification supports mapping legacy programs to modern service classes and prioritizing which services require more robust orchestration or error handling.


## 6. External Interfaces

High-impact and medium-impact gaps highlight inbound external interfaces and incomplete points of integration:

- **CXXXMLC (COPYBOOK, HIGH impact):** External interface definition for XML-related structures used by HABADTE and CXXXMLP. It likely describes inbound or outbound XML message formats. Its absence means that the precise field-level contract for XML integration is unknown.
- **HXHAPPPRF (PROGRAM, MEDIUM impact):** External MRN application profile handler called by XFXMRNROL. This is an inbound dependency for HABADTE workflows dealing with patient identifier changes.
- **"****HXPXML" (FILE, MEDIUM impact):** Undocumented XML file referenced by HABADTE, probably representing an external XML payload store. The four asterisk prefix suggests a special or temporary file.
- **PRINTER (FILE, MEDIUM impact):** Outbound print interface used by HABADTE to produce reports or statements via SPOOL.

From a modernization perspective:
- CXXXMLC and "****HXPXML" represent **XML integration boundaries** that must be re-specified as modern message contracts (e.g., JSON/REST, message queues).
- HXHAPPPRF forms part of an **application profile service** that must be modeled as an external service dependency.
- PRINTER is a legacy **SPOOL output interface** that should be replaced with modern reporting mechanisms.

The aggregated context does not list any other outbound SPOOL or printer interfaces beyond the PRINTER file reference. As such:

- **Outbound SPOOL interfaces:** No outbound SPOOL interfaces identified beyond PRINTER.


## 7. Performance and Security Notes

### 7.1 Performance Characteristics

Cyclomatic complexity and hotspot metrics highlight performance and maintainability considerations:

- **HABADTE:** cc=152, HIGH band, hotspot score 38, fan_out=13, file_ops=6.
  - High branching density implies numerous conditional paths and potential performance variability depending on data and status conditions.
  - Any modernization should consider refactoring HABADTE into multiple smaller services or use case handlers to reduce complexity and improve testability.

- **XFXTABL:** cc=9, LOW band but moderate hotspot score.
  - Multiple table reads and indicator-driven branching can contribute to latency if tables grow large. Caching or indexed queries in the modern platform can mitigate this.

- **XFXLDSC:** cc=5, LOW band with several file operations.
  - Reads across HXFLVL1-6; performance depends on level table sizes and indexing. Optimization opportunities include consolidating level lookups into a single query with joins in a relational database.

Other utility programs (XFXCNTR, XFXCYMD, XFXGETID, XFXMRNROL, HXXAPPPRF) have low cyclomatic complexity and limited file operations, posing minimal performance risk individually.

### 7.2 Security and PHI Considerations

PHI-tagged fields appear across multiple files:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)
- **OXPBNFIT:** XFBTEL (PhoneNumber)

Security implications:

- Access to HABADTE and downstream flows that read HAPTRFR, OMPMAST, and HXPDICT must be guarded with strict authorization and auditing in the modern system.
- MRN, account, and SSN fields must be encrypted or tokenized at rest and masked in user interfaces where full visibility is not required.
- Phone and name fields require appropriate privacy controls, including consent and opt-out mechanisms.

### 7.3 Tech Debt Overview

Tech debt summary:

- **Total findings:** 4
- **Total remediation hours:** 26.9
- **By severity:** HIGH: 1, MEDIUM: 3, LOW: 0

The single HIGH-severity item aligns with HABADTE’s high complexity, while medium-severity items likely correspond to missing components and integration gaps (HXHAPPPRF, XML interface definitions, printer handling). Modernization should allocate remediation capacity to:

1. Refactoring HABADTE into manageable services.
2. Defining and hardening XML integration contracts.
3. Clarifying and securing MRN and profile handling paths.

By addressing these areas, the modernized system can reduce operational risk while preserving the legacy semantics codified in the HABADTE application.
