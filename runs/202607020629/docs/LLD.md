# Low-Level Design (LLD) – HABADTE AS400 Application

## 1. Architecture Overview

### 1.1 Technology Stack Profile

The HABADTE application is built entirely on traditional IBM i / AS400 primitives:

- **RPGLE programs (13)** – primary business logic, including the main driver `HABADTE` and a set of reusable utilities (`XFXCNTR`, `XFXCYMD`, `XFXLDSC`, `XFXTABL`, `XFXGETID`, `XFXMRNROL`, `HXX*`).
- **SQLRPGLE programs (2)** – hybrid RPG with embedded SQL, notably `HXXAPPPRF` and `CXXXMLP`, used for application profile and XML handling.
- **DDS Physical Files (15)** – core data storage definitions for transactions (`HAPTRFR`), patient master data (`OMPMAST`), dictionaries (`HXPDICT`), benefits (`OXPBNFIT`), and level/tables (`HXPLVL*`, `HXPTABLD`).
- **DDS Logical Files (7)** – indexed or filtered views over PFs: `HAPIRNK`, `HMLMAST5H`, `HXLTABLD`, `HXLTABLP`, `HXLTABLS`, `HXPBNFIT`, and `HXPNSTN`.

This stack supports a batch‑oriented patient management flow where `HABADTE` orchestrates calls to utility programs and performs direct file IO.

### 1.2 Call Graph Summary

From `dep_edges`, the core call relationships are:

- **Entry Orchestrator**: `HABADTE` (RPGLE, domain PATIENT_MANAGEMENT)
  - Calls `XFXMRNROL` (MRN roll‑up logic)
  - Calls `XFXCNTR` repeatedly for counter/threshold decisions
  - Calls `XFXLDSC` for level description and mapping lookups
  - Calls `XFXCYMD` for calendar date validation and normalisation
  - Calls `XFXGETID` for identifier and XML routing resolution
  - Calls `XFXTABL` for generic table lookups
  - Uses copybooks `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, and missing `CXXXMLC` via COPY

- **Utility Chains**:
  - `XFXCYMD` → `XFXLEAP` (date and leap year validation)
  - `XFXMRNROL` → `HXHAPPPRF` and `HXXAPPPRF` (application profile and MRN roll logic)
  - `HXXAPPPRF` → `HXXCNTRL`, `HXXAPPPRFP` via COPY (control structures and profile templates)
  - `XFXGETID` → `HXXLDA` via COPY (load common data area structures for IDs)

Design patterns observed:

- **Controller/Utility pattern**: `HABADTE` acts as a controller delegating work to small utilities (`XFX*`) that encapsulate single responsibilities (date, level, table, ID logic).
- **Table‑driven configuration**: programs such as `XFXLDSC` and `XFXTABL` use DDS tables to drive behaviour, reducing procedural branching but increasing reliance on DDS metadata.
- **Copybook‑based structuring**: `HXX*` and `CXXXMLP` copy members provide shared data layouts for XML and control data.

## 2. Database Schema

The `data_dict_schema` provides a compact model of all PF and LF members.

### 2.1 Physical Files (PF)

Each PF subsection includes record format, key structure, uniqueness, and PHI exposure.

#### HAPTRFR (Transaction File)

- **Record format**: `HAFTRFR`
- **Unique key**: yes
- **Key fields**: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- **Total fields**: 28
- **PHI fields**: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

Purpose: stores patient transfer or transaction records keyed by level, account, date/time and type. It is read by `HABADTE` and participates in PHI flows for account and MRN identifiers.

#### HXPDICT (Dictionary / Master Reference)

- **Record format**: `HXFDICT`
- **Unique key**: no
- **Key fields**: none defined in the compact schema
- **Total fields**: 2705
- **PHI fields**: `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`

Purpose: extremely large dictionary and reference file containing patient names, MRNs, phone numbers, room/account information, and date of birth. It is the central PHI repository and likely drives many lookups through `XFXTABL` and related routines.

#### HXPLVL1–HXPLVL6 (Level Configuration Files)

- **Record formats**: `HXFLVL1`–`HXFLVL6`
- **Unique key**: yes
- **Key fields**:
  - HXPLVL1: `HX1NUM`
  - HXPLVL2: `HX2NUM`
  - HXPLVL3: `HX3NUM`
  - HXPLVL4: `HX4NUM`
  - HXPLVL5: `HX5NUM`
  - HXPLVL6: `HX6NUM`
- **Total fields**: 36–155 (increasing with level depth)
- **PHI fields**: none flagged

Purpose: store hierarchical level definitions (e.g., facility, unit, ward) used by `XFXLDSC` to resolve descriptive mappings. `XFXLDSC` both declares and reads these files.

#### HXPTABLD (Table Dictionary)

- **Record format**: `XFFTABLD`
- **Unique key**: no
- **Key fields**: `XFDTCD`, `XFDECD`
- **Total fields**: 7
- **PHI fields**: none

Purpose: central code/description table dictionary consumed by various table‑driven logic, including logical views and the `XFXTABL` program.

#### HXPXMLD / HXPXMLR (XML Detail and Route)

- **HXPXMLD**:
  - Record format: `HXFXMLD`
  - Unique key: yes
  - Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
  - Total fields: 4
  - PHI fields: none flagged

- **HXPXMLR**:
  - Record format: `HXFXMLR`
  - Unique key: yes
  - Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`
  - Total fields: 4
  - PHI fields: none flagged

