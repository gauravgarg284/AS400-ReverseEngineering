# AS400 Code Analysis Report

PIPELINE_RUN-ID: 202607011449

## 1. Component Inventory

### 1.1 Member Inventory

The analyzed codebase consists of 37 source members spanning physical files (PFs), logical files (LFs), RPG programs, and SQLRPGLE programs. The table below summarizes each member, its type, owning subsystem, and line count as derived from the reverse‑engineered source manifest.

| Member Name | Type    | Subsystem | Lines |
|-------------|---------|-----------|-------|
| HAPIRNK     | DDS_LF  | HAP       | 13    |
| HAPTRFR     | DDS_PF  | HAP       | 72    |
| HMLMAST5H   | DDS_LF  | UNGROUPED | 12    |
| HXLTABLD    | DDS_LF  | HXLT      | 11    |
| HXLTABLP    | DDS_LF  | HXLT      | 12    |
| HXLTABLS    | DDS_LF  | HXLT      | 12    |
| HXPBNFIT    | DDS_LF  | HXP       | 12    |
| HXPDICT     | DDS_PF  | HXP       | 6130  |
| HXPLVL1     | DDS_PF  | HXPL      | 49    |
| HXPLVL2     | DDS_PF  | HXPL      | 52    |
| HXPLVL3     | DDS_PF  | HXPL      | 52    |
| HXPLVL4     | DDS_PF  | HXPL      | 52    |
| HXPLVL5     | DDS_PF  | HXPL      | 55    |
| HXPLVL6     | DDS_PF  | HXPL      | 321   |
| HXPNSTN     | DDS_LF  | HXP       | 12    |
| HXPTABLD    | DDS_PF  | HXP       | 19    |
| HXPXMLD     | DDS_PF  | HXPX      | 19    |
| HXPXMLR     | DDS_PF  | HXPX      | 19    |
| OAPIRNK     | DDS_PF  | UNGROUPED | 80    |
| OMPMAST     | DDS_PF  | UNGROUPED | 310   |
| OXPBNFIT    | DDS_PF  | OXP       | 48    |
| OXPNSTN     | DDS_PF  | OXP       | 65    |
| HXXAPPPRF   | SQLRPGLE| HXXA      | 123   |
| XFXCNTR     | RPGLE   | XFXC      | 49    |
| XFXCYMD     | RPGLE   | XFXC      | 83    |
| XFXGETID    | RPGLE   | XFX       | 61    |
| XFXLDSC     | RPGLE   | XFXL      | 135   |
| XFXLEAP     | RPGLE   | XFXL      | 61    |
| XFXMRNROL   | RPGLE   | XFX       | 65    |
| XFXTABL     | RPGLE   | XFX       | 164   |
| CXXXMLP     | SQLRPGLE| UNGROUPED | 25    |
| HXXAPPPRFP  | RPGLE   | HXXA      | 42    |
| HXXCNTRL    | RPGLE   | HXX       | 8     |
| HXXLDA      | RPGLE   | HXXL      | 53    |
| HXXLEVEL    | RPGLE   | HXXL      | 25    |
| HXXXML      | RPGLE   | HXX       | 11    |
| HABADTE     | RPGLE   | HA        | 821   |

### 1.2 Component Types and Counts

The source members are distributed across the following types:

- DDS logical files (DDS_LF): 7
- DDS physical files (DDS_PF): 15
- SQLRPGLE programs: 2
- RPGLE programs: 13

The dataset contains 9,153 total lines of code across all members, with a completeness of 82.2% relative to the expected portfolio. The HXPDICT physical file is the largest definition with 6,130 lines, indicating a central dictionary‑style structure used pervasively by other components.

Subsystem distribution shows that:

- The HXPL* PFs (HXPLVL1–HXPLVL6) form a layered configuration hierarchy within the HXPL subsystem.
- HXP* and HXPNSTN/HXPBNFIT members support benefits and station‑level reference data in the HXP/OXP subsystems.
- XFX* programs represent shared utility and control flow services.
- HABADTE is the primary HA subsystem driver program orchestrating multiple utilities and file operations.

## 2. Missing Components

The reverse‑engineering process identified several referenced but missing components. These appear either as copybooks, external programs, or physical files used as base PFs for logical files. The gaps are grouped by impact level.

### 2.1 High‑Impact Gaps

| Name    | Type      | Referenced By | Impact |
|---------|-----------|--------------|--------|
| CXXXMLC | COPYBOOK  | HABAD        | HIGH   |
| TAPIRNK | FILE      | HAPIR        | HIGH   |
| TMPMAST | FILE      | HMLMA        | HIGH   |
| TXPBNFIT| FILE      | HXPBN        | HIGH   |
| TXPNSTN | FILE      | HXPNS        | HIGH   |

