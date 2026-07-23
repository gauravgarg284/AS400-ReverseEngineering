# Low-Level Design (LLD) – HABADTE Active Patients by Date and Time

## 1. Architecture Overview

### 1.1 Technology Stack and Source Types

The HABADTE solution is built on a traditional IBM i (AS400) stack using RPGLE/SQLRPGLE programs and DDS-defined physical and logical files. The aggregated source inventory shows:

- **DDS Logical Files (DDS_LF)**: 7
- **DDS Physical Files (DDS_PF)**: 19
- **SQLRPGLE Programs**: 2
- **RPGLE Programs**: 13

Physical files such as **OMPMAST**, **HAPTRFR**, **HXPDICT**, **OXPBNFIT**, and **OXPNSTN** store patient master data, transfer history, generic dictionary data, benefits, and nursing station information. Logical files (**HAPIRNK**, **HMLMAST5H**, **HXLTABLD/HXLTABLP/HXLTABLS**, **HXPBNFIT**, **HXPNSTN**) provide optimized key paths and filtered views tailored to reporting and lookup workflows.

RPGLE programs implement the core business logic, including the main census report **HABADTE** and utilities **XFXCNTR**, **XFXCYMD**, **XFXLDSC**, **XFXTABL**, **XFXGETID**, and **XFXMRNROL**. SQLRPGLE programs (**HXXAPPPRF**, **CXXXMLP**) interface with DB2 tables and XML-related configuration.

### 1.2 Call Graph Summary

The dependency edges define a clear orchestration pattern:

- **HABADTE** (main report driver) calls:
  - **XFXMRNROL** (MRN rollup logic)  
  - **XFXCNTR** (text centering utility; called three times)  
  - **XFXLDSC** (corporate level description lookup)  
  - **XFXCYMD** (date conversion, CCYYMMDD ↔ MMDDCCYY)  
  - **XFXGETID** (XML ID and fragment generation)  
  - **XFXTABL** (generic code/description table lookup)

- **XFXCYMD** calls **XFXLEAP** for leap-year-oriented calendar calculations.

- **XFXMRNROL** calls **HXHAPPPRF** and **HXXAPPPRF**, building a configuration-based MRN rollup layer.

- **HABADTE** and **XFXGETID** include copybooks **HXXLDA**, **HXXLEVEL**, **HXXXML**, **CXXXMLP**, **CXXXMLC** to share data areas, facility hierarchy, and XML layout definitions.

### 1.3 Design Patterns

Observed patterns include:

- **Utility Service Pattern**: XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, and XFXLEAP act as shared services. They implement narrow responsibilities (text formatting, date conversion, level description lookup, table lookup, XML ID retrieval) and are reused by HABADTE and other programs.

- **Configuration-Driven Behavior**: MRN rollup behavior in **XFXMRNROL** depends on external profile programs (**HXHAPPPRF**, **HXXAPPPRF**). This indicates that MRN aggregation rules are not hard-coded but derived from configuration tables and profile logic.

- **Layered Data Access**: The call graph and PFILE relationships show a layered approach where logical files (HAPIRNK, HMLMAST5H, HXLTABL*, HXPBNFIT, HXPNSTN) overlay physical files (TAPIRNK, TMPMAST, HXPTABLD, TXPBNFIT, TXPNSTN), decoupling logical views from storage structures.

- **Hybrid Output**: HABADTE uses both a **PRINTER** device file (for spool reports) and XML structures (**HXPXMLD**, **HXPXMLR**, **HXFXMLH**) for electronic reporting, suggesting an incremental transition from paper-based to XML-driven outputs.

## 2. Database Schema

The compact data dictionary schema defines the persistent model. Below is a PF/LF-oriented view.

### 2.1 Physical Files (PF)

For each physical file, we list record format, uniqueness, key fields, total fields, and PHI-sensitive fields.

