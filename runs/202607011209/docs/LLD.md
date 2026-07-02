# Low‑Level Design (LLD) – HABADTE Application Suite

## 1. Architecture Overview

### 1.1 Technology Stack and Component Types

The HABADTE application suite runs on an IBM i / AS400 platform and is implemented using classic IBM technologies:

- **DDS Physical Files (PF):** 15 members providing core persistent storage for patient, benefit, dictionary, and XML data.
- **DDS Logical Files (LF):** 7 members exposing alternative key sequences and projections over the physical files, particularly for ranking, master record views, and table mappings.
- **RPGLE Programs:** 13 members implementing core business logic, including patient management, date handling, table lookups, and MRN role determination.
- **SQLRPGLE Programs:** 2 members bridging RPG logic with embedded SQL, used for application profiles and XML processing.

This mix indicates a traditional database‑centric architecture with business logic primarily in RPG and data access modeled via DDS.

### 1.2 Call Graph Summary

The call and copy dependencies describe a clear layering:

- **Top‑level Controller:** `HABADTE` (RPGLE, domain PATIENT_MANAGEMENT) is the primary orchestration program.
- **Business Utility Layer (XFX* programs):**
  - `XFXMRNROL` – MRN role and patient identifier role logic; calls `HXHAPPPRF` (missing) and `HXXAPPPRF`.
  - `XFXCNTR` – counter/control logic, called three times by HABADTE.
  - `XFXCYMD` – calendar and date validation logic; calls `XFXLEAP` for leap‑year checks.
  - `XFXGETID` – identifier retrieval using XML record file `HXFXMLR`.
  - `XFXLDSC` – level description logic reading `HXFLVL1`–`HXFLVL6`.
  - `XFXTABL` – generic table lookup using the `XFFTABL*` family.
- **Application Profile and XML Layer (HXX* and CXX* programs):**
  - `HXXAPPPRF` / `HXXAPPPRFP` – application profile and associated processing routines.
  - `HXXCNTRL` – common control structures copied into profile programs.
  - `HXXLDA` – local data area and level‑related structures.
  - `HXXLEVEL` – level hierarchy metadata.
  - `HXXXML` – XML‑related control and layout definitions.
  - `CXXXMLP` / `CXXXMLC` – XML mapping programs/copybooks referenced from HABADTE.

`HABADTE` sits at the center of this graph, calling the XFX utilities, reading and writing DDS files, and copying profile/level/XML structures. The remaining programs exhibit limited fan‑out and are predominantly service routines.

### 1.3 Design Patterns

Several design patterns are evident from dependencies and component roles:

- **Hub‑and‑Spoke Controller Pattern:** HABADTE delegates specialized tasks (date validation, table lookup, identifier retrieval) to utility programs, reducing duplication and centralizing complex logic.
- **Table‑Driven Configuration:** The use of `HXPTABLD` with multiple logical files (`HXLTABLD`, `HXLTABLP`, `HXLTABLS`) and the `XFFTABL*` family suggests that many rules are represented as table entries rather than hard‑coded values.
- **Layered XML Handling:** XML data is handled through a combination of header/detail files (`HXFXMLH`, `HXFXMLD`, `HXFXMLR`) and copybooks/programs (`HXXXML`, `CXXXMLP`, `CXXXMLC`), isolating interface formats from core business logic.
- **Application Profile Abstraction:** MRN role logic invokes application profile modules (`HXXAPPPRF`), indicating that patient management behavior can be altered via configuration stored in profiles rather than direct code changes.

## 2. Database Schema

### 2.1 Physical Files (PF)

For each PF, the schema summary includes uniqueness, key fields, field volume, and PHI flags.

#### HAPTRFR

- **Record Format:** HAFTRFR
- **Unique Key:** Yes
- **Key Fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total Fields:** 28
- **PHI Fields:** AFACCT (AccountNumber), AFMRNO (MRN)

`HAPTRFR` holds transaction or transfer records tied to patient accounts and MRNs at a specific level. It is read by HABADTE and appears to drive patient transaction aggregation and XML output.

#### HXPDICT