High‑impact gaps correspond to objects that are either critical copybooks or base physical files. For example, CXXXMLC is a copybook included by HABAD/HABADTE; its absence means XML‑related record layouts or common prototypes are unknown. TAPIRNK and TMPMAST are base PFs for the HAPIRNK and HMLMAST5H logical files; without them, the underlying record structures and certain selection logic cannot be fully reconstructed. Similarly, TXPBNFIT and TXPNSTN are base PFs for benefits and station‑level logical files, affecting completeness of benefits and station reference data.

### 2.2 Medium‑Impact Gaps

| Name      | Type   | Referenced By | Impact |
|-----------|--------|--------------|--------|
| HXHAPPPRF | PROGRAM| XFXMR        | MEDIUM |
| ****HXPXML| FILE   | HABAD        | MEDIUM |
| PRINTER   | FILE   | HABAD        | MEDIUM |

Medium‑impact gaps include a missing HXHAPPPRF program called by XFXMRNROL, and file objects such as ****HXPXML and PRINTER used from HABADTE. The absence of HXHAPPPRF prevents full tracing of application‑profile logic invoked via XFXMRNROL. The printer and XML‑host files appear to be output or control‑file destinations; while not blocking logical understanding, they limit precise documentation of external interfaces and report streams.

### 2.3 Low‑Impact Gaps

No gaps were explicitly tagged as LOW impact in the aggregated context. All identified gaps are either HIGH or MEDIUM severity.

## 3. Duplicate and Reused Components

This section focuses on reuse patterns derived from the dependency edges. We examine both program reuse (targets of multiple CALL edges) and LF‑to‑PF relationships that share a common base PF.

### 3.1 Reused Programs and Services

Based on CALL‑type edges, several programs are reused by multiple callers:

- **XFXCNTR** is called three times from HABADTE (multiple CALL edges to XFXCNTR), indicating it implements a reusable counter or control‑flow utility. Its fan‑in score of 3 in the hotspot list confirms that multiple contexts within HABADTE rely on this utility.
- **XFXMRNROL** is called from HABADTE and in turn calls HXHAPPPRF and HXXAPPPRF. This program acts as a mediator for MRN (medical record number) rollover or normalization logic.
- **XFXLDSC** and **XFXTABL** are each called once from HABADTE but perform multiple file operations. XFXLDSC reads across all HXFLVL* PFs, centralizing level‑table lookups. XFXTABL reads from multiple XFFTABL* tables, encapsulating dictionary or code‑table access.
- **HXXAPPPRF** is called by XFXMRNROL and performs COPY operations on HXXCNTRL and HXXAPPPRFP, suggesting a layered approach to application profile control.

These reused programs form a shared services layer. While there are no exact duplicate program objects, reuse of a small set of utilities introduces tight coupling between HABADTE and the XFX/HXX subsystems.

### 3.2 Logical Files Sharing Common Physical Files

The dependency graph shows several logical files (LFs) referencing the same base PFs via PFILE_OF edges:

- **HXLTABLD, HXLTABLP, HXLTABLS** all point to **HXPTABLD**. Each LF exposes different keyed views over the same table (data codes, description mappings, and selection sets). This pattern is healthy reuse of a shared code‑table PF.
- **HXPBNFIT** is based on **TXPBNFIT**, and **HXPNSTN** is based on **TXPNSTN**. Both base PFs are missing, but multiple LFs suggest heavily reused benefits and station reference data.
- **HAPIRNK** is based on **TAPIRNK**, and **HMLMAST5H** on **TMPMAST**, representing logical slices over inbound ranking and master files.

No direct evidence of structurally duplicated PFs exists; rather, reuse is expressed through LF overlays. However, the missing base PFs mean physical duplication cannot be ruled out.

## 4. Dependency Analysis

### 4.1 Call Chain Overview

The dependency edges show a layered call structure:

- **HABADTE** is the primary controller program with outbound CALLs to:
  - XFXMRNROL
  - XFXCNTR (three separate call sites)
  - XFXLDSC
  - XFXCYMD
  - XFXGETID
  - XFXTABL

- **XFXMRNROL** calls:
  - HXHAPPPRF (missing program)
  - HXXAPPPRF

- **XFXCYMD** calls XFXLEAP to perform leap‑year‑related date validation.

This results in a call chain where HABADTE orchestrates the flow and delegates specialized tasks to XFX* utilities, which in turn may use HXX* application profile components.

### 4.2 Copy and Include Dependencies

Copybook and include relationships are captured via COPY edges:

- HXXAPPPRF copies in **HXXCNTRL** and **HXXAPPPRFP**, indicating common control blocks and possibly precompiled SQL stubs.
- XFXGETID copies **HXXLDA**, a low‑level data area definition containing shared identifiers.
- HABADTE copies several support modules:
  - HXXLDA (shared LDA for IDs)
  - HXXLEVEL (level table access or environment setup)
  - HXXXML (XML‑specific configuration)
  - CXXXMLP (SQLRPGLE copy for XML processing)
  - CXXXMLC (missing copybook, likely XML column layout)

