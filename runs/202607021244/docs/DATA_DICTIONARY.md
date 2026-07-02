# Data Dictionary – HABADTE AS400 Application (Run 202607021244)

This data dictionary summarises the physical and logical file structures inferred from the aggregated reverse-engineering context. It focuses on key identifiers, PHI-sensitive fields, and logical views critical to patient management and benefits processing.

## 1. Physical Files (PF)

### 1.1 HAPTRFR
- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **PHI-sensitive:** Yes (AFACCT, AFMRNO)

**Field Catalog**

| Field   | Type   | Length | Dec | Column Heading | PHI Category   |
|---------|--------|--------|-----|----------------|----------------|
| AFLVL6  | NUMERIC| ?      | ?   | Level 6        | None           |
| AFACCT  | CHAR   | ?      | -   | Account        | AccountNumber  |
| AFTRDT  | NUMERIC| ?      | ?   | Transfer Date  | None           |
| AFTRTM  | NUMERIC| ?      | ?   | Transfer Time  | None           |
| AFTYPE  | CHAR   | ?      | -   | Transfer Type  | None           |
| AFMRNO  | CHAR   | ?      | -   | MRN            | MRN            |
| ...     | ...    | ...    | ... | ...            | ...            |

(“...” denotes additional non-PHI fields inferred from the total field count of 28.)

### 1.2 HXPDICT
- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** (none declared)
- **PHI-sensitive:** Yes (multiple identifiers)

**Field Catalog (PHI-focused)**

| Field   | Type   | Length | Dec | Column Heading | PHI Category   |
|---------|--------|--------|-----|----------------|----------------|
| CCMRNO  | CHAR   | ?      | -   | MRN            | MRN            |
| XFBTEL  | CHAR   | ?      | -   | Phone          | PhoneNumber    |
| XCNAME  | CHAR   | ?      | -   | Name           | PatientName    |
| HXRMNO  | CHAR   | ?      | -   | Room           | RoomNumber     |
| XFRMNO  | CHAR   | ?      | -   | Room           | RoomNumber     |
| HVACCT  | CHAR   | ?      | -   | Account        | AccountNumber  |
| IMGMRN  | CHAR   | ?      | -   | MRN            | MRN            |
| HXGMRN  | CHAR   | ?      | -   | MRN            | MRN            |
| IHMRNO  | CHAR   | ?      | -   | MRN            | MRN            |
| IHACCT  | CHAR   | ?      | -   | Account        | AccountNumber  |
| WBDATE  | DATE   | ?      | -   | Birth Date     | DateOfBirth    |
| XMDMRN  | CHAR   | ?      | -   | MRN            | MRN            |
| ENNAME  | CHAR   | ?      | -   | Name           | PatientName    |
| ...     | ...    | ...    | ... | ...            | ...            |

HXPDICT contains a large set of additional technical and reference attributes (total fields: 2705) that are not individually listed here; only PHI-tagged fields are highlighted.

### 1.3 HXPLVL1
- **Record format:** HXFLVL1
- **Unique key:** Yes
- **Key fields:** HX1NUM
- **Total fields:** 36
- **PHI-sensitive:** No

All fields appear to be configuration or code values. No PHI categories are flagged.

### 1.4 HXPLVL2
- **Record format:** HXFLVL2
- **Unique key:** Yes
- **Key fields:** HX2NUM
- **Total fields:** 39
- **PHI-sensitive:** No

### 1.5 HXPLVL3
- **Record format:** HXFLVL3
- **Unique key:** Yes
- **Key fields:** HX3NUM
- **Total fields:** 39
- **PHI-sensitive:** No

### 1.6 HXPLVL4
- **Record format:** HXFLVL4
- **Unique key:** Yes
- **Key fields:** HX4NUM
- **Total fields:** 39
- **PHI-sensitive:** No

### 1.7 HXPLVL5
- **Record format:** HXFLVL5
- **Unique key:** Yes
- **Key fields:** HX5NUM
- **Total fields:** 42
- **PHI-sensitive:** No

### 1.8 HXPLVL6
- **Record format:** HXFLVL6
- **Unique key:** Yes
- **Key fields:** HX6NUM
- **Total fields:** 155
- **PHI-sensitive:** No

HXPLVL1–6 together model a multi-tier level hierarchy, used by XFXLDSC for derivation and description logic. No direct PHI presence is recorded.

### 1.9 HXPTABLD
- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI-sensitive:** No

This compact table drives code/description mappings and is the base PFILE for several logical views.

### 1.10 HXPXMLD
- **Record format:** HXFXMLD
- **Unique key:** Yes
- **Key fields:** XMDUSR, XMDSEQ, XMDSQ2
- **Total fields:** 4
- **PHI-sensitive:** No

### 1.11 HXPXMLR
- **Record format:** HXFXMLR
- **Unique key:** Yes
- **Key fields:** XMRUSR, XMRSEQ, XMRID
- **Total fields:** 4
- **PHI-sensitive:** No

HXPXMLD/R hold XML header/detail control data and serve as input/output for HABADTE and XFXGETID.

