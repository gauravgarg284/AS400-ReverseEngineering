# Low Level Design (LLD) – HABADTE Application

## 1. Architecture Overview

### 1.1 Technology Stack Composition

The HABADTE application is a traditional AS400 solution built with the following component types (from the aggregated manifest):

- **DDS Logical Files (DDS_LF):** 7
- **DDS Physical Files (DDS_PF):** 19
- **RPGLE Programs:** 13
- **SQLRPGLE Programs:** 2

Logical and physical files define the schema and access paths, while RPGLE and SQLRPGLE programs implement business logic. The main functional domain is **PATIENT_MANAGEMENT**, anchored by the HABADTE driver. The XFX* utilities belong to the **DATA_MAINTENANCE** domain and are reused by HABADTE and other programs.

### 1.2 Call Graph Summary

From dependency edges:

- HABADTE → XFXMRNROL, XFXCNTR (3×), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL
- XFXCYMD → XFXLEAP
- XFXMRNROL → HXHAPPPRF (missing), HXXAPPPRF
- HXXAPPPRF → HXXCNTRL, HXXAPPPRFP (COPY)
- HABADTE → HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC (COPY)
- XFXGETID → HXXLDA (COPY)

This forms a hub-and-spoke topology:

- **HABADTE** is the orchestration hub that coordinates patient transfer processing and XML output.
- **XFX* programs** are spokes providing specialized services:
  - XFXCNTR – counter and control number services
  - XFXCYMD – date validation and calendar rules
  - XFXLDSC – level/plan description resolution
  - XFXGETID – identifier retrieval for XML and other flows
  - XFXTABL – generic table-lookup service
  - XFXMRNROL – MRN rollover logic that bridges to HXXAPPPRF/HXHAPPPRF

### 1.3 Design Patterns Observed

- **Utility Service Pattern:** XFX* programs encapsulate small, reusable rules (dates, counters, level lookups) and are invoked by the main driver instead of duplicating logic.
- **Configuration via Tables:** HXPTABLD and related level tables (HXPLVL1–HXPLVL6) are used by XFXTABL and XFXLDSC as configuration/reference tables rather than hard-coded logic.
- **File-Driven Integration:** HABADTE reads/writes DDS-described files HXFXMLH and HXFXMLD and uses copybooks CXXXMLP/CXXXMLC, indicating an XML integration boundary where file layouts define the contract.

## 2. Database Schema

This section summarizes the PF/LF schema based on the compact data dictionary.

### 2.1 Physical Files (PF)

For each PF, we list record format, uniqueness, key fields, and PHI involvement.

- **HAPTRFR**  
  - Record format: HAFTRFR  
  - Unique: true  
  - Key fields: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE  
  - Total fields: 28  
  - PHI fields: AFACCT (AccountNumber), AFMRNO (MRN)  
  This table stores patient transfer records keyed by level, account, date/time, and transfer type.

- **HXPDICT**  
  - Record format: HXFDICT  
  - Unique: false  
  - Key fields: none  
  - Total fields: 2705  
  - PHI fields: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME  
  A large dictionary or cross-reference table containing many PHI attributes across accounts, MRNs, names, and contact information.

- **HXPLVL1–HXPLVL6**  
  - HXPLVL1: HXFLVL1, key HX1NUM, unique, 36 fields  
  - HXPLVL2: HXFLVL2, key HX2NUM, unique, 39 fields  
  - HXPLVL3: HXFLVL3, key HX3NUM, unique, 39 fields  
  - HXPLVL4: HXFLVL4, key HX4NUM, unique, 39 fields  
  - HXPLVL5: HXFLVL5, key HX5NUM, unique, 42 fields  
  - HXPLVL6: HXFLVL6, key HX6NUM, unique, 155 fields  
  These represent hierarchical level or plan tables used for benefit and status resolution.

- **HXPTABLD**  
  - Record format: XFFTABLD  
  - Unique: false  
  - Key fields: XFDTCD, XFDECD  
  - Total fields: 7  
  - PHI fields: none  
  A compact dictionary of table codes and descriptions used by XFXTABL.

- **HXPXMLD/HXPXMLR**  
  - HXPXMLD: HXFXMLD, key (XMDUSR, XMDSEQ, XMDSQ2), unique, 4 fields  
  - HXPXMLR: HXFXMLR, key (XMRUSR, XMRSEQ, XMRID), unique, 4 fields  
  These tables hold XML detail and header/record metadata keyed by user and sequence, providing a persisted structure for XML messages.

