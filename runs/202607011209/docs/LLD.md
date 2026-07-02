# Low-Level Design (LLD)

## 1. Architecture Overview

### 1.1 Technology Stack and Component Types

The AS400 application is built on a traditional IBM i stack with the following component types:

- **DDS Physical Files (PF)**: 15 members representing core data structures such as transfers, member master records, benefit tables, and multi-level configuration (e.g., HAPTRFR, HXPDICT, HXPLVL1–HXPLVL6, HXPTABLD, OMPMAST, OXPBNFIT, OXPNSTN).
- **DDS Logical Files (LF)**: 7 members that project alternative key sequences and views over PFs (e.g., HAPIRNK, HMLMAST5H, HXLTABLD/HXLTABLP/HXLTABLS, HXPBNFIT, HXPNSTN).
- **RPGLE Programs**: 13 members implementing procedural business logic, utility routines, and batch orchestration (e.g., HABADTE, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML).
- **SQLRPGLE Programs**: 2 members (HXXAPPPRF, CXXXMLP) providing modernized data access and XML-related logic using embedded SQL.

This mix reflects a system where the database layer is defined entirely via DDS, with the logic layer split between classic RPG and SQLRPGLE for newer modules.

### 1.2 Call Graph and Interaction Patterns

The dependency edges show a clear orchestration pattern:

- **Utility Services**
  - `XFXCYMD -> XFXLEAP` (CALL): calendar validation and leap-year handling.
  - `XFXTABL -> XFFTABLD/XFFTABL2/XFFTABL3/XFFTABL4` (READ): generic table lookup logic.

- **MRN Role and Application Profile Services**
  - `XFXMRNROL -> HXHAPPPRF` (CALL): MRN application profiling (missing program).
  - `XFXMRNROL -> HXXAPPPRF` (CALL): SQL-based profile computation using modern access.

- **Identifier and Descriptor Services**
  - `XFXGETID -> HXFXMLR` (READ): retrieving XML routing or identifier information.
  - `XFXLDSC -> HXFLVL1–HXFLVL6` (READ): reading multi-level configuration for descriptors.

- **Central Batch Integration Service**
  - `HABADTE -> XFXMRNROL, XFXCNTR (multiple calls), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL` (CALL).
  - `HABADTE -> HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC` (COPY).
  - `HABADTE -> HAPTRFR, XFFNSTN, HXFXMLH, HXFXMLD` (READ/UPDATE/WRITE).

This graph shows HABADTE as the central orchestrator calling specialized utilities to perform MRN resolution, date validation, ID generation, descriptor lookup, and table-based code translation, then emitting XML records via HXFXMLH/HXFXMLD.

### 1.3 Design Patterns

From the dependencies we can infer the following design patterns:

- **Batch-Oriented Service Facade**: HABADTE acts as a service facade for several underlying utilities (XFX* and HXX*). It coordinates file reads and writes and delegates complex calculations to specialized programs.
- **Lookup Table Pattern**: XFXTABL encapsulates read access to multiple dictionary files (XFFTABLD and variants), providing centralized code-to-description resolution.
- **Multi-Level Configuration Pattern**: HXPLVL1–HXPLVL6 form a set of configuration tables for different levels, with XFXLDSC reading them to construct composite descriptors.
- **MRN Role Resolution Pattern**: XFXMRNROL, HXHAPPPRF, and HXXAPPPRF form a chain of responsibility for determining MRN-related roles or profiles.
- **XML Integration Pattern**: CXXXMLP, CXXXMLC, HXFXMLR, HXFXMLH, HXFXMLD, and HXPXMLD/HXPXMLR together support XML payload construction, routing, and persistence.

These patterns should be preserved conceptually when migrating to a service-oriented or microservice architecture, even though the implementation technologies will change.

## 2. Database Schema

### 2.1 Physical Files (PF)

Below, each PF is documented with its record format, keys, uniqueness, and PHI exposure based on the aggregated schema.

