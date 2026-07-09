# AS400 Code Analysis Report

## Section 1 – Component Inventory

### 1.1 Member Inventory

The reverse‑engineered codebase consists of 41 source members across RPG, SQLRPGLE and DDS artifacts. The table below summarizes each member, its type, subsystem, and approximate size in lines of code.

| Member      | Type     | Subsystem  | Lines |
|------------|----------|------------|-------|
| HAPIRNK    | DDS_LF   | HAP        | 13    |
| HAPTRFR    | DDS_PF   | HAP        | 72    |
| HMLMAST5H  | DDS_LF   | UNGROUPED  | 12    |
| HXLTABLD   | DDS_LF   | HXLT       | 11    |
| HXLTABLP   | DDS_LF   | HXLT       | 12    |
| HXLTABLS   | DDS_LF   | HXLT       | 12    |
| HXPBNFIT   | DDS_LF   | HXP        | 12    |
| HXPDICT    | DDS_PF   | HXP        | 6130  |
| HXPLVL1    | DDS_PF   | HXPL       | 49    |
| HXPLVL2    | DDS_PF   | HXPL       | 52    |
| HXPLVL3    | DDS_PF   | HXPL       | 52    |
| HXPLVL4    | DDS_PF   | HXPL       | 52    |
| HXPLVL5    | DDS_PF   | HXPL       | 55    |
| HXPLVL6    | DDS_PF   | HXPL       | 321   |
| HXPNSTN    | DDS_LF   | HXP        | 12    |
| HXPTABLD   | DDS_PF   | HXP        | 19    |
| HXPXMLD    | DDS_PF   | HXPX       | 19    |
| HXPXMLR    | DDS_PF   | HXPX       | 19    |
| OAPIRNK    | DDS_PF   | UNGROUPED  | 80    |
| OMPMAST    | DDS_PF   | UNGROUPED  | 310   |
| OXPBNFIT   | DDS_PF   | OXP        | 48    |
| OXPNSTN    | DDS_PF   | OXP        | 65    |
| TAPIRNK    | DDS_PF   | UNGROUPED  | 167   |
| TMPMAST    | DDS_PF   | UNGROUPED  | 688   |
| TXPBNFIT   | DDS_PF   | TXP        | 164   |
| TXPNSTN    | DDS_PF   | TXP        | 116   |
| HXXAPPPRF  | SQLRPGLE | HXXA       | 123   |
| XFXCNTR    | RPGLE    | XFXC       | 49    |
| XFXCYMD    | RPGLE    | XFXC       | 83    |
| XFXGETID   | RPGLE    | XFX        | 61    |
| XFXLDSC    | RPGLE    | XFXL       | 135   |
| XFXLEAP    | RPGLE    | XFXL       | 61    |
| XFXMRNROL  | RPGLE    | XFX        | 65    |
| XFXTABL    | RPGLE    | XFX        | 164   |
| CXXXMLP    | SQLRPGLE | UNGROUPED  | 25    |
| HXXAPPPRFP | RPGLE    | HXXA       | 42    |
| HXXCNTRL   | RPGLE    | HXX        | 8     |
| HXXLDA     | RPGLE    | HXXL       | 53    |
| HXXLEVEL   | RPGLE    | HXXL       | 25    |
| HXXXML     | RPGLE    | HXX        | 11    |
| HABADTE    | RPGLE    | HA         | 821   |

The member inventory shows a data‑heavy design dominated by DDS physical files (PFs) and logical files (LFs), with a smaller but critical set of RPG/SQLRPGLE programs implementing business logic.

### 1.2 Inventory by Type

From the aggregated statistics:

- DDS_LF: 7 members
- DDS_PF: 19 members
- SQLRPGLE: 2 members
- RPGLE: 13 members

This distribution indicates a traditional AS/400 application built around DDS‑defined files with RPG programs providing service functions such as date validation (XFXCYMD), counter management (XFXCNTR), ID assignment (XFXGETID), table lookups (XFXTABL), MRN roll logic (XFXMRNROL), and a large central batch process HABADTE.

