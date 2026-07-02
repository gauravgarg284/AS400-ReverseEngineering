# AS400 Code Analysis Report – HABADTE Run 202607021244

## 1. Component Inventory

### 1.1 Member Overview

The HABADTE application landscape comprises 37 source members with a total of 9,153 lines of code. The members span RPGLE, SQLRPGLE, and DDS physical/logical files, structured into several subsystems that reflect functional domains such as patient management (HA/HAP/HXP), cross‑functional data maintenance (XFX* and HXX*), and table/level configuration (HXPL*, HXPTABLD, HXLT*).

**Members by type and subsystem**

| Member      | Type    | Subsystem  | Lines |
|------------|---------|-----------|------|
| HAPIRNK    | DDS_LF  | HAP       | 13   |
| HAPTRFR    | DDS_PF  | HAP       | 72   |
| HMLMAST5H  | DDS_LF  | UNGROUPED | 12   |
| HXLTABLD   | DDS_LF  | HXLT      | 11   |
| HXLTABLP   | DDS_LF  | HXLT      | 12   |
| HXLTABLS   | DDS_LF  | HXLT      | 12   |
| HXPBNFIT   | DDS_LF  | HXP       | 12   |
| HXPDICT    | DDS_PF  | HXP       | 6130 |
| HXPLVL1    | DDS_PF  | HXPL      | 49   |
| HXPLVL2    | DDS_PF  | HXPL      | 52   |
| HXPLVL3    | DDS_PF  | HXPL      | 52   |
| HXPLVL4    | DDS_PF  | HXPL      | 52   |
| HXPLVL5    | DDS_PF  | HXPL      | 55   |
| HXPLVL6    | DDS_PF  | HXPL      | 321  |
| HXPNSTN    | DDS_LF  | HXP       | 12   |
| HXPTABLD   | DDS_PF  | HXP       | 19   |
| HXPXMLD    | DDS_PF  | HXPX      | 19   |
| HXPXMLR    | DDS_PF  | HXPX      | 19   |
| OAPIRNK    | DDS_PF  | UNGROUPED | 80   |
| OMPMAST    | DDS_PF  | UNGROUPED | 310  |
| OXPBNFIT   | DDS_PF  | OXP       | 48   |
| OXPNSTN    | DDS_PF  | OXP       | 65   |
| HXXAPPPRF  | SQLRPGLE| HXXA      | 123  |
| XFXCNTR    | RPGLE   | XFXC      | 49   |
| XFXCYMD    | RPGLE   | XFXC      | 83   |
| XFXGETID   | RPGLE   | XFX       | 61   |
| XFXLDSC    | RPGLE   | XFXL      | 135  |
| XFXLEAP    | RPGLE   | XFXL      | 61   |
| XFXMRNROL  | RPGLE   | XFX       | 65   |
| XFXTABL    | RPGLE   | XFX       | 164  |
| CXXXMLP    | SQLRPGLE| UNGROUPED | 25   |
| HXXAPPPRFP | RPGLE   | HXXA      | 42   |
| HXXCNTRL   | RPGLE   | HXX       | 8    |
| HXXLDA     | RPGLE   | HXXL      | 53   |
| HXXLEVEL   | RPGLE   | HXXL      | 25   |
| HXXXML     | RPGLE   | HXX       | 11   |
| HABADTE    | RPGLE   | HA        | 821  |

### 1.2 Type Totals

From the aggregated metrics:

- **DDS logical files (DDS_LF):** 7
- **DDS physical files (DDS_PF):** 15
- **RPGLE programs:** 13
- **SQLRPGLE programs:** 2

This mix confirms a classic AS400 application: data persisted through DDS‑defined files, accessed and orchestrated primarily by RPGLE programs, with a small number of SQLRPGLE routines for profile and XML‑related operations.

## 2. Missing Components

The gap inventory surfaces eight missing or unresolved components referenced by the code base.

### 2.1 High‑Impact Gaps

