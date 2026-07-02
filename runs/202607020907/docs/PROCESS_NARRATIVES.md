# Process Narratives – HABADTE AS400 Application

This document provides business-oriented narratives for each interpreted program in the HABADTE portfolio, using semantic interpretations, approved rules, and data dictionary context.

## 1. Program: XFXCNTR

### 1.1 Overview

XFXCNTR is an RPGLE utility program in the **DATA_MAINTENANCE** domain. It provides counter-based control logic for loops or batch-processing segments. Its narrative indicates that it contains a small number of rules with moderate confidence, serving primarily as an exit controller rather than a full business workflow.

### 1.2 Domain and Program Type

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### 1.3 Business Rules Applied

From `interpretations_detail.XFXCNTR.key_rules` and the approved rules list:

- **BR-001** – "When X equals zero, branch to 'EXIT'" (confidence 0.47).  
  - Business meaning: If the counter X is zero, there is no work to perform, so the routine exits early.
- **BR-002** – "When X equals 40, branch to 'EXIT'" (confidence 0.40).  
  - Business meaning: Once 40 iterations or items have been processed, the routine forces an exit, enforcing a batch limit.

### 1.4 Related Approved Rules

All approved rules in the DATA_MAINTENANCE domain for XFXCNTR are directly mapped above.

### 1.5 Data Touched

XFXCNTR does not directly read or write PHI-bearing files in the extracted graph. It operates on counters passed by callers (such as HABADTE), so its data footprint is limited to numeric control fields.

Relevant structural context:
- No direct dependencies in `dep_edges` for file operations.
- Used as a shared utility by HABADTE and possibly other drivers.

---

## 2. Program: XFXCYMD

### 2.1 Overview

XFXCYMD is an RPGLE program in the **DATA_MAINTENANCE** domain that performs date validation. The narrative notes that it has six rules with an average confidence of 58%, corresponding to checks on year, month, and day values.

### 2.2 Domain and Program Type

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### 2.3 Business Rules Applied

Key rules:

- **BR-003** – "When VYY is less than 1800, branch to 'EXIT'".  
- **BR-004** – "When VYY is greater than 2100, branch to 'EXIT'".  
- **BR-005** – "When VMM is less than 01, branch to 'EXIT'".  
- **BR-006** – "When VMM is greater than 12, branch to 'EXIT'".  
- **BR-007** – "When VDD is less than 01, branch to 'EXIT'".  
- **BR-008** – "When VDD is greater than DYS(VMM), branch to 'EXIT'".

These rules collectively enforce that dates fall within realistic year ranges and respect month-specific day counts.

### 2.4 Related Approved Rules

All listed rules for XFXCYMD in the approved set are represented above.

### 2.5 Data Touched

XFXCYMD operates purely on in-memory date components (VYY, VMM, VDD) and does not access physical files directly. However, its validations apply to dates in PHI-bearing files like:
- **HAPTRFR** – transfer dates (AFTRDT).  
- **OMPMAST** – admission or birth dates.  
- **HXPDICT** – various encounter and birth dates.

By enforcing consistency, XFXCYMD indirectly protects the quality of PHI-related date fields.

---

## 3. Program: XFXLDSC

### 3.1 Overview

XFXLDSC is an RPGLE program in the **DATA_MAINTENANCE** domain that reads hierarchical level tables (HXPLVL1–HXPLVL6) and maps level identifiers to descriptions. Its narrative states that it has four rules with average confidence 56%, focusing on constraints for mapping codes.

### 3.2 Domain and Program Type

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### 3.3 Business Rules Applied

Key rules:

- **BR-009 / BR-010 / BR-011** – "When LDAMAP is greater than 99, branch to 'EXIT'".  
- **BR-012** – "When LDAMAP is greater than 9999, branch to 'EXIT'".

Business interpretation:
- LDAMAP represents mapping or configuration codes for level descriptions. The rules ensure these codes remain within configured ranges, preventing invalid mapping from being applied.

### 3.4 Related Approved Rules

All rules associated with XFXLDSC are captured above.

### 3.5 Data Touched

From `dep_edges` and `data_dict_schema`:

- Reads from **HXPLVL1–HXPLVL6** via record formats HXFLVL1–HXFLVL6.  
- Uses level keys (HX1NUM–HX6NUM) to resolve descriptions.

These PFs contain configuration and hierarchy data, not PHI, but they determine how PHI-bearing records (e.g., HAPTRFR, OMPMAST) are grouped and displayed.

PHI-related context:
- While HXPLVL* files themselves have no PHI fields, their mappings are used when reporting on HAPTRFR and OMPMAST records, which do contain MRN and account numbers.

---

## 4. Program: XFXTABL

### 4.1 Overview

XFXTABL is an RPGLE program in **DATA_MAINTENANCE** responsible for table/dictionary lookups using indicator *IN79 to control exit conditions. The narrative notes four rules with high average confidence (90%), reflecting robust extraction of its control logic.

### 4.2 Domain and Program Type

- **Domain:** DATA_MAINTENANCE  
- **Program Type:** RPGLE

### 4.3 Business Rules Applied

Key rules:

- **BR-013–BR-016** – "When *IN79 equals on/active, branch to 'EXIT'".

