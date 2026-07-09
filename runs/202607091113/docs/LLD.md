# Low-Level Design (LLD) – HABADTE Project

## Section 1 – Architecture Overview

### 1.1 Technology Stack and Member Types

The HABADTE project is an AS400 (IBM i) application composed of RPGLE/SQLRPGLE programs and DDS-defined physical/logical files.

- **RPGLE**: 13 programs, primarily implementing business logic and batch orchestration.
- **SQLRPGLE**: 2 programs (HXXAPPPRF, CXXXMLP), providing database-oriented or XML-related operations.
- **DDS Physical Files (PF)**: 19 files, modeling persistent data structures for patient accounts, benefits, level tables, and XML staging.
- **DDS Logical Files (LF)**: 7 files, providing alternate key paths and filtered views on PFs.

The total of 41 members and 10,288 lines indicates a compact but non-trivial codebase, with one very large dictionary file (HXPDICT) containing over 6,000 lines.

### 1.2 High-Level Call Graph

The call/copy/reference edges describe a star-shaped architecture centered on the HABADTE program:

- **Top-level driver**: HABADTE (RPGLE, domain PATIENT_MANAGEMENT) orchestrates transaction and export processing.
- **Utility programs**:
  - XFXMRNROL – MRN rollover logic.
  - XFXCNTR – field formatting and counter management.
  - XFXLDSC – level/descriptor lookup from level PFs.
  - XFXCYMD – date validation and calendar constraints.
  - XFXGETID – identifier retrieval and XML record access.
  - XFXTABL – generalized table lookup engine.
  - XFXLEAP – leap-year helper used by XFXCYMD.
- **SQL bridge**:
  - HXXAPPPRF (SQLRPGLE) – accesses application profile tables through SQL and includes control modules HXXCNTRL/HXXAPPPRFP via COPY.

Copy dependencies from HABADTE onto HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, and CXXXMLC show a layered design where common data structures and constants are centralized in copy members.

### 1.3 Design Patterns

- **Controller + Services** – HABADTE acts as a controller invoking specialized services (XFX* programs). Each service focuses on a narrow responsibility (dates, levels, tables, MRNs), consistent with a service-class architecture.
- **Table-driven configuration** – Many decisions are driven by table PFs with multiple logical overlays, indicating a configuration-driven approach rather than hard-coded branching.
- **XML staging pipeline** – Specialized PFs (HXPXMLD, HXPXMLR, unresolved ****HXPXML, HXFXMLH/HXFXMLD) and copy members indicate a structured pipeline for constructing XML payloads.

## Section 2 – Database Schema

This section summarizes PF/LF structures from the compact data dictionary schema.

### 2.1 Physical Files (PF)

Each PF subsection lists record format, uniqueness, key structure, and PHI flags.

#### HAPTRFR
- Record format: HAFTRFR
- Unique: true
- Key fields: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- Total fields: 28
- PHI fields: AFACCT (AccountNumber), AFMRNO (MRN)
- Purpose: transaction transfer records tied to level 6, account, date/time, and type, capturing financial or encounter-level transfers.

#### HXPDICT
- Record format: HXFDICT
- Unique: false
- Key fields: (none listed in the compact schema)
- Total fields: 2705
- PHI fields include: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME.
- Purpose: large dictionary of cross-reference data, including MRN, account, contact details, and other patient or encounter identifiers.

#### HXPLVL1 – HXPLVL6
- Record formats: HXFLVL1 … HXFLVL6
- Unique: true
- Keys:
  - HXPLVL1: key HX1NUM
  - HXPLVL2: key HX2NUM
  - HXPLVL3: key HX3NUM
  - HXPLVL4: key HX4NUM
  - HXPLVL5: key HX5NUM
  - HXPLVL6: key HX6NUM
- Total fields: 36–155 depending on level.
- PHI fields: none flagged.
- Purpose: hierarchical level tables representing codes and descriptions used throughout the application; accessed heavily by XFXLDSC.

#### HXPTABLD
- Record format: XFFTABLD
- Unique: false
- Keys: XFDTCD, XFDECD
- Total fields: 7
- PHI fields: none
- Purpose: generalized code/description table; underlies multiple logical files providing different ordering and view semantics.

#### HXPXMLD and HXPXMLR
- HXPXMLD
  - Record format: HXFXMLD
  - Unique: true
  - Keys: XMDUSR, XMDSEQ, XMDSQ2
  - Total fields: 4
- HXPXMLR
  - Record format: HXFXMLR
  - Unique: true
  - Keys: XMRUSR, XMRSEQ, XMRID
  - Total fields: 4
- Purpose: XML detail and header records for outbound/inbound payloads; used in conjunction with HABADTE and XFXGETID.

#### OAPIRNK
- Record format: HBFIRNK
- Unique: true
- Keys: BRKLV6, BRKACC, BRKSEQ
- Total fields: 33
- PHI fields: BRKMRN (MRN)
- Purpose: broker or break records tied to MRN and account, possibly capturing ranked or sequenced transaction history.

