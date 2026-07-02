# AS400 Code Analysis Report – HABADTE Run 202607011209

## 1. Component Inventory

### 1.1 Member Inventory

The HABADTE application landscape for this run consists of 37 source members spanning data definition (DDS), traditional RPG, and SQLRPGLE components. Table 1 summarizes the inventory.

| Member Name | Type    | Subsystem | Line Count |
|------------|---------|-----------|-----------|
| HAPIRNK    | DDS_LF  | HAP       | 13        |
| HAPTRFR    | DDS_PF  | HAP       | 72        |
| HMLMAST5H  | DDS_LF  | UNGROUPED | 12        |
| HXLTABLD   | DDS_LF  | HXLT      | 11        |
| HXLTABLP   | DDS_LF  | HXLT      | 12        |
| HXLTABLS   | DDS_LF  | HXLT      | 12        |
| HXPBNFIT   | DDS_LF  | HXP       | 12        |
| HXPDICT    | DDS_PF  | HXP       | 6130      |
| HXPLVL1    | DDS_PF  | HXPL      | 49        |
| HXPLVL2    | DDS_PF  | HXPL      | 52        |
| HXPLVL3    | DDS_PF  | HXPL      | 52        |
| HXPLVL4    | DDS_PF  | HXPL      | 52        |
| HXPLVL5    | DDS_PF  | HXPL      | 55        |
| HXPLVL6    | DDS_PF  | HXPL      | 321       |
| HXPNSTN    | DDS_LF  | HXP       | 12        |
| HXPTABLD   | DDS_PF  | HXP       | 19        |
| HXPXMLD    | DDS_PF  | HXPX      | 19        |
| HXPXMLR    | DDS_PF  | HXPX      | 19        |
| OAPIRNK    | DDS_PF  | UNGROUPED | 80        |
| OMPMAST    | DDS_PF  | UNGROUPED | 310       |
| OXPBNFIT   | DDS_PF  | OXP       | 48        |
| OXPNSTN    | DDS_PF  | OXP       | 65        |
| HXXAPPPRF  | SQLRPGLE| HXXA      | 123       |
| XFXCNTR    | RPGLE   | XFXC      | 49        |
| XFXCYMD    | RPGLE   | XFXC      | 83        |
| XFXGETID   | RPGLE   | XFX       | 61        |
| XFXLDSC    | RPGLE   | XFXL      | 135       |
| XFXLEAP    | RPGLE   | XFXL      | 61        |
| XFXMRNROL  | RPGLE   | XFX       | 65        |
| XFXTABL    | RPGLE   | XFX       | 164       |
| CXXXMLP    | SQLRPGLE| UNGROUPED | 25        |
| HXXAPPPRFP | RPGLE   | HXXA      | 42        |
| HXXCNTRL   | RPGLE   | HXX       | 8         |
| HXXLDA     | RPGLE   | HXXL      | 53        |
| HXXLEVEL   | RPGLE   | HXXL      | 25        |
| HXXXML     | RPGLE   | HXX       | 11        |
| HABADTE    | RPGLE   | HA        | 821       |

### 1.2 Component Counts by Type

Based on the aggregated source statistics:

- **DDS logical files (DDS_LF):** 7
- **DDS physical files (DDS_PF):** 15
- **RPGLE programs:** 13
- **SQLRPGLE programs:** 2

This mix shows a data‑centric design: the majority of artifacts are DDS definitions and RPG programs that operate directly on those files. The HABADTE RPGLE program, with 821 lines and high cyclomatic complexity, is the central patient management driver in this landscape.

## 2. Missing Components

The gap report identifies eight missing or unresolved components. These represent references in the codebase that do not have corresponding source members in the harvested set.

### 2.1 Gaps by Impact Level

**HIGH Impact Gaps**

| Name    | Type      | Referenced By |
|---------|-----------|---------------|
| CXXXMLC | COPYBOOK  | HABAD         |
| TAPIRNK | FILE      | HAPIR         |
| TMPMAST | FILE      | HMLMA         |
| TXPBNFIT| FILE      | HXPBN         |
| TXPNSTN | FILE      | HXPNS         |

