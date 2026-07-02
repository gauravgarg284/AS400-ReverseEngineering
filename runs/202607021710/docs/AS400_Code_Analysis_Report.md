# AS400 Code Analysis Report – HABADTE Suite

Run ID: 202607021710  
System: HABADTE AS400 admission and data maintenance suite

---

## 1. Component Inventory

This section summarizes all analyzed source members discovered in the HABADTE codebase. The inventory is derived from the consolidated `source_manifest.json` and represents the technical footprint that downstream modernization work must preserve.

### 1.1 Member Catalogue

| Member | Type    | Subsystem  | Lines |
|--------|---------|-----------|-------|
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

Total analyzed members: 37  
Total lines of source: 9,153

### 1.2 Inventory by Type

The technology mix is dominated by traditional DDS database files and RPG programs, with a small but important presence of embedded SQL.

| Type     | Count |
|----------|-------|
| DDS_LF   | 7     |
| DDS_PF   | 15    |
| SQLRPGLE | 2     |
| RPGLE    | 13    |

Observations:
- The database layer is rich and varied: 22 DDS-based files (physical + logical) support dictionary, benefits, master patient and reference code tables.
- The application layer consists of 13 RPGLE programs plus 2 SQLRPGLE programs, centered around the main HABADTE admission workflow and a suite of data‑maintenance utilities.
- The presence of SQLRPGLE (`HXXAPPPRF`, `CXXXMLP`) indicates partial modernization of data access patterns while the majority of business logic remains in native RPG.

---

## 2. Missing Components

Missing components are nodes referenced by code but not present in the scanned repository. These represent structural gaps that must be addressed before full functional equivalence can be guaranteed in a modernized platform.

We group gaps by impact level to highlight priorities.

### 2.1 High-Impact Gaps

| Name    | Type      | Referenced By | Impact |
|---------|-----------|---------------|--------|
| CXXXMLC | COPYBOOK  | HABADTE       | HIGH   |
| TAPIRNK | FILE      | HAPIRNK       | HIGH   |
| TMPMAST | FILE      | HMLMAST5H     | HIGH   |
| TXPBNFIT| FILE      | HXPBNFIT      | HIGH   |
| TXPNSTN | FILE      | HXPNSTN       | HIGH   |

Implications:
- **CXXXMLC (COPYBOOK)**: HABADTE includes this copy member in its XML handling section. Its absence means that record layouts, constants or XML envelope templates used for admission messages are unknown. Any modernization effort that recreates HABADTE’s XML export/import must reverse‑engineer this copybook from production or additional source libraries.
- **TAPIRNK and TMPMAST (physical files)**: Both are underlying PFs for logical views `HAPIRNK` and `HMLMAST5H`. Their structures drive interface and patient master records. Without them, only the logical key structure is visible; field‑level semantics and PHI distribution for these tables remain partially opaque.
- **TXPBNFIT and TXPNSTN (physical files)**: These sit under benefit and plan status logical files. Their absence complicates reconstruction of benefit entitlements and plan status reference data in the target environment.

### 2.2 Medium-Impact Gaps

| Name        | Type  | Referenced By | Impact |
|-------------|-------|---------------|--------|
| HXHAPPPRF   | PROGRAM | XFXMRNROL   | MEDIUM |
| ****HXPXML  | FILE    | HABADTE     | MEDIUM |
| PRINTER     | FILE    | HABADTE     | MEDIUM |

Implications:
- **HXHAPPPRF (program)**: Called from `XFXMRNROL`, this program likely manages application profiles or MRN roll‑ups. Its absence means MRN normalization behaviour is only partially visible. However, because `HXXAPPPRF` is present and performs SQL operations on application profile tables, the architectural role of HXHAPPPRF can be inferred for design, though detailed branching logic remains unknown.
- ******HXPXML and PRINTER files**: HABADTE references these as IO targets, probably for XML workfiles and print output. While the missing file definitions hinder a complete view of outbound message and report layouts, their impact is medium because they affect presentation and integration rather than core admission decision logic.

---

## 3. Duplicate and Reused Components

This section analyzes reuse patterns based on dependency edges. Reuse in this legacy context is primarily expressed as programs called multiple times and logical files sharing a common physical file.

### 3.1 Programs Reused via CALL

Inspecting CALL edges from the dependency graph:
- `HABADTE` calls:
  - `XFXMRNROL`
  - `XFXCNTR` (three separate call sites)
  - `XFXLDSC`
  - `XFXCYMD`
  - `XFXGETID`
  - `XFXTABL`
