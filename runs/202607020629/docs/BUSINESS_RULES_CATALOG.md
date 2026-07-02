# BUSINESS RULES CATALOG – HABADTE Project

This catalog lists all approved business rules extracted from the AS400 HABADTE codebase, grouped by domain.

## 1. DATA_MAINTENANCE Domain

### BR-001
- **Rule Text**: When X equals zero, branch to "EXIT".
- **Source Program**: XFXCNTR
- **Confidence Score**: 0.47
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact (Plain English)**: Prevents further processing when the counter is at zero, acting as an early termination guard.
- **Test Requirement**: Yes (confidence_score < 0.7)

### BR-002
- **Rule Text**: When X equals 40, branch to "EXIT".
- **Source Program**: XFXCNTR
- **Confidence Score**: 0.40
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Caps processing at 40 iterations/events to avoid runaway loops or exceeding business thresholds.
- **Test Requirement**: Yes

### BR-003
- **Rule Text**: When VYY is less than 1800, branch to "EXIT".
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Prevents using dates earlier than year 1800, ensuring data stays within a supported calendar range.
- **Test Requirement**: Yes

### BR-004
- **Rule Text**: When VYY is greater than 2100, branch to "EXIT".
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Rejects future dates beyond 2100 to avoid unrealistic or placeholder dates.
- **Test Requirement**: Yes

### BR-005
- **Rule Text**: When VMM is less than 01, branch to "EXIT".
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Ensures month values are not below 1, catching malformed dates.
- **Test Requirement**: Yes

### BR-006
- **Rule Text**: When VMM is greater than 12, branch to "EXIT".
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Disallows month values above 12, preventing invalid calendar dates.
- **Test Requirement**: Yes

### BR-007
- **Rule Text**: When VDD is less than 01, branch to "EXIT".
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Rejects day-of-month values below 1, enforcing basic date validity.
- **Test Requirement**: Yes

### BR-008
- **Rule Text**: When VDD is greater than DYS(VMM), branch to "EXIT".
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.68
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Validates that the day-of-month does not exceed the number of days in that month (including leap year logic), preventing impossible dates.
- **Test Requirement**: Yes

### BR-009
- **Rule Text**: When LDAMAP is greater than 99, branch to "EXIT".
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Enforces a maximum mapping code of 99 for certain level configurations, keeping mapping tables within expected bounds.
- **Test Requirement**: Yes

### BR-010
- **Rule Text**: When LDAMAP is greater than 99, branch to "EXIT".
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Same as BR-009; likely applied to a different branch or mapping type but with the same upper bound constraint.
- **Test Requirement**: Yes

### BR-011
- **Rule Text**: When LDAMAP is greater than 99, branch to "EXIT".
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Duplicate enforcement of LDAMAP upper limit, ensuring no configuration path bypasses the 99 cap.
- **Test Requirement**: Yes

### BR-012
- **Rule Text**: When LDAMAP is greater than 9999, branch to "EXIT".
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Defines an absolute maximum mapping value of 9999, allowing extended mapping ranges while still preventing out-of-bounds values.
- **Test Requirement**: Yes

### BR-013
- **Rule Text**: When *IN79 equals on/active, branch to "EXIT".
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Uses indicator *IN79 to disable table lookup logic when a particular configuration condition is active, likely to short-circuit optional processing.
- **Test Requirement**: No (confidence >= 0.7)

### BR-014
- **Rule Text**: When *IN79 equals on/active, branch to "EXIT".
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Reinforces BR-013 in another branch of table logic, ensuring indicator-based bypass is consistently applied.
- **Test Requirement**: No

### BR-015
- **Rule Text**: When *IN79 equals on/active, branch to "EXIT".
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Additional enforcement of the *IN79-driven bypass, indicating multiple code paths that share the same exit condition.
- **Test Requirement**: No

### BR-016
- **Rule Text**: When *IN79 equals on/active, branch to "EXIT".
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Ensures the indicator gate is applied across all relevant branches in XFXTABL; changing this rule would impact several lookup scenarios.
- **Test Requirement**: No

## 2. PATIENT_MANAGEMENT Domain

### BR-017
- **Rule Text**: When -FILE INDICATOR equals zero, branch to "SKIP".
- **Source Program**: HABADTE
- **Confidence Score**: 0.92
- **Domain**: PATIENT_MANAGEMENT
- **PHI in Path**: false
- **Business Impact**: Prevents processing of records flagged as inactive or not ready (file indicator 0), ensuring downstream messaging only includes valid transactions.
- **Test Requirement**: No

### BR-018
- **Rule Text**: When -FLAG INDICATOR equals void/voided, branch to "SKIP".
- **Source Program**: HABADTE
- **Confidence Score**: 0.99
- **Domain**: PATIENT_MANAGEMENT
- **PHI in Path**: false
- **Business Impact**: Excludes voided transactions from patient management outputs, avoiding billing or notification for cancelled records.
- **Test Requirement**: No

### BR-019
- **Rule Text**: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to "SKIP".
- **Source Program**: HABADTE
- **Confidence Score**: 0.99
- **Domain**: PATIENT_MANAGEMENT
- **PHI in Path**: false
- **Business Impact**: Filters out outpatient transactions for this particular batch or report, focusing processing on inpatients; critical for correct operational scope.
- **Test Requirement**: No