- **OAPIRNK**  
  - Record format: HBFIRNK  
  - Key: BRKLV6, BRKACC, BRKSEQ  
  - Unique: true, 33 fields  
  - PHI: BRKMRN (MRN)  
  Stores ranking or break information associated with accounts and MRNs.

- **OMPMAST**  
  - Record format: HMFMAST  
  - Key: MMPLV6, MMACCT  
  - Unique: true, 149 fields  
  - PHI: MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)  
  Core patient master data.

- **OXPBNFIT**  
  - Record format: XFFBNFIT  
  - Key: XFBUBN, XFBPLN  
  - Unique: true, 34 fields  
  - PHI: XFBTEL (PhoneNumber)  
  Benefit information keyed by benefit and plan.

- **OXPNSTN**  
  - Record format: XFFNSTN  
  - Key: XFNLV6, XFNSST  
  - Unique: true, 23 fields, no PHI  
  Represents status or notification codes at a level.

- **TAPIRNK / TMPMAST / TXPBNFIT / TXPNSTN**  
  These are “ATE TABLE” structures with no declared keys and varying field counts (43, 181, 12, 19). They appear to be staging or external copies of OAPIRNK, OMPMAST, OXPBNFIT, and OXPNSTN respectively.

### 2.2 Logical Files (LF)

Logical files provide keyed and filtered views over PFs:

- **HAPIRNK** (PFILE TAPIRNK) – record format HBFIRNK, key (BRKLV6, BRKACC, BRKSEQ). No select/omit. Used to access the rank data in PF TAPIRNK through the same layout as OAPIRNK.
- **HMLMAST5H** (PFILE TMPMAST) – record format HMFMAST, key (MMPNST, MMADDT, MMADTM). No select/omit. Provides a sequence based on patient status and admission datetime.
- **HXLTABLD / HXLTABLP / HXLTABLS** (PFILE HXPTABLD) – each uses record format XFFTABLD with keys:
  - HXLTABLD: (XFDTCD, XFDMAP)
  - HXLTABLP: (XFDTCD, XFDLDS)
  - HXLTABLS: (XFDTCD, XFDSDS)  
  These files supply different access paths for generic table lookup.
- **HXPBNFIT** (PFILE TXPBNFIT) – record format XFFBNFIT, key (XFBUBN, XFBPLN). Provides an indexed view over benefit staging data.
- **HXPNSTN** (PFILE TXPNSTN) – record format XFFNSTN, key (XFNLV6, XFNSST). Indexed view over status staging data.

## 3. Status and Type Reference Data

Status and type codes are not explicitly enumerated in the schema, but business rules offer hints:

- Date rules (XFXCYMD) enforce ranges (year 1800–2100, month 01–12), suggesting that date status is validated strictly.
- HABADTE rules reference flags like “void/voided” and “INPATIENT/OUTPATIENT” status. These are state codes embedded in fields rather than standalone tables.

From the approved rules set, there is no distinct dictionary of status codes extracted; codes are embedded in conditions.  
**Status/Type Reference Summary:** None identified as dedicated status/type tables beyond the general table/level structures (HXPTABLD, HXPLVL*). Values appear to be inline in program logic.

## 4. Stored Procedure Logic Mappings

Although RPG programs are not SQL stored procedures, we map their call relationships similarly.

### 4.1 Caller-to-Callee Mappings

- **HABADTE**  
  - Calls XFXMRNROL – MRN rollover and profile branching (HXHAPPPRF/HXXAPPPRF).  
  - Calls XFXCNTR (3×) – obtain or increment counters for different workflows.  
  - Calls XFXLDSC – resolve description data from HXPLVL1–HXPLVL6 tables.  
  - Calls XFXCYMD – validate and normalize dates, delegating leap-year logic to XFXLEAP.  
  - Calls XFXGETID – generate or retrieve identifiers, including XML record IDs via HXFXMLR.  
  - Calls XFXTABL – perform generic table lookups against XFFTABLD and its variants.

- **XFXCYMD**  
  - Calls XFXLEAP – encapsulated leap-year decision logic.

- **XFXMRNROL**  
  - Calls HXHAPPPRF (missing) – legacy MRN profile logic.  
  - Calls HXXAPPPRF – SQL-based MRN profile maintenance.

- **HXXAPPPRF**  
  - COPYs HXXCNTRL – control definitions.  
  - COPYs HXXAPPPRFP – additional procedures or prototypes.

- **HABADTE and XFXGETID**  
  - COPY HXXLDA – local data area or shared memory definition.  
  - HABADTE also COPYs HXXLEVEL, HXXXML, CXXXMLP, and CXXXMLC for level and XML definitions.