### 1.12 OAPIRNK
- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI-sensitive:** Yes (BRKMRN)

**Field Catalog (PHI-focused)**

| Field   | Type   | Length | Dec | Column Heading | PHI Category |
|---------|--------|--------|-----|----------------|-------------|
| BRKLV6  | NUMERIC| ?      | ?   | Level 6        | None        |
| BRKACC  | CHAR   | ?      | -   | Account        | None        |
| BRKSEQ  | NUMERIC| ?      | ?   | Sequence       | None        |
| BRKMRN  | CHAR   | ?      | -   | MRN            | MRN         |
| ...     | ...    | ...    | ... | ...            | ...         |

### 1.13 OMPMAST
- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI-sensitive:** Yes (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN)

**Field Catalog (PHI-focused)**

| Field   | Type   | Length | Dec | Column Heading | PHI Category   |
|---------|--------|--------|-----|----------------|----------------|
| MMPLV6  | NUMERIC| ?      | ?   | Plan Level     | None           |
| MMACCT  | CHAR   | ?      | -   | Account        | AccountNumber  |
| MMMRNO  | CHAR   | ?      | -   | MRN            | MRN            |
| MMNAME  | CHAR   | ?      | -   | Name           | PatientName    |
| MMPSSN  | CHAR   | ?      | -   | SSN            | SSN            |
| MMMMRN  | CHAR   | ?      | -   | MRN            | MRN            |
| ...     | ...    | ...    | ... | ...            | ...            |

### 1.14 OXPBNFIT
- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI-sensitive:** Yes (XFBTEL)

**Field Catalog (PHI-focused)**

| Field   | Type   | Length | Dec | Column Heading | PHI Category |
|---------|--------|--------|-----|----------------|-------------|
| XFBUBN  | CHAR   | ?      | -   | Benefit Code   | None        |
| XFBPLN  | CHAR   | ?      | -   | Plan Code      | None        |
| XFBTEL  | CHAR   | ?      | -   | Phone          | PhoneNumber |
| ...     | ...    | ...    | ... | ...            | ...         |

### 1.15 OXPNSTN
- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI-sensitive:** No

## 2. Logical Files (LF)

Each logical file is a view against one PF (PFILE), with key fields and optional select/omit rules.

### 2.1 HAPIRNK
- **PFILE:** TAPIRNK (missing PF)
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** None identified.

HAPIRNK provides indexed access to inquiry records by level, account, and sequence, used by benefit or inquiry routines. PHI category is inherited from BRKMRN in its base PF.

### 2.2 HMLMAST5H
- **PFILE:** TMPMAST (missing PF)
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit:** None identified.

This logical file offers a time- and station-based view over the patient master file, supporting retrospective or audit-style queries.

### 2.3 HXLTABLD
- **PFILE:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDMAP
- **Select/Omit:** None identified.

HXLTABLD exposes table records keyed by data code and map code, enabling mapping logic in XFXTABL and related programs.

### 2.4 HXLTABLP
- **PFILE:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDLDS
- **Select/Omit:** None identified.

HXLTABLP focuses on load-description keys, supporting alternative views of the same table content.

### 2.5 HXLTABLS
- **PFILE:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDSDS
- **Select/Omit:** None identified.

HXLTABLS exposes subset-description keys, creating yet another access path into HXPTABLD records.

### 2.6 HXPBNFIT
- **PFILE:** TXPBNFIT (missing PF)
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/Omit:** None identified.

Logical view into benefits/plan records; PHI categories follow XFBTEL from the base PF.

### 2.7 HXPNSTN
- **PFILE:** TXPNSTN (missing PF)
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/Omit:** None identified.

Provides a keyed view over station- or status-related data, supporting operational workflows.

## 3. PHI Sensitivity Summary

The following PFs contain PHI-tagged fields:

- **HAPTRFR:** AccountNumber, MRN
- **HXPDICT:** MRN, PhoneNumber, PatientName, RoomNumber, AccountNumber, DateOfBirth
- **OAPIRNK:** MRN
- **OMPMAST:** MRN, AccountNumber, PatientName, SSN
- **OXPBNFIT:** PhoneNumber

Logical files inheriting PHI from their PFILE include:

- **HAPIRNK** (from TAPIRNK/ OAPIRNK-like structure)
- **HMLMAST5H** (from TMPMAST/ OMPMAST-like structure)
- **HXPBNFIT** (from TXPBNFIT/ OXPBNFIT)

All other files in the schema are considered non-PHI from the perspective of this codebase, though downstream integrations may still introduce sensitive contexts.

## 4. Notes for Modernisation

- **Key structures should be preserved** as composite primary keys or unique indexes in the target platform, especially for HAPTRFR, OMPMAST, OAPIRNK, OXPBNFIT, and OXPNSTN.
- **PHI fields must be explicitly classified** in the target schema, mapped to strongly typed, masked, and access-controlled columns.
- **Logical file semantics** (alternative keys, domain-specific views) should be reimplemented via views or query endpoints rather than replicated DDS LFs.
- **Missing PFILEs (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN)** must be located or reconstructed from production environments to ensure full lineage coverage before any cutover.
