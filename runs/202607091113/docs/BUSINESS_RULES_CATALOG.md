# Business Rules Catalog – HABADTE Project

This catalog lists all approved business rules extracted from the AS400 RPG/SQLRPGLE programs, grouped by domain. Each rule includes its originating program, confidence score, PHI path flag, inferred business impact, and whether targeted testing is required during modernization.

---

## Domain: PATIENT_MANAGEMENT

### BR-017 – File Indicator Skip Rule

- **Rule ID:** BR-017  
- **Rule Text:** When -FILE INDICATOR equals zero, branch to 'SKIP'.  
- **Source Program:** HABADTE (RPGLE)  
- **Confidence Score:** 0.92  
- **PHI in Path:** false  
- **Business Impact:** Ensures that records marked as non-eligible by upstream processes (file indicator = 0) are excluded from downstream XML exports. Prevents duplicate or uninitialized records from polluting the outbound feed and keeps batch control totals aligned with business expectations.  
- **Test Required:** No (confidence ≥ 0.7)  

### BR-018 – Voided Transfer Skip Rule

- **Rule ID:** BR-018  
- **Rule Text:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.  
- **Source Program:** HABADTE (RPGLE)  
- **Confidence Score:** 0.99  
- **PHI in Path:** false  
- **Business Impact:** Prevents transfers that have been voided or cancelled from being exported. This keeps external systems consistent with the authoritative transaction state on the AS400 and avoids inaccurate billing or reporting.  
- **Test Required:** No  

### BR-019 – Outpatient Transfer Skip Rule

- **Rule ID:** BR-019  
- **Rule Text:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.  
- **Source Program:** HABADTE (RPGLE)  
- **Confidence Score:** 0.99  
- **PHI in Path:** false  
- **Business Impact:** Restricts the HABADTE output feed to inpatient transfers only. Outpatient-related records are excluded to avoid mixing care settings in downstream reporting or integration channels.  
- **Test Required:** No  

---

## Domain: DATA_MAINTENANCE

### BR-001 – Blank Field Exit Rule

- **Rule ID:** BR-001  
- **Rule Text:** Field formatting: exit if all blank (40 chars).  
- **Source Program:** XFXCNTR (RPGLE)  
- **Confidence Score:** 0.47  
- **PHI in Path:** false  
- **Business Impact:** Prevents unnecessary processing when a 40-character input field is entirely blank. This rule avoids treating blank strings as meaningful data, reducing noise in formatting routines and preventing accidental assignment of default or misleading values.  
- **Test Required:** Yes (confidence < 0.7)  

### BR-002 – Leading Character Non-Blank Rule

- **Rule ID:** BR-002  
- **Rule Text:** Field formatting: exit if first char non-blank.  
- **Source Program:** XFXCNTR (RPGLE)  
- **Confidence Score:** 0.40  
- **PHI in Path:** false  
- **Business Impact:** Ensures that certain formatting routines only proceed when a field starts with a blank character (e.g., right-aligning numeric values in a text field). This rule prevents double formatting or over-writing already formatted values.  
- **Test Required:** Yes  

### BR-003 – Minimum Year Validation

