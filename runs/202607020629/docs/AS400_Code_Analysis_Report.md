# AS400 Code Analysis Report – HABADTE Project

## 1. Component Inventory

This section summarizes the full inventory of source members reverse‑engineered from the HABADTE AS400 application. Counts and details are derived from the consolidated `source_manifest.json` in the aggregated context.

### 1.1 Member Type Totals

| Type      | Count |
|-----------|-------|
| DDS_PF    | 15    |
| DDS_LF    | 7     |
| SQLRPGLE  | 2     |
| RPGLE     | 13    |
| **Total** | **37** |

The portfolio is data‑centric: over half of the members are DDS physical and logical file definitions, with a relatively small number of procedural RPG programs, aligning with a batch‑style patient accounting and benefits processing workload.

### 1.2 Member Inventory

| Member      | Type     | Subsystem | Lines |
|-------------|----------|-----------|-------|
| HAPIRNK     | DDS_LF   | HAP       | 13    |
| HAPTRFR     | DDS_PF   | HAP       | 72    |
| HMLMAST5H   | DDS_LF   | UNGROUPED | 12    |
| HXLTABLD    | DDS_LF   | HXLT      | 11    |
| HXLTABLP    | DDS_LF   | HXLT      | 12    |
| HXLTABLS    | DDS_LF   | HXLT      | 12    |
| HXPBNFIT    | DDS_LF   | HXP       | 12    |
| HXPDICT     | DDS_PF   | HXP       | 6130  |
| HXPLVL1     | DDS_PF   | HXPL      | 49    |
| HXPLVL2     | DDS_PF   | HXPL      | 52    |
| HXPLVL3     | DDS_PF   | HXPL      | 52    |
| HXPLVL4     | DDS_PF   | HXPL      | 52    |
| HXPLVL5     | DDS_PF   | HXPL      | 55    |
| HXPLVL6     | DDS_PF   | HXPL      | 321   |
| HXPNSTN     | DDS_LF   | HXP       | 12    |
| HXPTABLD    | DDS_PF   | HXP       | 19    |
| HXPXMLD     | DDS_PF   | HXPX      | 19    |
| HXPXMLR     | DDS_PF   | HXPX      | 19    |
| OAPIRNK     | DDS_PF   | UNGROUPED | 80    |
| OMPMAST     | DDS_PF   | UNGROUPED | 310   |
| OXPBNFIT    | DDS_PF   | OXP       | 48    |
| OXPNSTN     | DDS_PF   | OXP       | 65    |
| HXXAPPPRF   | SQLRPGLE | HXXA      | 123   |
| XFXCNTR     | RPGLE    | XFXC      | 49    |
| XFXCYMD     | RPGLE    | XFXC      | 83    |
| XFXGETID    | RPGLE    | XFX       | 61    |
| XFXLDSC     | RPGLE    | XFXL      | 135   |
| XFXLEAP     | RPGLE    | XFXL      | 61    |
| XFXMRNROL   | RPGLE    | XFX       | 65    |
| XFXTABL     | RPGLE    | XFX       | 164   |
| CXXXMLP     | SQLRPGLE | UNGROUPED | 25    |
| HXXAPPPRFP  | RPGLE    | HXXA      | 42    |
| HXXCNTRL    | RPGLE    | HXX       | 8     |
| HXXLDA      | RPGLE    | HXXL      | 53    |
| HXXLEVEL    | RPGLE    | HXXL      | 25    |
| HXXXML      | RPGLE    | HXX       | 11    |
| HABADTE     | RPGLE    | HA        | 821   |

The main patient management driver is `HABADTE`, a large RPG program with 821 lines and high cyclomatic complexity, orchestrating lower‑level utilities (`XFX*` programs) and XML/control copybooks (`HXX*`, `CXXXMLP`).

## 2. Missing Components

The gap analysis identifies eight missing or incomplete components referenced by the code. These represent structural risks for modernization because their behavior must be understood or emulated.

### 2.1 Gap Inventory Grouped by Impact

**HIGH‑Impact Gaps**

