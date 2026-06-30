# Business Rules Catalog — HABADTE

## DATA_MAINTENANCE Domain

### XFXCNTR

| BR-ID  | Rule Title                          | Source Program | Line | Type       | Description | Business Impact | PHI Sensitive | Confidence | Status        | Test Requirement |
|--------|--------------------------------------|----------------|------|-----------|-------------|-----------------|--------------|------------|--------------|------------------|
| BR-001 | Counter equals zero → EXIT          | XFXCNTR        | 32   | validation | When X equals zero, branch to 'EXIT'. Prevents processing when counter is zero. | Low — Guards against invalid or empty counters. | No           | 0.47       | AUTO_ACCEPTED | Unit tests for X = 0 and X <> 0. |
| BR-002 | Counter equals 40 → EXIT            | XFXCNTR        | 35   | filter     | When X equals 40, branch to 'EXIT'. Stops processing at threshold 40. | Low/Medium — Limits looping or iteration count. | No           | 0.40       | AUTO_ACCEPTED | Unit tests for X = 40 and boundary values. |

### XFXCYMD

| BR-ID  | Rule Title                                      | Source Program | Line | Type   | Description | Business Impact | PHI Sensitive | Confidence | Status        | Test Requirement |
|--------|--------------------------------------------------|----------------|------|--------|-------------|-----------------|--------------|------------|--------------|------------------|
| BR-003 | Year < 1800 → EXIT                              | XFXCYMD        | 68   | filter | When VYY is less than 1800, branch to 'EXIT'. Rejects dates before 1800. | Medium — Prevents invalid historical dates. | No           | 0.56       | AUTO_ACCEPTED | Test years 1799, 1800, and valid ranges. |
| BR-004 | Year > 2100 → EXIT                              | XFXCYMD        | 69   | filter | When VYY is greater than 2100, branch to 'EXIT'. Rejects far-future dates. | Medium — Prevents unrealistic future dates. | No           | 0.56       | AUTO_ACCEPTED | Test years 2100, 2101, and boundary behaviour. |
| BR-005 | Month < 01 → EXIT                               | XFXCYMD        | 70   | filter | When VMM is less than 01, branch to 'EXIT'. Rejects month values below 1. | Medium — Ensures valid month range. | No           | 0.56       | AUTO_ACCEPTED | Test months 0, 1, and typical values. |
| BR-006 | Month > 12 → EXIT                               | XFXCYMD        | 71   | filter | When VMM is greater than 12, branch to 'EXIT'. Rejects month values above 12. | Medium — Ensures valid month range. | No           | 0.56       | AUTO_ACCEPTED | Test months 12, 13, and boundary behaviour. |
| BR-007 | Day < 01 → EXIT                                 | XFXCYMD        | 72   | filter | When VDD is less than 01, branch to 'EXIT'. Rejects day values below 1. | Medium — Ensures valid day range. | No           | 0.56       | AUTO_ACCEPTED | Test days 0, 1, and typical values. |
| BR-008 | Day > days-in-month → EXIT                      | XFXCYMD        | 73   | filter | When VDD is greater than DYS(VMM), branch to 'EXIT'. Rejects days beyond month length. | High — Prevents invalid dates like 31-Feb. | No           | 0.68       | AUTO_ACCEPTED | Test days at month boundaries, including leap years. |

### XFXLDSC

