# Business Rules Catalog – HABADTE Project

This catalog lists all approved business rules reverse-engineered from the HABADTE AS400 application. Rules are grouped by domain and capture their origin, PHI considerations, and testing requirements.

## DATA_MAINTENANCE Domain

### BR-001 – Field formatting: exit if all blank (40 chars)

- **Rule ID:** BR-001  
- **Rule Text:** Field formatting: exit if all blank (40 chars).  
- **Source Program:** XFXCNTR  
- **Confidence Score:** 0.47  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Prevents processing of input fields that contain only spaces, avoiding unnecessary downstream formatting and validation. Ensures that only meaningful user input is considered when updating or creating records.
- **Test Requirement:** Yes (confidence < 0.7).

### BR-002 – Field formatting: exit if first char non-blank

- **Rule ID:** BR-002  
- **Rule Text:** Field formatting: exit if first char non-blank.  
- **Source Program:** XFXCNTR  
- **Confidence Score:** 0.40  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Stops formatting logic when the first character is already populated, reducing the risk of overwriting user-entered content. Supports idempotent formatting for certain display fields.
- **Test Requirement:** Yes (confidence < 0.7).

### BR-003 – Date validation: reject year < 1800 (historical minimum)

- **Rule ID:** BR-003  
- **Rule Text:** Date validation: reject year < 1800 (historical minimum).  
- **Source Program:** XFXCYMD  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Protects data quality by preventing obviously incorrect historical dates from being stored or processed, maintaining meaningful temporal ranges for patient and configuration data.
- **Test Requirement:** Yes.

### BR-004 – Date validation: reject year > 2100 (forecast maximum)

- **Rule ID:** BR-004  
- **Rule Text:** Date validation: reject year > 2100 (forecast maximum).  
- **Source Program:** XFXCYMD  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Prevents future dates far beyond the planning horizon, avoiding erroneous scheduling or life-cycle dates and ensuring system reports remain realistic.
- **Test Requirement:** Yes.

### BR-005 – Date validation: reject month < 01 (calendar constraint)

- **Rule ID:** BR-005  
- **Rule Text:** Date validation: reject month < 01 (calendar constraint).  
- **Source Program:** XFXCYMD  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Enforces valid calendar months, preventing malformed month values from entering the system.
- **Test Requirement:** Yes.

### BR-006 – Date validation: reject month > 12 (calendar constraint)

- **Rule ID:** BR-006  
- **Rule Text:** Date validation: reject month > 12 (calendar constraint).  
- **Source Program:** XFXCYMD  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Ensures that only valid months 1–12 are accepted, maintaining integrity of date fields used throughout patient and configuration records.
- **Test Requirement:** Yes.

### BR-007 – Date validation: reject day < 01 (calendar constraint)

- **Rule ID:** BR-007  
- **Rule Text:** Date validation: reject day < 01 (calendar constraint).  
- **Source Program:** XFXCYMD  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Blocks invalid day values below 1, preventing malformed dates from impacting downstream calculations.
- **Test Requirement:** Yes.

### BR-008 – When VDD is greater than DYS(VMM), branch to 'EXIT'

- **Rule ID:** BR-008  
- **Rule Text:** When VDD is greater than DYS(VMM), branch to 'EXIT'.  
- **Source Program:** XFXCYMD  
- **Confidence Score:** 0.68  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Ensures that day values do not exceed the maximum valid day for a given month (including leap-year handling), preventing logical date errors.
- **Test Requirement:** Yes.

### BR-009 – Level lookup: reject if level code exceeds valid range

- **Rule ID:** BR-009  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Protects organisational hierarchy integrity by ensuring only valid level codes are used during lookups; prevents misaligned reporting structures.
- **Test Requirement:** Yes.

### BR-010 – Level lookup: reject if level code exceeds valid range

- **Rule ID:** BR-010  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Applies the same constraint to another level dimension (e.g., different hierarchy depth), ensuring data consistency across all levels.
- **Test Requirement:** Yes.

### BR-011 – Level lookup: reject if level code exceeds valid range

