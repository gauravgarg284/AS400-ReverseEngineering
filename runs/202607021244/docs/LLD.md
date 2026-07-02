# Low-Level Design (LLD) - HABADTE Project

## 1. Architecture Overview

The HABADTE project is an IBM i (AS/400) application centred on a single high-complexity RPGLE program, HABADTE, that produces an "Active Accounts by Admit Date and Time" report and corresponding XML output. The technology stack comprises:

- **DDS Physical Files (PF):** 15 files including patient master (OMPMAST), transfer history (HAPTRFR), dictionary/lookup structures (HXPDICT, HXPLVL*), benefits tables (OXPBNFIT), and nursing station definitions (OXPNSTN).
- **DDS Logical Files (LF):** 7 files such as HAPIRNK, HMLMAST5H, HXLTABLD/P/S, HXPBNFIT, and HXPNSTN, providing keyed and filtered views.
- **RPGLE Programs:** 13 components including HABADTE and utilities XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXLEAP, XFXMRNROL, XFXTABL, and HXX* helpers.
- **SQLRPGLE Programs:** HXXAPPPRF and CXXXMLP for database access and XML-related processing.

### 1.1 Call Graph Summary

The core call graph, derived from dependency edges, is:

- HABADTE → XFXMRNROL (MRN roll-up configuration)
- HABADTE → XFXCNTR (heading centering)
- HABADTE → XFXLDSC (level descriptions)
- HABADTE → XFXCYMD (date conversion)
- HABADTE → XFXGETID (XML fragment retrieval)
- HABADTE → XFXTABL (table-based code translation)
- XFXCYMD → XFXLEAP (leap-year logic)
- XFXMRNROL → HXHAPPPRF, HXXAPPPRF (profile management)

HABADTE also includes COPY references to HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, and missing CXXXMLC to gain field definitions, level control, and XML scaffolding.

### 1.2 Design Patterns Observed

The design employs several recurring patterns:

- **Utility Service Pattern:** Small, low-complexity programs (XFXCNTR, XFXCYMD, XFXGETID, XFXLDSC, XFXTABL, XFXLEAP, XFXMRNROL) act as stateless services called by HABADTE and potentially other drivers. Each encapsulates a single concern such as centering text, converting dates, reading XML IDs, or table lookups.
- **Table-Driven Configuration:** Files HXPTABLD, HXPLVL*, HXPDICT, OXPBNFIT, OXPNSTN, and logical files built over them represent configuration data (levels, codes, benefits, stations). XFXTABL is the generic interpreter of these configuration tables.
- **Report-Driven XML Generation:** HABADTE blends traditional printer file output with structured XML generation through HXPXMLR/HXPXMLD. XML tags are constructed via getidsr and writexmld subroutines, using layouts from CXXXMLP and missing CXXXMLC.

## 2. Database Schema

The compact data dictionary describes all PF and LF structures used by the project.

### 2.1 Physical Files (PF)

Each PF is summarized with key, uniqueness, field counts, and PHI flags.

#### HAPTRFR (PF)
- Record format: HAFTRFR
- Unique key: yes
- Key fields: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- Total fields: 28
- PHI-sensitive fields: AFACCT (AccountNumber), AFMRNO (MRN)

This file stores patient transfer records. Keys combine level (corporate structure), account, transfer date/time, and type. PHI is present via account and MRN identifiers.

#### HXPDICT (PF)
- Record format: HXFDICT
- Unique key: no
- Key fields: none specified
- Total fields: 2705
- PHI-sensitive fields: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

HXPDICT functions as a central dictionary, holding a wide variety of fields including MRNs, account numbers, names, phone numbers, room numbers, and dates of birth. It is a critical PHI store.

#### HXPLVL1–HXPLVL6 (PFs)
- HXPLVL1: HXFLVL1, key HX1NUM, unique, 36 fields
- HXPLVL2: HXFLVL2, key HX2NUM, unique, 39 fields
- HXPLVL3: HXFLVL3, key HX3NUM, unique, 39 fields
- HXPLVL4: HXFLVL4, key HX4NUM, unique, 39 fields
- HXPLVL5: HXFLVL5, key HX5NUM, unique, 42 fields
- HXPLVL6: HXFLVL6, key HX6NUM, unique, 155 fields

These level tables support corporate hierarchy (e.g., enterprise, region, facility). XFXLDSC reads HXFLVL* to produce level descriptions based on codes stored in HABADTE’s request parameters.

#### HXPTABLD (PF)
- Record format: XFFTABLD
- Key fields: XFDTCD, XFDECD
- Unique: no
- Total fields: 7
- PHI fields: none

HXPTABLD is a generic table-defining structure. Logical files HXLTABLD/P/S slice it into different views, and XFXTABL reads XFFTABLD to map codes (e.g., room class) to descriptions and flags.

#### HXPXMLD, HXPXMLR (PFs)
- HXPXMLD: HXFXMLD, key XMDUSR, XMDSEQ, XMDSQ2, unique, 4 fields
- HXPXMLR: HXFXMLR, key XMRUSR, XMRSEQ, XMRID, unique, 4 fields

