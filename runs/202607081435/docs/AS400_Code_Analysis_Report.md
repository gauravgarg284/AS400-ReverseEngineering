# AS400 Code Analysis Report – HABADTE

## 1. Component Inventory

### 1.1 Member Inventory

The HABADTE application comprises 41 members spanning RPG programs, SQLRPGLE routines, and DDS physical and logical files. The table below summarizes the structural footprint using the aggregated source manifest.

| Member | Type   | Subsystem | Lines |
|--------|--------|-----------|-------|
| HAPIRNK   | DDS_LF   | HAP        | 13 |
| HAPTRFR   | DDS_PF   | HAP        | 72 |
| HMLMAST5H | DDS_LF   | UNGROUPED  | 12 |
| HXLTABLD  | DDS_LF   | HXLT       | 11 |
| HXLTABLP  | DDS_LF   | HXLT       | 12 |
| HXLTABLS  | DDS_LF   | HXLT       | 12 |
| HXPBNFIT  | DDS_LF   | HXP        | 12 |
| HXPDICT   | DDS_PF   | HXP        | 6130 |
| HXPLVL1   | DDS_PF   | HXPL       | 49 |
| HXPLVL2   | DDS_PF   | HXPL       | 52 |
| HXPLVL3   | DDS_PF   | HXPL       | 52 |
| HXPLVL4   | DDS_PF   | HXPL       | 52 |
| HXPLVL5   | DDS_PF   | HXPL       | 55 |
| HXPLVL6   | DDS_PF   | HXPL       | 321 |
| HXPNSTN   | DDS_LF   | HXP        | 12 |
| HXPTABLD  | DDS_PF   | HXP        | 19 |
| HXPXMLD   | DDS_PF   | HXPX       | 19 |
| HXPXMLR   | DDS_PF   | HXPX       | 19 |
| OAPIRNK   | DDS_PF   | UNGROUPED  | 80 |
| OMPMAST   | DDS_PF   | UNGROUPED  | 310 |
| OXPBNFIT  | DDS_PF   | OXP        | 48 |
| OXPNSTN   | DDS_PF   | OXP        | 65 |
| TAPIRNK   | DDS_PF   | UNGROUPED  | 167 |
| TMPMAST   | DDS_PF   | UNGROUPED  | 688 |
| TXPBNFIT  | DDS_PF   | TXP        | 164 |
| TXPNSTN   | DDS_PF   | TXP        | 116 |
| HXXAPPPRF | SQLRPGLE | HXXA       | 123 |
| XFXCNTR   | RPGLE    | XFXC       | 49 |
| XFXCYMD   | RPGLE    | XFXC       | 83 |
| XFXGETID  | RPGLE    | XFX        | 61 |
| XFXLDSC   | RPGLE    | XFXL       | 135 |
| XFXLEAP   | RPGLE    | XFXL       | 61 |
| XFXMRNROL | RPGLE    | XFX        | 65 |
| XFXTABL   | RPGLE    | XFX        | 164 |
| CXXXMLP   | SQLRPGLE | UNGROUPED  | 25 |
| HXXAPPPRFP| RPGLE    | HXXA       | 42 |
| HXXCNTRL  | RPGLE    | HXX        | 8 |
| HXXLDA    | RPGLE    | HXXL       | 53 |
| HXXLEVEL  | RPGLE    | HXXL       | 25 |
| HXXXML    | RPGLE    | HXX        | 11 |
| HABADTE   | RPGLE    | HA         | 821 |

Total line count across all members is 10,288, with RPG and SQLRPGLE programs accounting for the core executable logic and DDS members defining the data schema and access paths.

### 1.2 Inventory by Type

Using the aggregated counts by type:

- **DDS_LF:** 7 logical files
- **DDS_PF:** 19 physical files
- **SQLRPGLE:** 2 SQL-based programs (HXXAPPPRF, CXXXMLP)
- **RPGLE:** 13 traditional RPG programs

This mix indicates a data-heavy application with a modest number of executable programs orchestrating patient management and benefit/level table maintenance.


## 2. Missing Components

The aggregated gap report identifies four missing or unresolved nodes that are referenced in the scanned codebase but not present as source members:

| Name      | Type      | Impact  | Referenced By |
|-----------|-----------|---------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH    | HABAD         |
| HXHAPPPRF | PROGRAM   | MEDIUM  | XFXMR         |
| ****HXPXML| FILE      | MEDIUM  | HABAD         |
| PRINTER   | FILE      | MEDIUM  | HABAD         |

These gaps represent dependencies that would have to be addressed or stubbed during modernization.

### 2.1 High-Impact Gap – CXXXMLC Copybook

The copy member **CXXXMLC** is referenced from the HABADTE program via a `COPY` edge but is not present in the harvested source set. Being classified as **HIGH impact**, this copybook likely contains critical declarations, constants, or utility procedures related to XML handling or external interface configuration.

Implications:
- Without CXXXMLC, the full signature of XML-related data structures and constants used by HABADTE cannot be validated.
- Any modernization effort must either recover this copy member from the source library or reconstruct equivalent structures based on runtime traces and downstream consumers.
- Automated conversion tools may fail when they encounter unresolved fields or prototypes defined in CXXXMLC, requiring manual intervention.

### 2.2 Medium-Impact Program Gap – HXHAPPPRF

The program **HXHAPPPRF** is called by **XFXMRNROL** but is missing in the harvested set. Its impact is rated **MEDIUM**, suggesting that it participates in ancillary workflows such as application profile maintenance rather than core patient billing logic.

Implications:
- Call chains involving XFXMRNROL are incomplete; downstream business effects of MRN rollbacks or reassignments cannot be fully traced.
- Test cases generated for MRN rollover scenarios should treat HXHAPPPRF as an external dependency and either mock it or isolate its invocation.
- When rebuilding services around XFXMRNROL, an integration contract must be defined for HXHAPPPRF, even if its implementation is deferred.

### 2.3 Medium-Impact File Gap – ****HXPXML

The placeholder file **"****HXPXML"** is referenced by HABADTE and flagged as **MEDIUM** impact. The naming suggests an XML payload or configuration store related to patient events.

Implications:
- File I/O paths for XML-based messaging from HABADTE are incomplete, limiting end-to-end data lineage for XML outbound processing.
- Any modernization of XML workflow must account for this file, mapping it to modern storage (e.g., object storage or message queues) once its true structure is recovered.

### 2.4 Medium-Impact File Gap – PRINTER

The **PRINTER** file is referenced by HABADTE, presumably as an output device file for report or statement printing.

Implications:
- SPOOL and print-related flows cannot be fully reconstructed; this affects legacy reporting and hard-copy artifacts.
- During modernization, PRINTER should be modeled as an outbound reporting interface, likely replaced with PDF generation, electronic statements, or modern print queues.

**Note:** Section 2 is the authoritative record of missing components for this analysis. These gaps are deliberately not re-listed elsewhere in the document to avoid conflicting interpretations.


## 3. Duplicate and Reused Components

### 3.1 Reused Programs (CALL Targets)

Dependency edges show several programs being invoked multiple times, indicating reuse:

- **XFXCNTR** – Target of three CALL edges from HABADTE. This program implements field-formatting exits (e.g., exit if all blank or first character non-blank) used across multiple HABADTE flows. Its repeated invocation suggests a reusable validation routine.
- **XFXMRNROL** – Called by HABADTE and in turn calls HXHAPPPRF and HXXAPPPRF. It encapsulates MRN rollover logic, acting as a shared service for adjusting patient identifiers.
- **XFXCYMD** – Called by HABADTE and internally calls XFXLEAP. It centralizes date validation, including year, month, and day constraints.
- **XFXLDSC** – Called by HABADTE and reads multiple level tables (HXFLVL1-6), providing a unified level lookup service.
- **XFXTABL** – Called by HABADTE and reads multiple dictionary tables (XFFTABLD, XFFTABL2-4). It performs general table lookups based on indicator states such as *IN79.

These reused programs form a core set of validation and lookup services that can be re-modeled as microservices or shared domain services in a modern architecture.

### 3.2 Reused Physical Files via Logical Views

The DDS logical files show reuse of physical files through multiple access paths:

- **HXPTABLD** (PF) is shared by three logical files:
  - HXLTABLD providing a mapping via keys (XFDTCD, XFDMAP)
  - HXLTABLP providing a "print" or presentation-oriented path via keys (XFDTCD, XFDLDS)
  - HXLTABLS providing a "screen" or selection path via keys (XFDTCD, XFDSDS)

  This pattern indicates a central table of codes and descriptions (HXPTABLD) exposed through specialized logical views for different UI or report contexts.

- **TXPBNFIT** (PF) is the base for logical file **HXPBNFIT**, which defines a keyed access path on benefit plan keys (XFBUBN, XFBPLN). It is reused by multiple programs (e.g., HXPBNFIT LF, OXPBNFIT PF) to represent benefit configurations.

- **TXPNSTN** (PF) underlies logical file **HXPNSTN**, offering a keyed access path on level/status (XFNLV6, XFNSST). This design supports normalization of status codes across subsystems.

- **TAPIRNK** and **TMPMAST** similarly act as base tables for HAPIRNK and HMLMAST5H logical files, enabling alternative indexing for patient rank and master data.

Reused physical files with multiple logical overlays show deliberate normalization of reference data (levels, statuses, benefits) and the use of per-context logical views rather than duplicating tables.


## 4. Dependency Analysis

### 4.1 Call Chains

The core call graph centered on HABADTE is as follows:

- **HABADTE → XFXMRNROL → (HXHAPPPRF, HXXAPPPRF)**
  - HABADTE delegates MRN rollover logic to XFXMRNROL, which then interacts with HXHAPPPRF (missing) and HXXAPPPRF (SQLRPGLE) for application profile updates.

- **HABADTE → XFXCNTR** (three separate calls)
  - Multiple entry points in HABADTE invoke XFXCNTR for field formatting exits, ensuring consistent handling of blank or partially populated fields.

- **HABADTE → XFXLDSC → HXPLVL1-6 (READ)**
  - Level lookup flows read across the six level tables, supporting multi-level configuration (e.g., enterprise, region, facility, unit, bed, level-6 account mappings).

- **HABADTE → XFXCYMD → XFXLEAP**
  - Date validation in HABADTE is performed by XFXCYMD, which calls XFXLEAP to evaluate leap-year behavior.

- **HABADTE → XFXGETID → HXFXMLR (READ)**
  - Identifier assignment or retrieval flows interact with HXFXMLR via XFXGETID, supporting XML record-based ID resolution.

- **HABADTE → XFXTABL → XFFTABLD / XFFTABL2-4 (READ)**
  - Table lookup logic uses multiple dictionary tables for reference codes, controlled via indicators like *IN79.

This call structure confirms that HABADTE is an orchestration program coordinating several specialized validation and lookup modules rather than implementing all business rules inline.

### 4.2 Data Access and File Operations

Key data access patterns include:

- HABADTE performs **READ** operations on:
  - HAPTRFR (patient transfer transactions)
  - XFFNSTN (normalized status codes)
  - HXFXMLH (XML header or control records)

- HABADTE performs **UPDATE/WRITE** operations on:
  - HXFXMLH (XML header – updated and written)
  - HXFXMLD (XML detail records – written)

- XFXTABL performs **READ** operations across XFFTABLD, XFFTABL2, XFFTABL3, and XFFTABL4, indicating heavy reliance on dictionary tables.

- XFXLDSC performs **READ** across all level files HXFLVL1-6, encapsulating hierarchical configuration logic.

These patterns show a clear segregation of responsibilities: HABADTE orchestrates workflows and persists XML-based outcomes; XFX* programs abstract table and level logic; supporting SQLRPGLE programs perform relational queries.

### 4.3 Hotspot Analysis and Risk Assessment

Based on the dependency hotspot metrics:

- **HABADTE** – Score 38, **fan_in=0, fan_out=13, file_ops=6**.
  - This is the primary orchestration program, with high fan-out to validation, lookup, and data-access routines. It also drives XML write operations and reads transactional and status tables.
  - Cyclomatic complexity for HABADTE is **152 (HIGH band)**, indicating dense branching logic and substantial workflow variation.
  - Risk: High modernization risk due to complexity and centrality. Refactoring should proceed incrementally, with extensive regression testing.

