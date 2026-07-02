# Business Rules Catalog – HABADTE Run 202607011209

## Overview

This catalog lists all approved business rules extracted from the AS400 HABADTE application and related utility programs. Rules are grouped by domain and annotated with business impact and testing guidance.

---

## Domain: DATA_MAINTENANCE

### BR-001

- **Rule Text**: When X equals zero, branch to `EXIT`.
- **Source Program**: XFXCNTR
- **Confidence Score**: 0.47
- **PHI in Path**: false
- **Business Impact**: Prevents processing when a counter or control value is at zero, avoiding meaningless or duplicate operations.
- **Test Requirement**: Yes (confidence < 0.7). Verify that X=0 causes immediate exit and X>0 continues processing.

### BR-002

- **Rule Text**: When X equals 40, branch to `EXIT`.
- **Source Program**: XFXCNTR
- **Confidence Score**: 0.40
- **PHI in Path**: false
- **Business Impact**: Establishes an upper boundary for the counter; likely used to limit the number of iterations or segments.
- **Test Requirement**: Yes. Validate X=40 exits and X<40 continues; check behavior for X>40.

### BR-003

- **Rule Text**: When VYY is less than 1800, branch to `EXIT`.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Rejects dates with years earlier than 1800, ensuring the system does not process implausible historical dates.
- **Test Requirement**: Yes. Ensure years <1800 are rejected and years >=1800 are considered.

### BR-004

- **Rule Text**: When VYY is greater than 2100, branch to `EXIT`.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Rejects dates beyond the year 2100, preventing future-date anomalies.
- **Test Requirement**: Yes. Confirm that years >2100 are rejected while near-future dates are accepted.

### BR-005

- **Rule Text**: When VMM is less than 01, branch to `EXIT`.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Ensures month values below 1 are treated as invalid.
- **Test Requirement**: Yes. Validate behavior for VMM=0 and VMM=1.

### BR-006

- **Rule Text**: When VMM is greater than 12, branch to `EXIT`.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Prevents months greater than 12, enforcing calendar validity.
- **Test Requirement**: Yes. Ensure months 13+ are rejected.

### BR-007

- **Rule Text**: When VDD is less than 01, branch to `EXIT`.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Validates that day-of-month is at least 1.
- **Test Requirement**: Yes. Confirm VDD=0 rejects and VDD>=1 proceeds.

### BR-008

- **Rule Text**: When VDD is greater than DYS(VMM), branch to `EXIT`.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.68
- **PHI in Path**: false
- **Business Impact**: Rejects days beyond the maximum for the given month (accounting for month length and, likely, leap years).
- **Test Requirement**: Yes. Test edge cases like 31 April and 29 February on non-leap years.

### BR-009

- **Rule Text**: When LDAMAP is greater than 99, branch to `EXIT`.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Prohibits level or mapping codes above 99, suggesting a constrained code set.
- **Test Requirement**: Yes. Validate that LDAMAP>99 triggers exit and <=99 is accepted.

### BR-010

- **Rule Text**: When LDAMAP is greater than 99, branch to `EXIT`.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Reinforces the same upper bound; may apply to a different field or context within the same program.
- **Test Requirement**: Yes. Confirm no scenarios bypass this check.

### BR-011

- **Rule Text**: When LDAMAP is greater than 99, branch to `EXIT`.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Similar to BR-009 and BR-010, ensuring mapping ranges stay within defined limits.
- **Test Requirement**: Yes. Regression tests should cover all LDAMAP comparisons.

### BR-012

- **Rule Text**: When LDAMAP is greater than 9999, branch to `EXIT`.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **PHI in Path**: false
- **Business Impact**: Introduces a broader upper limit (9999), possibly for a different category of mappings.
- **Test Requirement**: Yes. Verify that extreme values beyond 9999 are rejected.

### BR-013

- **Rule Text**: When *IN79 equals on/active, branch to `EXIT`.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **PHI in Path**: false
- **Business Impact**: Uses indicator *IN79 to short-circuit table processing when a desired condition is met or an error occurs.
- **Test Requirement**: No (confidence >= 0.7). Include indicator scenarios in integration tests.

### BR-014

- **Rule Text**: When *IN79 equals on/active, branch to `EXIT`.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **PHI in Path**: false
- **Business Impact**: Likely mirrors BR-013 for a different loop or section in the same program.
- **Test Requirement**: No. Covered if BR-013 paths are tested.

### BR-015

- **Rule Text**: When *IN79 equals on/active, branch to `EXIT`.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **PHI in Path**: false
- **Business Impact**: Ensures that once a termination condition is reached, the program does not continue scanning table entries.
- **Test Requirement**: No. Include as part of full-table processing tests.

### BR-016

- **Rule Text**: When *IN79 equals on/active, branch to `EXIT`.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **PHI in Path**: false
- **Business Impact**: Consolidates indicator-based control for table-processing flows.
- **Test Requirement**: No.

---

## Domain: PATIENT_MANAGEMENT

### BR-017

- **Rule Text**: When `-FILE INDICATOR` equals zero, branch to `SKIP`.
- **Source Program**: HABADTE
- **Confidence Score**: 0.92
- **PHI in Path**: false
- **Business Impact**: Ensures only valid, fully initialized records are processed. Records with missing or zero file indicator are skipped to avoid corrupt output.
- **Test Requirement**: No (confidence >= 0.7). Functional tests must confirm that records with indicator=0 are omitted from XML and API results.

### BR-018

- **Rule Text**: When `-FLAG INDICATOR` equals void/voided, branch to `SKIP`.
- **Source Program**: HABADTE
- **Confidence Score**: 0.99
- **PHI in Path**: false
- **Business Impact**: Prevents voided or reversed transactions from appearing in downstream reports and extracts, aligning with financial and clinical integrity requirements.
- **Test Requirement**: No. Integration tests should validate that voided transfers never appear in output datasets.

### BR-019

- **Rule Text**: When `-INPATIENT/OUTPATIENT FLAG` equals outpatient, branch to `SKIP`.
- **Source Program**: HABADTE
- **Confidence Score**: 0.99
- **PHI in Path**: false
- **Business Impact**: Restricts the HABADTE flow to inpatient records only. Outpatient transactions are intentionally excluded from this pathway.
- **Test Requirement**: No. Scenario tests should demonstrate that mixed inpatient/outpatient data sets produce only inpatient records.

---

## Summary

- Total approved rules: 19
- Domains covered: DATA_MAINTENANCE, PATIENT_MANAGEMENT
- Rules with mandatory dedicated tests (confidence < 0.7): BR-001–BR-012
- High-confidence rules suitable for reuse with regression coverage: BR-013–BR-019

This catalog forms the basis for migration test cases, configuration validation, and audit documentation for the HABADTE modernization effort.