#### HAPTRFR
- Record format: **HAFTRFR**
- Unique key: **Yes**
- Key fields: `AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE`
- Total fields: 28
- PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

This file stores transfer records at level 6, keyed by account, transfer date/time, and type. It is PHI-sensitive due to account and MRN fields.

#### HXPDICT
- Record format: **HXFDICT**
- Unique key: **No**
- Key fields: none recorded
- Total fields: 2705
- PHI fields: `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`.

HXPDICT is a large dictionary/master file combining multiple domains. It includes MRNs, account numbers, phone numbers, room numbers, patient names, and dates of birth, making it a central PHI repository.

#### HXPLVL1–HXPLVL6
- HXPLVL1: record format **HXFLVL1**, unique key, key `HX1NUM`, total fields 36, no PHI.
- HXPLVL2: record format **HXFLVL2**, unique key, key `HX2NUM`, total fields 39, no PHI.
- HXPLVL3: record format **HXFLVL3**, unique key, key `HX3NUM`, total fields 39, no PHI.
- HXPLVL4: record format **HXFLVL4**, unique key, key `HX4NUM`, total fields 39, no PHI.
- HXPLVL5: record format **HXFLVL5**, unique key, key `HX5NUM`, total fields 42, no PHI.
- HXPLVL6: record format **HXFLVL6**, unique key, key `HX6NUM`, total fields 155, no PHI.

These files represent hierarchical configuration or benefit levels. They are non-PHI, but critical for calculation logic and descriptor generation.

#### HXPTABLD
- Record format: **XFFTABLD**
- Unique key: **No**
- Key fields: `XFDTCD, XFDECD`
- Total fields: 7
- PHI fields: none

HXPTABLD is a generic dictionary table with data code and detail code keys, used heavily by XFXTABL and the HXLTAB* logical files.

#### HXPXMLD / HXPXMLR
- HXPXMLD: record format **HXFXMLD**, unique key, keys `XMDUSR, XMDSEQ, XMDSQ2`, total fields 4, no PHI.
- HXPXMLR: record format **HXFXMLR**, unique key, keys `XMRUSR, XMRSEQ, XMRID`, total fields 4, no PHI.

These files store XML detail and routing records keyed by user and sequence identifiers, forming part of the XML integration layer.

#### OAPIRNK
- Record format: **HBFIRNK**
- Unique key: **Yes**
- Key fields: `BRKLV6, BRKACC, BRKSEQ`
- Total fields: 33
- PHI fields: `BRKMRN` (MRN)

OAPIRNK holds rank or break records tied to MRNs and accounts, likely reflecting patient or member ranking per level.

#### OMPMAST
- Record format: **HMFMAST**
- Unique key: **Yes**
- Key fields: `MMPLV6, MMACCT`
- Total fields: 149
- PHI fields: `MMMRNO` (MRN), `MMACCT` (AccountNumber), `MMNAME` (PatientName), `MMPSSN` (SSN), `MMMMRN` (MRN).

OMPMAST is a master file for members/patients, carrying multiple identifiers including MRN, account, name, and SSN. It is a highly sensitive PF.

#### OXPBNFIT
- Record format: **XFFBNFIT**
- Unique key: **Yes**
- Key fields: `XFBUBN, XFBPLN`
- Total fields: 34
- PHI fields: `XFBTEL` (PhoneNumber).

OXPBNFIT stores benefit information keyed by benefit number and plan, with contact phone numbers.

#### OXPNSTN
- Record format: **XFFNSTN**
- Unique key: **Yes**
- Key fields: `XFNLV6, XFNSST`
- Total fields: 23
- PHI fields: none.

OXPNSTN captures institution or plan location data at level 6.

### 2.2 Logical Files (LF)

Each LF exposes a view over a base PF with alternative keys.

#### HAPIRNK
- PFILE: **TAPIRNK**
- Record format: **HBFIRNK**
- Key fields: `BRKLV6, BRKACC, BRKSEQ`
- Select/omit: none.

Provides a keyed view over TAPIRNK aligning with OAPIRNK’s structure.