The overall scanned codebase comprises 10,288 lines of code across 41 members, with approximately 91.1% completeness based on dependency coverage. Four gaps were detected, which are discussed in Section 2.

## Section 2 – Missing Components

The aggregator identified the following missing nodes and external references:

- CXXXMLC (COPYBOOK, HIGH impact, referenced by HABAD)
- HXHAPPPRF (PROGRAM, MEDIUM impact, referenced by XFXMR)
- ****HXPXML (FILE, MEDIUM impact, referenced by HABAD)
- PRINTER (FILE, MEDIUM impact, referenced by HABAD)

Although the abbreviated `referenced_by` names in the aggregated gaps differ slightly from the full program names in the dependency edges (for example, HABAD vs. HABADTE, XFXMR vs. XFXMRNROL), the pattern clearly shows that the principal driver program HABADTE and the MRN‑roll utility XFXMRNROL rely on artifacts not present in the scanned source set.

### 2.1 High‑Impact Gap – CXXXMLC Copybook

CXXXMLC is a copybook referenced by HABADTE through a COPY edge from HABADTE to CXXXMLC. As a HIGH impact gap, its absence indicates that common XML‑related declarations or shared constants are missing. This copybook likely defines structures, prototypes, or constants used for XML header formatting, message routing, or integration parameters.

Implications:

- Recompilation or migration of HABADTE will fail until CXXXMLC is supplied or its contents are reconstructed.
- Without this copybook, modernized code cannot reliably map XML document structures or interface parameters, making it risky to expose HABADTE’s functionality as an API.
- Any change in XML schema or interface contract would normally be centralized in CXXXMLC; its absence reduces confidence in understanding how XML data is bound to internal structures.

### 2.2 Medium‑Impact Gap – HXHAPPPRF Program

HXHAPPPRF is a program referenced by XFXMRNROL through a CALL edge. It is missing from the harvested members and has MEDIUM impact.

Implications:

- The MRN roll logic implemented in XFXMRNROL depends on HXHAPPPRF, likely to obtain application‑specific profile data or configuration.
- Testing MRN roll behavior in isolation will be incomplete without this program, as downstream business rules may be embedded there.
- For modernization, HXHAPPPRF must either be located in another library or reconstructed from production behavior; otherwise, MRN roll workflows will be partially unspecified.

### 2.3 Medium‑Impact Gap – ****HXPXML File

The pseudo‑file ****HXPXML is referenced by HABADTE as a FILE gap with MEDIUM impact. The asterisk prefix suggests a generated or environment‑specific file name, possibly an XML spool file or staging table used as an outbound interface.

Implications:

- HABADTE appears to read and/or write XML content via HXPXML‑related artifacts (HXPXMLD/HXPXMLR exist as PFs in the schema; ****HXPXML indicates an additional component such as a header or control file).
- Without this file definition, the exact structure of outbound XML documents cannot be fully reconstructed, hindering integration design in a target architecture.
- Runtime behavior may depend on this file for queueing or status tracking, meaning a modernization project must analyze production environments or job logs to derive the actual layout and usage.

### 2.4 Medium‑Impact Gap – PRINTER File

PRINTER is a FILE gap with MEDIUM impact, referenced by HABADTE. It likely represents a printer file or report definition used to produce output listings.

Implications:

- Report layouts and printed outputs produced by HABADTE cannot be fully understood without this printer file.
- Any modernization effort intending to replicate existing reports (e.g., PDFs, dashboards) must reconstruct the PRINTER file’s record format and layouts from DDS in other libraries or from physical print samples.
- Although print logic is often peripheral to core business rules, missing printer definitions complicate validation of end‑user visible outputs.

## Section 3 – Duplicate and Reused Components

Using dependency edges, we identify components that are heavily reused by multiple callers, as well as shared PF/LF relationships.

### 3.1 Reused Programs

