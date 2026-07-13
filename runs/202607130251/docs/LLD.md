# Low-Level Design (LLD) – HABADTE AS400 Application

## Section 1 – Architecture Overview

### 1.1 Technology Stack and Component Types

The HABADTE application is a traditional IBM i (AS400) workload that combines DDS-based database definitions with procedural RPG and SQLRPGLE programs. The component mix by type is:

- **DDS_LF (logical files):** 7
- **DDS_PF (physical files):** 19
- **RPGLE programs:** 13
- **SQLRPGLE programs:** 2

This mix reflects a schema‑rich environment where data structures are modelled via PFs and LFs, while application logic is primarily host-based RPG. SQLRPGLE programs (HXXAPPPRF, CXXXMLP) represent more recent extensions that rely on embedded SQL for data access.

### 1.2 Call Graph and Control Flow

The call graph is anchored on the HABADTE program, which operates in the **PATIENT_MANAGEMENT** domain and orchestrates several specialised utilities:

- HABADTE calls **XFXMRNROL**, which in turn calls **HXHAPPPRF** (missing) and **HXXAPPPRF**. This path represents MRN roll and application profile updates.
- HABADTE calls **XFXCNTR** multiple times to perform counter and field‑formatting logic.
- HABADTE calls **XFXLDSC** for hierarchical level lookups across HXPLVL1–HXPLVL6.
- HABADTE calls **XFXCYMD**, which calls **XFXLEAP** to perform date and leap‑year validation.
- HABADTE calls **XFXGETID**, which interacts with HXFXMLR to retrieve XML‑related identifiers.
- HABADTE calls **XFXTABL**, which reads table definitions from XFFTABLD and related tables.

Additionally, several programs include copy members:

- **HXXAPPPRF** copies **HXXCNTRL** and **HXXAPPPRFP**, centralising shared control and profile logic.
- **XFXGETID** and **HABADTE** both copy **HXXLDA**, consolidating local data area structures.
- **HABADTE** also copies **HXXLEVEL**, **HXXXML**, **CXXXMLP**, and the missing **CXXXMLC**, linking it tightly to the XML control layer.

These relationships yield a hub‑and‑spoke architecture: HABADTE is the hub, and the XFX* and HXX* modules form spokes that encapsulate reusable behaviours.

### 1.3 Design Patterns

Three notable design patterns emerge:

1. **Service‑style utility programs:** XFXCYMD (date validation), XFXLDSC (level lookup), XFXTABL (table lookup), and XFXCNTR (field formatting) behave as shared services called from HABADTE and potentially from other drivers.
2. **Copybook-driven configuration:** Control structures, constants, and XML layouts are abstracted into copy members (HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC), allowing shared configuration without code duplication.
3. **PF/LF separation:** Physical files store canonical data, while logical files provide alternate keyed views aligned to specific business flows (e.g., rank, benefit, and station views).

These patterns should be preserved in any modern design, for example by mapping utility programs to microservices and copybooks to shared configuration modules.

## Section 2 – Database Schema

The database schema is derived from the compact **data_dict_schema**. Each PF and LF is described below.

### 2.1 Physical Files (PF)

#### 2.1.1 HAPTRFR – Transfer File
- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

HAPTRFR stores transfer‑related records keyed by level‑6 code, account, transfer date/time, and type. The presence of account and MRN fields indicates that each record is directly associated with a patient account and encounter.

#### 2.1.2 HXPDICT – Dictionary / Master Data File
- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** (none defined as unique in the schema summary)
- **Total fields:** 2705
- **PHI fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HXPDICT is a very wide dictionary file containing a large number of attributes (2705 fields). It includes multiple MRN, account, patient name, room, and phone fields, making it a central PHI store. It likely feeds many dictionary‑driven behaviours and lookups.

#### 2.1.3 HXPLVL1–HXPLVL6 – Level Tables
- **HXPLVL1**: record_format HXFLVL1, key [HX1NUM], unique
- **HXPLVL2**: record_format HXFLVL2, key [HX2NUM], unique
- **HXPLVL3**: record_format HXFLVL3, key [HX3NUM], unique
- **HXPLVL4**: record_format HXFLVL4, key [HX4NUM], unique
- **HXPLVL5**: record_format HXFLVL5, key [HX5NUM], unique
- **HXPLVL6**: record_format HXFLVL6, key [HX6NUM], unique, total_fields 155

These files store hierarchical level definitions, each keyed by a level‑specific numeric identifier. HXPLVL6 is significantly wider, suggesting that higher levels carry more attributes.

#### 2.1.4 HXPTABLD – Table Definition File
- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI fields:** none

HXPTABLD holds table definitions keyed by data type/code and description code, forming the base for multiple logical views.

