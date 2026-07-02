# AS400 Code Analysis Report – HABADTE Application

## 1. Component Inventory

### 1.1 Member Summary

The HABADTE application portfolio consists of 37 source members spanning data definition (DDS), business logic (RPGLE/SQLRPGLE), and integration utility programs. The table below summarizes each member by name, type, subsystem, and line count as derived from the aggregated context.

| Member Name | Type     | Subsystem  | Lines |
|------------|----------|-----------|-------|
| HAPIRNK    | DDS_LF   | HAP       | 13    |
| HAPTRFR    | DDS_PF   | HAP       | 72    |
| HMLMAST5H  | DDS_LF   | UNGROUPED | 12    |
| HXLTABLD   | DDS_LF   | HXLT      | 11    |
| HXLTABLP   | DDS_LF   | HXLT      | 12    |
| HXLTABLS   | DDS_LF   | HXLT      | 12    |
| HXPBNFIT   | DDS_LF   | HXP       | 12    |
| HXPDICT    | DDS_PF   | HXP       | 6130  |
| HXPLVL1    | DDS_PF   | HXPL      | 49    |
| HXPLVL2    | DDS_PF   | HXPL      | 52    |
| HXPLVL3    | DDS_PF   | HXPL      | 52    |
| HXPLVL4    | DDS_PF   | HXPL      | 52    |
| HXPLVL5    | DDS_PF   | HXPL      | 55    |
| HXPLVL6    | DDS_PF   | HXPL      | 321   |
| HXPNSTN    | DDS_LF   | HXP       | 12    |
| HXPTABLD   | DDS_PF   | HXP       | 19    |
| HXPXMLD    | DDS_PF   | HXPX      | 19    |
| HXPXMLR    | DDS_PF   | HXPX      | 19    |
| OAPIRNK    | DDS_PF   | UNGROUPED | 80    |
| OMPMAST    | DDS_PF   | UNGROUPED | 310   |
| OXPBNFIT   | DDS_PF   | OXP       | 48    |
| OXPNSTN    | DDS_PF   | OXP       | 65    |
| HXXAPPPRF  | SQLRPGLE | HXXA      | 123   |
| XFXCNTR    | RPGLE    | XFXC      | 49    |
| XFXCYMD    | RPGLE    | XFXC      | 83    |
| XFXGETID   | RPGLE    | XFX       | 61    |
| XFXLDSC    | RPGLE    | XFXL      | 135   |
| XFXLEAP    | RPGLE    | XFXL      | 61    |
| XFXMRNROL  | RPGLE    | XFX       | 65    |
| XFXTABL    | RPGLE    | XFX       | 164   |
| CXXXMLP    | SQLRPGLE | UNGROUPED | 25    |
| HXXAPPPRFP | RPGLE    | HXXA      | 42    |
| HXXCNTRL   | RPGLE    | HXX       | 8     |
| HXXLDA     | RPGLE    | HXXL      | 53    |
| HXXLEVEL   | RPGLE    | HXXL      | 25    |
| HXXXML     | RPGLE    | HXX       | 11    |
| HABADTE    | RPGLE    | HA        | 821   |

This inventory shows a heavy concentration of DDS physical files (PF) for hierarchical level tables (HXPLVL1–6), patient master/transfer files (OMPMAST, HAPTRFR), and dictionary/reference tables (HXPDICT, HXPTABLD). The HABADTE RPGLE program is the main transactional driver with 821 lines of code.

### 1.2 Type Totals

From the aggregated type breakdown:

- DDS_LF: 7
- DDS_PF: 15
- SQLRPGLE: 2
- RPGLE: 13

The portfolio is data-centric, with roughly two-thirds of members dedicated to data definition and indexing, and the remaining members implementing supporting utilities, data validation routines, and the primary patient management workflow.

## 2. Missing Components

The gap analysis identifies several referenced-but-missing artifacts. These are grouped by impact:

### 2.1 High-Impact Gaps

| Name    | Type      | Referenced By | Impact |
|---------|-----------|--------------|--------|
| CXXXMLC | COPYBOOK  | HABAD        | HIGH   |
| TAPIRNK | FILE      | HAPIR        | HIGH   |
| TMPMAST | FILE      | HMLMA        | HIGH   |
| TXPBNFIT| FILE      | HXPBN        | HIGH   |
| TXPNSTN | FILE      | HXPNS        | HIGH   |

