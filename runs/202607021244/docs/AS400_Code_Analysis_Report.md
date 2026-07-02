# AS400 Code Analysis Report - HABADTE Project

## 1. Component Inventory

The HABADTE project consists of 37 source members spanning database description specifications (DDS), traditional RPGLE programs, and SQLRPGLE components. The inventory reflects a typical clinical reporting application on IBM i, with patient master files, transfer history, benefit tables, dictionary structures, and a focused reporting driver.

### 1.1 Member Inventory Table

| Name       | Type    | Subsystem   | Lines |
|-----------|---------|------------|-------|
| HAPIRNK   | DDS_LF  | HAP        | 13    |
| HAPTRFR   | DDS_PF  | HAP        | 72    |
| HMLMAST5H | DDS_LF  | UNGROUPED  | 12    |
| HXLTABLD  | DDS_LF  | HXLT       | 11    |
| HXLTABLP  | DDS_LF  | HXLT       | 12    |
| HXLTABLS  | DDS_LF  | HXLT       | 12    |
| HXPBNFIT  | DDS_LF  | HXP        | 12    |
| HXPDICT   | DDS_PF  | HXP        | 6130  |
| HXPLVL1   | DDS_PF  | HXPL       | 49    |
| HXPLVL2   | DDS_PF  | HXPL       | 52    |
| HXPLVL3   | DDS_PF  | HXPL       | 52    |
| HXPLVL4   | DDS_PF  | HXPL       | 52    |
| HXPLVL5   | DDS_PF  | HXPL       | 55    |
| HXPLVL6   | DDS_PF  | HXPL       | 321   |
| HXPNSTN   | DDS_LF  | HXP        | 12    |
| HXPTABLD  | DDS_PF  | HXP        | 19    |
| HXPXMLD   | DDS_PF  | HXPX       | 19    |
| HXPXMLR   | DDS_PF  | HXPX       | 19    |
| OAPIRNK   | DDS_PF  | UNGROUPED  | 80    |
| OMPMAST   | DDS_PF  | UNGROUPED  | 310   |
| OXPBNFIT  | DDS_PF  | OXP        | 48    |
| OXPNSTN   | DDS_PF  | OXP        | 65    |
| HXXAPPPRF | SQLRPGLE| HXXA       | 123   |
| XFXCNTR   | RPGLE   | XFXC       | 49    |
| XFXCYMD   | RPGLE   | XFXC       | 83    |
| XFXGETID  | RPGLE   | XFX        | 61    |
| XFXLDSC   | RPGLE   | XFXL       | 135   |
| XFXLEAP   | RPGLE   | XFXL       | 61    |
| XFXMRNROL | RPGLE   | XFX        | 65    |
| XFXTABL   | RPGLE   | XFX        | 164   |
| CXXXMLP   | SQLRPGLE| UNGROUPED  | 25    |
| HXXAPPPRFP| RPGLE   | HXXA       | 42    |
| HXXCNTRL  | RPGLE   | HXX        | 8     |
| HXXLDA    | RPGLE   | HXXL       | 53    |
| HXXLEVEL  | RPGLE   | HXXL       | 25    |
| HXXXML    | RPGLE   | HXX        | 11    |
| HABADTE   | RPGLE   | HA         | 821   |

### 1.2 Members by Type

From the aggregated statistics:

- DDS_LF: 7
- DDS_PF: 15
- SQLRPGLE: 2
- RPGLE: 13

This mix shows a database-heavy application with a small number of executable programs orchestrating complex patient reporting logic. The HABADTE program itself is relatively large (821 lines) and high in cyclomatic complexity (152, band HIGH), which positions it as the primary orchestration component.

## 2. Missing Components

Several referenced artefacts are missing from the scanned codebase. These represent unresolved COPY members, physical files, and device descriptions that the runtime expects.

### 2.1 Gap Table by Impact