| Name    | Type     | Referenced By |
|---------|----------|---------------|
| CXXXMLC | COPYBOOK | HABAD         |
| TAPIRNK | FILE     | HAPIR         |
| TMPMAST | FILE     | HMLMA         |
| TXPBNFIT| FILE     | HXPBN         |
| TXPNSTN | FILE     | HXPNS         |

These gaps are structurally critical:
- `CXXXMLC` is a missing copybook used by `HABADTE` to define XML‑related data structures or constants. Its absence obscures the full request/response payloads.
- `TAPIRNK` and `TMPMAST` are underlying physical files for logical views `HAPIRNK` and `HMLMAST5H`; without them the logical file definitions cannot be exercised, impacting any reporting over patient rankings or master records.
- `TXPBNFIT` and `TXPNSTN` are text‑based or DDS physical files backing benefits (`HXPBNFIT`) and outstanding balances or notices (`HXPNSTN`). Modernization must either locate these definitions or define surrogate schemas.

**MEDIUM‑Impact Gaps**

| Name      | Type   | Referenced By |
|-----------|--------|---------------|
| HXHAPPPRF | PROGRAM| XFXMR         |
| ****HXPXML| FILE   | HABAD         |
| PRINTER   | FILE   | HABAD         |

- `HXHAPPPRF` is a program called from `XFXMRNROL`. Its behavior appears related to application profile or security; the missing logic is mitigated by the low complexity of the caller, but exact authorization rules are unknown.
- `****HXPXML` and `PRINTER` are device/file references in `HABADTE` likely tied to XML output and print spooling. Their absence affects external interface modelling more than internal business rules.

**LOW‑Impact Gaps**

None identified in this codebase. All reported gaps have at least medium impact.

## 3. Duplicate and Reused Components

Re‑use patterns can be inferred from dependency edges. In the call graph, targets appearing multiple times for CALL edges represent shared routines, while logical files sharing the same physical file indicate common data abstractions.

### 3.1 Reused Programs

From the `dep_edges` call relations:

- `XFXCNTR` is called three times by `HABADTE`. It encapsulates counter or threshold logic (rules BR‑001 and BR‑002) and is reused at multiple decision points.
- `XFXMRNROL` is both directly called by `HABADTE` and delegates to `HXHAPPPRF` and `HXXAPPPRF`. Although not heavily reused by multiple callers, it is a central roll‑up routine for MRN (Medical Record Number) related processing.
- `XFXCYMD` is called by `HABADTE` and itself calls `XFXLEAP` to validate calendar dates. This pairing is reused wherever date validation is needed.
- `XFXGETID` is called from `HABADTE` and performs file reads against `HXFXMLR`. It likely centralizes ID lookup for XML messages.

These reused utilities should be treated as candidate service endpoints in a refactored architecture, with clear contracts around date validation, counters, identifiers, and table lookups.

### 3.2 Logical Files Over Shared Physical Files

Logical file relationships show multiple LFs pointing to the same PF:

- `HXLTABLD`, `HXLTABLP`, and `HXLTABLS` all reference physical file `HXPTABLD` and share record format `XFFTABLD` with varying key fields (`XFDMAP`, `XFDLDS`, `XFDSDS`). This indicates a central table dictionary used in different sort or filter orders.
- `HXPBNFIT` references `TXPBNFIT` with the `XFFBNFIT` format, and `HXPNSTN` references `TXPNSTN` with the `XFFNSTN` format. These logical files project benefit and notification information over base text/physical structures.

In modernization, these shared PF/LF structures should be modelled as core tables with multiple indexes or views, preserving the different access patterns encoded by DDS.

## 4. Dependency Analysis

### 4.1 Call Chain Overview

The call edges from `dep_edges` describe the procedural architecture:

- `HABADTE` is the top‑level orchestrator with fan‑out to:
  - `XFXMRNROL` (CALL)
  - `XFXCNTR` (CALLED three times)
  - `XFXLDSC` (CALL)
  - `XFXCYMD` (CALL)
  - `XFXGETID` (CALL)
  - `XFXTABL` (CALL)
  - Copybook and configuration members `HXXLDA`, `HXXLEVEL`, `HXXXML`, `CXXXMLP`, and `CXXXMLC` via COPY edges.