| Name     | Type      | Referenced By | Impact |
|----------|-----------|--------------|--------|
| TAPIRNK  | FILE      | HAPIRNK      | HIGH   |
| TMPMAST  | FILE      | HMLMA        | HIGH   |
| TXPBNFIT | FILE      | HXPBN        | HIGH   |
| TXPNSTN  | FILE      | HXPNS        | HIGH   |
| CXXXMLC  | COPYBOOK  | HABAD        | HIGH   |

High‑impact gaps primarily involve base physical files for logical files (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) and a central XML copybook (CXXXMLC). Their absence means:

- **Logical file chains are incomplete.** For example, HAPIRNK depends on TAPIRNK via a PFILE relationship; without TAPIRNK, full record structures and indexes cannot be validated.
- **Master patient/plan datasets are partially opaque.** TMPMAST and TXPBNFIT/ TXPNSTN likely hold core patient master and benefits/plan information referenced through logical views; reverse engineering cannot fully trace those paths.
- **XML configuration is partially documented.** CXXXMLC is copy‑included by HABADTE, suggesting shared XML layout definitions; its absence leaves portions of HABADTE’s XML handling inferred rather than explicit.

### 2.2 Medium‑Impact Gaps

| Name      | Type    | Referenced By | Impact |
|-----------|---------|--------------|--------|
| HXHAPPPRF | PROGRAM | XFXMR        | MEDIUM |
| ****HXPXML| FILE    | HABAD        | MEDIUM |
| PRINTER   | FILE    | HABAD        | MEDIUM |

The medium‑impact gaps concern a missing application profile program (HXHAPPPRF) and XML/printing files referenced by HABADTE. These affect:

- **Profile‑driven behavior in XFXMRNROL.** The program calls HXHAPPPRF and HXXAPPPRF; missing HXHAPPPRF means only part of the profile logic is visible.
- **XML processing completeness.** ****HXPXML and PRINTER references imply external configuration or output that cannot be fully inspected.

### 2.3 Low‑Impact Gaps

No explicitly low‑impact gaps are listed; all identified gaps are either HIGH or MEDIUM. This suggests that unresolved nodes generally touch key integration points rather than peripheral utilities.

## 3. Duplicate and Reused Components

### 3.1 Reused Programs via CALL

Analysis of CALL edges highlights several programs reused by multiple callers or invoked multiple times within the same caller:

- **XFXLEAP** is called by **XFXCYMD**, centralising leap‑year or date boundary logic. This is a positive reuse pattern: date validation delegated to a specialised helper.
- **XFXCNTR** is called three times by **HABADTE**, indicating that counter or control logic is reused within different processing branches of the HABADTE workflow.
- **XFXMRNROL** is called from HABADTE and in turn calls **HXHAPPPRF** and **HXXAPPPRF**, acting as a mediator between patient management logic and application profile routines.

The repeated CALLs from HABADTE to XFXCNTR within the same program show intra‑program reuse rather than duplication of logic blocks. From a structural perspective, this is a controlled duplication of invocation, not of logic implementation.

### 3.2 Logical Files Sharing the Same PF

Several logical files share common physical files, effectively behaving as multiple views over the same underlying table:

- **HAPIRNK → TAPIRNK**
- **HMLMAST5H → TMPMAST**
- **HXLTABLD / HXLTABLP / HXLTABLS → HXPTABLD**
- **HXPBNFIT → TXPBNFIT**
- **HXPNSTN → TXPNSTN**

Within the available schema, HXLTABLD, HXLTABLP, and HXLTABLS all reference **HXPTABLD**. These patterns are functionally appropriate in DDS‑based systems but introduce complexity when modernising, because identical record layouts are exposed with different key structures and select/omit criteria. They should be rationalised into fewer, more explicit views in a modern relational design.

## 4. Dependency Analysis

### 4.1 Call, Copy, and File Relationships

The dependency graph shows a tightly coupled set of programs around **HABADTE**:

- **Inbound calls into helpers**
  - XFXCYMD → XFXLEAP (CALL)
  - XFXMRNROL → HXHAPPPRF, HXXAPPPRF (CALL)
- **HABADTE orchestrates patient management:**
  - Calls: XFXMRNROL, XFXCNTR (multiple), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL
  - Copy‑includes: HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC
  - File operations: READ HAPTRFR, READ/UPDATE/WRITE HXFXMLH, WRITE HXFXMLD, READ XFFNSTN