1. **HAPTRFR**  
   - Record format: `HAFTRFR`  
   - Unique: `true`  
   - Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`  
   - Total fields: `28`  
   - PHI fields: `AFACCT` (AccountNumber), `AFMRNO` (MRN).  
   This table stores patient transfer records including facility level, account, transfer date/time, and type. It is used to retrieve current room assignments for a given census date.

2. **HXPDICT**  
   - Record format: `HXFDICT`  
   - Unique: `false`  
   - Key fields: `[]`  
   - Total fields: `2705`  
   - PHI fields: `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE`, `XMDMRN`, `ENNAME`.  
   This is a large dictionary/reference table containing patient identifiers, contact info, room numbers, account numbers, MRNs, dates of birth, and other extended attributes used across the system.

3. **HXPLVL1–HXPLVL6**  
   - HXPLVL1: record `HXFLVL1`, key `HX1NUM`, unique `true`, total fields `36`.  
   - HXPLVL2: record `HXFLVL2`, key `HX2NUM`, unique `true`, total fields `39`.  
   - HXPLVL3: record `HXFLVL3`, key `HX3NUM`, unique `true`, total fields `39`.  
   - HXPLVL4: record `HXFLVL4`, key `HX4NUM`, unique `true`, total fields `39`.  
   - HXPLVL5: record `HXFLVL5`, key `HX5NUM`, unique `true`, total fields `42`.  
   - HXPLVL6: record `HXFLVL6`, key `HX6NUM`, unique `true`, total fields `155`.  
   These tables hold corporate level hierarchy definitions (e.g., corporate, region, network, hospital, unit, and level 6 facility). XFXLDSC uses these files to resolve level descriptions used in HABADTE’s headings.

4. **HXPTABLD**  
   - Record format: `XFFTABLD`  
   - Unique: `false`  
   - Key fields: `XFDTCD`, `XFDECD`  
   - Total fields: `7`  
   - PHI fields: none.  
   Serves as a generic code table for dictionary mappings (e.g., room class codes). XFXTABL reads this table to derive short and long descriptions and mapping flags.

5. **HXPXMLD**  
   - Record format: `HXFXMLD`  
   - Unique: `true`  
   - Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`  
   - Total fields: `4`  
   - PHI fields: none.  
   Holds XML detail fragments for reports, with composite keys based on user, sequence, and sub-sequence.

6. **HXPXMLR**  
   - Record format: `HXFXMLR`  
   - Unique: `true`  
   - Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`  
   - Total fields: `4`  
   - PHI fields: none.  
   Stores XML report header records and configuration.

7. **OAPIRNK**  
   - Record format: `HBFIRNK`  
   - Unique: `true`  
   - Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`  
   - Total fields: `33`  
   - PHI fields: `BRKMRN` (MRN).  
   Likely holds patient ranking or index data per facility and account.

8. **OMPMAST**  
   - Record format: `HMFMAST`  
   - Unique: `true`  
   - Key fields: `MMPLV6`, `MMACCT`  
   - Total fields: `149`  
   - PHI fields: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN`, `MMMMRN`.  
   Core inpatient master table storing patient demographics and account-level data.

9. **OXPBNFIT**  
   - Record format: `XFFBNFIT`  
   - Unique: `true`  
   - Key fields: `XFBUBN`, `XFBPLN`  
   - Total fields: `34`  
   - PHI fields: `XFBTEL` (PhoneNumber).  
   Holds benefit/plan information including contact numbers.

10. **OXPNSTN**  
    - Record format: `XFFNSTN`  
    - Unique: `true`  
    - Key fields: `XFNLV6`, `XFNSST`  
    - Total fields: `23`  
    - PHI fields: none.  
    Nursing station configuration used via HXPNSTN/XFFNSTN to resolve station names for rooms.

11. **TAPIRNK/TMPMAST/TXPBNFIT/TXPNSTN**  
    - Record formats: `ATE TABLE` (indicator for table-style PFs)  
    - Unique: `false`  
    - Keys: `[]`  
    - Totals: `43` (TAPIRNK), `181` (TMPMAST), `12` (TXPBNFIT), `19` (TXPNSTN)  
    - PHI fields: none flagged in compact schema.  
    These tables are base PFs for logical views, used for ranking, master, benefit, and station-related lookups.

### 2.2 Logical Files (LF)

Logical files provide alternate keys and selections:

1. **HAPIRNK**  
   - PFILE: `TAPIRNK`  
   - Record format: `HBFIRNK`  
   - Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`  
   - Select/Omit: `[]`.  
   Offers a keyed view over TAPIRNK for account ranking per facility.

