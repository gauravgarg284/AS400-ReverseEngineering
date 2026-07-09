# Low-Level Design (LLD)

Run ID: 202607091113  
System: Legacy AS/400 (IBM i)

This Low-Level Design document captures architecture, schema, call mappings, and performance/security considerations derived from the reverse engineering summary for the HABAD/HXP/HXX/XFX domains.

---

## 1. Architecture Overview

### 1.1 Technology Stack and Component Mix

Based on `source_by_type`, the codebase consists of:

- 19 DDS physical files (DDS_PF)
- 7 DDS logical files (DDS_LF)
- 13 RPGLE programs
- 2 SQLRPGLE programs

Interpretation:

- The solution is **data‑centric**, with a large DDS schema underpinning transactional logic.
- RPGLE programs provide the primary control flow, orchestrating business processing and I/O.
- SQLRPGLE appears in targeted modules (e.g., HXXAPPPRF, CXXXMLP), probably for SQL‑based access to newer tables or external databases.

### 1.2 Call Graph Summary

Key CALL edges:

- `HABADTE` → `XFXMRNROL`
- `HABADTE` → `XFXCNTR` (three calls)
- `HABADTE` → `XFXLDSC`
- `HABADTE` → `XFXCYMD`
- `HABADTE` → `XFXGETID`
- `HABADTE` → `XFXTABL`
- `XFXCYMD` → `XFXLEAP`
- `XFXMRNROL` → `HXHAPPPRF` (missing program)
- `XFXMRNROL` → `HXXAPPPRF`

COPY edges provide shared code and structures:

- `HXXAPPPRF` → `HXXCNTRL`, `HXXAPPPRFP`
- `XFXGETID` → `HXXLDA`
- `HABADTE` → `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, `CXXXMLC`

PFILE_OF edges illustrate logical-to-physical relationships, and READ/WRITE edges describe file I/O.

### 1.3 Observed Design Patterns

- **Controller + Utility Pattern** – HABADTE acts as a controller delegating specialized tasks to utilities (`XFX*` programs). This is similar to a service façade calling multiple domain services.
- **Shared Data Access Layer** – XFXLDSC and XFXTABL encapsulate read logic for hierarchical and tabular configuration data. They function as a rudimentary DAL (Data Access Layer).
- **Copy‑Based Reuse** – Common structures and control blocks are centralized in copybooks (CXXXMLC, HXXCNTRL, HXXAPPPRFP, HXXLDA, HXXLEVEL, HXXXML). These correspond to shared DTOs/configs in a modern architecture.

---

## 2. Database Schema

This section describes the physical and logical files from `data_dict_schema`. For fields, the full listing is available in the data dictionary; here we emphasize structural attributes and PHI indicators.

### 2.1 Physical Files (PF)

Each PF entry includes record format, key structure, uniqueness, and PHI field presence.

#### HAPTRFR

- Record format: `HAFTRFR`
- Unique: `true`
- Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- Total fields: 28
- PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

Design notes:

- Multi‑column key suggests a composite identity of level‑6 entity, account, transfer date/time, and type.
- Presence of account and MRN identifiers means this file is central to PHI‑bearing transfer transactions.

#### HXPDICT

- Record format: `HXFDICT`
- Unique: `false`
- Key fields: none (dictionary/lookup style)
- Total fields: 2705
- PHI fields include `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`

Design notes:

- Large, non‑unique dictionary containing mixed clinical and administrative identifiers.
- Likely acts as a centralized code/value or cross‑reference dictionary, but carries significant PHI density.

#### HXPLVL1 – HXPLVL6

- Record formats: `HXFLVL1` … `HXFLVL6`
- Unique: `true` for each
- Keys: `HX1NUM`, `HX2NUM`, `HX3NUM`, `HX4NUM`, `HX5NUM`, `HX6NUM` respectively
- Total fields: 36 to 155, no PHI fields flagged

Design notes:

- Represent a multi‑level hierarchy (e.g., plan/benefit levels 1–6).
- Accessed via `XFXLDSC` which reads all six levels; this indicates layered configuration rather than patient‑level data.

#### HXPTABLD

- Record format: `XFFTABLD`
- Unique: `false`
- Key fields: `XFDTCD`, `XFDECD`
- Total fields: 7
- PHI fields: none

Design notes:

- Compact code table with dual keys (code + description) used by `XFXTABL`.

#### HXPXMLD and HXPXMLR

- `HXPXMLD` – Record format `HXFXMLD`, key `XMDUSR`, `XMDSEQ`, `XMDSQ2`, unique `true`, total fields 4.
- `HXPXMLR` – Record format `HXFXMLR`, key `XMRUSR`, `XMRSEQ`, `XMRID`, unique `true`, total fields 4.

Design notes:

- These appear to represent XML detail (`D`) and request/response headers (`R`) indexed by user and sequence identifiers.

#### OAPIRNK, OMPMAST, OXPBNFIT

- **OAPIRNK**
  - Record format: `HBFIRNK`
  - Unique: `true`
  - Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
  - Total fields: 33
  - PHI fields: `BRKMRN` (MRN)

- **OMPMAST**
  - Record format: `HMFMAST`
  - Unique: `true`
  - Keys: `MMPLV6`, `MMACCT`
  - Total fields: 149
  - PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`