- **XFXLDSC** – Score 15, fan_in=1, fan_out=0, file_ops=6.
  - Single caller (HABADTE) but multiple file reads. Risk arises from its role as a level-lookup service; incorrect behavior may silently corrupt configuration-driven decisions.

- **XFXTABL** – Score 11, fan_in=1, fan_out=0, file_ops=4.
  - Single caller with multiple table reads. Serves as a utility for dictionary lookups; moderate risk tied to correctness of code mappings and indicator handling.

- **XFXCNTR** – Score 9, fan_in=3, fan_out=0, file_ops=0.
  - High fan-in from HABADTE (three calls). Although file-free, it is critical to data entry validation and formatting; mistakes can cause downstream data quality issues.

- **XFXGETID**, **XFXMRNROL**, **HXXAPPPRF** – Score 7 each, moderate complexity and involvement in ID and profile management flows.

Overall, the architecture exhibits a single high-risk orchestrator (HABADTE) surrounded by lower-complexity service routines. Modernization strategies should focus first on isolating HABADTE into domain-specific services while preserving the semantics of the XFX* utility programs.


## 5. Summary of Key Findings

### 5.1 Architectural Observations

- The HABADTE application is structured around **one central orchestrator (HABADTE)** that coordinates a family of **validation and lookup services** (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL, HXXAPPPRF).
- Data schema is heavily normalized and exposed through **multiple logical files** that tailor access paths for different functional needs (e.g., benefits, statuses, levels, patient master overlays).
- XML-related components (HXFXMLH/D/R, HXPXMLD/R, CXXXMLP, missing CXXXMLC and ****HXPXML) show that the system already implements semi-structured data exchanges, likely for downstream systems or audit logs.
- The call graph is shallow but wide: HABADTE fans out to many dependencies, yet each dependency tends to have low fan-in, suggesting clear separation of concerns but tight coupling to the main program.

### 5.2 Patterns and Modernization Opportunities

- **Service Extraction:** XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, and XFXGETID can be lifted into discrete services (e.g., ValidationService, DateService, LevelLookupService, TableLookupService, IdService) with well-defined interfaces, reducing HABADTE’s complexity.
- **Configuration-Driven Design:** The extensive use of level files (HXPLVL1-6) and dictionary tables (XFFTABL*) suggests that business logic is partly data-driven. Modernization should preserve this characteristic by implementing configuration stores (e.g., database tables or configuration APIs) rather than hard-coding rules.
- **XML Integration Modernization:** XML header/detail files (HXFXMLH/D/R) and XML-related copybooks and programs (CXXXMLP, CXXXMLC, ****HXPXML) should be mapped to modern integration patterns such as message queues or REST/JSON endpoints, while retaining current validation and branching rules.
- **PHI-Aware Refactoring:** PHI-bearing files (HAPTRFR, HXPDICT, OAPIRNK, OMPMAST, OXPBNFIT) are accessed in roles aligned with patient transfers, dictionaries, and benefits. Any domain decomposition must ensure that PHI access remains confined to appropriate services with strong audit controls.

### 5.3 Recommendations

- **Prioritize HABADTE Refactor:** Given its high cyclomatic complexity and hotspot score, HABADTE should be decomposed into orchestrated services, guided by the existing user stories for patient management controls.
- **Stabilize Utility Programs First:** Before altering HABADTE, encapsulate and thoroughly test XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL, and HXXAPPPRF behind stable interfaces, ensuring their rules and data access behaviors are faithfully reproduced.
- **Model Reference Tables Explicitly:** Treat HXPLVL1-6, HXPTABLD and its logical views, TXPBNFIT/TXPNSTN and their logical views as canonical configuration entities in the target architecture, with migrations ensuring no loss of code-level semantics.
- **Strengthen Observability:** Introduce logging and metrics around key call chains (HABADTE → XFX* programs) to support safe incremental migration and post-migration validation.

### 5.4 Orphan Program Consideration

The aggregated context lists several programs as potential orphans (HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC). However, dependency metrics show non-zero fan_in for most of these (e.g., XFXCNTR has fan_in=3, XFXLDSC has fan_in=1). Therefore, they are **not true orphans** in the scanned codebase. All programs have defined callers in the analyzed application graph, and there are **no confirmed orphan programs** requiring special retirement or archival handling.
