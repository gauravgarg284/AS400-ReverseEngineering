# BUSINESS RULES CATALOG – HABADTE System

Run ID: 202607161154  
Project: HABADTE

This catalog lists all approved business rules from the reverse-engineering pipeline, grouped by domain, with traceability and testing indicators.

---

## Domain: DATA_MAINTENANCE

### XFXCNTR – Text Centering Utility

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-001 | Text centering — skip processing if the entire input field is blank (nothing to center). | XFXCNTR | 0.47 | No | Prevents unnecessary processing when input is empty and ensures that blank fields remain blank rather than being altered. | Yes |
| BR-002 | Text centering — skip processing if the text already starts at the first position (already left-aligned). | XFXCNTR | 0.40 | No | Avoids double-processing and unexpected shifts of already left-aligned text, preserving existing formatting. | Yes |

### XFXCYMD – Date Validation Utility

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-003 | Date validation — reject dates before 1800: the system does not support historical records predating 1800. | XFXCYMD | 0.56 | No | Enforces a valid historical range, preventing mis-entry of dates far in the past that could distort reporting or cause calculation errors. | Yes |
| BR-004 | Date validation — reject dates beyond 2100: the system does not support forecast dates past the year 2100. | XFXCYMD | 0.56 | No | Limits future dates to a reasonable range, preventing unrealistic or mistyped future dates from entering the system. | Yes |
| BR-005 | Date validation — month must be January (01) or later: a month value below 01 is not a valid calendar month. | XFXCYMD | 0.56 | No | Protects against invalid month values below the allowed range, ensuring date fields are structurally valid. | Yes |
| BR-006 | Date validation — month must be December (12) or earlier: a month value above 12 is not a valid calendar month. | XFXCYMD | 0.56 | No | Rejects dates with month values greater than 12, maintaining strict calendar validity. | Yes |
| BR-007 | Date validation — day must be 01 or later: a day value of 0 or less is not a valid calendar day. | XFXCYMD | 0.56 | No | Prevents dates with invalid day values from entering the system, reducing downstream calculation and reporting errors. | Yes |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. | XFXCYMD | 0.68 | No | Enforces that derived date differences stay within safe bounds; when the difference exceeds threshold, processing is halted to avoid invalid output. | Yes |

### XFXLDSC – Organisational Level Lookup

All four rules have identical logic but are applied to different organisational levels or contexts.

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-009 | Organizational level lookup — the level code is out of the valid range; return no description. | XFXLDSC | 0.56 | No | Ensures that invalid organisational level codes do not produce misleading descriptions, forcing callers to handle unknown codes explicitly. | Yes |
| BR-010 | Organizational level lookup — the level code is out of the valid range; return no description. | XFXLDSC | 0.56 | No | Same as BR-009, likely applied to a different level or data area; reinforces robust handling of invalid hierarchy codes. | Yes |
| BR-011 | Organizational level lookup — the level code is out of the valid range; return no description. | XFXLDSC | 0.56 | No | Ensures consistency of behaviour across multiple level lookups when codes fall outside configured values. | Yes |
| BR-012 | Organizational level lookup — the level code is out of the valid range; return no description. | XFXLDSC | 0.56 | No | Prevents accidental use of invalid organisational references in reporting and access control. | Yes |

### XFXTABL – Dictionary/Table Lookup

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Treats an internal indicator as a stop condition, allowing calling programs to short-circuit processing when a configuration or status threshold is reached. | No |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Duplicate variant of BR-013; indicates the pattern is applied in multiple branches of table lookup logic. | No |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Reinforces the same exit condition, ensuring consistent handling of the activation indicator across different codes. | No |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Provides a standardised mechanism for halting table-driven processing when a controlling flag is set. | No |

### HXXAPPPRF – Application Preferences

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-030 | Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed during processing. | HXXAPPPRF | 0.65 | No | Centralises configuration reads, ensuring that business behaviour (such as identifier display) is controlled by table-driven preferences rather than hard-coded logic. | Yes |
| BR-031 | Database table access — HXXAPPPRF reads from the 'HXPAPPL6' table to retrieve configuration or reference data needed during processing. | HXXAPPPRF | 0.65 | No | Uses a level-specific preference table, enabling facility or organisational overrides and more granular configuration. | Yes |
| BR-032 | Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed during processing. | HXXAPPPRF | 0.65 | No | Confirms repeated reliance on the main preference table and underscores the importance of maintaining its schema and content in the modernised system. | Yes |

---

## Domain: PATIENT_MANAGEMENT

