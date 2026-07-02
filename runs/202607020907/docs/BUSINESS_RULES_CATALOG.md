# Business Rules Catalog – HABADTE Portfolio

This catalog lists all approved business rules extracted from the HABADTE AS400 application, grouped by domain. Each entry includes its originating program, confidence score, PHI path flag, inferred business impact, and whether explicit test coverage is required.

## 1. DATA_MAINTENANCE Domain

### 1.1 Program: XFXCNTR (RPGLE)

RPGLE utility controlling counter-based exit behavior.

| BR-ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|-------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-001 | When X equals zero, branch to 'EXIT'. | XFXCNTR | 0.47 | false | Prevents processing when the main loop counter is zero, ensuring no work is done without available items. | Yes |
| BR-002 | When X equals 40, branch to 'EXIT'. | XFXCNTR | 0.40 | false | Imposes an upper bound of 40 iterations, limiting batch size or preventing runaway loops. | Yes |

### 1.2 Program: XFXCYMD (RPGLE)

Date validation routine enforcing calendar constraints.

| BR-ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|-------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-003 | When VYY is less than 1800, branch to 'EXIT'. | XFXCYMD | 0.56 | false | Rejects implausible historical dates, preventing corrupt or mis-keyed year values from entering the system. | Yes |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'. | XFXCYMD | 0.56 | false | Rejects future dates beyond reasonable planning horizon, guarding against keying errors and data anomalies. | Yes |
| BR-005 | When VMM is less than 01, branch to 'EXIT'. | XFXCYMD | 0.56 | false | Ensures month values start at January (1), preventing invalid month codes. | Yes |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'. | XFXCYMD | 0.56 | false | Ensures month values do not exceed December (12), preventing invalid month codes. | Yes |
| BR-007 | When VDD is less than 01, branch to 'EXIT'. | XFXCYMD | 0.56 | false | Guards against day values below 1, rejecting invalid date entries. | Yes |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. | XFXCYMD | 0.68 | false | Uses month-specific maximum days to prevent impossible dates (e.g., 31 February). | Yes |

### 1.3 Program: XFXLDSC (RPGLE)

Level description mapper controlling acceptable mapping ranges.

| BR-ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|-------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'. | XFXLDSC | 0.56 | false | Caps mapping codes to two digits, enforcing a constrained level mapping scheme. | Yes |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'. | XFXLDSC | 0.56 | false | Duplicate enforcement to ensure no overflow beyond configured mapping range. | Yes |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'. | XFXLDSC | 0.56 | false | Reinforces cut-off in variant branches, preventing inconsistent mapping rules. | Yes |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'. | XFXLDSC | 0.56 | false | Provides an absolute upper bound for extended mapping codes, protecting against corrupted configuration values. | Yes |

### 1.4 Program: XFXTABL (RPGLE)

Dictionary/table lookup utility using indicator *IN79 for control.

| BR-ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|-------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | false | Signals completion or a stop condition in table lookups, preventing unnecessary reads. | No |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | false | Reinforces the same control rule in an alternate branch or state. | No |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | false | Indicates the lookup should terminate once a match or condition is met. | No |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | false | Ensures consistent behavior across multiple segments of the lookup logic. | No |

## 2. PATIENT_MANAGEMENT Domain

### 2.1 Program: HABADTE (RPGLE)

Primary patient management driver controlling eligibility of transfer records for processing and reporting.

| BR-ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|-------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. | HABADTE | 0.92 | false | Excludes invalid or uninitialized records from processing, ensuring only complete file entries are included in transfer reporting. | No |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. | HABADTE | 0.99 | false | Prevents voided transfer events from appearing in reports and XML, aligning output with clinical corrections and reversals. | No |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. | HABADTE | 0.99 | false | Restricts processing to inpatient encounters, avoiding contamination of inpatient reports with outpatient data. | No |

### 2.2 Domain-Level Impact

The HABADTE rules collectively implement a robust eligibility filter for PHI-bearing transfer records. While `phi_in_path` is flagged as false in the extracted rules, the associated fields (AFACCT, AFMRNO) in HAPTRFR and OMPMAST carry MRN and account numbers, so test cases and migration plans must treat these flows as PHI-sensitive.

Key business impacts in the Patient Management domain:
- Ensuring that only clinically valid, non-voided inpatient transfers are included in downstream analytics and interfaces.
- Protecting consumers of the XML payload from misleading or incomplete data.
- Providing a clear boundary between inpatient and outpatient processing pipelines.

All rules in this catalog must be preserved in the modernized implementation, with explicit test cases for those marked "Yes" in the Test Required column and regression tests for the others in integrated scenarios.
