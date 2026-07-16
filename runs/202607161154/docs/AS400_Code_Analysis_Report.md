# AS400 Code Analysis Report – HABADTE System

Run ID: 202607161154  
Project: HABADTE (Patient admission and census reporting utilities)

---

## 1. Component Inventory

This section summarizes all scanned source members in the HABADTE application. Inventory is based on the aggregated source manifest.

### 1.1 Member Catalogue

| Member Name | Type    | Subsystem | Lines |
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

Total members: 41  
Total lines: 10,288

Subsystem distribution shows a clear separation between:
- **HA (HABADTE)** – main patient management/reporting driver.
- **XFX* programs** – shared utility services for text, dates, identifiers, levels and table lookups.
- **HXPL/HXP/HXPX** – dictionary, level hierarchy and XML payload tables.
- **O* and T* files** – operational master and crosswalk tables.

### 1.2 Members by Type

Source type totals:
- DDS_LF (logical files): 7
- DDS_PF (physical files): 19
- SQLRPGLE programs: 2
- RPGLE programs: 13

The codebase is predominantly data-centric, with 26 DDS-based files and 15 RPG-based programs. The DDS files form the persistence and reference layers (patient master, benefit plans, nursing stations, room transfer history and generic dictionary tables), while RPG/SQLRPGLE programs orchestrate business logic and reporting.

---

## 2. Missing Components

The gap analysis highlights source elements referenced by the scanned code but absent from the harvested set. These are important for both migration completeness and runtime behavior.

### 2.1 Gap List

From the aggregated gap report:

| Missing Name | Type      | Impact  | Referenced By |
|--------------|-----------|---------|---------------|
| CXXXMLC      | COPYBOOK  | HIGH    | HABADTE       |
| HXHAPPPRF    | PROGRAM   | MEDIUM  | XFXMRNROL     |
| ****HXPXML   | FILE      | MEDIUM  | HABADTE       |
| PRINTER      | FILE      | MEDIUM  | HABADTE       |

These four gaps account for the 8.9% incompleteness reported by the pipeline (completeness ~91.1%).

### 2.2 High-Impact Gaps – CXXXMLC

**CXXXMLC (COPYBOOK, impact HIGH, referenced by HABADTE)**

HABADTE copies CXXXMLC alongside several other control and XML-related copy members (HXXLDA, HXXLEVEL, HXXXML, CXXXMLP). The missing copybook likely defines shared data structures for XML header or control fields used when HABADTE writes records to `HXFXMLD` and `HXFXMLH`.

Implications:
- **Runtime risk:** Without CXXXMLC, fields used in XML header/control processing may be undefined or incorrectly mapped, breaking report export or downstream interfaces.
- **Migration risk:** Modernization cannot accurately reconstruct the XML payload schema used for census extracts. Any transformation to REST or message-based APIs must reconstruct CXXXMLC’s structures from behavioral clues.
- **Testing complexity:** Scenarios that exercise XML export (reading transfer history from `HAPTRFR` then writing to `HXFXMLH`/`HXFXMLD`) will be hard to simulate until the copybook is located or reverse-engineered.

### 2.3 Medium-Impact Gaps – HXHAPPPRF and ****HXPXML

**HXHAPPPRF (PROGRAM, impact MEDIUM, referenced by XFXMRNROL)**

XFXMRNROL calls both HXHAPPPRF and HXXAPPPRF. HXXAPPPRF is present and documented as an application preference accessor reading HXPAPPPRF/HXPAPPL6 tables. HXHAPPPRF likely plays a similar role but may be an older or facility-specific variant:
- Could contain MRN/account preference rules (e.g., whether to display MRN vs account number, or facility overrides).
- Missing program creates uncertainty around how XFXMRNROL chooses identifier display and preference resolution paths.

Implications:
- **Functional ambiguity:** Modernized MRN roll logic may not match the legacy behavior without understanding HXHAPPPRF’s decision rules.
- **Data lineage:** Some PHI exposure paths via MRN/account preferences could be overlooked if HXHAPPPRF also accesses PHI-bearing tables.

******HXPXML (FILE, impact MEDIUM, referenced by HABADTE)**

The placeholder-like name ****HXPXML suggests either a generic XML configuration file or a naming convention artifact. HABADTE already interacts with `HXPXMLD` and `HXPXMLR`, and with `HXFXMLH`/`HXFXMLD`. ****HXPXML may represent:
- An un-modeled XML control or routing table,
- A print/spool configuration for XML-based reports.

Implications:
- **Integration uncertainty:** Missing file makes it hard to establish the complete set of XML-related dependencies for HABADTE, especially if this file controls outbound destinations or formats.

### 2.4 Medium-Impact Gap – PRINTER File

**PRINTER (FILE, impact MEDIUM, referenced by HABADTE)**

HABADTE references a PRINTER file that is not present in the harvested DDS set. In many IBM i solutions this kind of file is used as a spool routing or configuration table mapping report IDs, printers and form types.