- **Record Format:** HXFDICT
- **Unique Key:** No
- **Key Fields:** (none defined in compact schema)
- **Total Fields:** 2705
- **PHI Fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

`HXPDICT` is a very large dictionary/registry file containing MRNs, account numbers, telephone numbers, names, room identifiers, dates of birth, and other sensitive attributes. It supports multiple business domains and is a core PHI repository.

#### HXPLVL1–HXPLVL6

- **HXPLVL1:**
  - Record Format: HXFLVL1
  - Unique Key: Yes
  - Key Fields: HX1NUM
  - Total Fields: 36
  - PHI Fields: None
- **HXPLVL2:**
  - Record Format: HXFLVL2
  - Unique Key: Yes
  - Key Fields: HX2NUM
  - Total Fields: 39
  - PHI Fields: None
- **HXPLVL3:**
  - Record Format: HXFLVL3
  - Unique Key: Yes
  - Key Fields: HX3NUM
  - Total Fields: 39
  - PHI Fields: None
- **HXPLVL4:**
  - Record Format: HXFLVL4
  - Unique Key: Yes
  - Key Fields: HX4NUM
  - Total Fields: 39
  - PHI Fields: None
- **HXPLVL5:**
  - Record Format: HXFLVL5
  - Unique Key: Yes
  - Key Fields: HX5NUM
  - Total Fields: 42
  - PHI Fields: None
- **HXPLVL6:**
  - Record Format: HXFLVL6
  - Unique Key: Yes
  - Key Fields: HX6NUM
  - Total Fields: 155
  - PHI Fields: None

These level files model hierarchical structures (e.g., benefit levels, coverage tiers, facility groupings). `XFXLDSC` declares and reads them to derive textual descriptions and control logic.

#### HXPTABLD

- **Record Format:** XFFTABLD
- **Unique Key:** No
- **Key Fields:** XFDTCD, XFDECD
- **Total Fields:** 7
- **PHI Fields:** None

`HXPTABLD` is a generic table file keyed by a data code and an extended code, used for mapping, printing, and status determination. Its logical views `HXLTABLD`, `HXLTABLP`, and `HXLTABLS` expose different projections.

#### HXPXMLD and HXPXMLR

- **HXPXMLD:**
  - Record Format: HXFXMLD
  - Unique Key: Yes
  - Key Fields: XMDUSR, XMDSEQ, XMDSQ2
  - Total Fields: 4
  - PHI Fields: None
- **HXPXMLR:**
  - Record Format: HXFXMLR
  - Unique Key: Yes
  - Key Fields: XMRUSR, XMRSEQ, XMRID
  - Total Fields: 4
  - PHI Fields: None

These compact XML detail and record files hold user‑specific XML fragments, sequence numbers, and IDs. HABADTE writes XML detail via HXFXMLD, and XFXGETID reads HXFXMLR to obtain identifiers.

#### OAPIRNK

- **Record Format:** HBFIRNK
- **Unique Key:** Yes
- **Key Fields:** BRKLV6, BRKACC, BRKSEQ
- **Total Fields:** 33
- **PHI Fields:** BRKMRN (MRN)

`OAPIRNK` stores patient ranking information keyed by level, account, and a sequence counter. The presence of MRN indicates patient‑specific ranking metrics.

#### OMPMAST

- **Record Format:** HMFMAST
- **Unique Key:** Yes
- **Key Fields:** MMPLV6, MMACCT
- **Total Fields:** 149
- **PHI Fields:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN

`OMPMAST` is a detailed patient master file containing multiple identifiers, names, account numbers, and SSNs. It is a central PHI asset supporting patient management operations.

#### OXPBNFIT

- **Record Format:** XFFBNFIT
- **Unique Key:** Yes
- **Key Fields:** XFBUBN, XFBPLN
- **Total Fields:** 34
- **PHI Fields:** XFBTEL (PhoneNumber)

`OXPBNFIT` holds benefit plan information keyed by a benefit number and plan code. The presence of telephone numbers suggests contact details associated with benefits.

#### OXPNSTN

- **Record Format:** XFFNSTN
- **Unique Key:** Yes
- **Key Fields:** XFNLV6, XFNSST
- **Total Fields:** 23
- **PHI Fields:** None

