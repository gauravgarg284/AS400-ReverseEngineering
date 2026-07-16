# Low-Level Design (LLD) – HABADTE AS400 Application

## Section 1 – Architecture Overview

### 1.1 Technology Stack Composition

The HABADTE application is implemented entirely on IBM i / AS400 using traditional RPG and DDS constructs, with limited use of SQLRPGLE. Based on the aggregated source statistics:

- **DDS_LF (Logical Files):** 7
- **DDS_PF (Physical Files):** 19
- **SQLRPGLE Programs:** 2
- **RPGLE Programs:** 13

This distribution reflects a file‑centric architecture where DDS physical files model the core data structures, logical files provide alternate access paths and projections, and RPGLE/SQLRPGLE programs implement business logic. The main HABADTE program and its satellite utilities form an application layer on top of the file schema.

### 1.2 Call Graph and Design Patterns

The call and copy edges expose several recurring design patterns:

- **Hub‑and‑Spoke Orchestration:** The RPGLE program **HABADTE** operates as the central orchestrator. It calls XFXMRNROL, XFXCNTR (three times), XFXLDSC, XFXCYMD, XFXGETID, and XFXTABL. HABADTE also copies HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, and CXXXMLC (missing), indicating heavy dependence on shared layouts and XML wrappers.
- **Utility Service Programs:** Smaller RPGLE utilities are invoked by HABADTE and other programs:
  - **XFXCNTR** – text centering logic reused across multiple print or display segments.
  - **XFXCYMD** – date validation and normalisation, delegating leap‑year calculations to XFXLEAP.
  - **XFXLDSC** – organisational level description lookup, reading multiple HXPLVLx files.
  - **XFXTABL** – table‑driven logic, reading XFFTABLD and related tables.
  - **XFXMRNROL** – medical record number (MRN) roll‑over and profile linkage via HXHAPPPRF (missing) and HXXAPPPRF.
  - **XFXGETID** – identifier and XML record linkage, copying HXXLDA and reading HXFXMLR.
- **Configuration and Control Modules:** The SQLRPGLE **HXXAPPPRF** copies HXXCNTRL and HXXAPPPRFP, encapsulating application profile control logic and configuration table access.
- **File‑Centric Processing:** Programs issue READ and WRITE operations to DDS files such as HAPTRFR, HXFLVLx, XFFTABLD, XFFNSTN, HXFXMLH, and HXFXMLD. HABADTE is the primary consumer of transfer records and XML header/detail files.

The presence of copybooks, utility programs, and table‑driven datasets demonstrates a layered design: HABADTE orchestrates workflows using reusable utilities and configuration tables rather than embedding all logic inline.

## Section 2 – Database Schema

The database schema extracted from the aggregated context includes 19 physical files and 7 logical files. Each subsection below summarises the structural attributes of these objects.

### 2.1 Physical Files (PF)

For each PF, the record format, uniqueness, key fields, and PHI flags are documented.

#### HAPTRFR

- Record format: **HAFTRFR**
- Unique key: **Yes**
- Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- Total fields: 28
- PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN)

HAPTRFR stores transfer records tied to level‑6 organisational units and account numbers. PHI fields indicate direct linkage to patient accounts and MRNs.

#### HXPDICT

- Record format: **HXFDICT**
- Unique key: **No**
- Key fields: none
- Total fields: 2705
- PHI fields: `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`

HXPDICT is a large dictionary or cross‑reference table with extensive PHI attributes including MRNs, account numbers, phone numbers, names, room numbers, and dates of birth. It likely backs many lookups and validations across the system.

#### HXPLVL1 – HXPLVL6

Each HXPLVLx PF stores organisational level metadata.

- **HXPLVL1** (HXFLVL1)
  - Unique, key field: `HX1NUM`
  - Total fields: 36
  - PHI fields: none
- **HXPLVL2** (HXFLVL2)
  - Unique, key field: `HX2NUM`
  - Total fields: 39
  - PHI fields: none
- **HXPLVL3** (HXFLVL3)
  - Unique, key field: `HX3NUM`
  - Total fields: 39
  - PHI fields: none
- **HXPLVL4** (HXFLVL4)
  - Unique, key field: `HX4NUM`
  - Total fields: 39
  - PHI fields: none
- **HXPLVL5** (HXFLVL5)
  - Unique, key field: `HX5NUM`
  - Total fields: 42
  - PHI fields: none
- **HXPLVL6** (HXFLVL6)
  - Unique, key field: `HX6NUM`
  - Total fields: 155
  - PHI fields: none

These tables form a hierarchy of organisational levels used by XFXLDSC for level description lookups.

