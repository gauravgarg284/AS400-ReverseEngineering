# Business Rules Catalog

## Overview

This catalog lists all approved business rules extracted from the HABADTE and related programs. Each entry includes its originating program, confidence score, PHI path indicator, inferred business impact, and whether explicit testing is required during migration.

## Rules by Domain

### DATA_MAINTENANCE Domain

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-001 | When X equals zero, branch to 'EXIT'. | XFXCNTR | 0.47 | No | Controls flow based on a counter field; mis-implementation could cause premature termination or missed processing steps. | Yes |
| BR-002 | When X equals 40, branch to 'EXIT'. | XFXCNTR | 0.40 | No | Enforces an upper bound on a counter or loop; incorrect handling might lead to runaway loops or performance issues. | Yes |
| BR-003 | When VYY is less than 1800, branch to 'EXIT'. | XFXCYMD | 0.56 | No | Rejects dates with year below 1800; protects against invalid historical dates and data corruption in downstream processes. | Yes |
| BR-004 | When VYY is greater than 2100, branch to 'EXIT'. | XFXCYMD | 0.56 | No | Rejects future dates beyond 2100; prevents unrealistic dates that could distort analytics. | Yes |
| BR-005 | When VMM is less than 01, branch to 'EXIT'. | XFXCYMD | 0.56 | No | Validates month lower bound; ensures month values are within calendar range. | Yes |
| BR-006 | When VMM is greater than 12, branch to 'EXIT'. | XFXCYMD | 0.56 | No | Validates month upper bound; prevents impossible month values. | Yes |
| BR-007 | When VDD is less than 01, branch to 'EXIT'. | XFXCYMD | 0.56 | No | Validates day lower bound; protects against zero or negative day values. | Yes |
| BR-008 | When VDD is greater than DYS(VMM), branch to 'EXIT'. | XFXCYMD | 0.68 | No | Validates day upper bound based on month-specific maximum; prevents invalid dates such as 31 February. | Yes |
| BR-009 | When LDAMAP is greater than 99, branch to 'EXIT'. | XFXLDSC | 0.56 | No | Enforces range on level mapping values; guards against unexpected mapping codes. | Yes |
| BR-010 | When LDAMAP is greater than 99, branch to 'EXIT'. | XFXLDSC | 0.56 | No | Duplicate or variant of BR-009; consistency across code paths is important. | Yes |
| BR-011 | When LDAMAP is greater than 99, branch to 'EXIT'. | XFXLDSC | 0.56 | No | Additional occurrence of LDAMAP range enforcement; ensures integrity of level description mapping. | Yes |
| BR-012 | When LDAMAP is greater than 9999, branch to 'EXIT'. | XFXLDSC | 0.56 | No | Enforces a broader upper bound on mapping identifiers; protects against overflow or mis-keyed codes. | Yes |
| BR-013 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Uses an indicator to stop processing when a condition is met; often tied to table-driven controls. | No |
| BR-014 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Additional branch controlled by the same indicator; consistency with BR-013 is required. | No |
| BR-015 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Further usage of indicator-driven exit logic; ensures table loops or reads terminate correctly. | No |
| BR-016 | When *IN79 equals on/active, branch to 'EXIT'. | XFXTABL | 0.90 | No | Reinforces the same indicator semantics; migration must preserve the indicator state model. | No |

### PATIENT_MANAGEMENT Domain

| Rule ID | Rule Text | Source Program | Confidence | PHI in Path | Business Impact | Test Required |
|---------|-----------|----------------|------------|-------------|-----------------|--------------|
| BR-017 | When -FILE INDICATOR equals zero, branch to 'SKIP'. | HABADTE | 0.92 | No | Ensures records flagged as logically removed or invalid are excluded from patient transfer extracts; failure would expose incorrect data. | No |
| BR-018 | When -FLAG INDICATOR equals void/voided, branch to 'SKIP'. | HABADTE | 0.99 | No | Prevents voided/cancelled transfers from being reported; critical for accurate billing and clinical movement history. | No |
| BR-019 | When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'. | HABADTE | 0.99 | No | Separates inpatient from outpatient activity; mis-implementation could mix populations and distort inpatient metrics. | No |