- **OXPBNFIT**
  - Record format: `XFFBNFIT`
  - Unique: `true`
  - Keys: `XFBUBN`, `XFBPLN`
  - Total fields: 34
  - PHI fields: `XFBTEL` (PhoneNumber)

Design notes:

- These PFs underpin member profile and benefit data; OMPMAST is particularly sensitive as it holds MRN, account, name, and SSN.

#### OXPNSTN, TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN

- `OXPNSTN` – unique PF keyed by `XFNLV6`, `XFNSST` with 23 fields.
- `TAPIRNK`, `TMPMAST`, `TXPBNFIT`, `TXPNSTN` – “ATE TABLE” record format, non‑unique, with total fields 43, 181, 12, and 19 respectively.

Design notes:

- TAPIRNK/TMPMAST/TXPBNFIT/TXPNSTN serve as base PFs for logical views (HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN). They likely hold common, possibly environment‑specific sets of records.

### 2.2 Logical Files (LF)

Each LF references a PFILE and provides a particular key sequence or subset.

#### HAPIRNK

- PFILE: `TAPIRNK`
- Record format: `HBFIRNK`
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Select/Omit: none

HAPIRNK provides a logical view over TAPIRNK keyed by level‑6 benefit and account sequence, enabling efficient retrieval of ranking or break records.

#### HMLMAST5H

- PFILE: `TMPMAST`
- Record format: `HMFMAST`
- Key fields: `MMPNST`, `MMADDT`, `MMADTM`
- Select/Omit: none

This LF provides chronological access to master profile records by plan/state and added date/time.

#### HXLTABLD, HXLTABLP, HXLTABLS

- PFILE: `HXPTABLD`
- Record format: `XFFTABLD`

Key fields:

- HXLTABLD: `XFDTCD`, `XFDMAP`
- HXLTABLP: `XFDTCD`, `XFDLDS`
- HXLTABLS: `XFDTCD`, `XFDSDS`

These LFs expose different alternate indexes over the same code table, facilitating various lookup patterns.

#### HXPBNFIT, HXPNSTN

- `HXPBNFIT` – PFILE `TXPBNFIT`, format `XFFBNFIT`, keys `XFBUBN`, `XFBPLN`.
- `HXPNSTN` – PFILE `TXPNSTN`, format `XFFNSTN`, keys `XFNLV6`, `XFNSST`.

These LFs present benefit and state/plan information for operational processing.

---

## 3. Status and Type Reference Data

The `approved_rules` list contains business rules that include validation logic and implicit status handling:

- BR-001/BR-002 – Field formatting rules (exit if all blank; exit if first character non‑blank).
- BR-003/BR-004/BR-005 – Date validation rules (reject year < 1800, year > 2100, month < 01).

No explicit status code tables are embedded in these rules (e.g., no “STATUS = ‘A’/‘I’” tokens in the extracted text). Instead, status decisions are implemented procedurally.

Status and type reference data therefore appear to be driven primarily by:

- Dictionary/code tables such as HXPTABLD/XFFTABLD and XFFTABL*.
- Status PFs like XFFNSTN accessed via HABADTE.

Because the rule text does not enumerate explicit status codes, this section records:

- **Status codes discovered from approved_rules: None identified.**

Reference data for statuses and types must be derived from the code tables and PFs themselves rather than from the extracted rule corpus.

---

## 4. Stored Procedure Logic Mappings

Here we treat RPGLE/SQLRPGLE programs as stored‑procedure‑like units. Mappings are based on CALL edges grouped by caller.

### 4.1 HABADTE

Callees:

- `XFXMRNROL` – MRN role management.
- `XFXCNTR` (three calls) – Counter/formatter routine used in multiple branches.
- `XFXLDSC` – Hierarchy descriptor loader.
- `XFXCYMD` – Date validation and calendar logic.
- `XFXGETID` – Identifier retrieval (reads HXFXMLR).
- `XFXTABL` – Generic table lookup.

Call characteristics:

- Total distinct callees: 6
- Total CALL edges: 8 (including repeated calls to XFXCNTR)

Design intent:

- HABADTE orchestrates a complex workflow where member transfers or benefits are processed, validated, and then rendered as XML or print output.

### 4.2 XFXCYMD

Callees:

- `XFXLEAP` – Leap‑year helper.

Design intent:

- Encapsulated date validation: XFXCYMD evaluates the date range rules (1800–2100, month ≥ 01) and delegates leap‑year logic to XFXLEAP.

### 4.3 XFXMRNROL

Callees:

- `HXHAPPPRF` – Missing program, profile handler.
- `HXXAPPPRF` – SQLRPGLE profile processing program.

Design intent:

- XFXMRNROL is a dispatcher for profile‑related operations, bridging from HABADTE to SQL‑driven profile logic in HXXAPPPRF and to legacy HXHAPPPRF.

### 4.4 HXXAPPPRF

Callees via copy (COPY edges):

- `HXXCNTRL`
- `HXXAPPPRFP`

