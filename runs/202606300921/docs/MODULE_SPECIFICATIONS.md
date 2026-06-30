# Module Specifications — HABADTE

This document provides per-program specifications derived from the source manifest, complexity metrics, and business rule interpretations.

## 1. HABADTE (Main Orchestrator)

### 1.1 Header

- Program Name: HABADTE
- Type: RPGLE
- Subsystem: PATIENT_MANAGEMENT
- Lines of Code: ~ (from manifest; part of 9,153 total)
- Cyclomatic Complexity: High (hotspot score 38, fan-out 13)
- Risk Level: High (core PHI processing, complex branching)
- Migration Difficulty Score: High

### 1.2 F-Spec Summary

| File       | Mode    | Device | Keyed | KLIST |
|------------|---------|--------|-------|-------|
| HAPTRFR    | DISK    | Disk   | Yes   | AFACCT/AFMRNO |
| OMPMAST    | DISK    | Disk   | Yes   | MMMRNO/MMACCT |
| OAPIRNK    | DISK    | Disk   | Yes   | Rank/account |
| OXPBNFIT   | DISK    | Disk   | Yes   | Benefit code |
| OXPNSTN    | DISK    | Disk   | Yes   | Institution code |
| HXPDICT    | DISK    | Disk   | Yes   | CCMRNO/HXRMNO |
| HXPXMLD    | DISK    | Disk   | Yes   | XMLID |
| HXPXMLR    | DISK    | Disk   | Yes   | XMLID |
| HXPTABLD   | DISK    | Disk   | Yes   | TBLID/KEYVAL |
| HXPLVL1–6  | DISK    | Disk   | Yes   | LVLKEY |
| HAPIRNK    | DISK    | Disk   | Yes   | Rank/account |
| HMLMAST5H  | DISK    | Disk   | Yes   | MRN/account |
| HXLTABLS   | DISK    | Disk   | Yes   | TBLID/KEYVAL |
| HXLTABLP   | DISK    | Disk   | Yes   | TBLID/KEYVAL/level |
| HXLTABLD   | DISK    | Disk   | Yes   | TBLID/KEYVAL |
| PRINTER    | PRINTER | Printer| No    | —     |

### 1.3 Key Subroutines

| Name         | Fan-In | Fan-Out | Complexity | Purpose |
|--------------|--------|--------|-----------|---------|
| INIT_CONTEXT | 1      | 5      | Medium    | Initializes LDA, preferences, and hierarchy context. |
| REC_FILTER   | 1      | 0      | Medium    | Applies BR-017–BR-019 to determine whether a record should be processed or skipped. |
| PROCESS_REC  | 1      | 10     | High      | Performs main per-record processing, calling XFX* utilities and updating outputs. |
| WRITE_XML    | 1      | 1      | Low/Med   | Writes XML messages to HXPXMLR based on processed data. |
| WRITE_SPOOL  | 1      | 1      | Low/Med   | Writes formatted output to PRINTER. |

### 1.4 Business Rules Owned

| BR-ID  | Description |
|--------|-------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. |

### 1.5 Tech Debt Items

No explicit tech debt entries are recorded. Migration team should consider:

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-HAB-001 | High — Complex branching and PHI handling; refactor into layered services. | 40 |
| TD-HAB-002 | Medium — Printer output logic; redesign for modern reporting. | 16 |

---

## 2. XFXCNTR (Counter Utility)

### 2.1 Header

- Program Name: XFXCNTR
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE
- Lines of Code: Small utility
- Cyclomatic Complexity: Low/Medium
- Risk Level: Low
- Migration Difficulty Score: Low

### 2.2 F-Spec Summary

| File      | Mode | Device | Keyed | KLIST |
|-----------|------|--------|-------|-------|
| HXPTABLD  | DISK | Disk   | Yes   | TBLID/KEYVAL |

### 2.3 Key Subroutines

| Name    | Fan-In | Fan-Out | Complexity | Purpose |
|---------|--------|--------|-----------|---------|
| CNTCHK  | 2+     | 0      | Low/Med   | Validates counter values and branches to EXIT when out of range. |

### 2.4 Business Rules Owned

| BR-ID  | Description |
|--------|-------------|
| BR-001 | When X equals zero, branch to 'EXIT'. |
| BR-002 | When X equals 40, branch to 'EXIT'. |

### 2.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-001 | Low — Convert range checks to parameterized validation in shared utility library. | 4 |

---

## 3. XFXCYMD (Date Utility)

### 3.1 Header

- Program Name: XFXCYMD
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE
- Cyclomatic Complexity: Medium
- Risk Level: Medium
- Migration Difficulty Score: Medium

### 3.2 F-Spec Summary

| File     | Mode | Device | Keyed | KLIST |
|----------|------|--------|-------|-------|
| HXPTABLD | DISK | Disk   | Yes   | TBLID/KEYVAL |

### 3.3 Key Subroutines

