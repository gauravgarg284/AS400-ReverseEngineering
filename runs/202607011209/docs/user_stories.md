# User Stories and Product Backlog

### HABADTE | Run: 202607011209

**Contents**
- Part 1: User Stories (by Epic)
- Part 2: Sprint Roadmap / Product Backlog

## Part 1: User Stories

# User Stories

**System:** HABADTE patient billing and transfer validation services on AS400 RPG platform.

**Run:** 1209 | **Total Stories:** 19 | **PHI Sensitive:** 0 | **PO Review Needed:** 12

---

### US-1209-001: When X equals zero branch to EXIT
**Epic:** Data Maintenance

As a system administrator,  
I want to When X equals zero, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When X equals zero, branch to 'EXIT'
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

### US-1209-002: When X equals 40 branch to EXIT
**Epic:** Data Maintenance

As a system administrator,  
I want to When X equals 40, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When X equals 40, branch to 'EXIT'
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

### US-1209-003: When VYY is less than 1800 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When VYY is less than 1800, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When VYY is less than 1800, branch to 'EXIT'
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

### US-1209-004: When VYY is greater than 2100 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When VYY is greater than 2100, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When VYY is greater than 2100, branch to 'EXIT'
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

### US-1209-005: When VMM is less than 01 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When VMM is less than 01, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When VMM is less than 01, branch to 'EXIT'
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

### US-1209-006: When VMM is greater than 12 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When VMM is greater than 12, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When VMM is greater than 12, branch to 'EXIT'
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

### US-1209-007: When VDD is less than 01 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When VDD is less than 01, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When VDD is less than 01, branch to 'EXIT'
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

### US-1209-008: When VDD is greater than DYSVMM branch to
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

### US-1209-009: When LDAMAP is greater than 99 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When LDAMAP is greater than 99, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When LDAMAP is greater than 99, branch to 'EXIT'
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

### US-1209-010: When LDAMAP is greater than 99 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When LDAMAP is greater than 99, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When LDAMAP is greater than 99, branch to 'EXIT'
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

### US-1209-011: When LDAMAP is greater than 99 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When LDAMAP is greater than 99, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When LDAMAP is greater than 99, branch to 'EXIT'
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

### US-1209-012: When LDAMAP is greater than 9999 branch to
**Epic:** Data Maintenance

As a system administrator,  
I want to When LDAMAP is greater than 9999, branch to 'EXIT',  
So that correctly filter records based on business criteria.

**Acceptance Criteria**
- AC-01: The system correctly applies: When LDAMAP is greater than 9999, branch to 'EXIT'
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

### US-1209-013: When IN79 equals onactive branch to EXIT
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

### US-1209-014: When IN79 equals onactive branch to EXIT
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
| Business Rule ID | BR-014 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-1209-015: When IN79 equals onactive branch to EXIT
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
| Business Rule ID | BR-015 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-1209-016: When IN79 equals onactive branch to EXIT
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
| Business Rule ID | BR-016 |
| Source Program | XFXTABL |
| Lines |  |
| Confidence | 0.90 |
| PHI Sensitive | No |

**Notes:** None

---

### US-1209-017: When FILE INDICATOR equals zero branch to SKIP
**Epic:** Patient Management

As a clinical staff member,  
I want to When -FILE INDICATOR equals zero, branch to 'SKIP',  
So that correctly filter records based on business criteria.

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

### US-1209-018: When FLAG INDICATOR equals voidvoided branch to SKIP
**Epic:** Patient Management

As a clinical staff member,  
I want to When -FLAG INDICATOR equals void/voided, branch to 'SKIP',  
So that correctly filter records based on business criteria.

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

### US-1209-019: When INPATIENTOUTPATIENT FLAG equals outpatient branch to SKIP
**Epic:** Patient Management

As a clinical staff member,  
I want to When -INPATIENT/OUTPATIENT FLAG equals outpatient, branch to 'SKIP',  
So that correctly filter records based on business criteria.

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

# HABADTE Product Backlog | Run: 202607011209 | DRAFT - Pending PO Review

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
- **Programs:** XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL
- **Story Count:** 16
- **Points:** TBD
- **PHI:** No

Stories: US-1209-001, US-1209-002, US-1209-003, US-1209-004, US-1209-005, US-1209-006, US-1209-007, US-1209-008, US-1209-009, US-1209-010, US-1209-011, US-1209-012, US-1209-013, US-1209-014, US-1209-015, US-1209-016

### E-002: Patient Management

- **Priority:** Medium
- **Domain:** PATIENT_MANAGEMENT
- **Programs:** HABADTE
- **Story Count:** 3
- **Points:** TBD
- **PHI:** No

Stories: US-1209-017, US-1209-018, US-1209-019

## 3. Sprint Roadmap

| Sprint | Theme | US-IDs | Points | Focus |
|--------|-------|--------|--------|-------|
| S1 | Foundation | US-1209-001, US-1209-002, US-1209-003, US-1209-004 | TBD | Baseline rule behaviour. |
| S2 | Core Logic | US-1209-005, US-1209-006, US-1209-007, US-1209-008 | TBD | Core business rule impl. |
| S3 | Workflows | US-1209-009, US-1209-010, US-1209-011, US-1209-012 | TBD | End-to-end workflow flows. |
| S4 | Compliance | US-1209-013, US-1209-014, US-1209-015, US-1209-016, US-1209-017, US-1209-018, US-1209-019 | TBD | PHI and audit controls. |
| S5 | Hardening | US-1209-001, US-1209-002, US-1209-003, US-1209-004, US-1209-005, US-1209-006, US-1209-007, US-1209-008, US-1209-009, US-1209-010, US-1209-011, US-1209-012, US-1209-013, US-1209-014, US-1209-015, US-1209-016, US-1209-017, US-1209-018, US-1209-019 | TBD | Regression and performance. |

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
