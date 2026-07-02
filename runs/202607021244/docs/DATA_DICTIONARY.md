# Data Dictionary - HABADTE Project

This data dictionary documents the physical and logical files participating in the HABADTE application, with emphasis on keys, PHI sensitivity, and relationships.

## 1. Physical Files (PF)

### 1.1 HAPTRFR

- **File name:** HAPTRFR
- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI-sensitive flag:** Yes (contains account and MRN identifiers)

#### Field Table

| Field   | Type    | Length | Dec | Column Heading        | PHI Category   |
|--------|---------|--------|-----|------------------------|----------------|
| AFLVL6 | Numeric | 6      | 0   | Level-6 Code          | None           |
| AFACCT | Numeric | 12     | 0   | Account Number        | AccountNumber  |
| AFTRDT | Numeric | 8      | 0   | Transfer Date (CCYYMMDD)| None         |
| AFTRTM | Numeric | 6      | 0   | Transfer Time (HHMMSS)| None           |
| AFTYPE | Char    | 1      | 0   | Transfer Type         | None           |
| AFMRNO | Char    | 12     | 0   | Medical Record Number | MRN            |
| ...    | ...     | ...    | ... | Other transfer fields | None           |

### 1.2 HXPDICT

- **File name:** HXPDICT
- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** (not specified in compact schema)
- **Total fields:** 2705
- **PHI-sensitive flag:** Yes (large concentration of PHI fields)

#### Selected PHI Fields

| Field   | Type    | Length | Dec | Column Heading          | PHI Category   |
|--------|---------|--------|-----|--------------------------|----------------|
| CCMRNO | Char    | 12     | 0   | MRN                      | MRN            |
| XFBTEL | Char    | 15     | 0   | Telephone Number         | PhoneNumber    |
| XCNAME | Char    | 40     | 0   | Patient Name             | PatientName    |
| HXRMNO | Char    | 10     | 0   | Room Number              | RoomNumber     |
| XFRMNO | Char    | 10     | 0   | Room Number              | RoomNumber     |
| HVACCT | Numeric | 12     | 0   | Account Number           | AccountNumber  |
| IMGMRN | Char    | 12     | 0   | MRN                      | MRN            |
| HXGMRN | Char    | 12     | 0   | MRN                      | MRN            |
| IHMRNO | Char    | 12     | 0   | MRN                      | MRN            |
| IHACCT | Numeric | 12     | 0   | Account Number           | AccountNumber  |
| WBDATE | Numeric | 8      | 0   | Date of Birth           | DateOfBirth    |
| XMDMRN | Char    | 12     | 0   | MRN                      | MRN            |
| ENNAME | Char    | 40     | 0   | Patient Name             | PatientName    |

Non-PHI fields in HXPDICT include various codes, descriptions, flags, and configuration values used across the system.

### 1.3 HXPLVL1

- **File name:** HXPLVL1
- **Record format:** HXFLVL1
- **Unique key:** Yes
- **Key fields:** HX1NUM
- **Total fields:** 36
- **PHI-sensitive flag:** No

#### Field Table (Representative)

| Field   | Type    | Length | Dec | Column Heading       | PHI Category |
|--------|---------|--------|-----|-----------------------|--------------|
| HX1NUM | Numeric | 6      | 0   | Level-1 Number       | None         |
| HX1DSC | Char    | 40     | 0   | Level-1 Description  | None         |
| HX1STS | Char    | 1      | 0   | Status Flag          | None         |
| ...    | ...     | ...    | ... | Other level fields   | None         |

### 1.4 HXPLVL2

- **File name:** HXPLVL2
- **Record format:** HXFLVL2
- **Unique key:** Yes
- **Key fields:** HX2NUM
- **Total fields:** 39
- **PHI-sensitive flag:** No

Similar structure to HXPLVL1 but for level 2.

### 1.5 HXPLVL3

- **File name:** HXPLVL3
- **Record format:** HXFLVL3
- **Unique key:** Yes
- **Key fields:** HX3NUM
- **Total fields:** 39
- **PHI-sensitive flag:** No