The CALL edges show several programs used repeatedly:

- **XFXCNTR**
  - Called by HABADTE three times (multiple CALL edges from HABADTE to XFXCNTR).
  - Implements counter and field formatting logic (as indicated by approved rules BR‑001 and BR‑002).
  - Its reuse suggests a generic routine used across multiple segments of HABADTE to validate or format fixed‑length fields.

- **XFXLDSC**
  - Called by HABADTE via CALL edges.
  - Performs multiple READ operations against HXPLVL1–HXPLVL6, indicating a shared description or level‑lookup service.
  - It centralizes level/benefit description retrieval, reducing duplication.

- **XFXCYMD**
  - Called by HABADTE and in turn calls XFXLEAP.
  - Encapsulates date validation logic (BR‑003 and BR‑004) and leap year checks, positioning it as a shared calendar validation service.

- **XFXGETID**
  - Called by HABADTE.
  - Reads HXFXMLR and copies HXXLDA; likely responsible for correlating internal IDs, MRNs or document identifiers.

- **XFXTABL**
  - Called by HABADTE.
  - Reads multiple table files (XFFTABLD, XFFTABL2, XFFTABL3, XFFTABL4), providing generalized table lookup services.

Collectively, these reusable programs form a small library of shared services used by the main batch driver HABADTE.

### 3.2 Shared PF/LF Structures

Several LFs share common PFs, indicating reuse at the data model level:

- **HXPTABLD** as a shared PF
  - Logical files HXLTABLD, HXLTABLP, and HXLTABLS all specify HXPTABLD as their PFILE.
  - Patterns observed include:
    - HXLTABLD keyed by (XFDTCD, XFDMAP)
    - HXLTABLP keyed by (XFDTCD, XFDLDS)
    - HXLTABLS keyed by (XFDTCD, XFDSDS)
  - This indicates a central table dictionary (HXPTABLD) with multiple LFs providing different access paths for mapping, plan, and selection data.

- **TMPMAST/TAPIRNK/TXPBNFIT/TXPNSTN** as test/temporary PFs
  - HMLMAST5H uses TMPMAST as its PFILE.
  - HAPIRNK uses TAPIRNK as its PFILE.
  - HXPBNFIT uses TXPBNFIT, and HXPNSTN uses TXPNSTN.
  - The O‑prefixed PFs (OMPMAST, OAPIRNK, OXPBNFIT, OXPNSTN) appear to be production or operational equivalents, whereas T‑prefixed PFs may be test or staging copies.

These patterns show a deliberate reuse of data structures with multiple logical access paths, which must be preserved in any relational redesign.

## Section 4 – Dependency Analysis

### 4.1 Call Chain Overview

The core call graph is anchored by **HABADTE**, an RPGLE program with high complexity (cyclomatic complexity cc = 152, HIGH band). Key relationships include:

- HABADTE → XFXMRNROL (CALL)
- HABADTE → XFXCNTR (multiple CALLs)
- HABADTE → XFXLDSC (CALL)
- HABADTE → XFXCYMD (CALL)
- HABADTE → XFXGETID (CALL)
- HABADTE → XFXTABL (CALL)
- HABADTE → support copy members: HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC

Downstream service relationships:

- XFXCYMD → XFXLEAP (CALL)
- XFXMRNROL → HXHAPPPRF (CALL, missing program)
- XFXMRNROL → HXXAPPPRF (CALL)

The call graph reveals a layered design where HABADTE orchestrates multiple utility programs for date validation, MRN management, ID resolution, table lookup, and XML handling.

### 4.2 File Access Patterns

File operations in the dependency edges show:

- HABADTE reads and writes XML‑related files:
  - READ HAPTRFR
  - READ XFFNSTN
  - READ/UPDATE/WRITE HXFXMLH
  - WRITE HXFXMLD
