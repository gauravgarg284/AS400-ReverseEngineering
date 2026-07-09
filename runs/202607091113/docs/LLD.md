# Low-Level Design (LLD)

## Section 1 – Architecture Overview

### 1.1 Technology Stack and Component Types

The application is a traditional IBM i (AS/400) system built primarily from DDS-defined physical and logical files and a mix of RPGLE and SQLRPGLE programs.

Type distribution:

- DDS_LF: 7 members
- DDS_PF: 19 members
- SQLRPGLE: 2 members
- RPGLE: 13 members

The DDS artifacts implement the database layer, while RPGLE and SQLRPGLE programs form the service and batch processing layer. The program set is characterized by:

- A single high-complexity batch driver (HABADTE) in subsystem HA.
- Supporting RPGLE utilities: XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML.
- SQLRPGLE components HXXAPPPRF and CXXXMLP that provide DB-aware and XML-aware functionality.

### 1.2 Call Graph Summary

Key CALL relationships derived from the dependency edges:

- HABADTE → XFXMRNROL
- HABADTE → XFXCNTR (three separate call sites)
- HABADTE → XFXLDSC
- HABADTE → XFXCYMD
- HABADTE → XFXGETID
- HABADTE → XFXTABL

Secondary call chains:

- XFXCYMD → XFXLEAP
- XFXMRNROL → HXHAPPPRF (missing program)
- XFXMRNROL → HXXAPPPRF (SQLRPGLE)

This call graph reveals a central orchestration model: HABADTE acts as a controller that delegates specialized tasks (date validation, table lookup, MRN handling, ID retrieval) to dedicated service routines.

### 1.3 Design Patterns

The following design patterns are evident:

- **Service Utility Pattern**: XFXCYMD (date validation), XFXLEAP (leap year detection), XFXCNTR (field formatting/counter handling), and XFXTABL (table lookup) act as stateless utilities invoked by HABADTE and potentially other programs.
- **Table-Driven Behavior**: Use of HXPTABLD plus associated logical files (HXLTABLD, HXLTABLP, HXLTABLS) and multiple XFFTABL* files indicates table-driven configuration for codes, descriptions, and mappings.
- **XML Integration Pattern**: HXPXMLD, HXPXMLR, HXFXMLH, and HXFXMLD form an XML message infrastructure, with CXXXMLP and the missing copybook CXXXMLC providing XML-specific behavior.
- **Layered Data Access**: Logical files over shared PFs (e.g., HXPTABLD with multiple LFs) provide separate access paths for different business views without duplicating underlying data.

## Section 2 – Database Schema

This section describes the PF and LF structures using the compact `data_dict_schema`.

### 2.1 Physical Files (PFs)

For each PF, the key elements are record format, key fields, uniqueness, total fields, and PHI-related fields.

#### HAPTRFR

- Record format: HAFTRFR
- Unique key: true
- Key fields: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- Total fields: 28
- PHI fields: AFACCT (AccountNumber), AFMRNO (MRN)

Design notes:

- Represents transfer records (likely patient or account transfers) keyed by level, account, and transfer date/time.
- Used by HABADTE via READ operations, feeding data into XML generation.

#### HXPDICT

- Record format: HXFDICT
- Unique key: false
- Key fields: none (likely accessed via multiple LFs in the full system)
- Total fields: 2705
- PHI fields (examples): CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

Design notes:

- Central dictionary file containing a broad set of reference and domain values, including MRNs, names, account numbers, and other patient-identifying data.
- High field count suggests consolidation of many logical concepts; modernization may benefit from decomposing this PF into more normalized structures.

#### HXPLVL1–HXPLVL6

Each HXPLVLx file (x = 1..6) represents level-specific metadata:

- HXPLVL1
  - Record format: HXFLVL1
  - Unique key: true
  - Key fields: HX1NUM
  - Total fields: 36
  - PHI fields: none
- HXPLVL2
  - Record format: HXFLVL2
  - Unique key: true
  - Key fields: HX2NUM
  - Total fields: 39
- HXPLVL3
  - Record format: HXFLVL3
  - Unique key: true
  - Key fields: HX3NUM
  - Total fields: 39
- HXPLVL4
  - Record format: HXFLVL4
  - Unique key: true
  - Key fields: HX4NUM
  - Total fields: 39
- HXPLVL5
  - Record format: HXFLVL5
  - Unique key: true
  - Key fields: HX5NUM
  - Total fields: 42
- HXPLVL6
  - Record format: HXFLVL6
  - Unique key: true
  - Key fields: HX6NUM
  - Total fields: 155

Design notes:

- These level master files are read by XFXLDSC to compute or display descriptive information for hierarchical levels (e.g., coverage tiers, networks, benefit levels).
- The increasing total_fields towards HXPLVL6 suggests deeper or more complex level detail.

#### HXPTABLD

- Record format: XFFTABLD
- Unique key: false
- Key fields: XFDTCD, XFDECD
- Total fields: 7
- PHI fields: none

Design notes:

- Core table dictionary for code-to-description mappings.
- Accessed directly by XFXTABL and indirectly via multiple LFs.

