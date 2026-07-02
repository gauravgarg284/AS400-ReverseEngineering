# AS400 Code Analysis Report

## 1. Component Inventory

### 1.1 Member Inventory

The analyzed AS400 codebase contains 37 source members spanning DDS physical and logical files, traditional RPGLE programs, and SQLRPGLE programs. The table below summarizes each member with its type, functional subsystem, and line count.

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

### 1.2 Member Type Distribution

The member inventory is dominated by DDS-based database definitions and RPG programs:

- **DDS_LF**: 7 logical file members
- **DDS_PF**: 15 physical file members
- **SQLRPGLE**: 2 members
- **RPGLE**: 13 members

This distribution indicates a traditional AS400 solution with a moderately normalized database layer (22 DDS-based files) and a procedural business logic layer. The presence of SQLRPGLE members (HXXAPPPRF, CXXXMLP) suggests incremental modernization where targeted routines have been refactored to use embedded SQL while the majority of logic remains in RPGLE.

The largest object by lines is **HXPDICT** (6130 lines), which functions as a high-density dictionary or master data file. The most complex program by size is **HABADTE** (821 lines), which appears to be the central batch driver coordinating multiple utility and service routines.

## 2. Missing Components

The gap analysis identifies several referenced-but-missing components. These represent incomplete inventory, broken dependency chains, or external interfaces that are not part of the current source export.

### 2.1 High-Impact Gaps

High-impact gaps are files or copybooks that the existing programs depend on for core behavior or data access:

- **CXXXMLC** (COPYBOOK) – referenced by HABAD. Missing copybook likely contains XML parsing or message layout definitions. Its absence introduces risk for any XML-based integration handled by HABADTE.
- **TAPIRNK** (FILE) – referenced by HAPIR. This is the physical file behind the HAPIRNK logical file; without it, the logical file definition cannot be resolved to actual data storage.
- **TMPMAST** (FILE) – referenced by HMLMA. Acts as the master patient or member file backing the HMLMAST5H logical file.
- **TXPBNFIT** (FILE) – referenced by HXPBN. Physical file supporting benefit-related logical file HXPBNFIT.
- **TXPNSTN** (FILE) – referenced by HXPNS. Physical file backing HXPNSTN, presumably storing plan or institution codes.

These missing PFs are core data structures. Their absence impairs reconstruction of primary keys, indexes, and integrity constraints for key domains such as transfers (HAPTRFR), benefits (OXPBNFIT, HXPBNFIT), and member demographics (OMPMAST, TMPMAST).

### 2.2 Medium-Impact Gaps

Medium-impact gaps relate to auxiliary programs or files that enrich functionality but may not be strictly required for minimal operation:

- **HXHAPPPRF** (PROGRAM) – referenced by XFXMR. This program is called from XFXMRNROL to perform application profiling or preference handling. Its absence limits understanding of how MRN-related roles or profiles are computed.
- ******HXPXML** (FILE) – referenced by HABAD. Likely a variant of the HXPXMLD/HXPXMLR family, and used for XML-based configuration or message payload storage.
- **PRINTER** (FILE) – referenced by HABAD. Represents a print output file or device definition used for spool/report generation.

These medium gaps reduce visibility into reporting, XML handling, and application profiling, but they can be stubbed or proxied in a modernization scenario while the main transactional paths are converted.

### 2.3 Low-Impact Gaps

No explicit low-impact gaps are recorded; all identified gaps are classified as HIGH or MEDIUM impact. This suggests the harvesting focused on structural and runtime-critical dependencies rather than optional utilities.

## 3. Duplicate and Reused Components

The dependency edges reveal several instances of reuse where a component acts as a common target in multiple calls or logical references.

### 3.1 Reused Programs (CALL Targets)

- **XFXCNTR** – Called multiple times by HABADTE. There are three distinct CALL edges from HABADTE to XFXCNTR, indicating that this counter or controller routine is reused across several phases of the batch process.
- **XFXLEAP** – Called by XFXCYMD to handle leap-year logic. While there is a single recorded call, the pairing indicates that XFXLEAP serves as a specialized utility reused whenever calendar calculations require leap-year consideration.
- **XFXMRNROL** – Called by HABADTE, and in turn calls HXHAPPPRF and HXXAPPPRF. This program is a reusable MRN role resolver, acting as a hub for patient/member role logic.

### 3.2 Logical Files Sharing Physical Bases

Multiple logical files project different views of the same underlying physical files:

- **HAPIRNK** (LF) relies on **TAPIRNK** (PF) via a `PFILE_OF` relationship.
- **HMLMAST5H** (LF) relies on **TMPMAST** (PF).
- **HXLTABLD**, **HXLTABLP**, **HXLTABLS** (LFs) all rely on **HXPTABLD** (PF), providing different key sequences (`XFDMAP`, `XFDLDS`, `XFDSDS`) over the same code table.
- **HXPBNFIT** (LF) relies on **TXPBNFIT** (PF), while **OXPBNFIT** is a PF with similar record format XFFBNFIT, suggesting production vs operational variants.
- **HXPNSTN** (LF) relies on **TXPNSTN** (PF), while **OXPNSTN** is another PF with XFFNSTN format.

This reuse pattern shows a normalized dictionary design where PFs hold master data and LFs expose alternative query patterns. In modernization, these logical views should be mapped to SQL indexes or views while preserving the domain semantics encoded in key fields.