#### HMLMAST5H
- PFILE: **TMPMAST**
- Record format: **HMFMAST**
- Key fields: `MMPNST, MMADDT, MMADTM`
- Select/omit: none.

Logical view over TMPMAST focused on admission date/time and posting station.

#### HXLTABLD / HXLTABLP / HXLTABLS
- PFILE: **HXPTABLD**
- Record format: **XFFTABLD**
- HXLTABLD keys: `XFDTCD, XFDMAP`
- HXLTABLP keys: `XFDTCD, XFDLDS`
- HXLTABLS keys: `XFDTCD, XFDSDS`
- Select/omit: none.

These LFs provide different keying over the same code table for map, load, and status sequences.

#### HXPBNFIT
- PFILE: **TXPBNFIT**
- Record format: **XFFBNFIT**
- Key fields: `XFBUBN, XFBPLN`
- Select/omit: none.

Logical view over benefit PF focused on benefit number and plan.

#### HXPNSTN
- PFILE: **TXPNSTN**
- Record format: **XFFNSTN**
- Key fields: `XFNLV6, XFNSST`
- Select/omit: none.

Logical view over institution/plan location PF keyed by level and station.

## 3. Status and Type Reference Data

Approved business rules show simple status-like branching criteria:

- BR-001: "When X equals zero, branch to 'EXIT'" (XFXCNTR).
- BR-002: "When X equals 40, branch to 'EXIT'" (XFXCNTR).
- BR-003: "When VYY is less than 1800, branch to 'EXIT'" (XFXCYMD).
- BR-004: "When VYY is greater than 2100, branch to 'EXIT'" (XFXCYMD).
- BR-005: "When VMM is less than 01, branch to 'EXIT'" (XFXCYMD).

These rules act as **validation boundaries** for counters and calendar values rather than conventional status codes. No explicit status or type code tables (e.g., STAT_CD, TYPE_CD) are visible in the approved_rules text, so dedicated status/type reference data is **None identified.**

In modernization, these rules should be captured as validation constraints (e.g., allowed ranges for year and month) rather than free-form branching logic.

## 4. Stored Procedure Logic Mappings

The CALL edges can be interpreted as stored procedure invocations grouped by caller.

### 4.1 Mappings by Caller

- **XFXCYMD (calendar validation)**
  - Calls: `XFXLEAP` – leap-year determination.

- **XFXMRNROL (MRN role resolver)**
  - Calls: `HXHAPPPRF` – profile logic (legacy RPG, missing in this export).
  - Calls: `HXXAPPPRF` – SQLRPGLE profile logic.

- **HABADTE (batch driver)**
  - Calls: `XFXMRNROL` – MRN role resolution.
  - Calls: `XFXCNTR` (three separate edges) – counter/iteration control across multiple phases.
  - Calls: `XFXLDSC` – level descriptor assembly using HXPLVL*.
  - Calls: `XFXCYMD` – calendar validation.
  - Calls: `XFXGETID` – identifier retrieval from HXFXMLR.
  - Calls: `XFXTABL` – table-based code translation.

- **HXXAPPPRF (SQLRPGLE app profile)**
  - Copies in: `HXXCNTRL`, `HXXAPPPRFP` – control and profile copy members used as shared structures.

These mappings reveal a layered structure where HABADTE orchestrates higher-level flows, delegating discrete responsibilities to specialized procedures. For modernization, each CALL target should be mapped to an individual service operation with a clear contract.

## 5. Service Class Method Reference

Using the hotspot scores, we can classify each program into conceptual service classes:

- **BatchService (High score)**
  - **HABADTE (score 38)** – primary BatchService handling integration, file I/O, and XML emission.

- **WorkflowService (Medium scores)**
  - **XFXLDSC (score 15)** – workflow for assembling level descriptors from multi-level tables.
  - **XFXTABL (score 11)** – workflow for generic table lookups.
  - **XFXCNTR (score 9)** – workflow controller/counter service.
  - **XFXMRNROL (score 7)** – workflow for MRN role resolution.
  - **XFXGETID (score 7)** – workflow for ID retrieval.
  - **HXXAPPPRF (score 7)** – workflow for application profiling.
  - **XFXCYMD (score 5)** – workflow for calendar validation.