HABADTE clearly acts as the central dispatcher, combining multiple helper programs and copybooks to implement patient admission/date processing, XML file handling, and outcome forwarding.

### 4.2 File Access Patterns

Key file operations include:

- **XFXGETID** reads HXFXMLR and declares/uses HXPXMLR, providing an ID lookup or mapping based on XML records.
- **XFXLDSC** declares and reads the level files HXPLVL1‑6 and HXFLVL1‑6, suggesting multi‑level configuration hierarchies (e.g., plan levels or benefit tiers).
- **XFXTABL** reads XFFTABLD and related table variants, encapsulating table‑driven decision logic.
- Logical files such as HAPIRNK, HMLMAST5H, HXLTABLD/P/S, HXPBNFIT, and HXPNSTN act as filtered access paths for physical files TAPIRNK, TMPMAST, HXPTABLD, TXPBNFIT, and TXPNSTN.

This pattern is consistent with a design where core patient/plan entities are stored centrally and accessed through domain‑specific logical views.

### 4.3 Hotspot Summary

The hotspot analysis ranks programs by a composite score combining fan‑in, fan‑out, and file operations:

- **HABADTE (score 38, fan_out 13, file_ops 6, cc 152 – HIGH)**: Primary control program driving patient management workflows and XML persistence. Its high complexity and broad integrations make it the top candidate for refactoring and encapsulation.
- **XFXLDSC (score 15)**: Level description handler with multiple file declarations and reads. It is the main gateway into the HXPLVL* level structures.
- **XFXTABL (score 11)**: Table‑driven decision service; moderate hotspot due to multiple table reads.
- **XFXCNTR (score 9, fan_in 3)**: Counter/control utility reused by HABADTE; fan‑in indicates that other components may rely on its control logic.
- **XFXMRNROL / HXXAPPPRF / XFXGETID (scores 7 each)**: Integration layer programs, bridging patient management, profiles, and XML IDs.

Hotspots lower in the list (HXLTABLD/P/S, HXPBNFIT, HXPNSTN, HMLMAST5H, HAPIRNK) mainly represent logical file views with limited code, but they are structurally important because they sit on top of missing PFs.

## 5. Summary of Key Findings

### 5.1 Critical Structural Issues

Based on the high/medium gaps and orphan program list:

- **Unresolved external dependencies:** Several PFs (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) and the CXXXMLC copybook are missing. These are central to benefits, patient master data, and XML configuration, reducing confidence in end‑to‑end lineage reconstruction.
- **Profile program asymmetry:** XFXMRNROL calls both HXHAPPPRF (missing) and HXXAPPPRF (present). Downstream behaviour depends on profiles that cannot be fully inspected, complicating privilege or behaviour audits.
- **XML and print integration gaps:** References to ****HXPXML and PRINTER in HABADTE indicate external XML and printing interfaces not fully captured in the source inventory.

### 5.2 Orphan Programs and Architectural Observations

Orphan programs (no callers in the dependency graph) include:

- HXXAPPPRF (SQLRPGLE)
- XFXCNTR
- XFXCYMD
- XFXGETID
- XFXLDSC

For XFX* members, “orphan” status simply means they are not called by other programs in the current snapshot except HABADTE and the relationships captured; some may be entry points or utilities invoked via menu commands, command objects, or batch job definitions not represented in the code graph. HXXAPPPRF may be called through SQL triggers, stored procedures, or job control definitions.

Architecturally, the code base exhibits:

- **Central orchestration** by HABADTE, tightly coupling patient management, date validation, profile lookup, and XML persistence.
- **Heavy reliance on DDS logical files** for alternative keys and domain‑specific views, which must be mapped carefully when migrating to relational schemas.
- **Delegated data maintenance utilities** (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL) that can be carved out as services or micro‑modules in a modern architecture.

Collectively, these observations indicate that modernisation should begin with stabilising the missing PFs/copybooks, reducing HABADTE’s cyclomatic complexity by extracting well‑bounded services, and documenting how external entry points (menus, commands, batch jobs) invoke the XFX* and HXX* modules.
