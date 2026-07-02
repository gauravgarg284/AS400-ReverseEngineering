# Low-Level Design (LLD) – HABADTE AS400 Suite

Run ID: 202607021710  
System: HABADTE AS400 admission and data maintenance suite

---

## 1. Architecture Overview

### 1.1 Technology Stack and Component Types

The HABADTE suite is built on IBM i / AS400 using classic technologies:

- **RPGLE programs (13)** – Implement business logic for admission processing and data maintenance (e.g., `HABADTE`, `XFXCNTR`, `XFXCYMD`, `XFXLDSC`, `XFXMRNROL`, `XFXTABL`).
- **SQLRPGLE programs (2)** – Encapsulate SQL-based operations for application profiles and XML processing (e.g., `HXXAPPPRF`, `CXXXMLP`).
- **DDS physical files (15)** – Persist transactional, master and reference data (`HAPTRFR`, `HXPDICT`, `HXPLVL1–6`, `HXPTABLD`, `HXPXMLD`, `HXPXMLR`, `OAPIRNK`, `OMPMAST`, `OXPBNFIT`, `OXPNSTN`).
- **DDS logical files (7)** – Provide indexed and filtered views over PFs (`HAPIRNK`, `HMLMAST5H`, `HXLTABLD`, `HXLTABLP`, `HXLTABLS`, `HXPBNFIT`, `HXPNSTN`).

The distribution by type confirms a traditional layered architecture: a robust DDS-based data layer, a procedural business layer in RPG, and selective SQL usage to access normalized relational tables.

### 1.2 Call Graph Summary

Key call relationships from the dependency graph:

- **Main driver**
  - `HABADTE` → `XFXMRNROL` (MRN roll-up orchestration)
  - `HABADTE` → `XFXCNTR` (branching on counters; multiple call sites)
  - `HABADTE` → `XFXLDSC` (level/description mapping)
  - `HABADTE` → `XFXCYMD` (date validation and normalization)
  - `HABADTE` → `XFXGETID` (identifier retrieval)
  - `HABADTE` → `XFXTABL` (dictionary lookups)

- **Supporting chains**
  - `XFXCYMD` → `XFXLEAP` (leap-year date logic).
  - `XFXMRNROL` → `HXHAPPPRF` and `HXXAPPPRF` (application profile updates).

Copybook dependencies:
- `HABADTE` includes `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, and the missing `CXXXMLC`.
- `XFXGETID` includes `HXXLDA`.
- `HXXAPPPRF` includes `HXXCNTRL` and `HXXAPPPRFP`.

### 1.3 Design Patterns Observed

- **Hub-and-Spoke Controller**: `HABADTE` acts as the central controller, delegating specialized tasks to focused utility programs. This maps well to a modern façade or API gateway plus downstream services.
- **Shared Data Structures via Copybooks**: Copy members (`HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, `HXXCNTRL`, `HXXAPPPRFP`) define common record layouts and control blocks. These should become shared DTOs or schema‑first model classes.
- **Database Encapsulation**: Programs like `XFXLDSC` and `XFXTABL` encapsulate multi-table and dictionary access behind procedural interfaces, which can be translated into repository/service layers.

---

## 2. Database Schema

This section describes the compact schema for physical and logical files as extracted into `data_dict_schema`.

### 2.1 Physical Files (PFs)

Each PF subsection summarizes record format, key structure, uniqueness, size, and PHI flags.

#### 2.1.1 HAPTRFR

- **Record format**: `HAFTRFR`
- **Unique key**: Yes
- **Key fields**: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- **Total fields**: 28
- **PHI fields**: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

Design notes:
- Represents transfer transactions at level 6 (`AFLVL6`) for a particular account and transfer date/time.
- Forms the backbone of admission transfer history and must be migrated with strong referential integrity to patient and account master tables.

#### 2.1.2 HXPDICT

- **Record format**: `HXFDICT`
- **Unique key**: No
- **Key fields**: none defined in compact schema (likely multi-column dictionary keys in DDS).
- **Total fields**: 2705
- **PHI fields**: `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`

Design notes:
- Large dictionary file used to store a wide variety of reference and possibly semi‑master data, including MRNs, contact details and room/account mapping.
- Given its size and PHI density, modernization should split this into domain-specific tables (patient, location, contact, account) to avoid a monolithic dictionary pattern.

#### 2.1.3 HXPLVL1–HXPLVL6

