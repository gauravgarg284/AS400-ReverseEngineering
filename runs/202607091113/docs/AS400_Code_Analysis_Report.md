# AS/400 Code Analysis Report

Run ID: 202607091113  
System: Legacy AS/400 (IBM i)  
Primary Domain: Eligibility, benefits, and member profile processing (HAP/HXP/HXX/XFX subsystems)

---

## 1. Component Inventory

### 1.1 Member Inventory

The table below summarizes all discovered source members, grouped by subsystem. Line counts are taken directly from the harvested source and give a proxy for relative size and maintenance effort.

| Member Name | Type    | Subsystem  | Lines |
|-------------|---------|-----------:|------:|
| HAPIRNK     | DDS_LF  | HAP        | 13   |
| HAPTRFR     | DDS_PF  | HAP        | 72   |
| HMLMAST5H   | DDS_LF  | UNGROUPED  | 12   |
| HXLTABLD    | DDS_LF  | HXLT       | 11   |
| HXLTABLP    | DDS_LF  | HXLT       | 12   |
| HXLTABLS    | DDS_LF  | HXLT       | 12   |
| HXPBNFIT    | DDS_LF  | HXP        | 12   |
| HXPDICT     | DDS_PF  | HXP        | 6130 |
| HXPLVL1     | DDS_PF  | HXPL       | 49   |
| HXPLVL2     | DDS_PF  | HXPL       | 52   |
| HXPLVL3     | DDS_PF  | HXPL       | 52   |
| HXPLVL4     | DDS_PF  | HXPL       | 52   |
| HXPLVL5     | DDS_PF  | HXPL       | 55   |
| HXPLVL6     | DDS_PF  | HXPL       | 321  |
| HXPNSTN     | DDS_LF  | HXP        | 12   |
| HXPTABLD    | DDS_PF  | HXP        | 19   |
| HXPXMLD     | DDS_PF  | HXPX       | 19   |
| HXPXMLR     | DDS_PF  | HXPX       | 19   |
| OAPIRNK     | DDS_PF  | UNGROUPED  | 80   |
| OMPMAST     | DDS_PF  | UNGROUPED  | 310  |
| OXPBNFIT    | DDS_PF  | OXP        | 48   |
| OXPNSTN     | DDS_PF  | OXP        | 65   |
| TAPIRNK     | DDS_PF  | UNGROUPED  | 167  |
| TMPMAST     | DDS_PF  | UNGROUPED  | 688  |
| TXPBNFIT    | DDS_PF  | TXP        | 164  |
| TXPNSTN     | DDS_PF  | TXP        | 116  |
| HXXAPPPRF   | SQLRPGLE| HXXA       | 123  |
| XFXCNTR     | RPGLE   | XFXC       | 49   |
| XFXCYMD     | RPGLE   | XFXC       | 83   |
| XFXGETID    | RPGLE   | XFX        | 61   |
| XFXLDSC     | RPGLE   | XFXL       | 135  |
| XFXLEAP     | RPGLE   | XFXL       | 61   |
| XFXMRNROL   | RPGLE   | XFX        | 65   |
| XFXTABL     | RPGLE   | XFX        | 164  |
| CXXXMLP     | SQLRPGLE| UNGROUPED  | 25   |
| HXXAPPPRFP  | RPGLE   | HXXA       | 42   |
| HXXCNTRL    | RPGLE   | HXX        | 8    |
| HXXLDA      | RPGLE   | HXXL       | 53   |
| HXXLEVEL    | RPGLE   | HXXL       | 25   |
| HXXXML      | RPGLE   | HXX        | 11   |
| HABADTE     | RPGLE   | HA         | 821  |

### 1.2 Component Type Totals

Totals by source type (from the harvested manifest):

- DDS Logical Files (DDS_LF): 7
- DDS Physical Files (DDS_PF): 19
- RPGLE Programs: 13
- SQLRPGLE Programs: 2

This distribution shows a data‑centric application with a heavy DDS schema footprint and a moderate number of procedural programs that orchestrate benefits, member profiles, and XML/file handling.

---

## 2. Missing Components

The gap analysis identified several referenced artifacts that are not present in the harvested library list. These are critical for understanding runtime behavior and for de‑risking modernization.

### 2.1 Gap List

| Name      | Type      | Impact | Referenced By |
|-----------|-----------|--------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH   | HABAD         |
| HXHAPPPRF | PROGRAM   | MEDIUM | XFXMR         |
| ****HXPXML| FILE      | MEDIUM | HABAD         |
| PRINTER   | FILE      | MEDIUM | HABAD         |

Note: The `referenced_by` values in the original inventory correspond to truncated member names (`HABAD`, `XFXMR`). In the dependency edges, these map to fully qualified programs such as `HABADTE` and `XFXMRNROL`.

### 2.2 High‑Impact Gaps