These files store XML report detail and header records. HABADTE and XFXGETID use them to coordinate XML sequence numbers and payload fragments.

#### OAPIRNK (PF)
- Record format: HBFIRNK
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Unique: yes
- Total fields: 33
- PHI fields: BRKMRN (MRN)

#### OMPMAST (PF)
- Record format: HMFMAST
- Key fields: MMPLV6, MMACCT
- Unique: yes
- Total fields: 149
- PHI fields: MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)

OMPMAST is the central patient account master, containing identifiers, names, and SSNs.

#### OXPBNFIT (PF)
- Record format: XFFBNFIT
- Key fields: XFBUBN, XFBPLN
- Unique: yes
- Total fields: 34
- PHI fields: XFBTEL (PhoneNumber)

#### OXPNSTN (PF)
- Record format: XFFNSTN
- Key fields: XFNLV6, XFNSST
- Unique: yes
- Total fields: 23
- PHI fields: none

### 2.2 Logical Files (LF)

#### HAPIRNK (LF)
- Based on PF: TAPIRNK (missing PF)
- Record format: HBFIRNK
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Select/omit: none

HAPIRNK positions transfer data by level/account/sequence over TAPIRNK. The base PF is missing in this snapshot.

#### HMLMAST5H (LF)
- Based on PF: TMPMAST (missing PF)
- Record format: HMFMAST
- Key fields: MMPNST, MMADDT, MMADTM
- Select/omit: none

Provides a view over patient master data keyed by nursing station and admit date/time, used by HABADTE’s census logic.

#### HXLTABLD / HXLTABLP / HXLTABLS (LFs)
- PFILE: HXPTABLD
- Record format: XFFTABLD
- Key fields:
  - HXLTABLD: XFDTCD, XFDMAP
  - HXLTABLP: XFDTCD, XFDLDS
  - HXLTABLS: XFDTCD, XFDSDS
- Select/omit: none

These logical files present different slices over the same dictionary table, exposing mapping, long description, and short description fields.

#### HXPBNFIT (LF)
- PFILE: TXPBNFIT (missing PF)
- Record format: XFFBNFIT
- Key fields: XFBUBN, XFBPLN

#### HXPNSTN (LF)
- PFILE: TXPNSTN (missing PF)
- Record format: XFFNSTN
- Key fields: XFNLV6, XFNSST

## 3. Status and Type Reference Data

Approved business rules primarily express validation and exit conditions. Status and type code references include:

- From XFXCNTR:
  - "When X equals zero, branch to 'EXIT'" (BR-001).
  - "When X equals 40, branch to 'EXIT'" (BR-002).

- From XFXCYMD:
  - Date validity rules such as "When VYY is less than 1800, branch to 'EXIT'" and "When VYY is greater than 2100, branch to 'EXIT'".
  - Month range checks (VMM < 01 or VMM > 12).

- From HABADTE:
  - File indicator, void flag, and inpatient/outpatient flag causing branch to SKIP.

These rules imply that X and VMM/VYY/VDD are status or numeric control fields representing positions, month/year values, or other coded settings. However, explicit enumerated status lookup tables are not present in the approved_rules text. Therefore, detailed status code documentation beyond these numeric thresholds is not available.

**Conclusion for this section:** None identified beyond the numeric and flag-based checks captured in the rules above.

## 4. Stored Procedure Logic Mappings

CALL edges grouped by caller:

### 4.1 HABADTE

- Calls XFXMRNROL once to determine MRN roll-up behaviour.
- Calls XFXCNTR three times to center report heading, hospital title, and date strings.
- Calls XFXLDSC once to retrieve hospital level descriptions.
- Calls XFXCYMD once to convert census date and once indirectly via xml timestamp sections.
- Calls XFXGETID once to retrieve XML fragments identified by getid values.
- Calls XFXTABL once (roomClassDtl) to resolve room class descriptions and leave-account flags.

Within HABADTE, stored procedure-style patterns exist where each call is followed by parameter passing and an EXSR (subroutine) that handles output, mirroring service method calls in modern architectures.

### 4.2 XFXCYMD

- Calls XFXLEAP to determine leap-year properties before applying date rules.

### 4.3 XFXMRNROL

- Calls HXHAPPPRF and HXXAPPPRF to read MRN roll-up configuration. These programs are treated like stored procedures where the call returns a single flag indicating whether roll-up is active.

No additional callers are recorded for XFXCNTR, XFXLDSC, XFXTABL, XFXGETID, XFXLEAP, but their roles as utilities imply they are shared across the broader application beyond the HABADTE slice.

## 5. Service Class Method Reference

Dep hotspot scores allow classification into service-style groupings:

- **BatchService (high score ≥ 20):**
  - HABADTE (score 38) – main batch/report service orchestrating file reads and XML/printer output.