#### HXPXMLD and HXPXMLR

- HXPXMLD
  - Record format: HXFXMLD
  - Unique key: true
  - Key fields: XMDUSR, XMDSEQ, XMDSQ2
  - Total fields: 4
- HXPXMLR
  - Record format: HXFXMLR
  - Unique key: true
  - Key fields: XMRUSR, XMRSEQ, XMRID
  - Total fields: 4

Design notes:

- Represent XML document headers and detail records or request/response tracking.
- XFXGETID reads HXFXMLR; HABADTE writes HXFXMLD and updates HXFXMLH (not explicitly listed as PF but referenced via file operations).

#### OAPIRNK

- Record format: HBFIRNK
- Unique key: true
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Total fields: 33
- PHI fields: BRKMRN (MRN)

#### OMPMAST

- Record format: HMFMAST
- Unique key: true
- Key fields: MMPLV6, MMACCT
- Total fields: 149
- PHI fields: MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)

Design notes:

- Core patient and account master file; high PHI sensitivity.

#### OXPBNFIT

- Record format: XFFBNFIT
- Unique key: true
- Key fields: XFBUBN, XFBPLN
- Total fields: 34
- PHI fields: XFBTEL (PhoneNumber)

#### OXPNSTN

- Record format: XFFNSTN
- Unique key: true
- Key fields: XFNLV6, XFNSST
- Total fields: 23

#### TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN

- TAPIRNK
  - Record format: ATE TABLE
  - Unique key: false
  - Total fields: 43
- TMPMAST
  - Record format: ATE TABLE
  - Unique key: false
  - Total fields: 181
- TXPBNFIT
  - Record format: ATE TABLE
  - Unique key: false
  - Total fields: 12
- TXPNSTN
  - Record format: ATE TABLE
  - Unique key: false
  - Total fields: 19

Design notes:

- Likely test or transitional equivalents of the O*-prefixed PFs, used for staging or non-production environments.

### 2.2 Logical Files (LFs)

#### HAPIRNK

- PFILE: TAPIRNK
- Record format: HBFIRNK
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Select/omit: none

Provides a keyed view of TAPIRNK to support rank/sequence-based queries.

#### HMLMAST5H

- PFILE: TMPMAST
- Record format: HMFMAST
- Key fields: MMPNST, MMADDT, MMADTM
- Select/omit: none

Provides a view of TMPMAST sorted by plan/state and admission date/time.

#### HXLTABLD, HXLTABLP, HXLTABLS

All three LFs share the PF HXPTABLD but differ in keys:

- HXLTABLD
  - PFILE: HXPTABLD
  - Record format: XFFTABLD
  - Key fields: XFDTCD, XFDMAP
- HXLTABLP
  - PFILE: HXPTABLD
  - Record format: XFFTABLD
  - Key fields: XFDTCD, XFDLDS
- HXLTABLS
  - PFILE: HXPTABLD
  - Record format: XFFTABLD
  - Key fields: XFDTCD, XFDSDS

These provide specialized access paths for distinct mapping dimensions while reusing the same base table file.

#### HXPBNFIT

- PFILE: TXPBNFIT
- Record format: XFFBNFIT
- Key fields: XFBUBN, XFBPLN

#### HXPNSTN

- PFILE: TXPNSTN
- Record format: XFFNSTN
- Key fields: XFNLV6, XFNSST

Together, these LFs provide production-style views on test/staging PFs.

## Section 3 – Status and Type Reference Data

Status and type codes are primarily expressed via table and dictionary structures and referenced in business rules.

Using `approved_rules`:

- BR-001 / BR-002 (XFXCNTR): Field formatting rules that enforce blank/non-blank patterns, likely applied to status or type fields entered as fixed-length codes.
- BR-003 / BR-004 / BR-005 (XFXCYMD): Date validation rules that define acceptable ranges for year and month.

There is no explicit enumeration of status or type codes in the rules themselves (only their validation behavior). The concrete code values are likely stored in HXPTABLD/XFFTABL* or HXPDICT.

Status and type codes are therefore inferred rather than directly listed in the rule text.

None identified as explicit status-code lists in the business rule text; all status-related behavior is table driven.

## Section 4 – Stored Procedure Logic Mappings

This section describes program-to-program CALL relationships, treating each caller as an analog to a stored procedure.

### 4.1 HABADTE Caller Map

HABADTE is the primary caller in the system:

- Callees:
  - XFXMRNROL (1 call site)
  - XFXCNTR (3 call sites)
  - XFXLDSC (1 call site)
  - XFXCYMD (1 call site)
  - XFXGETID (1 call site)
  - XFXTABL (1 call site)

Semantics:

- HABADTE orchestrates transfer processing, MRN management, date validation, ID assignment, and XML/tables-driven logic. Each call to XFXCNTR likely aligns with separate fields or sections requiring formatting or validation.

### 4.2 Secondary Callers

- XFXCYMD
  - Callee: XFXLEAP (1 call site)
  - Purpose: Offloads leap-year detection from the main date validation logic.

