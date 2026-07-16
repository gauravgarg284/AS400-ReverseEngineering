# Low-Level Design (LLD) – HABADTE System

Run ID: 202607161154  
Project: HABADTE (Patient admission and census reporting utilities)

---

## 1. Architecture Overview

### 1.1 Technology Stack Summary

The HABADTE codebase is implemented on IBM i/AS400 using classic RPG and DDS artifacts:

- **RPGLE programs (13):** Core business logic and utilities (HABADTE, XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL, HXX* copy modules).
- **SQLRPGLE programs (2):** HXXAPPPRF and CXXXMLP, combining RPG semantics with embedded SQL for configuration retrieval and XML processing.
- **DDS physical files (19):** Patient master (OMPMAST, TMPMAST), transfer history (HAPTRFR), benefit plans (OXPBNFIT, TXPBNFIT), nursing station tables (OXPNSTN, TXPNSTN), XML tables (HXPXMLD, HXPXMLR), dictionary tables (HXPDICT, HXPLVL*), and printer/auxiliary tables (TAPIRNK, etc.).
- **DDS logical files (7):** Specialized views over physical files (HAPIRNK, HMLMAST5H, HXLTABLD, HXLTABLP, HXLTABLS, HXPBNFIT, HXPNSTN), implementing business-specific access paths.

This architecture follows a layered pattern:
- **Persistence layer:** DDS PF/LF files model business entities and crosswalk tables.
- **Utility/service layer:** XFX* programs provide reusable functions for text, date, organizational level, table lookups, ID generation and MRN/account preferences.
- **Application layer:** HABADTE orchestrates patient selection, census filtering, and output generation (XML and likely spool/print) using the utility layer and persistence layer.

### 1.2 Call Graph Summary

Using dep_edges, the main call relationships are:

- **HABADTE**
  - Calls: XFXMRNROL, XFXCNTR (three times), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL.
  - Copies: HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, CXXXMLC.
  - Reads: HAPTRFR, XFFNSTN, HXFXMLH.
  - Writes/updates: HXFXMLH, HXFXMLD.

- **XFXCYMD**
  - Calls: XFXLEAP for leap-year computation.

- **XFXMRNROL**
  - Calls: HXHAPPPRF (missing) and HXXAPPPRF (present).

- **HXXAPPPRF**
  - Copies: HXXCNTRL, HXXAPPPRFP.

- **XFXGETID**
  - Copies: HXXLDA.
  - Reads: HXFXMLR (declared as HXPXMLR/HXFXMLR).

- **XFXLDSC**
  - Reads: HXFLVL1–HXFLVL6 and declares HXPLVL1–HXPLVL6.

- **XFXTABL**
  - Reads: XFFTABLD and related dictionary tables.

These edges form a directed call graph where HABADTE is the root node, and XFX*/HXX* programs are leaf or intermediate nodes providing specialized services.

### 1.3 Design Patterns

Several design patterns emerge:

- **Service-Facade Pattern:** HABADTE acts as a façade over multiple shared services (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL). Each service encapsulates a cohesive responsibility, isolating complexity.
- **Table-Driven Design:** HXPDICT, HXPTABLD, HXPLVL*, OXPBNFIT/OXPNSTN and related tables drive behavior. XFXLDSC and XFXTABL embody table-driven logic, using configuration in DDS files rather than hard-coded rules.
- **Logical View Modeling:** Logical files like HAPIRNK, HMLMAST5H, HXLTABL* and HXPBNFIT/HXPNSTN implement business-specific views over generalized physical files, mirroring the concept of database views or projections.
- **Export Adapter Pattern:** HABADTE integrates XML export via HXFXMLH/HXFXMLD and copybooks CXXXMLP/CXXXMLC, acting as an adapter between internal census processing and downstream XML consumers.

---

## 2. Database Schema

The data dictionary schema is summarized here for both physical and logical files.

### 2.1 Physical Files (PF)

Each PF subsection lists record format, key structure, uniqueness, field count and PHI indicators.

#### HAPTRFR – Transfer History

- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6 (organizational level), AFACCT (account), AFTRDT (transfer date), AFTRTM (transfer time), AFTYPE (transfer type).
- **Total fields:** 28
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

This table tracks patient movement across nursing stations and room classes. HABADTE reads HAPTRFR to derive current room and organizational context for each patient on the census date.

#### HXPDICT – Dictionary / Crosswalk

- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** none (operates as a generic dictionary table).
- **Total fields:** 2705
- **PHI fields:** CCMRNO (MRN), XFBTEL (PhoneNumber), XCNAME (PatientName), HXRMNO/XFRMNO (RoomNumber), HVACCT/IHACCT (AccountNumber), IMGMRN/HXGMRN/IHMRNO/XMDMRN (MRN), WBDATE (DateOfBirth), ENNAME (PatientName).