| Name     | Fan-In | Fan-Out | Complexity | Purpose |
|----------|--------|--------|-----------|---------|
| DATECHK  | 2+     | 0      | Medium    | Validates CYMD date values against allowed ranges. |

### 3.4 Business Rules Owned

| BR-ID  | Description |
|--------|-------------|
| BR-003 | When VYY is less than 1800, branch to 'EXIT'. |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'. |
| BR-005 | When VMM is less than 01, branch to 'EXIT'. |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'. |
| BR-007 | When VDD is less than 01, branch to 'EXIT'. |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. |

### 3.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-002 | Medium — Replace custom date validation with standard Java date libraries and unit tests. | 8 |

---

## 4. XFXLDSC (LDA Map Utility)

### 4.1 Header

- Program Name: XFXLDSC
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE
- Cyclomatic Complexity: Medium
- Risk Level: Low/Medium

### 4.2 F-Spec Summary

| File  | Mode | Device | Keyed | KLIST |
|-------|------|--------|-------|-------|
| *LDA  | N/A  | Data Area | N/A | N/A |

### 4.3 Key Subroutines

| Name      | Fan-In | Fan-Out | Complexity | Purpose |
|-----------|--------|--------|-----------|---------|
| LDAMAPCHK | 2+     | 0      | Medium    | Validates LDAMAP value and branches to EXIT when invalid. |

### 4.4 Business Rules Owned

| BR-ID  | Description |
|--------|-------------|
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'. |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'. |

### 4.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-003 | Low — Consolidate duplicate LDAMAP checks and document LDA usage. | 4 |

---

## 5. XFXTABL (Table-Driven Branch Utility)

### 5.1 Header

- Program Name: XFXTABL
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE
- Cyclomatic Complexity: Medium
- Risk Level: Medium

### 5.2 F-Spec Summary

| File     | Mode | Device | Keyed | KLIST |
|----------|------|--------|-------|-------|
| HXPTABLD | DISK | Disk   | Yes   | TBLID/KEYVAL |

### 5.3 Key Subroutines

| Name   | Fan-In | Fan-Out | Complexity | Purpose |
|--------|--------|--------|-----------|---------|
| INDCHK | 2+     | 0      | Medium    | Checks *IN79 and table entries to branch to EXIT or continue. |

### 5.4 Business Rules Owned

| BR-ID  | Description |
|--------|-------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. |

### 5.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-004 | Medium — Normalize indicator-based branching and centralize configuration. | 6 |

---

## 6. XFXGETID (ID Utility)

### 6.1 Header

- Program Name: XFXGETID
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE
- Cyclomatic Complexity: Low
- Risk Level: Low

### 6.2 F-Spec Summary

| File | Mode | Device | Keyed | KLIST |
|------|------|--------|-------|-------|
| HXPTABLD | DISK | Disk | Yes | TBLID/KEYVAL |

### 6.3 Key Subroutines

| Name  | Fan-In | Fan-Out | Complexity | Purpose |
|-------|--------|--------|-----------|---------|
| GETID | 2+     | 0      | Low       | Generates or retrieves IDs based on table entries. |

### 6.4 Business Rules Owned

No explicit BR-IDs recorded.

### 6.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-005 | Low — Ensure uniqueness and concurrency-safe ID generation in SQL Server. | 6 |

---

## 7. XFXMRNROL (MRN Role Utility)

### 7.1 Header

- Program Name: XFXMRNROL
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE

### 7.2 F-Spec Summary

| File    | Mode | Device | Keyed | KLIST |
|---------|------|--------|-------|-------|
| OMPMAST | DISK | Disk   | Yes   | MMMRNO/MMACCT |

### 7.3 Key Subroutines

| Name    | Fan-In | Fan-Out | Complexity | Purpose |
|---------|--------|--------|-----------|---------|
| MRNROLE | 2+     | 0      | Medium    | Determines MRN role/status for HABADTE processing. |

### 7.4 Business Rules Owned

No explicit BR-IDs recorded but likely influences record inclusion.

### 7.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-006 | Medium — Document MRN role semantics and ensure consistent mapping in new system. | 8 |

---

## 8. XFXLEAP (Leap Year Utility)

### 8.1 Header

- Program Name: XFXLEAP
- Type: RPGLE
- Subsystem: DATA_MAINTENANCE

### 8.2 F-Spec Summary

| File | Mode | Device | Keyed | KLIST |
|------|------|--------|-------|-------|
| None | N/A  | N/A    | N/A   | N/A   |

### 8.3 Key Subroutines

| Name    | Fan-In | Fan-Out | Complexity | Purpose |
|---------|--------|--------|-----------|---------|
| LEAPCHK | 2+     | 0      | Low       | Determines leap year status for date validation. |

### 8.4 Business Rules Owned

No explicit BR-IDs recorded.

### 8.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-XFX-007 | Low — Replace with language/library-level leap year checks. | 2 |

---

