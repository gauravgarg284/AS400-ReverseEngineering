# Business Rules Catalog

Project: HABADTE

This catalog lists all approved business rules identified in the AS400 RPG programs, grouped by domain. Each entry includes the originating program, confidence score, PHI involvement, inferred business impact, and whether explicit test coverage is required in the migration.

## Domain: DATA_MAINTENANCE

### Program: XFXCNTR

| Rule ID | Rule Text                                      | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                     | Test Required |
|---------|-------------------------------------------------|----------------|------------|------------------|-------------|---------------------------------------------------------------------|--------------|
| BR-001  | When X equals zero, branch to 'EXIT'           | XFXCNTR        | 0.47       | DATA_MAINTENANCE | false       | Prevents processing when counter value is uninitialized or zero.    | Yes          |
| BR-002  | When X equals 40, branch to 'EXIT'             | XFXCNTR        | 0.40       | DATA_MAINTENANCE | false       | Enforces upper bound on counter range (likely configuration limit). | Yes          |

### Program: XFXCYMD

| Rule ID | Rule Text                                                   | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                      | Test Required |
|---------|-------------------------------------------------------------|----------------|------------|------------------|-------------|----------------------------------------------------------------------|--------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'                | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Rejects dates earlier than supported calendar range.                 | Yes          |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'             | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Rejects dates beyond supported calendar range.                       | Yes          |
| BR-005  | When VMM is less than 01, branch to 'EXIT'                  | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Rejects month values below January.                                  | Yes          |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'               | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Rejects month values above December.                                 | Yes          |
| BR-007  | When VDD is less than 01, branch to 'EXIT'                  | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Rejects day values below 1, preventing invalid dates.                | Yes          |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'         | XFXCYMD        | 0.68       | DATA_MAINTENANCE | false       | Rejects days beyond month-specific maximum (e.g., 31, 30, 28, 29).   | Yes          |

### Program: XFXLDSC

| Rule ID | Rule Text                                       | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                  | Test Required |
|---------|--------------------------------------------------|----------------|------------|------------------|-------------|------------------------------------------------------------------|--------------|
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'| XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Enforces upper limit on load/mapping code values.                | Yes          |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'| XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Duplicate rule, likely for alternative path; same limit enforced.| Yes          |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'| XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Additional branch with same constraint across contexts.          | Yes          |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT'| XFXLDSC      | 0.56       | DATA_MAINTENANCE | false       | Enforces extended upper limit for wide-range mapping values.     | Yes          |

### Program: XFXTABL

| Rule ID | Rule Text                                            | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                   | Test Required |
|---------|------------------------------------------------------|----------------|------------|------------------|-------------|-------------------------------------------------------------------|--------------|
| BR-013  | When *IN79 equals on/active, branch to 'EXIT'        | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Short-circuits processing when configuration indicator is active.| No           |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT'        | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Variant of same behavior in another subroutine.                  | No           |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT'        | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Ensures consistent exit behavior across table segments.          | No           |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT'        | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Prevents additional processing when indicator is already active. | No           |

## Domain: PATIENT_MANAGEMENT

### Program: HABADTE

| Rule ID | Rule Text                                                             | Source Program | Confidence | Domain             | PHI In Path | Business Impact                                                                                 | Test Required |
|---------|------------------------------------------------------------------------|----------------|------------|--------------------|-------------|-------------------------------------------------------------------------------------------------|--------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'                    | HABADTE        | 0.92       | PATIENT_MANAGEMENT | false       | Excludes records with uninitialized/invalid file indicator from patient transfer extracts.      | No           |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'             | HABADTE        | 0.99       | PATIENT_MANAGEMENT | false       | Prevents voided or cancelled transfers from appearing in inpatient movement reports.           | No           |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'   | HABADTE        | 0.99       | PATIENT_MANAGEMENT | false       | Ensures extract focuses strictly on inpatient records, excluding outpatient episodes.          | No           |
