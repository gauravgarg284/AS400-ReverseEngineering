# Low-Level Design (LLD)

PIPELINE_RUN-ID: 202607011449

## 1. Architecture Overview

### 1.1 Technology Stack Composition

The reverse-engineered system is implemented entirely on the IBM i (AS/400) platform using classic DDS-based database definitions and RPG-family programs.

Source type distribution:

- DDS logical files (DDS_LF): 7
- DDS physical files (DDS_PF): 15
- SQLRPGLE programs: 2
- RPGLE programs: 13

This mix reflects a traditional IBM i workload where:

- DDS_PF members (e.g., HAPTRFR, HXPDICT, HXPLVL1–HXPLVL6, OAPIRNK, OMPMAST, OXPBNFIT, OXPNSTN) define the core data model.
- DDS_LF members (e.g., HAPIRNK, HMLMAST5H, HXLTABLD, HXLTABLP, HXLTABLS, HXPBNFIT, HXPNSTN) provide alternate keyed access paths for query and batch processing.
- RPGLE and SQLRPGLE programs (HABADTE, XFX*, HXX*, CXXXMLP) implement business logic, utility services, and SQL-based operations.

### 1.2 Call Graph Summary

At a high level, the calling structure is:

- **HABADTE**: Primary batch driver and orchestration program. It:
  - Calls XFXMRNROL, XFXCNTR (multiple times), XFXLDSC, XFXCYMD, XFXGETID, and XFXTABL.
  - Performs file operations against HAPTRFR, XFFNSTN, HXFXMLH, and HXFXMLD.

- **XFXMRNROL**:
  - Called from HABADTE.
  - Calls HXHAPPPRF (missing) and HXXAPPPRF, indicating a delegation to application-profile logic.

- **XFXCYMD**:
  - Called by HABADTE.
  - Calls XFXLEAP for leap-year and date validation.

- **HXXAPPPRF**:
  - Called by XFXMRNROL.
  - Performs COPY operations against HXXCNTRL and HXXAPPPRFP.

- **XFXGETID**:
  - Called by HABADTE.
  - Copies HXXLDA and reads HXFXMLR.

Other utilities, such as XFXLDSC and XFXTABL, are called from HABADTE and provide data lookups across level and table PFs.

The call graph forms a multi-tiered architecture:

- Tier 1: HABADTE (entry point / batch controller).
- Tier 2: XFX* utilities and MRN/ID services.
- Tier 3: HXX* control and application profile modules.

### 1.3 Design Patterns Observed

Several patterns emerge from the dependency analysis:

- **Controller–Service Pattern**: HABADTE acts as a controller orchestrating services provided by XFX* programs. Each XFX program encapsulates a specific concern (e.g., date validation, MRN handling, ID retrieval, table lookup).

- **Table-Driven Design**: Extensive use of PFs and LFs, especially HXPTABLD and the HXPLVL* series, points to table-driven logic. Configuration and categorizations are encoded in data tables rather than code constants.

- **Logical Overlay Pattern**: Logical files (LFs) such as HXLTABLD/HXLTABLP/HXLTABLS and HMLMAST5H provide alternative views on PFs. This allows the same physical dataset to support multiple access patterns and business processes.

- **Copybook-Based Modularization**: Copy dependencies (CXXXMLP, CXXXMLC, HXXLDA, HXXLEVEL, HXXXML, HXXCNTRL, HXXAPPPRFP) indicate reuse of common data structures and control blocks across programs, aligning with a modular design approach.

## 2. Database Schema

The database schema is derived from the compact data dictionary schema in the aggregated context. It is split into physical file (PF) and logical file (LF) subsections.

### 2.1 Physical Files (PF)

Each PF subsection summarizes record format, key structure, uniqueness, field count, and PHI presence.

#### 2.1.1 HAPTRFR

- Record format: HAFTRFR
- Unique key: Yes
- Key fields: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- Total fields: 28
- PHI fields: AFACCT (AccountNumber), AFMRNO (MRN)

HAPTRFR stores transfer records keyed by level 6 identifier, account number, transfer date/time, and type. The presence of account and MRN fields marks it as PHI-bearing. HABADTE reads HAPTRFR to drive downstream XML generation.

#### 2.1.2 HXPDICT

- Record format: HXFDICT
- Unique key: No
- Key fields: (none specified)
- Total fields: 2705
- PHI fields: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HXPDICT is a large dictionary-style PF with over 2,700 fields. It carries significant PHI, including MRNs, phone numbers, patient names, account numbers, and dates of birth. It likely functions as a central reference dictionary for multiple subsystems.

#### 2.1.3 HXPLVL1–HXPLVL6

Each HXPLVLx PF represents a level table with a specific numeric key:

