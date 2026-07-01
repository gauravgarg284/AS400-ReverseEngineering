# Business Rules Catalog

This catalog enumerates the approved business rules extracted from the HABADTE project, grouped by domain.

## Domain: DATA_MAINTENANCE

### BR-001

- **Rule Text**: When X equals zero, branch to 'EXIT'.
- **Source Program**: XFXCNTR
- **Confidence Score**: 0.47
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Prevents processing when control value X is uninitialized or indicates no action, avoiding unnecessary downstream logic.
- **Test Required**: Yes (confidence < 0.7)

### BR-002

- **Rule Text**: When X equals 40, branch to 'EXIT'.
- **Source Program**: XFXCNTR
- **Confidence Score**: 0.40
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Treats a specific control value of 40 as a terminal condition, likely representing an out-of-range or sentinel value.
- **Test Required**: Yes (confidence < 0.7)

### BR-003

- **Rule Text**: When VYY is less than 1800, branch to 'EXIT'.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Ensures year values lower than 1800 are treated as invalid and not processed, maintaining date data quality.
- **Test Required**: Yes (confidence < 0.7)

### BR-004

- **Rule Text**: When VYY is greater than 2100, branch to 'EXIT'.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Ensures year values greater than 2100 are treated as invalid, preventing future-dated or corrupt dates from being used.
- **Test Required**: Yes (confidence < 0.7)

### BR-005

- **Rule Text**: When VMM is less than 01, branch to 'EXIT'.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Rejects invalid month values below 1, ensuring proper date construction.
- **Test Required**: Yes (confidence < 0.7)

### BR-006

- **Rule Text**: When VMM is greater than 12, branch to 'EXIT'.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Rejects invalid month values above 12, preventing malformed dates.
- **Test Required**: Yes (confidence < 0.7)

### BR-007

- **Rule Text**: When VDD is less than 01, branch to 'EXIT'.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Rejects invalid day values below 1, maintaining date integrity.
- **Test Required**: Yes (confidence < 0.7)

### BR-008

- **Rule Text**: When VDD is greater than DYS(VMM), branch to 'EXIT'.
- **Source Program**: XFXCYMD
- **Confidence Score**: 0.68
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Ensures day values do not exceed the number of days in the given month, incorporating leap-year logic and preventing invalid dates.
- **Test Required**: Yes (confidence < 0.7)

### BR-009

- **Rule Text**: When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Protects against mapping codes exceeding configured ranges, likely preventing unrecognized level descriptors.
- **Test Required**: Yes (confidence < 0.7)

### BR-010

- **Rule Text**: When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Duplicate of BR-009 in effect; ensures mapping code upper bounds are enforced consistently.
- **Test Required**: Yes (confidence < 0.7)

### BR-011

- **Rule Text**: When LDAMAP is greater than 99, branch to 'EXIT'.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Another repeated check aligning with BR-009/BR-010; reinforces consistency across branches.
- **Test Required**: Yes (confidence < 0.7)

### BR-012

- **Rule Text**: When LDAMAP is greater than 9999, branch to 'EXIT'.
- **Source Program**: XFXLDSC
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Provides a higher-level safeguard for extended mapping ranges, ensuring that outlier codes are not processed.
- **Test Required**: Yes (confidence < 0.7)

### BR-013

- **Rule Text**: When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Uses a status indicator (*IN79) to terminate processing when a specific condition or override is active, preventing unintended updates.
- **Test Required**: No (confidence >= 0.7)

### BR-014

- **Rule Text**: When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Functionally equivalent to BR-013; likely guards another branch or section of the same program.
- **Test Required**: No (confidence >= 0.7)

### BR-015

- **Rule Text**: When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Additional repetition of the *IN79 control pattern, emphasizing the importance of this indicator in table maintenance flows.
- **Test Required**: No (confidence >= 0.7)

### BR-016

- **Rule Text**: When *IN79 equals on/active, branch to 'EXIT'.
- **Source Program**: XFXTABL
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **PHI in Path**: false
- **Business Impact**: Reinforces the consistent application of the *IN79 control flag across multiple branches.
- **Test Required**: No (confidence >= 0.7)

## Domain: PATIENT_MANAGEMENT

### BR-017

- **Rule Text**: When -FILE INDICATOR equals zero, branch to 'SKIP'.
- **Source Program**: HABADTE
- **Confidence Score**: 0.92
- **Domain**: PATIENT_MANAGEMENT
- **PHI in Path**: false
- **Business Impact**: Ensures that records with inactive or null file indicators are not processed, preventing noise and incorrect patient activity calculations.
- **Test Required**: No (confidence >= 0.7)

### BR-018

- **Rule Text**: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.
- **Source Program**: HABADTE
- **Confidence Score**: 0.99
- **Domain**: PATIENT_MANAGEMENT
- **PHI in Path**: false
- **Business Impact**: Enforces exclusion of voided patient transactions, aligning with financial and clinical expectations for accurate reporting.
- **Test Required**: No (confidence >= 0.7)

### BR-019

- **Rule Text**: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.
- **Source Program**: HABADTE
- **Confidence Score**: 0.99
- **Domain**: PATIENT_MANAGEMENT
- **PHI in Path**: false
- **Business Impact**: Ensures that only inpatient records are included in specific HABADTE outputs, avoiding contamination of inpatient-only analytics with outpatient data.
- **Test Required**: No (confidence >= 0.7)
