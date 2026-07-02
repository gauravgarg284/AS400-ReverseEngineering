# BUSINESS RULES CATALOG – HABADTE Run 202607021244

This catalog lists all approved business rules extracted from the AS400 HABADTE application landscape. Rules are grouped by business domain and include their origin program, confidence, PHI path flags, inferred business impact, and testing requirements.

---

## DATA_MAINTENANCE Domain

### Program: XFXCNTR (Counter Control)

| Rule ID | Rule Text                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|---------------------------------------------|----------------|-----------|------------|-----------------|--------------|
| BR-001  | When X equals zero, branch to 'EXIT'       | XFXCNTR        | 0.47      | false      | Prevents processing when control counter indicates "no items" or initial state; ensures downstream routines do not run on empty sets. | Yes |
| BR-002  | When X equals 40, branch to 'EXIT'         | XFXCNTR        | 0.40      | false      | Enforces an upper limit of 40 units/iterations, avoiding runaway loops or exceeding report/record limits. | Yes |

### Program: XFXCYMD (Date Validation)

| Rule ID | Rule Text                                                          | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|--------------------------------------------------------------------|----------------|-----------|------------|-----------------|--------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'                      | XFXCYMD        | 0.56      | false      | Protects against obviously invalid historical years, preventing incorrect date calculations and data corruption. | Yes |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'                   | XFXCYMD        | 0.56      | false      | Prevents future dates far beyond system horizon, maintaining data quality for reports and ageing logic. | Yes |
| BR-005  | When VMM is less than 01, branch to 'EXIT'                        | XFXCYMD        | 0.56      | false      | Ensures month values are within 1–12, avoiding invalid date construction. | Yes |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'                     | XFXCYMD        | 0.56      | false      | Blocks invalid month values greater than 12, maintaining calendar integrity. | Yes |
| BR-007  | When VDD is less than 01, branch to 'EXIT'                        | XFXCYMD        | 0.56      | false      | Ensures day values are positive, preventing malformed dates. | Yes |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'               | XFXCYMD        | 0.68      | false      | Validates day-of-month against actual month length (including leap year logic), preventing impossible dates like 31 February. | Yes |

### Program: XFXLDSC (Level Description Mapping)

| Rule ID | Rule Text                                       | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|-------------------------------------------------|----------------|-----------|------------|-----------------|--------------|
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'   | XFXLDSC        | 0.56      | false      | Enforces upper bounds on mapping codes, ensuring only supported level mappings are processed. | Yes |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'   | XFXLDSC        | 0.56      | false      | Duplicate variant of BR-009, reinforcing the same constraint across branches. | Yes |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'   | XFXLDSC        | 0.56      | false      | Additional branch enforcing mapping code limit, preventing out-of-range configuration usage. | Yes |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT' | XFXLDSC        | 0.56      | false      | Guards against very large mapping codes, likely representing invalid or unconfigured levels. | Yes |

### Program: XFXTABL (Table-Driven Control)

| Rule ID | Rule Text                                            | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|------------------------------------------------------|----------------|-----------|------------|-----------------|--------------|
| BR-013  | When *IN79 equals on/active, branch to 'EXIT'       | XFXTABL        | 0.90      | false      | Uses configuration indicator *IN79 to short-circuit processing based on table-driven conditions, protecting from unsupported states. | No |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT'       | XFXTABL        | 0.90      | false      | Variant of BR-013 applied to a different branch; ensures consistent behaviour when indicator is active. | No |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT'       | XFXTABL        | 0.90      | false      | Reinforces global exit behaviour when *IN79 is ON across multiple table routines. | No |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT'       | XFXTABL        | 0.90      | false      | Final branch implementing table-driven exit, preventing invalid or unwanted transitions. | No |

### Program: HXXAPPPRF (SQL Profile Access)

| Rule ID | Rule Text                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|---------------------------------------------|----------------|-----------|------------|-----------------|--------------|
| BR-020  | SQL program accesses table 'HXPAPPPRF'     | HXXAPPPRF      | 0.65      | false      | Establishes dependency on application profile table HXPAPPPRF, influencing how user/application preferences affect data maintenance behaviour. | Yes |

---

## PATIENT_MANAGEMENT Domain

### Program: HABADTE (Patient Transfer Controller)

| Rule ID | Rule Text                                                        | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|------------------------------------------------------------------|----------------|-----------|------------|-----------------|--------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'              | HABADTE        | 0.92      | false      | Ensures only active file records are processed, reducing the risk of exporting stale or inactive transfers. | No |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'       | HABADTE        | 0.99      | false      | Excludes voided transfers from reporting and XML export, aligning system outputs with financial and clinical reality. | No |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' | HABADTE    | 0.99      | false      | Restricts processing to inpatient movements when required, ensuring reports and integrations reflect inpatient activity only. | No |

---

## Testing Guidance Summary

- Rules with **confidence < 0.7** (e.g., BR-001–BR-012, BR-020) require targeted unit and integration tests to confirm thresholds and branches match SME expectations.
- High-confidence rules (BR-013–BR-019) should still be regression-tested but are less likely to change.

This catalog serves as the authoritative list of business rules for HABADTE-related components in the modernisation effort.