- HXPLVL1 (HXFLVL1): Key HX1NUM, unique=true, total_fields=36, PHI fields: none.
- HXPLVL2 (HXFLVL2): Key HX2NUM, unique=true, total_fields=39, PHI fields: none.
- HXPLVL3 (HXFLVL3): Key HX3NUM, unique=true, total_fields=39, PHI fields: none.
- HXPLVL4 (HXFLVL4): Key HX4NUM, unique=true, total_fields=39, PHI fields: none.
- HXPLVL5 (HXFLVL5): Key HX5NUM, unique=true, total_fields=42, PHI fields: none.
- HXPLVL6 (HXFLVL6): Key HX6NUM, unique=true, total_fields=155, PHI fields: none.

These PFs form a hierarchical level model. XFXLDSC reads across all six PFs, implying composite descriptions or multi-level reference data.

#### 2.1.4 HXPTABLD

- Record format: XFFTABLD
- Unique key: No
- Key fields: XFDTCD, XFDECD
- Total fields: 7
- PHI fields: none

HXPTABLD is a non-unique code-table PF. Logical overlays (HXLTABLD, HXLTABLP, HXLTABLS) use different key composition and selection logic to support specific lookup needs.

#### 2.1.5 HXPXMLD

- Record format: HXFXMLD
- Unique key: Yes
- Key fields: XMDUSR, XMDSEQ, XMDSQ2
- Total fields: 4
- PHI fields: none

HXPXMLD stores XML detail records keyed by user and sequence numbers. HABADTE writes to this file, indicating its role as a persistent XML payload store.

#### 2.1.6 HXPXMLR

- Record format: HXFXMLR
- Unique key: Yes
- Key fields: XMRUSR, XMRSEQ, XMRID
- Total fields: 4
- PHI fields: none

HXPXMLR is the XML request or result header PF. XFXGETID reads from HXFXMLR to derive IDs, suggesting it stores XML-based request metadata.

#### 2.1.7 OAPIRNK

- Record format: HBFIRNK
- Unique key: Yes
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Total fields: 33
- PHI fields: BRKMRN (MRN)

OAPIRNK holds ranking or break records, keyed by a level identifier, account, and sequence. The MRN-bearing field BRKMRN introduces PHI.

#### 2.1.8 OMPMAST

- Record format: HMFMAST
- Unique key: Yes
- Key fields: MMPLV6, MMACCT
- Total fields: 149
- PHI fields: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN

OMPMAST is a master PF for patient or account entities. It holds MRN, account number, patient name, and SSN, all of which are highly sensitive. This PF underpins many patient-centric operations.

#### 2.1.9 OXPBNFIT

- Record format: XFFBNFIT
- Unique key: Yes
- Key fields: XFBUBN, XFBPLN
- Total fields: 34
- PHI fields: XFBTEL (PhoneNumber)

OXPBNFIT stores benefit data keyed by benefit number and plan. The presence of a telephone number field indicates PHI.

#### 2.1.10 OXPNSTN

- Record format: XFFNSTN
- Unique key: Yes
- Key fields: XFNLV6, XFNSST
- Total fields: 23
- PHI fields: none

OXPNSTN stores station-level data keyed by level and station code.

### 2.2 Logical Files (LF)

Logical files provide alternate indexed views of PFs.

#### 2.2.1 HAPIRNK

- Based on PF: TAPIRNK (missing)
- Record format: HBFIRNK
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Select/omit: none specified

HAPIRNK is a logical overlay on TAPIRNK, aligning with the OAPIRNK PF’s format. It likely provides indexed access to ranking or break records for the HAP subsystem. The missing TAPIRNK PF means the underlying physical structure is inferred but not fully known.

#### 2.2.2 HMLMAST5H

- Based on PF: TMPMAST (missing)
- Record format: HMFMAST
- Key fields: MMPNST, MMADDT, MMADTM
- Select/omit: none specified

HMLMAST5H presents a view of TMPMAST keyed by post or posting date/time fields, providing chronological access to master records. Lack of TMPMAST prevents exact field-level analysis.

#### 2.2.3 HXLTABLD

- Based on PF: HXPTABLD
- Record format: XFFTABLD
- Key fields: XFDTCD, XFDMAP
- Select/omit: none specified

HXLTABLD offers an alternate key for HXPTABLD focusing on data-code and mapping fields. It supports specific lookup scenarios where code-to-mapping resolution is required.

#### 2.2.4 HXLTABLP

- Based on PF: HXPTABLD
- Record format: XFFTABLD
- Key fields: XFDTCD, XFDLDS
- Select/omit: none specified

HXLTABLP provides a variant keyed by data-code and a “loads” or “list” field, likely representing different logical slices of the same codes.

#### 2.2.5 HXLTABLS
n- Based on PF: HXPTABLD
- Record format: XFFTABLD
- Key fields: XFDTCD, XFDSDS
- Select/omit: none specified

