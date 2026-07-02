# Low-Level Design (LLD)

## 1. Architecture Overview

### 1.1 Technology Stack and Component Types

The analyzed AS400 portfolio is primarily implemented in ILE RPG (RPGLE) with at least one SQLRPGLE program. The core business domains represented are **DATA_MAINTENANCE** and **PATIENT_MANAGEMENT**.

Key interpreted programs:

- **XFXCNTR** – RPGLE, DATA_MAINTENANCE, counter management and branching logic based on numeric thresholds.
- **XFXCYMD** – RPGLE, DATA_MAINTENANCE, calendar/date validation logic.
- **XFXLDSC** – RPGLE, DATA_MAINTENANCE, load/description logic for code tables.
- **XFXTABL** – RPGLE, DATA_MAINTENANCE, table lookup and validation utilities.
- **HABADTE** – RPGLE, PATIENT_MANAGEMENT, patient admission/bed‑assignment or transfer control.
- **HXXAPPPRF** – SQLRPGLE, PATIENT_MANAGEMENT, application profile management.

The codebase interacts with multiple physical files (PFs) that carry PHI (Protected Health Information), including OMPMAST, HAPTRFR, OXPBNFIT, HXPDICT, and OAPIRNK. Logical files (LFs) provide alternate keyed views for reporting and operational access.

### 1.2 Call Graph and Flow

While the full call graph is not enumerated in the summary, the domain interpretations and gap references establish the following architectural patterns:

- **Utility Layer (XFX*)** – XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, and XFXLEAP form a reusable toolkit for counters, date arithmetic, leap‑year checks, identifier generation, and generic table access. These are invoked from business programs across domains.
- **Domain Orchestrator Layer** – HABADTE coordinates patient management logic and interacts with external artifacts such as copybooks (CXXXMLC) and output destinations (PRINTER, ****HXPXML), indicating responsibility for both interactive and batch output.
- **Profile and Ranking Layer** – HXXAPPPRF and HXHAPPPRF (missing program) provide profile/ranking logic for patient or application entities. These interact with PHI‑carrying files such as OAPIRNK and TAPIRNK.

### 1.3 Design Patterns

The system implements several recognizable patterns common in AS400 applications:

- **Table‑Driven Validation** – XFXTABL implements business rules by reading table entries instead of hardcoding constants in logic, aligning with a rules‑table pattern.
- **Utility/Helper Services** – XFXCYMD and XFXLEAP encapsulate date validation and leap‑year logic, providing a reusable service façade for chronological checks.
- **Batch Orchestration** – HABADTE orchestrates multi‑step flows, reading flags and status indicators to decide whether to skip records or route them to downstream processing.

These patterns should be preserved when modernizing, for example by migrating XFX* programs into shared microservices or domain services.

## 2. Database Schema

The database schema is derived from the aggregated data dictionary schema, with special focus on PHI‑bearing PFs and their related LFs.

### 2.1 Physical File: OMPMAST (Patient Master)

- **Record Format**: (derived from data_dict_schema) – primary patient master record.
- **Unique Key**: Likely includes patient identifier (e.g., MRN) and facility or account key.
- **Key Fields**: MRN, facility code, possibly encounter or admission id.
- **PHI Fields**: Name, date of birth, contact details, and other identifying attributes.

Field characteristics include:

- **Identifiers** – Patient ID, account number, and cross‑reference keys to benefits and ranking tables.
- **Demographics** – Data fields categorized as PHI under HIPAA, including patient name, date of birth, and address components.

### 2.2 Physical File: HAPTRFR (Patient Transfer)

- **Record Format**: Transfer transaction records.
- **Unique Key**: Composite of patient identifier and transfer timestamp or sequence.
- **Key Fields**: Patient ID, from‑location, to‑location, transfer date/time.
- **PHI Fields**: Patient identifier and related demographic cross‑references.

The transfer file supports HABADTE and other patient‑management programs by storing movement events across wards or facilities.

### 2.3 Physical File: OXPBNFIT (Benefits)

- **Record Format**: Benefit or coverage details for patients.
- **Unique Key**: Policy/plan identifier and patient link.
- **Key Fields**: Plan ID, patient ID, effective/expiration dates.
- **PHI Fields**: Indirect PHI via patient identifiers and possibly subscriber information.

### 2.4 Physical File: HXPDICT (Dictionary/Lookup)