#### HXPTABLD

- Record format: **XFFTABLD**
- Unique key: **No**
- Key fields: `XFDTCD`, `XFDECD`
- Total fields: 7
- PHI fields: none

HXPTABLD is a compact table‑driven dictionary, with multiple logical views providing specialised access paths.

#### HXPXMLD

- Record format: **HXFXMLD**
- Unique key: **Yes**
- Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
- Total fields: 4
- PHI fields: none

Stores XML detail segments keyed by user and sequence, written by HABADTE.

#### HXPXMLR

- Record format: **HXFXMLR**
- Unique key: **Yes**
- Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`
- Total fields: 4
- PHI fields: none

Stores XML response or request records referenced by XFXGETID.

#### OAPIRNK

- Record format: **HBFIRNK**
- Unique key: **Yes**
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Total fields: 33
- PHI fields: `BRKMRN` (MRN)

#### OMPMAST

- Record format: **HMFMAST**
- Unique key: **Yes**
- Key fields: `MMPLV6`, `MMACCT`
- Total fields: 149
- PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`

A central patient master table containing MRNs, account numbers, patient names, and SSNs.

#### OXPBNFIT

- Record format: **XFFBNFIT**
- Unique key: **Yes**
- Key fields: `XFBUBN`, `XFBPLN`
- Total fields: 34
- PHI fields: `XFBTEL` (PhoneNumber)

#### OXPNSTN

- Record format: **XFFNSTN**
- Unique key: **Yes**
- Key fields: `XFNLV6`, `XFNSST`
- Total fields: 23
- PHI fields: none

#### TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN

These PFs are flagged as "ATE TABLE" with non‑unique keys:

- **TAPIRNK** – 43 fields, no PHI.
- **TMPMAST** – 181 fields, no PHI.
- **TXPBNFIT** – 12 fields, no PHI.
- **TXPNSTN** – 19 fields, no PHI.

They act as base tables for several logical files and support reference and staging use cases.

### 2.2 Logical Files (LF)

Each LF defines an access path or projection over a base PF.

#### HAPIRNK

- Based on PF: **TAPIRNK**
- Record format: **HBFIRNK**
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Select/Omit: none

Provides an indexed view over TAPIRNK for break and level lookups.

#### HMLMAST5H

- Based on PF: **TMPMAST**
- Record format: **HMFMAST**
- Key fields: `MMPNST`, `MMADDT`, `MMADTM`
- Select/Omit: none

Logical view over the TEMP master table, keyed by patient status and admission date/time.

#### HXLTABLD, HXLTABLP, HXLTABLS

All three LFs are based on **HXPTABLD (XFFTABLD)** and differ in their key sequencing:

- **HXLTABLD** – keys `XFDTCD`, `XFDMAP`
- **HXLTABLP** – keys `XFDTCD`, `XFDLDS`
- **HXLTABLS** – keys `XFDTCD`, `XFDSDS`

These represent specialised access paths into a shared dictionary table for different mapping, load, and subset criteria.

#### HXPBNFIT

- Based on PF: **TXPBNFIT**
- Record format: **XFFBNFIT**
- Key fields: `XFBUBN`, `XFBPLN`
- Select/Omit: none

#### HXPNSTN

- Based on PF: **TXPNSTN**
- Record format: **XFFNSTN**
- Key fields: `XFNLV6`, `XFNSST`
- Select/Omit: none

## Section 3 – Status and Type Reference Data

Approved business rules do not explicitly expose enumerated status code sets (e.g. lists of numeric codes), but rule text implies several status or type indicators:

- From HABADTE rules:
  - "Accounts marked as Voided (flag = 'V')" – status flag `V` indicates cancelled/voided accounts.
  - "I/O indicator = 'O' (Outpatient)" – type indicator `O` represents outpatient records; inpatients are implied by other values.
  - "file indicator = 0 is a pre-admission" – numeric indicator `0` denotes pre‑admission records.

These flags form part of the status/type reference data used by the patient census logic. No additional enumerated code tables are visible in the approved_rules subset. If further status codes exist, they are embedded in un‑approved rules or in constants within source code outside the approved set.

If modernisation requires formal status code catalogs, these indicators should be normalised into dedicated reference tables or enums based on a broader scan of the source.

## Section 4 – Stored Procedure Logic Mappings

Although the system does not use DB2 stored procedures explicitly, RPGLE programs behave as procedural units. The CALL edges can be treated as logical mappings.

### 4.1 Caller–Callee Grouping

- **HABADTE**
  - Calls: XFXMRNROL, XFXCNTR (three invocations), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL.
  - The multiple calls to XFXCNTR suggest repeated text‑centering operations for different report lines or sections.

