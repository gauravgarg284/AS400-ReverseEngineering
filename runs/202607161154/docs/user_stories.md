# User Stories and Product Backlog

### HABADTE | Run: 202607161154



**Contents**

- Part 1: User Stories (by Epic)

- Part 2: Sprint Roadmap / Product Backlog



## Part 1: User Stories

# User Stories

**System:** HABADTE patient admission and census reporting utilities on AS400.

**Run:** 2026 | **Total Stories:** 26 | **PHI Sensitive:** 0 | **PO Review Needed:** 20

---

### US-2026-001: Text centering skip processing if the entire input
**Epic:** Data Maintenance

As a system administrator,  
I want to Text centering — skip processing if the entire input field is blank (nothing to center).,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Text centering — skip processing if the entire input field is blank (nothing to center).
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-001 |
| Source Program | XFXCNTR |
| Lines |  |
| Confidence | 0.47 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.47) - Product Owner must verify this story.

---

### US-2026-002: Text centering skip processing if the text already
**Epic:** Data Maintenance

As a system administrator,  
I want to Text centering — skip processing if the text already starts at the first position (already left-aligned).,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Text centering — skip processing if the text already starts at the first position (already left-alig
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-002 |
| Source Program | XFXCNTR |
| Lines |  |
| Confidence | 0.40 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.40) - Product Owner must verify this story.

---

### US-2026-003: Date validation reject dates before 1800 the system
**Epic:** Data Maintenance

As a system administrator,  
I want to Date validation — reject dates before 1800: the system does not support historical records predating 1800.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation — reject dates before 1800: the system does not support historical records predating
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-003 |
| Source Program | XFXCYMD |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-004: Date validation reject dates beyond 2100 the system
**Epic:** Data Maintenance

As a system administrator,  
I want to Date validation — reject dates beyond 2100: the system does not support forecast dates past the year 2100.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation — reject dates beyond 2100: the system does not support forecast dates past the year
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-004 |
| Source Program | XFXCYMD |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-005: Date validation month must be January 01 or
**Epic:** Data Maintenance

As a system administrator,  
I want to Date validation — month must be January (01) or later: a month value below 01 is not a valid calendar month.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation — month must be January (01) or later: a month value below 01 is not a valid calenda
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-005 |
| Source Program | XFXCYMD |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-006: Date validation month must be December 12 or
**Epic:** Data Maintenance

As a system administrator,  
I want to Date validation — month must be December (12) or earlier: a month value above 12 is not a valid calendar month.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation — month must be December (12) or earlier: a month value above 12 is not a valid cale
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-006 |
| Source Program | XFXCYMD |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-007: Date validation day must be 01 or later
**Epic:** Data Maintenance

As a system administrator,  
I want to Date validation — day must be 01 or later: a day value of 0 or less is not a valid calendar day.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation — day must be 01 or later: a day value of 0 or less is not a valid calendar day.
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-007 |
| Source Program | XFXCYMD |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-008: When VDD is greater than DYSVMM branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When VDD is greater than DYS(VMM), branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When VDD is greater than DYS(VMM), branch to 'EXIT'
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-008 |
| Source Program | XFXCYMD |
| Lines |  |
| Confidence | 0.68 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.68) - Product Owner must verify this story.

---

### US-2026-009: Organizational level lookup the level code is out
**Epic:** Data Maintenance

As a system administrator,  
I want to Organizational level lookup — the level code is out of the valid range; return no description.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Organizational level lookup — the level code is out of the valid range; return no description.
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-009 |
| Source Program | XFXLDSC |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-010: Organizational level lookup the level code is out
**Epic:** Data Maintenance

As a system administrator,  
I want to Organizational level lookup — the level code is out of the valid range; return no description.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Organizational level lookup — the level code is out of the valid range; return no description.
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-012 |
| Source Program | XFXLDSC |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-011: When IN79 equals onactive branch to EXIT
**Epic:** Data Maintenance