Purpose: hold XML message routing and detail segments for downstream systems. `XFXGETID` reads `HXFXMLR`, while `HABADTE` writes `HXFXMLD` and updates `HXFXMLH` (header), forming a message pipeline.

#### OAPIRNK (Ranking File)

- Record format: `HBFIRNK`
- Unique key: yes
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Total fields: 33
- PHI fields: `BRKMRN` (MRN)

Purpose: persists ranking or score information for patients by level and account. PHI exposure is limited to MRN, but linkage back to `HXPDICT` and `OMPMAST` amplifies sensitivity.

#### OMPMAST (Patient Master)

- Record format: `HMFMAST`
- Unique key: yes
- Key fields: `MMPLV6`, `MMACCT`
- Total fields: 149
- PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`

Purpose: primary patient master file storing MRN, account, names and SSN. It is the cornerstone of patient identity and must be handled with stringent privacy controls.

#### OXPBNFIT (Benefit File)

- Record format: `XFFBNFIT`
- Unique key: yes
- Key fields: `XFBUBN`, `XFBPLN`
- Total fields: 34
- PHI fields: `XFBTEL` (PhoneNumber)

Purpose: stores benefit plan information and contact numbers for benefits. It is linked to the logical file `HXPBNFIT` and may feed patient benefit calculations in `HABADTE`.

#### OXPNSTN (Notification / Statement File)

- Record format: `XFFNSTN`
- Unique key: yes
- Key fields: `XFNLV6`, `XFNSST`
- Total fields: 23
- PHI fields: none flagged

Purpose: likely stores notification or statement status by level and status code, used by `HABADTE` when reading `XFFNSTN`.

### 2.2 Logical Files (LF)

#### HAPIRNK

- **PFILE**: `TAPIRNK` (missing)
- **Record format**: `HBFIRNK`
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`
- **Select/Omit**: none

This LF provides ranked views over the ranking file. Because `TAPIRNK` is missing, the LF is structurally defined but cannot be instantiated without recovering the PF.

#### HMLMAST5H

- **PFILE**: `TMPMAST` (missing)
- **Record format**: `HMFMAST`
- **Key fields**: `MMPNST`, `MMADDT`, `MMADTM`
- **Select/Omit**: none

Represents a time‑ordered view of the patient master by posting date/time. It supports historical reporting; recovery of `TMPMAST` is required to make this view operational.

#### HXLTABLD / HXLTABLP / HXLTABLS

All three logical files share:

- **PFILE**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**:
  - `HXLTABLD`: `XFDTCD`, `XFDMAP`
  - `HXLTABLP`: `XFDTCD`, `XFDLDS`
  - `HXLTABLS`: `XFDTCD`, `XFDSDS`
- **Select/Omit**: none

