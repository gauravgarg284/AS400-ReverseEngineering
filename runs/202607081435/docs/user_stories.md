# User Stories and Product Backlog

### HABADTE | Run: 202607081435

**Contents**
- Part 1: User Stories (by Epic)
- Part 2: Sprint Roadmap / Product Backlog

## Part 1: User Stories

# User Stories

**System:** HABADTE: Legacy AS400 application for patient transaction and benefit table maintenance.

**Run:** 2026 | **Total Stories:** 20 | **PHI Sensitive:** 0 | **PO Review Needed:** 13

---

### US-2026-001: Field formatting exit if all blank 40 chars
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Field formatting: exit if all blank (40 chars),  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Field formatting: exit if all blank (40 chars)
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

### US-2026-002: Field formatting exit if first char nonblank
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Field formatting: exit if first char non-blank,  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Field formatting: exit if first char non-blank
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

### US-2026-003: Date validation reject year 1800 historical minimum
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Date validation: reject year < 1800 (historical minimum),  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation: reject year < 1800 (historical minimum)
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

### US-2026-004: Date validation reject year 2100 forecast maximum
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Date validation: reject year > 2100 (forecast maximum),  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation: reject year > 2100 (forecast maximum)
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

### US-2026-005: Date validation reject month 01 calendar constraint
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Date validation: reject month < 01 (calendar constraint),  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation: reject month < 01 (calendar constraint)
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

### US-2026-006: Date validation reject month 12 calendar constraint
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Date validation: reject month > 12 (calendar constraint),  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation: reject month > 12 (calendar constraint)
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

### US-2026-007: Date validation reject day 01 calendar constraint
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Date validation: reject day < 01 (calendar constraint),  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Date validation: reject day < 01 (calendar constraint)
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
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to When VDD is greater than DYS(VMM), branch to 'EXIT',  
So that support correct business operation.

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

### US-2026-009: Level lookup reject if level code exceeds valid
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Level lookup: reject if level code exceeds valid range,  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Level lookup: reject if level code exceeds valid range
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

### US-2026-010: Level lookup reject if level code exceeds valid
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Level lookup: reject if level code exceeds valid range,  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Level lookup: reject if level code exceeds valid range
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-010 |
| Source Program | XFXLDSC |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-011: Level lookup reject if level code exceeds valid
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Level lookup: reject if level code exceeds valid range,  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Level lookup: reject if level code exceeds valid range
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-011 |
| Source Program | XFXLDSC |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-012: Level lookup reject if level code exceeds valid
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to Level lookup: reject if level code exceeds valid range,  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: Level lookup: reject if level code exceeds valid range
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

### US-2026-013: When IN79 equals onactive branch to EXIT
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to When *IN79 equals on/active, branch to 'EXIT',  
So that support correct business operation.

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

### US-2026-014: When IN79 equals onactive branch to EXIT
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to When *IN79 equals on/active, branch to 'EXIT',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: When *IN79 equals on/active, branch to 'EXIT'
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-014 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-015: When IN79 equals onactive branch to EXIT
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to When *IN79 equals on/active, branch to 'EXIT',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: When *IN79 equals on/active, branch to 'EXIT'
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-015 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-016: When IN79 equals onactive branch to EXIT
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to When *IN79 equals on/active, branch to 'EXIT',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: When *IN79 equals on/active, branch to 'EXIT'
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-016 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-017: SQL program accesses table HXPAPPPRF
**Epic:** Data Maintenance Controls

As a system administrator,  
I want to SQL program accesses table 'HXPAPPPRF',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: SQL program accesses table 'HXPAPPPRF'
- AC-02: The system behaves correctly when the rule condition is not met (negative path)

**Source Traceability**
| Field | Value |
|-------|-------|
| Business Rule ID | BR-020 |
| Source Program | HXXAPPPRF |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-018: When FILE INDICATOR equals zero branch to SKIP
**Epic:** Patient Management Controls

As a clinical staff member,  
I want to When -FILE INDICATOR equals zero, branch to 'SKIP',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: When -FILE INDICATOR equals zero, branch to 'SKIP'
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

### US-2026-019: When FLAG INDICATOR equals voidvoided branch to SKIP
**Epic:** Patient Management Controls

As a clinical staff member,  
I want to When -FLAG INDICATOR equals void/voided, branch to 'SKIP',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: When -FLAG INDICATOR equals void/voided, branch to 'SKIP'
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

### US-2026-020: When INPATIENTOUTPATIENT FLAG equals outpatient branch to SKIP
**Epic:** Patient Management Controls

As a clinical staff member,  
I want to When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP',  
So that support correct business operation.

**Acceptance Criteria**
- AC-01: The system correctly applies: When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP'
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

## Part 2: Sprint Roadmap / Product Backlog

# HABADTE Product Backlog | Run: 202607081435 | DRAFT - Pending PO Review

## 1. Product Vision & MVP

Vision: Modernize HABADTE from IBM i/AS400 to cloud-native preserving all validated business logic.

MVP Scope: PATIENT_MANAGEMENT / BILLING / SCHEDULING = Yes; others = Phase 2.

| # | Epic | Domain | In MVP | Rationale |
|---|------|--------|--------|-----------|
| 1 | Data Maintenance Controls | DATA_MAINTENANCE | Phase 2 | Pending domain classification |
| 2 | Patient Management Controls | PATIENT_MANAGEMENT | Yes | Core business domain |

Out of Scope: missing source nodes, dead code, low-confidence stories.

## 2. Epic Catalogue

### E-001: Data Maintenance Controls

- **Priority:** Medium
- **Domain:** DATA_MAINTENANCE
- **Programs:** XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, HXXAPPPRF, XFXGETID, XFXLEAP, XFXMRNROL
- **Story Count:** 17
- **Points:** TBD
- **PHI:** No

Stories: US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012, US-2026-013, US-2026-014, US-2026-015, US-2026-016, US-2026-017

### E-002: Patient Management Controls

- **Priority:** Medium
- **Domain:** PATIENT_MANAGEMENT
- **Programs:** HABADTE
- **Story Count:** 3
- **Points:** TBD
- **PHI:** No

Stories: US-2026-018, US-2026-019, US-2026-020

## 3. Sprint Roadmap

| Sprint | Theme | US-IDs | Points | Focus |
|--------|-------|--------|--------|-------|
| S1 | Foundation | US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005 | TBD | Baseline rule behaviour. |
| S2 | Core Logic | US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010 | TBD | Core business rule impl. |
| S3 | Workflows | US-2026-011, US-2026-012, US-2026-013, US-2026-014, US-2026-015 | TBD | End-to-end workflow flows. |
| S4 | Compliance | US-2026-016, US-2026-017, US-2026-018, US-2026-019, US-2026-020 | TBD | PHI and audit controls. |
| S5 | Hardening | US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012, US-2026-013, US-2026-014, US-2026-015, US-2026-016, US-2026-017, US-2026-018, US-2026-019, US-2026-020 | TBD | Regression and performance. |

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
