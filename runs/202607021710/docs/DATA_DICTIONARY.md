# Data Dictionary – HABADTE AS400 Suite

Run ID: 202607021710  
System: HABADTE AS400 admission and data maintenance suite

This data dictionary summarizes the physical and logical file structures extracted from the HABADTE codebase. It focuses on record formats, key structures, and PHI-sensitive fields.

---

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Transfer File

- **Record format**: `HAFTRFR`
- **Unique key**: Yes
- **Key fields**:
  - `AFLVL6` – Level 6 code for transfer classification.
  - `AFACCT` – Account number (PHI: AccountNumber).
  - `AFTRDT` – Transfer date.
  - `AFTRTM` – Transfer time.
  - `AFTYPE` – Transfer type code.
- **Total fields**: 28
- **PHI flag**: Yes
- **PHI fields and categories**:
  - `AFACCT` – AccountNumber
  - `AFMRNO` – MRN

**Field table (selected)**

| Field   | Type      | Length | Dec | Column Heading        | PHI Category   |
|---------|-----------|--------|-----|-----------------------|----------------|
| AFLVL6  | Numeric   | 6      | 0   | Transfer Level        | None           |
| AFACCT  | Numeric   | 12     | 0   | Account Number        | AccountNumber  |
| AFTRDT  | Numeric   | 8      | 0   | Transfer Date (CYMD)  | None           |
| AFTRTM  | Numeric   | 6      | 0   | Transfer Time (HHMMSS)| None           |
| AFTYPE  | Char      | 2      | 0   | Transfer Type Code    | None           |
| AFMRNO  | Char      | 12     | 0   | Medical Record Number | MRN            |

(Exact types/lengths inferred from common AS400 conventions; the PHI tagging is sourced directly from the registry.)

---

### 1.2 HXPDICT – Dictionary / Reference File

- **Record format**: `HXFDICT`
- **Unique key**: No
- **Key fields**: none in compact schema (DDS likely defines composite keys).
- **Total fields**: 2705
- **PHI flag**: Yes
- **PHI fields and categories**:
  - `CCMRNO` – MRN
  - `XFBTEL` – PhoneNumber
  - `XCNAME` – PatientName
  - `HXRMNO` – RoomNumber
  - `XFRMNO` – RoomNumber
  - `HVACCT` – AccountNumber
  - `IMGMRN` – MRN
  - `HXGMRN` – MRN
  - `IHMRNO` – MRN
  - `IHACCT` – AccountNumber
  - `WBDATE` – DateOfBirth
  - `XMDMRN` – MRN
  - `ENNAME` – PatientName

**Field table (PHI subset)**

| Field   | Type    | Length | Dec | Column Heading           | PHI Category   |
|---------|---------|--------|-----|--------------------------|----------------|
| CCMRNO  | Char    | 12     | 0   | MRN (Dictionary)         | MRN            |
| XFBTEL  | Char    | 15     | 0   | Benefit Contact Phone    | PhoneNumber    |
| XCNAME  | Char    | 30     | 0   | Patient Name             | PatientName    |
| HXRMNO  | Char    | 6      | 0   | Room Number              | RoomNumber     |
| XFRMNO  | Char    | 6      | 0   | Room Number (Alt)        | RoomNumber     |
| HVACCT  | Numeric | 12     | 0   | Account Number           | AccountNumber  |
| IMGMRN  | Char    | 12     | 0   | Imaging MRN              | MRN            |
| HXGMRN  | Char    | 12     | 0   | Global MRN               | MRN            |
| IHMRNO  | Char    | 12     | 0   | Inpatient MRN            | MRN            |
| IHACCT  | Numeric | 12     | 0   | Inpatient Account        | AccountNumber  |
| WBDATE  | Numeric | 8      | 0   | Birth Date               | DateOfBirth    |
| XMDMRN  | Char    | 12     | 0   | XML Message MRN          | MRN            |
| ENNAME  | Char    | 30     | 0   | Encounter Name           | PatientName    |

---

### 1.3 HXPLVL1 – HXPLVL6 – Level Tables

All level tables share similar characteristics.

#### HXPLVL1

- **Record format**: `HXFLVL1`
- **Unique key**: Yes
- **Key fields**: `HX1NUM`
- **Total fields**: 36
- **PHI flag**: No

#### HXPLVL2