- **Record formats**: `HXFLVL1`, `HXFLVL2`, `HXFLVL3`, `HXFLVL4`, `HXFLVL5`, `HXFLVL6`
- **Unique keys**: Yes
- **Key fields**:
  - L1: `HX1NUM`
  - L2: `HX2NUM`
  - L3: `HX3NUM`
  - L4: `HX4NUM`
  - L5: `HX5NUM`
  - L6: `HX6NUM`
- **Total fields**: 36–42 for L1–L5; 155 for L6
- **PHI**: None flagged

Design notes:
- These level tables hold multi-tier configuration or rating information, which `XFXLDSC` reads to resolve levels into descriptions or attributes.
- Modern representation should treat them as normalized configuration tables with foreign keys between levels rather than loosely related PFs.

#### 2.1.4 HXPTABLD

- **Record format**: `XFFTABLD`
- **Unique key**: No
- **Key fields**: `XFDTCD`, `XFDECD`
- **Total fields**: 7
- **PHI**: None flagged

Design notes:
- Core dictionary of data type codes (`XFDTCD`) and dictionary entries (`XFDECD`). Logical files slice this dictionary by mapping (`XFDMAP`), level descriptor (`XFDLDS`), and status descriptor (`XFDSDS`).

#### 2.1.5 HXPXMLD / HXPXMLR

- **HXPXMLD**
  - Record format: `HXFXMLD`
  - Unique key: Yes
  - Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
  - Total fields: 4
  - PHI: None flagged

- **HXPXMLR**
  - Record format: `HXFXMLR`
  - Unique key: Yes
  - Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`
  - Total fields: 4
  - PHI: None flagged

Design notes:
- These are XML header/detail workfiles for outbound/inbound messages. HABADTE writes `HXFXMLD` and reads `HXFXMLH` (header), while `XFXGETID` interacts with `HXFXMLR` for response records.

#### 2.1.6 OAPIRNK

- Record format: `HBFIRNK`
- Unique key: Yes
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Total fields: 33
- PHI fields: `BRKMRN` (MRN)

Design notes:
- Index/summary of transfer or break records at level 6. Contains MRN for traceability.

#### 2.1.7 OMPMAST

- Record format: `HMFMAST`
- Unique key: Yes
- Key fields: `MMPLV6`, `MMACCT`
- Total fields: 149
- PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`

Design notes:
- Patient master file; a core PHI repository. Must be migrated with robust encryption, access control and audit logging.

#### 2.1.8 OXPBNFIT

- Record format: `XFFBNFIT`
- Unique key: Yes
- Key fields: `XFBUBN`, `XFBPLN`
- Total fields: 34
- PHI fields: `XFBTEL` (PhoneNumber)

Design notes:
- Benefit plan file, where `XFBTEL` likely stores contact information for benefits. Forms the master for coverage data.

#### 2.1.9 OXPNSTN

- Record format: `XFFNSTN`
- Unique key: Yes
- Key fields: `XFNLV6`, `XFNSST`
- Total fields: 23
- PHI: None flagged

Design notes:
- Plan status file keyed by level and status codes.

### 2.2 Logical Files (LFs)

Logical files provide tailored views over PFs.

#### 2.2.1 HAPIRNK

