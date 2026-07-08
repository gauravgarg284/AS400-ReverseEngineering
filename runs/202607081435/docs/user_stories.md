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