| BR-ID  | Rule Title                            | Source Program | Line | Type   | Description | Business Impact | PHI Sensitive | Confidence | Status        | Test Requirement |
|--------|----------------------------------------|----------------|------|--------|-------------|-----------------|--------------|------------|--------------|------------------|
| BR-009 | LDA map > 99 → EXIT                   | XFXLDSC        | 75   | filter | When LDAMAP is greater than 99, branch to 'EXIT'. Rejects invalid LDA mappings. | Medium — Ensures LDA map is in expected range. | No           | 0.56       | AUTO_ACCEPTED | Test LDAMAP at 99, 100, and typical values. |
| BR-010 | LDA map > 99 → EXIT                   | XFXLDSC        | 84   | filter | Duplicate of BR-009 at another code location. | Medium — Same as BR-009. | No           | 0.56       | AUTO_ACCEPTED | Same as BR-009. |
| BR-011 | LDA map > 99 → EXIT                   | XFXLDSC        | 93   | filter | Duplicate of BR-009 at another code location. | Medium — Same as BR-009. | No           | 0.56       | AUTO_ACCEPTED | Same as BR-009. |
| BR-012 | LDA map > 9999 → EXIT                 | XFXLDSC        | 102  | filter | When LDAMAP is greater than 9999, branch to 'EXIT'. Upper bound validation. | Medium — Prevents large/unexpected LDA values. | No           | 0.56       | AUTO_ACCEPTED | Test LDAMAP at 9999, 10000, and typical values. |

### XFXTABL

| BR-ID  | Rule Title                              | Source Program | Line | Type   | Description | Business Impact | PHI Sensitive | Confidence | Status        | Test Requirement |
|--------|------------------------------------------|----------------|------|--------|-------------|-----------------|--------------|------------|--------------|------------------|
| BR-013 | Indicator *IN79 = *ON → EXIT            | XFXTABL        | 99   | filter | When *IN79 equals on/active, branch to 'EXIT'. Table-driven branch. | Medium — Controls processing based on indicator. | No           | 0.90       | AUTO_ACCEPTED | Test *IN79 ON/OFF with table entries. |
| BR-014 | Indicator *IN79 = *ON → EXIT            | XFXTABL        | 114  | filter | Same rule at another location. | Medium — Same as BR-013. | No           | 0.90       | AUTO_ACCEPTED | Same as BR-013. |
| BR-015 | Indicator *IN79 = *ON → EXIT            | XFXTABL        | 129  | filter | Same rule at another location. | Medium — Same as BR-013. | No           | 0.90       | AUTO_ACCEPTED | Same as BR-013. |
| BR-016 | Indicator *IN79 = *ON → EXIT            | XFXTABL        | 144  | filter | Same rule at another location. | Medium — Same as BR-013. | No           | 0.90       | AUTO_ACCEPTED | Same as BR-013. |

## PATIENT_MANAGEMENT Domain

### HABADTE

| BR-ID  | Rule Title                                   | Source Program | Line | Type       | Description | Business Impact | PHI Sensitive | Confidence | Status        | Test Requirement |
|--------|-----------------------------------------------|----------------|------|-----------|-------------|-----------------|--------------|------------|--------------|------------------|
| BR-017 | File indicator = 0 → SKIP                    | HABADTE        | 143  | validation | When -FILE INDICATOR equals zero, branch to 'SKIP'. Excludes records with invalid file indicator. | High — Prevents processing of invalid/placeholder records. | No           | 0.92       | AUTO_ACCEPTED | Test MMPFIL = 0 vs non-zero; verify SKIP behaviour. |
| BR-018 | Void flag = 'V' → SKIP                       | HABADTE        | 144  | filter     | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. Excludes voided records. | High — Ensures voided records are not included in outputs. | No           | 0.99       | AUTO_ACCEPTED | Test MMPFLG = 'V' vs other values; verify SKIP behaviour. |
| BR-019 | Outpatient flag = 'O' → SKIP                 | HABADTE        | 145  | filter     | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. Excludes outpatient records. | High — Controls inpatient vs outpatient processing. | No           | 0.99       | AUTO_ACCEPTED | Test MMIORO = 'O' vs 'I'; verify SKIP behaviour. |

---

## Summary

- Total rules: 19.
- Domains: DATA_MAINTENANCE, PATIENT_MANAGEMENT.
- All rules AUTO_ACCEPTED; none require SME review.
- No rules directly manipulate PHI values, but many operate on control fields for PHI-bearing records.