- `XFXCYMD` delegates to `XFXLEAP` for leap year handling.
- `XFXMRNROL` calls `HXHAPPPRF` and `HXXAPPPRF`.
- `HXXAPPPRF` uses COPY edges to `HXXCNTRL` and `HXXAPPPRFP`, suggesting a layered definition of application profile and control structures.

### 4.2 File Access Patterns and Hotspots

File access edges reveal which programs perform IO:

- `XFXGETID` reads `HXFXMLR`.
- `XFXLDSC` reads all level files `HXFLVL1`–`HXFLVL6`, tying business logic to hierarchical level configurations.
- `XFXTABL` reads table files `XFFTABLD`–`XFFTABL4`, representing generic table look‑ups.
- `HABADTE` reads `HAPTRFR` and `XFFNSTN`, and reads/updates/writes XML header/detail structures (`HXFXMLH`, `HXFXMLD`).

Hotspot metrics from `dep_hotspots` confirm the centrality of these components:

| Program    | Score | Fan‑In | Fan‑Out | File Ops |
|-----------|-------|--------|---------|----------|
| HABADTE   | 38    | 0      | 13      | 6        |
| XFXLDSC   | 15    | 1      | 0       | 6        |
| XFXTABL   | 11    | 1      | 0       | 4        |
| XFXCNTR   | 9     | 3      | 0       | 0        |
| XFXGETID  | 7     | 1      | 1       | 1        |

`HABADTE` is clearly the architectural hub, while `XFXLDSC` and `XFXTABL` are data‑access utilities. `XFXCNTR` and `XFXGETID` are compact but structurally important because of their fan‑in.

## 5. Summary of Key Findings

### 5.1 High/Medium Impact Structural Issues

Using `high_medium_gaps` and orphan program data:

| Issue Type        | Component  | Details |
|-------------------|-----------|---------|
| Missing COPYBOOK  | CXXXMLC   | High impact; used by HABADTE for XML structures. Required to fully understand outbound/inbound XML payloads. |
| Missing Program   | HXHAPPPRF | Medium impact; called by XFXMRNROL, likely application profile logic. |
| Missing PF for LF | TAPIRNK   | High impact; underlying file for HAPIRNK logical view, related to patient ranking. |
| Missing PF for LF | TMPMAST   | High impact; underlying file for HMLMAST5H, likely patient master history. |

Orphan programs:

- `HXXAPPPRF` (SQLRPGLE)
- `XFXCNTR`
- `XFXCYMD`
- `XFXGETID`
- `XFXLDSC`

These orphaned components have no callers in the extracted set, which may reflect partial export or specialized batch entry points. For modernization, they should be assessed individually to determine whether they are legacy utilities to be retired or standalone services that require API endpoints.

### 5.2 Architectural Observations

- The system is tightly coupled around `HABADTE`, with multiple direct calls and file operations. Refactoring should aim to decompose this program into smaller services aligned with date validation, MRN roll‑up, table lookups, and XML generation.
- Data access is concentrated through DDS level and table files, indicating that many business rules are encoded via table‑driven configurations rather than procedural code alone.
- PHI‑bearing files such as `HAPTRFR`, `HXPDICT`, `OAPIRNK`, `OMPMAST`, and `OXPBNFIT` underpin patient transaction, dictionary, ranking, and benefits processing. Strong data‑governance controls should accompany any migration of these structures.
- The presence of missing files and copybooks means the extracted model is approximately 82% complete. Any production‑grade modernization must either recover the missing artifacts or explicitly document assumed behavior based on SME interviews.

Overall, the HABADTE codebase exhibits a classic AS400 pattern: a large central batch program coordinating a set of reusable utilities over a rich DDS schema. The structural documentation produced here provides a foundation for low‑level design, data conversion planning, and incremental replacement of high‑risk hotspots.