- XFXMRNROL
  - Callees:
    - HXHAPPPRF (1 call site; missing program)
    - HXXAPPPRF (1 call site; SQLRPGLE)
  - Purpose: MRN roll operations, potentially conditioning behavior on application profiles.

No other CALL edges are present in the aggregated dependency list. Most other dependencies are COPYs or file access operations.

## Section 5 – Service Class Method Reference

Based on `dep_hotspots`, programs are classified into service classes using their hotspot scores.

Classification rules:

- Score ≥ 20: BatchService (high-volume, orchestrating batch processes)
- Score between 8 and 19: WorkflowService (mid-level coordination or complex operations)
- Score < 8: UtilityService (simple, reusable routines)

### 5.1 BatchService

- **HABADTE**
  - Score: 38
  - Fan-in: 0
  - Fan-out: 13
  - File operations: 6
  - Class: BatchService
  - Role: Core batch controller for transfer and XML processing. Handles complex branching and file I/O.

### 5.2 WorkflowService

- **XFXLDSC**
  - Score: 15
  - Fan-in: 1
  - Fan-out: 0
  - File operations: 6
  - Class: WorkflowService
  - Role: Orchestrates reads across HXPLVL1–HXPLVL6 for level/benefit lookup.

- **XFXTABL**
  - Score: 11
  - Fan-in: 1
  - Fan-out: 0
  - File operations: 4
  - Class: WorkflowService
  - Role: Performs multi-table lookups, centralizing table-driven behavior.

- **XFXCNTR**
  - Score: 9
  - Fan-in: 3
  - Fan-out: 0
  - File operations: 0
  - Class: WorkflowService (upper Utility boundary)
  - Role: Controls counters and field formatting in several contexts.

- **XFXMRNROL**
  - Score: 7 (borderline), but with fan_out 2 and MRN semantics
  - Class: UtilityService, but critical for patient identifier workflows.

### 5.3 UtilityService

- XFXGETID (score 7, fan-in 1, fan-out 1, file_ops 1)
- HXXAPPPRF (score 7, fan-in 1, fan-out 2)
- XFXCYMD (score 5, fan-in 1, fan-out 1)
- XFXLEAP (score 3, fan-in 1, fan-out 0)
- HXHAPPPRF (score 3, fan-in 1, fan-out 0; missing implementation)
- Remaining hotspots with scores 2 (HMLMAST5H, HXLTABLD, HXPNSTN, HAPIRNK, HXLTABLS, HXLTABLP, HXPBNFIT) operate as low-level utilities around PF/LF access.

## Section 6 – External Interfaces

The `high_medium_gaps` list highlights external interfaces and missing components.

- CXXXMLC (COPYBOOK, HIGH impact) – part of the XML infrastructure used by HABADTE and CXXXMLP.
- HXHAPPPRF (PROGRAM, MEDIUM impact) – externalized application profile logic invoked by XFXMRNROL.
- ****HXPXML (FILE, MEDIUM impact) – XML-related file used by HABADTE as an inbound or outbound interface.

These are treated as inbound interfaces, as they represent external dependencies not present in the scanned repository.

For spool/printer interfaces, only the PRINTER file is referenced in the broader gap report. No explicit SPOOL file operations are captured in the dependency edges for this run.

No outbound SPOOL interfaces identified.

## Section 7 – Performance and Security Notes

### 7.1 Performance Considerations

From `complexity_per_program`:

- HABADTE: cc = 152, HIGH band
- XFXTABL: cc = 9, LOW band
- XFXLDSC: cc = 5, LOW band
- Remaining programs: cc ≤ 7, LOW band

Implications:

- HABADTE’s high cyclomatic complexity makes it the primary candidate for performance optimization and refactoring.
- Other programs have low complexity and are likely not performance hot spots in isolation; their impact arises from being invoked frequently within HABADTE.

### 7.2 Security and PHI Handling

PHI-tagged fields from `phi_flagged_fields`:

- HAPTRFR: AFACCT (AccountNumber), AFMRNO (MRN)
- HXPDICT: CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN (MRN), HVACCT, IHACCT (AccountNumber), XFBTEL (PhoneNumber), XCNAME, ENNAME (PatientName), HXRMNO, XFRMNO (RoomNumber), WBDATE (DateOfBirth)
- OAPIRNK: BRKMRN (MRN)
- OMPMAST: MMMRNO, MMMMRN (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN)
- OXPBNFIT: XFBTEL (PhoneNumber)

Security notes:

- PHI is concentrated in HXPDICT and OMPMAST, with additional MRN/Account fields in HAPTRFR and OAPIRNK.
- Programs accessing these PFs (HABADTE, XFXGETID, XFXMRNROL and others in the full system) should be subject to stricter access controls and audit logging in any modernized environment.

### 7.3 Technical Debt Overview

From `tech_debt_summary`:

- Total findings: 4
- Total remediation hours: 26.9
- By severity: HIGH = 1, MEDIUM = 3, LOW = 0

These findings likely correspond to complex, high-risk areas such as HABADTE, missing components, and concentrated PHI access points. Addressing them will require targeted refactoring and hardening of integration paths and security boundaries.
