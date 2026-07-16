# Data Dictionary – HABADTE Application

This data dictionary summarises the physical and logical file structures used by the HABADTE AS400 application. All content is derived from the aggregated data_dict_schema in the reverse‑engineering context.

## 1. Physical Files (PF)

Each subsection describes a single DDS physical file, including record format, uniqueness, key fields, and PHI sensitivity. Field‑level details are presented where available; when a file’s detailed field list is not present in the compact schema, only high‑level characteristics are documented.

### 1.1 HAPTRFR – Transfer Records

- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI sensitive:** Yes
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

**Field Table**

| Field Name | Type         | Length | Dec | Column Heading           | PHI Category   |
|-----------|-------------|--------|-----|--------------------------|----------------|
| AFLVL6    | Packed/Num   | n/a    | n/a | Level 6 Code             | None           |
| AFACCT    | Char/Num     | n/a    | n/a | Account Number           | AccountNumber  |
| AFTRDT    | Date/Num     | n/a    | n/a | Transfer Date            | None           |
| AFTRTM    | Time/Num     | n/a    | n/a | Transfer Time            | None           |
| AFTYPE    | Char         | n/a    | n/a | Transfer Type            | None           |
| AFMRNO    | Char/Num     | n/a    | n/a | Medical Record Number    | MRN            |
| ...       | ...          | ...    | ... | Other transfer fields    | None           |

### 1.2 HXPDICT – Dictionary / Cross-Reference

- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** (none)
- **Total fields:** 2705
- **PHI sensitive:** Yes
- **PHI fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

Given its size, HXPDICT behaves as a central dictionary or cross‑reference table containing identifiers, names, contact details, and date‑of‑birth information.

**Selected Field Table**

| Field Name | Type       | Length | Dec | Column Heading              | PHI Category   |
|-----------|-----------|--------|-----|-----------------------------|----------------|
| CCMRNO    | Char/Num   | n/a    | n/a | Cross‑Reference MRN         | MRN            |
| XFBTEL    | Char       | n/a    | n/a | Benefit Telephone           | PhoneNumber    |
| XCNAME    | Char       | n/a    | n/a | Cross‑Reference Name        | PatientName    |
| HXRMNO    | Char/Num   | n/a    | n/a | Hospital Room Number        | RoomNumber     |
| XFRMNO    | Char/Num   | n/a    | n/a | Alternate Room Number       | RoomNumber     |
| HVACCT    | Char/Num   | n/a    | n/a | Hospital Account Number     | AccountNumber  |
| IMGMRN    | Char/Num   | n/a    | n/a | Imaging MRN                 | MRN            |
| HXGMRN    | Char/Num   | n/a    | n/a | General MRN                 | MRN            |
| IHMRNO    | Char/Num   | n/a    | n/a | Inpatient MRN               | MRN            |
| IHACCT    | Char/Num   | n/a    | n/a | Inpatient Account           | AccountNumber  |
| WBDATE    | Date       | n/a    | n/a | Date of Birth               | DateOfBirth    |
| XMDMRN    | Char/Num   | n/a    | n/a | XML Detail MRN              | MRN            |
| ENNAME    | Char       | n/a    | n/a | Encounter Name              | PatientName    |
| ...       | ...        | ...    | ... | Additional dictionary fields| None / Mixed   |

### 1.3 HXPLVL1 – HXPLVL6 – Organisational Level Files

Each HXPLVLx file stores organisational level metadata used for level description lookups.

#### HXPLVL1

- **Record format:** HXFLVL1
- **Unique key:** Yes
- **Key fields:** HX1NUM
- **Total fields:** 36
- **PHI sensitive:** No

#### HXPLVL2

- **Record format:** HXFLVL2
- **Unique key:** Yes
- **Key fields:** HX2NUM
- **Total fields:** 39
- **PHI sensitive:** No

#### HXPLVL3

- **Record format:** HXFLVL3
- **Unique key:** Yes
- **Key fields:** HX3NUM
- **Total fields:** 39
- **PHI sensitive:** No

#### HXPLVL4

- **Record format:** HXFLVL4
- **Unique key:** Yes
- **Key fields:** HX4NUM
- **Total fields:** 39
- **PHI sensitive:** No

#### HXPLVL5

- **Record format:** HXFLVL5
- **Unique key:** Yes
- **Key fields:** HX5NUM
- **Total fields:** 42
- **PHI sensitive:** No

#### HXPLVL6