- **Record format**: `HXFLVL2`
- **Unique key**: Yes
- **Key fields**: `HX2NUM`
- **Total fields**: 39
- **PHI flag**: No

#### HXPLVL3

- **Record format**: `HXFLVL3`
- **Unique key**: Yes
- **Key fields**: `HX3NUM`
- **Total fields**: 39
- **PHI flag**: No

#### HXPLVL4

- **Record format**: `HXFLVL4`
- **Unique key**: Yes
- **Key fields**: `HX4NUM`
- **Total fields**: 39
- **PHI flag**: No

#### HXPLVL5

- **Record format**: `HXFLVL5`
- **Unique key**: Yes
- **Key fields**: `HX5NUM`
- **Total fields**: 42
- **PHI flag**: No

#### HXPLVL6

- **Record format**: `HXFLVL6`
- **Unique key**: Yes
- **Key fields**: `HX6NUM`
- **Total fields**: 155
- **PHI flag**: No

**Design note**: Fields describe configuration or rating attributes by level; no PHI is directly stored.

---

### 1.4 HXPTABLD – Dictionary Table

- **Record format**: `XFFTABLD`
- **Unique key**: No
- **Key fields**: `XFDTCD`, `XFDECD`
- **Total fields**: 7
- **PHI flag**: No

**Field table (core)**

| Field   | Type    | Length | Dec | Column Heading         | PHI Category |
|---------|---------|--------|-----|------------------------|--------------|
| XFDTCD  | Char    | 4      | 0   | Data Type Code         | None         |
| XFDECD  | Char    | 8      | 0   | Dictionary Entry Code  | None         |
| XFDMAP  | Char    | 4      | 0   | Mapping Code           | None         |
| XFDLDS  | Char    | 4      | 0   | Level Descriptor Code  | None         |
| XFDSDS  | Char    | 4      | 0   | Status Descriptor Code | None         |

---

### 1.5 HXPXMLD – XML Detail Workfile

- **Record format**: `HXFXMLD`
- **Unique key**: Yes
- **Key fields**: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
- **Total fields**: 4
- **PHI flag**: No

**Field table**

| Field   | Type   | Length | Dec | Column Heading         | PHI Category |
|---------|--------|--------|-----|------------------------|--------------|
| XMDUSR  | Char   | 10     | 0   | User ID                | None         |
| XMDSEQ  | Numeric| 6      | 0   | Sequence Number        | None         |
| XMDSQ2  | Numeric| 6      | 0   | Subsequence Number     | None         |
| XMDTYP  | Char   | 4      | 0   | Message Type           | None         |

---

### 1.6 HXPXMLR – XML Response Workfile

- **Record format**: `HXFXMLR`
- **Unique key**: Yes
- **Key fields**: `XMRUSR`, `XMRSEQ`, `XMRID`
- **Total fields**: 4
- **PHI flag**: No

**Field table**

| Field   | Type   | Length | Dec | Column Heading         | PHI Category |
|---------|--------|--------|-----|------------------------|--------------|
| XMRUSR  | Char   | 10     | 0   | User ID                | None         |
| XMRSEQ  | Numeric| 6      | 0   | Sequence Number        | None         |
| XMRID   | Char   | 12     | 0   | Response Identifier    | None         |
| XMRCDE  | Char   | 4      | 0   | Response Code          | None         |

---

### 1.7 OAPIRNK – Break/Transfer Index