## 4. Dependency Analysis

### 4.1 Call Chain Overview

The call-based dependencies define a clear orchestration hierarchy:

- **Calendar and Leap-Year Services**
  - `XFXCYMD -> XFXLEAP` (CALL) – Date validation and leap-year determination.

- **MRN Role and Application Profiling**
  - `XFXMRNROL -> HXHAPPPRF` (CALL) – Application profile resolution (missing program).
  - `XFXMRNROL -> HXXAPPPRF` (CALL) – SQLRPGLE-based profile logic using modern data access.

- **Central Batch Driver HABADTE**
  - Calls: XFXMRNROL, XFXCNTR (three times), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL.
  - Copies in: HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC.

HABADTE is the dominant orchestrator, with **13 fan-out edges** and multiple file operations, making it the primary batch service.

### 4.2 File Access and Operations

Key file access patterns include:

- **READ Operations**
  - XFXGETID reads **HXFXMLR**, likely to obtain XML-based identifiers or routing data.
  - XFXLDSC reads each of the HXPLVL1–HXPLVL6 PFs, aggregating multi-level configuration or benefits tiers.
  - XFXTABL reads XFFTABLD and its variants, acting as a generalized table lookup engine.
  - HABADTE reads **HAPTRFR**, **XFFNSTN**, and **HXFXMLH**, touching transfer records, institution codes, and XML header data.

- **WRITE / UPDATE Operations**
  - HABADTE updates and writes **HXFXMLH**, suggesting that it generates or updates XML header/control records.
  - HABADTE writes **HXFXMLD**, implying creation of XML detail segments as part of outbound messaging.

These patterns mark HABADTE as a batch integration engine: it reads transactional and reference data, applies MRN and calendar rules via utility programs, and emits XML-based messages.

### 4.3 Hotspot Summary

The dependency hotspots quantify structural complexity:

- **HABADTE** – score 38, fan_out 13, file_ops 6. Central batch/integration service, high change risk.
- **XFXLDSC** – score 15, fan_in 1, file_ops 6. Level descriptor service reading multiple PFs; moderate complexity.
- **XFXTABL** – score 11, fan_in 1, file_ops 4. Generic table lookup service.
- **XFXCNTR** – score 9, fan_in 3. Shared counter/controller utility.
- **XFXMRNROL**, **XFXGETID**, **HXXAPPPRF** – scores around 7 with mixed fan-in/fan-out, operating as service methods within the MRN/profile domain.

In modernization planning, HABADTE and the XFX* utilities should be prioritized for refactoring into discrete services with clear interfaces, as they represent core orchestration logic and cross-cutting concerns.

## 5. Summary of Key Findings

### 5.1 Critical Structural Issues (High/Medium Gaps)

Using the high_medium_gaps subset:

- **CXXXMLC (COPYBOOK, HIGH)** – missing XML layout definitions. Risk: undocumented message schemas and brittle integrations.
- **HXHAPPPRF (PROGRAM, MEDIUM)** – missing profiling routine. Risk: incomplete understanding of MRN role decisions; potential behavioral differences after migration.
- **TAPIRNK (FILE, HIGH)** – missing PF for rank or transfer records. Risk: inability to fully reconstruct the data model behind HAPIRNK and OAPIRNK.

Combined with other high-impact missing PFs (TMPMAST, TXPBNFIT, TXPNSTN), these gaps indicate that the current export is structurally strong but lacks several primary physical tables from production.

### 5.2 Orphan Programs

The orphan_programs list identifies routines with no recorded callers:

- HXXAPPPRF (SQLRPGLE)
- XFXCNTR (RPGLE)
- XFXCYMD (RPGLE)
- XFXGETID (RPGLE)
- XFXLDSC (RPGLE)

Although dependency analysis shows some of these programs being called, they appear as orphans in the harvested graph, suggesting either incomplete or indirect call-path detection (e.g., via command objects or CL programs not included in the scan). For modernization, these programs should be treated as **service utilities** with externally triggered entry points and reconciled against job schedules and CL command lists.

### 5.3 Architectural Observations

- The solution is **file-centric**, with heavy use of DDS PF/LF objects and explicit READ/WRITE operations. Logical files are used extensively to provide alternate views and access paths over common PFs.
- **HABADTE** acts as a monolithic batch orchestrator, coordinating MRN, calendar, and table lookup services. Its high cyclomatic complexity (152) and dense dependency footprint make it a candidate for decomposition into smaller, domain-focused services.
- The presence of **XML-related programs and files** (HXPXMLD, HXPXMLR, HXFXMLH, CXXXMLP, CXXXMLC) suggests an integration layer that emits XML payloads, likely for downstream systems or external interfaces.
- Multiple PFs contain **PHI-related fields** (MRN, account number, SSN, phone, name), confirming that privacy and access control must be central to any modernization design.

Overall, the codebase is structurally coherent, with clear separation between core dictionary data (HXPDICT, HXPTABLD, HXPLVL*), transactional records (HAPTRFR, OMPMAST, OAPIRNK, OXPBNFIT, OXPNSTN), and service utilities (XFX*, HXX*). The main technical risk lies in the incomplete set of physical files and copybooks, and in the complexity and opacity of the central HABADTE batch driver.