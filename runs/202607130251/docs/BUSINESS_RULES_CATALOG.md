# Business Rules Catalog – HABADTE Application

This catalog lists all approved business rules identified in the HABADTE subsystem, grouped by domain. Each entry includes the originating program, confidence score, PHI involvement, inferred business impact, and whether explicit test coverage is required.

---

## Domain: PATIENT_MANAGEMENT

### HABADTE – Census Filtering Rules

#### BR-017 – Exclude Pre-Admitted Patients

- **Rule Text:** "Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These must be excluded from the active census."
- **Source Program:** HABADTE
- **Confidence Score:** 0.92
- **Domain:** PATIENT_MANAGEMENT
- **PHI in Path:** false
- **Business Impact (Plain English):**
  - Ensures that the census shows only formally admitted patients.
  - Prevents inflating occupancy counts with pre‑admission records, which would distort bed utilisation and staffing decisions.
- **Test Requirement:** No (confidence ≥ 0.7, but should still be included in regression suites).

#### BR-018 – Exclude Voided Accounts

- **Rule Text:** "Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the active census."
- **Source Program:** HABADTE
- **Confidence Score:** 0.99
- **Domain:** PATIENT_MANAGEMENT
- **PHI in Path:** false
- **Business Impact:**
  - Prevents cancelled or erroneous accounts from appearing as active patients.
  - Reduces financial and clinical reconciliation errors caused by showing voided accounts in operational dashboards.
- **Test Requirement:** No.

#### BR-019 – Exclude Outpatients

- **Rule Text:** "Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded."
- **Source Program:** HABADTE
- **Confidence Score:** 0.99
- **Domain:** PATIENT_MANAGEMENT
- **PHI in Path:** false
- **Business Impact:**
  - Ensures that inpatient census metrics are not polluted with outpatient visits.
  - Supports accurate bed management, inpatient staffing, and occupancy‑rate reporting.
- **Test Requirement:** No.

---

## Domain: DATA_MAINTENANCE

### XFXCNTR – Text Centering Utility

#### BR-001 – Skip Blank Input

- **Rule Text:** "Text centering — skip processing if the entire input field is blank (nothing to center)."
- **Source Program:** XFXCNTR
- **Confidence Score:** 0.47
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Avoids unnecessary processing and potential formatting artefacts for empty fields.
  - Ensures that blank labels or headings remain blank rather than being altered.
- **Test Requirement:** Yes (confidence < 0.7). Include unit tests verifying that blank inputs are returned unchanged and do not cause errors.

#### BR-002 – Skip Already Left-Aligned Text

- **Rule Text:** "Text centering — skip processing if the text already starts at the first position (already left-aligned)."
- **Source Program:** XFXCNTR
- **Confidence Score:** 0.4
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Preserves existing alignment when centering logic is unnecessary.
  - Reduces CPU usage and avoids shifting text that is already properly formatted.
- **Test Requirement:** Yes (confidence < 0.7). Tests should assert that left‑aligned text remains unchanged.

### XFXCYMD – Date Validation

#### BR-003 – Reject Dates Before 1800

- **Rule Text:** "Date validation — reject dates before 1800: the system does not support historical records predating 1800."
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Prevents impossible or irrelevant historical dates from entering the system.
  - Helps avoid miscalculations in age and date‑based analytics.
- **Test Requirement:** Yes. Test both boundary (1799-12-31 rejected, 1800-01-01 accepted) and error handling.

#### BR-004 – Reject Dates Beyond 2100

- **Rule Text:** "Date validation — reject dates beyond 2100: the system does not support forecast dates past the year 2100."
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Prevents unrealistic future dates that could break reports and projections.
  - Keeps time‑based business logic within a defined operational horizon.
- **Test Requirement:** Yes. Validate rejection of dates > 2100‑12‑31 and acceptance of boundary values.

#### BR-005 – Month Must Be January or Later

- **Rule Text:** "Date validation — month must be January (01) or later: a month value below 01 is not a valid calendar month."
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Ensures numeric month values represent real calendar months.
  - Prevents downstream parsing errors in reporting and scheduling.