### 1.6 HXPLVL4

- **File name:** HXPLVL4
- **Record format:** HXFLVL4
- **Unique key:** Yes
- **Key fields:** HX4NUM
- **Total fields:** 39
- **PHI-sensitive flag:** No

### 1.7 HXPLVL5

- **File name:** HXPLVL5
- **Record format:** HXFLVL5
- **Unique key:** Yes
- **Key fields:** HX5NUM
- **Total fields:** 42
- **PHI-sensitive flag:** No

### 1.8 HXPLVL6

- **File name:** HXPLVL6
- **Record format:** HXFLVL6
- **Unique key:** Yes
- **Key fields:** HX6NUM
- **Total fields:** 155
- **PHI-sensitive flag:** No

HXPLVL6 extends the corporate structure hierarchy with the richest set of attributes. These level files are consumed by XFXLDSC to translate codes such as MMPLV6 into human-readable facility or organizational descriptions.

### 1.9 HXPTABLD

- **File name:** HXPTABLD
- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI-sensitive flag:** No

#### Field Table

| Field   | Type    | Length | Dec | Column Heading      | PHI Category |
|--------|---------|--------|-----|----------------------|--------------|
| XFDTCD | Char    | 4      | 0   | Data Type Code      | None         |
| XFDECD | Char    | 4      | 0   | Detail Code         | None         |
| XFDMAP | Numeric | 4      | 0   | Map Code            | None         |
| XFDLDS | Char    | 40     | 0   | Long Description    | None         |
| XFDSDS | Char    | 20     | 0   | Short Description   | None         |
| ...    | ...     | ...    | ... | Other table fields  | None         |

### 1.10 HXPXMLD

- **File name:** HXPXMLD
- **Record format:** HXFXMLD
- **Unique key:** Yes
- **Key fields:** XMDUSR, XMDSEQ, XMDSQ2
- **Total fields:** 4
- **PHI-sensitive flag:** No

#### Field Table

| Field   | Type    | Length | Dec | Column Heading   | PHI Category |
|--------|---------|--------|-----|-------------------|--------------|
| XMDUSR | Char    | 10     | 0   | User ID          | None         |
| XMDSEQ | Numeric | 7      | 0   | XML Sequence     | None         |
| XMDSQ2 | Numeric | 7      | 0   | XML Subsequence  | None         |
| XMDDTA | Char    | 256    | 0   | XML Data         | None         |

### 1.11 HXPXMLR

- **File name:** HXPXMLR
- **Record format:** HXFXMLR
- **Unique key:** Yes
- **Key fields:** XMRUSR, XMRSEQ, XMRID
- **Total fields:** 4
- **PHI-sensitive flag:** No

### 1.12 OAPIRNK

- **File name:** OAPIRNK
- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI-sensitive flag:** Yes

#### Selected Fields

| Field   | Type    | Length | Dec | Column Heading      | PHI Category |
|--------|---------|--------|-----|----------------------|--------------|
| BRKLV6 | Numeric | 6      | 0   | Level-6 Code        | None         |
| BRKACC | Numeric | 12     | 0   | Account Number      | AccountNumber|
| BRKSEQ | Numeric | 4      | 0   | Break Sequence      | None         |
| BRKMRN | Char    | 12     | 0   | Medical Record No.  | MRN          |

### 1.13 OMPMAST

- **File name:** OMPMAST
- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI-sensitive flag:** Yes

#### Selected PHI Fields

| Field   | Type    | Length | Dec | Column Heading         | PHI Category   |
|--------|---------|--------|-----|-------------------------|----------------|
| MMPLV6 | Numeric | 6      | 0   | Level-6 Code           | None           |
| MMACCT | Numeric | 12     | 0   | Account Number         | AccountNumber  |
| MMNAME | Char    | 40     | 0   | Patient Name           | PatientName    |
| MMPSSN | Char    | 9      | 0   | Social Security Number | SSN            |
| MMMRNO | Char    | 12     | 0   | MRN                    | MRN            |
| MMMMRN | Char    | 12     | 0   | Alternate MRN          | MRN            |

