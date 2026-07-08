# BUSINESS RULES CATALOG – HABADTE

## 1. Overview

This catalog lists all approved business rules extracted from the HABADTE application and related utility programs. Rules are grouped by domain and documented with their source program, confidence score, PHI involvement, inferred business impact, and test requirements.

## 2. Rules by Domain

---

## 2.1 DATA_MAINTENANCE Domain

### BR-001 – Field Formatting: Exit If All Blank (40 Chars)

- **Rule ID:** BR-001
- **Source Program:** XFXCNTR (RPGLE)
- **Confidence Score:** 0.47
- **PHI In Path:** No

**Rule Text**

Field formatting: exit if all blank (40 chars).

**Plain-English Business Impact**

This rule ensures that 40-character input fields are not processed when they contain only blanks. It prevents meaningless or empty data from triggering downstream logic, improving data quality in maintenance screens and batch jobs.

**Test Requirement**

- **Required:** Yes (confidence < 0.7)
- **Focus:** Verify that all-blank inputs cause an immediate exit and that non-blank inputs proceed as expected.

---

### BR-002 – Field Formatting: Exit If First Char Non-Blank

- **Rule ID:** BR-002
- **Source Program:** XFXCNTR (RPGLE)
- **Confidence Score:** 0.40
- **PHI In Path:** No

**Rule Text**

Field formatting: exit if first char non-blank.

**Plain-English Business Impact**

The rule enforces a specific formatting convention where the first character being non-blank implies that the field should not be reformatted or processed further. It protects special codes or prefixes that must remain untouched.

**Test Requirement**

- **Required:** Yes
- **Focus:** Confirm that inputs with a non-blank first character bypass formatting logic, while those with a blank first character are processed normally.

---

### BR-003 – Date Validation: Reject Year < 1800

- **Rule ID:** BR-003
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Date validation: reject year < 1800 (historical minimum).

**Plain-English Business Impact**

This rule prevents entry or processing of dates earlier than 1800, which are considered invalid for patient or transaction records. It protects against data-entry mistakes such as accidentally typing 1700-series years.

**Test Requirement**

- **Required:** Yes
- **Focus:** Verify that dates with year < 1800 are rejected and that borderline values (e.g., 1800-01-01) are accepted.

---

### BR-004 – Date Validation: Reject Year > 2100

- **Rule ID:** BR-004
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Date validation: reject year > 2100 (forecast maximum).

**Plain-English Business Impact**

This rule ensures that future dates beyond 2100 are treated as invalid. It avoids unrealistic scheduling or transfer dates that could cause reporting and billing anomalies.

**Test Requirement**

- **Required:** Yes
- **Focus:** Validate rejection of dates with year > 2100, and acceptance of 2099/2100 boundary cases.

---

### BR-005 – Date Validation: Reject Month < 01

- **Rule ID:** BR-005
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Date validation: reject month < 01 (calendar constraint).

**Plain-English Business Impact**

Rejecting months less than 01 prevents malformed dates such as 00-XX-YYYY, which could slip through numeric-only checks. It preserves calendar integrity and downstream processing correctness.

**Test Requirement**

- **Required:** Yes
- **Focus:** Ensure that any month value 0 or negative is rejected; month = 01 should be accepted.

---

### BR-006 – Date Validation: Reject Month > 12

- **Rule ID:** BR-006
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Date validation: reject month > 12 (calendar constraint).

**Plain-English Business Impact**

This rule rejects month values greater than 12, preventing invalid dates such as 13-XX-YYYY. It keeps all stored dates within the standard calendar.

**Test Requirement**

- **Required:** Yes
- **Focus:** Test month values 12 and 13 to confirm correct behavior.

---

### BR-007 – Date Validation: Reject Day < 01

- **Rule ID:** BR-007
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Date validation: reject day < 01 (calendar constraint).

**Plain-English Business Impact**

This rule prevents malformed dates with day values less than 1, protecting against data-entry mistakes and ensuring all dates are calendar-valid.

**Test Requirement**

- **Required:** Yes
- **Focus:** Confirm that day values 0 or negative are rejected; day = 01 is accepted.

---

### BR-008 – Exit When VDD > DYS(VMM)

- **Rule ID:** BR-008
- **Source Program:** XFXCYMD (RPGLE)
- **Confidence Score:** 0.68
- **PHI In Path:** No

**Rule Text**

When VDD is greater than DYS(VMM), branch to 'EXIT'.

**Plain-English Business Impact**

This rule ensures that a validation date (VDD) does not exceed the calculated number of days in the current month (DYS(VMM)). It protects against impossible dates such as February 30 or April 31, reinforcing leap-year and month-length logic.

**Test Requirement**

- **Required:** Yes (confidence < 0.7)
- **Focus:** Test typical months and leap-year February to confirm exit behavior when the day exceeds the month’s maximum.

---

### BR-009 – Level Lookup: Reject If Level Code Exceeds Valid Range

- **Rule ID:** BR-009
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Level lookup: reject if level code exceeds valid range.

**Plain-English Business Impact**

