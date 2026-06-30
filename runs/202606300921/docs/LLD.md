# Low-Level Design (LLD) — HABADTE

## Section 1 — Architecture Overview

### 1.1 Technology Stack

| Layer            | Technology / Object Type | Notes |
|------------------|--------------------------|-------|
| Application Code | RPGLE, SQLRPGLE         | Core business logic and utilities (HABADTE, XFX* programs, HXX* helpers). |
| Data Definition  | DDS PF/LF               | Physical and logical files for patient, benefit, institution, dictionary, and lookup tables. |
| Database Engine  | IBM i DB2               | Underlying database for PF/LF objects. |
| Messaging / XML  | Custom XML via PFs      | HXPXMLD/HXPXMLR store XML definitions and messages. |
| Configuration    | Table-driven via HXPTABLD/HXLTABL* | Lookup-based configuration for preferences, statuses, and mappings. |
| Batch Orchestration | HABADTE RPGLE program | Main driver for patient management batch logic. |

### 1.2 Component Interaction

The HABADTE system follows a hub-and-spoke pattern:

- HABADTE (RPGLE) orchestrates batch processing of patient/account records.
- It calls utility programs (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXMRNROL, XFXLEAP) to perform reusable calculations and validations.
- It leverages copy members (HXXLEVEL, HXXAPPPRFP, HXXLDA, HXXCNTRL, HXXXML, CXXXMLP, CXXXMLC) to share structures and logic across modules.
- Data is read from PFs such as OMPMAST, HAPTRFR, OAPIRNK, OXPBNFIT, OXPNSTN, HXPDICT, HXPTABLD, HXPLVL1–6, and written to HXPXMLR and PRINTER.
- Logical files (HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN, HXLTABLS, HXLTABLP, HXLTABLD) provide alternate keying and filtering.

### 1.3 Design Patterns

#### Call Patterns

- HABADTE uses direct program calls to XFX* utilities for specialized logic.
- Copy members encapsulate common code segments that are included at compile time.
- Preference and hierarchy logic is centralized in HXXAPPPRF/HXXLEVEL and invoked from HABADTE.

#### Data Coupling

- Strong coupling to PF/LF schemas via DDS; logical files reuse PF formats using REF().
- Table-driven configuration (HXPTABLD + HXLTABL*) reduces hard-coded constants and allows runtime configuration.

#### Error Handling

- Many utilities implement "branch to EXIT" rules when data falls outside valid ranges (BR-001–BR-016).
- HABADTE uses skip logic for records based on flags (BR-017–BR-019), effectively filtering out invalid or non-target records.

#### Initialization

- LDA (Local Data Area) is initialized via HXXLDA and described via XFXLDSC.
- Preferences are loaded at startup via HXXAPPPRF/HXXAPPPRFP and HXPTABLD/HXLTABL*.

#### Framework Usage

- No external framework; the system relies on IBM i native constructs (RPG, DDS, PF/LF, data areas) and table-driven configuration.

---

## Section 2 — Database Schema

### 2.1 Physical Files (PF)

For each PF, the field details are derived from the data dictionary and PHI registry.

#### 2.1.1 HAPTRFR (Transfer File)

- Record Format: HAPTRFRR
- Unique Key: Composite of AFACCT, AFMRNO, and transfer sequence.
- Key Fields: AFACCT (Account), AFMRNO (MRN), transfer date/time.
- PHI Sensitive: Yes.

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| AFACCT  | CHAR     | 10     | 0   | No       | Yes| Yes| Account number. |
| AFMRNO  | CHAR     | 10     | 0   | No       | Yes| Yes| Medical Record Number. |
| ...     | ...      | ...    | ... | ...      | ...| ...| Other transfer-related fields (amounts, dates, flags). |

#### 2.1.2 OMPMAST (Master Patient File)

- Record Format: OMPMASTR
- Unique Key: MMMRNO (MRN) or MMACCT (Account).
- Key Fields: MMMRNO, MMACCT.
- PHI Sensitive: Yes.

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| MMMRNO  | CHAR     | 10     | 0   | No       | Yes| No | Medical Record Number. |
| MMACCT  | CHAR     | 10     | 0   | No       | Yes| No | Account number. |
| MMNAME  | CHAR     | 30     | 0   | No       | No | No | Patient name. |
| MMPSSN  | CHAR     | 11     | 0   | Yes      | No | No | Social Security Number / national ID. |
| ...     | ...      | ...    | ... | ...      | ...| ...| Additional demographic and status fields. |

#### 2.1.3 OXPBNFIT (Benefit File)