HXLTABLS focuses on data-code and a “subset” or “sub-code” field, allowing specialized subsets of the table to be addressed.

#### 2.2.6 HXPBNFIT (LF)

- Based on PF: TXPBNFIT (missing)
- Record format: XFFBNFIT
- Key fields: XFBUBN, XFBPLN
- Select/omit: none specified

The HXPBNFIT LF mirrors the key structure of the OXPBNFIT PF but is based on TXPBNFIT. It likely feeds the HXP subsystem with benefit data.

#### 2.2.7 HXPNSTN

- Based on PF: TXPNSTN (missing)
- Record format: XFFNSTN
- Key fields: XFNLV6, XFNSST
- Select/omit: none specified

HXPNSTN overlays TXPNSTN to provide an ordered view on station-level data.

## 3. Status and Type Reference Data

The approved business rules provide hints about status and type codes used in the system.

Key rules include:

- BR-001: "When X equals zero, branch to 'EXIT'" (from XFXCNTR)
- BR-002: "When X equals 40, branch to 'EXIT'" (from XFXCNTR)
- BR-003: "When VYY is less than 1800, branch to 'EXIT'" (from XFXCYMD)
- BR-004: "When VYY is greater than 2100, branch to 'EXIT'" (from XFXCYMD)
- BR-005: "When VMM is less than 01, branch to 'EXIT'" (from XFXCYMD)

These rules indicate:

- XFXCNTR uses integer codes (e.g., 0, 40) to control branching, likely representing status or step codes in a batch process.
- XFXCYMD enforces date validation on VYY (year) and VMM (month), rejecting years outside 1800–2100 and months less than 1.

No explicit textual status or type code tables are embedded in the rule text beyond these numeric values. Therefore, distinct reference-data tables for statuses and types cannot be fully reconstructed from rules alone.

Where additional status/type codes exist, they are most likely encoded in:

- HXPDICT (dictionary PF), where code fields reference enumerated states.
- HXPTABLD and its logical views, where XFDTCD and related fields represent classification codes.

## 4. Stored Procedure Logic Mappings

In this context, stored procedure logic corresponds to CALL-based relationships between programs.

### 4.1 HABADTE Caller Group

**HABADTE** acts as the top-level caller.

- Callees:
  - XFXMRNROL: MRN rollover or MRN-related logic.
  - XFXCNTR: control or counter utility (called three times).
  - XFXLDSC: level description lookup across HXPLVL1–HXPLVL6.
  - XFXCYMD: date validation and calendar-related checks.
  - XFXGETID: ID allocation or retrieval based on HXFXMLR.
  - XFXTABL: table/dictionary lookup over XFFTABLD* tables.

Call frequencies (inferred from multiple edges):

- XFXCNTR: 3 CALL edges from HABADTE.
- XFXMRNROL, XFXLDSC, XFXCYMD, XFXGETID, XFXTABL: 1 CALL edge each.

### 4.2 XFXMRNROL Caller Group

**XFXMRNROL** acts as an intermediate caller.

- Callees:
  - HXHAPPPRF (missing): likely a host or external application profile program.
  - HXXAPPPRF: SQLRPGLE program with embedded SQL for profile operations.

This chain indicates that MRN-related operations route through XFXMRNROL to application-profile logic.

### 4.3 XFXCYMD Caller Group

**XFXCYMD** has a relatively simple mapping:

- Callee:
  - XFXLEAP: a utility for leap-year calculations.

### 4.4 Additional Copy-Based Logic

While COPY edges are not CALLs, they establish tight coupling:

- HXXAPPPRF copies HXXCNTRL and HXXAPPPRFP.
- XFXGETID and HABADTE copy HXXLDA.
- HABADTE copies HXXLEVEL, HXXXML, CXXXMLP, and the missing CXXXMLC.

In a modernization context, these copybook relationships should be modeled as shared modules or common structures in the target language, and their initialization order must be retained.

## 5. Service Class Method Reference

We classify programs by hotspot scores to infer service classes.

### 5.1 BatchService-Class Programs (High Score)

- **HABADTE**
  - Score: 38
  - Fan-in: 0 (entry point)
  - Fan-out: 13
  - File operations: 6
  - Classification: BatchService

HABADTE is the central batch driver orchestrating multiple operations: reading HAPTRFR and station data, invoking MRN and ID services, performing dictionary lookups, and writing XML headers and details.

### 5.2 WorkflowService-Class Programs (Medium Score)

Programs with medium hotspot scores are treated as WorkflowServices:

- **XFXLDSC**
  - Score: 15, fan-in: 1, file_ops: 6.
  - Role: traverses level tables HXPLVL1–HXPLVL6 to compute descriptions or derived attributes.

- **XFXTABL**
  - Score: 11, fan-in: 1, file_ops: 4.
  - Role: handles generic table lookups across XFFTABLD and related tables.