This rule ensures that configuration-driven level codes remain within configured ranges for hierarchical location tables. It prevents misconfigured or obsolete level codes from being accepted.

**Test Requirement**

- **Required:** Yes
- **Focus:** Verify behavior when level code is within range vs above the configured maximum.

---

### BR-010 – Level Lookup: Reject If Level Code Exceeds Valid Range

- **Rule ID:** BR-010
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Level lookup: reject if level code exceeds valid range.

**Plain-English Business Impact**

A repeated variant of BR-009, likely applied to a different hierarchical level (e.g., Level 2 or Level 3). It protects the integrity of multi-level location configuration.

**Test Requirement**

- **Required:** Yes
- **Focus:** Similar to BR-009; must be tested for the corresponding level table.

---

### BR-011 – Level Lookup: Reject If Level Code Exceeds Valid Range

- **Rule ID:** BR-011
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Level lookup: reject if level code exceeds valid range.

**Plain-English Business Impact**

Another variant covering additional level hierarchies. Improper level codes could cause mis-routing of transfers or incorrect reporting.

**Test Requirement**

- **Required:** Yes
- **Focus:** Verify correct rejection and logging for invalid codes at the associated level.

---

### BR-012 – Level Lookup: Reject If Level Code Exceeds Valid Range

- **Rule ID:** BR-012
- **Source Program:** XFXLDSC (RPGLE)
- **Confidence Score:** 0.56
- **PHI In Path:** No

**Rule Text**

Level lookup: reject if level code exceeds valid range.

**Plain-English Business Impact**

The final variant of the level range checks, ensuring all levels in the hierarchy are governed by consistent boundaries.

**Test Requirement**

- **Required:** Yes
- **Focus:** Confirm that invalid level codes are rejected at this level as well.

---

### BR-013 – Exit When *IN79 Equals On/Active (Variant 1)

- **Rule ID:** BR-013
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.90
- **PHI In Path:** No

**Rule Text**

When *IN79 equals on/active, branch to 'EXIT'.

**Plain-English Business Impact**

This rule uses indicator *IN79 to short-circuit table lookup logic when a certain condition is active. It likely protects against redundant or conflicting table searches, improving performance and avoiding duplicate processing.

**Test Requirement**

- **Required:** Yes (despite high confidence, critical control logic)
- **Focus:** Ensure that when *IN79 is "on", the routine exits; when "off", normal table lookup proceeds.

---

### BR-014 – Exit When *IN79 Equals On/Active (Variant 2)

- **Rule ID:** BR-014
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.90
- **PHI In Path:** No

**Rule Text**

When *IN79 equals on/active, branch to 'EXIT'.

**Plain-English Business Impact**

Additional branch of the same indicator-driven logic, likely in another section of the program. It maintains consistency of control flow wherever *IN79 is used.

**Test Requirement**

- **Required:** Yes
- **Focus:** Validate behavior across all contexts where *IN79 is referenced.

---

### BR-015 – Exit When *IN79 Equals On/Active (Variant 3)

- **Rule ID:** BR-015
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.90
- **PHI In Path:** No

**Rule Text**

When *IN79 equals on/active, branch to 'EXIT'.

**Plain-English Business Impact**

Third variant controlling table lookup sections. It reinforces that once a certain condition is met, further search or processing should halt.

**Test Requirement**

- **Required:** Yes
- **Focus:** Same as BR-013/014, ensuring consistent behavior.

---

### BR-016 – Exit When *IN79 Equals On/Active (Variant 4)

- **Rule ID:** BR-016
- **Source Program:** XFXTABL (RPGLE)
- **Confidence Score:** 0.90
- **PHI In Path:** No

**Rule Text**

When *IN79 equals on/active, branch to 'EXIT'.

**Plain-English Business Impact**

Fourth occurrence, reinforcing a pattern of using indicator *IN79 as a guard across multiple branches. Failure to implement this correctly could cause duplicated or conflicting table results.

**Test Requirement**

- **Required:** Yes
- **Focus:** Verify all program paths respect the indicator’s semantics.

---

### BR-020 – SQL Program Accesses Table 'HXPAPPPRF'

- **Rule ID:** BR-020
- **Source Program:** HXXAPPPRF (SQLRPGLE)
- **Confidence Score:** 0.65
- **PHI In Path:** No (table not flagged as PHI in context)

**Rule Text**

SQL program accesses table 'HXPAPPPRF'.

**Plain-English Business Impact**

This rule indicates that the application profile SQL program reads or updates the HXPAPPPRF table to manage application-level preferences or profiles. Correct implementation is important for ensuring that MRN rollover and related operations interact properly with application configuration.

**Test Requirement**

- **Required:** Yes (confidence < 0.7)
- **Focus:** Confirm that SQL operations target the right table, preserve referential integrity, and handle failure scenarios gracefully.

---

## 2.2 PATIENT_MANAGEMENT Domain

### BR-017 – Skip When File Indicator Equals Zero

- **Rule ID:** BR-017
- **Source Program:** HABADTE (RPGLE)
- **Confidence Score:** 0.92
- **PHI In Path:** No (indicator, not patient fields)

