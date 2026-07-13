# AS400 Code Analysis Report – HABADTE Project

## Section 1 – Component Inventory

### 1.1 Member Inventory

The HABADTE project comprises 41 source members spanning physical files (PF), logical files (LF), and RPG programs. The table below summarises each member’s role, subsystem, and size.

| Member | Type    | Subsystem | Lines |
|--------|---------|-----------|-------|
| HAPIRNK   | DDS_LF   | HAP        | 13  |
| HAPTRFR   | DDS_PF   | HAP        | 72  |
| HMLMAST5H | DDS_LF   | UNGROUPED  | 12  |
| HXLTABLD  | DDS_LF   | HXLT       | 11  |
| HXLTABLP  | DDS_LF   | HXLT       | 12  |
| HXLTABLS  | DDS_LF   | HXLT       | 12  |
| HXPBNFIT  | DDS_LF   | HXP        | 12  |
| HXPDICT   | DDS_PF   | HXP        | 6130 |
| HXPLVL1   | DDS_PF   | HXPL       | 49  |
| HXPLVL2   | DDS_PF   | HXPL       | 52  |
| HXPLVL3   | DDS_PF   | HXPL       | 52  |
| HXPLVL4   | DDS_PF   | HXPL       | 52  |
| HXPLVL5   | DDS_PF   | HXPL       | 55  |
| HXPLVL6   | DDS_PF   | HXPL       | 321 |
| HXPNSTN   | DDS_LF   | HXP        | 12  |
| HXPTABLD  | DDS_PF   | HXP        | 19  |
| HXPXMLD   | DDS_PF   | HXPX       | 19  |
| HXPXMLR   | DDS_PF   | HXPX       | 19  |
| OAPIRNK   | DDS_PF   | UNGROUPED  | 80  |
| OMPMAST   | DDS_PF   | UNGROUPED  | 310 |
| OXPBNFIT  | DDS_PF   | OXP        | 48  |
| OXPNSTN   | DDS_PF   | OXP        | 65  |
| TAPIRNK   | DDS_PF   | UNGROUPED  | 167 |
| TMPMAST   | DDS_PF   | UNGROUPED  | 688 |
| TXPBNFIT  | DDS_PF   | TXP        | 164 |
| TXPNSTN   | DDS_PF   | TXP        | 116 |
| HXXAPPPRF | SQLRPGLE | HXXA       | 123 |
| XFXCNTR   | RPGLE    | XFXC       | 49  |
| XFXCYMD   | RPGLE    | XFXC       | 83  |
| XFXGETID  | RPGLE    | XFX        | 61  |
| XFXLDSC   | RPGLE    | XFXL       | 135 |
| XFXLEAP   | RPGLE    | XFXL       | 61  |
| XFXMRNROL | RPGLE    | XFX        | 65  |
| XFXTABL   | RPGLE    | XFX        | 164 |
| CXXXMLP   | SQLRPGLE | UNGROUPED  | 25  |
| HXXAPPPRFP| RPGLE    | HXXA       | 42  |
| HXXCNTRL  | RPGLE    | HXX        | 8   |
| HXXLDA    | RPGLE    | HXXL       | 53  |
| HXXLEVEL  | RPGLE    | HXXL       | 25  |
| HXXXML    | RPGLE    | HXX        | 11  |
| HABADTE   | RPGLE    | HA         | 821 |

### 1.2 Component Types and Technology Mix

The codebase is dominated by DDS-described database files with a supporting set of service-style RPG programs:

- DDS logical files (DDS_LF): 7
- DDS physical files (DDS_PF): 19
- RPGLE programs: 13
- SQLRPGLE programs: 2

This spread indicates a traditional AS400 application with a rich data model and a central, high‑complexity driver program (HABADTE) orchestrating logic. The PF/LF combinations (e.g., HXPTABLD with HXLTABLD/HXLTABLP/HXLTABLS) show a clear separation between base data and multiple logical views for different access paths.

The main HABADTE program, at 821 lines and high cyclomatic complexity, acts as the primary coordination point for patient‑management logic, delegating specialised tasks such as date validation, MRN roll logic, and table lookups to smaller utility programs.

## Section 2 – Missing Components

The gap analysis identifies components referenced in the code but not present in the harvested source set. These represent incomplete visibility into the runtime behaviour and potential external integrations.

### 2.1 High‑Impact Gaps

- **CXXXMLC (COPYBOOK, impact: HIGH, referenced_by: HABAD)**  
  A core copy member providing XML control or layout definitions for HABADTE. Its absence means parts of the XML header/footer, control structures, or shared constants used when generating XML output cannot be fully reconstructed. Any changes to XML structure or schema mappings that reside in this copybook are not visible to the reverse‑engineering effort.

