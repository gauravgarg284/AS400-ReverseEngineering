# AS400 Code Analysis Report – HABADTE Subsystem

## Section 1 – Component Inventory

The HABADTE application comprises 41 source members spanning database definitions (DDS), RPG business logic, and SQLRPGLE service routines. The inventory below is derived from the aggregated source manifest and represents the full technical footprint in the analysed library.

### 1.1 Member Inventory Table

| Member Name | Type    | Subsystem  | Lines |
|------------|---------|-----------|-------|
| HAPIRNK    | DDS_LF  | HAP       | 13    |
| HAPTRFR    | DDS_PF  | HAP       | 72    |
| HMLMAST5H  | DDS_LF  | UNGROUPED | 12    |
| HXLTABLD   | DDS_LF  | HXLT      | 11    |
| HXLTABLP   | DDS_LF  | HXLT      | 12    |
| HXLTABLS   | DDS_LF  | HXLT      | 12    |
| HXPBNFIT   | DDS_LF  | HXP       | 12    |
| HXPDICT    | DDS_PF  | HXP       | 6130  |
| HXPLVL1    | DDS_PF  | HXPL      | 49    |
| HXPLVL2    | DDS_PF  | HXPL      | 52    |
| HXPLVL3    | DDS_PF  | HXPL      | 52    |
| HXPLVL4    | DDS_PF  | HXPL      | 52    |
| HXPLVL5    | DDS_PF  | HXPL      | 55    |
| HXPLVL6    | DDS_PF  | HXPL      | 321   |
| HXPNSTN    | DDS_LF  | HXP       | 12    |
| HXPTABLD   | DDS_PF  | HXP       | 19    |
| HXPXMLD    | DDS_PF  | HXPX      | 19    |
| HXPXMLR    | DDS_PF  | HXPX      | 19    |
| OAPIRNK    | DDS_PF  | UNGROUPED | 80    |
| OMPMAST    | DDS_PF  | UNGROUPED | 310   |
| OXPBNFIT   | DDS_PF  | OXP       | 48    |
| OXPNSTN    | DDS_PF  | OXP       | 65    |
| TAPIRNK    | DDS_PF  | UNGROUPED | 167   |
| TMPMAST    | DDS_PF  | UNGROUPED | 688   |
| TXPBNFIT   | DDS_PF  | TXP       | 164   |
| TXPNSTN    | DDS_PF  | TXP       | 116   |
| HXXAPPPRF  | SQLRPGLE| HXXA      | 123   |
| XFXCNTR    | RPGLE   | XFXC      | 49    |
| XFXCYMD    | RPGLE   | XFXC      | 83    |
| XFXGETID   | RPGLE   | XFX       | 61    |
| XFXLDSC    | RPGLE   | XFXL      | 135   |
| XFXLEAP    | RPGLE   | XFXL      | 61    |
| XFXMRNROL  | RPGLE   | XFX       | 65    |
| XFXTABL    | RPGLE   | XFX       | 164   |
| CXXXMLP    | SQLRPGLE| UNGROUPED | 25    |
| HXXAPPPRFP | RPGLE   | HXXA      | 42    |
| HXXCNTRL   | RPGLE   | HXX       | 8     |
| HXXLDA     | RPGLE   | HXXL      | 53    |
| HXXLEVEL   | RPGLE   | HXXL      | 25    |
| HXXXML     | RPGLE   | HXX       | 11    |
| HABADTE    | RPGLE   | HA        | 821   |

This inventory shows a data-heavy design: 19 physical files, 7 logical files, 2 SQLRPGLE members, and 13 RPGLE programs. The HABADTE RPGLE main program is by far the largest with 821 lines, reflecting its role as the patient census driver and coordinator of downstream services.

### 1.2 Members by Type

From the aggregated statistics:

- DDS_LF: 7
- DDS_PF: 19
- SQLRPGLE: 2
- RPGLE: 13

The prevalence of DDS_PF and DDS_LF members confirms a strongly file‑centric architecture, typical of legacy AS400 applications. The overall footprint is 10,288 lines of source code across 41 members, with completeness at ~91%, indicating that a small set of referenced components are missing from the scanned library.

## Section 2 – Missing Components

The gap catalogue identifies four missing or unresolved components that are referenced by the code but not present in the harvested source:

| Name       | Type      | Impact  | Referenced By |
|-----------|-----------|--------|---------------|
| CXXXMLC   | COPYBOOK  | HIGH   | HABAD         |
| HXHAPPPRF | PROGRAM   | MEDIUM | XFXMR         |
| ****HXPXML| FILE      | MEDIUM | HABAD         |
| PRINTER   | FILE      | MEDIUM | HABAD         |

