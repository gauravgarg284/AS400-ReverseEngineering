# AS400 Code Analysis Report

## 1. Component Inventory

### 1.1 Overview

This report summarizes the harvested AS400 codebase consisting of 37 source members and approximately 9,153 lines of code. The portfolio spans classic RPGLE business logic, SQLRPGLE programs, and associated control artifacts such as display/files and support utilities. The focus of this analysis is to provide a structural inventory, highlight gaps in the collected sources, and identify dependency hotspots and orphaned components that carry architectural risk.

The aggregated pipeline statistics show an estimated harvesting completeness of 82.2%. This indicates that most core components are present, but a small number of referenced files and copybooks remain missing and must be treated as potential integration points with other subsystems.

### 1.2 Member Inventory

The table below summarizes the programs for which interpretations and rules were extracted. These entries represent a subset of the full 37‑member portfolio but illustrate the primary domains and technologies in scope.

| Program    | Type     | Domain              |
|-----------|----------|---------------------|
| XFXCNTR   | RPGLE    | DATA_MAINTENANCE    |
| XFXCYMD   | RPGLE    | DATA_MAINTENANCE    |
| XFXLDSC   | RPGLE    | DATA_MAINTENANCE    |
| XFXTABL   | RPGLE    | DATA_MAINTENANCE    |
| HABADTE   | RPGLE    | PATIENT_MANAGEMENT  |
| HXXAPPPRF | SQLRPGLE | PATIENT_MANAGEMENT* |

All of the interpreted programs above are implemented in modern ILE RPG (RPGLE or SQLRPGLE) and operate primarily in the DATA_MAINTENANCE and PATIENT_MANAGEMENT domains.

### 1.3 Type Distribution

Based on the aggregated context, the following high‑level types are present:

- RPGLE programs handling core data validation (e.g., XFXCYMD) and code table maintenance (e.g., XFXTABL).
- SQLRPGLE program HXXAPPPRF providing application‑profile logic with embedded SQL access.
- Physical and logical files underpinning PATIENT_MANAGEMENT data, including PHI‑bearing tables such as OMPMAST and HAPTRFR.

Although a full type histogram is not available in the summary, the portfolio is clearly RPG‑centric with a small number of SQLRPGLE utilities and multiple PF/LF definitions captured in the separate data dictionary.

## 2. Missing Components

The gap analysis identifies eight referenced artifacts that were not present in the harvested ZIP. These represent either missing copybooks, external programs, or physical/logical file definitions.

| Name        | Type      | Impact | Referenced By |
|------------|-----------|--------|---------------|
| CXXXMLC     | COPYBOOK  | HIGH   | HABADTE       |
| HXHAPPPRF   | PROGRAM   | MEDIUM | XFXMRNROL     |
| TAPIRNK     | FILE      | HIGH   | HAPIRNK       |
| TMPMAST     | FILE      | HIGH   | HMLMAST5H     |
| TXPBNFIT    | FILE      | HIGH   | HXPBNFIT      |
| TXPNSTN     | FILE      | HIGH   | HXPNSTN       |
| ****HXPXML  | FILE      | MEDIUM | HABADTE       |
| PRINTER     | FILE      | MEDIUM | HABADTE       |

### 2.1 High‑Impact Gaps

High‑impact gaps (CXXXMLC, TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) are likely critical to the correct execution of billing, benefits, and patient master workflows. Their absence means that the reverse‑engineered dependency graph will under‑represent certain downstream effects, particularly for batch processing and integration jobs.

### 2.2 Medium‑Impact Gaps

Medium‑impact entries (HXHAPPPRF, ****HXPXML, PRINTER) appear to be auxiliary programs or files, such as printer configuration or XML export targets. While not blocking core logic comprehension, they represent integration points to external systems or report channels and should be located in subsequent exports.

## 3. Duplicate and Reused Components

### 3.1 Reused Programs

Although the detailed dependency edge list is not fully enumerated in this summary, several patterns can be inferred:

- HABADTE references multiple external artifacts (CXXXMLC, ****HXPXML, PRINTER), indicating it serves as a central orchestration program for patient transfer or admission flows.
- XFXMRNROL calls HXHAPPPRF, suggesting that MRN (medical record number) roll logic and application profile logic are loosely coupled components.

In a complete dependency list, reused programs would appear as targets in multiple CALL edges. HABADTE and the XFX* utilities are strong candidates for such reuse, and they should be treated as shared services in any modernization effort.

### 3.2 Logical Views over Shared Physical Files

From the PHI exposure list and data dictionary context (OXPBNFIT, OMPMAST, HAPTRFR, HXPDICT, OAPIRNK), it is clear that several logical files and programs converge on common PFs representing patient, benefits, and ranking data. This implies multiple logical views (LFs) over shared PFs, often used to support different report formats or access patterns.

## 4. Dependency Analysis

### 4.1 Call Chain Characteristics

The dependency graph indicates a small but interconnected set of programs:

- HABADTE operates in the PATIENT_MANAGEMENT domain and interacts with printer and XML‑related files, pointing to both user‑facing and batch export responsibilities.
- The XFX* family (XFXCNTR, XFXCYMD, XFXLDSC, XFXTABL, XFXMRNROL, XFXGETID, XFXLEAP) forms a shared utility layer for counter management, date validation, leap‑year calculations, table lookups, and MRN roll operations.
- HXXAPPPRF, though orphaned in the current graph, likely functions as an application profile manager with SQL‑based persistence.

In modern terms, the graph resembles a set of utility micro‑services (XFX*) called from patient‑domain orchestrators (HABADTE and peers), with data persisted in PFs flagged for PHI content.

### 4.2 Hotspots

The dependency hotspots (programs with high fan‑in and fan‑out and significant file operations) are not explicitly listed by score in the summary, but we can infer candidates:

- HABADTE, due to its references to multiple files and copybooks, is a probable hotspot in the patient transfer/bed assignment flows.
- XFXCYMD and XFXTABL, which implement date validation and table lookups used across data‑maintenance routines, are likely shared utilities with moderate fan‑in.

These hotspots are important modernization anchors: refactoring them into well‑defined services or modules will yield high benefit and reduce coupling across the system.

## 5. Summary of Key Findings

### 5.1 Critical Structural Issues

Based on the high/medium gaps and orphan program list, the following structural risks are notable:

- **Missing copybook CXXXMLC**: This prevents a full reconstruction of the HABADTE program interface and may hide additional validation or interface logic.
- **Missing PHI‑adjacent files (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN)**: These files are referenced by programs dealing with patient ranking, master records, benefits, and notifications. Their absence limits understanding of critical financial and clinical workflows.
- **Multiple PHI‑bearing PFs**: Files such as OMPMAST, HAPTRFR, and OXPBNFIT are flagged for PHI exposure. Any modernization or data‑replication activity must treat them as highly sensitive and ensure access controls replicate AS400 security semantics.

### 5.2 Orphan Programs

The following programs appear with no callers in the harvested dependency graph:

- HXXAPPPRF (SQLRPGLE)
- XFXCNTR
- XFXCYMD
- XFXGETID
- XFXLDSC
- XFXLEAP
- XFXMRNROL
- XFXTABL
- HABADTE

In practice, these programs are likely invoked by external job control language (CLP/CLLE), menus, or scheduler entries not captured in this export. They should not be considered dead code. Instead, they represent top‑level entry points into the application and must be cataloged as such in any target‑state architecture.

### 5.3 Architectural Observations

- The codebase is organized into a clear separation between **utility/date/counter services (XFX*)** and **patient‑management orchestration (HABADTE and related components)**.
- Several key programs exhibit characteristics of shared services, even though they are written as traditional RPG modules. This will ease decomposition into service‑oriented or microservice architectures.
- The harvesting completeness (82.2%) is sufficient for meaningful structural and rule analysis, but the identified gaps must be addressed before any final cut‑over or high‑risk refactoring.

Overall, the AS400 codebase exhibits a modular but tightly coupled structure, with a small number of high‑value programs mediating access to PHI‑heavy data structures. These findings should be used to prioritize remediation, security hardening, and modernization sequencing.