These copybooks bring in control structures and possibly parameter lists for profile operations.

### 4.5 XFXGETID

Callees:

- `HXXLDA` (COPY) – Local data area or shared structure.

XFXGETID also performs a READ of HXFXMLR to retrieve identifiers for XML payloads.

### 4.6 Additional Callers

Other programs use copybooks for shared behavior:

- `HABADTE` → `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, `CXXXMLC`

These copy relationships define shared control blocks and XML payload layouts.

---

## 5. Service Class Method Reference

Using `dep_hotspots`, we categorize programs into service classes based on their score and usage.

Classification rules:

- Score ≥ 25 → BatchService
- Score 10–24 → WorkflowService
- Score < 10 → UtilityService

### 5.1 BatchService

- **HABADTE** (score 38, fan_out 13, file_ops 6) – BatchService

Responsibilities:

- Drives end‑to‑end benefit or transfer batch processes.
- Coordinates MRN role updates, table lookups, hierarchy loading, and XML output.

### 5.2 WorkflowService

- **XFXLDSC** (score 15) – WorkflowService
- **XFXTABL** (score 11) – WorkflowService

Responsibilities:

- XFXLDSC: Orchestrates sequential reads across hierarchical HXPLVL* tables.
- XFXTABL: Provides generalized lookup pipeline across multiple XFFTABL* tables.

### 5.3 UtilityService

All other hotspot entries are classed as UtilityService:

- XFXCNTR (score 9, fan_in 3)
- XFXMRNROL (score 7)
- XFXGETID (score 7)
- HXXAPPPRF (score 7)
- XFXCYMD (score 5)
- XFXLEAP (score 3)
- HXHAPPPRF (score 3)
- HMLMAST5H, HXLTABLD, HXPNSTN, HAPIRNK, HXLTABLS, HXLTABLP, HXPBNFIT (scores 2)

These utilities encapsulate focused logic units that should map cleanly to microservices or shared library methods in a modernized platform.

---

## 6. External Interfaces

External interfaces are inferred from high‑impact gaps and file operations.

### 6.1 Inbound Interfaces (High/Medium Gaps)

From `high_medium_gaps`:

- `CXXXMLC` (COPYBOOK, HIGH) – Defines XML‑related structures used by HABADTE. Acts as an inbound contract for XML message layouts.
- `HXHAPPPRF` (PROGRAM, MEDIUM) – Externalized profile handler, invoked by XFXMRNROL.
- `****HXPXML` (FILE, MEDIUM) – XML‑related file, likely an inbound/outbound XML staging area.

These components are treated as external interfaces because they are missing from the harvested set but referenced by the codebase. Their behavior and data contracts must be obtained from external libraries or documentation before full migration.

### 6.2 Outbound Print / Spool Interfaces

The `PRINTER` file is referenced by HABADTE but not present in the harvested DDS. This strongly suggests an external printer file or output queue definition.

The lack of additional spool‑related artifacts in the summary leads to the following statement:

- **No outbound SPOOL interfaces identified.**

Any print behavior is likely localized within HABADTE and its missing PRINTER file definition.

---

## 7. Performance and Security Notes

### 7.1 Complexity and Performance Hotspots

From `complexity_per_program`:

- **HABADTE** – cyclomatic complexity 152, band HIGH.
- All other measured programs (HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, CXXXMLP, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML) are in the LOW band with cc values between 1 and 9.

Performance implications:

- HABADTE is the primary candidate for performance tuning and refactoring. Its high complexity can lead to:
  - Difficult branch prediction and cache behavior in compiled code.
  - Higher risk of unoptimized paths and duplicated logic.
- Utility programs are relatively simple and should not be major performance bottlenecks.

### 7.2 PHI Exposure

From `phi_flagged_fields`, PHI is concentrated in:

- **HAPTRFR** – AFACCT, AFMRNO.
- **HXPDICT** – CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME.
- **OAPIRNK** – BRKMRN.
- **OMPMAST** – MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN.
- **OXPBNFIT** – XFBTEL.

Security considerations:

- Any modernization must enforce **field‑level masking and encryption** for MRN, account, SSN, and contact details.
- Data lineage shows that HABADTE reads HAPTRFR and other PHI‑bearing files and writes XML structures (HXFXMLD/HXFXMLH). XML payloads therefore inherit PHI and must be treated as sensitive.

### 7.3 Technical Debt Summary

From `tech_debt_summary`:

- Total findings: 4
- Total remediation hours: 26.9
- By severity:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

Interpretation:

- The bulk of technical debt is concentrated in a few areas, likely HABADTE and surrounding utilities.
- Estimated remediation effort (~27 hours) is moderate, but given HABADTE’s central role, these hours should be prioritized early in any modernization wave.

### 7.4 Security and Compliance Recommendations

- Introduce **centralized PHI handling** services to replace ad‑hoc field usage in HABADTE and HXPDICT.
- Apply **access controls** around OMPMAST and HXPDICT, which combine identity and clinical context.
- Implement **audit logging** for HABADTE’s interactions with HXFXML* files to track PHI exports.

These recommendations align the legacy design with contemporary security and compliance expectations.
