# Module Specifications - HABADTE Project

## Overview

This document describes each program and database object discovered in the HABADTE AS400 application landscape, with emphasis on migration to Java / Spring Boot / SQL Server. Metrics and dependencies are derived from the reverse-engineered source metadata.

Global notes:
- Cyclomatic complexity and risk bands are sourced from the aggregated summary metrics.
- Migration difficulty is inferred from size (lines of code), dependency fan-out, and complexity band.
- Tech debt is only available as a global summary for this run; no per-program findings were provided.

### Global Tech Debt Summary

No detailed tech debt breakdown was provided in the aggregated summary for this run. All modules should be reviewed during modernization for:
- Error handling consistency
- Data validation and null/zero checks
- Encapsulation of PHI-handling logic into dedicated services

---

## Programs and Files

Below, every member from the harvested source inventory is documented. For database files (PF/LF), program-centric sections are minimal, focusing on structure and usage.