- **UtilityService (Lower scores)**
  - **HXHAPPPRF, XFXLEAP** (score 3) – utility methods for profile logic and leap-year calculation.
  - Various LF-based utilities (HXLTABLS, HMLMAST5H, HXPNSTN, HAPIRNK, HXLTABLD, HXPBNFIT, HXLTABLP) with scores around 2, representing focused data access utilities.

This classification provides a starting point for grouping functions into microservices or modules in a modern architecture.

## 6. External Interfaces

High-impact gaps are interpreted as inbound external interfaces:

- **TAPIRNK (FILE)** – inbound rank/break data. External system likely populates TAPIRNK, which is then consumed via HAPIRNK/OAPIRNK.
- **TMPMAST (FILE)** – inbound member master data, populated by upstream registration or enrollment systems.
- **TXPBNFIT (FILE)** – inbound benefit configuration, likely received from benefits administration.
- **TXPNSTN (FILE)** – inbound institution/plan location data.
- **CXXXMLC (COPYBOOK)** – inbound XML layout definitions shared with external consumers.

These inbound interfaces must be explicitly modeled in the modern solution as data feeds or APIs.

There are no explicit outbound SPOOL or printer file definitions beyond the missing **PRINTER** file reference in HABADTE; since no concrete SPOOL file is present in the schema, **No outbound SPOOL interfaces identified.**

## 7. Performance and Security Notes

### 7.1 Cyclomatic Complexity and Performance Hotspots

Complexity metrics highlight:

- **HABADTE** – cc=152, band=HIGH. This is a performance and maintainability hotspot. Its complexity stems from extensive branching and orchestration logic, making it sensitive to changes and difficult to test.
- All other measured programs (HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, CXXXMLP, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML) have cc values between 1 and 9 and are classified as LOW complexity.

This suggests that the system’s performance bottlenecks will concentrate in HABADTE’s batch processing, especially around file I/O and XML creation, while utility modules remain simple and fast.

### 7.2 PHI and Sensitive Data Handling

PHI-tagged fields span multiple files:

- **HAPTRFR**: AFACCT (AccountNumber), AFMRNO (MRN).
- **HXPDICT**: numerous MRN, account, phone, room, name, and DOB fields.
- **OAPIRNK**: BRKMRN (MRN).
- **OMPMAST**: MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN).
- **OXPBNFIT**: XFBTEL (PhoneNumber).

These fields indicate that the system processes highly sensitive clinical/financial data. In the legacy implementation, sensitivity is encoded as metadata rather than explicit access checks in the scanned rules; however, modernization must enforce:

- Field-level encryption or tokenization for MRN, account, SSN, and phone fields.
- Role-based access controls around OMPMAST, HXPDICT, and HAPTRFR operations.
- Audit logging for any service that reads or writes PHI fields.

### 7.3 Technical Debt Summary

The aggregated tech_debt_summary shows:

- total_findings: 0
- total_remediation_hours: 0.0
- by_severity: HIGH=0, MEDIUM=0, LOW=0

This indicates that the automated tech-debt analysis did not flag any explicit issues (e.g., hard-coded credentials, obsolete APIs). Nonetheless, structural complexity in HABADTE and missing physical files/copybooks represent **implicit technical debt** that must be addressed during modernization, even if not captured as discrete findings.

From a low-level design perspective, the modernization effort should:

1. Decompose HABADTE into manageable service operations.
2. Map PF/LF structures to relational tables and views with explicit constraints and PHI annotations.
3. Promote XFX* and HXX* utilities into well-defined services with API contracts.
4. Replace XML copybooks and file-based messaging with standardized integration mechanisms (e.g., REST/JSON or message queues) while preserving the XML semantics where required.