These high‑impact gaps reflect core data structures or copy members that are referenced in patient and benefit processing flows but not available for analysis. 

- **CXXXMLC** is a copybook included by HABAD, likely defining XML mapping or interface structures used by HABADTE and related programs. Its absence means XML layout and external integration structures are inferred only from usage points.
- **TAPIRNK** and **TMPMAST** represent physical files behind the HAPIRNK and HMLMAST5H logical files, which are part of patient ranking and master records. Missing base PF definitions limit full understanding of record formats and indexes.
- **TXPBNFIT** and **TXPNSTN** are base files for benefit and location/extension logic. Their omission constrains reconstruction of benefit plan keys and facility/namespace structures.

**MEDIUM Impact Gaps**

| Name     | Type    | Referenced By |
|----------|---------|---------------|
| HXHAPPPRF| PROGRAM | XFXMR         |
| ****HXPXML| FILE   | HABAD         |
| PRINTER  | FILE    | HABAD         |

- **HXHAPPPRF** is a missing program called by XFXMRNROL; this is likely an application profile handler driving patient or visit‑level context. Its absence affects profiling logic but not basic data structures.
- ******HXPXML** and **PRINTER** are file references within HABAD, probably XML staging and spool/printer output targets. Their gap affects reconstruction of outbound interfaces and reporting channels.

**LOW Impact Gaps**

None identified in this codebase. All documented gaps have either HIGH or MEDIUM impact on analysis.

## 3. Duplicate and Reused Components

This section analyzes reuse patterns by examining dependency edges where a component appears multiple times as a target.

### 3.1 Frequently Called Programs

From the call graph:

- **XFXCNTR** is called three times by HABADTE. Multiple call edges suggest that counter or control logic is reused across different phases of the HABADTE workflow, possibly in initialization, per‑record processing, and finalization.
- **XFXCYMD** is called by HABADTE and itself calls **XFXLEAP**. This indicates reusable date validation and leap‑year logic delegated to a specialized routine.
- **XFXLDSC** is read by HABADTE indirectly through its file operations; while not directly called, it is central to level‑description handling.
- **XFXTABL** is called by HABADTE for table‑driven control and reads multiple table variants (XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4), showing a reusable catalog of code tables.

These reused programs function as shared service routines in the data‑maintenance and patient‑management domains.

### 3.2 Logical Files Sharing Base Physical Files

The PFILE relationships in the DDS show several logical files layered over common physical files:

- **HAPIRNK → TAPIRNK**
- **HMLMAST5H → TMPMAST**
- **HXLTABLD / HXLTABLP / HXLTABLS → HXPTABLD**
- **HXPBNFIT → TXPBNFIT**
- **HXPNSTN → TXPNSTN**

These patterns indicate heavy reuse of underlying dictionary or table structures. For example, the three HXLTABL* logical files view the HXPTABLD table through different key sequences and select patterns to support mapping, printing, and status views. HXPBNFIT and HXPNSTN similarly provide logical projections over benefit and station/namespace data.

## 4. Dependency Analysis

### 4.1 Call Chain Overview

The dependency edges define a clear control hierarchy:

- **HABADTE** is the primary driver program, calling:
  - XFXMRNROL – MRN role or patient identifier role resolution
  - XFXCNTR – counter/control logic
  - XFXLDSC – level description lookup across HXPLVL1–HXPLVL6
  - XFXCYMD – date validation and calendar logic
  - XFXGETID – identifier retrieval using HXFXMLR
  - XFXTABL – table‑driven control rules

- **XFXCYMD** calls **XFXLEAP**, offloading leap‑year checks.
- **XFXMRNROL** calls **HXHAPPPRF** (missing) and **HXXAPPPRF**, linking MRN role logic with application profile context.
- **HXXAPPPRF** copies **HXXCNTRL** and **HXXAPPPRFP**, sharing control structures and profile processing routines.
- **XFXGETID** copies **HXXLDA** and reads **HXFXMLR**, bridging XML record layouts and identifier retrieval.
- **HABADTE** copies HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, and CXXXMLC, reinforcing its central role as a coordinator of XML templates, level descriptors, and control data.