HXPDICT is a large, multi-purpose dictionary table hosting patient identifiers, contact details, and various crosswalks. It is a major PHI repository and must be carefully controlled in the modernized architecture.

#### HXPLVL1–HXPLVL6 – Organizational Level Hierarchy

- **Record formats:** HXFLVL1–HXFLVL6
- **Unique key:** Yes (HX1NUM–HX6NUM respectively)
- **Key fields:** level number fields (HX1NUM, HX2NUM, HX3NUM, HX4NUM, HX5NUM, HX6NUM).
- **Total fields:** 36, 39, 39, 39, 42, 155 respectively.
- **PHI fields:** none.

These tables define hierarchical organizational structures (facility, region, campus, building, ward, unit). XFXLDSC reads them to resolve level codes to descriptions, enabling HABADTE to filter and label census data by organization.

#### HXPTABLD – Dictionary Table

- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD (data type code), XFDECD (detail code).
- **Total fields:** 7
- **PHI fields:** none.

HXPTABLD is the base dictionary table for status/type codes. Logical files HXLTABLD, HXLTABLP and HXLTABLS provide different key views over this base table.

#### HXPXMLD / HXPXMLR – XML Detail and Reference

- **HXPXMLD**
  - Record format: HXFXMLD
  - Unique key: Yes
  - Key fields: XMDUSR (user), XMDSEQ (sequence), XMDSQ2 (secondary sequence).
  - Total fields: 4
  - PHI fields: none.

- **HXPXMLR**
  - Record format: HXFXMLR
  - Unique key: Yes
  - Key fields: XMRUSR (user), XMRSEQ (sequence), XMRID (identifier).
  - Total fields: 4
  - PHI fields: none.

These tables store XML payload data and possibly header/metadata. XFXGETID reads HXFXMLR to obtain identifiers for XML records; HABADTE writes XML detail via HXFXMLD.

#### OAPIRNK – Operational APIRNK

- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6 (level), BRKACC (account), BRKSEQ (sequence).
- **Total fields:** 33
- **PHI fields:** BRKMRN (MRN).

#### OMPMAST – Patient Master

- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6 (level), MMACCT (account).
- **Total fields:** 149
- **PHI fields:** MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN).

OMPMAST is a core patient master file. Although HABADTE’s primary runtime view is mediated via TMPMAST/HMLMAST5H, this PF represents the canonical master record and PHI hub.

#### OXPBNFIT / OXPNSTN – Benefit and Nursing Station Tables

- **OXPBNFIT**
  - Record format: XFFBNFIT
  - Unique key: Yes
  - Key fields: XFBUBN (benefit number), XFBPLN (plan).
  - Total fields: 34
  - PHI fields: XFBTEL (PhoneNumber).

- **OXPNSTN**
  - Record format: XFFNSTN
  - Unique key: Yes
  - Key fields: XFNLV6 (level), XFNSST (station).
  - Total fields: 23
  - PHI fields: none.

#### TAPIRNK / TMPMAST / TXPBNFIT / TXPNSTN – Table-Based PFs

- **Record formats:** ATE TABLE
- **Unique key:** No
- **Key fields:** none.
- **Total fields:** TAPIRNK (43), TMPMAST (181), TXPBNFIT (12), TXPNSTN (19).
- **PHI fields:** none flagged in compact schema.

These PFs serve as table-based versions of the operational records (APIRNK, patient master, benefit, nursing station). Logical files use them to provide keyed access paths.

### 2.2 Logical Files (LF)

#### HAPIRNK

- **PFILE:** TAPIRNK
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/omit:** none.

Provides a keyed view over TAPIRNK records matching level, account and sequence, useful for APIRNK-style reporting.

#### HMLMAST5H

- **PFILE:** TMPMAST
- **Record format:** HMFMAST
- **Key fields:** MMPNST (nursing station), MMADDT (admission date), MMADTM (admission time)
- **Select/omit:** none.

This LF tailors the patient master to admissions per nursing station and date/time – crucial for census calculations.

#### HXLTABLD / HXLTABLP / HXLTABLS

All three LFs share PFILE HXPTABLD and record format XFFTABLD but vary by key:

- **HXLTABLD:** keys XFDTCD + XFDMAP (data type + mapping code)
- **HXLTABLP:** keys XFDTCD + XFDLDS (data type + logical description code)
- **HXLTABLS:** keys XFDTCD + XFDSDS (data type + status description code)

