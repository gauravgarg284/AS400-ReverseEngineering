# BUSINESS RULES CATALOG – HABADTE Suite

Run ID: 202607021710  
Project: HABADTE AS400 admission and data maintenance suite

This catalog lists all approved business rules extracted from the legacy AS400 RPG and SQLRPGLE programs, grouped by domain.

---

## 1. Data Maintenance Domain

### Program: XFXCNTR (RPGLE)

| Rule ID | Rule Text                                     | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|-----------------------------------------------|----------------|------------|-------------|-----------------|---------------|
| BR-001  | When X equals zero, branch to 'EXIT'          | XFXCNTR        | 0.47       | No          | Controls early exit when a counter reaches zero, preventing further processing in loops or batches. | Yes |
| BR-002  | When X equals 40, branch to 'EXIT'            | XFXCNTR        | 0.40       | No          | Enforces an upper limit of 40 iterations or units, ensuring processing does not exceed configured thresholds. | Yes |

### Program: XFXCYMD (RPGLE)

| Rule ID | Rule Text                                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|-------------------------------------------------------------|----------------|------------|-------------|-----------------|---------------|
| BR-003  | When VYY is less than 1800, branch to 'EXIT'                | XFXCYMD        | 0.56       | No          | Prevents use of dates before year 1800, avoiding invalid historical dates in admission or reporting. | Yes |
| BR-004  | When VYY is greater than 2100, branch to 'EXIT'             | XFXCYMD        | 0.56       | No          | Disallows dates beyond year 2100, limiting future-dated records to a reasonable window. | Yes |
| BR-005  | When VMM is less than 01, branch to 'EXIT'                  | XFXCYMD        | 0.56       | No          | Rejects months less than January, ensuring basic calendar validity. | Yes |
| BR-006  | When VMM is greater than 12, branch to 'EXIT'               | XFXCYMD        | 0.56       | No          | Rejects months greater than December, enforcing standard calendar rules. | Yes |
| BR-007  | When VDD is less than 01, branch to 'EXIT'                  | XFXCYMD        | 0.56       | No          | Disallows zero or negative day values, preventing malformed dates. | Yes |
| BR-008  | When VDD is greater than DYS(VMM), branch to 'EXIT'         | XFXCYMD        | 0.68       | No          | Ensures day-of-month does not exceed the number of days in the given month (accounting for leap years via XFXLEAP). | Yes |

### Program: XFXLDSC (RPGLE)

| Rule ID | Rule Text                                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|-------------------------------------------------------------|----------------|------------|-------------|-----------------|---------------|
| BR-009  | When LDAMAP is greater than 99, branch to 'EXIT'            | XFXLDSC        | 0.56       | No          | Prevents use of mapping codes beyond two digits, protecting level-to-description mappings from configuration errors. | Yes |
| BR-010  | When LDAMAP is greater than 99, branch to 'EXIT'            | XFXLDSC        | 0.56       | No          | Duplicate threshold enforcement; indicates multiple code paths relying on the same mapping limit. | Yes |
| BR-011  | When LDAMAP is greater than 99, branch to 'EXIT'            | XFXLDSC        | 0.56       | No          | Additional guardrail on mapping range, supporting different branches in the same program. | Yes |
| BR-012  | When LDAMAP is greater than 9999, branch to 'EXIT'          | XFXLDSC        | 0.56       | No          | Extends mapping validation to four-digit codes, ensuring high-range mappings remain within defined boundaries. | Yes |

### Program: XFXTABL (RPGLE)

| Rule ID | Rule Text                                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|-------------------------------------------------------------|----------------|------------|-------------|-----------------|---------------|
| BR-013  | When *IN79 equals on/active, branch to 'EXIT'               | XFXTABL        | 0.90       | No          | Uses indicator *IN79 to short-circuit processing when a dictionary entry or operation is already active, avoiding duplicates. | No |
| BR-014  | When *IN79 equals on/active, branch to 'EXIT'               | XFXTABL        | 0.90       | No          | Mirrors BR-013 in a different branch, reinforcing the same active-state guard. | No |
| BR-015  | When *IN79 equals on/active, branch to 'EXIT'               | XFXTABL        | 0.90       | No          | Additional application of the active indicator rule, suggesting reusable pattern across table operations. | No |
| BR-016  | When *IN79 equals on/active, branch to 'EXIT'               | XFXTABL        | 0.90       | No          | Fourth occurrence, highlighting system-wide reliance on on/active checks to prevent redundant processing. | No |

### Program: HXXAPPPRF (SQLRPGLE)

| Rule ID | Rule Text                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|---------------------------------------------|----------------|------------|-------------|-----------------|---------------|
| BR-020  | SQL program accesses table 'HXPAPPPRF'      | HXXAPPPRF      | 0.65       | No          | Centralizes application profile changes in an SQL-accessible table, enabling profile-based controls and auditing. | Yes |

---

## 2. Patient Management Domain

### Program: HABADTE (RPGLE)

| Rule ID | Rule Text                                                   | Source Program | Confidence | PHI In Path | Business Impact | Test Required |
|---------|-------------------------------------------------------------|----------------|------------|-------------|-----------------|---------------|
| BR-017  | When -FILE INDICATOR equals zero, branch to 'SKIP'          | HABADTE        | 0.92       | No          | Excludes inactive or uninitialized records from admission processing, improving data quality and avoiding noise. | No |
| BR-018  | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'   | HABADTE        | 0.99       | No          | Prevents cancelled or voided encounters from entering the admission workflow, ensuring financial and clinical accuracy. | No |
| BR-019  | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' | HABADTE | 0.99 | No | Ensures only inpatient encounters are included in inpatient admission workflows, separating outpatient processing paths. | No |

---

## 3. Cross-Cutting Observations

- All **Data Maintenance** rules guard configuration and reference data: date ranges, mapping codes and active indicators in dictionaries.
- **Patient Management** rules in HABADTE are high-confidence and focus on filtering out records that should not be part of the inpatient admission stream.
- No rule directly traverses PHI fields; PHI presence arises from table schemas (HAPTRFR, OMPMAST, HXPDICT, OXPBNFIT, OAPIRNK), but the rules themselves operate on status and control fields.

---

## 4. Testing Priorities

Rules requiring explicit test coverage (confidence < 0.7 or complex impact):

- BR-001, BR-002 (XFXCNTR) – boundary tests for counter values (0, 40, >40).
- BR-003–BR-008 (XFXCYMD) – date validation tests across valid/invalid years, months and days, including leap-year scenarios.
- BR-009–BR-012 (XFXLDSC) – mapping-range tests with LDAMAP at thresholds (99, 100, 9999, 10000).
- BR-020 (HXXAPPPRF) – SQL access tests ensuring correct profile table usage and transactional behavior.

Rules considered stable (confidence ≥ 0.9) but still needing regression tests:

- BR-013–BR-016 (XFXTABL) – indicator-driven early-exit logic.
- BR-017–BR-019 (HABADTE) – core admission filters (file indicator, void, outpatient).