High-impact gaps represent structural dependencies that are essential for correct compilation or execution. CXXXMLC is a missing copybook included from HABADTE (the truncation HABAD in the manifest reflects naming normalization), so any modernization effort must reconstruct or replace this XML-related include. Similarly, TAPIRNK, TMPMAST, TXPBNFIT, and TXPNSTN are base physical files for logical views (HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN) and must be defined in the target environment to preserve data access semantics.

### 2.2 Medium-Impact Gaps

| Name      | Type    | Referenced By | Impact |
|-----------|---------|--------------|--------|
| HXHAPPPRF | PROGRAM | XFXMR        | MEDIUM |
| ****HXPXML| FILE    | HABAD        | MEDIUM |
| PRINTER   | FILE    | HABAD        | MEDIUM |

Medium-impact items affect specific flows rather than the entire application. HXHAPPPRF is a called program from XFXMRNROL, likely implementing application profile logic for HABADTE. The ****HXPXML file reference suggests a generic or wildcarded XML work file set used by HABADTE; absence of this definition will break XML persistence of patient data. The PRINTER file reference indicates an external print/spool device; in modernization this must be mapped to a report, PDF, or queue service.

### 2.3 Low-Impact Gaps

No explicitly LOW impact gaps are flagged in the aggregated context. Modernization planning should nonetheless verify that auxiliary objects (display files, messages, etc.) are present, but they are not blocking at this stage.

## 3. Duplicate and Reused Components

### 3.1 Program Reuse via CALL

Using the dependency edges, we can identify programs that are heavily reused as call targets:

- **XFXCNTR** – Target of three CALL edges from HABADTE. This indicates that counter or control logic is invoked from multiple branches within the main patient management flow (e.g., initialization, validation, and finalization phases).
- **XFXLDSC** – Called once by HABADTE and acts as a level-description reader across HXPLVL1–6, with six READ operations on level PFs. While the CALL fan-in is low, the file footprint is broad.
- **XFXTABL** – Called once by HABADTE and performs table lookups against XFFTABLD and related table variants. It has four READ edges, indicating centralization of code table translation.
- **XFXMRNROL** – Called by HABADTE and itself calls HXHAPPPRF and HXXAPPPRF. It participates in MRN rollover logic, bridging patient identifiers across versions.

These reused components should be treated as shared services in any refactoring, since their logic fan-out touches critical data structures.

### 3.2 Logical Files Sharing Base PFs

Logical files that reference the same base physical file create implicit duplication of access patterns:

- **HXLTABLD, HXLTABLP, HXLTABLS** all have PFILE_OF edges to **HXPTABLD**. They represent different logical projections (by data code, mapping, or description) over a single table-defining PF. This indicates a design pattern of multiple index paths rather than normalized relational tables.
- **HXPBNFIT** has PFILE_OF to **TXPBNFIT**, and **HXPNSTN** has PFILE_OF to **TXPNSTN**. These logical files provide indexed views over benefit and institution tables. 
- **HAPIRNK** has PFILE_OF to **TAPIRNK**, while **HMLMAST5H** has PFILE_OF to **TMPMAST**. In both cases, the PFs themselves are missing from the code set, but are critical for logical file resolution.

For modernization, these patterns suggest that many business queries depend on DDS logical file definitions. When migrating to a relational database or ORM, the different access paths must be modeled as indexes or views to avoid performance regressions.

## 4. Dependency Analysis

### 4.1 Call/Copy/Ref Graph

The dependency edges show a clear layering:

- **Top-level driver:** HABADTE
  - Calls XFXMRNROL, XFXCNTR (multiple times), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL.
  - Copies configuration and XML-related artifacts: HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC.
  - Reads and writes patient transfer and XML header/detail files: HAPTRFR, XFFNSTN, HXFXMLH (READ/UPDATE/WRITE), HXFXMLD (WRITE).

- **Intermediate services and utilities:**
  - XFXMRNROL calls HXHAPPPRF and HXXAPPPRF, indicating a bridge from MRN rollover to HXX application profile logic.
  - XFXCYMD calls XFXLEAP, encapsulating leap-year-aware date validation.
  - HXXAPPPRF copies HXXCNTRL and HXXAPPPRFP, showing a pattern of control and parameter includes for SQLRPGLE-based application profile routines.
  - XFXGETID copies HXXLDA and reads HXFXMLR, integrating logical data area configuration with XML record access.