These logical views support table-driven resolution of mapping, logical description and status fields via XFXTABL.

#### HXPBNFIT

- **PFILE:** TXPBNFIT
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/omit:** none.

Provides keyed benefit plan view used for patient benefits lookups.

#### HXPNSTN

- **PFILE:** TXPNSTN
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/omit:** none.

Supplies nursing station reference data keyed by facility and station code.

---

## 3. Status and Type Reference Data

Status and type codes are primarily driven by HXPTABLD and accessed via XFXTABL. Approved rules mention several behaviors that depend on these codes, such as application preferences and census filters.

The aggregated rules do not explicitly enumerate status or type codes beyond text descriptions (e.g., voided flag = 'V', I/O indicator = 'O'). Therefore, specific code lists (e.g., all valid status codes) are not available in the summary.

Status and type codes inferred from rules include:
- **Voided account flag:** 'V' (HABADTE excludes accounts with flag='V').
- **Inpatient/outpatient indicator:** 'O' for Outpatient (excluded from census), other values represent inpatient or other modes.
- **Pre-admission file indicator:** 0 for pre-admission (excluded from active census).

Beyond these, XFXTABL likely resolves many more status/type codes from XFFTABLD; however, the exact value set is not enumerated in [SUMMARY].

Conclusion: **None identified as a complete reference list.** Only a small subset of status/type codes is concretely visible via rules.

---

## 4. Stored Procedure Logic Mappings

For the purposes of LLD, each CALL edge is treated as a stored procedure invocation mapping business workflows to specific programs.

### 4.1 HABADTE Caller Group

Caller: **HABADTE**

Callees and call counts:
- XFXMRNROL – 1 call
- XFXCNTR – 3 calls
- XFXLDSC – 1 call
- XFXCYMD – 1 call
- XFXGETID – 1 call
- XFXTABL – 1 call

Behavior mapping:
- **XFXMRNROL:** resolves MRN vs account number preferences using HXHAPPPRF/HXXAPPPRF. Drives display and export identifiers.
- **XFXCNTR:** performs text centering for report headings or labels, applied in multiple contexts.
- **XFXLDSC:** looks up organizational level descriptions from HXPLVL* tables, enabling HABADTE to filter on level and enrich output labels.
- **XFXCYMD:** validates census dates and patient dates (admission/discharge) against calendar rules.
- **XFXGETID:** retrieves XML record identifiers from HXFXMLR so HABADTE can write consistent XML detail records.
- **XFXTABL:** resolves table-driven status and type codes via HXPTABLD/XFFTABL*.

### 4.2 Utility Caller Groups

Caller: **XFXCYMD**

- Callee: XFXLEAP – 1 call.
- Mapping: XFXCYMD delegates leap-year-specific validation to XFXLEAP, isolating leap-year logic.

Caller: **XFXMRNROL**

- Callees: HXHAPPPRF, HXXAPPPRF – 1 call each.
- Mapping: MRN roll logic uses configuration/preferences from both programs (one missing). This pair defines application preference and facility-specific overrides.

Caller: **HXXAPPPRF**

- Callees via COPY: HXXCNTRL, HXXAPPPRFP.
- Mapping: HXXAPPPRF composes its internal structures and SQL definitions from shared control and preference copybooks.

Caller: **XFXGETID**

- Callee via COPY: HXXLDA.
- Mapping: ID retrieval uses layout and data area definitions provided by HXXLDA.

These mappings should be preserved in the modernized environment, with equivalent stored procedures or service methods representing each relationship.

---

## 5. Service Class Method Reference

Based on hotspot scores, programs are categorized into service classes.

### 5.1 BatchService

High hotspot scores indicate batch-like orchestrators:

- **HABADTE (score 38): BatchService**
  - Role: Batch census report generator.
  - Behavior: Iterates through patient master/transfer history, applies complex filters (pre-admission, voided, outpatient, census date admission/discharge checks), resolves room and nursing station details, and writes XML/spool outputs.

### 5.2 WorkflowService

Moderate scores reflect workflow-centric services:

- **XFXLDSC (score 15): WorkflowService**
  - Role: Organizational level resolution. Drives workflows that depend on hierarchical level validation and description lookup.

- **XFXTABL (score 11): WorkflowService**
  - Role: Table-driven status/type resolution. Used wherever codes must be translated to descriptions or behaviors.

- **XFXMRNROL (score 7): WorkflowService**
  - Role: MRN/account preference workflow, bridging configuration tables via HXHAPPPRF/HXXAPPPRF.

- **HXXAPPPRF (score 7): WorkflowService**
  - Role: Application preference access workflow with embedded SQL.