These entries are authoritative for this run and represent the only known structural gaps.

### 2.1 High‑Impact Gap – CXXXMLC Copybook

CXXXMLC is a missing copybook referenced from the HABADTE main program via a COPY edge. Copybooks typically encapsulate shared field layouts, constants, or reusable logic. A high impact rating here reflects that HABADTE likely depends on CXXXMLC for XML‑related structures or routines. Without this copybook, downstream modernization efforts risk mis‑modeling message formats or losing subtle validation logic embedded in the copybook.

Implications:

- Any attempt to reconstruct HABADTE’s XML interface will be incomplete until CXXXMLC is obtained.
- Refactoring of HABADTE into services (REST, SOAP, or messaging) must assume that XML marshalling and configuration details are partially externalized in the missing copybook.
- Automated tooling that relies solely on the scanned library will under‑report fields or control structures defined in CXXXMLC.

### 2.2 Medium‑Impact Gap – HXHAPPPRF Program

HXHAPPPRF is a missing program called by XFXMRNROL. XFXMRNROL appears as a utility that rolls or translates medical record numbers and delegates to HXHAPPPRF and HXXAPPPRF for application profile logic. The absence of HXHAPPPRF means that only part of the MRN roll‑over workflow is visible.

Implications:

- Behavioural analysis of MRN profile maintenance is incomplete; only calls to HXXAPPPRF can be traced.
- Test case generation for MRN roll logic must treat HXHAPPPRF as a black‑box dependency and infer its side effects from database access patterns rather than source inspection.
- Migration teams must anticipate another module with similar responsibilities to HXXAPPPRF and plan stubs or adapters accordingly.

### 2.3 Medium‑Impact Gaps – ****HXPXML and PRINTER Files

Two missing file definitions are flagged:

- ****HXPXML – categorised as a FILE gap, referenced by HABADTE.
- PRINTER – another FILE gap, referenced by HABADTE.

These gaps suggest that HABADTE interacts with additional file‑based interfaces for XML payload handling and printing or spool management that were not part of the harvested library.

Implications:

- XML interface mapping is partially visible through HXPXMLD, HXPXMLR, and HXFXMLH operations, but the full schema and storage patterns may reside in the missing ****HXPXML definition.
- PRINTER likely represents a spool or printer file used for census reports. Its absence limits precise documentation of output layouts, page controls, and SCS/ECS attributes.
- Modernization design must treat outbound print and XML interfaces as external systems with contracts to be reconstructed from operations logs, spool samples, or DB2 metadata.

## Section 3 – Duplicate and Reused Components

Re‑use patterns can be inferred from dependency edges where the same target appears in multiple CALL or PFILE_OF relationships.

### 3.1 Reused RPG Service Programs

Based on CALL edges:

- **XFXCNTR** is called three times by HABADTE. Multiple CALL entries from HABADTE to XFXCNTR indicate that text centering logic is reused across several output segments (for example, centering headings or labels in different report lines).
- **XFXCYMD** is called from HABADTE and in turn calls **XFXLEAP**. This forms a small date‑validation cluster reused whenever HABADTE needs to validate or normalise census dates.
- **XFXMRNROL** is called from HABADTE and itself calls **HXHAPPPRF** and **HXXAPPPRF**, demonstrating reuse of MRN profile logic across the patient management domain.

These reused components embody cross‑cutting concerns: text formatting, date validation, and MRN profile handling. Their reuse increases their retrofit importance during modernization, as changes to these utilities will propagate widely.

### 3.2 Shared Logical Files over Common Physical Tables

Logical files demonstrate reuse of underlying physical files via PFILE_OF edges:

- **HXPTABLD** is the base PF for three LFs: HXLTABLD, HXLTABLP, HXLTABLS. Each LF projects different key sequences or subsets of the same table, indicating heavy reuse of a central table‑driven configuration dictionary (XFFTABLD).
- **TXPBNFIT** is the base PF for the LF HXPBNFIT. The logical view provides benefit‑plan specific access patterns over the shared benefit table.
- **TXPNSTN** underpins the LF HXPNSTN, offering a tuned access path for level 6 and status lookups.
- **TMPMAST** backs HMLMAST5H, a logical view that restructures patient master information for census processing.

These shared PF/LF patterns reflect an architecture that relies on table‑driven reference data, with logical files providing performance‑optimised or business‑specific views.

## Section 4 – Dependency Analysis

### 4.1 Call Graph Summary

The call graph constructed from dep_edges demonstrates a hub‑and‑spoke architecture centred on HABADTE:

