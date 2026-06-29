# User Stories

**System:** AS400 reverse engineered system

**Run:** 2026 | **Total Stories:** 3 | **PHI Sensitive:** 0 | **PO Review Needed:** 0

---

### US-2026-001: When FILE INDICATOR equals zero branch to SKIP
**Epic:** Unclassified

As a system user,  
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
| Confidence | 0.87 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-002: When FLAG INDICATOR equals voidvoided branch to SKIP
**Epic:** Unclassified

As a system user,  
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
| Confidence | 0.95 |
| PHI Sensitive | No |

**Notes:** None

---

### US-2026-003: When INPATIENTOUTPATIENT FLAG equals outpatient branch to SKIP
**Epic:** Unclassified

As a system user,  
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
| Confidence | 0.95 |
| PHI Sensitive | No |

**Notes:** None