- **XFXCYMD**
  - Calls: XFXLEAP.
  - Mapping: date validation procedure delegates leap‑year calculation to a dedicated helper.

- **XFXMRNROL**
  - Calls: HXHAPPPRF (missing), HXXAPPPRF.
  - Mapping: MRN roll logic invokes application profile procedures to update related tables.

- **HXXAPPPRF**
  - Copies: HXXCNTRL, HXXAPPPRFP.
  - Mapping: configuration and control copybooks are integrated into SQLRPGLE behaviour.

Each caller is a logical stored procedure from the perspective of external integration. For example, HABADTE’s call graph can be mapped to a service interface where each callee is a sub‑operation (validate dates, roll MRN, format text, etc.).

## Section 5 – Service Class Method Reference

Using hotspot scores, programs can be categorised into service classes:

- **BatchService (High score):**
  - **HABADTE** (score 38, fan_out 13, file_ops 6)
    - Acts as a batch‑style census engine that reads transfer and status tables, applies filters and rules, and produces XML and possibly spool outputs.

- **WorkflowService (Medium scores):**
  - **XFXLDSC** (score 15) – organisational level description workflow, reading multiple level tables.
  - **XFXTABL** (score 11) – table‑driven workflow service managing dictionary lookups.

- **UtilityService (Lower scores):**
  - **XFXCNTR** (score 9) – text centering utility.
  - **XFXMRNROL** (score 7) – MRN roll utility used by HABADTE.
  - **HXXAPPPRF** (score 7) – application profile utility wrapping SQL access.
  - **XFXGETID** (score 7) – identifier/XML linkage utility.
  - **XFXCYMD** (score 5) – date validation utility.
  - **XFXLEAP** (score 3) – leap‑year helper.

This classification supports service‑oriented refactoring, where high‑score programs become primary services and low‑score programs remain internal helpers.

## Section 6 – External Interfaces

High‑impact gaps and file operations reveal external interfaces:

- **XML Interface:**
  - HABADTE reads and writes **HXFXMLH** and **HXFXMLD**, and other components reference **HXPXMLD** and **HXPXMLR**. The missing ****HXPXML file suggests an additional XML repository or configuration structure.
  - These files together represent an inbound/outbound XML interface used to export the census or consume configuration.

- **Print/Spool Interface:**
  - The missing **PRINTER** file, referenced by HABADTE, indicates an outbound spool or printer file for census reports.

High‑medium gaps list CXXXMLC (copybook), HXHAPPPRF (program), and ****HXPXML (file) as inbound dependencies, meaning that external systems or unharvested modules feed data or structures used by HABADTE.

No explicit outbound SPOOL interfaces besides the PRINTER gap are visible in the analysed edges. **No outbound SPOOL interfaces identified.** The PRINTER reference should be treated as a likely spool output until its definition is recovered.

## Section 7 – Performance and Security Notes

### 7.1 Complexity Hotspots

Cyclomatic complexity per program highlights one major hotspot:

- **HABADTE:** cc = 152, band = HIGH – complex control flow covering many branches and conditions across patient census logic.
- All other measured programs (HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, CXXXMLP, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML) fall in the LOW band with cc values between 1 and 9.

Performance considerations:

- HABADTE’s high complexity increases the risk of performance issues, particularly in batch runs with large patient populations.
- XFXLDSC and XFXTABL, given their file read counts and hotspot scores, should be reviewed for efficient indexing and query patterns on HXPLVLx and XFFTABLx tables.

### 7.2 PHI Field Exposure

PHI‑tagged fields are concentrated in the following files:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT:** XFBTEL (PhoneNumber)

Security implications:

- Multiple files contain MRN, account, name, and SSN data. Access to these tables must be tightly controlled, with auditing enabled on HABADTE, XFXGETID, and any other program that reads or writes these PFs.
- Encryption or tokenisation should be considered for SSN (MMPSSN) and MRN fields when migrating to modern platforms.

### 7.3 Tech Debt Overview

The tech_debt_summary reports:

- Total findings: 4
- Total remediation hours: 26.9
- Severity distribution: HIGH – 1, MEDIUM – 3, LOW – 0

Though the structural analysis does not list each individual finding, the severity mix suggests focused remediation needs on a small number of high‑impact issues, likely related to HABADTE’s complexity, missing components, and PHI handling.

From a low‑level design perspective, remediation should prioritise:

- Reducing HABADTE’s cyclomatic complexity via modularisation.
- Clarifying and documenting external interfaces (XML, printer) where gaps exist.
- Strengthening security controls around PHI‑heavy tables.