## 5. Service Class Method Reference

Using hotspot scores, we classify programs into service types.

- **BatchService (high score)**  
  - HABADTE (score 38, fan_out 13, file_ops 6) – primary batch driver for patient transfer and XML generation.

- **WorkflowService (medium score)**  
  - XFXLDSC (score 15) – orchestrates multi-table level lookups; acts as a workflow around level data.  
  - XFXTABL (score 11) – centralizes table lookup logic with multiple file reads.

- **UtilityService (lower score)**  
  - XFXCNTR (score 9, fan_in 3, no file_ops) – pure logic utility for counters.  
  - XFXMRNROL (score 7) – routing and MRN-specific logic with low internal complexity.  
  - HXXAPPPRF (score 7) – SQL-based application profile maintenance utility.  
  - XFXGETID (score 7) – ID retrieval utility touching XML record files.  
  - XFXCYMD (score 5) – date validation utility.  
  - XFXLEAP (score 3) – leap-year check function.  
  - HXHAPPPRF (score 3, missing) – legacy counterpart to HXXAPPPRF.

Lower-score logical files HAPIRNK, HXPNSTN, HXLTABLD, HMLMAST5H, HXLTABLS, HXPBNFIT, HXLTABLP also act as simple data access utilities.

## 6. External Interfaces

High and medium impact gaps indicate external interfaces or missing dependencies:

- **CXXXMLC (COPYBOOK, HIGH)** – defines part of the XML interface for HABADTE. Its absence indicates that part of the XML schema, constants, or control logic is external to this repository.
- **HXHAPPPRF (PROGRAM, MEDIUM)** – likely an older or alternate MRN profile implementation, potentially deployed from another library. XFXMRNROL’s behavior may vary depending on which profile program is active.
- ******HXPXML (FILE, MEDIUM)** – represents one or more XML-related DDS files that are not captured. These likely store configuration or additional XML segments.
- **PRINTER (FILE, MEDIUM)** – a print/spool file definition used for reporting.

Outbound spool or printer interfaces cannot be fully described because the PRINTER file definition is missing.  
**Outbound SPOOL Interfaces:** No outbound SPOOL interfaces identified in the available DDS definitions; reporting exists but its layout is not fully recoverable from the current scan.

## 7. Performance and Security Notes

### 7.1 Performance – Cyclomatic Complexity

From `complexity_per_program`:

- HABADTE – cc=152 (HIGH). This is the main hotspot and poses the greatest performance and maintainability risk. Deep branching and multiple file operations suggest complex business rules and exception handling.
- XFXTABL – cc=9 (LOW band but relatively higher among utilities). Table-driven logic with modest branching.
- XFXCYMD – cc=7 (LOW). Date validation with multiple conditional checks.
- XFXLDSC – cc=5 (LOW). Multi-level lookup logic.
- All other programs (HXXAPPPRF, XFXCNTR, XFXGETID, XFXLEAP, XFXMRNROL, CXXXMLP, HXXAPPPRFP, HXXCNTRL, HXXLDA, HXXLEVEL, HXXXML) – cc=1–3 (LOW). Simple flow with limited branching.

Modernization should focus on decomposing HABADTE into smaller, testable units and applying performance profiling around its file I/O, especially on HXFXMLH/HXFXMLD and HAPTRFR.

### 7.2 Security – PHI Exposure

PHI-tagged fields are concentrated in a few PFs:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT:** XFBTEL (PhoneNumber)

All identified PHI exposure paths in the compact lineage are marked as **ISOLATED**, meaning PHI fields are not further propagated into additional files within the scanned subset. However, since HABADTE generates XML output, it is likely that PHI is included in XML records even if not explicitly traced here. Any modernization must treat HABADTE’s XML interface as PHI-bearing.

### 7.3 Technical Debt Summary

Tech debt is summarized as:

- Total findings: 4
- Total remediation effort: 26.9 hours
- By severity:
  - HIGH: 1
  - MEDIUM: 3
  - LOW: 0

While the numerical effort is moderate, risk is skewed toward the single HIGH-severity finding. Given the complexity profile, that finding is likely associated with HABADTE. Recommended actions:

- Prioritize refactoring HABADTE to reduce complexity and isolate PHI-handling logic.
- Implement structured error handling and logging around XFX* utilities to minimize ripple effects from changes.
- Document the external XML and print interfaces (CXXXMLC, HXPXML*, PRINTER) as part of the security review to ensure PHI is transmitted and stored according to enterprise standards.