As a system administrator,  
I want to When *IN79 equals on/active, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When *IN79 equals on/active, branch to 'EXIT'
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-013 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-012: Database table access HXXAPPPRF reads from the HXPAPPPRF
**Epic:** Data Maintenance

As a system administrator,  
I want to Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or reference data needed du,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Database table access — HXXAPPPRF reads from the 'HXPAPPPRF' table to retrieve configuration or refe
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-030 |
| Source Program | HXXAPPPRF |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-013: Database table access HXXAPPPRF reads from the HXPAPPL6
**Epic:** Data Maintenance

As a system administrator,  
I want to Database table access — HXXAPPPRF reads from the 'HXPAPPL6' table to retrieve configuration or reference data needed dur,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Database table access — HXXAPPPRF reads from the 'HXPAPPL6' table to retrieve configuration or refer
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-031 |
| Source Program | HXXAPPPRF |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-014: Exclude PreAdmitted Patients A patient with file indicator
**Epic:** Patient Management

As a clinical staff member,  
I want to Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formally admitted. These m,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Exclude Pre-Admitted Patients: A patient with file indicator = 0 is a pre-admission — not yet formal
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-017 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.92 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-015: Exclude Voided Accounts Accounts marked as Voided flag
**Epic:** Patient Management

As a clinical staff member,  
I want to Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never appear on the activ,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Exclude Voided Accounts: Accounts marked as Voided (flag = 'V') are cancelled records and must never
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-018 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.99 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-016: Exclude Outpatients Only inpatients appear on this census
**Epic:** Patient Management

As a clinical staff member,  
I want to Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpatient) are excluded.,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Exclude Outpatients: Only inpatients appear on this census. Accounts with I/O indicator = 'O' (Outpa
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-019 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.99 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-017: Organisational Level Filter Each patient record must belong
**Epic:** Patient Management

As a clinical staff member,  
I want to Organisational Level Filter: Each patient record must belong to the requested organisation. The report accepts an org le,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Organisational Level Filter: Each patient record must belong to the requested organisation. The repo
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-020 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-018: Census Date Admission Date Check A patient admitted
**Epic:** Patient Management

As a clinical staff member,  
I want to Census Date — Admission Date Check: A patient admitted after the census date is not yet active on that date. If the pati,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Census Date — Admission Date Check: A patient admitted after the census date is not yet active on th
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-021 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.85 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-019: Census Date Discharge Date Check A patient who
**Epic:** Patient Management

As a clinical staff member,  
I want to Census Date — Discharge Date Check: A patient who has already been discharged on or before the census date is no longer ,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Census Date — Discharge Date Check: A patient who has already been discharged on or before the censu
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-022 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.85 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-020: Room Number Retrieval from Transfer History The patients
**Epic:** Patient Management

As a clinical staff member,  
I want to Room Number Retrieval from Transfer History: The patient's current room number is not stored directly on the patient rec,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Room Number Retrieval from Transfer History: The patient's current room number is not stored directl
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-023 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-021: Room Class Description Lookup The 2character room class
**Epic:** Patient Management

As a clinical staff member,  
I want to Room Class Description Lookup: The 2-character room class code (AFNRMC) retrieved from the transfer history is resolved ,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Room Class Description Lookup: The 2-character room class code (AFNRMC) retrieved from the transfer 
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-024 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-022: Hospital Therapeutic Leave Flag When the room class
**Epic:** Patient Management

As a clinical staff member,  
I want to Hospital / Therapeutic Leave Flag: When the room class description is looked up, the HCS mapping field (XFDMAP) is also ,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Hospital / Therapeutic Leave Flag: When the room class description is looked up, the HCS mapping fie
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-025 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.57 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.57) - Product Owner must verify this story.

---

### US-2026-023: Nursing Station Name Resolution For each qualifying patient
**Epic:** Patient Management

As a clinical staff member,  
I want to Nursing Station Name Resolution: For each qualifying patient, the nursing station name is looked up from the Nursing Sta,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Nursing Station Name Resolution: For each qualifying patient, the nursing station name is looked up 
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-026 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-024: Patient Identifier Display MRN vs Account Number The
**Epic:** Patient Management

