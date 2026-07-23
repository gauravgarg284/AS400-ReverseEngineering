# AS400 Code Analysis Report – HABADTE Active Patients by Date and Time

## 1. Component Inventory

### 1.1 Member Inventory

The HABADTE project contains 41 scanned members spanning patient management, reference data, level hierarchy, XML reporting, and various support utilities. The table below summarizes each member by name, type, subsystem, and approximate line count based on the aggregated manifest.

| Member Name | Type    | Subsystem  | Lines |
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
| TAPIRNK     | DDS_PF  | UNGROUPED | 167   |
| TMPMAST     | DDS_PF  | UNGROUPED | 688   |
| TXPBNFIT    | DDS_PF  | TXP       | 164   |
| TXPNSTN     | DDS_PF  | TXP       | 116   |
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

### 1.2 Source Type Totals

Source type distribution for the HABADTE codebase:

- DDS Logical Files (DDS_LF): 7
- DDS Physical Files (DDS_PF): 19
- SQLRPGLE Programs: 2
- RPGLE Programs: 13

This composition reflects a traditional AS400 application where DDS-based physical and logical files serve as the data layer, while RPGLE and SQLRPGLE programs implement business logic, reporting, XML export, and utility services.

The HABADTE main program (RPGLE, 821 lines) is the primary orchestrator of the "Active Patients by Date and Time" census report, integrating patient master data (OMPMAST, TMPMAST), transfer history (HAPTRFR), nursing station metadata (OXPNSTN/TXPNSTN and HXPNSTN), and XML reporting structures (HXPXMLD/HXPXMLR) via XFX* utility programs.

## 2. Missing Components

The aggregated gap report identifies four missing or unresolved components:

1. **CXXXMLC (COPYBOOK, impact=HIGH, referenced_by=HABAD)**  
   A copybook providing XML configuration or common definitions (e.g., constants, table metadata, or service interfaces) for XML reporting flows. It is referenced as `/copy copysrc,cxxxmlc` in HABADTE but the source member was not found in the scanned set.

2. **HXHAPPPRF (PROGRAM, impact=MEDIUM, referenced_by=XFXMR)**  
   A program likely responsible for application profile or preference management. XFXMRNROL calls HXHAPPPRF to obtain configuration necessary for MRN rollup logic.

3. ******HXPXML (FILE, impact=MEDIUM, referenced_by=HABAD)**  
   A file (or collection of XML control files) used by HABADTE for XML reporting. The wildcard-like naming suggests a group of related XML header/detail files (e.g., HXPXMLH/HXPXMLD/HXPXMLR); one of these is referenced but not fully resolved in the manifest.

4. **PRINTER (FILE, impact=MEDIUM, referenced_by=HABAD)**  
   A printer device file used by HABADTE for traditional spool-based reporting. The DDS or device description for this printer file is not present in the scanned source.

### 2.1 Implications of HIGH-Impact Gaps

**CXXXMLC (COPYBOOK)** is classified as HIGH impact because it feeds the core XML reporting flow used by HABADTE. Missing XML configuration or field mappings can break downstream consumers that rely on structured XML output, such as external reporting engines or integration platforms. Without the copybook, modernization teams cannot fully reconstruct:

- XML tag layout and identifiers used by HABADTE.  
- Table and column mappings for patient census data in XML.  
- Any environment-specific options (e.g., facility, user, or run-time flags) that alter XML shape.

Remediation requires recovering this copybook from other environments or inferring its structure from usage patterns in HABADTE and CXXXMLP.

### 2.2 Implications of MEDIUM-Impact Gaps

**HXHAPPPRF (PROGRAM)** mediates MRN rollup behavior via XFXMRNROL. Its absence limits understanding of how corporate structure and application profiles influence MRN aggregation. While HABADTE can still be analyzed using XFXMRNROL’s visible logic, full behavior (e.g., facility-specific rollup rules) cannot be guaranteed.

******HXPXML (FILE)** indicates that at least one XML control file is missing, even though HXPXMLD and HXPXMLR are present. The missing file likely stores XML headers or definitions (e.g., HXFXMLH) referenced for update/write in HABADTE. This partially obscures:

- How XML header records are versioned, sequenced, and associated with detail records.  
- The full lifecycle of XML reports (creation, update, archival).

**PRINTER (FILE)** is a MEDIUM-impact gap because spool output is secondary to the primary XML reporting path but still important for legacy operations. Without device definitions, the modernization team needs to:

- Confirm printer line widths, fonts, and control sequences.  
- Infer the expected layout solely from RPG O-specs in HABADTE.

These MEDIUM-impact gaps collectively affect documentation accuracy for deployment, environment configuration, and integration points, but do not prevent understanding the core census business logic.

## 3. Duplicate and Reused Components

Reused components are visible in the dependency edges, particularly CALL edges where a single target program is invoked multiple times:

- **XFXCNTR** is called three times by HABADTE (`HABADTE -> XFXCNTR` edges appear three times). This program is a reusable text-centering utility used to center report headings (report title, hospital name, census date strings) on spool output and possibly in XML metadata.
- **XFXMRNROL** is called once by HABADTE and itself calls HXHAPPPRF and HXXAPPPRF. It encapsulates MRN rollup logic shared across different reporting contexts.
- **XFXLDSC** is called from HABADTE and performs level description lookup against the corporate level hierarchy (HXPLVL1–HXPLVL6 and their logical/physical variants). The levels and descriptions are reused across multiple hierarchy-aware workflows.
- **XFXCYMD** is called by HABADTE and by XFXCYMD itself uses XFXLEAP, forming a reusable date-conversion engine (CCYYMMDD ↔ MMDDCCYY) used across programs for canonical date formatting.
- **XFXTABL** is called from HABADTE as a generic code/description lookup service, reused for resolving room class description and determining whether a room placement should be treated as leave or active census.

Logical file reuse is also evident:

- **HXPTABLD** is shared by three logical files: HXLTABLD, HXLTABLP, HXLTABLS. Each LF defines a different key (data code + mapping code, data code + long description, data code + short description), but all consume the same base PF. This pattern centralizes code tables while exposing multiple access paths optimized for different use cases.
- **TXPBNFIT** is the physical source for HXPBNFIT (LF), and **TXPNSTN** similarly backs HXPNSTN. These LFs act as view-like overlays for benefit and nursing station reference data.

This reuse indicates a clear separation of concerns: HABADTE relies heavily on shared utilities (XFX*) and shared reference tables (HXPTABLD, TXPBNFIT, TXPNSTN) rather than embedding logic directly. For modernization, these reused components are prime candidates for refactoring into discrete services or microservices.

## 4. Dependency Analysis

### 4.1 Call Chains

The call graph reveals HABADTE as a top-level driver:

- **HABADTE** → **XFXMRNROL** → (**HXHAPPPRF**, **HXXAPPPRF**)  
  HABADTE delegates MRN rollup configuration to XFXMRNROL, which in turn uses profile programs.

- **HABADTE** → **XFXCNTR** (called three times)  
  Used to center headings like "ACTIVE ACCTS - ADMIT DATE AND TIME", hospital title, and census date strings.

- **HABADTE** → **XFXLDSC**  
  Looks up corporate level descriptions, populating `lvldsc` for facility/hospital name composition.

- **HABADTE** → **XFXCYMD**  
  Converts census date and run date between CCYYMMDD and printable formats.

- **HABADTE** → **XFXGETID**  
  Retrieves XML report element IDs and data fragments (e.g., `PF`, `RN`, `RNF`, `RNX`, etc.) used to construct XML report sections.

- **HABADTE** → **XFXTABL**  
  Resolves room class descriptions and flags accounts as leave based on mapping code in the HXPDICT/HXPTABLD tables.

- **HABADTE** → copybooks **HXXLDA**, **HXXLEVEL**, **HXXXML**, **CXXXMLP**, **CXXXMLC**  
  These copybooks load common structures for data areas, level configuration, and XML reporting.

### 4.2 File Operations and Hotspots

Key file operations driven by HABADTE include:

- **READ HAPTRFR** using the transfer key (AFLVL6, AFACCT) to identify room assignments within the selected census date range.  
- **READ XFFNSTN** via HXPNSTN to retrieve nursing station names for the patient’s current room.  
- **READ/UPDATE/WRITE HXFXMLH** and **WRITE HXFXMLD** to persist XML header and detail segments for the census report.

Hotspot analysis (score combining fan-in, fan-out, and file operations) shows:

- **HABADTE**: score 38, fan_in 0, fan_out 13, file_ops 6. It is the main batch/report driver and a critical modernization target.  
- **XFXLDSC**: score 15, fan_in 1, fan_out 0, file_ops 6. Core level hierarchy service reading HXPLVL1–HXPLVL6 and FLVL variants.  
- **XFXTABL**: score 11, fan_in 1, fan_out 0, file_ops 4. Centralized code/description table lookup service.  
- **XFXCNTR**: score 9, fan_in 3, fan_out 0, file_ops 0. Pure computation utility (no database IO), ideal for stateless service extraction.  
- **XFXMRNROL** and **HXXAPPPRF**: scores 7, with calls to profile programs and no direct file IO, forming a configuration lookup tier.

Risk assessment:

- HABADTE’s high complexity (cc=152, band=HIGH) and high fan-out make it a high-risk modernization candidate; changes in its call interfaces or file access patterns propagate broadly.  
- Utility programs (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL) have low complexity but high reuse, so they are low risk individually yet high impact collectively. They should be carefully encapsulated with stable APIs.

## 5. Summary of Key Findings

Architecturally, the HABADTE application exhibits a layered design typical of mature AS400 systems:

- **Data Layer**: DDS physical files (OMPMAST, HAPTRFR, HXPDICT, HXPLVL*, OXPBNFIT, OXPNSTN, TAPIRNK/TMPMAST/TXPBNFIT/TXPNSTN) with logical overlays (HAPIRNK, HMLMAST5H, HXLTABL*, HXPBNFIT, HXPNSTN). This layer is heavily normalized and uses composite keys aligned to corporate level 6 (facility), account, and sequence fields.

- **Service/Utility Layer**: XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL, HXXAPPPRF, and copybooks HXX* and CXXXMLP/CXXXMLC. These encapsulate common cross-cutting concerns: text formatting, date handling, level hierarchy, code table resolution, MRN rollup, and XML ID management.

- **Orchestration Layer**: HABADTE, which ties patient master, transfer history, station metadata, and MRN rollup into a cohesive census report, producing both spool output and structured XML for downstream consumers.

Observed patterns and recommendations:

- **Clear separation of concerns**: Business rules around inclusion/exclusion logic (pre-admits, voided accounts, outpatients, discharge date checks) reside primarily in HABADTE, while reusable infrastructure concerns are delegated to XFX* utilities. This separation should be preserved during modernization by mapping utilities to shared services and HABADTE to a dedicated reporting service.

- **Stable reference structures**: Level tables (HXPLVL1–HXPLVL6) and code tables (HXPTABLD and its LFs) are central to facility and classification semantics. These should be migrated with high fidelity and exposed through declarative APIs to avoid re-implementing lookup logic scattered across programs.

- **High centralization of XML reporting**: XML report construction is tightly integrated into HABADTE via XFXGETID, XML copybooks, and HXPXMLD/HXPXMLR/HXFXMLH. A dedicated XML reporting service interface can simplify modernization and decouple layout definitions from business logic.

- **Orphan program analysis**: The aggregated context flags HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC as orphan programs. However, dependency edges show that all of these programs have at least one caller (HABADTE or XFXMRNROL). Therefore, **there are no true orphan programs** in the scanned codebase. All analyzed programs participate in at least one call chain and should be considered part of the active application surface.

Overall, the codebase is structurally consistent, with clear reuse of utilities and reference data. Modernization efforts should focus first on the HABADTE orchestration and its dependencies, ensuring that missing XML configuration (CXXXMLC, ****HXPXML) and environment-specific PRINTER definitions are addressed before refactoring into contemporary service architectures.
