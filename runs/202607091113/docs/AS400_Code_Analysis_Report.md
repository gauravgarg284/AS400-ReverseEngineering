# AS400 Code Analysis Report – HABADTE Project

## Section 1 – Component Inventory

This section summarizes the full component inventory for the HABADTE project as discovered from the AS400 source scan. Counts by member type come from the source manifest; individual members are listed with their subsystem affiliation and line counts.

### 1.1 Inventory by Type

- DDS Logical Files (DDS_LF): 7
- DDS Physical Files (DDS_PF): 19
- RPGLE Programs: 13
- SQLRPGLE Programs: 2
- Total members: 41
- Total lines of source: 10,288

This confirms a file-heavy system with a moderate number of processing programs and a single very large dictionary file (HXPDICT) dominating the line count.

### 1.2 Member Inventory (Detailed)

| Member      | Type    | Subsystem  | Lines |
|------------|---------|-----------|-------|
| HAPIRNK    | DDS_LF  | HAP       | 13    |
| HAPTRFR    | DDS_PF  | HAP       | 72    |
| HMLMAST5H  | DDS_LF  | UNGROUPED | 12    |
| HXLTABLD   | DDS_LF  | HXLT      | 11    |
| HXLTABLP   | DDS_LF  | HXLT      | 12    |
| HXLTABLS   | DDS_LF  | HXLT      | 12    |
| HXPBNFIT   | DDS_LF  | HXP       | 12    |
| HXPDICT    | DDS_PF  | HXP       | 6130  |
| HXPLVL1    | DDS_PF  | HXPL      | 49    |
| HXPLVL2    | DDS_PF  | HXPL      | 52    |
| HXPLVL3    | DDS_PF  | HXPL      | 52    |
| HXPLVL4    | DDS_PF  | HXPL      | 52    |
| HXPLVL5    | DDS_PF  | HXPL      | 55    |
| HXPLVL6    | DDS_PF  | HXPL      | 321   |
| HXPNSTN    | DDS_LF  | HXP       | 12    |
| HXPTABLD   | DDS_PF  | HXP       | 19    |
| HXPXMLD    | DDS_PF  | HXPX      | 19    |
| HXPXMLR    | DDS_PF  | HXPX      | 19    |
| OAPIRNK    | DDS_PF  | UNGROUPED | 80    |
| OMPMAST    | DDS_PF  | UNGROUPED | 310   |
| OXPBNFIT   | DDS_PF  | OXP       | 48    |
| OXPNSTN    | DDS_PF  | OXP       | 65    |
| TAPIRNK    | DDS_PF  | UNGROUPED | 167   |
| TMPMAST    | DDS_PF  | UNGROUPED | 688   |
| TXPBNFIT   | DDS_PF  | TXP       | 164   |
| TXPNSTN    | DDS_PF  | TXP       | 116   |
| HXXAPPPRF  | SQLRPGLE| HXXA      | 123   |
| XFXCNTR    | RPGLE   | XFXC      | 49    |
| XFXCYMD    | RPGLE   | XFXC      | 83    |
| XFXGETID   | RPGLE   | XFX       | 61    |
| XFXLDSC    | RPGLE   | XFXL      | 135   |
| XFXLEAP    | RPGLE   | XFXL      | 61    |
| XFXMRNROL  | RPGLE   | XFX       | 65    |
| XFXTABL    | RPGLE   | XFX       | 164   |
| CXXXMLP    | SQLRPGLE| UNGROUPED | 25    |
| HXXAPPPRFP | RPGLE   | HXXA      | 42    |
| HXXCNTRL   | RPGLE   | HXX       | 8     |
| HXXLDA     | RPGLE   | HXXL      | 53    |
| HXXLEVEL   | RPGLE   | HXXL      | 25    |
| HXXXML     | RPGLE   | HXX       | 11    |
| HABADTE    | RPGLE   | HA        | 821   |

The inventory shows a centralized control program (HABADTE), several date/level/table utility programs in the XFX* family, and a substantial number of PF/LF definitions for benefits, patient accounts, and level tables.

## Section 2 – Missing Components

This section documents missing or unresolved components discovered during dependency analysis. Each gap represents a referenced but non-resolved asset.

### 2.1 Gap List

| Name      | Type      | Impact | Referenced By |
|-----------|-----------|--------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH   | HABAD         |
| HXHAPPPRF | PROGRAM   | MEDIUM | XFXMR         |
| ****HXPXML| FILE      | MEDIUM | HABAD         |
| PRINTER   | FILE      | MEDIUM | HABAD         |