`OXPNSTN` models station or namespace attributes keyed by level and status; it is read by HABADTE indirectly via XFFNSTN.

### 2.2 Logical Files (LF)

Logical files define alternate views for reporting and processing.

- **HAPIRNK**
  - PFILE: TAPIRNK (missing)
  - Record Format: HBFIRNK
  - Key Fields: BRKLV6, BRKACC, BRKSEQ
  - Select/Omit: none documented
  - Role: inbound interface providing ranked record views over TAPIRNK.

- **HMLMAST5H**
  - PFILE: TMPMAST (missing)
  - Record Format: HMFMAST
  - Key Fields: MMPNST, MMADDT, MMADTM
  - Role: temporal view over master records keyed by posting status and add date/time.

- **HXLTABLD / HXLTABLP / HXLTABLS**
  - Common PFILE: HXPTABLD
  - Record Format: XFFTABLD
  - Key Fields:
    - HXLTABLD: XFDTCD, XFDMAP
    - HXLTABLP: XFDTCD, XFDLDS
    - HXLTABLS: XFDTCD, XFDSDS
  - Role: provide distinct projections for mapping, long descriptions, and short descriptions over common table data.

- **HXPBNFIT**
  - PFILE: TXPBNFIT (missing)
  - Record Format: XFFBNFIT
  - Key Fields: XFBUBN, XFBPLN
  - Role: logical view over benefit plan records.

- **HXPNSTN**
  - PFILE: TXPNSTN (missing)
  - Record Format: XFFNSTN
  - Key Fields: XFNLV6, XFNSST
  - Role: logical view over station or namespace status.

## 3. Status and Type Reference Data

The approved business rules include several status‑driven conditions:

- Rules checking **indicator fields** (e.g., `*IN79` equals on/active in XFXTABL).
- Rules where **flag fields** represent void or inpatient/outpatient states (HABADTE rules: file indicator equals zero, flag indicator equals void/voided, inpatient/outpatient flag equals outpatient).
- Rules validating numeric ranges for dates (VYY, VMM, VDD) to guard against invalid years, months, and days.

No explicit coded status value tables (e.g., `STATUS = 'A'`) are present in the rule text, implying statuses are encoded as indicators and flags rather than enumerated constants.

Accordingly, **no centralized status/type code dictionary has been identified in the rule corpus**. Status and type semantics are inferred from indicator flags and table lookups rather than explicit reference data.

## 4. Stored Procedure Logic Mappings

Although the environment uses RPG programs rather than SQL stored procedures, the CALL edges effectively represent stored‑procedure‑like invocations.

### 4.1 Call Groups by Caller

- **HABADTE**
  - Callees: XFXMRNROL, XFXCNTR (3 calls), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL
  - Role: orchestrator that sequences MRN role resolution, counter setup, level description resolution, date validation, identifier retrieval, and table‑driven rules.

- **XFXCYMD**
  - Callees: XFXLEAP
  - Role: specialized date logic delegating leap‑year computation.

- **XFXMRNROL**
  - Callees: HXHAPPPRF (missing), HXXAPPPRF
  - Role: MRN role resolution that leverages application profile handlers.

- **HXXAPPPRF**
  - Copy targets: HXXCNTRL, HXXAPPPRFP
  - Role: profile processor that imports shared control and profile logic.

- **XFXGETID**
  - Copy targets: HXXLDA
  - File operations: READ HXFXMLR
  - Role: identifier retrieval with shared local data area structures.

These mappings demonstrate how HABADTE’s batch or workflow controller interacts with a set of quasi‑stored procedures implemented in RPG.

## 5. Service Class Method Reference

Using hotspot scores, components can be classified into service tiers:

- **BatchService (High Score)**
  - `HABADTE` (score 38, fan_out 13, file_ops 6) – central batch/workflow engine performing significant IO and orchestration.

- **WorkflowService (Medium Score)**
  - `XFXLDSC` (score 15) – level description service reading multiple level files.
  - `XFXTABL` (score 11) – table lookup service reading several table variants.
  - `XFXCNTR` (score 9) – count/control service invoked repeatedly.
  - `XFXMRNROL` (score 7) – MRN role service coordinating profile logic.
  - `XFXGETID` (score 7) – identifier retrieval service.
  - `HXXAPPPRF` (score 7) – application profile processing service.