- **HABADTE** calls: XFXMRNROL, XFXCNTR (three times), XFXLDSC, XFXCYMD, XFXGETID, XFXTABL.
- **XFXCYMD** calls: XFXLEAP, delegating leap‑year calculations to a dedicated utility.
- **XFXMRNROL** calls: HXHAPPPRF (missing) and HXXAPPPRF, binding MRN roll logic to application profile maintenance.
- **HXXAPPPRF** includes HXXCNTRL and HXXAPPPRFP via COPY, showing layered configuration and control structures.
- **XFXGETID** copies HXXLDA and reads HXFXMLR, making it a bridge between logical data areas and XML response records.

Copybook dependencies radiate from HABADTE to HXXLDA, HXXLEVEL, HXXXML, CXXXMLP, and the missing CXXXMLC. These COPY relationships signal heavy reliance on shared layouts and XML wrappers.

### 4.2 File Access Patterns and Hotspots

File‑level operations highlight HABADTE’s central role:

- HABADTE reads **HAPTRFR**, indicating direct access to transfer records with PHI sensitive fields AFACCT and AFMRNO.
- HABADTE reads **XFFNSTN** (the PF behind OXPNSTN/TXPNSTN views) to resolve organisational or status hierarchy.
- HABADTE interacts with **HXFXMLH** via READ, UPDATE, and WRITE, and writes to **HXFXMLD**, evidencing bidirectional XML header and detail processing.

Hotspot metrics further quantify dependency centrality:

- **HABADTE**: score 38, fan_in 0, fan_out 13, file_ops 6 – the primary orchestration program with high complexity and many downstream calls and file operations.
- **XFXLDSC**: score 15, fan_in 1, fan_out 0, file_ops 6 – a key organisational level description service used by HABADTE and reading multiple HXPLVLx files.
- **XFXTABL**: score 11, fan_in 1, fan_out 0, file_ops 4 – a table‑driven control program reading several XFFTABLx tables.
- **XFXCNTR**: score 9, fan_in 3, fan_out 0 – a widely reused text‑centering utility.

Together, these hotspots delineate the critical service cluster that should be prioritised for decomposition into reusable services during modernization.

## Section 5 – Summary of Key Findings

The HABADTE subsystem presents a classic AS400 design with a single high‑complexity orchestrator surrounded by smaller, reusable utilities and a dense file schema.

Architectural observations:

- **Central orchestration:** HABADTE, with cyclomatic complexity 152 and hotspot score 38, is the functional centre of the patient census workflow. It coordinates MRN roll‑over, organisational level lookups, table‑driven logic, and XML header/detail processing.
- **Service clustering:** Utility programs XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXMRNROL, and XFXGETID form a service cluster that provides formatting, validation, lookup, table management, and identifier generation. Their clear responsibilities and low individual complexity make them strong candidates for extraction as micro‑services or shared libraries.
- **Table‑driven configuration:** PFs HXPTABLD, HXPLVL1–HXPLVL6, TXPBNFIT, and TXPNSTN, together with their LFs, implement table‑driven configuration and reference data. This design can be retained in a modern platform as configuration tables or typed reference services.
- **XML and print interfaces:** HXPXMLD, HXPXMLR, HXFXMLH, and the missing ****HXPXML point to an XML‑based interface for downstream systems, while the PRINTER file gap indicates spool‑based report output. These external interfaces must be treated as first‑class integration points in the target architecture.
- **PHI concentration:** PHI‑tagged fields are concentrated in HXPDICT, HAPTRFR, OAPIRNK, OMPMAST, and OXPBNFIT. Although this report focuses on structural analysis, the presence of PHI fields within the core census and dictionary tables will have direct implications for security design and audit requirements.

Orphan program assessment:

The orphan_programs list includes HXXAPPPRF, XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC. However, the dependency graph shows that HABADTE calls XFXCNTR, XFXCYMD, XFXGETID, and XFXLDSC, and XFXMRNROL calls HXXAPPPRF. Since each of these programs has at least one known caller, there are **no true orphans** in this codebase. All active programs participate in defined call chains.

Recommendations:

- Prioritise HABADTE and its immediate service cluster for detailed refactoring analysis, given their centrality and low individual complexity (except HABADTE itself).
- Reconstruct missing components CXXXMLC, HXHAPPPRF, ****HXPXML, and PRINTER before finalising interface contracts for XML and print outputs.
- Preserve table‑driven patterns around HXPTABLD and HXPLVLx files, translating them into explicit configuration modules or metadata‑driven services in the target platform.
- Use the identified hotspots as starting points for application decomposition, ensuring that utilities like XFXCNTR and XFXCYMD are exposed via consistent service interfaces.