**CXXXMLC (COPYBOOK, HIGH impact)**  
This copy member is included from `HABADTE` via a COPY edge. Given that `HABADTE` is the primary hotspot (fan‑out 13, high complexity, multiple file updates), a missing copybook here suggests opaque shared definitions—likely XML envelope layout, status codes, or common I/O structures. Without CXXXMLC, the fields and constants used by HABADTE cannot be fully resolved, which directly affects:

- Accurate reconstruction of XML payloads and outbound document formats.
- Understanding of error‑handling branches that depend on copy‑level constants.
- Reliable refactoring of message construction and parsing logic.

### 2.3 Medium‑Impact Gaps

**HXHAPPPRF (PROGRAM, MEDIUM impact)**  
HXHAPPPRF appears in a CALL edge from `XFXMRNROL`. That program is a hotspot (fan_in 1, fan_out 2) and sits in the MRN/role management area. The missing callee likely performs profile‑level operations (e.g., loading or persisting application profiles). Its absence:

- Prevents full reconstruction of the MRN role roll‑up behavior.
- Hides key side‑effects on the HXX* profile files.

**`****HXPXML` (FILE, MEDIUM impact)**  
This file is referenced from `HABADTE`. Because HABADTE interacts with multiple XML‑related objects (HXFXMLH, HXPXMLD, HXPXMLR), this missing file probably represents an XML control or history file (for example, header or spool‑like tracking). Missing metadata here impairs:

- End‑to‑end mapping of XML request and response persistence.
- Identification of archival vs operational XML datasets.

**PRINTER (FILE, MEDIUM impact)**  
`PRINTER` is referenced from HABADTE. This is likely an output queue or printer‑file configuration. Without the DDS definition:

- Report layout, print line formats, and control codes cannot be derived.
- Modernization teams cannot easily map printer output to PDFs or downstream channels.

### 2.4 Implications by Category

**Copybook Gaps**  
Missing copybooks like CXXXMLC typically centralize constants, external data structures, and shared prototypes. Their absence complicates automated translation because:

- Struct definitions have to be inferred from usage patterns rather than authoritative declarations.
- Duplicated logic may hide behind copy‑level includes, making deduplication much harder.

**Program Gaps**  
HXHAPPPRF being absent means we see outbound calls from XFXMRNROL but cannot determine:

- Whether this call is mandatory or conditional for specific workflows.
- The extent of database updates, especially on HXXAPPPRF and related tables.

**File Gaps**  
Files like PRINTER and ****HXPXML are part of the overall data lineage and are important for:

- Migration of printed reports and spool output.
- Complete PHI data lineage, especially if XML payloads include patient identifiers.

These gaps should be remediated by either sourcing the missing objects from alternate libraries or documenting them as contractual external dependencies before any greenfield rewrite.

---

## 3. Duplicate and Reused Components

This section focuses on reuse patterns derived from the dependency edges.

### 3.1 Reused Programs (CALL Targets)

From the CALL edges:

- `HABADTE` calls multiple XFX utilities (XFXMRNROL, XFXCNTR, XFXLDSC, XFXCYMD, XFXGETID, XFXTABL), in some cases multiple times.
- `XFXCYMD` calls `XFXLEAP`.
- `XFXMRNROL` calls both `HXHAPPPRF` and `HXXAPPPRF`.

The most heavily reused utility in terms of call frequency is:

- **XFXCNTR** – Called three times from `HABADTE`. This pattern strongly suggests a generic counter/formatting routine used in different branches of the main ABAD transaction logic.

Other notable reuse patterns:

- **XFXLDSC** – Single caller (`HABADTE`) but extensive reuse of file reads across all HXPLVL* tables. This indicates a centralized loader for hierarchical level descriptors.
- **XFXTABL** – Called from `HABADTE` and reading multiple table variants (XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4), functioning as a generic table lookup utility.

### 3.2 Shared Physical Files

PFILE_OF relationships show logical files sharing the same physical file:

- `HXLTABLD`, `HXLTABLP`, and `HXLTABLS` are all logical views over **HXPTABLD**.
- `HXPBNFIT` is a logical file over **TXPBNFIT**.
- `HXPNSTN` is a logical file over **TXPNSTN**.
- `HAPIRNK` is a logical file over **TAPIRNK**.
- `HMLMAST5H` is a logical file over **TMPMAST**.

These patterns show a normalized set of physical tables with multiple logical projections supporting different key sequences or filtered subsets. For example:

- **HXPTABLD / HXLTABL*** – A shared code‑table PF with multiple LFs providing distinct key orders (by code, map, or description) for different processing flows.
- **TMPMAST / HMLMAST5H** – TMPMAST appears to be a master profile table with HMLMAST5H providing a filtered or sorted view, likely used for high‑volume read operations.

Recognizing these shared structures is critical for schema consolidation in a relational or document‑based modern platform.

---

## 4. Dependency Analysis

### 4.1 Call Chain Overview

The core procedural call graph centers around `HABADTE`:

- `HABADTE` → `XFXMRNROL` → (`HXHAPPPRF`, `HXXAPPPRF`)
- `HABADTE` → `XFXCNTR` (three separate call sites)
- `HABADTE` → `XFXLDSC` → `HXPLVL1` … `HXPLVL6` (READs)
- `HABADTE` → `XFXCYMD` → `XFXLEAP`
- `HABADTE` → `XFXGETID` → `HXFXMLR` (READ)
- `HABADTE` → `XFXTABL` → `XFFTABL*` (READ)

File I/O edges from `HABADTE` include:

- READ **HAPTRFR** (transfer records)
- READ **XFFNSTN** (status code table)
- READ/UPDATE/WRITE **HXFXMLH** (XML header/history)
- WRITE **HXFXMLD** (XML detail)

This topology establishes HABADTE as the orchestrator responsible for:

- Driving benefit or transaction processing flows.
- Coordinating MRN role assignments via XFXMRNROL and HXXAPPPRF.
- Managing XML persistence via HXFXML* files.

### 4.2 Hotspot Programs

From the hotspot analysis:

- **HABADTE** – Score 38, fan_in 0, fan_out 13, file_ops 6, cyclomatic complexity 152 (HIGH). This is the central batch/controller program with dense branching and extensive external I/O.
- **XFXLDSC** – Score 15, fan_in 1, fan_out 0, file_ops 6. Acts as a data access layer for the HXPLVL* hierarchy.
- **XFXTABL** – Score 11, fan_in 1, fan_out 0, file_ops 4. Generalized table lookup service.
- **XFXCNTR** – Score 9, fan_in 3, fan_out 0. Heavily reused counter/formatter.
- **XFXMRNROL** and **HXXAPPPRF** – Scores 7; both handle profile‑level logic driven from HABADTE.

Risk assessment:

- Any modernization or refactor must prioritize **HABADTE**. Its high complexity and centrality suggest that defects or behavioral regressions here would have system‑wide impact.
- XFXLDSC and XFXTABL should be treated as reusable service modules when designing new service boundaries—they encapsulate recurring patterns of data access.

### 4.3 Data Access Chains

The combination of PFILE_OF and READ/WRITE edges reveals multi‑step lineage such as:

- HABADTE → XFXLDSC → HXPLVL* physical files (multi‑level benefit hierarchy).
- HABADTE → XFXTABL → HXPTABLD / XFFTABL* (configuration or code tables).
- HABADTE → HAPTRFR → downstream XML headers/details (HXFXMLH, HXFXMLD).

These chains should be preserved when modeling domain services such as “Transfer Processing” or “XML Export Service”.

---

## 5. Summary of Key Findings

### 5.1 Architectural Observations

1. **Central Orchestrator Pattern** – HABADTE acts as a master orchestrator, coordinating benefit transfers, MRN role assignments, and XML output. Its large cyclomatic complexity (152) and high hotspot score confirm that it aggregates multiple concerns (I/O, business rules, formatting).

2. **Utility‑Heavy Design** – Multiple small XFX* programs (XFXCNTR, XFXCYMD, XFXLEAP, XFXLDSC, XFXTABL, XFXGETID) behave as utilities, providing focused services such as date validation, leap‑year logic, table lookups, and ID retrieval. This is a positive pattern, but their coupling to HABADTE makes them effectively part of a larger monolith.

3. **Hierarchical Data Model** – The HXPLVL1–HXPLVL6 physical files, accessed via XFXLDSC, implement a multi‑level hierarchy (likely benefit or plan levels). The schema is normalized and accessed via common utilities, which is well‑suited to translation into relational tables or hierarchical collections.

4. **Profile‑Centric Processing** – HXXAPPPRF and its copy variants (HXXAPPPRFP) plus the profile‑related dependencies in XFXMRNROL suggest a consistent pattern for handling application profiles and MRN roles, tied back to master tables like OMPMAST and TMPMAST.

### 5.2 Patterns and Recommendations

- **Decompose HABADTE** – Given its high complexity, HABADTE should be decomposed into separate services aligned with:
  - XML generation and persistence.
  - Transfer/benefit transaction processing.
  - MRN role/profile updates.

- **Promote XFX Utilities to First‑Class Services** – XFXCNTR, XFXCYMD, XFXLDSC, and XFXTABL can be carved out as reusable domain services (e.g., `CounterService`, `DateValidationService`, `HierarchyLookupService`, `CodeTableService`).

- **Focus on XML and Print Pipelines** – The combination of HXFXML* files, missing ****HXPXML, and PRINTER indicates a legacy output pipeline mixing XML and printed reports. A modern architecture should separate message generation, persistence, and delivery channels to reduce coupling.

### 5.3 Orphan Program Assessment

The orphan list includes HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC. However, hotspot data and dependency edges clearly show non‑zero fan_in for most of these programs (e.g., XFXCNTR is called three times; XFXLDSC has fan_in 1). Therefore:

- **No true orphans detected. All programs have defined callers in the scanned codebase.**

Any future cleanup should treat “orphan” flags with caution and cross‑check against call edges and hotspot fan_in/fan_out metrics before retiring code.
