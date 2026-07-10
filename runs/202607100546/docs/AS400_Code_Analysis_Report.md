# AS400 Code Analysis Report – HABADTE Application

## 1. Component Inventory

This section summarizes all analyzed source members for the HABADTE application based on the harvested manifest. The inventory combines DDS physical and logical files with RPG and SQLRPGLE programs.

### 1.1 Member Inventory Table

| Member Name | Type     | Subsystem  | Lines |
|------------:|----------|-----------|------:|
| HAPIRNK     | DDS_LF   | HAP       | 13   |
| HAPTRFR     | DDS_PF   | HAP       | 72   |
| HMLMAST5H   | DDS_LF   | UNGROUPED | 12   |
| HXLTABLD    | DDS_LF   | HXLT      | 11   |
| HXLTABLP    | DDS_LF   | HXLT      | 12   |
| HXLTABLS    | DDS_LF   | HXLT      | 12   |
| HXPBNFIT    | DDS_LF   | HXP       | 12   |
| HXPDICT     | DDS_PF   | HXP       | 6130 |
| HXPLVL1     | DDS_PF   | HXPL      | 49   |
| HXPLVL2     | DDS_PF   | HXPL      | 52   |
| HXPLVL3     | DDS_PF   | HXPL      | 52   |
| HXPLVL4     | DDS_PF   | HXPL      | 52   |
| HXPLVL5     | DDS_PF   | HXPL      | 55   |
| HXPLVL6     | DDS_PF   | HXPL      | 321  |
| HXPNSTN     | DDS_LF   | HXP       | 12   |
| HXPTABLD    | DDS_PF   | HXP       | 19   |
| HXPXMLD     | DDS_PF   | HXPX      | 19   |
| HXPXMLR     | DDS_PF   | HXPX      | 19   |
| OAPIRNK     | DDS_PF   | UNGROUPED | 80   |
| OMPMAST     | DDS_PF   | UNGROUPED | 310  |
| OXPBNFIT    | DDS_PF   | OXP       | 48   |
| OXPNSTN     | DDS_PF   | OXP       | 65   |
| TAPIRNK     | DDS_PF   | UNGROUPED | 167  |
| TMPMAST     | DDS_PF   | UNGROUPED | 688  |
| TXPBNFIT    | DDS_PF   | TXP       | 164  |
| TXPNSTN     | DDS_PF   | TXP       | 116  |
| HXXAPPPRF   | SQLRPGLE | HXXA      | 123  |
| XFXCNTR     | RPGLE    | XFXC      | 49   |
| XFXCYMD     | RPGLE    | XFXC      | 83   |
| XFXGETID    | RPGLE    | XFX       | 61   |
| XFXLDSC     | RPGLE    | XFXL      | 135  |
| XFXLEAP     | RPGLE    | XFXL      | 61   |
| XFXMRNROL   | RPGLE    | XFX       | 65   |
| XFXTABL     | RPGLE    | XFX       | 164  |
| CXXXMLP     | SQLRPGLE | UNGROUPED | 25   |
| HXXAPPPRFP  | RPGLE    | HXXA      | 42   |
| HXXCNTRL    | RPGLE    | HXX       | 8    |
| HXXLDA      | RPGLE    | HXXL      | 53   |
| HXXLEVEL    | RPGLE    | HXXL      | 25   |
| HXXXML      | RPGLE    | HXX       | 11   |
| HABADTE     | RPGLE    | HA        | 821  |

The mix of members shows a traditional AS400 application where persistent state is modeled primarily through DDS physical files and logical views, with a moderate set of RPG business services.

### 1.2 Inventory by Type

From the aggregated manifest:

- DDS_LF: 7 members
- DDS_PF: 19 members
- SQLRPGLE: 2 members
- RPGLE: 13 members

In total, 41 source members comprising 10,288 lines were scanned. The physical file count (19) highlights a rich data model, while the relatively small set of 15 programs (RPGLE/SQLRPGLE) suggests that business logic is concentrated into a limited number of executables, notably HABADTE and the XFX* utilities.

## 2. Missing Components

This section documents components referenced in the source but missing from the scanned codebase. These gaps are critical for scoping modernization efforts and understanding runtime dependencies that lie outside this repository.

### 2.1 Gap Inventory

| Name      | Type     | Impact | Referenced By |
|-----------|----------|--------|---------------|
| CXXXMLC   | COPYBOOK | HIGH   | HABAD         |
| HXHAPPPRF | PROGRAM  | MEDIUM | XFXMR         |
| ****HXPXML| FILE     | MEDIUM | HABAD         |
| PRINTER   | FILE     | MEDIUM | HABAD         |

### 2.2 Interpretation of Gaps

- **CXXXMLC (COPYBOOK, HIGH impact)**  
  This copybook is referenced as an included member from HABAD* code (the dep graph shows a COPY edge from HABADTE to CXXXMLC). Its absence indicates that some XML-related layout, constants, or I/O structures are not available. Given that HABADTE also copies CXXXMLP and HXXXML, CXXXMLC likely centralizes XML configuration or shared field definitions. A missing copybook at this level can break compilation and hide transformation rules used when emitting XML.