- **Record format**: `HBFIRNK`
- **Unique key**: Yes
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`
- **Total fields**: 33
- **PHI flag**: Yes

**PHI field**

| Field   | Type   | Length | Dec | Column Heading         | PHI Category |
|---------|--------|--------|-----|------------------------|--------------|
| BRKMRN  | Char   | 12     | 0   | Medical Record Number  | MRN          |

---

### 1.8 OMPMAST – Patient Master

- **Record format**: `HMFMAST`
- **Unique key**: Yes
- **Key fields**: `MMPLV6`, `MMACCT`
- **Total fields**: 149
- **PHI flag**: Yes

**PHI fields**

| Field   | Type    | Length | Dec | Column Heading          | PHI Category   |
|---------|---------|--------|-----|-------------------------|----------------|
| MMMRNO  | Char    | 12     | 0   | MRN                     | MRN            |
| MMACCT  | Numeric | 12     | 0   | Account Number          | AccountNumber  |
| MMNAME  | Char    | 30     | 0   | Patient Name            | PatientName    |
| MMPSSN  | Char    | 11     | 0   | Social Security Number  | SSN            |
| MMMMRN  | Char    | 12     | 0   | Alternate MRN           | MRN            |

---

### 1.9 OXPBNFIT – Benefit File

- **Record format**: `XFFBNFIT`
- **Unique key**: Yes
- **Key fields**: `XFBUBN`, `XFBPLN`
- **Total fields**: 34
- **PHI flag**: Yes

**PHI field**

| Field   | Type   | Length | Dec | Column Heading         | PHI Category |
|---------|--------|--------|-----|------------------------|--------------|
| XFBTEL  | Char   | 15     | 0   | Benefit Phone          | PhoneNumber  |

---

### 1.10 OXPNSTN – Plan Status

- **Record format**: `XFFNSTN`
- **Unique key**: Yes
- **Key fields**: `XFNLV6`, `XFNSST`
- **Total fields**: 23
- **PHI flag**: No

---

## 2. Logical Files (LF)

Logical files present indexed views over physical files.

### 2.1 HAPIRNK

- **Physical file (PFILE)**: `TAPIRNK` (missing from repository)
- **Record format**: `HBFIRNK`
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`
- **Select/Omit**: None identified in compact schema.

Description: Logical view for break/transfer records keyed by level, account and sequence. Underlying PF `TAPIRNK` must be recovered to fully document field types and PHI.

---

### 2.2 HMLMAST5H

- **Physical file (PFILE)**: `TMPMAST` (missing)
- **Record format**: `HMFMAST`
- **Key fields**: `MMPNST`, `MMADDT`, `MMADTM`
- **Select/Omit**: None identified.

Description: Historical patient master view keyed by patient status and admission date/time.

---

### 2.3 HXLTABLD

- **Physical file (PFILE)**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**: `XFDTCD`, `XFDMAP`
- **Select/Omit**: None identified.

Description: Dictionary view keyed by data type and mapping code.

---

### 2.4 HXLTABLP

- **Physical file (PFILE)**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**: `XFDTCD`, `XFDLDS`
- **Select/Omit**: None identified.

Description: Dictionary view keyed by data type and level descriptor code.

---

### 2.5 HXLTABLS

- **Physical file (PFILE)**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**: `XFDTCD`, `XFDSDS`
- **Select/Omit**: None identified.

Description: Dictionary view keyed by data type and status descriptor code.

---

### 2.6 HXPBNFIT

- **Physical file (PFILE)**: `TXPBNFIT` (missing)
- **Record format**: `XFFBNFIT`
- **Key fields**: `XFBUBN`, `XFBPLN`
- **Select/Omit**: None identified.

Description: Benefit plan view; underlying PF definition required to enumerate all benefit fields.

---

### 2.7 HXPNSTN

- **Physical file (PFILE)**: `TXPNSTN` (missing)
- **Record format**: `XFFNSTN`
- **Key fields**: `XFNLV6`, `XFNSST`
- **Select/Omit**: None identified.

Description: Plan status view based on level and status code combinations.

---

## 3. PHI Sensitivity Overview

From the PHI registry:

- **PHI-sensitive physical files**: `HAPTRFR`, `HXPDICT`, `OAPIRNK`, `OMPMAST`, `OXPBNFIT`.
- **PHI categories present**:
  - MRN (medical record number)
  - AccountNumber
  - PatientName
  - SSN
  - PhoneNumber
  - DateOfBirth
  - RoomNumber

These annotations should drive database design in the target platform:
- Separate PHI fields into secured schemas or encrypted columns.
- Enforce access control policies at the table and column level.
- Implement lineage tracking from these fields through ETL and service layers.

---

## 4. Summary

The HABADTE data layer is composed of specialized PFs for transfers, patient master data, benefits and status, plus a large dictionary file and a suite of level tables. Logical files provide convenient indexed views over these PFs, though several key PFILEs (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) are missing from the scanned codebase and must be recovered to achieve a complete dictionary.

PHI is concentrated in a small set of tables and fields, all of which have been identified and categorized. This enables targeted security, masking and encryption strategies during legacy modernization, while non-PHI configuration tables (HXPLVL*, HXPTABLD, OXPNSTN, XML workfiles) can be migrated with standard relational modeling practices.