- XFXLDSC reads level PFs HXFLVL1–HXFLVL6, providing level master data.
- XFXGETID reads HXFXMLR, likely to retrieve existing XML message or identifier records.
- XFXTABL reads table files XFFTABLD–XFFTABL4, indicating a generalized table‑driven design.

This pattern suggests that HABADTE produces XML output (HXFXMLD/HXFXMLH) based on transfer records (HAPTRFR), status records (XFFNSTN), and various master tables.

### 4.3 Hotspot Analysis and Risk

Hotspot metrics computed from dep_hotspots show:

- **HABADTE** – score 38, fan_out 13, file_ops 6, fan_in 0
  - Central driver responsible for orchestrating multiple services and file operations.
  - High complexity and high fan‑out make it the primary risk area for regression during modernization.

- **XFXLDSC** – score 15, fan_in 1, file_ops 6
  - Specialized for level master reads; moderate hotspot due to multiple file operations.

- **XFXTABL** – score 11, fan_in 1, file_ops 4
  - Critical for table‑driven behavior; changes in table layouts will heavily affect this program.

- **XFXCNTR, XFXMRNROL, XFXGETID, HXXAPPPRF** – scores between 7 and 9, with low to moderate fan‑in/fan‑out
  - Important reusable utilities but lower global risk than HABADTE.

Overall risk assessment:

- HABADTE must be treated as a core batch service; any refactoring should be accompanied by detailed regression tests, especially around XML generation and MRN handling.
- Supporting utilities have low cyclomatic complexity (cc ≤ 9, LOW band), suggesting they are good candidates for early extraction into services or micro‑functions.

## Section 5 – Summary of Key Findings

### 5.1 Architectural Observations

- The application is structured around a single high‑complexity batch driver (HABADTE) that orchestrates multiple low‑complexity utility programs and a rich DDS file schema.
- Data design favors extensive use of DDS PF/LF combinations, especially around table dictionaries (HXPTABLD and its LFs), level master files (HXPLVL1–HXPLVL6), and patient/account master files (OMPMAST, HXPDICT, OAPIRNK, OXPBNFIT).
- Control logic relies heavily on externalized tables and XML structures, indicated by the presence of HXPTABLD, HXPXMLD/HXPXMLR, and the missing CXXXMLC and ****HXPXML artifacts.

### 5.2 Patterns and Anti‑Patterns

- **Positive patterns**
  - Shared utilities for date validation, leap year handling, field formatting, and table lookups improve consistency and reduce duplication.
  - Use of logical files over shared PFs provides multiple, well‑defined access paths for different processing needs.

- **Risk areas**
  - The high complexity of HABADTE combined with its responsibility for both business rules and integration (XML and printer outputs) makes it a monolithic bottleneck.
  - Missing components (copybooks, external programs, printer and XML files) reduce observability and increase the risk of mis‑specifying interfaces during modernization.

### 5.3 Orphan Programs

The orphan_programs list includes HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC. However, the hotspot and dependency data clearly show that these programs have non‑zero fan_in (they are called by HABADTE or other programs). As a result, there are **no true orphans** in the scanned codebase.

No true orphans detected. All programs have defined callers in the scanned codebase.

### 5.4 Recommendations

- Prioritize detailed documentation and test coverage for HABADTE, including its interactions with XML files (HXFXMLH, HXFXMLD, HXPXML*), table files (HXPTABLD/XFFTABL*), and MRN‑related utilities (XFXMRNROL, XFXGETID).
- Recover or reconstruct the missing CXXXMLC copybook, HXHAPPPRF program, ****HXPXML file, and PRINTER file before attempting full‑scale modernization of XML and reporting flows.
- Preserve the PF/LF relationships, especially those centered on HXPTABLD and TMPMAST/TAPIRNK/TXPBNFIT/TXPNSTN, when mapping the schema into a modern relational or document model.
- Consider splitting HABADTE’s responsibilities into separate services (e.g., XML generation, MRN management, report production) while keeping utilities like XFXCYMD, XFXCNTR, and XFXTABL as reusable core libraries.