2. **HMLMAST5H**  
   - PFILE: `TMPMAST`  
   - Record format: `HMFMAST`  
   - Key fields: `MMPNST`, `MMADDT`, `MMADTM`  
   - Select/Omit: `[]`.  
   Provides a view of master records ordered by nursing station and admit date/time; used for station-specific reporting.

3. **HXLTABLD/HXLTABLP/HXLTABLS**  
   - PFILE: `HXPTABLD`  
   - Record format: `XFFTABLD`  
   - HXLTABLD keys: `XFDTCD`, `XFDMAP`  
   - HXLTABLP keys: `XFDTCD`, `XFDLDS`  
   - HXLTABLS keys: `XFDTCD`, `XFDSDS`  
   - Select/Omit: `[]`.  
   These logical files define specific key sequences for table codes: mapping code, long description, and short description. XFXTABL uses them to support flexible lookups by code and by description.

4. **HXPBNFIT**  
   - PFILE: `TXPBNFIT`  
   - Record format: `XFFBNFIT`  
   - Key fields: `XFBUBN`, `XFBPLN`  
   - Select/Omit: `[]`.  
   Provides keyed access to benefit plan configurations for patient accounts.

5. **HXPNSTN**  
   - PFILE: `TXPNSTN`  
   - Record format: `XFFNSTN`  
   - Key fields: `XFNLV6`, `XFNSST`  
   - Select/Omit: `[]`.  
   Supplies a direct keyed view over nursing station definitions used during HABADTE’s room/station resolution.

## 3. Status and Type Reference Data

Approved business rules contain implicit references to status or type codes that are meaningful at design time:

- HABADTE uses flags and types such as **file indicator=0** for pre-admits, **void flag = 'V'**, and **I/O indicator = 'O'** for outpatients. These codes represent record status that drive inclusion/exclusion logic.  
- MRN rollup configuration and table lookups use type codes like **room class code**, passed into XFXTABL as `tcode` and `ecode` to resolve descriptive text.

The compact rules set does not enumerate a centralized code table specifically for status codes beyond these values, and no additional explicit status/type reference codes are extracted from approved_rules beyond date validation ranges. Therefore, for this LLD:

> Status and type reference data are embedded in business rules (e.g., flags 'V', 'O', file indicator 0) and in table codes accessed via XFXTABL. No standalone status code catalog was identified in the aggregated rules.

If a future extraction pass surfaces explicit status value tables or enumerations, they should be modeled as separate reference entities.

## 4. Stored Procedure Logic Mappings

We treat RPGLE programs as stored procedure equivalents. CALL edges map caller-to-callee relationships:

### 4.1 HABADTE Caller Group

HABADTE calls the following programs:

- **XFXMRNROL** – 1 call  
  - Purpose: Determines MRN rollup setting (`mrnRollup`) for the report based on configuration.  
- **XFXCNTR** – 3 calls  
  - Purpose: Centers text strings (`rpthdg`, `hospnm`, `dates`) used in headings.  
- **XFXLDSC** – 1 call  
  - Purpose: Retrieves level description for facility-level code; returns `lvldsc`.  
- **XFXCYMD** – 1 call  
  - Purpose: Converts CCYYMMDD to printable date formats and vice versa.  
- **XFXGETID** – multiple calls via `getidsr`  
  - Purpose: Resolve XML element IDs and static text fragments based on `reportseq` and `getid`.  