- **UtilityService (Low Score)**
  - `XFXCYMD` (score 5) – date validation utility.
  - `XFXLEAP` (score 3) – leap‑year utility.
  - `HXHAPPPRF`, `HXLTABLD`, `HXLTABLP`, `HXLTABLS`, `HXPBNFIT`, `HXPNSTN`, `HMLMAST5H`, `HAPIRNK` (scores 2–3) – IO‑focused utilities providing specific logical views.

This classification is helpful when designing microservice boundaries or modularizing the monolith: HABADTE becomes a batch service orchestrating workflow and utility services.

## 6. External Interfaces

High‑impact gaps highlight likely inbound interfaces:

- **TAPIRNK / HAPIRNK:** Logical file HAPIRNK over missing PF TAPIRNK suggests an inbound feed of ranking data, potentially from external scoring engines.
- **TMPMAST / HMLMAST5H:** Logical view HMLMAST5H over TMPMAST implies an inbound or shared master record store integrated with other systems.
- **TXPBNFIT / HXPBNFIT:** Benefit plan data likely originates from external benefits administration systems.
- **TXPNSTN / HXPNSTN:** Station or namespace information may be sourced from facility or location management systems.
- **CXXXMLC and ****HXPXML:** XML copybooks and files referenced from HABADTE indicate inbound/outbound XML payloads exchanged with external applications, such as billing or EMR systems.

No explicit SPOOL or printer outbound interfaces are defined in the aggregated schema. Although a `PRINTER` file gap exists, without a corresponding DDS definition or usage beyond HABADTE’s reference, **no outbound SPOOL interfaces are concretely identified**.

## 7. Performance and Security Notes

### 7.1 Complexity Hotspots

Cyclomatic complexity scores highlight the main performance and maintainability concern:

- **HABADTE:** cc = 152 (HIGH band). This program is likely to contain nested branching, multiple loops, and extensive error‑handling paths. Any refactoring or optimization must start here, as it can impact patient processing throughput and defect rates.
- **XFXTABL:** cc = 9 (LOW band but highest among utilities), combining multiple table reads and indicator checks.
- All other programs, including XFXCYMD (cc=7), XFXLDSC (cc=5), and the application profile utilities, have LOW complexity.

### 7.2 PHI Exposure

PHI‑tagged fields appear in several files:

- **Account and MRN Fields:** AFACCT, AFMRNO (HAPTRFR); CCMRNO, IMGMRN, HXGMRN, IHMRNO, IHACCT, HVACCT, XMDMRN (HXPDICT); MMMRNO, MMMMRN, MMACCT (OMPMAST); BRKMRN (OAPIRNK).
- **Patient Identity Fields:** XCNAME, ENNAME (HXPDICT); MMNAME (OMPMAST).
- **Contact and Sensitive Identifiers:** XFBTEL (HXPDICT, OXPBNFIT); MMPSSN (OMPMAST); WBDATE (date of birth) in HXPDICT.

Most PHI exposure is **isolated to dictionary and master files** serving HABADTE’s processing. The aggregated lineage shows PHI fields flagged as "ISOLATED", meaning they do not propagate widely across programs, but their concentration in HXPDICT and OMPMAST makes those files high‑risk assets.

### 7.3 Tech Debt and Risk Summary

The tech debt summary reports:

- **Total findings:** 0
- **Total remediation hours:** 0.0
- **Severity distribution:** HIGH = 0, MEDIUM = 0, LOW = 0

Formal tech‑debt scanning did not flag specific code hygiene issues, but architectural analysis indicates implicit risks:

- **Single high‑complexity controller (HABADTE)** concentrating logic and IO.
- **Missing DDS for key PFs and copybooks**, which hampers impact analysis and test planning.
- **Tight coupling to PHI‑heavy files**, demanding careful security and audit controls.

Overall, the design is typical for mature AS400 applications, with clear data structures and modular utility programs. Modernization should focus on decomposing HABADTE, externalizing table‑driven rules, and encapsulating PHI data behind secure service layers.