- **Data access nodes:**
  - XFXLDSC reads HXFLVL1–6, traversing multiple level PFs.
  - XFXTABL reads XFFTABLD and related table variants, abstracting code table access.
  - HAPIRNK, HMLMAST5H, HXLTABLD/LP/LS, HXPBNFIT, HXPNSTN are logical wrappers around TAPIRNK, TMPMAST, HXPTABLD, TXPBNFIT, and TXPNSTN respectively.

### 4.2 Hotspot Programs

Hotspots combine fan-in, fan-out, and file operations:

- **HABADTE (score 38, fan_out 13, file_ops 6)** – Primary patient-management orchestrator. It integrates MRN rollover, date validation, table lookups, XML persistence, and transfer/notification file handling. Its high cyclomatic complexity (152, band HIGH) confirms that this is the main candidate for modularization.
- **XFXLDSC (score 15, fan_in 1, file_ops 6)** – Level description service, reading six PFs and likely mapping patient or account levels to human-readable descriptions.
- **XFXTABL (score 11, fan_in 1, file_ops 4)** – Central code table lookup program.
- **XFXCNTR (score 9, fan_in 3, file_ops 0)** – Pure logic utility controlling counters and branch conditions, invoked from HABADTE multiple times.
- **XFXMRNROL / HXXAPPPRF (score 7 each)** – MRN rollover and application profile logic, forming a pair of services important for patient identifier management.
- **XFXGETID (score 7)** – ID assignment logic tied to XML header records.

These hotspots should be isolated into service components with clear interfaces in a modern architecture (e.g., PatientTransferService, MRNRolloverService, CodeTableService), reducing coupling with HABADTE.

## 5. Summary of Key Findings

### 5.1 Critical Issues from High/Medium Gaps

Key high/medium gaps (from the filtered set) are:

| Artifact   | Type      | Impact | Referenced By |
|-----------|-----------|--------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH   | HABADTE       |
| HXHAPPPRF | PROGRAM   | MEDIUM | XFXMRNROL     |
| TAPIRNK   | FILE      | HIGH   | HAPIRNK       |

CXXXMLC is essential to HABADTE’s XML handling; its absence means that parts of the XML marshalling/unmarshalling logic are unknown. HXHAPPPRF affects MRN rollover behavior, and missing TAPIRNK compromises key index access for IRN/transfer reporting.

Other high-impact PF gaps (TMPMAST, TXPBNFIT, TXPNSTN) and medium-impact file gaps (****HXPXML, PRINTER) further indicate that the current source set is incomplete relative to a running production environment. Migration teams must source these definitions from the AS400 or configuration repositories.

### 5.2 Orphan Programs and Architectural Observations

Orphan programs (no inbound CALL edges) include:

- HXXAPPPRF (SQLRPGLE)
- XFXCNTR (RPGLE)
- XFXCYMD (RPGLE)
- XFXGETID (RPGLE)
- XFXLDSC (RPGLE)

The classification as "orphan" is relative to call edges, but the dependency graph shows that HABADTE calls several of these directly. This discrepancy likely reflects the scope of the dependency extraction or naming normalization. Functionally, these programs act as shared utilities even if some call edges are implicit or via command-level invocation.

Architecturally, HABADTE is a monolithic driver that orchestrates multiple utility modules for validation (XFXCYMD, XFXCNTR), lookup (XFXLDSC, XFXTABL), and integration (XFXGETID, XFXMRNROL, HXXAPPPRF). Data is stored in a set of PFs with logical projection LFs, and XML files (HXFXMLH/D/R) provide a semi-structured layer for outbound interfaces.

From a modernization perspective, the key recommendations are:

- Break down HABADTE into smaller services aligned with hotspots and dependency clusters.
- Recreate missing PF, LF, copybooks, and programs before attempting automated code transformation.
- Map logical-files-based access patterns to indexed views or repositories in the target platform.
- Treat XML-related artifacts (CXXXMLP/CXXXMLC, HXFXML*) and printer file PRINTER as external interface boundaries for integration via APIs or messaging.