- **XFXTABL** – 1 call  
  - Purpose: Resolve room class descriptions and determine if a mapping indicates hospital/therapeutic leave.

### 4.2 XFXCYMD Caller Group

- **XFXCYMD** → **XFXLEAP** – 1 call  
  - Purpose: Uses leap-year logic when evaluating day ranges and validating dates.

### 4.3 XFXMRNROL Caller Group

- **XFXMRNROL** → **HXHAPPPRF** – 1 call  
- **XFXMRNROL** → **HXXAPPPRF** – 1 call  

Both targets operate as configuration providers, returning MRN rollup behavior rules.

### 4.4 Copy-Based Logic Mappings

Copy dependencies establish shared structure rather than executable calls:

- **HXXAPPPRF** includes **HXXCNTRL**, **HXXAPPPRFP**.  
- **XFXGETID** and **HABADTE** include **HXXLDA** (data area definitions).  
- **HABADTE** includes **HXXLEVEL** (corporate level structures) and **HXXXML**, **CXXXMLP**, **CXXXMLC** (XML reporting layout).

These mappings should be preserved as common modules or shared libraries in a modern environment.

## 5. Service Class Method Reference

Using hotspot scores and fan-in/fan-out patterns, we can classify service-like programs:

- **BatchService (High-score programs)**  
  - **HABADTE** (score 38, fan_out 13, file_ops 6): Acts as a batch reporting service generating both spool and XML census output. High complexity (cc=152) and extensive IO justify treating it as a central batch service.

- **WorkflowService (Medium-score programs)**  
  - **XFXLDSC** (score 15, fan_in 1, file_ops 6): Implements workflow around resolving level descriptions from hierarchical tables.  
  - **XFXTABL** (score 11, fan_in 1, file_ops 4): Handles workflow for resolving code/descriptions from table files and can be treated as a reference data service.  
  - **XFXMRNROL** (score 7, fan_in 1, fan_out 2): Controls MRN rollup decisions by invoking profile programs; this is a configuration-driven workflow service.

- **UtilityService (Low-score programs)**  
  - **XFXCNTR** (score 9, fan_in 3, file_ops 0): Stateless string-centering utility.  
  - **XFXCYMD** (score 5, fan_in 1, fan_out 1): Date validation and conversion utility.  
  - **XFXGETID** (score 7, fan_in 1, fan_out 1, file_ops 1): XML ID resolution utility.  
  - **XFXLEAP**, **HXXAPPPRF**, **HXXAPPPRFP**, **HXXCNTRL**, **HXXLDA**, **HXXLEVEL**, **HXXXML**, **CXXXMLP**: Supporting utilities and shared modules with low complexity.

This classification guides future decomposition into microservices or modular components: HABADTE as a batch/reporting service, XFX* programs as shared infrastructure services.

## 6. External Interfaces

High/medium impact gaps and certain dependencies indicate external-facing interfaces:

- **XML Reporting Interface**: HABADTE writes XML fragments to **HXPXMLD** and interacts with **HXFXMLH** and **HXPXMLR** via XFXGETID and XML copybooks. Missing **CXXXMLC** and "****HXPXML" file definition highlight an incomplete view of the XML interface but confirm that XML output is a primary external integration point (possibly consumed by downstream reporting or data warehouse processes).

- **Configuration/Profiles Interface**: **HXHAPPPRF** and **HXXAPPPRF** act as configuration/profile providers, potentially reading DB2 tables beyond the schema subset shown. While their internal tables are not fully documented here, they represent inbound configuration interfaces for MRN rollup behavior.

- **Printer/Spool Interface**: HABADTE uses a `PRINTER` device file to produce formatted spool output. The absence of the printer DDS definition means physical printer characteristics are not documented, but spool output is clearly an interface to hospital print infrastructure.

From the compact high_medium_gaps list:

- **CXXXMLC (COPYBOOK, HIGH)** – inbound XML configuration interface; HABADTE depends on it to build XML payloads.  
- **HXHAPPPRF (PROGRAM, MEDIUM)** – inbound MRN rollup profile interface used by XFXMRNROL.  
- ******HXPXML (FILE, MEDIUM)** – incomplete XML header/config file definition referenced by HABADTE.

No outbound SPOOL interface beyond the internal printer device file was explicitly identified in the aggregated context. Therefore:

> No outbound SPOOL interfaces identified.

(Spool output remains an internal AS400 interface, not an integration endpoint in the aggregated view.)

## 7. Performance and Security Notes

### 7.1 Complexity and Performance

Cyclomatic complexity metrics highlight HABADTE as the primary performance and maintainability risk:

- **HABADTE**: `cc=152`, band `HIGH`. This reflects extensive conditional logic (filters for pre-admits, voids, outpatients, discharge checks, MRN rollup behavior) and multiple subroutines for XML writing and printer output. Any changes to report behavior must be carefully tested due to the large number of execution paths.

All other measured programs have **LOW** complexity (cc ≤ 9):

- **XFXTABL**: `cc=9` – most complex utility, managing table lookup flows including exit conditions when input indicators (e.g., *IN79) are active.  
- **XFXCYMD**: `cc=7` – date validation path with multiple boundary checks for year and month.  
- **XFXLDSC**: `cc=5` – level lookup logic.  
- Remaining XFX* and HXX* utilities show `cc=1–3`, indicating straightforward flows.

Performance considerations:

- HABADTE’s combination of high complexity and multiple file operations (HAPTRFR, XFFNSTN, HXFXMLH/HXFXMLD, plus dictionary lookups via XFXTABL) suggests that batch window and indexing strategies are important. However, DDS PFs use composite keys (facility + account + date/time), which should keep lookup operations efficient.

### 7.2 PHI-Sensitive Fields

The application manipulates several PHI-tagged fields:

- **HAPTRFR**: `AFACCT` (AccountNumber), `AFMRNO` (MRN).  
- **HXPDICT**: identifiers and contact data including `CCMRNO`, `XFBTEL`, `XCNAME`, `HXRMNO`, `XFRMNO`, `HVACCT`, `IMGMRN`, `HXGMRN`, `IHMRNO`, `IHACCT`, `WBDATE` (DOB), `XMDMRN`, `ENNAME`.  
- **OAPIRNK**: `BRKMRN` (MRN).  
- **OMPMAST**: `MMMRNO`, `MMACCT`, `MMNAME`, `MMPSSN` (SSN), `MMMMRN`.  
- **OXPBNFIT**: `XFBTEL` (PhoneNumber).

These fields appear in flows where HABADTE reads patient master and transfer data, potentially includes names, MRNs, and account numbers in spool headings and XML payloads. Although the current dataset is synthetic/metadata-only, in production these are sensitive and must be handled under HIPAA-aligned controls.

Security implications:

- XML exports via HXPXMLD/HXPXMLR likely propagate PHI to downstream systems. Modernization should enforce encryption in transit, strict access control, and data minimization in XML payloads.  
- Spool reports may contain patient identifiers and must be governed by secure printing policies (e.g., restricted printers, audit trails).

### 7.3 Tech Debt Summary

The aggregated tech debt summary reports:

- **Total findings**: 4  
- **Total remediation hours**: 26.9  
- **By severity**: HIGH=1, MEDIUM=3, LOW=0.

While individual findings are not enumerated here, the dominant concerns likely involve:

- HABADTE’s high complexity, making it hard to modify without regression risk.  
- Missing or tightly coupled XML and printer configuration components (CXXXMLC, ****HXPXML, PRINTER) that complicate deployment automation.  
- PHI-heavy tables (HXPDICT, OMPMAST) being accessed directly, potentially across multiple modules, which increases the attack surface.

Addressing these issues in a modernization initiative would include refactoring HABADTE into clearer service endpoints, externalizing XML and printer configuration, and introducing centralized security services for PHI access.