- **Based on PF**: `TAPIRNK` (missing from repository)
- **Record format**: `HBFIRNK`
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`
- **Select/Omit**: None in compact schema

Design notes:
- Logical view over transfer index, keyed to quickly retrieve transfer sequences by account and level.

#### 2.2.2 HMLMAST5H

- **Based on PF**: `TMPMAST` (missing)
- **Record format**: `HMFMAST`
- **Key fields**: `MMPNST`, `MMADDT`, `MMADTM`
- **Select/Omit**: None

Design notes:
- Historical patient master view keyed by patient status and admission date/time.

#### 2.2.3 HXLTABLD / HXLTABLP / HXLTABLS

- **PF**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Keys**:
  - HXLTABLD: `XFDTCD`, `XFDMAP`
  - HXLTABLP: `XFDTCD`, `XFDLDS`
  - HXLTABLS: `XFDTCD`, `XFDSDS`

Design notes:
- These LFs partition the dictionary by different secondary keys to support flexible lookups for mapping codes, level descriptors and status descriptors.

#### 2.2.4 HXPBNFIT

- **PF**: `TXPBNFIT` (missing)
- **Record format**: `XFFBNFIT`
- **Key fields**: `XFBUBN`, `XFBPLN`

#### 2.2.5 HXPNSTN

- **PF**: `TXPNSTN` (missing)
- **Record format**: `XFFNSTN`
- **Key fields**: `XFNLV6`, `XFNSST`

Design notes:
- Both benefit and status logical files sit over missing PFs, underscoring the need to recover those physical definitions for complete schema design.

---

## 3. Status and Type Reference Data

Approved business rules provide limited insight into explicit status or type codes used in the system. The extracted rule texts do not contain stable code lists (they are mainly conditional branches), so no dedicated status-code catalogue can be built from them.

From `approved_rules` and interpretation narratives:
- Most rules describe threshold checks (e.g., VYY, VMM, VDD ranges), indicator flags (e.g., `*IN79` on/active) and file/flag indicators (void/voided, outpatient), but no enumerated code tables are spelled out.

Status and type reference conclusion:
- **Status codes in rules**: None identified directly as maintainable reference tables.
- **Reference data**: Likely resides in dictionary files like `HXPTABLD`, `OXPBNFIT`, `OXPNSTN`, but details are embedded in PF definitions outside the compact ruleset.

Therefore, for this LLD section:
- Status and Type Reference Data: **None identified in this codebase.**

---

## 4. Stored Procedure Logic Mappings

Even though the platform uses RPG rather than SQL stored procedures, call edges can be treated as procedure mappings between caller and callee.

### 4.1 Call Groups by Caller

#### HABADTE (main admission controller)

Callees:
- `XFXMRNROL` – MRN roll-up service
- `XFXCNTR` (three calls) – counter-based branching logic
- `XFXLDSC` – level description service
- `XFXCYMD` – calendar/date validation
- `XFXGETID` – ID retrieval service
- `XFXTABL` – dictionary lookup service

These calls implement the user stories around file/flag indicators and patient status, by delegating specific checks to focused programs.

#### XFXCYMD

Callees:
- `XFXLEAP` – leap-year calculation.

Role:
- Provides robust date validation using year (VYY), month (VMM) and day (VDD), enforcing ranges and leap-year rules before HABADTE accepts an admission date.

#### XFXMRNROL

Callees:
- `HXHAPPPRF` (missing) – application profile handler.
- `HXXAPPPRF` – SQL program accessing application profile tables (e.g., HXPAPPPRF, HXPAPPL6).

Role:
- Coordinates MRN roll-up and profile adjustments, ensuring patient identifiers and associated application profiles are consistent.

#### HXXAPPPRF

Copies:
- `HXXCNTRL`, `HXXAPPPRFP` – provide control structures and profile layouts.

Role:
- Executes SQL statements to read/update profile records, representing a bridge between traditional RPG logic and normalized relational tables.

#### XFXGETID

Callees / data dependencies:
- Copies `HXXLDA` for local data area definition.
- Reads `HXFLVL1` and `HXFXMLR` via data lineage.

Role:
- Handles retrieval or assignment of identifiers based on XML response records and configuration.

### 4.2 Call Count Perspective

Programs can be categorized by the number of incoming calls:
- High fan-in: `XFXCNTR` (multiple calls from HABADTE), `XFXTABL`, `XFXLDSC`.
- Medium: `XFXMRNROL`, `XFXCYMD`, `XFXGETID`, `HXXAPPPRF`.
- Low: `XFXLEAP`, `HXHAPPPRF`.

In a target architecture, high fan‑in components should become stateless, idempotent services with clearly versioned interfaces to prevent tight coupling.

---

## 5. Service Class Method Reference

Using the hotspot scores, we classify programs into conceptual service classes for modernization:

### 5.1 BatchService (High Score)

Characteristics: high hotspot score, broad fan‑out, significant file operations.

- **HABADTE (score 38)** – BatchService
  - Methods: admission processing workflow, XML header/detail generation, transfer and plan status updates.
  - Should become a set of orchestrated batch or workflow services (e.g., `AdmissionService`, `TransferService`, `XmlIntegrationService`) or orchestrations in a workflow engine.

### 5.2 WorkflowService (Medium Score)

Characteristics: moderate score, focused on orchestrating or evaluating data across several tables.

- **XFXLDSC (score 15)** – WorkflowService
  - Methods: level description computation, reading HXPLVL1–6 and HXFLVL1–6; mapping levels to attributes.

- **XFXTABL (score 11)** – WorkflowService
  - Methods: dictionary lookup across XFFTABLD and related formats; resolution of codes into human‑readable descriptions.

- **XFXCNTR (score 9)** – WorkflowService
  - Methods: branch control based on counter `X` (checks 0, 40, etc.), used at multiple decision points in HABADTE.

- **HXXAPPPRF (score 7)** – WorkflowService
  - Methods: SQL-based application profile maintenance driven by calls from XFXMRNROL.

- **XFXGETID, XFXMRNROL, XFXCYMD (scores 5–7)** – WorkflowServices
  - Methods: ID retrieval, MRN roll-up orchestration, date validation.

### 5.3 UtilityService (Low Score)

Characteristics: low fan‑in, specialized calculations or formatting.

- **XFXLEAP (score 3)** – UtilityService
  - Method: leap-year determination, reused by XFXCYMD.

- **HXLTABLD / HXLTABLP / HXLTABLS / HAPIRNK / HMLMAST5H / HXPBNFIT / HXPNSTN (scores 2)** – UtilityService
  - Methods: lightweight logical views; in a modern stack these map to database views or simple repository queries.

---

## 6. External Interfaces

### 6.1 Inbound Interfaces (High-Impact Gaps)

High and medium impact gaps reveal external touch points:

- **CXXXMLC (COPYBOOK)** – Defines segments of XML messages processed by HABADTE and CXXXMLP. As it is missing, its fields and message structure must be reconstructed from live environments or supplementary documentation. It likely represents inbound/outbound XML schemas for admission events.
- **TAPIRNK / TMPMAST / TXPBNFIT / TXPNSTN (PFs)** – These PFs underpin logical files used by HABADTE and related programs. Their references suggest files loaded from external systems (e.g., eligibility, plan, and patient master feeds).
- ******HXPXML (FILE)** – Referenced by HABADTE as part of XML processing; may serve as an intermediate workfile for inbound/outbound XML messages.

These components should be modeled as explicit integration points in the modern system, with well-defined contracts (e.g., JSON/XML schemas, API endpoints, batch file interfaces).

### 6.2 Outbound SPOOL / Print Interfaces

HABADTE references a `PRINTER` file but no DDS definition is present in the scanned repository. Static analysis therefore cannot fully describe the outbound print/report behaviour.

Outbound SPOOL interfaces conclusion:
- **No outbound SPOOL interfaces identified.**

(From the available metadata: we see a PRINTER file name, but no concrete spool file or report definition to confirm a structured print interface.)

---

## 7. Performance and Security Notes

### 7.1 Performance Analysis (Cyclomatic Complexity)

Cyclomatic complexity per program:

- `HABADTE`: cc=152 (HIGH band)
- Supporting programs: cc ranges from 1 to 9 (LOW band) – `XFXCNTR` (3), `XFXCYMD` (7), `XFXLDSC` (5), `XFXTABL` (9), others at 1.

Implications:
- HABADTE is a clear performance risk if migrated monolithically. It likely contains nested decision logic for admission screening, XML processing and file I/O in a single large routine.
- Supporting programs are relatively simple, suggesting a refactoring strategy where HABADTE is decomposed into calls to more granular services, while retaining existing supporting program structure as reference implementations.

Recommended actions:
- Introduce profiling and tracing when running HABADTE workloads pre‑migration, focusing on hotspots around XML and file updates.
- During modernization, implement asynchronous workflows or separate microservices for complex segments (e.g., MRN roll-up, benefits and status evaluation) to reduce single-thread bottlenecks.

### 7.2 Security and PHI Exposure

PHI-flagged files and fields:

- `HAPTRFR`: `AFACCT` (AccountNumber), `AFMRNO` (MRN)
- `HXPDICT`: MRN, account and contact fields including `CCMRNO`, `XFBTEL`, `XCNAME`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`
- `OAPIRNK`: `BRKMRN` (MRN)
- `OMPMAST`: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`
- `OXPBNFIT`: `XFBTEL` (PhoneNumber)

Total PHI fields: 22 across 5 files.

Security implications:
- Patient master and transfer records contain MRN, account, name and SSN, making them high‑risk from a compliance standpoint.
- Dictionary files (HXPDICT) mix PHI with configuration/reference data, complicating access control design.

Recommended security measures:
- Introduce field‑level encryption or tokenization for MRN, account numbers, names and SSNs in the modern database.
- Separate PHI-bearing columns into dedicated protected schemas or views, with strict role-based access controls.
- Implement audit logging for all CRUD operations involving PHI fields.

### 7.3 Tech Debt Summary

From `tech_debt_summary`:

- **Total findings**: 4
- **Total remediation hours (estimate)**: 26.9
- **By severity**:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

The concentration of tech debt in high/medium severity categories aligns with the structural gaps (missing copybooks/files) and high-complexity routines (HABADTE). Addressing these items early in the modernization program will reduce overall risk and rework.

In summary, the HABADTE suite exhibits a clear separation between a complex admission controller and a set of relatively simple specialized utilities. Its DDS-based schema is rich and PHI‑heavy but well structured around level, dictionary, benefits and patient master tables. With careful refactoring and stringent PHI controls, the system can be transformed into a modular, secure and performant cloud-native architecture.