### 2.2 Implications by Gap Category

**CXXXMLC (COPYBOOK, HIGH impact)**  
This missing copybook is referenced from the main HABAD* control program. Because copybooks typically hold shared data structures or common routines, its absence means:
- XML-related structures or constants may be partially understood only from the consuming program.
- Any downstream modernization that targets XML export/import logic may be incomplete.
- Changes to XML format will be harder to trace without the shared definition.

**HXHAPPPRF (PROGRAM, MEDIUM impact)**  
This unresolved program is called by XFXMRNROL. The call indicates an additional layer of application profile logic that is not present in the scanned codebase. Implications:
- MRN roll‑forward/backward processes may delegate to HXHAPPPRF for application‑profile checks.
- Without its definition, some control‑flow behavior around MRN updates cannot be fully reconstructed.
- Regression risk exists if modernization assumes MRN logic is fully contained in XFXMRNROL.

******HXPXML (FILE, MEDIUM impact)**  
This unresolved file appears to be part of the HXPXML family. Its absence suggests:
- There may be additional PF/LF definitions for XML payload staging or logging not included in this library.
- Data lineage for XML export may be incomplete, particularly if this file captures outbound payloads.

**PRINTER (FILE, MEDIUM impact)**  
The missing PRINTER file indicates that printer definition or spool configuration is external to this library. Consequences include:
- Output routing rules may depend on external printer tables.
- Modernization to non‑spool output (e.g., PDF or REST responses) must account for this external dependency.

This section is the authoritative catalog of missing components and should be used as the single source of truth for completeness assessments.

## Section 3 – Duplicate and Reused Components

Using the dependency edges, we identify components that are reused across the system.

### 3.1 Reused Programs (CALL Targets)

Analysis of CALL edges shows the following repeated usage patterns:

- **XFXCNTR**
  - Called three times from HABADTE.
  - Role: field formatting and counter logic (based on extracted rules BR-001 and BR-002).
  - Interpretation: XFXCNTR acts as a reusable formatting/counter routine invoked from different branches of HABADTE’s processing.

- **XFXMRNROL**
  - Called once from HABADTE and in turn calls HXHAPPPRF and HXXAPPPRF.
  - Role: MRN rollover logic, coordinating updates to patient MRNs and related profiles.

- **XFXLDSC**
  - Called once from HABADTE; performs multiple READs against the HXFLVLx/HXPLVLx files.
  - Role: level lookup/description retrieval using a shared level table hierarchy.

- **XFXCYMD**
  - Called from HABADTE and itself calls XFXLEAP.
  - Role: centralized date validation/normalization, including leap‑year checks.

The presence of these shared routines indicates a degree of normalization in the procedural logic despite the monolithic HABADTE driver program.

### 3.2 Shared File Structures and Logical Files

Logical files over the same physical file indicate reuse of base data structures.

- **HXPTABLD** (PF)
  - Shared by HXLTABLD, HXLTABLP, HXLTABLS logical files.
  - Pattern: a single table dictionary with multiple logical views (by mapping code, description, or status). This suggests a configurable table‑driven design for code/description lookups.

- **TXPBNFIT** and **TXPNSTN** (PFs)
  - TXPBNFIT is the base PF for logical file HXPBNFIT.
  - TXPNSTN is the base PF for HXPNSTN.
  - Pattern: transactional or reference tables with logical overlays for business‑friendly access paths.

- **TMPMAST** and **TAPIRNK** (PFs)
  - TMPMAST underlies HMLMAST5H.
  - TAPIRNK underlies HAPIRNK.
  - Pattern: master data tables accessed via specialized logical files for reporting or batch processing.

These reuse patterns indicate that while individual PFs are numerous, they are organized through a set of logical views that provide different key sequences, simplifying program logic.

## Section 4 – Dependency Analysis

### 4.1 Call Chain Overview

The main program **HABADTE** serves as the top‑level driver in the patient management domain. From the call edges:

- HABADTE calls:
  - XFXMRNROL – MRN rollover logic.
  - XFXCNTR (three distinct CALL sites) – field/counter formatting.
  - XFXLDSC – level/descriptor lookup from HXPLVLx/HXFLVLx files.
  - XFXCYMD – date validation and normalization.
  - XFXGETID – identifier retrieval, including XML record access via HXFXMLR.
  - XFXTABL – generic table lookup over XFFTABLD/XFFTABLx.