As a clinical staff member,  
I want to Patient Identifier Display — MRN vs Account Number: The report can show either the patient's Medical Record Number (MMMR,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Patient Identifier Display — MRN vs Account Number: The report can show either the patient's Medical
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-027 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-025: Application Preference Lookup with Facility Override System preferences
**Epic:** Patient Management

As a clinical staff member,  
I want to Application Preference Lookup with Facility Override: System preferences are stored in the Application Preference table ,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Application Preference Lookup with Facility Override: System preferences are stored in the Applicati
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-028 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-026: Patient Counting Three Running Totals As each qualifying
**Epic:** Patient Management

As a clinical staff member,  
I want to Patient Counting — Three Running Totals: As each qualifying patient is processed, three counters are maintained: (1) Tot,  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: Patient Counting — Three Running Totals: As each qualifying patient is processed, three counters are
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-029 |
| Source Program | HABADTE |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.



## Part 2: Sprint Roadmap / Product Backlog

# HABADTE Product Backlog | Run: 202607161154 | DRAFT - Pending PO Review

## 1. Product Vision & MVP

Vision: Modernize HABADTE from IBM i/AS400 to cloud-native preserving all validated business logic.

MVP Scope: PATIENT_MANAGEMENT / BILLING / SCHEDULING = Yes; others = Phase 2.

| # | Epic | Domain | In MVP | Rationale |
|---|------|--------|--------|-----------|
| 1 | Data Maintenance | DATA_MAINTENANCE | Phase 2 | Pending domain classification |
| 2 | Patient Management | PATIENT_MANAGEMENT | Yes | Core business domain |

Out of Scope: missing source nodes, dead code, low-confidence stories.

## 2. Epic Catalogue

### E-001: Data Maintenance

- **Priority:** Medium
- **Domain:** DATA_MAINTENANCE
- **Programs:** XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, HXXAPPPRF
- **Story Count:** 13
- **Points:** TBD
- **PHI:** No

Stories: US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012, US-2026-013

### E-002: Patient Management

- **Priority:** Medium
- **Domain:** PATIENT_MANAGEMENT
- **Programs:** HABADTE
- **Story Count:** 13
- **Points:** TBD
- **PHI:** No

Stories: US-2026-014, US-2026-015, US-2026-016, US-2026-017, US-2026-018, US-2026-019, US-2026-020, US-2026-021, US-2026-022, US-2026-023, US-2026-024, US-2026-025, US-2026-026

## 3. Sprint Roadmap

| Sprint | Theme | US-IDs | Points | Focus |
|--------|-------|--------|--------|-------|
| S1 | Foundation | US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006 | TBD | Baseline rule behaviour. |
| S2 | Core Logic | US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012 | TBD | Core business rule impl. |
| S3 | Workflows | US-2026-013, US-2026-014, US-2026-015, US-2026-016, US-2026-017, US-2026-018 | TBD | End-to-end workflow flows. |
| S4 | Compliance | US-2026-019, US-2026-020, US-2026-021, US-2026-022, US-2026-023, US-2026-024, US-2026-025, US-2026-026 | TBD | PHI and audit controls. |
| S5 | Hardening | US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012, US-2026-013, US-2026-014, US-2026-015, US-2026-016, US-2026-017, US-2026-018, US-2026-019, US-2026-020, US-2026-021, US-2026-022, US-2026-023, US-2026-024, US-2026-025, US-2026-026 | TBD | Regression and performance. |

## 4. Definition of Done

1. All acceptance criteria implemented and verified by QA.
2. Automated test coverage at or above 80% for changed components.
3. Mainframe SME has validated functional equivalence.
4. PHI masking and access controls verified where applicable.
5. Peer code review completed with no critical findings.
6. Integration tests executed successfully across dependent components.
7. Throughput and performance benchmarks meet agreed thresholds.
8. User and technical documentation updated and reviewed.
9. Static application security testing (SAST) shows no critical findings.
10. Product Owner has formally accepted the story.

## Appendix A: Missing Source

No HIGH or MEDIUM impact missing source nodes identified.