**Rule Text**

When -FILE INDICATOR equals zero, branch to 'SKIP'.

**Plain-English Business Impact**

This rule prevents HABADTE from processing records when the associated file context is inactive or empty. It reduces the risk of generating incomplete or misleading transfer outputs when source data is not properly initialized.

**Test Requirement**

- **Required:** No (confidence ≥ 0.7), but recommended due to core control behavior.

---

### BR-018 – Skip When Flag Indicator Equals Void/VoidED

- **Rule ID:** BR-018
- **Source Program:** HABADTE (RPGLE)
- **Confidence Score:** 0.99
- **PHI In Path:** No

**Rule Text**

When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.

**Plain-English Business Impact**

This rule ensures that logically voided transfer records are excluded from reporting and downstream processing. It preserves data integrity and avoids double-counting or misrepresenting cancelled transfers.

**Test Requirement**

- **Required:** No (very high confidence), but functional tests should still cover void scenarios as part of acceptance.

---

### BR-019 – Skip When Inpatient/Outpatient Flag Equals Outpatient

- **Rule ID:** BR-019
- **Source Program:** HABADTE (RPGLE)
- **Confidence Score:** 0.99
- **PHI In Path:** No

**Rule Text**

When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'.

**Plain-English Business Impact**

The rule focuses HABADTE’s outputs on inpatient records only, excluding outpatient encounters from the transfer audit report. It aligns the process scope with bed-management and inpatient billing requirements.

**Test Requirement**

- **Required:** No (very high confidence), but regression tests must cover inpatient vs outpatient scenarios to prove behavior.

---

## 3. Summary Table

The following consolidated table summarizes all rules.

| Rule ID | Domain             | Source Program | Confidence | PHI In Path | Business Impact (Short)                                              | Test Required |
|---------|--------------------|----------------|------------|------------|---------------------------------------------------------------------|--------------|
| BR-001  | DATA_MAINTENANCE   | XFXCNTR        | 0.47       | No         | Prevent processing of all-blank 40-char fields.                    | Yes          |
| BR-002  | DATA_MAINTENANCE   | XFXCNTR        | 0.40       | No         | Bypass formatting when first character is non-blank.               | Yes          |
| BR-003  | DATA_MAINTENANCE   | XFXCYMD        | 0.56       | No         | Reject dates with year before 1800.                                | Yes          |
| BR-004  | DATA_MAINTENANCE   | XFXCYMD        | 0.56       | No         | Reject dates with year after 2100.                                 | Yes          |
| BR-005  | DATA_MAINTENANCE   | XFXCYMD        | 0.56       | No         | Reject months less than 01.                                        | Yes          |
| BR-006  | DATA_MAINTENANCE   | XFXCYMD        | 0.56       | No         | Reject months greater than 12.                                     | Yes          |
| BR-007  | DATA_MAINTENANCE   | XFXCYMD        | 0.56       | No         | Reject days less than 01.                                          | Yes          |
| BR-008  | DATA_MAINTENANCE   | XFXCYMD        | 0.68       | No         | Exit when day exceeds month length (e.g., February 30).           | Yes          |
| BR-009  | DATA_MAINTENANCE   | XFXLDSC        | 0.56       | No         | Reject level codes above valid range (level variant 1).           | Yes          |
| BR-010  | DATA_MAINTENANCE   | XFXLDSC        | 0.56       | No         | Reject level codes above valid range (variant 2).                 | Yes          |
| BR-011  | DATA_MAINTENANCE   | XFXLDSC        | 0.56       | No         | Reject level codes above valid range (variant 3).                 | Yes          |
| BR-012  | DATA_MAINTENANCE   | XFXLDSC        | 0.56       | No         | Reject level codes above valid range (variant 4).                 | Yes          |
| BR-013  | DATA_MAINTENANCE   | XFXTABL        | 0.90       | No         | Exit table logic when indicator *IN79 is active (branch 1).       | Yes          |
| BR-014  | DATA_MAINTENANCE   | XFXTABL        | 0.90       | No         | Exit table logic when indicator *IN79 is active (branch 2).       | Yes          |
| BR-015  | DATA_MAINTENANCE   | XFXTABL        | 0.90       | No         | Exit table logic when indicator *IN79 is active (branch 3).       | Yes          |
| BR-016  | DATA_MAINTENANCE   | XFXTABL        | 0.90       | No         | Exit table logic when indicator *IN79 is active (branch 4).       | Yes          |
| BR-017  | PATIENT_MANAGEMENT | HABADTE        | 0.92       | No         | Skip records when file indicator is zero.                         | No           |
| BR-018  | PATIENT_MANAGEMENT | HABADTE        | 0.99       | No         | Skip voided transfer records.                                     | No           |
| BR-019  | PATIENT_MANAGEMENT | HABADTE        | 0.99       | No         | Skip outpatient encounters, focus on inpatients.                  | No           |
| BR-020  | DATA_MAINTENANCE   | HXXAPPPRF      | 0.65       | No         | Ensure SQL program uses HXPAPPPRF table for application profiles. | Yes          |