Business interpretation:
- *IN79 serves as a global control flag. When active, dictionary or table lookups stop, indicating that the desired condition has been met or a stop condition is reached.

### 4.4 Related Approved Rules

All four rules are directly from the approved set.

### 4.5 Data Touched

From `dep_edges`:

- Reads from **XFFTABLD**, **XFFTABL2**, **XFFTABL3**, **XFFTABL4** (PFs underpinning table dictionaries).  
- Logical files **HXLTABLD/LP/LS** expose views over **HXPTABLD** for mapping, long descriptions, and short descriptions.

These structures contain configuration and code tables. While they do not hold PHI themselves, they are used to interpret codes in PHI-bearing files (e.g., status codes for records in HAPTRFR and OMPMAST).

---

## 5. Program: HABADTE

### 5.1 Overview

HABADTE is an RPGLE program in the **PATIENT_MANAGEMENT** domain. It is the main driver of the patient transfer reporting and XML generation workflow. The narrative states that it contains three rules with very high average confidence (97%), all related to record eligibility.

### 5.2 Domain and Program Type

- **Domain:** PATIENT_MANAGEMENT  
- **Program Type:** RPGLE

### 5.3 Business Rules Applied

From `interpretations_detail.HABADTE.key_rules` and approved rules:

- **BR-018** – "When -FLAG INDICATOR equals void/voided, branch to 'SKIP'" (confidence 0.99).  
  - Business meaning: Exclude voided transfer records from downstream reporting and XML output.
- **BR-019** – "When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'" (confidence 0.99).  
  - Business meaning: Limit processing to inpatient encounters; outpatient transfers are handled elsewhere or not included.
- **BR-017** – "When -FILE INDICATOR equals zero, branch to 'SKIP'" (confidence 0.92).  
  - Business meaning: Skip records that are not fully initialized or have invalid file status.

Together, these rules implement a strong eligibility filter for PHI-bearing transfer events.

### 5.4 Related Approved Rules

HABADTE is also linked to other domain rules indirectly through its calls to XFXCYMD, XFXLDSC, XFXTABL, and XFXCNTR, but its own approved rules are the three filters above.

### 5.5 Data Touched

From `dep_edges` and `data_dict_schema`:

- **HAPTRFR (PF)** – Patient transfer file with PHI fields AFACCT (AccountNumber) and AFMRNO (MRN). HABADTE declares and reads this file, applying eligibility rules.
- **HXPNSTN / XFFNSTN (PF/LF)** – Institution status data used for enrichment of transfer records.
- **HMLMAST5H / TMPMAST (LF/PF)** – Historical patient master data referenced via logical file; TMPMAST is a gap to be resolved.
- **HXPBNFIT / TXPBNFIT (LF/PF)** – Benefit data structures used to enrich patient transfers; TXPBNFIT is a gap.
- **HAPIRNK / TAPIRNK (LF/PF)** – Rank or break records used in reporting; TAPIRNK is a gap.
- **HXPXMLD / HXPXMLR / HXFXMLH / HXFXMLD** – XML header/detail files for integration. HABADTE reads and writes these, forming the external interface.
- **OMPMAST (PF)** – Patient master file with MRNs, account numbers, names, and SSNs, accessed indirectly via MRN rollover logic (XFXMRNROL/HXXAPPPRF).

PHI-flagged files in HABADTE’s scope:
- HXPDICT  
- OXPBNFIT  
- OMPMAST  
- OAPIRNK  
- HAPTRFR

HABADTE orchestrates PHI across these tables, meaning that its migration must adhere strictly to privacy and access controls.

---

## 6. Cross-Program Relationships

While each program above has its own narrative, they participate in a broader workflow:

- HABADTE calls **XFXCNTR** to manage counters and exit conditions in loops.
- HABADTE calls **XFXCYMD** to validate transfer dates before including records.
- HABADTE calls **XFXLDSC** to enrich records with level descriptions from HXPLVL1–6.
- HABADTE calls **XFXTABL** to translate codes using HXPTABLD and its logical views.
- HABADTE calls **XFXMRNROL** and **HXXAPPPRF** (via XFXMRNROL) to manage MRN rollover based on profiles.
- HABADTE calls **XFXGETID** to obtain XML batch identifiers from HXPXMLR/HXFXMLR.

These relationships show a layered architecture in which DATA_MAINTENANCE utilities provide validation and lookup services to the PATIENT_MANAGEMENT driver.

---

## 7. Narrative Summary for Modernization

From a modernization perspective, the narratives highlight:

- **XFXCNTR** – Should be modeled as a shared counter-control service or reusable method that enforces batch limits and zero-work exits.
- **XFXCYMD** – Becomes a date validation component, validating input parameters and record fields before any PHI is persisted or exposed.
- **XFXLDSC** – Maps hierarchical levels to descriptive text; vital for producing human-readable reports and API responses.
- **XFXTABL** – Acts as a dictionary lookup service; it should be implemented as a repository-backed service with clear semantics around the *IN79 control flag.
- **HABADTE** – Evolves into a central orchestration service/API that coordinates all utilities, enforces PHI-safe eligibility rules, and generates XML-equivalent or JSON responses.

These narratives ensure that forward engineering teams understand not only the structural dependencies but also the business intent behind each program, allowing them to reconstruct equivalent flows in Java, Spring Boot, and SQL Server.