Implications:
- **Operational behavior:** The census report’s printer selection logic (e.g., which ward printer receives which census output) cannot be fully replicated.
- **Modernization design:** When migrating to a cloud-native platform, the PRINTER file’s semantics will need to be rediscovered to define equivalent print queues, PDF destinations or electronic delivery channels.

### 2.5 Gap Category Implications Summary

- **High-impact copybook (CXXXMLC):** Blocks full understanding of XML export structures and may hide critical control fields. Addressing this gap is prerequisite for safe replacement of HABADTE’s XML output with modern APIs.
- **Medium-impact programs/files (HXHAPPPRF, ****HXPXML, PRINTER):** Affect secondary concerns – preference resolution, XML configuration and printer routing. They are less likely to break core census filtering logic but will impact completeness of the user experience and integration behavior.

This section is authoritative for missing elements. Later sections focus on present components and their interactions.

---

## 3. Duplicate and Reused Components

The dependency edges reveal several patterns of reuse and centralization.

### 3.1 Reused Programs

Using CALL edges:
- **XFXCNTR** is called three times by HABADTE. This indicates that text centering logic (BR-001, BR-002) is applied in multiple places within the census report to format headings, patient names or labels. The centralized implementation reduces duplication of text alignment routines.
- **XFXCYMD** is called by HABADTE and itself calls XFXLEAP. XFXCYMD encapsulates date validation logic (reject out-of-range years and invalid months/days) and delegates leap-year specific checks to XFXLEAP. This reuse pattern separates general date validation from specialized leap-year computation.
- **XFXMRNROL** is called by HABADTE and in turn calls HXHAPPPRF and HXXAPPPRF. This creates a hub for MRN/account preference logic – HABADTE never calls the preference programs directly, instead using XFXMRNROL.
- **XFXLDSC** is called by HABADTE and performs organizational level lookups across HXPLVL1–HXPLVL6. It acts as a reusable resolver for level codes and descriptions.
- **XFXGETID** is called by HABADTE and reads `HXFXMLR`. It serves as an ID generator or retriever for XML records.

This structure shows a clear application service layer: HABADTE orchestrates patient census logic while delegating reusable utilities (text, dates, identifiers, levels, table lookups) to XFX* programs.

### 3.2 Logical Files Sharing Base Physical Files

From PFILE_OF edges:
- **HXPTABLD** is the base PF for three logical files:
  - HXLTABLD (keyed by XFDTCD + XFDMAP)
  - HXLTABLP (keyed by XFDTCD + XFDLDS)
  - HXLTABLS (keyed by XFDTCD + XFDSDS)

  This pattern indicates a shared dictionary table (HXPTABLD) partitioned logically by different key combinations, supporting mapping lookups for multiple dimensions (mapping codes, logical descriptions, status codes). XFXTABL reads XFFTABLD and related tables, reusing these logical views.

- **TMPMAST** is the base PF for HMLMAST5H, keyed by nursing station and admission date/time. This logical file tailors the patient master subset for the HABADTE census context (e.g., inpatients only).

- **TAPIRNK** is the base PF for HAPIRNK, keyed by level and account sequence. This supports transfer or rank-based reporting.

- **TXPBNFIT** and **TXPNSTN** serve as base PFs for HXPBNFIT and HXPNSTN logical files respectively, providing benefit and nursing station reference views keyed by business identifiers.

These reuse patterns indicate deliberate normalization: one physical file acts as the canonical store, while multiple logical files provide different indexing strategies for specific workflows.

---

## 4. Dependency Analysis

### 4.1 Call Graph Summary

Core call/copy edges:
- **HABADTE** (main driver)
  - Calls: XFXMRNROL, XFXCNTR (three times), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL.
  - Copies: HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC.
  - Reads: HAPTRFR (transfer history), XFFNSTN (nursing station), HXFXMLH.
  - Writes/updates: HXFXMLH, HXFXMLD (XML header/detail records).

- **XFXCYMD**
  - Calls: XFXLEAP for leap-year-specific date validation.

- **XFXMRNROL**
  - Calls: HXHAPPPRF (missing) and HXXAPPPRF (present), both preference-related programs.

- **HXXAPPPRF**
  - Copies: HXXCNTRL, HXXAPPPRFP – likely shared structures and SQL definitions.

- **XFXGETID**
  - Copies: HXXLDA.
  - Declares and reads: HXPXMLR / HXFXMLR for XML-related identifiers.

- **XFXLDSC**
  - Reads HXFLVL1–HXFLVL6 and declares HXPLVL1–HXPLVL6, providing organizational level descriptions.

- **XFXTABL**
  - Reads several dictionary tables (XFFTABLD and more) to resolve status and type codes.

This call graph emphasizes HABADTE as a high-fan-out orchestrator and XFX* / HXX* programs as specialized services.

### 4.2 Hotspot Analysis

From dep_hotspots:

| Program    | Score | Fan-in | Fan-out | File Ops |
|-----------|-------|--------|---------|----------|
| HABADTE   | 38    | 0      | 13      | 6        |
| XFXLDSC   | 15    | 1      | 0       | 6        |
| XFXTABL   | 11    | 1      | 0       | 4        |
| XFXCNTR   | 9     | 3      | 0       | 0        |
| XFXGETID  | 7     | 1      | 1       | 1        |
| HXXAPPPRF | 7     | 1      | 2       | 0        |
| XFXMRNROL | 7     | 1      | 2       | 0        |
| XFXCYMD   | 5     | 1      | 1       | 0        |

Observations:
- **HABADTE is the primary hotspot**: High fan-out to utility programs and multiple file operations. Its cyclomatic complexity (152, HIGH) confirms that most patient management logic resides here – filtering patients, resolving room and nursing station, applying census date rules and composing XML outputs.
- **XFXLDSC and XFXTABL are data access hotspots**: Both have moderate scores driven by multiple file reads. XFXLDSC covers hierarchical organizational levels; XFXTABL centralizes lookups to dictionary tables for status and type codes. Any schema changes in HXPLVL* or XFFTABLD will directly impact these programs.
- **XFXCNTR is a utility hotspot with high reuse but no file IO**: With fan-in=3 and score=9, it focuses purely on in-memory text alignment, making it ideal for migration into a reusable service or shared library function.
- **HXXAPPPRF and XFXMRNROL are preference/identifier hotspots**: They mediate configuration lookups and MRN/account display rules. Their scores reflect their role in controlling how PHI is shown.

Risk assessment:
- **HABADTE** – high change risk and regression surface. Any modernization must be tightly governed with extensive automated tests.
- **XFXLDSC / XFXTABL** – moderate risk hotspots due to schema coupling. Database migration or refactoring must preserve their assumptions on keys and fields.
- **Preference and MRN utilities (HXXAPPPRF, XFXMRNROL)** – moderate compliance risk because they influence PHI display behavior.

---

## 5. Summary of Key Findings

This section focuses on architectural patterns and recommendations, not on missing components.

### 5.1 Architectural Observations

1. **Centralized orchestration in HABADTE**  
   HABADTE is the single, highly complex driver coordinating patient selection, organizational filtering, room and nursing station resolution, and XML/export operations. Dependencies on multiple utilities and files show a layered design: HABADTE orchestrates, XFX*/HXX* perform specific tasks, DDS files store data.

2. **Utility-oriented XFX* layer**  
   The XFX* programs (CNTR, CYMD, LDSC, TABL, GETID, LEAP, MRNROL) form a cohesive utility/service layer. Each program encapsulates a narrow concern: text centering, date validation, organizational level lookup, table lookups, leap year determination, MRN/account preference resolution and ID retrieval.

3. **Normalized dictionary and hierarchy tables**  
   HXPTABLD with its three logical views, HXPLVL1–HXPLVL6 for organizational levels, and HXPDICT as a very large dictionary PF illustrate a normalized design where tables serve as generic configuration and crosswalk structures. Logical files provide concrete business keys for specific workflows.

4. **XML export path integrated into core reporting**  
   HABADTE’s write operations to HXFXMLH/HXFXMLD and use of XML-related copybooks show that census outputs are not purely spool-based – they are also persisted as XML-like records, possibly for downstream systems. This integration of reporting and data interchange is central to modernization design.

5. **PHI concentrated in a small number of master/crosswalk files**  
   PHI flagged fields are limited to HAPTRFR, HXPDICT, OAPIRNK, OMPMAST and OXPBNFIT. HABADTE uses HAPTRFR and nursing station/room-related files to derive patient census views, while XFXMRNROL and preference programs influence how identifiers (MRN/account) are displayed.

### 5.2 Orphan Programs Assessment

The aggregator lists apparent orphan programs: HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC. However, dependency edges clearly show that HABADTE calls XFXCNTR, XFXCYMD, XFXGETID and XFXLDSC, and XFXMRNROL calls HXXAPPPRF. Since their fan-in is non-zero, these are not true orphans.

Conclusion: **No true orphans detected. All programs have defined callers in the scanned codebase.**

### 5.3 Recommendations

1. **Prioritize HABADTE stabilization and test harnesses**  
   Given its high complexity and hotspot score, HABADTE should be the focus of early regression and characterization testing. Build detailed test cases around census date rules, patient inclusion/exclusion, room and nursing station resolution and XML export.

2. **Extract and formalize utility services**  
   XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID and XFXMRNROL should be modeled as discrete services or library modules in the target architecture. This preserves existing reuse and simplifies verification.

3. **Model dictionary/hierarchy tables and logical views explicitly**  
   Create entity models for HXPTABLD/HXLTABL*, HXPLVL*, and HXPDICT, including their PHI-bearing fields. Reflect logical file keys as indexes or projections in the new data model.

4. **Design explicit XML/export and printing interfaces**  
   Replace implicit file-based XML and PRINTER behaviors with explicit APIs or message contracts. This will mitigate the impact of missing components like CXXXMLC and PRINTER.

5. **Address tech debt hotspots**  
   Tech debt summary shows 4 findings (~26.9 hours) with one HIGH severity. These likely align with HABADTE’s complexity and the XML/printing gaps. Remediating them during modernization will reduce long-term maintenance risk.