These LFs provide alternative indexes into the same table dictionary based on map, load, and sort descriptors, supporting flexible table resolution in `XFXTABL` and `XFXLDSC`.

#### HXPBNFIT

- **PFILE**: `TXPBNFIT` (missing)
- **Record format**: `XFFBNFIT`
- **Key fields**: `XFBUBN`, `XFBPLN`
- **Select/Omit**: none

Exposes benefit information with keyed access. Because `TXPBNFIT` is missing, this view must be reconstructed from external exports or upstream benefit systems during modernization.

#### HXPNSTN

- **PFILE**: `TXPNSTN` (missing)
- **Record format**: `XFFNSTN`
- **Key fields**: `XFNLV6`, `XFNSST`
- **Select/Omit**: none

Provides a logical view over notification or statement status. Its PFILE gap affects understanding of outbound communication flows.

## 3. Status and Type Reference Data

The `approved_rules` set does not explicitly enumerate textual status codes beyond conditions on indicator fields and numeric values. Business rules primarily focus on branching logic:

- In `XFXCNTR`: branches when counter `X` equals 0 or 40.
- In `XFXCYMD`: branches when year `VYY` or month `VMM` fall outside valid ranges.
- In `XFXTABL`: branches based on indicator `*IN79`.
- In `HABADTE`: branches on void flag, inpatient/outpatient flag, and file indicator.

No centralized status code or type reference list is encoded in the extracted rule text. Any status/type mapping (e.g., inpatient vs outpatient, void vs active) is tied to specific indicators rather than a shared reference table.

**Status and Type Reference Data**: None identified in this codebase as standalone lookup tables. All status semantics are embedded in program logic and DDS field names.

## 4. Stored Procedure Logic Mappings

Although AS400 RPG programs are not stored procedures in the SQL sense, we can model their call relationships as procedure mappings.

### 4.1 Call Grouping by Caller

