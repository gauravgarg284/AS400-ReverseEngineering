# Business Rules Catalog

This catalog lists all approved rules extracted from the HABADTE project, grouped by domain.

---

## Domain: DATA_MAINTENANCE

### BR-001
- **Rule Text:** When X equals zero, branch to 'EXIT'.
- **Source Program:** XFXCNTR
- **Confidence Score:** 0.47
- **PHI in Path:** false
- **Business Impact (Plain English):**
  - Ensures that when the counter value is zero, the process terminates or exits the current routine, preventing unnecessary iterations or invalid state processing.
- **Test Requirement:** Yes (confidence < 0.7).

### BR-002
- **Rule Text:** When X equals 40, branch to 'EXIT'.
- **Source Program:** XFXCNTR
- **Confidence Score:** 0.4
- **PHI in Path:** false
- **Business Impact:**
  - Caps the number of iterations or records processed by a counter-driven loop at 40, preventing runaway processing and controlling batch size.
- **Test Requirement:** Yes.

### BR-003
- **Rule Text:** When VYY is less than 1800, branch to 'EXIT'.
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Rejects dates with a year before 1800, ensuring that obviously invalid historical dates are not processed or stored.
- **Test Requirement:** Yes.

### BR-004
- **Rule Text:** When VYY is greater than 2100, branch to 'EXIT'.
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Rejects dates with a year beyond 2100, protecting the system from future-dated records far outside plausible operational windows.
- **Test Requirement:** Yes.

### BR-005
- **Rule Text:** When VMM is less than 01, branch to 'EXIT'.
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Prevents processing of dates where the month value is below 1, catching malformed or default month values.
- **Test Requirement:** Yes.

### BR-006
- **Rule Text:** When VMM is greater than 12, branch to 'EXIT'.
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Stops processing when the month field exceeds 12, ensuring that only valid calendar months are accepted.
- **Test Requirement:** Yes.

### BR-007
- **Rule Text:** When VDD is less than 01, branch to 'EXIT'.
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Rejects date records with a day less than 1, catching invalid or uninitialized day values.
- **Test Requirement:** Yes.

### BR-008
- **Rule Text:** When VDD is greater than DYS(VMM), branch to 'EXIT'.
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.68
- **PHI in Path:** false
- **Business Impact:**
  - Ensures that the day value does not exceed the number of days in the given month, preventing impossible dates like 31 February.
- **Test Requirement:** Yes.

### BR-009
- **Rule Text:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Forces termination of mapping logic when LDAMAP exceeds an expected bound (99), limiting the scope of lookup maps to supported indices.
- **Test Requirement:** Yes.

### BR-010
- **Rule Text:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Duplicate or alternative path of BR-009, reinforcing upper limit control on LDAMAP-based mapping operations.
- **Test Requirement:** Yes.

### BR-011
- **Rule Text:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Additional branch enforcing the same limit as BR-009/BR-010, likely tied to different conditions or segments of the mapping routine.
- **Test Requirement:** Yes.

### BR-012
- **Rule Text:** When LDAMAP is greater than 9999, branch to 'EXIT'.
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Business Impact:**
  - Prevents extremely large or malformed mapping keys, treating LDAMAP values beyond 9999 as erroneous and terminating processing.
- **Test Requirement:** Yes.

### BR-013
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL
- **Confidence Score:** 0.9
- **PHI in Path:** false
- **Business Impact:**
  - Uses indicator *IN79 as a control flag; when it is active, the table processing routine exits, signaling that further processing is disabled or complete.
- **Test Requirement:** No (confidence >= 0.7).

### BR-014
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL
- **Confidence Score:** 0.9
- **PHI in Path:** false
- **Business Impact:**
  - A parallel branch to BR-013, likely for a different section of table handling, but enforcing the same shutdown behavior when the indicator is active.
- **Test Requirement:** No.

### BR-015
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL
- **Confidence Score:** 0.9
- **PHI in Path:** false
- **Business Impact:**
  - Ensures consistent exit behavior across multiple table-handling segments whenever *IN79 indicates an active termination condition.
- **Test Requirement:** No.

### BR-016
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL
- **Confidence Score:** 0.9
- **PHI in Path:** false
- **Business Impact:**
  - Reinforces that *IN79 is the master kill-switch for the table processing routine; any active state leads to controlled termination.
- **Test Requirement:** No.

---

## Domain: PATIENT_MANAGEMENT

### BR-017
- **Rule Text:** When -FILE INDICATOR equals zero, branch to 'SKIP'.
- **Source Program:** HABADTE
- **Confidence Score:** 0.92
- **PHI in Path:** false (although HABADTE interacts with PHI-bearing files such as HAPTRFR and OMPMAST, the rule itself is based on a non-PHI indicator).
- **Business Impact:**
  - Records marked with a zero file indicator are treated as inactive or logically deleted and are omitted from patient-account processing and XML output. This prevents inclusion of stale or placeholder records.
- **Test Requirement:** No.

### BR-018
- **Rule Text:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- **Source Program:** HABADTE
- **Confidence Score:** 0.99
- **PHI in Path:** false (void flag itself is non-PHI, though applied to PHI-bearing records).
- **Business Impact:**
  - Ensures that encounters or transactions explicitly voided are excluded from operational reporting and downstream integrations, while still available for audit. This supports financial reconciliation and patient record integrity.
- **Test Requirement:** No.

### BR-019
- **Rule Text:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.
- **Source Program:** HABADTE
- **Confidence Score:** 0.99
- **PHI in Path:** false.
- **Business Impact:**
  - Filters out outpatient encounters from this specific inpatient-focused flow. Outpatient records must be handled by separate processes, preventing mixing of care settings in admission-discharge-transfer reporting.
- **Test Requirement:** No.