- Record Format: OXPBNFITR
- Unique Key: Benefit code.
- PHI Sensitive: Yes (XFBTEL).

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| XFBTEL  | CHAR     | 15     | 0   | Yes      | No | No | Benefit contact telephone number. |
| ...     | ...      | ...    | ... | ...      | ...| ...| Benefit code, descriptions, status flags. |

#### 2.1.4 OXPNSTN (Institution File)

- Record Format: OXPNSTNR
- Unique Key: Institution code.
- PHI Sensitive: No (institution-level data).

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| INSTCD  | CHAR     | 6      | 0   | No       | Yes| No | Institution code. |
| ...     | ...      | ...    | ... | ...      | ...| ...| Institution name, address, type. |

#### 2.1.5 HXPDICT (Dictionary File)

- Record Format: HXPDICTR
- Unique Key: CCMRNO or HXRMNO.
- PHI Sensitive: Yes.

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| CCMRNO  | CHAR     | 10     | 0   | No       | Yes| Yes| Cross-reference MRN. |
| HXRMNO  | CHAR     | 10     | 0   | No       | Yes| Yes| HABADTE-specific MRN. |
| XCNAME  | CHAR     | 30     | 0   | Yes      | No | No | Cross-reference name. |
| XFBTEL  | CHAR     | 15     | 0   | Yes      | No | No | Telephone number. |

#### 2.1.6 HXPTABLD (Table Definition File)

- Record Format: HXPTABLDR
- Unique Key: Table ID + key.
- PHI Sensitive: No.

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| TBLID   | CHAR     | 8      | 0   | No       | Yes| No | Table identifier. |
| KEYVAL  | CHAR     | 10     | 0   | No       | Yes| No | Key value. |
| DESC    | CHAR     | 40     | 0   | Yes      | No | No | Description. |

#### 2.1.7 HXPXMLD / HXPXMLR (XML Files)

- HXPXMLD defines XML layouts; HXPXMLR stores runtime XML messages.

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| XMLID   | CHAR     | 20     | 0   | No       | Yes| No | XML layout identifier. |
| XMLTEXT | CLOB     | 32767  | 0   | Yes      | No | No | XML content. |

#### 2.1.8 HXPLVL1–HXPLVL6 (Org Levels)

Each level PF represents a hierarchy layer.

| Column  | DB2 Type | Length | Dec | Nullable | PK | FK | Description |
|---------|----------|--------|-----|----------|----|----|-------------|
| LVLKEY  | CHAR     | 10     | 0   | No       | Yes| No | Level key (e.g., enterprise/region/facility). |
| LVLNAME | CHAR     | 40     | 0   | Yes      | No | No | Level name. |

### 2.2 Logical Files (LF)

#### 2.2.1 HAPIRNK

- Based On PF: OAPIRNK / TAPIRNK.
- Format Source: OAPIRNK.
- Key Fields: Rank, account.
- Select/Omit: May select active ranks only.

#### 2.2.2 HMLMAST5H

- Based On PF: TMPMAST (missing).
- Format Source: TMPMAST.
- Key Fields: MRN, account.
- Select/Omit: HABADTE-specific subset (e.g., inpatients only).

#### 2.2.3 HXPBNFIT

- Based On PF: OXPBNFIT / TXPBNFIT.
- Key Fields: Benefit code.
- Select/Omit: Active benefits; may omit expired ones.

#### 2.2.4 HXPNSTN

- Based On PF: OXPNSTN / TXPNSTN.
- Key Fields: Institution code.

#### 2.2.5 HXLTABLS / HXLTABLP / HXLTABLD

- Based On PF: HXPTABLD.
- Key Fields: Table ID + key value (and possibly level).
- Select/Omit: Status-based selection (e.g., omit inactive entries).

---

## Section 3 — Status and Type Reference Data

Status codes are inferred from CABEQ-like literal constants and table definitions in HXPTABLD/HXLTABL*.

| Status Code | Meaning                | Source |
|-------------|------------------------|--------|
| ACT         | Active record          | HXPTABLD / CABEQ literals. |
| INACT       | Inactive record        | HXPTABLD / CABEQ literals. |
| VOID        | Voided record          | HABADTE flags (BR-018). |
| IP          | Inpatient              | HABADTE flags (BR-019). |
| OP          | Outpatient             | HABADTE flags (BR-019). |

---

## Section 4 — Stored Procedure Logic Mappings

Top subroutines with fan_in > 1 (called from multiple contexts) should be mapped to stored procedures or shared services.