The copy dependencies highlight a design that factors environment setup, XML handling, and ID storage into reusable include members. The missing CXXXMLC limits full reconstruction of the XML layout, but the pattern itself is clear.

### 4.3 File Access Patterns

READ and WRITE edges describe how programs interact with database and XML holding tables:

- **XFXGETID** reads from **HXFXMLR**, using XML result or request records to allocate or look up IDs.
- **XFXLDSC** reads across **HXFLVL1–HXFLVL6**, consuming level hierarchy data from the corresponding PFs.
- **XFXTABL** reads from **XFFTABLD** and related tables (XFFTABL2–XFFTABL4), acting as a generic table lookup engine.
- **HABADTE** reads from **HAPTRFR**, **XFFNSTN**, and **HXFXMLH**; it updates and writes **HXFXMLH** and writes **HXFXMLD**, indicating a workflow where transfer records and station data drive the creation of XML header and detail records.

These access patterns establish HABADTE as the batch driver that transforms file‑based transfer data into XML payloads, using shared XFX services for validation and lookup.

### 4.4 Hotspot Summary

Hotspot scores from the aggregated context quantify complexity and centrality:

- **HABADTE** (score 38, fan_out 13, file_ops 6) is the dominant orchestrator. It resides in the HA subsystem and coordinates multiple service calls and file operations.
- **XFXLDSC** (score 15, fan_in 1, file_ops 6) is a key level‑table reader, indicating frequent use of multi‑level configuration.
- **XFXTABL** (score 11, fan_in 1, file_ops 4) represents shared code table access.
- **XFXCNTR** (score 9, fan_in 3) is a control utility with multiple inbound call sites.
- **XFXMRNROL** and **HXXAPPPRF** both have moderate scores and act as intermediaries for MRN rollover and application profile configuration.

The hotspot profile shows a classic pattern: one batch driver, several mid‑tier services, and numerous low‑complexity helper programs and file definitions.

## 5. Summary of Key Findings

### 5.1 Critical Gaps and Architectural Risks

Drawing from the high and medium‑impact gaps:

- The missing **CXXXMLC** copybook is a critical structural gap. It likely contains XML column layouts or prototypes used by HABADTE. Without it, downstream modernization must infer field‑level XML mappings from runtime behavior or from partner systems.
- Missing PFs **TAPIRNK**, **TMPMAST**, **TXPBNFIT**, and **TXPNSTN** are high‑impact for data modeling. Logical files HAPIRNK, HMLMAST5H, HXPBNFIT, and HXPNSTN cannot be fully defined without them. These PFs should be prioritized for retrieval or reconstruction from backup.
- The missing **HXHAPPPRF** program reduces visibility into application‑profile logic invoked through XFXMRNROL. This presents a moderate risk to understanding MRN‑related behavior and profile‑driven branching.
- Printer and XML host file definitions (****HXPXML, PRINTER) are absent, obscuring details of outbound reporting and external XML storage. This affects interface documentation more than internal logic.

### 5.2 Orphan Programs and Unused Assets

The following programs have no callers in the current dependency graph:

- HXXAPPPRF (SQLRPGLE)
- XFXCNTR (RPGLE)
- XFXCYMD (RPGLE)
- XFXGETID (RPGLE)
- XFXLDSC (RPGLE)

In practice, these are not truly unused; they are invoked by HABADTE or other components, but the static analysis treats them as "orphan" when considering only entry points from outside the extracted scope. For modernization, these should be treated as internal service classes rather than dead code. However, their lack of direct external callers means they can often be encapsulated behind a service facade.

### 5.3 Architectural Observations

- The system follows a **hub‑and‑spoke** architecture with HABADTE as the hub, delegating to a family of XFX* utilities and HXX* control components.
- Database access is centralized in utility programs (XFXLDSC, XFXTABL) rather than scattered across all business modules, which is favorable for service extraction.
- Logical files are heavily used to provide alternate keys and sliced views; base PF definitions are critical and must be preserved or reconstituted.
- Overall cyclomatic complexity is low across most programs, except HABADTE, which has a high complexity (cc=152, band=HIGH). This suggests HABADTE should be a prime candidate for refactoring into smaller services.
- No explicit tech‑debt findings were recorded (total_findings = 0), but the structural gaps and single high‑complexity driver program represent implicit modernization risk.

In summary, the AS400 codebase exhibits a well‑structured service and data layer around a complex batch driver. Remediation should prioritize recovering missing base PFs and copybooks, documenting the MRN rollover and XML mapping flows, and decomposing HABADTE into modular services for future platforms.