- **Record format:** HXFLVL6
- **Unique key:** Yes
- **Key fields:** HX6NUM
- **Total fields:** 155
- **PHI sensitive:** No

**Field Table (Representative)**

| Field Name | Type     | Length | Dec | Column Heading            | PHI Category |
|-----------|---------|--------|-----|---------------------------|--------------|
| HXxNUM    | Packed  | n/a    | n/a | Level Number (1–6)       | None         |
| HXxDESC   | Char    | n/a    | n/a | Level Description         | None         |
| ...       | ...     | ...    | ... | Additional level metadata | None         |

("HXx" denotes the level‑specific prefix such as HX1, HX2, etc.)

### 1.4 HXPTABLD – Table‑Driven Dictionary

- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI sensitive:** No

HXPTABLD holds compact dictionary or translation entries keyed by a data code (XFDTCD) and detail code (XFDECD), used extensively by table‑driven logic in XFXTABL.

**Field Table**

| Field Name | Type   | Length | Dec | Column Heading           | PHI Category |
|-----------|-------|--------|-----|--------------------------|--------------|
| XFDTCD    | Char  | n/a    | n/a | Data Code                | None         |
| XFDECD    | Char  | n/a    | n/a | Detail Code              | None         |
| XFDMAP    | Char  | n/a    | n/a | Mapping Code             | None         |
| XFDLDS    | Char  | n/a    | n/a | Load Sequence            | None         |
| XFDSDS    | Char  | n/a    | n/a | Subset Sequence          | None         |
| ...       | ...   | ...    | ... | Additional attributes    | None         |

### 1.5 HXPXMLD – XML Detail File

- **Record format:** HXFXMLD
- **Unique key:** Yes
- **Key fields:** XMDUSR, XMDSEQ, XMDSQ2
- **Total fields:** 4
- **PHI sensitive:** No

**Field Table**

| Field Name | Type   | Length | Dec | Column Heading         | PHI Category |
|-----------|-------|--------|-----|------------------------|--------------|
| XMDUSR    | Char  | n/a    | n/a | User ID                | None         |
| XMDSEQ    | Num   | n/a    | n/a | Sequence Number        | None         |
| XMDSQ2    | Num   | n/a    | n/a | Secondary Sequence     | None         |
| XMDSEG    | Char  | n/a    | n/a | XML Detail Segment     | None         |

### 1.6 HXPXMLR – XML Record File

- **Record format:** HXFXMLR
- **Unique key:** Yes
- **Key fields:** XMRUSR, XMRSEQ, XMRID
- **Total fields:** 4
- **PHI sensitive:** No

**Field Table**

| Field Name | Type   | Length | Dec | Column Heading         | PHI Category |
|-----------|-------|--------|-----|------------------------|--------------|
| XMRUSR    | Char  | n/a    | n/a | User ID                | None         |
| XMRSEQ    | Num   | n/a    | n/a | Sequence Number        | None         |
| XMRID     | Char  | n/a    | n/a | XML Record Identifier  | None         |
| XMRSEG    | Char  | n/a    | n/a | XML Record Segment     | None         |

### 1.7 OAPIRNK – Break/Rank File

- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI sensitive:** Yes
- **PHI fields:** BRKMRN (MRN)

**Field Table (Selected)**

| Field Name | Type   | Length | Dec | Column Heading         | PHI Category |
|-----------|-------|--------|-----|------------------------|--------------|
| BRKLV6    | Num   | n/a    | n/a | Level 6 Code           | None         |
| BRKACC    | Char  | n/a    | n/a | Account Number         | None         |
| BRKSEQ    | Num   | n/a    | n/a | Break Sequence         | None         |
| BRKMRN    | Char  | n/a    | n/a | Medical Record Number  | MRN          |
| ...       | ...   | ...    | ... | Additional rank fields | None         |

### 1.8 OMPMAST – Patient Master File

- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI sensitive:** Yes
- **PHI fields:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN

**Field Table (Selected)**

| Field Name | Type   | Length | Dec | Column Heading         | PHI Category   |
|-----------|-------|--------|-----|------------------------|----------------|
| MMPLV6    | Num   | n/a    | n/a | Level 6 Code           | None           |
| MMACCT    | Char  | n/a    | n/a | Account Number         | AccountNumber  |
| MMNAME    | Char  | n/a    | n/a | Patient Name           | PatientName    |
| MMMRNO    | Char  | n/a    | n/a | Primary MRN            | MRN            |
| MMMMRN    | Char  | n/a    | n/a | Secondary MRN          | MRN            |
| MMPSSN    | Char  | n/a    | n/a | Social Security Number | SSN            |
| ...       | ...   | ...    | ... | Additional demographic  | Mixed          |