- **XFXCNTR**
  - Score: 9, fan-in: 3, file_ops: 0.
  - Role: controls branching and step progression based on numeric codes.

- **XFXMRNROL**
  - Score: 7, fan-in: 1, fan-out: 2.
  - Role: orchestrates MRN rollover or profile-driven MRN operations.

- **XFXGETID**
  - Score: 7, fan-in: 1, fan-out: 1.
  - Role: obtains identifiers based on XML headers in HXFXMLR.

- **HXXAPPPRF**
  - Score: 7, fan-in: 1, fan-out: 2.
  - Role: executes application profile logic using HXXCNTRL and HXXAPPPRFP.

### 5.3 UtilityService-Class Programs (Low Score)

Remaining programs with lower scores are UtilityServices:

- **XFXCYMD**
  - Score: 5, fan-in: 1, fan-out: 1.
  - Role: date validation.

- **HXHAPPPRF**
  - Score: 3, fan-in: 1.
  - Role: external or host application profile logic.

- **XFXLEAP**
  - Score: 3, fan-in: 1.
  - Role: leap-year computation.

- Shorter PF/LF hotspots (e.g., HXLTABLS, HMLMAST5H, HXPNSTN, HAPIRNK, HXLTABLD, HXPBNFIT) with score=2 are simple accessors and do not represent complex services.

## 6. External Interfaces

External interfaces have been inferred from high/medium gaps and file dependencies.

### 6.1 Inbound Interfaces

High-impact gaps represent inbound interfaces where data arrives from external systems:

- **TAPIRNK (FILE)**
  - Referenced by: HAPIRNK.
  - Role: base PF for inbound ranking or break records.

- **TMPMAST (FILE)**
  - Referenced by: HMLMAST5H.
  - Role: base PF for master data, likely representing inbound master records.

- **TXPBNFIT (FILE)**
  - Referenced by: HXPBNFIT.
  - Role: base PF for benefits data imported into HXP/OXP subsystems.

- **TXPNSTN (FILE)**
  - Referenced by: HXPNSTN.
  - Role: base PF for station-level reference data.

- **CXXXMLC (COPYBOOK)**
  - Referenced by: HABADTE.
  - Role: likely provides column or layout definitions for XML structures used when reading/writing external XML files.

- **HXHAPPPRF (PROGRAM)**
  - Referenced by: XFXMRNROL.
  - Role: external application profile program invoked within the process, potentially external to the current library list.

### 6.2 Outbound Interfaces

HABADTE’s file operations indicate outbound interfaces:

- XML Headers/Details:
  - HXFXMLH: read/update/write operations.
  - HXFXMLD: write operations.

These PFs likely feed downstream systems via XML export processes. HXPXMLD and HXPXMLR PF definitions confirm that XML data is persisted in DDS tables.

No outbound SPOOL interfaces were positively identified, and the PRINTER file is missing. Therefore:

- No outbound SPOOL interfaces identified.

## 7. Performance and Security Notes

### 7.1 Complexity-Based Performance Risks

Cyclomatic complexity per program shows:

- HABADTE: cc=152 (band=HIGH).
- All other programs: cc between 1 and 9 (band=LOW).

The extreme complexity of HABADTE indicates a significant performance and maintainability risk. Long control paths, nested conditions, and multiple CALL sites increase the cost of change and testing. For modernization:

- HABADTE should be refactored into smaller services aligned with XFX and HXX utility boundaries.
- Critical path analysis should focus on HABADTE’s most frequently executed branches.

### 7.2 PHI Exposure and Security

PHI-tagged fields appear in several PFs:

- **HAPTRFR**: AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT**: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK**: BRKMRN (MRN)
- **OMPMAST**: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT**: XFBTEL (PhoneNumber)

Security implications:

- Multiple PFs store MRNs, account numbers, patient names, SSNs, phone numbers, and date of birth. These must be protected with appropriate authority checks and masking when exported.
- XFX* utilities accessing HXPDICT, OMPMAST, or OAPIRNK indirectly inherit PHI handling responsibilities.
- Any XML generation via HABADTE must ensure PHI is either encrypted, tokenized, or omitted according to policy.

### 7.3 Tech Debt Summary

The structured tech debt assessment reports:

- total_findings: 0
- total_remediation_hours: 0.0
- by_severity: HIGH=0, MEDIUM=0, LOW=0

Although no explicit findings were recorded, the following implicit issues warrant remediation:

- Single high-complexity driver program (HABADTE) concentrating control flow.
- Missing PFs and copybooks (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN, CXXXMLC, HXHAPPPRF, PRINTER) reducing observability and increasing migration risk.
- Widespread PHI fields requiring consistent masking and access control.

Addressing these risks should be part of any modernization or refactoring effort, even if the automated tech debt estimator reports zero hours.
