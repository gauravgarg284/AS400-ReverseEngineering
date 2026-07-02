# Business Rules Catalog – HABADTE Project

This catalog lists all approved business rules extracted from the AS400 HABADTE ecosystem.
Each rule is grouped by domain and annotated with business impact and testing requirements for the Java / Spring Boot / SQL Server migration.

---

## 1. DATA_MAINTENANCE Domain

### BR-001 – Counter Equals Zero Exit

- **Rule ID:** BR-001
- **Source Program:** XFXCNTR
- **Confidence Score:** 0.47
- **PHI in Path:** false
- **Rule Text:** When X equals zero, branch to `EXIT`.

**Business Impact (Plain English):**
When the counter variable `X` is zero, the routine stops processing. This typically indicates that there are no items left to process or the loop has reached its lower bound.
Ensuring this exit behavior avoids unnecessary iterations and prevents attempts to process non-existent records.

**Test Requirement:** Yes (confidence < 0.7).

---

### BR-002 – Counter Equals Upper-Bound Exit

- **Rule ID:** BR-002
- **Source Program:** XFXCNTR
- **Confidence Score:** 0.40
- **PHI in Path:** false
- **Rule Text:** When X equals 40, branch to `EXIT`.

**Business Impact:**
The counter stops processing once `X` hits 40, enforcing an upper limit on how many items the loop will process.
This protects the batch from overruns and indicates a business rule that only up to 40 records (or units) are considered per cycle.

**Test Requirement:** Yes (confidence < 0.7).

---

### BR-003 – Year Less Than 1800 Exit

- **Rule ID:** BR-003
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `VYY` is less than 1800, branch to `EXIT`.

**Business Impact:**
Dates with year earlier than 1800 are treated as invalid and excluded from further processing.
The system enforces a minimum historical boundary, ensuring data quality for reports and downstream analytics.

**Test Requirement:** Yes (confidence < 0.7).

---

### BR-004 – Year Greater Than 2100 Exit

- **Rule ID:** BR-004
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `VYY` is greater than 2100, branch to `EXIT`.

**Business Impact:**
Future dates beyond year 2100 are rejected as invalid, helping avoid erroneous future-dated transactions.
This maintains realistic date ranges and shields reporting from out-of-range anomalies.

**Test Requirement:** Yes.

---

### BR-005 – Month Less Than 01 Exit

- **Rule ID:** BR-005
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `VMM` is less than 01, branch to `EXIT`.

**Business Impact:**
Month values below 1 are impossible and must be filtered out.
This rule ensures that malformed month values do not propagate into scheduling or billing logic.

**Test Requirement:** Yes.

---

### BR-006 – Month Greater Than 12 Exit

- **Rule ID:** BR-006
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `VMM` is greater than 12, branch to `EXIT`.

**Business Impact:**
Month values above 12 are invalid and excluded from processing.
Together with BR-005, the rule enforces strict calendar month boundaries.

**Test Requirement:** Yes.

---

### BR-007 – Day Less Than 01 Exit

- **Rule ID:** BR-007
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `VDD` is less than 01, branch to `EXIT`.

**Business Impact:**
Day values below 1 are invalid and removed from the processing stream.
This prevents nonsensical dates from affecting admission/discharge calculations.

**Test Requirement:** Yes.

---

### BR-008 – Day Greater Than Month-Max Exit

- **Rule ID:** BR-008
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.68
- **PHI in Path:** false
- **Rule Text:** When `VDD` is greater than `DYS(VMM)`, branch to `EXIT`.

**Business Impact:**
This rule ensures that the day value does not exceed the maximum allowed for the given month (including leap-year handling through DYS).
It protects downstream processes from invalid calendar dates.

**Test Requirement:** Yes (confidence < 0.7).

---

### BR-009 – LDAMAP Greater Than 99 Exit (Variant 1)

- **Rule ID:** BR-009
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `LDAMAP` is greater than 99, branch to `EXIT`.

**Business Impact:**
Mapping codes above 99 are treated as unsupported or invalid.
The rule constrains valid layouts or configuration codes to a defined range, avoiding unknown mappings.

**Test Requirement:** Yes.

---

### BR-010 – LDAMAP Greater Than 99 Exit (Variant 2)

- **Rule ID:** BR-010
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `LDAMAP` is greater than 99, branch to `EXIT`.

**Business Impact:**
This is a duplicate/variant of BR-009, reinforcing the same upper bound on `LDAMAP`.
Its presence suggests multiple code paths share the same constraint.

**Test Requirement:** Yes.

---

### BR-011 – LDAMAP Greater Than 99 Exit (Variant 3)

