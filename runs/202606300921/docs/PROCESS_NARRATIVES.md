# Process Narratives — HABADTE

## DATA_MAINTENANCE Domain

### Program: XFXCNTR

#### Overview

XFXCNTR is a small utility program used in the Data Maintenance domain to validate counter values and control processing loops. It ensures that counters stay within expected ranges and terminates processing when thresholds are reached.

#### Trigger

Called by other programs (e.g., HABADTE or utilities) whenever a counter-based loop or iteration needs validation.

#### Process Flow

1. Receive current counter value X.
2. Check if X equals zero (BR-001); if so, branch to EXIT.
3. Check if X equals 40 (BR-002); if so, branch to EXIT.
4. If neither condition is met, continue processing.

#### Business Rules Applied

- BR-001: When X equals zero, branch to 'EXIT'.
- BR-002: When X equals 40, branch to 'EXIT'.

#### Error Handling

- If X is outside expected range, processing is terminated via EXIT, preventing further operations.

#### Data Used

- Counter variable X.

#### Outputs Produced

- Control flow decision (continue vs EXIT); no persistent data output.

---

### Program: XFXCYMD

#### Overview

XFXCYMD validates and interprets date values expressed as year (VYY), month (VMM), and day (VDD). It ensures that dates fall within reasonable historical and future ranges and that day values are valid for the given month.

#### Trigger

Called by other programs whenever a date needs validation or conversion (e.g., when constructing report dates or effective dates).

#### Process Flow

1. Receive VYY, VMM, VDD.
2. Validate year range using BR-003 and BR-004.
3. Validate month range using BR-005 and BR-006.
4. Validate day range using BR-007 and BR-008.
5. If all checks pass, proceed with date conversion to internal format.

#### Business Rules Applied

- BR-003: Year < 1800 → EXIT.
- BR-004: Year > 2100 → EXIT.
- BR-005: Month < 01 → EXIT.
- BR-006: Month > 12 → EXIT.
- BR-007: Day < 01 → EXIT.
- BR-008: Day > DYS(VMM) → EXIT.

#### Error Handling

- If any rule fails, the program branches to EXIT and returns an invalid date status to the caller.

#### Data Used

- VYY, VMM, VDD, and derived days-in-month via DYS(VMM).

#### Outputs Produced

- Valid/invalid date indicator; possibly a converted date value.

---

### Program: XFXLDSC

#### Overview

XFXLDSC validates Local Data Area (LDA) mapping values (LDAMAP) used for context or configuration. It ensures that mapping codes fall within defined ranges.

#### Trigger

Called during initialization or configuration loading when LDAMAP values are read or set.

#### Process Flow

1. Receive LDAMAP value.
2. Apply BR-009, BR-010, BR-011 to check if LDAMAP > 99.
3. Apply BR-012 to check if LDAMAP > 9999.
4. If any condition is true, branch to EXIT; otherwise, continue.

#### Business Rules Applied

- BR-009, BR-010, BR-011: LDAMAP > 99 → EXIT.
- BR-012: LDAMAP > 9999 → EXIT.

#### Error Handling

- Invalid LDAMAP values cause an EXIT, signalling configuration errors to the caller.

#### Data Used

- LDAMAP from LDA or configuration sources.

#### Outputs Produced

- Valid/invalid mapping indicator; no direct persistent data output.

---

### Program: XFXTABL

#### Overview

XFXTABL is a table-driven branching utility that uses indicator *IN79 and table entries to determine whether processing should continue or exit. It centralizes configuration-based control logic.

#### Trigger

Called when table-based configuration needs to control processing, such as enabling/disabling features or paths.

#### Process Flow

1. Load relevant table entries from HXPTABLD.
2. Evaluate indicator *IN79.
3. If *IN79 is *ON (BR-013–BR-016), branch to EXIT.
4. If *IN79 is *OFF, continue processing.

#### Business Rules Applied

- BR-013–BR-016: Indicator *IN79 = *ON → EXIT.

#### Error Handling

- Misconfigured indicators result in EXIT, preventing unintended processing paths.

#### Data Used

- Indicator *IN79.
- Table entries from HXPTABLD.

#### Outputs Produced

- Control flow decision (continue vs EXIT).

---

## PATIENT_MANAGEMENT Domain

### Program: HABADTE

#### Overview

HABADTE is the main patient management batch program. It reads patient and account records, applies exclusion filters, enriches data from multiple sources, and produces XML and printer outputs for downstream systems.

#### Trigger

Executed as a scheduled batch job or manually triggered run, typically once per day or per billing cycle.

#### Process Flow (Happy Path)

1. Initialize run context, including date, institution, and preferences.
2. Load organizational hierarchy from HXPLVL1–HXPLVL6.
3. Query patient master records from OMPMAST (and related files).
4. For each record, apply BR-017–BR-019 to determine inclusion.
5. For included records, perform data enrichment (rank, benefit, institution, dictionary).
6. Assemble XML messages and spool output lines.
7. Update counters and summary statistics.
8. Write final XML and printer outputs.

#### Business Rules Applied

- BR-017: File indicator = 0 → SKIP.
- BR-018: Void flag = 'V' → SKIP.
- BR-019: Outpatient flag = 'O' → SKIP.

These rules ensure that only valid, non-voided inpatient records are processed.

#### Error Handling

- Records failing any rule are skipped; counters track reasons (invalid file indicator, void, outpatient).
- Missing enrichment data (rank, benefit, institution, dictionary) does not cause errors; fields are left null and optionally flagged.

#### Data Used

- Master patient data (OMPMAST).
- Transfer records (HAPTRFR).
- Rank data (OAPIRNK/TAPIRNK via HAPIRNK).
- Benefit data (OXPBNFIT/TXPBNFIT via HXPBNFIT).
- Institution data (OXPNSTN/TXPNSTN via HXPNSTN).
- Dictionary data (HXPDICT).
- XML definitions and messages (HXPXMLD/HXPXMLR).
- Lookup tables (HXPTABLD/HXLTABLS/HXLTABLP/HXLTABLD).

#### Outputs Produced

- XML messages written to HXPXMLR (to be migrated to an XML/JSON messaging endpoint).
- Printer output written to PRINTER (to be migrated to reporting/printing services).
- Aggregated counters and summary statistics for monitoring.