| Program   | Subroutine       | Parameters                      | Files Affected            | Key Logic (Summary)                                      | Status Transition |
|-----------|------------------|---------------------------------|---------------------------|----------------------------------------------------------|-------------------|
| XFXCNTR   | CNTCHK           | Counter value                   | HXPTABLD/HXLTABL*        | Implements BR-001, BR-002; branches to EXIT when out of range. | Counter flags set/cleared. |
| XFXCYMD   | DATECHK          | VYY, VMM, VDD                   | HXPTABLD/HXLTABL*        | Implements BR-003–BR-008; validates date ranges.        | Date validity flag. |
| XFXLDSC   | LDAMAPCHK        | LDAMAP                          | *LDA                      | Implements BR-009–BR-012; validates LDA map values.     | LDA validity indicator. |
| XFXTABL   | INDCHK           | *IN79, table key                | HXPTABLD/HXLTABL*        | Implements BR-013–BR-016; branches on indicator and table entries. | Indicator-based routing. |
| HABADTE   | REC_FILTER       | File indicator, flag indicator, IP/OP flag | HAPTRFR, OMPMAST | Implements BR-017–BR-019; skips non-target records.     | Record inclusion/exclusion. |

---

## Section 5 — Service Class Method Reference

For migration, programs can be grouped into service classes.

| Class            | Programs                      | Role |
|------------------|-------------------------------|------|
| DataService      | HABADTE, HXXAPPPRF, HXXLEVEL  | Data access and preference/hierarchy resolution. |
| WorkflowService  | HABADTE                       | Batch orchestration and record-level workflow. |
| UtilityService   | XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXGETID, XFXLEAP, XFXMRNROL | Shared utility logic (counters, dates, LDA, table-driven branching, ID generation, MRN role evaluation). |
| BatchService     | HABADTE                       | Batch driver for patient/account processing. |

---

## Section 6 — External Interfaces

### 6.1 Inbound Interfaces from Missing Sources

High-impact nodes marked as MISSING_SOURCE:

| Source    | Type   | Used By | Purpose |
|-----------|--------|--------|---------|
| TAPIRNK   | PF     | HAPIRNK| Rank/priority input for accounts. |
| TMPMAST   | PF     | HMLMAST5H | Master patient input subset. |
| TXPBNFIT  | PF     | HXPBNFIT | Benefit definition input. |
| TXPNSTN   | PF     | HXPNSTN | Institution/location input. |
| ****HXPXML| PF     | HABADTE | XML layout/input definitions. |
| PRINTER   | PF     | HABADTE | Spool output; printer interface. |

### 6.2 Outbound Interfaces from SPOOL Nodes

| Target   | Type | Produced By | Purpose |
|----------|------|------------|---------|
| PRINTER  | PF   | HABADTE    | Printed reports or statements based on processed records. |
| HXPXMLR  | PF   | HABADTE    | XML messages for downstream systems. |

---

## Section 7 — Performance and Security Notes

### 7.1 Top Programs by Cyclomatic Complexity

Complexity scores indicate relative risk and performance sensitivity.

| Program  | Cyclomatic Complexity | Notes |
|----------|-----------------------|-------|
| HABADTE  | High (hotspot score 38, fan-out 13) | Main orchestrator with many branches and calls. |
| XFXCYMD  | Medium                | Date validation logic with multiple range checks. |
| XFXTABL  | Medium                | Table-driven branching with indicator checks. |
| XFXLDSC  | Medium                | LDA map validations. |
| XFXCNTR  | Low/Medium            | Simple range-based branching. |

### 7.2 PHI Access Control Summary

- PHI fields appear in HAPTRFR, OXPBNFIT, OMPMAST, HXPDICT, and OAPIRNK.
- Access to these PFs must be restricted via:
  - Row-level security (e.g., institution or org-level scoping).
  - Column-level masking for identifiers (MRN, account, SSN, phone, name).
  - Audit logging for all access paths in the migrated system.

### 7.3 Tech Debt Totals by Category

Based on tech_debt_findings.json:

| Category              | Hours | Notes |
|-----------------------|-------|-------|
| Dead Code / Orphans   | 0     | No explicit tech debt recorded; orphan programs are structurally isolated but not flagged as debt. |
| Complexity Hotspots   | 0     | No formal debt entries despite HABADTE hotspot. |
| PHI Handling          | 0     | PHI is recognized but not yet recorded as tech debt. |
| Configuration Cleanup | 0     | Table-driven configuration appears consistent. |

Total tech debt hours: 0 (no findings recorded). For migration planning, additional manual tech debt assessment is recommended.
