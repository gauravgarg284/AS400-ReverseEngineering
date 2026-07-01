# HABADTE Product Backlog | Run: 202607011038 | DRAFT - Pending PO Review

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

Stories: US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012, US-2026-013, US-2026-014, US-2026-015, US-2026-016

### E-002: Patient Management

- **Priority:** Medium
- **Domain:** PATIENT_MANAGEMENT
- **Programs:** HABADTE
- **Story Count:** 3
- **Points:** TBD
- **PHI:** No

Stories: US-2026-017, US-2026-018, US-2026-019

## 3. Sprint Roadmap

| Sprint | Theme | US-IDs | Points | Focus |
|--------|-------|--------|--------|-------|
| S1 | Foundation | US-2026-001, US-2026-002, US-2026-003, US-2026-004 | TBD | Baseline rule behaviour. |
| S2 | Core Logic | US-2026-005, US-2026-006, US-2026-007, US-2026-008 | TBD | Core business rule impl. |
| S3 | Workflows | US-2026-009, US-2026-010, US-2026-011, US-2026-012 | TBD | End-to-end workflow flows. |
| S4 | Compliance | US-2026-013, US-2026-014, US-2026-015, US-2026-016, US-2026-017, US-2026-018, US-2026-019 | TBD | PHI and audit controls. |
| S5 | Hardening | US-2026-001, US-2026-002, US-2026-003, US-2026-004, US-2026-005, US-2026-006, US-2026-007, US-2026-008, US-2026-009, US-2026-010, US-2026-011, US-2026-012, US-2026-013, US-2026-014, US-2026-015, US-2026-016, US-2026-017, US-2026-018, US-2026-019 | TBD | Regression and performance. |

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

| Name | Impact | Referenced By |
|------|--------|---------------|
| CXXXMLC | HIGH | HABADTE |
| HXHAPPPRF | MEDIUM | XFXMRNROL |
| TAPIRNK | HIGH | HAPIRNK |
| TMPMAST | HIGH | HMLMAST5H |
| TXPBNFIT | HIGH | HXPBNFIT |
| TXPNSTN | HIGH | HXPNSTN |
| ****HXPXML | MEDIUM | HABADTE |
| PRINTER | MEDIUM | HABADTE |