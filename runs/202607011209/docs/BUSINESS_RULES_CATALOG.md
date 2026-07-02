# Business Rules Catalog – HABADTE Project

This catalog lists all approved business rules identified in the legacy AS400 codebase, grouped by domain. Each entry includes technical origin and an inferred business impact for use in testing and migration.

---

## 1. DATA_MAINTENANCE Domain

### BR-001 – Counter Lower Bound

- **Rule Text:** When X equals zero, branch to 'EXIT'.
- **Source Program:** XFXCNTR (RPGLE)
- **Confidence Score:** 0.47
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact (Plain English):**  
The counter X cannot be zero during normal processing. Zero is treated as a sentinel value meaning "stop processing" or "no iterations required". If this rule is removed or changed, loops may run unexpectedly or process records when they should terminate.

**Test Requirement:** Yes (confidence < 0.7)

---

### BR-002 – Counter Upper Bound

- **Rule Text:** When X equals 40, branch to 'EXIT'.
- **Source Program:** XFXCNTR (RPGLE)
- **Confidence Score:** 0.4
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
The counter X is capped at 39; when it reaches 40, processing exits. This protects the system from processing an unexpectedly large set of iterations or records. Incorrect implementation could lead to performance issues or unbounded loops.

**Test Requirement:** Yes

---

### BR-003 – Year Below Minimum

- **Rule Text:** When VYY is less than 1800, branch to 'EXIT'.
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Any year before 1800 is considered invalid. This rule prevents malformed or default dates from entering date-sensitive logic. If mis-implemented, records with nonsense years may pass validation and distort reporting.

**Test Requirement:** Yes

---

### BR-004 – Year Above Maximum

- **Rule Text:** When VYY is greater than 2100, branch to 'EXIT'.
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Future years beyond 2100 are treated as invalid, preventing data entry errors and protecting long-range date calculations. If omitted, wildly future dates could corrupt schedules or projections.

**Test Requirement:** Yes

---

### BR-005 – Month Below Minimum

- **Rule Text:** When VMM is less than 01, branch to 'EXIT'.
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Months lower than 1 (January) are rejected as invalid. This rule blocks malformed dates and enforces calendar semantics.

**Test Requirement:** Yes

---

### BR-006 – Month Above Maximum

- **Rule Text:** When VMM is greater than 12, branch to 'EXIT'.
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Months above 12 are invalid. Allowing such values would break downstream month-based logic (billing cycles, reports).

**Test Requirement:** Yes

---

### BR-007 – Day Below Minimum

- **Rule Text:** When VDD is less than 01, branch to 'EXIT'.
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Days lower than 1 are invalid, preventing malformed date entries.

**Test Requirement:** Yes

---

### BR-008 – Day Above Month Maximum

- **Rule Text:** When VDD is greater than DYS(VMM), branch to 'EXIT'.
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.68
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
The day value cannot exceed the number of days in the given month (and implicitly year, via leap-year logic). This ensures dates like 31 February are rejected.

**Test Requirement:** Yes

---

### BR-009 – LDAMAP Upper Bound (99)

- **Rule Text:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Map identifiers above 99 are treated as invalid or out-of-range. This reflects a design assumption that only 0–99 descriptor maps are supported. Violating this may send descriptors to undefined tables.

**Test Requirement:** Yes

---

### BR-010 – LDAMAP Upper Bound (99) – Variant 2

- **Rule Text:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Reinforces the same constraint across a different code path or branch. Ensures consistency of LDAMAP validation.

**Test Requirement:** Yes

---

### BR-011 – LDAMAP Upper Bound (99) – Variant 3

- **Rule Text:** When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Duplicate of BR-009/BR-010 in another branch; ensures no path bypasses the LDAMAP limit.

**Test Requirement:** Yes

---

### BR-012 – LDAMAP Upper Bound (9999)

- **Rule Text:** When LDAMAP is greater than 9999, branch to 'EXIT'.
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Defines a hard upper bound at 9999 for descriptor maps, possibly for high-level configuration or extended maps. Protects the system from misconfigured map codes.

**Test Requirement:** Yes

---

### BR-013 – Table Lookup Guard (Indicator On)

- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.9
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Indicator *IN79 acts as a global guard. When active, specific table lookup logic is disabled or short-circuited, preventing optional or risky lookups from running.

**Test Requirement:** No (high confidence)

---

### BR-014 – Table Lookup Guard – Variant 2

- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.9
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Ensures that a second branch also respects the indicator guard; may correspond to a different table or subtype of lookup.

**Test Requirement:** No

---

### BR-015 – Table Lookup Guard – Variant 3

- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.9
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Duplicate guard for another section of XFXTABL, maintaining a consistent disable mechanism.

**Test Requirement:** No

---

### BR-016 – Table Lookup Guard – Variant 4

- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.9
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false

**Business Impact:**  
Fourth variant ensuring that all table lookup phases honor the same global switch. Changing this behavior could unintentionally re-enable disabled lookups.

**Test Requirement:** No

---

## 2. PATIENT_MANAGEMENT Domain

### BR-017 – File Indicator Skip

- **Rule Text:** When -FILE INDICATOR equals zero, branch to 'SKIP'.
- **Source Program:** HABADTE (RPGLE)
- **Confidence Score:** 0.92
- **Domain:** PATIENT_MANAGEMENT
- **PHI in Path:** false

**Business Impact:**  
Records marked with a file indicator of zero are treated as not ready or not applicable for reporting. This prevents incomplete or suppressed records from entering inpatient transfer outputs.

**Test Requirement:** No

---

### BR-018 – Void/Void Flag Skip

- **Rule Text:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- **Source Program:** HABADTE (RPGLE)
- **Confidence Score:** 0.99
- **Domain:** PATIENT_MANAGEMENT
- **PHI in Path:** false

**Business Impact:**  
Voided transfer records are excluded from downstream outputs, ensuring that only active, non-reversed movements appear in patient tracking and integration feeds.

**Test Requirement:** No

---

### BR-019 – Outpatient Flag Skip

- **Rule Text:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.
- **Source Program:** HABADTE (RPGLE)
- **Confidence Score:** 0.99
- **Domain:** PATIENT_MANAGEMENT
- **PHI in Path:** false

**Business Impact:**  
Outpatient records are not included in the inpatient transfer batch. This maintains a clear separation between inpatient bed movements and outpatient encounters, avoiding mixed-domain reporting.

**Test Requirement:** No

---

## 3. Cross-Domain Testing Guidance

For modernization, the following cross-cutting scenarios should be explicitly tested:

1. **Date Validation** (BR-003–BR-008)  
   - Verify that invalid dates (year < 1800 or > 2100, month < 1 or > 12, day outside valid range) are rejected consistently across all services.

2. **Descriptor Map Integrity** (BR-009–BR-012)  
   - Ensure that LDAMAP values outside allowed ranges cause safe termination of descriptor logic without corrupting outputs.

3. **Table Lookup Guard Behavior** (BR-013–BR-016)  
   - Confirm that indicator-based guards correctly disable or enable lookups, and that toggling *IN79 changes behavior in a controlled manner.

4. **Inpatient Transfer Filtering** (BR-017–BR-019)  
   - Validate that file, void, and inpatient/outpatient flags produce expected inclusion/exclusion behavior and accurate summary counts.

This catalog should be used as a baseline for automated tests and acceptance criteria in the Java/Spring Boot modernization.