Other fields include demographic information, insurance details, and contact data.

### 1.14 OXPBNFIT

- **File name:** OXPBNFIT
- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI-sensitive flag:** Yes

#### Selected Fields

| Field   | Type    | Length | Dec | Column Heading       | PHI Category |
|--------|---------|--------|-----|-----------------------|--------------|
| XFBUBN | Char    | 10     | 0   | Benefit Number       | None         |
| XFBPLN | Char    | 10     | 0   | Plan Code            | None         |
| XFBTEL | Char    | 15     | 0   | Contact Phone Number | PhoneNumber  |

### 1.15 OXPNSTN

- **File name:** OXPNSTN
- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI-sensitive flag:** No

This file contains nursing station definitions keyed by level and station code.

## 2. Logical Files (LF)

Logical files provide secondary access paths and projections over the PFs.

### 2.1 HAPIRNK

- **File name:** HAPIRNK
- **Based on PF:** TAPIRNK (missing)
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/omit description:** None (all records)

HAPIRNK exposes transfer or break information with a key aligned to OAPIRNK and HAPTRFR structures, but its base PF TAPIRNK is not present in the current snapshot.

### 2.2 HMLMAST5H

- **File name:** HMLMAST5H
- **Based on PF:** TMPMAST (missing)
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/omit description:** None

This LF focuses on patient master records keyed by nursing station and admit date/time, suitable for census-style reports such as HABADTE.

### 2.3 HXLTABLD

- **File name:** HXLTABLD
- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDMAP
- **Select/omit description:** None

### 2.4 HXLTABLP

- **File name:** HXLTABLP
- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDLDS
- **Select/omit description:** None

### 2.5 HXLTABLS

- **File name:** HXLTABLS
- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDSDS
- **Select/omit description:** None

These three logical files provide alternative keys and projections onto the same configuration table, enabling the XFXTABL utility to retrieve mappings, long descriptions, and short descriptions depending on context.

### 2.6 HXPBNFIT

- **File name:** HXPBNFIT
- **Based on PF:** TXPBNFIT (missing)
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/omit description:** None

HXPBNFIT is used by benefits-related components and potentially by HABADTE when enriching account information with benefit data.

### 2.7 HXPNSTN

- **File name:** HXPNSTN
- **Based on PF:** TXPNSTN (missing)
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/omit description:** None

HXPNSTN provides nursing station names and attributes based on level and station code. HABADTE chains to HXPNSTN via key `nstkey` to populate dtlnurse with friendly station names.

## 3. PHI Summary Across Files

PHI categories and their locations:

- **MRN (Medical Record Number):** AFMRNO (HAPTRFR), CCMRNO/IMGMRN/HXGMRN/IHMRNO/XMDMRN (HXPDICT), BRKMRN (OAPIRNK), MMMRNO/MMMMRN (OMPMAST).
- **AccountNumber:** AFACCT (HAPTRFR), HVACCT/IHACCT (HXPDICT), MMACCT (OMPMAST), BRKACC (OAPIRNK).
- **PatientName:** XCNAME/ENNAME (HXPDICT), MMNAME (OMPMAST).
- **PhoneNumber:** XFBTEL (HXPDICT, OXPBNFIT).
- **RoomNumber:** HXRMNO/XFRMNO (HXPDICT).
- **DateOfBirth:** WBDATE (HXPDICT).
- **SSN:** MMPSSN (OMPMAST).

These fields are central to the HABADTE report and associated utilities, and they should be treated as restricted-access data in any modernization effort.

## 4. Relationships Overview

Summarizing physical-logical relationships:

- TAPIRNK → HAPIRNK (LF)
- TMPMAST → HMLMAST5H (LF)
- HXPTABLD → HXLTABLD / HXLTABLP / HXLTABLS (LFs)
- TXPBNFIT → HXPBNFIT (LF)
- TXPNSTN → HXPNSTN (LF)

These relationships define the database backbone supporting HABADTE’s census and reporting logic. Missing PFs (TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN) must be restored or stubbed before full end-to-end testing.