- **Rule ID:** BR-003  
- **Rule Text:** Date validation: reject year < 1800 (historical minimum).  
- **Source Program:** XFXCYMD (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Protects against implausible historical dates due to data entry mistakes or uninitialized date fields. Ensures downstream logic does not misinterpret invalid historical dates as legitimate events.  
- **Test Required:** Yes  

### BR-004 – Maximum Year Validation

- **Rule ID:** BR-004  
- **Rule Text:** Date validation: reject year > 2100 (forecast maximum).  
- **Source Program:** XFXCYMD (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Prevents acceptance of out-of-range future dates, reducing risk of corrupted timelines, erroneous schedules, or incorrect ageing calculations.  
- **Test Required:** Yes  

### BR-005 – Minimum Month Validation

- **Rule ID:** BR-005  
- **Rule Text:** Date validation: reject month < 01 (calendar constraint).  
- **Source Program:** XFXCYMD (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Ensures that month values respect calendar constraints and guards against invalid zero or negative months caused by parsing or arithmetic errors.  
- **Test Required:** Yes  

### BR-006 – Maximum Month Validation

- **Rule ID:** BR-006  
- **Rule Text:** Date validation: reject month > 12 (calendar constraint).  
- **Source Program:** XFXCYMD (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Blocks logically impossible month values, preventing invalid dates from entering further processing or being stored in databases.  
- **Test Required:** Yes  

### BR-007 – Minimum Day Validation

- **Rule ID:** BR-007  
- **Rule Text:** Date validation: reject day < 01 (calendar constraint).  
- **Source Program:** XFXCYMD (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Avoids invalid day values (e.g., zero or negative) from being treated as actual dates. This helps maintain integrity of timelines and audit information.  
- **Test Required:** Yes  

### BR-008 – Day-in-Month Validation

- **Rule ID:** BR-008  
- **Rule Text:** When VDD is greater than DYS(VMM), branch to 'EXIT'.  
- **Source Program:** XFXCYMD (RPGLE)  
- **Confidence Score:** 0.68  
- **PHI in Path:** false  
- **Business Impact:** Validates that the day value does not exceed the number of days in the given month (including leap year adjustments). This prevents invalid dates like 31-Feb from being accepted.  
- **Test Required:** Yes (confidence < 0.7)  

### BR-009 – Level Code Range Validation (1)

- **Rule ID:** BR-009  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Ensures organizational level codes used for lookup are within the configured range so that invalid or deprecated levels do not drive reporting or routing logic.  
- **Test Required:** Yes  

### BR-010 – Level Code Range Validation (2)

- **Rule ID:** BR-010  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Same as BR-009, applied to a different level dimension or use case; helps enforce consistency of level hierarchies.  
- **Test Required:** Yes  

### BR-011 – Level Code Range Validation (3)

- **Rule ID:** BR-011  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Additional coverage for level validation; may support alternative level types or sub-levels.  
- **Test Required:** Yes  

### BR-012 – Level Code Range Validation (4)

- **Rule ID:** BR-012  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC (RPGLE)  
- **Confidence Score:** 0.56  
- **PHI in Path:** false  
- **Business Impact:** Completes the level validation set, ensuring any level code feeding organizational hierarchies is vetted before use.  
- **Test Required:** Yes  

### BR-013 – Table Lookup Early Exit (1)

- **Rule ID:** BR-013  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL (RPGLE)  
- **Confidence Score:** 0.90  
- **PHI in Path:** false  
- **Business Impact:** Provides a control indicator that allows table-driven lookup routines to terminate early, e.g., when no further table entries are required or when a configuration condition is met. This improves performance and enforces configuration-driven behavior.  
- **Test Required:** No  

### BR-014 – Table Lookup Early Exit (2)

- **Rule ID:** BR-014  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL (RPGLE)  
- **Confidence Score:** 0.90  
- **PHI in Path:** false  
- **Business Impact:** Same as BR-013; applies to a specific branch or table category within XFXTABL, reinforcing consistent use of control indicators.  
- **Test Required:** No  

### BR-015 – Table Lookup Early Exit (3)

- **Rule ID:** BR-015  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL (RPGLE)  
- **Confidence Score:** 0.90  
- **PHI in Path:** false  
- **Business Impact:** Additional occurrence of the early-exit pattern; may apply to another category of table lookups.  
- **Test Required:** No  

### BR-016 – Table Lookup Early Exit (4)

- **Rule ID:** BR-016  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL (RPGLE)  
- **Confidence Score:** 0.90  
- **PHI in Path:** false  
- **Business Impact:** Completes the set of indicator-driven exits in XFXTABL, ensuring that all relevant code paths respect the *IN79 indicator as a control flag.  
- **Test Required:** No  

### BR-020 – Application Profile Table Access

- **Rule ID:** BR-020  
- **Rule Text:** SQL program accesses table 'HXPAPPPRF'.  
- **Source Program:** HXXAPPPRF (SQLRPGLE)  
- **Confidence Score:** 0.65  
- **PHI in Path:** false  
- **Business Impact:** Indicates that application-level profile data is stored in table HXPAPPPRF and is accessed by HXXAPPPRF to drive configuration. Any migration must preserve this configuration table or replace it with equivalent application settings.  
- **Test Required:** Yes  