- **HXHAPPPRF (PROGRAM, MEDIUM impact)**  
  HXHAPPPRF is called by XFXMRNROL. The dep graph records a CALL from XFXMRNROL to HXHAPPPRF and HXXAPPPRF. The latter is present, but HXHAPPPRF is missing. This suggests that XFXMRNROL is designed to route work to alternative implementations, potentially differentiating legacy native logic (HXHAPPPRF) from a newer SQL-based profile (HXXAPPPRF). At runtime, only one branch may be active, but from a modernization standpoint we must confirm which path is actually used in production.

- **"****HXPXML" (FILE, MEDIUM impact)**  
  The placeholder-style name indicates a family of XML header/detail work files (for example HXFXMLH / HXPXML*). HABADTE reads and writes XML-related files (HXFXMLH, HXFXMLD, HXFXMLR). The missing ****HXPXML definition suggests that some dictionary or control file for XML export is not captured in the current extraction, which could affect understanding of how XML segments are grouped or sequenced.

- **PRINTER (FILE, MEDIUM impact)**  
  A generic PRINTER file definition is referenced by HABAD* code. This is typically a spool/printer file or an externally-described print file used for reports. Its absence means that page layout, headings, and reported fields are not visible. Functionally the program will still process core business rules, but the reporting interface cannot be fully reconstructed from this dataset.

### 2.3 Implications by Gap Category

- **Copybook gaps (CXXXMLC)** – These block full recompilation and can hide shared validation or mapping rules. For modernization, treat CXXXMLC as part of the application contract for XML-based integrations. Reverse-engineering downstream XML consumers and any companion copybooks (such as CXXXMLP) will be required to re-establish a complete schema.

- **Program gaps (HXHAPPPRF)** – Missing programs referenced from XFXMRNROL may represent optional or decommissioned flows. Before retiring or refactoring XFXMRNROL, determine whether production job streams ever call the HXHAPPPRF branch. If yes, the modernization design must either reimplement its behavior or formally de-scope it with business approval.

- **File gaps (****HXPXML, PRINTER)** – Missing file objects translate into blind spots in data lineage and reporting. For XML control files, this mainly affects the understanding of configuration and routing. For PRINTER, the main risk lies in not capturing regulatory or patient-facing report formats that may be subject to compliance review.

## 3. Duplicate and Reused Components

Using the dependency edges, we identify reusable services and shared files.

### 3.1 Programs Reused Across Callers

From the CALL edges:

- `HABADTE` calls:
  - XFXMRNROL
  - XFXCNTR (three separate CALL edges)
  - XFXLDSC
  - XFXCYMD
  - XFXGETID
  - XFXTABL
- `XFXCYMD` calls XFXLEAP.
- `XFXMRNROL` calls HXHAPPPRF (missing) and HXXAPPPRF.

Based on targets that appear multiple times:

- **XFXCNTR** – called three times from HABADTE. This indicates intensive reuse of a counter or control-number utility within a single driver program. It is likely invoked for multiple numbering contexts (transaction, sequence, or batch run identifiers). In modernization, XFXCNTR is a clear candidate for a centrally deployed ID generation service.

- **XFXLEAP** – called from XFXCYMD to determine leap-year behavior. Although only one caller is observed, the function is isolated and has very low cyclomatic complexity. It can be elevated into a shared date-utility library.

- **HXXAPPPRF** – invoked by XFXMRNROL alongside a missing legacy implementation (HXHAPPPRF). The presence of both suggests deliberate reuse of a profile-maintenance module, with one path using SQLRPGLE and one using native I/O.

### 3.2 Shared Physical/Logical Files

From PFILE_OF edges:

- HXLTABLD, HXLTABLP, HXLTABLS are all logical views over HXPTABLD.
- HXPBNFIT is a logical view over TXPBNFIT.
- HXPNSTN is a logical view over TXPNSTN.
- HAPIRNK is a logical view over TAPIRNK.
- HMLMAST5H is a logical view over TMPMAST.

These patterns show a common design approach: store raw reference and master data in staging tables (TXP*, TXPNSTN, TAPIRNK, TMPMAST) and present application-specific filtered or keyed views through H*-prefixed logical files. For example, HXLTABLD/P/S all project different field subsets or sort orders from HXPTABLD, which is then read by XFXTABL.

This reuse will be important when mapping to relational or service-based architectures: instead of migrating each LF separately, we can define a single table per PF with multiple indexes or views representing existing access paths.

## 4. Dependency Analysis

### 4.1 Call Graph Overview

The call graph is centered around HABADTE:

- HABADTE acts as the primary driver in the PATIENT_MANAGEMENT domain, orchestrating patient account transfer logic.
- It delegates specialized responsibilities to XFX* utilities:
  - **XFXMRNROL** – rolls or validates MRN-related data and calls HXHAPPPRF (missing) and HXXAPPPRF.
  - **XFXCNTR** – encapsulates counter/sequence management and is invoked multiple times per run.
  - **XFXLDSC** – reads level tables HXPLVL1–HXPLVL6 to resolve level descriptions.
  - **XFXCYMD** – validates date ranges and uses XFXLEAP for leap-year checks.
  - **XFXGETID** – retrieves identifiers, reading XML-related records via HXFXMLR.
  - **XFXTABL** – reads table data XFFTABLD and related variants.

Secondary call chains include:

- XFXCYMD → XFXLEAP
- XFXMRNROL → HXHAPPPRF (missing) and HXXAPPPRF

### 4.2 File Access Patterns

File access edges show how key entities are touched:

- HABADTE reads HAPTRFR (patient transfer records) and XFFNSTN, and performs READ/UPDATE/WRITE on HXFXMLH and WRITE on HXFXMLD, indicating that it builds XML output records.
- XFXLDSC declares and reads HXPLVL1–HXPLVL6 and corresponding HXFLVL* formats, confirming its role as a level/plan lookup service.
- XFXGETID declares and reads HXPXMLR/HXFXMLR to obtain XML-related IDs.
- XFXTABL reads XFFTABLD and related table segments, acting as a generic table-lookup engine.

These patterns highlight HABADTE as the orchestration layer, XFX* programs as domain utilities, and DDS PF/LF structures as both master data and configuration sources.

### 4.3 Hotspot Programs and Risk Assessment

From `dep_hotspots`:

- **HABADTE** – score 38, fan_in 0, fan_out 13, file_ops 6. This is a primary batch/driver program with extensive outbound calls and file operations. Its high fan-out and file touchpoints make it the most complex and risky component to modify, especially given its cyclomatic complexity of 152 (HIGH band).
- **XFXLDSC** – score 15, fan_in 1, file_ops 6. A focused data-access service for level tables. Moderate hotspot due to multiple file reads but limited control complexity (cc=5).
- **XFXTABL** – score 11, fan_in 1, file_ops 4. A table-lookup utility that centralizes business reference data access. Cyclomatic complexity is still in the LOW band but higher than other utilities (cc=9).
- **XFXCNTR** – score 9, fan_in 3, file_ops 0. High reuse (fan_in=3) but no direct file access, indicating a pure logic or wrapper component.
- **XFXMRNROL, HXXAPPPRF, XFXGETID, XFXCYMD** – scores between 5 and 7 with low cyclomatic complexity. These are important but structurally manageable.

Given the hotspot profile, modernization should prioritize stabilizing and encapsulating HABADTE first, then extracting reusable services around XFXLDSC and XFXTABL where appropriate.

## 5. Summary of Key Findings

1. **Central orchestration by HABADTE** – HABADTE is the core driver in the PATIENT_MANAGEMENT domain, coordinating MRN rollover, date validation, level lookups, ID retrieval, table access, and XML record generation. Any modernization or refactoring will need a robust regression strategy around this program.

2. **Clear utility layer in XFX* programs** – The XFX* suite forms a reusable utility layer for counters, dates, level descriptions, MRN rollover, table lookups, and ID retrieval. Most of these programs have LOW cyclomatic complexity, making them good candidates for early extraction into shared services or microservices.

3. **Data-centric architecture with heavy use of PF/LF abstractions** – The application leans on DDS logical files (HAPIRNK, HMLMAST5H, HXLTAB* and others) to project and filter underlying physical files. Modern designs should preserve this intent using indexed views or ORM mappings rather than attempting a one-to-one migration of every LF.

4. **XML as an integration channel** – HABADTE’s interaction with HXFXMLH/HXFXMLD/HXFXMLR and the missing CXXXMLC copybook demonstrates an XML-based integration or reporting pipeline. Modernization plans must account for this external contract, including field ordering and segment structure.

5. **Technical risk concentrated in a small number of programs** – While the overall codebase has mostly LOW complexity programs, HABADTE stands out with cyclomatic complexity 152 and the highest hotspot score. Focusing on modularizing HABADTE (for example, by extracting independent processing phases into separate procedures or services) would significantly reduce maintenance risk.

6. **Orphan program review** – The orphan list includes HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC, but hotspot data contradicts this by showing non-zero fan_in for some of them. Because at least XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC clearly have callers, they are not true orphans. For this reason, no orphan programs are reported as candidates for retirement. All active programs have at least one defined caller in the scanned graph.

7. **Recommendations** –
   - Encapsulate HABADTE behind a service interface and incrementally extract self-contained logic (date validation, level lookup, MRN rollover) into isolated components.
   - Normalize access to PF/LF data through repository or gateway services that expose intent (e.g., "fetch active patient levels") instead of raw DDS constructs.
   - Reconstruct and document the XML interface using the existing HXFXML* files, HXPXML* patterns, and CXXXMLP; then locate or recreate CXXXMLC to complete the schema.
   - Validate the runtime relevance of HXHAPPPRF and PRINTER objects with operations teams before deciding to reimplement or retire them.
