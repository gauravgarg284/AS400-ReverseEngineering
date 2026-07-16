# User Stories

**System:** HABADTE - Active Patients by Admit Date reporting and shared data maintenance utilities

**Run:** 2026 | **Total Stories:** 22 | **PHI Sensitive:** 0 | **PO Review Needed:** 15

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
| Business Rule ID | BR-010 |
| Source Program | XFXLDSC |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-011: Organizational level lookup the level code is out
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
| Business Rule ID | BR-011 |
| Source Program | XFXLDSC |
| Lines |  |
| Confidence | 0.56 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.56) - Product Owner must verify this story.

---

### US-2026-012: Organizational level lookup the level code is out
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

### US-2026-013: When IN79 equals onactive branch to EXIT
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

### US-2026-014: When IN79 equals onactive branch to EXIT
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

### US-2026-015: When IN79 equals onactive branch to EXIT
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

### US-2026-016: When IN79 equals onactive branch to EXIT
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

### US-2026-017: Database table access HXXAPPPRF reads from the HXPAPPPRF
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
| Business Rule ID | BR-020 |
| Source Program | HXXAPPPRF |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-018: Database table access HXXAPPPRF reads from the HXPAPPL6
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
| Business Rule ID | BR-021 |
| Source Program | HXXAPPPRF |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-019: Database table access HXXAPPPRF reads from the HXPAPPPRF
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
| Business Rule ID | BR-022 |
| Source Program | HXXAPPPRF |
| Lines |  |
| Confidence | 0.65 |
| PHI Sensitive | No |

**Notes:** None
WARNING: Low confidence rule (score 0.65) - Product Owner must verify this story.

---

### US-2026-020: Exclude PreAdmitted Patients A patient with file indicator
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

### US-2026-021: Exclude Voided Accounts Accounts marked as Voided flag
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

### US-2026-022: Exclude Outpatients Only inpatients appear on this census
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