#### OMPMAST
- Record format: HMFMAST
- Unique: true
- Keys: MMPLV6, MMACCT
- Total fields: 149
- PHI fields: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- Purpose: primary patient or member master table containing identifiers, names, SSNs, and other demographic/financial attributes.

#### OXPBNFIT
- Record format: XFFBNFIT
- Unique: true
- Keys: XFBUBN, XFBPLN
- Total fields: 34
- PHI fields: XFBTEL (PhoneNumber)
- Purpose: benefit table keyed by benefit and plan identifiers, with associated contact or notification details.

#### OXPNSTN
- Record format: XFFNSTN
- Unique: true
- Keys: XFNLV6, XFNSST
- Total fields: 23
- Purpose: station or state reference data keyed by level and state code.

#### TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN
- Record formats: "ATE TABLE" (compacted naming, but representing PFs).
- Unique: false
- Keys: not specified in schema (none listed).
- Total fields:
  - TAPIRNK: 43
  - TMPMAST: 181
  - TXPBNFIT: 12
  - TXPNSTN: 19
- Purpose: table PFs representing application-level equivalents to OAPIRNK/OMPMAST/OXPBNFIT/OXPNSTN.

### 2.2 Logical Files (LF)

Logical files provide alternate access paths over PFs.

#### HAPIRNK
- PFILE: TAPIRNK
- Record format: HBFIRNK
- Key fields: BRKLV6, BRKACC, BRKSEQ
- Select/omit: none
- Purpose: indexed view over TAPIRNK providing ordered break records.

#### HMLMAST5H
- PFILE: TMPMAST
- Record format: HMFMAST
- Key fields: MMPNST, MMADDT, MMADTM
- Select/omit: none
- Purpose: time-sequenced view of master records by posting state and add date/time.

#### HXLTABLD / HXLTABLP / HXLTABLS
- PFILE: HXPTABLD
- Record format: XFFTABLD
- Keys:
  - HXLTABLD: XFDTCD, XFDMAP
  - HXLTABLP: XFDTCD, XFDLDS
  - HXLTABLS: XFDTCD, XFDSDS
- Select/omit: none
- Purpose: multiple logical overlays providing different sort orders or projections over the same code/description table.

#### HXPBNFIT
- PFILE: TXPBNFIT
- Record format: XFFBNFIT
- Key fields: XFBUBN, XFBPLN
- Select/omit: none
- Purpose: logical view over benefit PF for efficient plan/benefit queries.

#### HXPNSTN
- PFILE: TXPNSTN
- Record format: XFFNSTN
- Key fields: XFNLV6, XFNSST
- Select/omit: none
- Purpose: logical view over station/state PF keyed by level and state.

## Section 3 – Status and Type Reference Data

Approved business rules point to implicit status and type codes embedded in logic:

- **HABADTE**
  - Rule BR-018: When a "flag indicator" equals a void/voided status, processing branches to SKIP.
  - Rule BR-019: When the inpatient/outpatient flag equals outpatient, processing branches to SKIP.
  - Rule BR-017: When a file indicator equals zero, processing branches to SKIP.

These rules imply presence of status codes for void, inpatient/outpatient classifications, and file indicators. Specific code values (e.g., 'V', 'O') are not extracted in the compact summary, so they must be inferred from source if needed.

Other approved rules from XFXCYMD and XFXTABL describe date and indicator behavior but do not name explicit status codes. Therefore, dedicated status and type reference tables are likely encoded in HXPTABLD/HXPLVLx but are not explicitly enumerated here.

If additional status/type code catalogs are required, they should be derived from the content of HXPTABLD and associated text columns.

## Section 4 – Stored Procedure Logic Mappings

Although this is not a SQL stored procedure environment, we can treat CALL chains as procedural mappings.

### 4.1 Caller-to-Callee Map

- **HABADTE**
  - Calls XFXMRNROL
  - Calls XFXCNTR (three distinct sites)
  - Calls XFXLDSC
  - Calls XFXCYMD
  - Calls XFXGETID
  - Calls XFXTABL

- **XFXCYMD**
  - Calls XFXLEAP

- **XFXMRNROL**
  - Calls HXHAPPPRF (missing program)
  - Calls HXXAPPPRF (SQLRPGLE)

- **HXXAPPPRF**
  - COPY dependencies on HXXCNTRL and HXXAPPPRFP (control and profile modules).

- **XFXGETID**
  - COPY dependency on HXXLDA; uses HXPXMLR/HXFXMLR for identifier lookup.

- **HABADTE**
  - COPY dependencies on HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC.

### 4.2 Mapping Semantics

