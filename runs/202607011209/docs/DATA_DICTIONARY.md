# Data Dictionary

## 1. Physical Files (PF)

### 1.1 HAPTRFR
- **Record format**: HAFTRFR
- **Unique key**: Yes
- **Key fields**: AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields**: 28
- **PHI-sensitive**: Yes (AFACCT, AFMRNO)

**Fields**

| Field   | Type        | Length | Dec | Column Heading | PHI Category   |
|---------|------------|--------|-----|----------------|----------------|
| AFACCT  | (unknown)   | (n/a)  | -   | Account        | AccountNumber  |
| AFMRNO  | (unknown)   | (n/a)  | -   | MRN            | MRN            |
| AFLVL6  | (unknown)   | (n/a)  | -   | Level 6        | None           |
| AFTRDT  | (unknown)   | (n/a)  | -   | Transfer Date  | None           |
| AFTRTM  | (unknown)   | (n/a)  | -   | Transfer Time  | None           |
| AFTYPE  | (unknown)   | (n/a)  | -   | Type           | None           |
| ...     | ...         | ...    | ... | ...            | ...            |

(Additional non-PHI fields are present but not individually enumerated in the aggregated schema.)

### 1.2 HXPDICT
- **Record format**: HXFDICT
- **Unique key**: No
- **Key fields**: (none recorded)
- **Total fields**: 2705
- **PHI-sensitive**: Yes (multiple PHI fields)

**Fields (PHI subset)**

| Field   | Type      | Length | Dec | Column Heading | PHI Category   |
|---------|----------|--------|-----|----------------|----------------|
| CCMRNO  | (unknown)| (n/a)  | -   | MRN            | MRN            |
| XFBTEL  | (unknown)| (n/a)  | -   | Phone          | PhoneNumber    |
| XCNAME  | (unknown)| (n/a)  | -   | Name           | PatientName    |
| HXRMNO  | (unknown)| (n/a)  | -   | Room Number    | RoomNumber     |
| XFRMNO  | (unknown)| (n/a)  | -   | Room Number    | RoomNumber     |
| HVACCT  | (unknown)| (n/a)  | -   | Account        | AccountNumber  |
| IMGMRN  | (unknown)| (n/a)  | -   | MRN            | MRN            |
| HXGMRN  | (unknown)| (n/a)  | -   | MRN            | MRN            |
| IHMRNO  | (unknown)| (n/a)  | -   | MRN            | MRN            |
| IHACCT  | (unknown)| (n/a)  | -   | Account        | AccountNumber  |
| WBDATE  | (unknown)| (n/a)  | -   | Birth Date     | DateOfBirth    |
| XMDMRN  | (unknown)| (n/a)  | -   | MRN            | MRN            |
| ENNAME  | (unknown)| (n/a)  | -   | Name           | PatientName    |

All remaining fields are non-PHI dictionary attributes; they should be classified and documented in a full export, but the aggregated schema focuses on PHI hotspots.

### 1.3 HXPLVL1
- **Record format**: HXFLVL1
- **Unique key**: Yes
- **Key fields**: HX1NUM
- **Total fields**: 36
- **PHI-sensitive**: No

This file holds level-1 configuration. No fields are flagged as PHI.

### 1.4 HXPLVL2
- **Record format**: HXFLVL2
- **Unique key**: Yes
- **Key fields**: HX2NUM
- **Total fields**: 39
- **PHI-sensitive**: No

### 1.5 HXPLVL3
- **Record format**: HXFLVL3
- **Unique key**: Yes
- **Key fields**: HX3NUM
- **Total fields**: 39
- **PHI-sensitive**: No

### 1.6 HXPLVL4
- **Record format**: HXFLVL4
- **Unique key**: Yes
- **Key fields**: HX4NUM
- **Total fields**: 39
- **PHI-sensitive**: No

### 1.7 HXPLVL5
- **Record format**: HXFLVL5
- **Unique key**: Yes
- **Key fields**: HX5NUM
- **Total fields**: 42
- **PHI-sensitive**: No

### 1.8 HXPLVL6
- **Record format**: HXFLVL6
- **Unique key**: Yes
- **Key fields**: HX6NUM
- **Total fields**: 155
- **PHI-sensitive**: No

HXPLVL1–HXPLVL6 together form a hierarchy of level-based configuration or benefits structures. None of these levels contain PHI-tagged fields.

### 1.9 HXPTABLD
- **Record format**: XFFTABLD
- **Unique key**: No
- **Key fields**: XFDTCD, XFDECD
- **Total fields**: 7
- **PHI-sensitive**: No

HXPTABLD serves as a compact dictionary table keyed by data code and detail code. It is used via logical files for different key sequences.

### 1.10 HXPXMLD
- **Record format**: HXFXMLD
- **Unique key**: Yes
- **Key fields**: XMDUSR, XMDSEQ, XMDSQ2
- **Total fields**: 4
- **PHI-sensitive**: No

This file stores XML detail records keyed by user and sequence. No fields are flagged as PHI.

### 1.11 HXPXMLR
- **Record format**: HXFXMLR
- **Unique key**: Yes
- **Key fields**: XMRUSR, XMRSEQ, XMRID
- **Total fields**: 4
- **PHI-sensitive**: No

HXPXMLR stores XML routing or header records keyed by user and identifier.

### 1.12 OAPIRNK
- **Record format**: HBFIRNK
- **Unique key**: Yes
- **Key fields**: BRKLV6, BRKACC, BRKSEQ
- **Total fields**: 33
- **PHI-sensitive**: Yes (BRKMRN)