| Name      | Type      | Impact | Referenced By |
|----------|-----------|--------|---------------|
| CXXXMLC  | COPYBOOK  | HIGH   | HABAD         |
| TAPIRNK  | FILE      | HIGH   | HAPIR         |
| TMPMAST  | FILE      | HIGH   | HMLMA         |
| TXPBNFIT | FILE      | HIGH   | HXPBN         |
| TXPNSTN  | FILE      | HIGH   | HXPNS        |
| HXHAPPPRF| PROGRAM   | MEDIUM | XFXMR        |
| ****HXPXML| FILE     | MEDIUM | HABAD        |
| PRINTER  | FILE      | MEDIUM | HABAD        |

High-impact gaps correspond mostly to missing physical files (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) and an XML copybook (CXXXMLC). Without these components, key logical files (HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN) cannot resolve their base physical files, and HABADTE cannot fully generate XML output. Medium-impact gaps relate to optional XML infrastructure and a control program (HXHAPPPRF) used by MRN rollup logic.

Operationally, these gaps indicate that the local reverse-engineering snapshot is incomplete relative to the production library. Any modernization or testing must either reconstruct these artefacts from other environments or create stubs to mimic their behaviour.

## 3. Duplicate and Reused Components

The dependency graph shows several forms of reuse:

1. **Programs called multiple times:**
   - `XFXCNTR` is called three times from `HABADTE`. This program centers headings and text by manipulating spacing, and repeated calls signal its use as a generic presentation helper.
   - `XFXLDSC`, `XFXCYMD`, `XFXTABL`, and `XFXGETID` are each called from HABADTE as shared services for date conversion, table lookups, and XML identifier generation.

2. **Logical files referencing common physical files:**
   - `HXLTABLD`, `HXLTABLP`, and `HXLTABLS` all reference physical file `HXPTABLD` as PFILE_OF edges, exposing different views (detail, page-level, and status slices) over the same dictionary structure.
   - `HXPBNFIT` references `TXPBNFIT`, and `HXPNSTN` references `TXPNSTN`. These logical files present filtered and keyed views for benefits and nursing station definitions.
   - `HAPIRNK` references `TAPIRNK`, and `HMLMAST5H` references `TMPMAST`, making them specialized projection files for transfer and master patient data, respectively.

The reuse patterns are appropriate for AS/400 environments, but the absence of the underlying physical files in this snapshot means that data access is conceptually defined but not fully resolvable. The multiple CALL edges to `XFXCNTR` highlight this component as a shared presentation utility rather than a true duplicate; the same compiled object is reused rather than having separate clones.

## 4. Dependency Analysis

### 4.1 Program Call Chain

Key CALL relationships from the dependency edges include:

- `HABADTE` → `XFXMRNROL` (MRN roll-up logic determining whether to report by Medical Record Number or account).
- `HABADTE` → `XFXCNTR` (centered report headings and labels).
- `HABADTE` → `XFXLDSC` (corporate structure description lookup for hospital levels).
- `HABADTE` → `XFXCYMD` (date format conversion between CCYYMMDD and presentation formats).
- `HABADTE` → `XFXGETID` (retrieval of XML token fragments used in report layout).
- `HABADTE` → `XFXTABL` (table-based lookup for room class descriptions and hospital/therapeutic leave mapping).
- `XFXCYMD` → `XFXLEAP` (leap-year calculations supporting date validation).
- `XFXMRNROL` → `HXHAPPPRF` and `HXXAPPPRF` (application profile programs controlling MRN roll-up behaviour).

COPY relationships emphasise shared definitions:

- `HABADTE` copies `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, and missing `CXXXMLC` to obtain LDA layout, level mapping, XML scaffolding, and shared constants.
- `XFXGETID` copies `HXXLDA` to access LDA-based user and sequence information.
- `HXXAPPPRF` copies `HXXCNTRL` and `HXXAPPPRFP` for control data and profile parameters.

File access relationships show HABADTE as the principal data consumer:

- READ `HAPTRFR` to obtain room transfers and admit information.
- READ `XFFNSTN` via the `HXPNSTN` logical file to resolve nursing station descriptions.
- READ/UPDATE/WRITE `HXFXMLH` and WRITE `HXFXMLD` to drive XML report header and detail generation.

### 4.2 Hotspot Summary

The `dep_hotspots` metrics rank programs by combined fan-in/fan-out and file operations:

- **HABADTE** – score 38, fan_out 13, file_ops 6. This is the central batch/report driver, orchestrating calls to utility programs and performing multiple file reads and XML writes.
- **XFXLDSC** – score 15, fan_in 1, file_ops 6. Acts as a level-description service over the HXPLVL* files.
- **XFXTABL** – score 11, fan_in 1, file_ops 4. Serves as a general table-driven lookup engine for codes such as room class.
- **XFXCNTR** – score 9, fan_in 3. A frequently reused text-centering helper.
- **XFXMRNROL** and **HXXAPPPRF** – scores 7, each acting as configuration and profile drivers for MRN roll-up.

This hotspot profile confirms that modernization efforts should focus first on HABADTE and its XML/reporting infrastructure, then on XFXLDSC and XFXTABL as reusable services that would benefit from refactoring and service extraction.

## 5. Summary of Key Findings

### 5.1 Critical Issues from High/Medium Gaps

The filtered high/medium gaps include:

| Name      | Type      | Impact | Referenced By |
|----------|-----------|--------|---------------|
| CXXXMLC  | COPYBOOK  | HIGH   | HABADTE       |
| HXHAPPPRF| PROGRAM   | MEDIUM | XFXMRNROL     |
| TAPIRNK  | FILE      | HIGH   | HAPIRNK       |

Key structural risks:

- **XML Copybook Missing (CXXXMLC):** HABADTE relies on CXXXMLC for XML constants or layout definitions, but the copybook is absent. While some XML scaffolding is visible in HABADTE’s own logic, missing shared definitions increase the risk of inconsistent XML across reports or runtime failures when compiling in a production-like environment.
- **External Profile Program (HXHAPPPRF):** XFXMRNROL calls HXHAPPPRF to decide MRN roll-up behaviour. Without this program, MRN-rollup behaviour may default or fail, potentially affecting how accounts are grouped in HABADTE’s report (MED REC # versus ACCOUNT # headings).
- **Base Files for Logical Views (TAPIRNK and other high-impact files):** Logical file HAPIRNK expects TAPIRNK, and similar patterns exist for TMPMAST, TXPBNFIT, and TXPNSTN. Their absence breaks the physical/logical file relationship necessary for the OS/400 database engine, making it impossible to test report behaviour involving transfer history, master accounts, benefits, and nursing stations.

### 5.2 Orphan Programs and Architectural Observations

Orphan programs identified:

- `HXXAPPPRF` (SQLRPGLE)
- `XFXCNTR` (RPGLE)
- `XFXCYMD` (RPGLE)
- `XFXGETID` (RPGLE)
- `XFXLDSC` (RPGLE)

In the aggregated context, these are labelled as having no callers; however, HABADTE does call several of them. This discrepancy suggests that the orphan list is computed across the wider library, but the HABADTE project view may be partial. Architecturally, these programs function as shared utilities and would be good candidates for extraction into reusable services in a refactored architecture (e.g., date-service, text-format-service, XML-token-service).

Overall, the HABADTE codebase presents:

- A high-complexity reporting driver (HABADTE) tightly coupled to XML generation and multiple DDS files.
- A set of small, low-complexity utility programs encapsulating cross-cutting concerns like centering text, date transformations, table-driven translations, and MRN roll-up.
- Data structures focused on patient master information, transfer history, benefits, and nursing stations, with clear PHI sensitivity concentrated in OMPMAST, HXPDICT, HAPTRFR, OAPIRNK, and OXPBNFIT.

These findings should guide modernization to prioritize decoupling HABADTE’s logic into smaller services, reconstructing missing base files and copybooks, and hardening PHI-related structures before migration.