### 2.2 Medium‑Impact Gaps

- **HXHAPPPRF (PROGRAM, impact: MEDIUM, referenced_by: XFXMR)**  
  This program is called from XFXMRNROL but is missing from the harvested set. It likely contains additional application profile or MRN roll logic. The call dependency indicates an integration point between MRN roll processing and application‑profile maintenance. Reverse‑engineers must treat this as an opaque operation when replaying end‑to‑end behaviour.

- ******HXPXML (FILE, impact: MEDIUM, referenced_by: HABAD)**  
  A file (likely PF or LF) used in XML processing paths from HABADTE. Without its DDS definition, field‑level mappings and output record structure for XML persistence cannot be fully documented. This impacts traceability of XML payloads and may hide additional PHI or audit fields.

- **PRINTER (FILE, impact: MEDIUM, referenced_by: HABAD)**  
  A printer or spool‑file definition used by HABADTE for output. Its absence means print layout, report identifiers, and downstream distribution channels are unknown. This particularly affects documentation of outbound reporting channels and any compliance‑specific print streams.

### 2.3 Implications by Category

**Copybook gaps:** Missing copybooks like CXXXMLC reduce confidence in shared constant sets, XML tags, and condition flags. Implementers should assume that additional constants and configuration flags are embedded there. Any modernisation should plan to discover or reconstruct these from runtime logs or deployment libraries.

**Program gaps:** The missing HXHAPPPRF program suggests that MRN roll processing spans multiple modules. During migration or re‑implementation, this module must either be located in another library or its behaviour inferred from surrounding logic and available business rules in XFXMRNROL and HXXAPPPRF.

**File gaps:** Unknown files such as ****HXPXML and PRINTER prevent complete lineage mapping for some outputs. They likely model XML staging tables and printer device files. When modernising, treat these as external interfaces rather than purely internal persistence layers.

## Section 3 – Duplicate and Reused Components

### 3.1 Reused Programs (CALL Targets)

Analysis of call edges highlights several programs that are reused across the codebase:

- **XFXCNTR** is called multiple times from HABADTE. There are three explicit CALL relationships from HABADTE to XFXCNTR, indicating that it is invoked at several decision points, most likely to perform field‑formatting and counter management. Reuse at multiple call sites emphasises its role as a generic formatting/validation utility.

- **XFXMRNROL** is called from HABADTE and in turn calls HXHAPPPRF and HXXAPPPRF. This positions XFXMRNROL as an MRN roll coordinator orchestrating profile updates across multiple programs.

- **XFXCYMD** is both a callee of HABADTE and a caller of XFXLEAP. It centralises date validation and leap‑year logic that is reused wherever HABADTE requires robust date handling.

- **XFXLDSC** has fan_in from HABADTE and interacts with multiple level tables (HXPLVL1–HXPLVL6). Its reuse concentrates all level‑lookup behaviour in one location.

This reuse pattern shows a clear carving of cross‑cutting concerns into shared utilities, reducing duplication of date and table logic but making these modules critical for any refactoring of core behaviour.

### 3.2 Shared PF/LF Structures

Logical files are consistently layered over shared physical files:

- **HXPTABLD** is the base PF for three LFs: HXLTABLD, HXLTABLP, and HXLTABLS. Each LF provides a different key structure (e.g., XFDTCD + XFDMAP vs XFDTCD + XFDLDS vs XFDTCD + XFDSDS), enabling alternate access paths and business‑specific views over the same table definition data.

- **TXPBNFIT** is the base PF for LF HXPBNFIT, giving logical access to benefit data by user/plan keys (XFBUBN, XFBPLN).

- **TXPNSTN** feeds LF HXPNSTN, providing alternate access to nested or level‑6 station data.

- **TAPIRNK** and **TMPMAST** are base tables whose logical views (HAPIRNK, HMLMAST5H) enable domain‑specific selection and ordering for rank and master data.

These patterns confirm that the design exploits DDS logical files to provide denormalised business views and performance‑tuned access paths without duplicating data. Any schema modernisation should preserve these differentiated access patterns, for example via SQL indexed views or application‑level service methods.

## Section 4 – Dependency Analysis

### 4.1 Call Chain Overview

The call graph shows HABADTE as the top‑level driver orchestrating a set of specialised service programs:

- **HABADTE → XFXMRNROL → (HXHAPPPRF, HXXAPPPRF)**  
  HABADTE delegates MRN roll operations to XFXMRNROL, which then calls HXHAPPPRF (missing) and HXXAPPPRF. The combination suggests a separation between legacy profile update logic and newer SQL‑based profile management in HXXAPPPRF.