#### 2.1.5 HXPXMLD and HXPXMLR – XML Detail and Header
- **HXPXMLD:** record_format HXFXMLD, key [XMDUSR, XMDSEQ, XMDSQ2], unique, total_fields 4
- **HXPXMLR:** record_format HXFXMLR, key [XMRUSR, XMRSEQ, XMRID], unique, total_fields 4

These files model XML header and detail records. They are accessed via XFXGETID and HABADTE for XML generation and correlation.

#### 2.1.6 OAPIRNK – Rank File
- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI fields:** BRKMRN (MRN)

OAPIRNK records rank information keyed by level‑6, account, and sequence. It is PHI‑bearing because of its MRN field.

#### 2.1.7 OMPMAST – Master File
- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI fields:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN

OMPMAST is a master record file with extensive PHI content, including MRN, account, name, and SSN. It likely represents patient or member master records.

#### 2.1.8 OXPBNFIT – Benefit File
- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI fields:** XFBTEL (PhoneNumber)

OXPBNFIT defines benefit plans keyed by user benefit number and plan, with a PHI phone field.

#### 2.1.9 OXPNSTN – Station File
- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI fields:** none

OXPNSTN holds station or level‑6 station mappings, used by HABADTE when reading XFFNSTN.

#### 2.1.10 TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN – External / Staging Tables
- **TAPIRNK:** record_format "ATE TABLE", total_fields 43, non‑unique
- **TMPMAST:** record_format "ATE TABLE", total_fields 181, non‑unique
- **TXPBNFIT:** record_format "ATE TABLE", total_fields 12, non‑unique
- **TXPNSTN:** record_format "ATE TABLE", total_fields 19, non‑unique

These physical files are described as generic "ATE TABLE" in the compact schema, likely representing upstream or staging tables that feed the on‑line PFs and LFs.

### 2.2 Logical Files (LF)

#### 2.2.1 HAPIRNK (LF over TAPIRNK)
- **Based on PF:** TAPIRNK
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** none

HAPIRNK provides a keyed view over TAPIRNK, aligning rank data to level‑6 and account keys.

#### 2.2.2 HMLMAST5H (LF over TMPMAST)
- **Based on PF:** TMPMAST
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit:** none

HMLMAST5H provides a chronological view over master data, keyed by posting station and add date/time.

#### 2.2.3 HXLTABLD / HXLTABLP / HXLTABLS (LFs over HXPTABLD)
- **HXLTABLD:** key [XFDTCD, XFDMAP]
- **HXLTABLP:** key [XFDTCD, XFDLDS]
- **HXLTABLS:** key [XFDTCD, XFDSDS]

Each LF exposes a different access path over the same table data, targeting different table map, load, and subset codes.

#### 2.2.4 HXPBNFIT (LF over TXPBNFIT)
- **Based on PF:** TXPBNFIT
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/Omit:** none

Provides logical access to benefit staging data aligned to the live OXPBNFIT structure.

#### 2.2.5 HXPNSTN (LF over TXPNSTN)
- **Based on PF:** TXPNSTN
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/Omit:** none

Provides station/level‑6 view over TXPNSTN.

## Section 3 – Status and Type Reference Data

Approved rules from DATA_MAINTENANCE and PATIENT_MANAGEMENT domains mention status‑like indicators and flags, but no explicit code lists are named (e.g., no literal 'STATUS=' tables). Instead, status behaviour is embedded in rules such as:

- When a **flag indicator** equals void/voided, skip processing.
- When an **inpatient/outpatient flag** equals outpatient, skip processing.
- When a **file indicator** equals zero, skip processing.

These rules imply status values encoded in indicators and flags rather than central status tables. None of the approved_rules explicitly enumerate status codes, so no discrete status reference data can be derived from this context.

None identified.

## Section 4 – Stored Procedure Logic Mappings

Although the system does not expose DB2 stored procedures, the CALL graph behaves similarly. Each caller–callee relationship is documented below.

### 4.1 HABADTE Caller Map

HABADTE issues the following CALLs:

- **XFXMRNROL** – MRN roll logic, including subsequent calls to HXHAPPPRF and HXXAPPPRF.
- **XFXCNTR** (three calls) – field formatting and counter management; used to enforce rules such as exiting when fields are blank or leading characters are non‑blank.
- **XFXLDSC** – level description lookup via HXPLVL1–HXPLVL6.
- **XFXCYMD** – date validation, which then calls XFXLEAP.
- **XFXGETID** – ID retrieval based on XML record files.
- **XFXTABL** – generic table lookup across multiple table variants.

### 4.2 XFXCYMD and XFXLEAP

- **XFXCYMD → XFXLEAP** – date validation delegates leap‑year checks to XFXLEAP, encapsulating the leap‑year calculation.