- **HABADTE**
  - Callees: `XFXMRNROL`, `XFXCNTR` (three distinct paths), `XFXLDSC`, `XFXCYMD`, `XFXGETID`, `XFXTABL`
  - Copy dependencies: `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, `CXXXMLC`
  - File operations: `HAPTRFR` (READ), `XFFNSTN` (READ), `HXFXMLH` (READ/UPDATE/WRITE), `HXFXMLD` (WRITE)

- **XFXCYMD**
  - Callees: `XFXLEAP`
  - Purpose: validate calendar dates; enforce year/month/day limits and detect leap years.

- **XFXMRNROL**
  - Callees: `HXHAPPPRF`, `HXXAPPPRF`
  - Purpose: roll MRN data into application‑profile structures; may perform authorization or configuration validation.

- **HXXAPPPRF**
  - Callees via COPY: `HXXCNTRL`, `HXXAPPPRFP`
  - Purpose: define application control structures and profile prototypes used across MRN roll and control logic.

- **XFXGETID**
  - Callees via COPY: `HXXLDA`
  - File IO: declares and reads `HXFXMLR` to obtain identifiers and routing information.

This mapping supports creation of sequence diagrams and service invocation charts in a target microservice architecture, where each caller becomes a service client and each callee a dependency.

## 5. Service Class Method Reference

Using `dep_hotspots`, programs are ranked by structural score and categorised into service classes.

- **BatchService (High score)**:
  - `HABADTE` (score 38): core batch driver processing patient transactions and building XML messages. It should be refactored into a workflow service orchestrating specialised methods for MRN roll‑up, benefit evaluation, and outbound messaging.

- **WorkflowService (Medium score)**:
  - `XFXLDSC` (score 15): orchestrates level description reads across multiple files. Acts as a workflow around level configuration.
  - `XFXTABL` (score 11): orchestrates table lookups across multiple table variants. Represents a generic workflow for code/description resolution.
  - `XFXCNTR` (score 9): though small, high fan‑in and use as a decision point justify classifying it as a workflow component for counter‑based branching.
  - `XFXGETID` (score 7): handles ID resolution and XML routing; sits between data access and outbound messaging.
  - `HXXAPPPRF` and `XFXMRNROL` (scores 7): embody workflows around application profile definitions and MRN roll‑up.

- **UtilityService (Low score)**:
  - `XFXCYMD` (score 5) and `XFXLEAP` (score 3): fine‑grained utilities for date validation; in a modern design they become library functions or shared microservices.
  - Copy‑based members (`HXXCNTRL`, `HXXAPPPRFP`, `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`) serve as utility schema definitions rather than independent services.

This classification guides service decomposition: high‑score programs become candidate top‑level services, medium‑score programs form workflow/stage services, and low‑score utilities supply supporting functions.

## 6. External Interfaces

High‑impact gaps indicate inbound or outbound interfaces not fully captured in the extracted code:

- `TAPIRNK` and `TMPMAST` PFs – inbound interfaces for ranking and master posting data, likely populated by upstream batch jobs or external systems.
- `TXPBNFIT` and `TXPNSTN` PFs – inbound benefit and notification feeds from other modules or external benefit providers.
- `****HXPXML` – probable XML file or device representing an outbound XML interface.
- `PRINTER` – device/file representing outbound print spooling for statements or notifications.

These constitute external interfaces that must be documented with SME input and possibly traced to upstream/downstream systems outside this export.

No outbound SPOOL interfaces identified beyond the `PRINTER` file reference; its implementation details are missing with the current artifacts.

## 7. Performance and Security Notes

### 7.1 Cyclomatic Complexity and Performance Hotspots

From `complexity_per_program`:

- `HABADTE`: **cc=152 (HIGH)** – significantly complex with many branches and decisions. This is the principal performance and maintainability hotspot. Any change in business rules here is costly and risky.
- All other programs (`HXXAPPPRF`, `XFXCNTR`, `XFXCYMD`, `XFXGETID`, `XFXLDSC`, `XFXLEAP`, `XFXMRNROL`, `XFXTABL`, `CXXXMLP`, `HXXAPPPRFP`, `HXXCNTRL`, `HXXLDA`, `HXXLEVEL`, `HXXXML`) have **cc=1–9 (LOW)**, indicating simple, well‑bounded logic.

Performance implications:

- Optimisation efforts should focus on `HABADTE`, especially costly loops around file IO to `HAPTRFR`, `XFFNSTN`, and XML header/detail files.
- Utility programs are unlikely to be performance bottlenecks but must be monitored due to their high fan‑in.

### 7.2 PHI Fields and Security Controls

From `phi_flagged_fields`, key PHI exposures include:

- **Patient and Identity Data**:
  - MRN fields: `AFMRNO`, `CCMRNO`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `XMDMRN`, `MMMRNO`, `MMMMRN`.
  - Names: `XCNAME`, `ENNAME`, `MMNAME`.
  - SSN: `MMPSSN`.

- **Account and Contact Data**:
  - Accounts: `AFACCT`, `HVACCT`, `IHACCT`, `MMACCT`.
  - Phone numbers: `XFBTEL` (in both `HXPDICT` and `OXPBNFIT`).
  - Room numbers: `HXRMNO`, `XFRMNO`.
  - Date of birth: `WBDATE`.

These fields appear in core PFs (`HAPTRFR`, `HXPDICT`, `OAPIRNK`, `OMPMAST`, `OXPBNFIT`). Security requirements for modernization include:

- Field‑level encryption or tokenisation for MRN, SSN, account numbers and phone numbers.
- Strict access control for any services exposing `HXPDICT` and `OMPMAST` data.
- Audit logging around operations that read or write PHI‑bearing files, particularly in `HABADTE` and `XFXGETID`.

### 7.3 Tech Debt Summary

`tech_debt_summary` reports:

- **Total findings**: 0
- **Total remediation hours**: 0.0
- **By severity (HIGH/MEDIUM/LOW)**: all zeros

Despite the formal tech‑debt report showing no findings, practical concerns include:

- High complexity in `HABADTE` and reliance on missing components (gaps) represent latent technical debt.
- Tight coupling between controller logic and file IO calls suggests refactoring into service layers.

In planning remediation, focus should be on decomposing `HABADTE`, recovering or replacing missing PFs and copybooks, and hardening PHI‑related data access paths rather than purely relying on automated tech‑debt scoring.