- `XFXCYMD` calls `XFXLEAP`.
- `XFXMRNROL` calls `HXHAPPPRF` and `HXXAPPPRF`.

Reused programs (targets of multiple CALL edges):
- **XFXCNTR** – fan‑in 3 and hotspot score 9. This program encapsulates generic counter/branching logic, enforcing rules such as "When X equals zero, branch to EXIT" and "When X equals 40, branch to EXIT". Its repeated invocation from HABADTE indicates it is a shared decision routine used across several admission steps.
- **XFXLDSC** – although only called from HABADTE, it is internally complex (cc=5, file_ops=6) and central to level‑based description logic over the HXPLVLn files. This makes it an internal service class for rate/level mapping.
- **XFXMRNROL** – called from HABADTE and, in turn, calling `HXHAPPPRF` and `HXXAPPPRF`, this program acts as an MRN roll‑up orchestrator, reused anywhere MRN normalization and application profile updates are required.

These reusable components should be refactored into shared services or domain functions in a modern architecture, maintaining their fan‑in relationships to avoid duplication of complex branching logic.

### 3.2 Logical Files Sharing Physical Files

The dependency graph highlights logical‑to‑physical relationships via `PFILE_OF` edges:
- `HAPIRNK` → `TAPIRNK`
- `HMLMAST5H` → `TMPMAST`
- `HXLTABLD`, `HXLTABLP`, `HXLTABLS` → `HXPTABLD`
- `HXPBNFIT` → `TXPBNFIT`
- `HXPNSTN` → `TXPNSTN`

Notable reuse patterns:
- **HXPTABLD** is the physical backbone for three logical views (`HXLTABLD`, `HXLTABLP`, `HXLTABLS`), each projecting the same dictionary content with slightly different key structures. Modernization should consolidate these into well‑named views or queryable reference tables with clear semantics for data type codes, mapping codes, and load descriptors.
- **TXPBNFIT / TXPNSTN** underpin benefit and plan status views. These should become canonical master tables for benefits and status enumerations, with enforceable foreign keys from transactional tables.

---

## 4. Dependency Analysis

### 4.1 Call and Copy Graph

Key dependency edges:

**Application call chains**
- `HABADTE` → `XFXMRNROL` (MRN roll‑up)
- `HABADTE` → `XFXCNTR` (multiple decision points)
- `HABADTE` → `XFXLDSC` (level/description mapping)
- `HABADTE` → `XFXCYMD` (date validation)
- `HABADTE` → `XFXGETID` (identifier allocation / lookup)
- `HABADTE` → `XFXTABL` (dictionary/table lookups)
- `XFXCYMD` → `XFXLEAP` (leap‑year awareness within date validation)
- `XFXMRNROL` → `HXHAPPPRF`, `HXXAPPPRF` (profile updates)