### HABADTE – Patient Census Driver

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-017 | Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census. | HABADTE | 0.92 | No | Ensures census reflects only truly admitted patients, not scheduled or tentative admissions, which is critical for accurate occupancy and staffing decisions. | No |
| BR-018 | Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census. | HABADTE | 0.99 | No | Prevents cancelled or erroneous accounts from polluting census metrics and downstream billing or reporting processes. | No |
| BR-019 | Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded. | HABADTE | 0.99 | No | Maintains the clinical scope of the report by excluding outpatients, ensuring that census numbers reflect inpatient load only. | No |
| BR-020 | Organisational Level Filter: Each patient record must belong to the requested organisation. The report accepts an org level (1–6) and org code from the caller. If the patient's org code at that level does not match, the record is excluded. | HABADTE | 0.65 | No | Ensures that users see only patients belonging to their organisational scope (e.g., facility or unit), supporting governance and data minimisation. | Yes |
| BR-021 | Census Date — Admission Date Check: A patient admitted after the census date is not yet active on that date. If the patient's admission date is later than the requested census date, the patient is excluded from the report. | HABADTE | 0.85 | No | Aligns census with temporal reality, preventing future admissions from appearing in past-dated census reports. | No |
| BR-022 | Census Date — Discharge Date Check: A patient who has already been discharged on or before the census date is no longer active. If discharge date is set (non-zero) and is on or before the census date, the patient is excluded. A discharge date of zero means the patient has not been discharged and is always considered active (subject to other filters). | HABADTE | 0.85 | No | Maintains accurate active population counts by excluding patients already discharged and properly handling "not yet discharged" as an active status. | No |
| BR-023 | Room Number Retrieval from Transfer History: The patient's current room number is not stored directly on the patient record. Instead, HABADTE reads transfer history (HAPTRFR) and selects the latest transfer up to the census date to derive current room and location. | HABADTE | 0.65 | No | Provides accurate bed and location information for each patient, essential for operational workflows such as rounding, transport and capacity planning. | Yes |
| BR-024 | Room Class Description Lookup: The 2-character room class code (AFNRMC) retrieved from the transfer history is resolved to a description and status via dictionary tables (HXPTABLD/HXLTABLD/HXLTABLP/HXLTABLS) through XFXTABL. | HABADTE | 0.65 | No | Translates technical codes into human-readable room classes (e.g., ICU, general ward), improving usability and reducing misinterpretation. | Yes |
| BR-025 | Hospital / Therapeutic Leave Flag: When the room class description is looked up, the HCS mapping field (XFDMAP) is also interpreted to determine whether the patient is on hospital leave or therapeutic leave. | HABADTE | 0.57 | No | Enables accurate tracking of patients who are temporarily off the unit, affecting occupancy, billing and clinical workflows. | Yes |
| BR-026 | Nursing Station Name Resolution: For each qualifying patient, the nursing station name is looked up from the Nursing Station table (HXPNSTN/TXPNSTN) based on organisational level and station code. | HABADTE | 0.65 | No | Ensures that reports display meaningful station names rather than raw codes, improving usability for clinical staff. | Yes |
| BR-027 | Patient Identifier Display — MRN vs Account Number: The report can show either the patient's Medical Record Number (MRN) or account number based on application preferences and facility overrides, resolved via XFXMRNROL/HXXAPPPRF/HXHAPPPRF. | HABADTE | 0.65 | Yes | Controls which identifier is presented, balancing clinical familiarity (MRN) with financial tracking (account), and has direct impact on PHI presentation and privacy. | Yes |
| BR-028 | Application Preference Lookup with Facility Override: System preferences are stored in the Application Preference tables and can be overridden at facility level. HABADTE uses these preferences to determine display and behaviour (e.g., identifier mode). | HABADTE | 0.65 | Yes | Ensures behaviour is configurable and can vary by facility, enabling gradual rollout and local customisation while retaining central control. | Yes |
| BR-029 | Patient Counting — Three Running Totals: As each qualifying patient is processed, three counters are maintained (e.g., total, inpatient, leave). | HABADTE | 0.65 | No | Provides aggregated statistics used for capacity planning, performance monitoring and downstream analytics. | Yes |

---

## Testing Guidance Summary

- **Rules with confidence < 0.7** (most DATA_MAINTENANCE utilities and several HABADTE enrichments) are marked **Test Required = Yes**. These must have targeted unit tests and regression scenarios derived from production data or SME review.
- **High-confidence core filters** (BR-017, BR-018, BR-019, BR-021, BR-022) are still critical but may require fewer exploratory tests; focus on boundary conditions around dates and flags.
- **PHI-related rules** (BR-027, BR-028) must be tested not only functionally but also for compliance: correct masking, logging and access control in the modernised platform.