- HABADTE orchestrates patient transactions, delegating:
  - MRN rollover to XFXMRNROL and further to HXXAPPPRF/HXHAPPPRF.
  - Field formatting and counters to XFXCNTR.
  - Level validation and descriptions to XFXLDSC.
  - Date validation to XFXCYMD (including leap-year logic via XFXLEAP).
  - Identifier allocation and XML header retrieval to XFXGETID.
  - Code/description lookups to XFXTABL.

These mappings define how a future service-oriented or microservice architecture should be decomposed: each XFX* program can be mapped to a discrete service endpoint.

## Section 5 – Service Class Method Reference

Using hotspot scores as an indicator of service class:

- **BatchService (high score)**
  - HABADTE (score 38, fan_out 13, file_ops 6)
    - Acts as a BatchService orchestrating end-to-end patient transaction processing and XML output generation.

- **WorkflowService (medium score)**
  - XFXLDSC (score 15, fan_in 1, file_ops 6)
    - LevelDescriptorService – resolves level codes and their descriptions.
  - XFXTABL (score 11, fan_in 1, file_ops 4)
    - TableLookupService – performs generalized code/description lookups from tabular PFs.
  - XFXMRNROL (score 7, fan_in 1, fan_out 2)
    - MRNRolloverService – coordinates MRN changes with profile programs.
  - XFXGETID (score 7, fan_in 1, file_ops 1)
    - IdentifierService – retrieves or allocates identifiers via XML record tables.
  - HXXAPPPRF (score 7, fan_in 1, fan_out 2)
    - AppProfileService – SQL-based profile management.

- **UtilityService (low score)**
  - XFXCNTR (score 9, fan_in 3, file_ops 0)
    - Formatting/Counter utility.
  - XFXCYMD (score 5, fan_in 1, fan_out 1)
    - DateValidation utility.
  - XFXLEAP (score 3, fan_in 1, file_ops 0)
    - LeapYear utility.
  - Various PF/LF-based hotspots with score 2 (HMLMAST5H, HXLTABLD, HXLTABLS, HXLTABLP, HXPBNFIT, HXPNSTN) can be treated as data access utility components.

## Section 6 – External Interfaces

High/medium impact gaps characterize inbound interfaces or external dependencies:

- **CXXXMLC (COPYBOOK, HIGH)** – likely defines XML structures for external messaging. Its reference from HABAD* indicates that the application exchanges XML payloads with external systems, possibly a downstream integration engine or repository.
- **HXHAPPPRF (PROGRAM, MEDIUM)** – unresolved application profile program; indicates external or unscanned module responsible for profile enforcement.
- ******HXPXML (FILE, MEDIUM)** – missing file in the XML family, implying additional XML staging or audit tables.
- **PRINTER (FILE, MEDIUM)** – unresolved printer configuration file, representing an external dependency on system-level printing or spool configuration.

No explicit outbound spool interfaces are present in the available edges; therefore:  
**No outbound SPOOL interfaces identified.**

## Section 7 – Performance and Security Notes

### 7.1 Performance Considerations

Cyclomatic complexity and hotspot metrics highlight performance and maintainability risks:

- HABADTE has a cyclomatic complexity of 152 (HIGH band) and interacts with multiple files and services. This concentration of branching and I/O suggests:
  - Higher risk of regression when modifying logic.
  - Potential performance bottlenecks if run as a long batch or high-volume job.
- The XFX* utilities have LOW complexity and clearly defined responsibilities; they are good candidates for reuse without major refactoring, but their integration points should be carefully profiled.

### 7.2 PHI Exposure

The following PHI fields are present in the schema:

- **HAPTRFR**: AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT**: CCMRNO (MRN), XFBTEL (PhoneNumber), XCNAME (PatientName), HXRMNO/XFRMNO (RoomNumber), HVACCT (AccountNumber), IMGMRN/HXGMRN/IHMRNO/XMDMRN (MRN), IHACCT (AccountNumber), WBDATE (DateOfBirth), ENNAME (PatientName)
- **OAPIRNK**: BRKMRN (MRN)
- **OMPMAST**: MMMRNO, MMMMRN (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN)
- **OXPBNFIT**: XFBTEL (PhoneNumber)

Most PHI-bearing PFs are master or dictionary tables. Current lineage analysis shows these PHI fields as **ISOLATED** in the available flow paths, meaning no clear propagation path into external systems has been detected in the scanned members. However, the XML pipeline and missing XML/printer components could still transmit PHI; these should be treated as potential exposure points.

### 7.3 Tech Debt and Risk

Tech debt summary:

- Total findings: 4
- Total estimated remediation effort: 26.9 hours
- By severity: 1 HIGH, 3 MEDIUM, 0 LOW

Most risk is concentrated in the HABADTE program due to its size and complexity. Recommended mitigations include:

- Decomposing HABADTE into smaller modules aligned with the service classes outlined above.
- Isolating PHI-heavy operations into well-defined components with clear access controls and auditing.
- Reconstructing missing XML/printer artifacts or replacing them with modern integration and output channels.