**Fields (PHI subset)**

| Field   | Type      | Length | Dec | Column Heading | PHI Category |
|---------|----------|--------|-----|----------------|--------------|
| BRKMRN  | (unknown)| (n/a)  | -   | MRN            | MRN          |

Other fields describe rank, account, and level attributes.

### 1.13 OMPMAST
- **Record format**: HMFMAST
- **Unique key**: Yes
- **Key fields**: MMPLV6, MMACCT
- **Total fields**: 149
- **PHI-sensitive**: Yes (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN)

**Fields (PHI subset)**

| Field   | Type      | Length | Dec | Column Heading | PHI Category   |
|---------|----------|--------|-----|----------------|----------------|
| MMMRNO  | (unknown)| (n/a)  | -   | MRN            | MRN            |
| MMACCT  | (unknown)| (n/a)  | -   | Account        | AccountNumber  |
| MMNAME  | (unknown)| (n/a)  | -   | Name           | PatientName    |
| MMPSSN  | (unknown)| (n/a)  | -   | SSN            | SSN            |
| MMMMRN  | (unknown)| (n/a)  | -   | MRN            | MRN            |

Other non-PHI fields capture demographic, plan, and status attributes.

### 1.14 OXPBNFIT
- **Record format**: XFFBNFIT
- **Unique key**: Yes
- **Key fields**: XFBUBN, XFBPLN
- **Total fields**: 34
- **PHI-sensitive**: Yes (XFBTEL)

**Fields (PHI subset)**

| Field   | Type      | Length | Dec | Column Heading | PHI Category   |
|---------|----------|--------|-----|----------------|----------------|
| XFBTEL  | (unknown)| (n/a)  | -   | Phone          | PhoneNumber    |

The remainder of fields describe benefit attributes.

### 1.15 OXPNSTN
- **Record format**: XFFNSTN
- **Unique key**: Yes
- **Key fields**: XFNLV6, XFNSST
- **Total fields**: 23
- **PHI-sensitive**: No

OXPNSTN defines institution or plan station records for level 6.

## 2. Logical Files (LF)

Logical files present alternate views over PFs with specific key sequences.

### 2.1 HAPIRNK
- **Based on PF**: TAPIRNK
- **Record format**: HBFIRNK
- **Key fields**: BRKLV6, BRKACC, BRKSEQ
- **Select/omit**: None

HAPIRNK provides a keyed view of rank or break records at level 6, aligned with the structure used by OAPIRNK.

### 2.2 HMLMAST5H
- **Based on PF**: TMPMAST
- **Record format**: HMFMAST
- **Key fields**: MMPNST, MMADDT, MMADTM
- **Select/omit**: None

HMLMAST5H offers a view of member master data (TMPMAST) keyed by posting station and admission date/time, supporting queries by admission episode.

### 2.3 HXLTABLD
- **Based on PF**: HXPTABLD
- **Record format**: XFFTABLD
- **Key fields**: XFDTCD, XFDMAP
- **Select/omit**: None

HXLTABLD exposes dictionary entries keyed by data code and map code.

### 2.4 HXLTABLP
- **Based on PF**: HXPTABLD
- **Record format**: XFFTABLD
- **Key fields**: XFDTCD, XFDLDS
- **Select/omit**: None

HXLTABLP exposes entries keyed by data code and load sequence.

### 2.5 HXLTABLS
- **Based on PF**: HXPTABLD
- **Record format**: XFFTABLD
- **Key fields**: XFDTCD, XFDSDS
- **Select/omit**: None

HXLTABLS exposes entries keyed by data code and status sequence.

### 2.6 HXPBNFIT
- **Based on PF**: TXPBNFIT
- **Record format**: XFFBNFIT
- **Key fields**: XFBUBN, XFBPLN
- **Select/omit**: None

HXPBNFIT provides a logical view over benefit records focusing on benefit number and plan.

### 2.7 HXPNSTN
- **Based on PF**: TXPNSTN
- **Record format**: XFFNSTN
- **Key fields**: XFNLV6, XFNSST
- **Select/omit**: None

HXPNSTN provides a logical view over institution or station records keyed by level and station.

## 3. PHI Overview

Across the schema, PHI is concentrated in the following files and fields:

- **HAPTRFR**: AFACCT (AccountNumber), AFMRNO (MRN).
- **HXPDICT**: CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME.
- **OAPIRNK**: BRKMRN (MRN).
- **OMPMAST**: MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN.
- **OXPBNFIT**: XFBTEL (PhoneNumber).

These fields should be treated as sensitive in any downstream design, with appropriate encryption, masking, and access controls.

## 4. Summary

The data dictionary reveals a file-centric data model typical of legacy AS400 applications. Core domains include:

- Transfers and rank records (HAPTRFR, OAPIRNK, HAPIRNK).
- Member or patient master data (OMPMAST, HMLMAST5H, TMPMAST).
- Multi-level configuration and benefits structures (HXPLVL1–HXPLVL6, HXPTABLD, HXLTAB*).
- Benefits and institution station data (OXPBNFIT, HXPBNFIT, OXPNSTN, HXPNSTN).
- Large dictionary and routing structures with PHI (HXPDICT).
- XML integration records (HXPXMLD, HXPXMLR).

This dictionary provides the structural basis for mapping DDS-based files to relational tables and views in a modern platform, with explicit identification of PHI fields to guide security and compliance design.