- **Record Format**: Code/description pairs for statuses, types, and ranking codes.
- **Unique Key**: Dictionary code.
- **Key Fields**: Code, type, language or environment.
- **PHI Fields**: Typically none directly, but used by PHI‑bearing programs.

### 2.5 Physical File: OAPIRNK (Ranking)

- **Record Format**: Ranking metrics for patients or providers.
- **Unique Key**: Ranking ID with patient or provider key.
- **Key Fields**: Rank code, score, effective date.
- **PHI Fields**: Indirect through patient/provider IDs.

### 2.6 Logical Files

Logical files provide alternate views over the PFs above. For example:

- LFs over OMPMAST keyed by admission date or facility to support batch extracts.
- LFs over OAPIRNK providing ranking by score or date range for analytics.
- LFs over OXPBNFIT grouped by plan or coverage type for benefits reporting.

Each LF definition includes:

- **PFILE**: Base PF (e.g., OMPMAST, OXPBNFIT, OAPIRNK).
- **Record Format**: Derived format from the PF.
- **Key Fields**: Alternate keys such as facility, date, ranking score.
- **Select/Omit**: Criteria for active/inactive records, voided flags, or date ranges.

## 3. Status and Type Reference Data

Status and reference code behavior can be derived from approved business rules.

### 3.1 Status Rules from Approved Rules

- **File and Flag Indicators in HABADTE (PATIENT_MANAGEMENT)**:
  - *BR‑017*: When FILE INDICATOR equals zero, the logic branches to `SKIP`, indicating no active record and terminating processing for that case.
  - *BR‑018*: When FLAG INDICATOR equals `void/voided`, the record is skipped; voided events are excluded from further processing.
  - *BR‑019*: When INPATIENT/OUTPATIENT FLAG equals `outpatient`, the record is skipped, suggesting this program is specific to inpatient flows.

- **Counter and Threshold Rules in XFXCNTR (DATA_MAINTENANCE)**:
  - *BR‑001*: When X equals 0, branch to `EXIT` – effectively a guard to bypass processing when counter is uninitialized.
  - *BR‑002*: When X equals 40, branch to `EXIT` – likely a maximum threshold for iteration or paging.

- **Date Validation Rules in XFXCYMD**:
  - *BR‑003–BR‑008*: Enforce year between 1800 and 2100, month 1–12, day between 1 and the correct days‑in‑month. Invalid dates branch to `EXIT`.

These rules form the foundation of status and type reference behavior. If no additional explicit status code tables are defined in the dictionary, these rules themselves define the acceptable status ranges.

If explicit status code PFs/LFs exist beyond HXPDICT, they are not visible in the current summary. Therefore, no additional enumerated status code lists can be documented.

## 4. Stored Procedure Logic Mappings

In the AS400 context, stored procedure‑like behavior is implemented via RPG program calls. Based on the dependency edges implied by the summary:

### 4.1 Caller: HABADTE (Patient Management)

- **Responsibilities**:
  - Read patient transfer records from HAPTRFR.
  - Consult external copybook CXXXMLC for configuration or mapping logic.
  - Write or format outputs to PRINTER file and ****HXPXML, which likely represents an XML export file.
- **Mapping**:
  - Each call to HABADTE can be treated as a procedure invocation that takes patient transfer context and produces either printed or XML output.

### 4.2 Caller: XFXMRNROL (MRN Roll Logic)

- **Responsibilities**:
  - Perform MRN rollover or re‑assignment operations.
  - Call HXHAPPPRF (missing program) to resolve application profile or ranking data.
- **Mapping**:
  - XFXMRNROL behaves like a stored procedure that orchestrates MRN changes and delegates profile updates to HXHAPPPRF.

### 4.3 Utility Call Chains

- XFXCNTR, XFXCYMD, XFXLDSC, and XFXTABL are invoked from various callers as sub‑procedures performing validation and lookup logic.
- Though their individual call counts are not enumerated, they should be treated as parameterized functions with well‑defined side effects (e.g., date validation, table lookup).

## 5. Service Class Method Reference

Using dependency hotspot characteristics, programs are classified into conceptual service classes:

- **BatchService** (high score/hotspot programs):
  - **HABADTE** – orchestrates batch or scheduled jobs for patient transfers and report generation; interacts with multiple PFs and output devices.

- **WorkflowService** (medium score programs):
  - **XFXMRNROL** – coordinates MRN roll operations by calling HXHAPPPRF and updating master/ranking tables.
  - **HXXAPPPRF** – SQLRPGLE program handling application profile workflows, including SQL updates and queries.

