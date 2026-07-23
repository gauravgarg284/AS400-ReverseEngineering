# Business Rules Catalog – HABADTE Project

This catalog lists all approved business rules extracted from the HABADTE application and its supporting utilities, grouped by domain. Each entry includes the originating program, confidence score, PHI involvement, inferred business impact, and testing requirement.

## 1. Data Maintenance Domain

### 1.1 XFXCNTR – Text Centering Utility

#### BR-001 – Text Centering — Blank String Optimization

- **Source Program**: XFXCNTR (RPGLE)
- **Confidence Score**: 0.47
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Before attempting to center a string for screen display or report formatting, the system checks if the input is completely blank.
- **PHI in Path**: false
- **Business Impact (Plain-English)**: Prevents unnecessary processing and ensures that empty headings or labels are not altered, which reduces CPU usage and avoids misleading display of blanks as meaningful content.
- **Test Requirement**: Yes (confidence < 0.7)

#### BR-002 – Text Centering — Full String Optimization

- **Source Program**: XFXCNTR (RPGLE)
- **Confidence Score**: 0.40
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: When formatting text for UI screens or reports, the system checks if the string is already occupying the maximum field width.
- **PHI in Path**: false
- **Business Impact**: Avoids shifting or re-centering strings that already fill the output area (e.g., 40 characters wide), preserving layout and preventing truncation.
- **Test Requirement**: Yes

---

### 1.2 XFXCYMD – Date Validation Utility

#### BR-003 – Year Minimum Validation

