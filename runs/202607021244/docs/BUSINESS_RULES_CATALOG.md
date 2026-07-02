# Business Rules Catalog - HABADTE Project

This catalog lists all approved business rules extracted from the HABADTE codebase and related utility programs. Rules are grouped by domain and annotated with business impact and testing requirements.

## 1. Data Maintenance Domain

### 1.1 Centering and Position Control (XFXCNTR)

| BR-ID  | Rule Text                                      | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                | Test Required |
|--------|-------------------------------------------------|----------------|------------|------------------|-------------|----------------------------------------------------------------|--------------|
| BR-001 | When X equals zero, branch to 'EXIT'.          | XFXCNTR        | 0.47       | DATA_MAINTENANCE | false       | Prevents centering logic from running when starting position is invalid or uninitialized; avoids misaligned headings. | Yes          |
| BR-002 | When X equals 40, branch to 'EXIT'.            | XFXCNTR        | 0.40       | DATA_MAINTENANCE | false       | Exits centering when position reaches boundary (column 40), ensuring text does not overflow layout.                   | Yes          |

### 1.2 Date Validation (XFXCYMD)

| BR-ID  | Rule Text                                                      | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                                                  | Test Required |
|--------|-----------------------------------------------------------------|----------------|------------|------------------|-------------|--------------------------------------------------------------------------------------------------|--------------|
| BR-003 | When VYY is less than 1800, branch to 'EXIT'.                  | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Prevents processing of years before 1800, guarding against malformed or default year values.    | Yes          |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'.               | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Prevents dates far in the future; protects reporting logic from unrealistic future dates.        | Yes          |
| BR-005 | When VMM is less than 01, branch to 'EXIT'.                    | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Enforces month lower bound; stops processing when month is below January.                        | Yes          |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'.                 | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Enforces month upper bound; stops processing when month is beyond December.                      | Yes          |
| BR-007 | When VDD is less than 01, branch to 'EXIT'.                    | XFXCYMD        | 0.56       | DATA_MAINTENANCE | false       | Prevents days below 1; avoids invalid date-of-month values.                                      | Yes          |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'.           | XFXCYMD        | 0.68       | DATA_MAINTENANCE | false       | Ensures day does not exceed days-in-month (including leap-year logic via XFXLEAP).               | Yes          |

### 1.3 Level Mapping Validation (XFXLDSC)

| BR-ID  | Rule Text                                                      | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                                                  | Test Required |
|--------|-----------------------------------------------------------------|----------------|------------|------------------|-------------|--------------------------------------------------------------------------------------------------|--------------|
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'.              | XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Stops level description lookup when mapping code exceeds allowed two-digit range; avoids mislabelled facilities. | Yes          |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'.              | XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Duplicate condition ensuring mapping range enforcement across multiple branches.                 | Yes          |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'.              | XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Additional enforcement in different code path; collectively ensure consistent mapping validation. | Yes          |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'.            | XFXLDSC        | 0.56       | DATA_MAINTENANCE | false       | Guards against four-digit overflow in mapping codes; protects from invalid level combinations.    | Yes          |

### 1.4 Table-Driven Configuration Control (XFXTABL)

| BR-ID  | Rule Text                                                      | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                                                  | Test Required |
|--------|-----------------------------------------------------------------|----------------|------------|------------------|-------------|--------------------------------------------------------------------------------------------------|--------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'.                 | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Terminates table lookup when control indicator is active; prevents redundant or conflicting mappings. | No           |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'.                 | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Same behaviour in an alternate branch; ensures consistent exit when indicator is on.            | No           |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'.                 | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Further duplicate to cover additional mapping routines; keeps configuration reads bounded.      | No           |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'.                 | XFXTABL        | 0.90       | DATA_MAINTENANCE | false       | Ensures table processing stops when flagged as complete or disabled.                            | No           |

### 1.5 Application Profile Access (HXXAPPPRF)

| BR-ID  | Rule Text                                      | Source Program | Confidence | Domain           | PHI In Path | Business Impact                                                | Test Required |
|--------|-------------------------------------------------|----------------|------------|------------------|-------------|----------------------------------------------------------------|--------------|
| BR-020 | SQL program accesses table 'HXPAPPPRF'.        | HXXAPPPRF      | 0.65       | DATA_MAINTENANCE | false       | Reads application profile settings that drive MRN roll-up, report modes, and facility-level configuration. | Yes          |

(Interpretations detail also references BR-021 and BR-022 as profile-access rules; they follow the same pattern and will be documented in extended profile specs.)

## 2. Patient Management Domain

### 2.1 Census Record Inclusion/Exclusion (HABADTE)

| BR-ID  | Rule Text                                                      | Source Program | Confidence | Domain             | PHI In Path | Business Impact                                                                                  | Test Required |
|--------|-----------------------------------------------------------------|----------------|------------|--------------------|-------------|--------------------------------------------------------------------------------------------------|--------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'.            | HABADTE        | 0.92       | PATIENT_MANAGEMENT | false       | Excludes records with no active file indicator; ensures only valid accounts are counted in census. | No           |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'.     | HABADTE        | 0.99       | PATIENT_MANAGEMENT | false       | Prevents voided accounts from appearing in census; protects against counting cancelled or corrected encounters. | No           |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. | HABADTE  | 0.99       | PATIENT_MANAGEMENT | false       | Ensures outpatient records are excluded from inpatient census; maintains clinical and reporting integrity. | No           |

These rules act collectively to restrict the census to active inpatient accounts only.

## 3. Cross-Program Rule Relationships

- **Date validation rules (BR-003–BR-008)** underpin any logic that uses admit or transfer dates, including HABADTE’s selection of census snapshots.
- **Level mapping rules (BR-009–BR-012)** enforce correct facility and corporate hierarchy descriptions used in report headers.
- **Table configuration rules (BR-013–BR-016)** safeguard room class and benefit lookups executed both inside HABADTE and shared utility functions.
- **Profile access rule (BR-020)** feeds MRN roll-up decisions consumed by XFXMRNROL and ultimately HABADTE.
- **Census inclusion rules (BR-017–BR-019)** are central to patient management and must be mirrored exactly in any migrated implementation.

Each rule listed above must be preserved in the modernization effort. Rules marked "Test Required: Yes" require explicit unit and integration test cases, while high-confidence rules (>= 0.7) still require regression coverage but are less likely to be misinterpreted.