- **Rule ID:** BR-011
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `LDAMAP` is greater than 99, branch to `EXIT`.

**Business Impact:**
Additional reinforcement of the mapping upper limit, potentially in separate routines.
Ensures no mapping configuration drifts beyond the supported range.

**Test Requirement:** Yes.

---

### BR-012 – LDAMAP Greater Than 9999 Exit

- **Rule ID:** BR-012
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56
- **PHI in Path:** false
- **Rule Text:** When `LDAMAP` is greater than 9999, branch to `EXIT`.

**Business Impact:**
Defines a hard maximum mapping code beyond four digits.
Protects against extreme or corrupt mapping codes entering hierarchy resolution logic.

**Test Requirement:** Yes.

---

### BR-013 – Indicator *IN79 Active Exit (Variant 1)

- **Rule ID:** BR-013
- **Source Program:** XFXTABL
- **Confidence Score:** 0.90
- **PHI in Path:** false
- **Rule Text:** When `*IN79` equals on/active, branch to `EXIT`.

**Business Impact:**
When control indicator `*IN79` is active, table maintenance or processing should immediately stop.
This flag likely represents a system-wide disable switch for a specific batch.

**Test Requirement:** No (confidence ≥ 0.7).

---

### BR-014 – Indicator *IN79 Active Exit (Variant 2)

- **Rule ID:** BR-014
- **Source Program:** XFXTABL
- **Confidence Score:** 0.90
- **PHI in Path:** false
- **Rule Text:** When `*IN79` equals on/active, branch to `EXIT`.

**Business Impact:**
Additional code path respecting the same indicator, ensuring all relevant routines shut down when `*IN79` is ON.

**Test Requirement:** No.

---

### BR-015 – Indicator *IN79 Active Exit (Variant 3)

- **Rule ID:** BR-015
- **Source Program:** XFXTABL
- **Confidence Score:** 0.90
- **PHI in Path:** false
- **Rule Text:** When `*IN79` equals on/active, branch to `EXIT`.

**Business Impact:**
Further reinforcement of the shutdown behavior tied to `*IN79`.
Guarantees no processing continues when the flag is asserted.

**Test Requirement:** No.

---

### BR-016 – Indicator *IN79 Active Exit (Variant 4)

- **Rule ID:** BR-016
- **Source Program:** XFXTABL
- **Confidence Score:** 0.90
- **PHI in Path:** false
- **Rule Text:** When `*IN79` equals on/active, branch to `EXIT`.

**Business Impact:**
This final variant reflects multiple sections of XFXTABL that honor the same indicator.
The business meaning is a consistent global kill-switch for certain table operations.

**Test Requirement:** No.

---

## 2. PATIENT_MANAGEMENT Domain

### BR-017 – File Indicator Equals Zero Skip

- **Rule ID:** BR-017
- **Source Program:** HABADTE
- **Confidence Score:** 0.92
- **PHI in Path:** false
- **Rule Text:** When `-FILE INDICATOR` equals zero, branch to `SKIP`.

**Business Impact:**
Records flagged with a zero file indicator are considered invalid or inactive and must not participate in patient-management processing.
This rule prevents incomplete or uninitialized records from affecting counts, invoices, or clinical dashboards.

**Test Requirement:** No (confidence ≥ 0.7).

---

### BR-018 – Void Indicator Skip

- **Rule ID:** BR-018
- **Source Program:** HABADTE
- **Confidence Score:** 0.99
- **PHI in Path:** false
- **Rule Text:** When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.

**Business Impact:**
Any record marked as void or voided is excluded from further processing.
This ensures that corrected or cancelled encounters do not appear in active patient views or billing calculations.

**Test Requirement:** No.

---

### BR-019 – Outpatient Flag Skip

- **Rule ID:** BR-019
- **Source Program:** HABADTE
- **Confidence Score:** 0.99
- **PHI in Path:** false
- **Rule Text:** When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.

**Business Impact:**
Outpatient encounters are filtered out of HABADTE’s inpatient-oriented processing stream.
This rule keeps inpatient workflows focused on admitted stays, while outpatient activity is handled elsewhere.

**Test Requirement:** No.

---

## 3. Testing Strategy Overview

Rules with confidence scores below 0.7 (BR-001–BR-012 and BR-008 specifically) require explicit test coverage:

- Unit tests to validate conditional logic and boundary values (e.g., year, month, day, mapping thresholds, counter limits).
- Integration tests against migrated tables to ensure SQL translations reflect the exit/skip behavior of the original RPG programs.

High-confidence rules (BR-013–BR-019) still need regression tests but can be prioritized lower, focusing on critical patient filters and control indicators.