## 9. HXXAPPPRF (Application Preference Program)

### 9.1 Header

- Program Name: HXXAPPPRF
- Type: SQLRPGLE
- Subsystem: Configuration / Preferences

### 9.2 F-Spec Summary

| File     | Mode | Device | Keyed | KLIST |
|----------|------|--------|-------|-------|
| HXPTABLD | DISK | Disk   | Yes   | TBLID/KEYVAL |

### 9.3 Key Subroutines

| Name        | Fan-In | Fan-Out | Complexity | Purpose |
|-------------|--------|--------|-----------|---------|
| PREF_LOOKUP | 2+     | 0      | Medium    | Retrieves preference values with override logic. |

### 9.4 Business Rules Owned

No explicit BR-IDs recorded but preferences influence HABADTE behaviour.

### 9.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-HXX-001 | Medium — Externalize preferences into configuration service/API. | 12 |

---

## 10. HXXLEVEL (Org Level Program)

### 10.1 Header

- Program Name: HXXLEVEL
- Type: RPGLE
- Subsystem: Configuration / Hierarchy

### 10.2 F-Spec Summary

| File      | Mode | Device | Keyed | KLIST |
|-----------|------|--------|-------|-------|
| HXPLVL1–6 | DISK | Disk   | Yes   | LVLKEY |

### 10.3 Key Subroutines

| Name      | Fan-In | Fan-Out | Complexity | Purpose |
|-----------|--------|--------|-----------|---------|
| LVL_LOOKUP| 2+     | 0      | Medium    | Resolves organizational hierarchy levels. |

### 10.4 Business Rules Owned

No explicit BR-IDs recorded.

### 10.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-HXX-002 | Medium — Model hierarchy in normalized SQL tables and service layer. | 10 |

---

## 11. HXXLDA (LDA Initialization Program)

### 11.1 Header

- Program Name: HXXLDA
- Type: RPGLE
- Subsystem: Infrastructure

### 11.2 F-Spec Summary

| File | Mode | Device | Keyed | KLIST |
|------|------|--------|-------|-------|
| *LDA | N/A  | Data Area | N/A | N/A |

### 11.3 Key Subroutines

| Name   | Fan-In | Fan-Out | Complexity | Purpose |
|--------|--------|--------|-----------|---------|
| INITLDA| 2+     | 0      | Low/Med   | Initializes LDA with run context. |

### 11.4 Business Rules Owned

No explicit BR-IDs recorded.

### 11.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-HXX-003 | Medium — Replace LDA usage with explicit context objects in Java. | 8 |

---

## 12. HXXCNTRL (Control Program)

### 12.1 Header

- Program Name: HXXCNTRL
- Type: RPGLE
- Subsystem: Infrastructure / Control

### 12.2 F-Spec Summary

| File | Mode | Device | Keyed | KLIST |
|------|------|--------|-------|-------|
| HXPTABLD | DISK | Disk | Yes | TBLID/KEYVAL |

### 12.3 Key Subroutines

| Name    | Fan-In | Fan-Out | Complexity | Purpose |
|---------|--------|--------|-----------|---------|
| CTL_INIT| 2+     | 0      | Medium    | Initializes control flags and parameters for HABADTE. |

### 12.4 Business Rules Owned

No explicit BR-IDs recorded.

### 12.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-HXX-004 | Medium — Consolidate control logic into configuration and orchestration layer. | 10 |

---

## 13. HXXXML (XML Helper Program)

### 13.1 Header

- Program Name: HXXXML
- Type: RPGLE
- Subsystem: XML / Messaging

### 13.2 F-Spec Summary

| File     | Mode | Device | Keyed | KLIST |
|----------|------|--------|-------|-------|
| HXPXMLD  | DISK | Disk   | Yes   | XMLID |
| HXPXMLR  | DISK | Disk   | Yes   | XMLID |

### 13.3 Key Subroutines

| Name      | Fan-In | Fan-Out | Complexity | Purpose |
|-----------|--------|--------|-----------|---------|
| XML_BUILD | 2+     | 0      | Medium    | Constructs XML messages based on data dictionary and preferences. |
| XML_PARSE | 2+     | 0      | Medium    | Parses incoming XML into internal structures. |

### 13.4 Business Rules Owned

No explicit BR-IDs recorded.

### 13.5 Tech Debt Items

| TD-ID | Impact | Estimated Hours |
|-------|--------|-----------------|
| TD-HXX-005 | High — Replace custom XML handling with modern libraries and schemas. | 24 |

---

## 14. Summary

The HABADTE system consists of 5 interpreted programs and several utility and helper modules. Core business rules are concentrated in HABADTE and XFX* utilities. Migration should prioritize:

1. Refactoring HABADTE into layered services (controller, service, repository).
2. Converting XFX* utilities into shared service components with unit tests.
3. Externalizing preferences, hierarchy, and control data into normalized tables and configuration services.
4. Implementing robust PHI handling and auditing across all modules.