- **Rule ID:** BR-011  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Further validates hierarchical levels to avoid referencing non-existent organisational nodes.
- **Test Requirement:** Yes.

### BR-012 – Level lookup: reject if level code exceeds valid range

- **Rule ID:** BR-012  
- **Rule Text:** Level lookup: reject if level code exceeds valid range.  
- **Source Program:** XFXLDSC  
- **Confidence Score:** 0.56  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Completes the set of hierarchical validations, reinforcing that only configured levels participate in reporting and routing.
- **Test Requirement:** Yes.

### BR-013 – When *IN79 equals on/active, branch to 'EXIT'

- **Rule ID:** BR-013  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL  
- **Confidence Score:** 0.90  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Acts as a guard flag: when a controlling indicator is active, table lookup stops, allowing the caller to bypass certain code translations.
- **Test Requirement:** No (confidence ≥ 0.7).

### BR-014 – When *IN79 equals on/active, branch to 'EXIT'

- **Rule ID:** BR-014  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL  
- **Confidence Score:** 0.90  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Mirrors BR-013 for another branch or table path, ensuring consistent use of the control indicator across lookups.
- **Test Requirement:** No.

### BR-015 – When *IN79 equals on/active, branch to 'EXIT'

- **Rule ID:** BR-015  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL  
- **Confidence Score:** 0.90  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Provides controlled early-exit behaviour in another context, avoiding unnecessary table reads when conditions are already satisfied.
- **Test Requirement:** No.

### BR-016 – When *IN79 equals on/active, branch to 'EXIT'

- **Rule ID:** BR-016  
- **Rule Text:** When *IN79 equals on/active, branch to 'EXIT'.  
- **Source Program:** XFXTABL  
- **Confidence Score:** 0.90  
- **PHI in Path:** No  
- **Plain-English Business Impact:**
  - Completes the set of controlled exits governed by indicator *IN79, ensuring consistent handling of configured flags.
- **Test Requirement:** No.

### BR-020 – SQL program accesses table 'HXPAPPPRF'

- **Rule ID:** BR-020  
- **Rule Text:** SQL program accesses table 'HXPAPPPRF'.  
- **Source Program:** HXXAPPPRF  
- **Confidence Score:** 0.65  
- **PHI in Path:** No (no explicit PHI fields flagged in this rule).  
- **Plain-English Business Impact:**
  - Indicates that application profile data is read or updated via SQL, affecting how user or application-specific configuration is maintained.
- **Test Requirement:** Yes.

## PATIENT_MANAGEMENT Domain

### BR-017 – When -FILE INDICATOR equals zero, branch to 'SKIP'

- **Rule ID:** BR-017  
- **Rule Text:** When -FILE INDICATOR equals zero, branch to 'SKIP'.  
- **Source Program:** HABADTE  
- **Confidence Score:** 0.92  
- **PHI in Path:** No (indicator-level logic only).  
- **Plain-English Business Impact:**
  - Ensures that only records with an active backing file are processed. This prevents emitting incomplete or logically closed records into patient transfer outputs.
- **Test Requirement:** No.

### BR-018 – When -FLAG INDICATOR equals void/voided, branch to 'SKIP'

- **Rule ID:** BR-018  
- **Rule Text:** When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.  
- **Source Program:** HABADTE  
- **Confidence Score:** 0.99  
- **PHI in Path:** No (decision based on status flag).  
- **Plain-English Business Impact:**
  - Prevents voided patient transfer records from appearing in XML outputs or downstream feeds, maintaining clinical and financial accuracy.
- **Test Requirement:** No.

### BR-019 – When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'

- **Rule ID:** BR-019  
- **Rule Text:** When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.  
- **Source Program:** HABADTE  
- **Confidence Score:** 0.99  
- **PHI in Path:** No (uses flags tied to patient type).  
- **Plain-English Business Impact:**
  - Ensures that the HABADTE report focuses on inpatient activity by excluding outpatient records from the transfer report, aligning with its clinical reporting scope.
- **Test Requirement:** No.