- **HABADTE → XFXCNTR** (three distinct calls)  
  These calls correspond to reusable field‑formatting rules such as exiting when fields are blank or when leading characters are non‑blank.

- **HABADTE → XFXLDSC**  
  HABADTE calls into level‑description logic that reads multiple hierarchy level files (HXPLVL1–HXPLVL6). This supports multi‑tier level lookups, potentially for patient, location, or benefit hierarchies.

- **HABADTE → XFXCYMD → XFXLEAP**  
  Date validation cascades into a leap‑year utility, encapsulating calendar rules in dedicated programs.

- **HABADTE → XFXGETID**  
  ID retrieval logic accesses XML‑related files, specifically HXFXMLR, to obtain identifiers tied to XML records.

- **HABADTE → XFXTABL**  
  Table lookup logic reads from multiple table variants (XFFTABLD and XFFTABL2–XFFTABL4), supporting coded‑value translations and dictionary‑driven behaviour.

### 4.2 File Access Patterns

File dependency edges reveal how programs interact with key PFs and LFs:

- **HABADTE** reads **HAPTRFR** (transfer data), **XFFNSTN** (station or level‑6 data), and **HXFXMLH/HXFXMLD** (XML header/detail records). It performs READ, UPDATE, and WRITE operations on HXFXMLH and WRITE operations on HXFXMLD, confirming its role as an XML report or extract generator.

- **XFXLDSC** declares and reads **HXPLVL1–HXPLVL6** and their associated format files HXFLVL1–HXFLVL6, performing hierarchical level lookups.

- **XFXTABL** reads **XFFTABLD** and related table variants, acting as a generalised table‑lookup engine for code translations.

- **XFXGETID** declares and reads **HXPXMLR/HXFXMLR**, retrieving identifiers tied to XML records, likely used when correlating XML output with persistent state.

These access patterns show a clear layering: HABADTE orchestrates high‑level flows, while downstream utilities encapsulate specialised file access behaviour.

### 4.3 Hotspot and Risk Assessment

The hotspot list ranks programs by a composite score derived from fan‑in, fan‑out, and file operations:

- **HABADTE** – score 38, fan_in 0, fan_out 13, file_ops 6. This program is the central orchestrator and has high cyclomatic complexity (152, HIGH band). It is the primary risk for regression during modernisation and should be targeted for decomposition into smaller services.

- **XFXLDSC** – score 15, fan_in 1, fan_out 0, file_ops 6. Acts as a level‑lookup service. Any change to level hierarchies must be aligned with this program.

- **XFXTABL** – score 11, fan_in 1, fan_out 0, file_ops 4. Provides reusable table‑lookup behaviour and should be treated as a core shared service.

- **XFXCNTR, XFXMRNROL, HXXAPPPRF, XFXGETID, XFXCYMD** – moderate scores with low complexity indicate stable, focused utilities. They are less risky individually but collectively form the infrastructure on which HABADTE depends.

Overall, the architecture is hub‑and‑spoke, with HABADTE at the centre and several high‑reuse utilities at the periphery.

## Section 5 – Summary of Key Findings

### 5.1 Architectural Observations

- The system is organised around a single high‑complexity driver (HABADTE) that orchestrates multiple smaller utilities. This centralisation is common in legacy RPG systems but poses maintainability risks.
- Utility programs (XFXCYMD, XFXLDSC, XFXTABL, XFXCNTR, XFXGETID, XFXMRNROL) encapsulate cross‑cutting concerns such as date handling, level lookup, table lookups, and MRN roll logic, which is an architectural strength.
- Logical files are consistently used to provide alternate access paths over shared physical tables, particularly for table dictionary and level structures, which would map naturally to indexed views or service methods in a modern architecture.

### 5.2 Orphan Program Analysis

The orphan list includes HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC. However, the dependency graph shows that HABADTE calls XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC, and XFXMRNROL calls HXXAPPPRF. Therefore, these are not true orphans in the scanned codebase.

No true orphans detected. All programs have defined callers in the scanned codebase.

### 5.3 Recommendations

- **Decompose HABADTE** into smaller service modules aligned with MRN management, XML generation, and transfer processing to reduce complexity and improve testability.
- **Formalise shared utilities** (XFXCYMD, XFXLDSC, XFXTABL, XFXCNTR) as explicit services or libraries in the target architecture, preserving their current reuse patterns.
- **Document external interfaces** associated with XML files and printer definitions once the missing components are located, ensuring that all outbound integrations are captured before modernisation.