**Copy and shared definitions**
- `HABADTE` copies `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, and missing `CXXXMLC`. These copybooks define shared data structures for admission local data area, level mapping, XML envelopes, and possibly control blocks.
- `HXXAPPPRF` copies `HXXCNTRL` and `HXXAPPPRFP`, composing its SQLRPGLE logic from shared control and profile structures.
- `XFXGETID` copies `HXXLDA`, indicating reuse of the admission local data area definition in identifier logic.

**Database access edges**
- `XFXGETID` reads `HXFXMLR`, tying ID generation to XML response records.
- `XFXLDSC` declares and reads `HXPLVL1`–`HXPLVL6` (and corresponding runtime formats `HXFLVL1`–`HXFLVL6`), forming a level cascade used for rate or coverage determination.
- `XFXTABL` reads a family of dictionary formats `XFFTABLD`, `XFFTABL2`, `XFFTABL3`, `XFFTABL4`.
- `HABADTE` reads `HAPTRFR` (transfer transactions) and `XFFNSTN` (plan status) and writes/updates `HXFXMLH` and `HXFXMLD`, which are part of the XML header/detail workfiles.

### 4.2 Hotspot Summary

Based on the computed hotspot scores:

- **HABADTE (score 38, fan_out 13, file_ops 6, cc 152 – HIGH)**
  - Role: Main admission driver. It orchestrates calls into all supporting data‑maintenance routines and manages IO to transfer, status and XML files.
  - Risk: High complexity and centrality make it the most critical candidate for incremental strangler‑pattern modernization, starting with well‑bounded segments (screen handling, XML MARSHALLING, admission eligibility checks).

- **XFXLDSC (score 15, file_ops 6)**
  - Role: Level description and mapping service, performing multi‑table reads across HXPLVLn to determine descriptive attributes for different coverage or pricing levels.
  - Risk: Medium. Changes here can affect many downstream pricing or eligibility calculations; refactoring should be treated as a shared library change, not a local tweak.

- **XFXTABL (score 11, file_ops 4, cc 9)**
  - Role: Dictionary table lookups, likely mapping code values to descriptive text using `XFFTABLD` and related formats.
  - Risk: Medium. It is an ideal candidate to transform into a configuration microservice or reusable domain utility.

- **XFXCNTR, XFXMRNROL, HXXAPPPRF, XFXGETID**
  - Role: Smaller but important services (branching, MRN roll‑up, profile updates, ID retrieval) with non‑trivial fan‑in.
  - Risk: Low to medium; despite low cyclomatic complexity, incorrect migration can impact core admission behaviour or downstream profiles.

---

## 5. Summary of Key Findings

### 5.1 Critical Structural Issues

Derived from high/medium gaps and orphan program analysis:

| Issue Type | Component   | Details |
|-----------|-------------|---------|
| GAP-HIGH  | CXXXMLC     | Missing copybook used by HABADTE for XML control and layout; without it, full XML payload structure is unknown. |
| GAP-HIGH  | TAPIRNK     | Missing PF behind HAPIRNK; transfer index data model incomplete. |
| GAP-HIGH  | TMPMAST     | Missing PF behind HMLMAST5H; master patient historical view cannot be fully reconstructed. |
| GAP-HIGH  | TXPBNFIT    | Missing PF behind HXPBNFIT; benefit plan mapping tables partially opaque. |
| GAP-HIGH  | TXPNSTN     | Missing PF behind HXPNSTN; plan status reference codes incomplete. |
| GAP-MED   | HXHAPPPRF   | Program called by XFXMRNROL; MRN roll‑up behaviour only partially visible. |
| GAP-MED   | ****HXPXML  | File referenced by HABADTE for XML work; layout unknown. |
| GAP-MED   | PRINTER     | Output file referenced by HABADTE; report formats not captured. |

Orphan programs (no callers in the scanned set):
- `HXXAPPPRF` (SQLRPGLE)
- `XFXCNTR` (RPGLE)
- `XFXCYMD` (RPGLE)
- `XFXGETID` (RPGLE)
- `XFXLDSC` (RPGLE)

These appear as orphans in the dependency graph, but user story traces and calls from HABADTE show they are functionally central. The orphan status is a limitation of the static analysis scope (e.g., missing higher‑level drivers), not an indication that they are unused. Modernization must therefore treat them as active components.

### 5.2 Architectural Observations

- The system exhibits a clear **hub-and-spoke** pattern, with HABADTE as the hub orchestrating specialized spokes (date validation, dictionary access, MRN roll‑up, XML handling).
- Reuse is achieved through **shared copybooks and PFILE logicals**, which should map naturally to shared DTOs and database views in a modern stack.
- All cyclomatic complexity hotspots are concentrated in HABADTE, while supporting programs remain simple, suggesting a viable refactoring strategy: first, carve HABADTE into smaller, well‑named service endpoints that delegate to existing low‑complexity routines.
- PHI is concentrated in a small set of physical files (HAPTRFR, HXPDICT, OXPBNFIT, OAPIRNK, OMPMAST). These need explicit data governance and encryption/masking strategies during migration.

### 5.3 Recommendations for Modernization

1. **Fill High-Impact Gaps Early**: Recover source for CXXXMLC and missing PFs (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) from alternate libraries or production environments before detailed design sign‑off.
2. **Stabilize HABADTE First**: Given its high complexity and central role, define clear functional slices (screen I/O, XML integration, admission eligibility). Wrap each slice with tests based on extracted business rules, then refactor incrementally.
3. **Promote Reused Programs to Services**: XFXCNTR, XFXLDSC, XFXMRNROL, XFXTABL and HXXAPPPRF should map to domain‑specific services (e.g., CounterService, LevelService, MRNService, DictionaryService, ProfileService) with well‑defined contracts.
4. **Normalize Dictionary and Level Tables**: HXPTABLD and HXPLVL1–6 should become strongly typed configuration tables with referential integrity and documented semantics; this will reduce hidden coupling in the modern architecture.
5. **Address PHI Handling**: Files flagged with MRN, account numbers, names, SSN and phone numbers must be wrapped with access control and masking rules. Column‑level lineage already indicates these fields are relatively isolated, easing the introduction of field‑level encryption.

In combination, these actions will ensure that the legacy HABADTE suite can be safely and predictably modernized, preserving its admission and data‑maintenance semantics while improving maintainability and security.