- **Test Requirement:** Yes. Include tests for month=00 and negative values.

#### BR-006 – Month Must Be December or Earlier

- **Rule Text:** "Date validation — month must be December (12) or earlier: a month value above 12 is not a valid calendar month."
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Guards against invalid upper‑bound month values.
  - Supports reliable calendar computations and UI rendering.
- **Test Requirement:** Yes. Include tests for month=13 and higher.

#### BR-007 – Day Must Be 01 or Later

- **Rule Text:** "Date validation — day must be 01 or later: a day value of 0 or less is not a valid calendar day."
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.56
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Prevents impossible day values from entering date fields.
  - Avoids corrupt date records that would fail age and duration calculations.
- **Test Requirement:** Yes. Verify that day=0 and negative days are rejected.

#### BR-008 – VDD Greater Than DYS(VMM) Branches to EXIT

- **Rule Text:** "When VDD is greater than DYS(VMM), branch to 'EXIT'"
- **Source Program:** XFXCYMD
- **Confidence Score:** 0.68
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Acts as an overflow or invalid‑day safeguard when comparing day values against the maximum days in a month.
  - Ensures that invalid day combinations (like 31 in February) cause control to exit the validation routine rather than silently accepting bad dates.
- **Test Requirement:** Yes (confidence < 0.7). Tests must cover months with 28, 30, and 31 days.

### XFXLDSC – Organisational Level Description Lookup

#### BR-009 / BR-010 / BR-011 / BR-012 – Out-of-Range Level Code Returns No Description

- **Rule Text (representative):** "Organizational level lookup — the level code is out of the valid range; return no description."
- **Source Program:** XFXLDSC
- **Confidence Score:** 0.56 (each)
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Prevents misleading or incorrect organisational descriptions when level codes are invalid.
  - Helps upstream processes detect configuration gaps in level hierarchies.
- **Test Requirement:** Yes. Design tests with invalid level codes at each level (1–6) and verify empty descriptions and appropriate error flags.

### XFXTABL – Table-Driven Logic

#### BR-013 / BR-014 / BR-015 / BR-016 – Indicator *IN79 Causes Exit

- **Rule Text (representative):** "When *IN79 equals on/active, branch to 'EXIT'"
- **Source Program:** XFXTABL
- **Confidence Score:** 0.9 (each)
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Provides a configurable stop condition for table‑driven processing.
  - Allows operations to disable or short‑circuit certain dictionary loops based on configuration or status.
- **Test Requirement:** No (high confidence), but include regression tests to verify that *IN79=on always halts processing and *IN79=off allows it to continue.

### HXXAPPPRF – Application Profile Table Access

#### BR-020 – Read HXPAPPPRF for Configuration Data

- **Rule Text:** "Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed during processing."
- **Source Program:** HXXAPPPRF
- **Confidence Score:** 0.65
- **Domain:** DATA_MAINTENANCE
- **PHI in Path:** false
- **Business Impact:**
  - Centralizes configuration and reference data used by patient census and related workflows.
  - Any changes to HXPAPPPRF directly affect processing preferences (e.g., default filters, output formats, or organisational mappings).
- **Test Requirement:** Yes. Tests must validate that configuration is correctly loaded, that missing or corrupt config entries produce safe defaults or explicit errors, and that changes in HXPAPPPRF are reflected in behaviour.

---

## PHI Summary

Although none of the rules explicitly reference PHI fields, the following PFs participate in workflows where rules are applied:

- **HAPTRFR** – AFACCT (AccountNumber), AFMRNO (MRN).
- **HXPDICT** – MRN, AccountNumber, PhoneNumber, PatientName, RoomNumber, DateOfBirth fields.
- **OAPIRNK** – BRKMRN (MRN).
- **OMPMAST** – MRNs, account numbers, names, SSNs.
- **OXPBNFIT** – XFBTEL (PhoneNumber).

Security and compliance tests must ensure that:
- Rule implementations do not leak PHI in logs or error messages.
- Access to rule‑driven queries over PHI tables is restricted to authorised roles.
- All PHI‑touching operations are auditable.