### 4.3 XFXMRNROL and Profile Programs

- **XFXMRNROL → HXHAPPPRF** (missing) – legacy or alternative MRN profile logic.
- **XFXMRNROL → HXXAPPPRF** – SQLRPGLE‑based profile logic using modern SQL access.

### 4.4 HXXAPPPRF Copy Relationships

- **HXXAPPPRF** copies HXXCNTRL and HXXAPPPRFP, effectively inlining shared control and profile routines that function as internal subroutines or shared modules.

### 4.5 XFXGETID and HABADTE Copy Relationships

- **XFXGETID** copies HXXLDA and declares HXPXMLR/HXFXMLR.
- **HABADTE** copies HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, and CXXXMLC, indicating that much of its state and XML behaviour is defined via shared copy structures.

These mappings should be converted to explicit service and library calls in a modern architecture, preserving the caller–callee relationships.

## Section 5 – Service Class Method Reference

The hotspot analysis allows classification of programs into service classes:

- **BatchService (high score):**
  - **HABADTE** (score 38): Orchestrates patient‑management flows, XML generation, and transfer processing. High fan‑out and file operations qualify it as a batch‑style service.

- **WorkflowService (medium score):**
  - **XFXLDSC** (score 15): Manages multi‑level lookups; acts as a workflow component for resolving level hierarchies.
  - **XFXTABL** (score 11): Implements code/table lookups used across flows.
  - **XFXCNTR** (score 9): Implements formatting rules and counter logic reused across multiple call sites.
  - **XFXMRNROL** (score 7): Coordinates MRN roll and profile updates.
  - **HXXAPPPRF** (score 7): Executes SQL‑based profile updates.
  - **XFXGETID** (score 7): Retrieves identifiers for XML records.
  - **XFXCYMD** (score 5): Validates dates and delegates leap‑year decisions.

- **UtilityService (low score):**
  - **HXHAPPPRF** (score 3): Additional profile logic (implementation missing).
  - **XFXLEAP** (score 3): Leap‑year utility.
  - **HAPIRNK, HXPNSTN, HXLTABLD, HMLMAST5H, HXLTABLS, HXPBNFIT, HXLTABLP** (score 2 each): PF/LF processing utilities that provide narrow, file‑specific services.

These classifications can guide service decomposition in target architectures.

## Section 6 – External Interfaces

High‑impact and medium‑impact gaps indicate external or partially external interfaces:

- **CXXXMLC (COPYBOOK)** – defines XML control structures used by HABADTE. Its absence suggests that XML formatting rules, schema versioning, or tag names are driven by configuration external to the compiled codebase.
- **HXHAPPPRF (PROGRAM)** – called from XFXMRNROL. Its missing implementation indicates an externalised or separately deployed module, representing an integration point for MRN or profile management.
- ******HXPXML (FILE)** – a missing file referenced from HABADTE, likely representing an external XML staging or control table.
- **PRINTER (FILE)** – a missing printer/spool file referenced from HABADTE, representing an outbound report interface.

Inbound interfaces are thus primarily XML and printer‑based, with configuration encapsulated in copybooks and external files.

No outbound SPOOL interfaces identified.

## Section 7 – Performance and Security Notes

### 7.1 Cyclomatic Complexity and Performance

The complexity distribution shows that most programs have low complexity (cc between 1 and 9). The exception is **HABADTE**, with cyclomatic complexity **152** and band **HIGH**. This implies:

- Deeply nested decision logic within HABADTE.
- Multiple code paths controlling XML generation, MRN roll, and transfer handling.
- Increased risk of performance bottlenecks and regression when changes are made.

All other programs (XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, HXXAPPPRF, and the HXX* utilities) fall into the LOW band. They provide focused functionality and are good candidates for direct translation or wrapping.

### 7.2 PHI and Sensitive Data

PHI‑flagged fields appear in multiple PFs:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT:** XFBTEL (PhoneNumber)

All PHI exposure paths recorded in the lineage are **ISOLATED**, indicating that PHI is not widely propagated across derived datasets within the analysed scope. Nevertheless, any modernisation must enforce strong access controls around HABADTE, HXPDICT, OMPMAST, and related PFs.

### 7.3 Technical Debt

The tech_debt_summary identifies **4 findings** totalling **26.9 hours** of remediation effort, with severity breakdown:

- HIGH: 1
- MEDIUM: 3
- LOW: 0

Given HABADTE’s high complexity and central role, the HIGH‑severity item is likely associated with its structure or coupling to XML and MRN logic. Recommended remediation steps include:

- Refactoring HABADTE into smaller modules aligned with individual responsibilities.
- Introducing explicit service interfaces for MRN roll, XML generation, and table lookups.
- Reducing dependency on copybooks for control structures by externalising configuration.