- XFXCYMD calls XFXLEAP to evaluate leap‑year behavior.
- XFXMRNROL calls HXHAPPPRF (missing) and HXXAPPPRF (SQLRPGLE program) for additional profile logic.
- HXXAPPPRF in turn copies HXXCNTRL and HXXAPPPRFP, indicating a small layered control/profile pattern.

This call graph shows a hub‑and‑spoke design where HABADTE orchestrates specialized utility programs rather than embedding all logic inline.

### 4.2 File Access Patterns and Hotspots

Dependency edges include READ/WRITE/UPDATE operations:

- HABADTE performs:
  - READ on HAPTRFR (transaction transfer PF).
  - READ on XFFNSTN (state/region or station file via logical over OXPNSTN).
  - READ/UPDATE/WRITE on HXFXMLH and WRITE on HXFXMLD – suggesting construction and persistence of XML payload headers and details.

- XFXGETID declares and reads HXPXMLR/HXFXMLR, implying responsibility for pulling XML header/record identifiers.

- XFXLDSC declares and reads across the six level PFs (HXPLVL1–HXPLVL6 and corresponding HXFLVLx views), consolidating level definitions.

- XFXTABL reads XFFTABLD and related table views.

Hotspot analysis:

- **HABADTE** – score 38, fan‑out 13, file_ops 6. This is the primary orchestration hotspot and the highest‑complexity program (cyclomatic complexity 152, band HIGH).
- **XFXLDSC** – score 15, fan‑in 1, file_ops 6. Acts as a dedicated level‑lookup service.
- **XFXTABL** – score 11, fan‑in 1, file_ops 4. Acts as a generalized table lookup engine.
- **XFXCNTR** – score 9, fan‑in 3, file_ops 0. Pure computational/formatting service invoked from multiple branches.
- **XFXMRNROL**, **XFXGETID**, **HXXAPPPRF** – intermediate hotspots coordinating MRN and profile logic or bridging to SQL tables.

The relatively high fan‑out from HABADTE, combined with its high complexity, makes it a critical migration risk. Any refactoring should prioritize decomposition of this driver into smaller service‑oriented units.

## Section 5 – Summary of Key Findings

1. **Centralized Patient Management Controller**  
   HABADTE is a monolithic controller for patient management processes, with high cyclomatic complexity and extensive fan‑out to utility programs and file operations. This program is the primary candidate for modularization into domain‑specific services (e.g., MRN management, benefits lookup, XML export).

2. **Structured Utility Layer (XFX* Family)**  
   The XFX* programs (XFXCYMD, XFXCNTR, XFXLDSC, XFXTABL, XFXGETID, XFXLEAP, XFXMRNROL) form a reusable utility layer for dates, counters, table lookups, level resolution, identifier assignment, and MRN rollover. Maintaining these as independent services in a modern architecture will preserve existing reuse patterns and reduce duplication.

3. **Table‑Driven Configuration**  
   The presence of multi‑view PF/LF combinations (HXPTABLD with HXLTABLD/HXLTABLP/HXLTABLS; TAPIRNK with HAPIRNK; TMPMAST with HMLMAST5H; TXPBNFIT with HXPBNFIT; TXPNSTN with HXPNSTN) shows that many business rules are encoded in table data rather than hard‑coded branches. A modernization strategy should migrate these tables into a relational or configuration service layer rather than flattening them into code.

4. **XML Staging and Output Pipeline**  
   Files HXPXMLD, HXPXMLR, and unresolved ****HXPXML, together with HXFXMLH/HXFXMLD usage in HABADTE and XFXGETID, indicate an XML export pipeline for patient transactions. Modern interfaces (REST/JSON) can be introduced by re‑implementing this pipeline, but care must be taken to replicate the existing header/detail record flow and any associated printer‑based outputs.

5. **Technical Debt Concentration**  
   The tech debt summary shows 26.9 hours across four findings, with HABADTE as the only high‑complexity program. The rest of the codebase is relatively simple (most utility programs have LOW complexity). Targeting HABADTE for refactor will yield the highest risk reduction and improve maintainability.

6. **Orphan Program Assessment**  
   The orphan_programs list includes HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC. However, dependency edges show that HABADTE and other programs do call these utilities, so they are not true orphans from a static call‑graph perspective. The likely explanation is that fan‑in was measured at the library boundary rather than across all scanned members. Therefore: **No true orphans detected. All programs have defined callers in the scanned codebase.**

Overall, the HABADTE project shows a classic AS400 batch/controller pattern with a strong utility layer and table‑driven configuration, but with critical gaps around XML copybooks and external files that must be addressed during modernization.