- **Source Program**: XFXCYMD (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Date validation: reject year < 1800 (historical minimum).
- **PHI in Path**: false
- **Business Impact**: Prevents entry or processing of clearly invalid historical dates, reducing data corruption and ensuring reports operate within a realistic historical range.
- **Test Requirement**: Yes

#### BR-004 – Year Maximum Validation

- **Source Program**: XFXCYMD (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Date validation: reject year > 2100 (forecast maximum).
- **PHI in Path**: false
- **Business Impact**: Stops future dates far beyond plausible planning horizons from entering the system, avoiding erroneous scheduling or projections.
- **Test Requirement**: Yes

#### BR-005 – Month Minimum Validation

- **Source Program**: XFXCYMD (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Date validation: reject month < 01 (calendar constraint).
- **PHI in Path**: false
- **Business Impact**: Ensures month values obey calendar rules and prevents malformed dates, which is important for downstream reporting and calculations.
- **Test Requirement**: Yes

#### BR-006 – Month Maximum Validation

- **Source Program**: XFXCYMD (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Date validation: reject month > 12 (calendar constraint).
- **PHI in Path**: false
- **Business Impact**: Prevents invalid month values from being stored or processed.
- **Test Requirement**: Yes

#### BR-007 – Day Minimum Validation

- **Source Program**: XFXCYMD (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Date validation: reject day < 01 (calendar constraint).
- **PHI in Path**: false
- **Business Impact**: Guards against impossible day values, protecting the integrity of admission, discharge, and census dates.
- **Test Requirement**: Yes

#### BR-008 – Day vs Month Boundary Check

- **Source Program**: XFXCYMD (RPGLE)
- **Confidence Score**: 0.68
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: When VDD is greater than DYS(VMM), branch to 'EXIT'.
- **PHI in Path**: false
- **Business Impact**: Ensures that the day component of a date does not exceed the maximum days for a given month (including leap-year handling), preventing invalid dates such as 31 February.
- **Test Requirement**: Yes (confidence < 0.7)

---

### 1.3 XFXLDSC – Level Description Lookup

#### BR-009 – Level Code Range Validation

- **Source Program**: XFXLDSC (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Level lookup: reject if level code exceeds valid range.
- **PHI in Path**: false
- **Business Impact**: Prevents invalid organisational level codes from being used when resolving hierarchy descriptions, avoiding misclassification of facilities or departments.
- **Test Requirement**: Yes

#### BR-012 – Level Code Range Validation (Variant)

- **Source Program**: XFXLDSC (RPGLE)
- **Confidence Score**: 0.56
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: Level lookup: reject if level code exceeds valid range.
- **PHI in Path**: false
- **Business Impact**: Reinforces the same validation across multiple code paths, ensuring consistent hierarchy usage.
- **Test Requirement**: Yes

---

### 1.4 XFXTABL – Generic Table Lookup

#### BR-013 – Indicator-Based Exit

- **Source Program**: XFXTABL (RPGLE)
- **Confidence Score**: 0.90
- **Domain**: DATA_MAINTENANCE
- **Rule Text**: When *IN79 equals on/active, branch to 'EXIT'.
- **PHI in Path**: false
- **Business Impact**: Provides an early-exit mechanism controlled by a status indicator, allowing calling programs to bypass table lookups when not needed (e.g., during error or special-case conditions).
- **Test Requirement**: No (confidence ≥ 0.7)

---

## 2. Patient Management Domain

### 2.1 HABADTE – Active Census Report

#### BR-017 – Exclude Pre-Admitted Patients

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.92
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census.
- **PHI in Path**: false (logic operates on status indicators, though underlying accounts are PHI-bearing).
- **Business Impact**: Keeps census focused on active inpatients, preventing over-counting by excluding accounts that have not yet resulted in an admission.
- **Test Requirement**: No

#### BR-018 – Exclude Voided Accounts

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.99
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census.
- **PHI in Path**: false (status flag only).
- **Business Impact**: Prevents invalid or cancelled accounts from influencing census counts or operational decisions.
- **Test Requirement**: No

#### BR-019 – Exclude Outpatients

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.99
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.
- **PHI in Path**: false.
- **Business Impact**: Ensures the census reflects inpatient occupancy only, which is critical for bed management and staffing calculations.
- **Test Requirement**: No

#### BR-020 – Organisational Level Filter

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.65
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Organisational Level Filter: Each patient record must belong to the requested organisation. The report accepts an org level (1–6) and org code from the caller. If the patient's org code at that level does not match, the record is excluded.
- **PHI in Path**: false (filters on non-PHI org codes).
- **Business Impact**: Aligns the census with a specific organisational scope (e.g., particular facility or unit), enabling accurate local census reporting.
- **Test Requirement**: Yes (confidence < 0.7)

#### BR-021 – Admission Date Check

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.85
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Census Date — Admission Date Check: A patient admitted after the census date is not yet active on that date. If the patient's admission date is later than the requested census date, the patient is excluded from the report.
- **PHI in Path**: false (uses dates only).
- **Business Impact**: Prevents future admissions from appearing in current census snapshots, keeping occupancy metrics accurate.
- **Test Requirement**: No

#### BR-022 – Discharge Date Check

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.85
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Census Date — Discharge Date Check: A patient who has already been discharged on or before the census date is no longer active. If discharge date is set (non-zero) and is on or before the census date, the patient is excluded. A discharge date of zero means the patient has not been discharged and is always considered active (subject to other filters).
- **PHI in Path**: false.
- **Business Impact**: Ensures census counts reflect current in-hospital presence, not historical stays.
- **Test Requirement**: No

#### BR-023 – Room Number Retrieval from Transfer History

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.65
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Room Number Retrieval from Transfer History: The patient's current room number is not stored directly on the patient record. Instead, it is taken from the most recent entry in the ATD Transfer History file.
- **PHI in Path**: true (roomNumber and accountNumber are PHI in production).
- **Business Impact**: Ensures room assignments reflect actual movement history, making the census accurate for bed control and nursing station workflows.
- **Test Requirement**: Yes

#### BR-024 – Room Class Description Lookup

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.65
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Room Class Description Lookup: The 2-character room class code (AFNRMC) retrieved from the transfer history is resolved to a human-readable description. The lookup uses the system reference table (HXPDICT/HXPTABLD).
- **PHI in Path**: false (room class is typically non-PHI classification).
- **Business Impact**: Enables operational reporting by category (e.g., ICU vs general ward), supporting staffing and financial analytics.
- **Test Requirement**: Yes

#### BR-025 – Hospital / Therapeutic Leave Flag

- **Source Program**: HABADTE (RPGLE)
- **Confidence Score**: 0.57
- **Domain**: PATIENT_MANAGEMENT
- **Rule Text**: Hospital / Therapeutic Leave Flag: When the room class description is looked up, the HCS mapping field (XFDMAP) is also examined. If the 10th character of XFDMAP is 'H', the patient is on Hospital Leave; if it is 'T', the patient is on Therapeutic Leave.
- **PHI in Path**: false (status flag only).
- **Business Impact**: Distinguishes different kinds of leave for operational and clinical reporting, affecting how beds and staffing are counted.
- **Test Requirement**: Yes

---

## 3. Testing Prioritization Summary

Rules requiring explicit test coverage (confidence < 0.7):

- BR-001, BR-002 (text centering edge behavior).
- BR-003–BR-008 (date validation boundaries and leap-year handling).
- BR-009, BR-012 (hierarchy level code validation).
- BR-020 (organisational filter correctness).
- BR-023 (transfer history-based room resolution).
- BR-024 (room class description lookup mapping).
- BR-025 (leave flag derivation from XFDMAP).

Rules with high confidence and lower testing priority (still require regression tests but less exploratory focus):

- BR-013 (indicator-based early exit for table lookup).
- BR-017–BR-019 (core inclusion/exclusion filters for census).
- BR-021–BR-022 (admission/discharge date boundaries).
