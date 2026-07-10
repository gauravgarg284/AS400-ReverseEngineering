# BUSINESS_RULES_CATALOG – HABADTE Application

This catalog lists all approved business rules extracted from the HABADTE codebase and related utility programs. Each rule entry includes metadata, PHI path information, inferred business impact, and whether additional targeted testing is required during modernization.

---

## 1. Rules by Domain

### 1.1 DATA_MAINTENANCE Domain

#### XFXCNTR – Counter / Formatting Utility

| Rule ID | Rule Text | Source Program | Confidence | Domain | PHI in Path | Business Impact | Test Required |
|--------:|----------|----------------|-----------:|--------|-------------|-----------------|--------------|
| BR-001 | Field formatting: exit if all blank (40 chars) | XFXCNTR | 0.47 | DATA_MAINTENANCE | false | Prevents allocation of new counters/values when the input field is entirely blank, avoiding creation of meaningless identifiers. | Yes |
| BR-002 | Field formatting: exit if first char non-blank | XFXCNTR | 0.40 | DATA_MAINTENANCE | false | Ensures fields expected to be blank (e.g., for new assignment) are not reused if already populated, preventing accidental overwrite of existing values. | Yes |

#### XFXCYMD – Date Validation Utility

| Rule ID | Rule Text | Source Program | Confidence | Domain | PHI in Path | Business Impact | Test Required |
|--------:|----------|----------------|-----------:|--------|-------------|-----------------|--------------|
| BR-003 | Date validation: reject year < 1800 (historical minimum) | XFXCYMD | 0.56 | DATA_MAINTENANCE | false | Prevents obviously invalid historical dates from being accepted, ensuring reports and calculations do not include spurious events. | Yes |
| BR-004 | Date validation: reject year > 2100 (forecast maximum) | XFXCYMD | 0.56 | DATA_MAINTENANCE | false | Guards against data-entry or interface errors that push dates far into the future, which could break aging and forecasting logic. | Yes |
| BR-005 | Date validation: reject month < 01 (calendar constraint) | XFXCYMD | 0.56 | DATA_MAINTENANCE | false | Enforces calendar correctness to prevent malformed month values that would fail downstream date handling. | Yes |
| BR-006 | Date validation: reject month > 12 (calendar constraint) | XFXCYMD | 0.56 | DATA_MAINTENANCE | false | Ensures month codes remain within 1–12, improving reliability of scheduling and interval calculations. | Yes |
| BR-007 | Date validation: reject day < 01 (calendar constraint) | XFXCYMD | 0.56 | DATA_MAINTENANCE | false | Avoids impossible day values (e.g., 00) that may corrupt timelines or reporting logic. | Yes |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT' | XFXCYMD | 0.68 | DATA_MAINTENANCE | false | Detects cases where the provided day-of-month exceeds the valid days for the specified month/year and rejects the date, preventing invalid calendar combinations. | Yes |

#### XFXLDSC – Level Description Lookup

| Rule ID | Rule Text | Source Program | Confidence | Domain | PHI in Path | Business Impact | Test Required |
|--------:|----------|----------------|-----------:|--------|-------------|-----------------|--------------|
| BR-009 | Level lookup: reject if level code exceeds valid range | XFXLDSC | 0.56 | DATA_MAINTENANCE | false | Stops processing of invalid level codes, ensuring that plan or facility hierarchies remain coherent. | Yes |
| BR-010 | Level lookup: reject if level code exceeds valid range | XFXLDSC | 0.56 | DATA_MAINTENANCE | false | Same as BR-009, applied to a different level or branch in the lookup logic; protects integrity of level-based routing. | Yes |
| BR-011 | Level lookup: reject if level code exceeds valid range | XFXLDSC | 0.56 | DATA_MAINTENANCE | false | Additional guard for another level dimension; prevents misclassification of encounters or benefits. | Yes |
| BR-012 | Level lookup: reject if level code exceeds valid range | XFXLDSC | 0.56 | DATA_MAINTENANCE | false | Final defensive check in the level lookup chain; avoids falling through with undefined level definitions. | Yes |

#### XFXTABL – Generic Table Lookup

| Rule ID | Rule Text | Source Program | Confidence | Domain | PHI in Path | Business Impact | Test Required |
|--------:|----------|----------------|-----------:|--------|-------------|-----------------|--------------|
| BR-013 | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL | 0.90 | DATA_MAINTENANCE | false | Indicates that a controlling indicator (*IN79) can short-circuit table processing, enabling configuration-driven bypass of certain lookups. | No |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL | 0.90 | DATA_MAINTENANCE | false | Mirror of BR-013 for a different code path or table; ensures consistent behavior when the control flag is set. | No |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL | 0.90 | DATA_MAINTENANCE | false | Additional branch using the same indicator; reduces runtime processing when lookups are disabled by configuration. | No |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT' | XFXTABL | 0.90 | DATA_MAINTENANCE | false | Final use of the same rule pattern; ensures that all variants of the table lookup respect the same global enable/disable flag. | No |

#### HXXAPPPRF – Application Profile SQL Program

| Rule ID | Rule Text | Source Program | Confidence | Domain | PHI in Path | Business Impact | Test Required |
|--------:|----------|----------------|-----------:|--------|-------------|-----------------|--------------|
| BR-020 | SQL program accesses table 'HXPAPPPRF' | HXXAPPPRF | 0.65 | DATA_MAINTENANCE | false | Indicates that application profiles are stored in HXPAPPPRF and must be migrated, preserving their structure and access patterns. | Yes |

### 1.2 PATIENT_MANAGEMENT Domain

#### HABADTE – Patient Transfer Driver

| Rule ID | Rule Text | Source Program | Confidence | Domain | PHI in Path | Business Impact | Test Required |
|--------:|----------|----------------|-----------:|--------|-------------|-----------------|--------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP' | HABADTE | 0.92 | PATIENT_MANAGEMENT | false | Ensures that only records explicitly marked for processing are exported, preventing accidental inclusion of draft or staging transfers. | No |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP' | HABADTE | 0.99 | PATIENT_MANAGEMENT | false | Protects downstream consumers from receiving voided or cancelled transfer records, aligning XML output with official patient history. | No |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP' | HABADTE | 0.99 | PATIENT_MANAGEMENT | false | Supports configuration of inpatient-only runs by excluding outpatient transfers, ensuring reporting matches clinical expectations. | No |

---

## 2. PHI Exposure Overview

Although none of the individual rules above directly manipulate PHI fields, several operate over tables that contain PHI (HAPTRFR, OMPMAST, HXPDICT, OAPIRNK, OXPBNFIT). For modernization:

- Treat any rule that gatekeeps record inclusion (BR-017–BR-019) as **PHI-relevant** because they indirectly control which PHI-bearing records are propagated to XML outputs and external integrations.
- Ensure audit logging captures when and why records are skipped, without logging full PHI values unnecessarily.

---

## 3. Testing Strategy Summary

- Rules with **confidence < 0.7** (BR-001–BR-008, BR-009–BR-012, BR-020) must have explicit, data-driven test cases covering positive and negative scenarios.
- HABADTE’s high-confidence filters (BR-017–BR-019) are still critical and should be validated with end-to-end regression tests against a representative data set.
- Utility rules in XFXCYMD and XFXLDSC should be tested with boundary values (e.g., year 1799/1800/2100/2101; invalid vs valid level codes) to confirm behavior matches the legacy implementation.