This call graph reveals a star topology where HABADTE orchestrates data‑maintenance utilities (XFX*) and application profile routines (HXX*), with most other programs having low fan‑out.

### 4.2 File Access and Data Flow

Key READ/WRITE operations:

- **HABADTE**:
  - READs HAPTRFR (transaction/transfer records for patient accounts).
  - READs XFFNSTN (station/namespace table).
  - READs, UPDATES, and WRITEs HXFXMLH (XML header or control file).
  - WRITEs HXFXMLD (XML detail records).

- **XFXGETID**:
  - READs HXFXMLR (XML record file) to derive or confirm patient identifiers.

- **XFXLDSC**:
  - READs HXFLVL1–HXFLVL6, suggesting progressive level or hierarchy resolution (e.g., benefit or coverage levels).

- **XFXTABL**:
  - READs XFFTABLD and three additional variants (XFFTABL2–XFFTABL4), implying multiple related code table structures.

These operations map a data flow where transactional data (HAPTRFR) and static reference data (HXPLVL*, XFFNSTN, table files) feed into HABADTE, which in turn writes XML outputs via HXFXMLH/D. The XML files likely underpin downstream integration or reporting.

### 4.3 Hotspots and Complexity

The hotspot analysis ranks programs by a composite score (fan‑in, fan‑out, and file operations):

- **HABADTE** – score 38, fan‑out 13, file_ops 6; primary batch or workflow driver.
- **XFXLDSC** – score 15; file_ops 6 across hierarchical level files.
- **XFXTABL** – score 11; file_ops 4 across table variants.
- **XFXCNTR**, **XFXMRNROL**, **XFXGETID**, and **HXXAPPPRF** have moderate scores (7–9) reflecting shared business services.

Cyclomatic complexity data shows HABADTE with **cc=152 (HIGH band)**, making it the dominant risk component from a maintainability and testing standpoint. All other RPGLE and SQLRPGLE programs have cc ≤ 9 and are classified as LOW complexity.

## 5. Summary of Key Findings

### 5.1 Critical Issues and Architectural Risks

Using the high/medium gap list and orphan program inventory, the following key findings emerge:

1. **Central driver HABADTE depends on missing XML copybook CXXXMLC and file ****HXPXML.** This limits full reconstruction of XML structures and may hide external interface contracts. Any modernization effort must recover these members or reconstruct their schemas from downstream systems.
2. **Core patient and benefit data are exposed through missing physical files TAPIRNK, TMPMAST, TXPBNFIT, and TXPNSTN.** Logical files and related programs reference these, but without the underlying DDS we lack full field definitions and indexes. This is a primary data‑model gap.
3. **MRN role processing calls a missing program HXHAPPPRF.** While HXXAPPPRF exists and is invoked, the missing HXHAPPPRF indicates an alternate or earlier implementation. The call from XFXMRNROL suggests this program is still part of the intended control path.
4. **HABADTE is an architectural hotspot with very high complexity and broad fan‑out.** Any change to patient management workflows must be carefully regression‑tested. Its coupling to many utility programs and XML/file operations makes it a candidate for refactoring into smaller service modules.
5. **Orphan programs (HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC) currently show no callers beyond HABADTE or internal references in this harvested set.** In practice, these may be reusable utilities, but from the available inventory they appear isolated. This affects reuse analysis and may indicate missing higher‑level drivers.

### 5.2 Architectural Observations

- The design follows a **hub‑and‑spoke pattern** with HABADTE as the hub orchestrating utility spokes in the XFX and HXX subsystems.
- Data is persisted primarily in DDS physical files with logical files providing optimized keys and projections, consistent with traditional AS400 design.
- Table‑driven logic (XFXTABL + HXPTABLD family) is heavily used, which is favorable for modernization because business rules can be externalized into configuration stores.
- The absence of printer and XML output file definitions in the harvested set constrains integration analysis but clearly points to **outbound reporting and interface responsibilities** inside HABADTE.

Overall, the codebase is structurally sound but tightly coupled around a single high‑complexity driver. Addressing the documented gaps and refactoring HABADTE into modular services will significantly improve maintainability and support migration off the AS400 platform.