# Business Rules Catalog – HABADTE Project

This catalog lists all approved rules extracted from the AS400 RPG and SQLRPGLE programs for run `202607011209`, grouped by business domain.

## DATA_MAINTENANCE Domain

### Program: XFXCNTR (RPGLE)

| Rule ID | Rule Text                                      | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-------------------------------------------------|----------------|------------|------------|----------------|---------------|
| BR-001  | When X equals zero, branch to 'EXIT'           | XFXCNTR        | 0.47       | false      | Controls termination of counter-driven loops when the index is at the starting boundary; incorrect handling could either skip all processing or result in infinite loops. | Yes |
| BR-002  | When X equals 40, branch to 'EXIT'             | XFXCNTR        | 0.40       | false      | Enforces an upper bound on iteration (e.g. maximum 40 entries or attempts); misconfiguration may cause partial processing or performance issues. | Yes |

### Program: XFXCYMD (RPGLE)

| Rule ID | Rule Text                                                   | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-------------------------------------------------------------|----------------|------------|------------|----------------|---------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'                | XFXCYMD        | 0.56       | false      | Prevents usage of dates before year 1800, protecting calculations from invalid historical values. | Yes |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'             | XFXCYMD        | 0.56       | false      | Rejects dates beyond year 2100, avoiding future or mis-keyed year entries. | Yes |
| BR-005  | When VMM is less than 01, branch to 'EXIT'                  | XFXCYMD        | 0.56       | false      | Ensures months are not less than January (01); guards against invalid month values. | Yes |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'               | XFXCYMD        | 0.56       | false      | Ensures months do not exceed December (12); invalid months are rejected. | Yes |
| BR-007  | When VDD is less than 01, branch to 'EXIT'                  | XFXCYMD        | 0.56       | false      | Prevents day-of-month values below 1; avoids nonsensical dates. | Yes |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'         | XFXCYMD        | 0.68       | false      | Rejects day values greater than valid days in the month (e.g. Feb 30); ensures date arithmetic remains valid. | Yes |

### Program: XFXLDSC (RPGLE)

| Rule ID | Rule Text                                      | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-------------------------------------------------|----------------|------------|------------|----------------|---------------|
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'   | XFXLDSC        | 0.56       | false      | Prevents lookup using mapping codes beyond a two-digit range, ensuring only configured mappings are used. | Yes |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'   | XFXLDSC        | 0.56       | false      | Duplicate enforcement of LDAMAP upper bound; likely applied in multiple branches of logic. | Yes |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'   | XFXLDSC        | 0.56       | false      | Additional guard on the same threshold, possibly covering alternate pathways. | Yes |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT' | XFXLDSC        | 0.56       | false      | Enforces an extended upper bound for scenarios where larger mapping ranges are allowed; helps avoid mis-keyed mapping IDs. | Yes |

### Program: XFXTABL (RPGLE)

| Rule ID | Rule Text                                      | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-------------------------------------------------|----------------|------------|------------|----------------|---------------|
| BR-013  | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL        | 0.90       | false      | Uses a panel indicator to short-circuit table-driven logic when a certain UI state is active; prevents unnecessary background processing. | No |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL        | 0.90       | false      | Repeats the same control in another branch, ensuring consistent behaviour across code paths. | No |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL        | 0.90       | false      | Additional enforcement suggesting multiple table segments rely on the same indicator. | No |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL        | 0.90       | false      | Final guard ensuring no table logic executes when the indicator tells the program to stop. | No |

## PATIENT_MANAGEMENT Domain

### Program: HABADTE (RPGLE)

| Rule ID | Rule Text                                                                 | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|---------------------------------------------------------------------------|----------------|------------|------------|----------------|---------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'                        | HABADTE        | 0.92       | false      | Suppresses processing for an entire file/account when the file indicator signals no data or disabled processing; ensures system does not produce misleading reports. | No |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'                 | HABADTE        | 0.99       | false      | Excludes voided transfer records from inpatient reporting and XML outputs, maintaining clinical and financial accuracy. | No |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'       | HABADTE        | 0.99       | false      | Ensures outpatient events are not mixed into inpatient transfer histories, preserving reporting integrity. | No |

Business impact across the patient-management rules:

- Protects downstream consumers from voided and inappropriate (outpatient) events.
- Provides a clear control for toggling processing at the file/account level.
- Ensures that any migration preserves the same selection semantics for inpatient transfer exports.