- **WorkflowService (medium score 10–19):**
  - XFXLDSC (score 15) – workflow for translating level codes to descriptions using HXPLVL* and HXFLVL* files.
  - XFXTABL (score 11) – workflow for reading generic code tables and returning mapped descriptions and flags.

- **UtilityService (low score < 10):**
  - XFXCNTR (score 9) – text-centering utility.
  - XFXMRNROL (score 7) – MRN roll-up decision utility.
  - HXXAPPPRF (score 7) – application profile utility.
  - XFXGETID (score 7) – XML-id utility reading HXFXMLR.
  - XFXCYMD (score 5) – date conversion utility.
  - XFXLEAP (score 3) – leap-year calculation utility.
  - HXHAPPPRF (score 3) – profile utility used by XFXMRNROL.
  - Several logical files (HMLMAST5H, HXLTABLS, HXPNSTN, HXPBNFIT, HAPIRNK, HXLTABLD, HXLTABLP) appear as hotspots with low scores due to file_ops; these are data services backing the utility programs.

This categorization provides a clear map for refactoring: HABADTE becomes a batch service, XFXLDSC and XFXTABL become workflow/microservices, and the others form a shared utilities library.

## 6. External Interfaces

High-impact gaps suggest external interfaces not fully present in this reverse-engineered snapshot:

- CXXXMLC (COPYBOOK, HIGH impact, referenced by HABADTE) – defines XML layout or constants used by getidsr and writexmld. It behaves like an interface to the generic XML framework (CXXXMLP and CXXXMLC together).
- TAPIRNK (FILE, HIGH impact, referenced by HAPIRNK) – represents external transfer history storage; logical file HAPIRNK acts as the local view.
- TMPMAST (FILE, HIGH impact, referenced by HMLMAST5H) – external patient master storage; logical file HMLMAST5H is the view used by HABADTE.
- TXPBNFIT (FILE, HIGH impact, referenced by HXPBNFIT) – external benefits table storage.
- TXPNSTN (FILE, HIGH impact, referenced by HXPNSTN) – external nursing station codes storage.

These PFs are conceptually part of the same database but absent from the analyzed library, indicating reliance on other libraries or environments.

No explicit outbound SPOOL or external messaging interfaces are recorded in the aggregated dependency data beyond traditional printer file output and XML file writes. Therefore:

**No outbound SPOOL interfaces identified.**

## 7. Performance and Security Notes

### 7.1 Performance (Cyclomatic Complexity)

Complexity per program:

- HABADTE: cc 152, band HIGH – main performance and maintainability risk. The code contains numerous conditional branches for level filtering, date range checks, room/nursing station retrieval, XML tag orchestration, and counter updates (totalActive, totalLeave, total). This complexity suggests high testing effort for refactoring and risk of subtle defects.
- XFXTABL: cc 9, band LOW – relatively simple, but a core dependency for translating codes.
- XFXCYMD: cc 7, band LOW – encapsulates multiple date validation rules.
- XFXLDSC: cc 5, band LOW – moderate complexity for level description resolution.
- All other utilities (XFXCNTR, XFXGETID, XFXLEAP, XFXMRNROL, HXXAPPPRF, HXX* helpers, CXXXMLP) have cc 1–3 and band LOW.

The architectural recommendation is to decompose HABADTE into smaller routines or external services (for admission filtering, room/nursing station enrichment, XML header/detail generation) while leaving low-complexity utilities largely intact.

### 7.2 Security and PHI Exposure

PHI-flagged fields and categories:

- HAPTRFR: AFACCT (AccountNumber), AFMRNO (MRN)
- HXPDICT: CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN (MRN); XCNAME, ENNAME (PatientName); XFBTEL (PhoneNumber); HXRMNO, XFRMNO (RoomNumber); HVACCT, IHACCT (AccountNumber); WBDATE (DateOfBirth)
- OAPIRNK: BRKMRN (MRN)
- OMPMAST: MMMRNO, MMMMRN (MRN); MMACCT (AccountNumber); MMNAME (PatientName); MMPSSN (SSN)
- OXPBNFIT: XFBTEL (PhoneNumber)

Data lineage shows that most PHI fields have exposure level "ISOLATED" within the HABADTE slice, meaning they are not broadly propagated across many programs. However, HABADTE itself reads transfer and nursing station data and produces a report listing account numbers, names, room numbers, admit dates, and times. XML output also embeds user ID (LDAUSR), hospital title, census date, and timestamp.

Tech debt summary:

- Total findings: 4
- Total remediation hours: 26.9
- Severity: 1 HIGH, 3 MEDIUM

The high-severity finding likely corresponds to HABADTE’s complexity and combined PHI exposure via reporting. Medium findings relate to incomplete XML framework, missing base files, and legacy table structures.

From a security standpoint, modernization should ensure:

- Adequate access controls on OMPMAST, HXPDICT, HAPTRFR, OAPIRNK, and OXPBNFIT.
- Encryption or tokenization for SSN (MMPSSN) and MRNs where feasible.
- Review of XML output (HXPXMLD/HXPXMLR) to avoid unintended PHI leakage beyond intended consumers.
