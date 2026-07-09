# Business Rules Catalog – HABADTE Project

This catalog lists all approved business rules extracted from the HABADTE project, grouped by domain.

## Domain: DATA_MAINTENANCE

| Rule ID | Source Program | Confidence | PHI In Path | Rule Text | Business Impact | Test Required |
|---------|----------------|------------|-------------|-----------|-----------------|---------------|
| BR-001 | XFXCNTR | 0.47 | false | Field formatting: exit if all blank (40 chars) | Prevents unnecessary processing of empty fields and ensures only meaningful data is formatted. | Yes |
| BR-002 | XFXCNTR | 0.40 | false | Field formatting: exit if first char non-blank | Avoids reformatting fields that already contain data, preserving existing formatting. | Yes |
| BR-003 | XFXCYMD | 0.56 | false | Date validation: reject year < 1800 (historical minimum) | Ensures invalid historical dates are blocked, protecting data quality. | Yes |
| BR-004 | XFXCYMD | 0.56 | false | Date validation: reject year > 2100 (forecast maximum) | Prevents future dates beyond supported range, aligning with business calendar limits. | Yes |
| BR-005 | XFXCYMD | 0.56 | false | Date validation: reject month < 01 (calendar constraint) | Blocks malformed dates where month underflows valid range. | Yes |
| BR-006 | XFXCYMD | 0.56 | false | Date validation: reject month > 12 (calendar constraint) | Blocks malformed dates where month exceeds valid range. | Yes |
| BR-007 | XFXCYMD | 0.56 | false | Date validation: reject day < 01 (calendar constraint) | Ensures day component is not below valid minimum. | Yes |
| BR-008 | XFXCYMD | 0.68 | false | When VDD is greater than DYS(VMM), branch to 'EXIT' | Protects against invalid day values beyond days-in-month, including leap years. | Yes |
| BR-009 | XFXLDSC | 0.56 | false | Level lookup: reject if level code exceeds valid range | Prevents use of undefined level codes and enforces table-driven configuration. | Yes |
| BR-010 | XFXLDSC | 0.56 | false | Level lookup: reject if level code exceeds valid range | Same as BR-009, applied to another level dimension. | Yes |
| BR-011 | XFXLDSC | 0.56 | false | Level lookup: reject if level code exceeds valid range | Same as BR-009, applied to another level dimension. | Yes |
| BR-012 | XFXLDSC | 0.56 | false | Level lookup: reject if level code exceeds valid range | Same as BR-009, applied to another level dimension. | Yes |
| BR-013 | XFXTABL | 0.90 | false | When *IN79 equals on/active, branch to 'EXIT' | Stops table processing based on a control indicator, preventing duplicate or unwanted table reads. | No |
| BR-014 | XFXTABL | 0.90 | false | When *IN79 equals on/active, branch to 'EXIT' | Same as BR-013, applied to another control branch. | No |
| BR-015 | XFXTABL | 0.90 | false | When *IN79 equals on/active, branch to 'EXIT' | Same as BR-013, applied to another control branch. | No |
| BR-016 | XFXTABL | 0.90 | false | When *IN79 equals on/active, branch to 'EXIT' | Same as BR-013, applied to another control branch. | No |
| BR-020 | HXXAPPPRF | 0.65 | false | SQL program accesses table 'HXPAPPPRF' | Governs how application profile rows are accessed in SQL, impacting configuration and security. | Yes |

## Domain: PATIENT_MANAGEMENT

| Rule ID | Source Program | Confidence | PHI In Path | Rule Text | Business Impact | Test Required |
|---------|----------------|------------|-------------|-----------|-----------------|---------------|
| BR-017 | HABADTE | 0.92 | false | When -FILE INDICATOR equals zero, branch to 'SKIP' | Ensures records flagged as absent are not processed, preventing false reporting. | No |
| BR-018 | HABADTE | 0.99 | false | When -FLAG INDICATOR equals void/voided, branch to 'SKIP' | Excludes voided transactions from downstream processing, protecting financial and clinical accuracy. | No |
| BR-019 | HABADTE | 0.99 | false | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' | Filters out outpatient cases when the report targets inpatient-only scenarios. | No |