- **UtilityService** (low score/shared utilities):
  - **XFXCNTR** – counter management utility with simple branch logic based on X.
  - **XFXCYMD** – date validation service validating year, month, and day.
  - **XFXLDSC** – dictionary/description loader for table entries.
  - **XFXTABL** – generic table lookup and rule evaluation.
  - **XFXGETID** – identifier generation utility.
  - **XFXLEAP** – leap‑year calculation helper.

These classifications provide a mapping for future service decomposition: utility services become shared libraries or microservices, workflow services become orchestrators, and batch services align with scheduled or streaming jobs.

## 6. External Interfaces

### 6.1 Inbound Interfaces (from High/Medium Gaps)

The high‑ and medium‑impact gaps identify several external interface points:

- **COPYBOOK CXXXMLC** – High‑impact. Likely contains layout or mapping definitions for XML structures or printer parameters. Its use in HABADTE suggests configuration‑driven mapping for external outputs.
- **File TAPIRNK** – High‑impact. Referenced by HAPIRNK, probably a ranking or scoring PF used by reporting or decision support systems.
- **File TMPMAST** – High‑impact. Referenced by HMLMAST5H, possibly a temporary or transactional master used by external loading/ETL processes.
- **File TXPBNFIT** – High‑impact. Referenced by HXPBNFIT, representing benefits export or transformation staging.
- **File TXPNSTN** – High‑impact. Referenced by HXPNSTN, likely used to stage notification or statement data.
- **File ****HXPXML** – Medium‑impact. Referenced by HABADTE, appears to be an XML output or staging file for downstream systems.
- **File PRINTER** – Medium‑impact. Represents a printer spool or device file for human‑readable reports.

These interfaces mark boundaries between the core application and external consumers such as report systems, downstream data warehouses, or integration hubs.

### 6.2 Outbound SPOOL Interfaces

No explicit outbound SPOOL interfaces beyond the PRINTER file are identified in the aggregated summary. Consequently, the following statement applies:

- **No outbound SPOOL interfaces identified.**

If additional SPOOL files exist in the broader codebase, they are not present in this harvested run and will need to be cataloged separately.

## 7. Performance and Security Notes

### 7.1 Complexity and Performance

The complexity scores in the summary indicate that no programs are flagged as critical or high‑risk purely from a cyclomatic complexity perspective (critical_risk_programs and high_risk_programs both zero). However, several design factors influence performance:

- Utility programs (XFXCYMD, XFXTABL) are likely called from many contexts; their performance characteristics will have a multiplicative effect across the application.
- HABADTE, as a batch orchestrator accessing multiple PFs and external files, should be reviewed for I/O patterns, locking behavior, and exception handling.

Even with moderate complexity, performance bottlenecks may arise from inefficient I/O or lack of proper indexing on PF/LF keys.

### 7.2 PHI Exposure and Security

Five files are explicitly flagged as PHI‑bearing:

- **OXPBNFIT** – benefits information.
- **OMPMAST** – patient master data.
- **HAPTRFR** – patient transfer transactions.
- **HXPDICT** – dictionaries used by PHI‑handling programs.
- **OAPIRNK** – ranking metrics tied to patient/provider identifiers.

Security implications:

- Access to these PFs must be strictly controlled, with authorization enforced at the program and file level, mirroring existing AS400 authority models.
- Any modernization (e.g., exposing these records via APIs) must enforce equivalent or stronger access control, audit logging, and encryption in transit.

### 7.3 Technical Debt

The aggregated technical debt summary indicates zero recorded findings and zero total remediation hours in this run. This likely reflects that a detailed static‑analysis pass for technical debt was not configured rather than an absence of technical debt.

Given the age and structure of the codebase, likely areas of technical debt include:

- Tight coupling between PFs and business logic within RPG programs.
- Dependency on missing external artifacts (e.g., CXXXMLC, TAPIRNK) that complicate deployment and testing.
- Lack of explicit error handling and logging in older utility modules.

These should be validated in a subsequent deeper code review and incorporated into a modernization backlog.

In summary, this LLD provides a structural and behavioral blueprint for the AS400 application, with clear mappings from programs to PF/LF schemas, external interfaces, and service‑class roles. It should serve as a foundation for both incremental refactoring and larger‑scale modernization initiatives.