- **XFXGETID (score 7): WorkflowService**
  - Role: Identifier retrieval workflow for XML records.

### 5.3 UtilityService

Lower scores and limited file IO indicate pure utility services:

- **XFXCNTR (score 9): UtilityService** – text centering logic, reusable everywhere.
- **XFXCYMD (score 5): UtilityService** – date validation.
- **XFXLEAP (score 3): UtilityService** – leap-year detection.

### 5.4 Supporting Logical Views

Logical files like HMLMAST5H, HXLTABLD, HXLTABLP, HXLTABLS, HAPIRNK, HXPBNFIT and HXPNSTN support service classes by providing the required indexed views for each workflow (patient selection, dictionary lookups, benefits and nursing stations).

---

## 6. External Interfaces

High/medium gaps reflect external-facing interfaces or unmodeled integration points.

### 6.1 Inbound External Interfaces (from Gaps)

- **CXXXMLC (COPYBOOK, impact HIGH)** – forms part of the XML interface layer. It likely defines inbound or outbound XML header/control fields for HABADTE’s export.

- **HXHAPPPRF (PROGRAM, impact MEDIUM)** – external preference/ configuration provider invoked by XFXMRNROL. May represent an older or facility-specific module outside the core HABADTE project.

- ******HXPXML (FILE, impact MEDIUM)** – appears as a placeholder for XML-related configuration or data, possibly representing a file in another subsystem.

These are treated as inbound interfaces in the LLD because they supply configuration or structural definitions used by the core HABADTE workflows.

### 6.2 Outbound Interfaces

HABADTE writes to XML tables HXFXMLH/HXFXMLD and likely to a PRINTER file for spool routing. However, the summary does not provide an explicit description for outbound spool interfaces such as printer queues or external messaging.

Conclusion: **No outbound SPOOL interfaces identified.** Outbound behavior is inferred but not fully described in the aggregated context.

---

## 7. Performance and Security Notes

### 7.1 Performance Considerations

Cyclomatic complexity per program:

- **HABADTE:** cc=152 (HIGH) – the main hotspot. Performance considerations include:
  - Large iteration over TMPMAST/HMLMAST5H and HAPTRFR.
  - Multiple joins/lookups via XFX utilities and dictionary tables.
  - XML writing to HXFXMLH/HXFXMLD.

- **XFXTABL:** cc=9 (LOW band but relatively higher among utilities) – multiple branch conditions tied to different table lookups.

- Other utilities (**XFXLDSC, XFXCYMD, XFXCNTR, XFXGETID, XFXMRNROL, HXXAPPPRF**) have low complexity numbers but can influence performance through their file IO patterns.

Performance recommendations:
- Index and optimize key paths for TMPMAST/HMLMAST5H, HAPTRFR, HXFLVL*, HXPTABLD and HXFXML* tables in the modernized database.
- Consider decomposing HABADTE into smaller services or steps (e.g., data selection, enrichment, export) to reduce per-service complexity.

### 7.2 Security and PHI Exposure

PHI flagged fields:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN).
- **HXPDICT:** CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN (MRN), XFBTEL (PhoneNumber), XCNAME/ENNAME (PatientName), HXRMNO/XFRMNO (RoomNumber), HVACCT/IHACCT (AccountNumber), WBDATE (DateOfBirth).
- **OAPIRNK:** BRKMRN (MRN).
- **OMPMAST:** MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN).
- **OXPBNFIT:** XFBTEL (PhoneNumber).

Compact data lineage shows PHI exposures as mostly **isolated** within these tables, with no long, multi-hop transformations identified in the summary.

Security notes:
- MRN and account numbers appear in both operational and table-based masters; access control should be enforced at service-layer level (HABADTE, XFXMRNROL, preference services).
- Phone numbers and names in HXPDICT/OMPMAST must be masked or controlled when building new UI or API layers.
- SSN in OMPMAST (MMPSSN) demands stricter controls and auditing.

### 7.3 Tech Debt Summary

Tech debt summary:
- **Total findings:** 4
- **Total remediation hours:** 26.9
- **By severity:** HIGH=1, MEDIUM=3, LOW=0.

While details of each finding are in upstream artifacts, the severity aligns with observed hotspots:
- High severity likely linked to HABADTE’s complexity and XML/printing gaps.
- Medium severities may correspond to missing preference and XML components or schema entanglements.

Modernization effort should allocate time to:
- Refactor HABADTE into manageable components.
- Replace missing/opaque interfaces (CXXXMLC, HXHAPPPRF, ****HXPXML, PRINTER) with explicit, well-documented APIs.
- Implement robust PHI masking and auditing around master and dictionary tables.