### 1.9 OXPBNFIT – Benefit Plan File

- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI sensitive:** Yes
- **PHI fields:** XFBTEL (PhoneNumber)

**Field Table (Selected)**

| Field Name | Type   | Length | Dec | Column Heading       | PHI Category |
|-----------|-------|--------|-----|----------------------|--------------|
| XFBUBN    | Char  | n/a    | n/a | Benefit Number       | None         |
| XFBPLN    | Char  | n/a    | n/a | Plan Code            | None         |
| XFBTEL    | Char  | n/a    | n/a | Contact Telephone    | PhoneNumber  |
| ...       | ...   | ...    | ... | Additional plan data | None         |

### 1.10 OXPNSTN – Status File

- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI sensitive:** No

**Field Table (Representative)**

| Field Name | Type   | Length | Dec | Column Heading       | PHI Category |
|-----------|-------|--------|-----|----------------------|--------------|
| XFNLV6    | Num   | n/a    | n/a | Level 6 Code         | None         |
| XFNSST    | Char  | n/a    | n/a | Status Code          | None         |
| ...       | ...   | ...    | ... | Additional status    | None         |

### 1.11 TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN – ATE Tables

These PFs are indicated as "ATE TABLE" in the schema, representing general tables rather than classic DDS record formats.

#### TAPIRNK

- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (none)
- **Total fields:** 43
- **PHI sensitive:** No

#### TMPMAST

- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (none)
- **Total fields:** 181
- **PHI sensitive:** No

#### TXPBNFIT

- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (none)
- **Total fields:** 12
- **PHI sensitive:** No

#### TXPNSTN

- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (none)
- **Total fields:** 19
- **PHI sensitive:** No

These tables serve as base PFs for logical files, providing raw datasets which are reshaped by LFs for application‑specific access.

## 2. Logical Files (LF)

Logical files define derived record formats and keyed access paths over physical files. Each subsection documents the base PF, record format, key fields, and any select/omit criteria.

### 2.1 HAPIRNK

- **Based on PF:** TAPIRNK
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** None

This LF provides an indexed view on TAPIRNK for break and ranking operations keyed by level‑6 code, account, and sequence.

### 2.2 HMLMAST5H

- **Based on PF:** TMPMAST
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit:** None

HMLMAST5H reshapes the TEMP master table into an admission‑centric view keyed by patient status and admission date/time, supporting census snapshots.

### 2.3 HXLTABLD

- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDMAP
- **Select/Omit:** None

HXLTABLD exposes dictionary records keyed by data code and mapping code, used to drive table‑based mapping logic.

### 2.4 HXLTABLP

- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDLDS
- **Select/Omit:** None

HXLTABLP provides an alternate access path focusing on load sequence, enabling ordered processing of dictionary entries.

### 2.5 HXLTABLS

- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDSDS
- **Select/Omit:** None

HXLTABLS offers a subset‑oriented access path, selecting records based on subset sequence for filtered views.

### 2.6 HXPBNFIT

- **Based on PF:** TXPBNFIT
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/Omit:** None

Provides keyed access to benefit plan data, aligning with benefit number and plan code.

### 2.7 HXPNSTN

- **Based on PF:** TXPNSTN
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/Omit:** None

Offers a keyed interface to status data by organisational level and status code.

## 3. PHI Sensitivity Summary

From the compact schema, the following PFs carry PHI‑tagged fields:

- **HAPTRFR** – account numbers and MRNs.
- **HXPDICT** – MRNs, account numbers, phone numbers, patient names, room numbers, dates of birth.
- **OAPIRNK** – MRNs.
- **OMPMAST** – MRNs, account numbers, patient names, SSNs.
- **OXPBNFIT** – phone numbers.

Logical files over non‑PHI ATE tables (HAPIRNK, HMLMAST5H, HXLTABLD, HXLTABLP, HXLTABLS, HXPBNFIT, HXPNSTN) inherit PHI exposure only indirectly if joined or combined with PHI‑heavy PFs at the program level.

For modernization planning, PHI fields should be treated as sensitive attributes requiring appropriate encryption, masking, or strict access control in the target environment.
