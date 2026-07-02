# Business Processing Rules & Functional Specification

## HABADTE Transfer Activity Detail Extract (HABADTE)

### For Java / Spring Boot / SQL Server Migration

**Source Program:** HABADTE.RPGLE

**Document Purpose:** Complete business rules sufficient to re-implement in Java + Spring Boot + SQL Server

(1) Business Purpose

The HABADTE program produces a detailed extract of patient transfer activity for inpatient stays. It is used by patient accounting and clinical operations teams to understand movement between service locations and to feed downstream reporting and billing processes.

> Core business question: "For a given patient account and level, what inpatient transfer events should be considered valid and included in the activity extract?"

The report produces a row per qualifying transfer, along with summary totals of valid transfers, skipped/voided records, and counts by inpatient/outpatient status.

(2) Inputs (API Request Parameters)

