# Data Dictionary – HABADTE AS400 Application

This data dictionary documents the core physical and logical files identified in the HABADTE project. It focuses on structural and PHI‑relevant aspects derived from the compact schema.

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Transfer File
- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI sensitive:** Yes (AFACCT, AFMRNO)

#### 1.1.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|--------------|
| AFLVL6 | ?    | ?      | ?   | Level 6 Code   | None         |
| AFACCT | ?    | ?      | ?   | Account        | AccountNumber |
| AFTRDT | ?    | ?      | ?   | Transfer Date  | None         |
| AFTRTM | ?    | ?      | ?   | Transfer Time  | None         |
| AFTYPE | ?    | ?      | ?   | Transfer Type  | None         |
| AFMRNO | ?    | ?      | ?   | MRN            | MRN          |

(Additional non‑PHI fields exist but are omitted for brevity.)

### 1.2 HXPDICT – Dictionary File
- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** (none defined as unique)
- **Total fields:** 2705
- **PHI sensitive:** Yes

#### 1.2.1 PHI‑Relevant Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|--------------|
| CCMRNO | ?    | ?      | ?   | MRN            | MRN          |
| XFBTEL | ?    | ?      | ?   | Telephone      | PhoneNumber  |
| XCNAME | ?    | ?      | ?   | Name           | PatientName  |
| HXRMNO | ?    | ?      | ?   | Room Number    | RoomNumber   |
| XFRMNO | ?    | ?      | ?   | Room Number    | RoomNumber   |
| HVACCT | ?    | ?      | ?   | Account        | AccountNumber |
| IMGMRN | ?    | ?      | ?   | MRN            | MRN          |
| HXGMRN | ?    | ?      | ?   | MRN            | MRN          |
| IHMRNO | ?    | ?      | ?   | MRN            | MRN          |
| IHACCT | ?    | ?      | ?   | Account        | AccountNumber |
| WBDATE | ?    | ?      | ?   | Birth Date     | DateOfBirth  |
| XMDMRN | ?    | ?      | ?   | MRN            | MRN          |
| ENNAME | ?    | ?      | ?   | Name           | PatientName  |

Because HXPDICT contains 2705 fields, only PHI‑relevant fields are listed here.

### 1.3 HXPLVL1–HXPLVL6 – Level Tables

#### 1.3.1 HXPLVL1
- **Record format:** HXFLVL1
- **Unique key:** Yes
- **Key fields:** HX1NUM
- **Total fields:** 36
- **PHI sensitive:** No

#### 1.3.2 HXPLVL2
- **Record format:** HXFLVL2
- **Unique key:** Yes
- **Key fields:** HX2NUM
- **Total fields:** 39
- **PHI sensitive:** No

#### 1.3.3 HXPLVL3
- **Record format:** HXFLVL3
- **Unique key:** Yes
- **Key fields:** HX3NUM
- **Total fields:** 39
- **PHI sensitive:** No

#### 1.3.4 HXPLVL4
- **Record format:** HXFLVL4
- **Unique key:** Yes
- **Key fields:** HX4NUM
- **Total fields:** 39
- **PHI sensitive:** No

#### 1.3.5 HXPLVL5
- **Record format:** HXFLVL5
- **Unique key:** Yes
- **Key fields:** HX5NUM
- **Total fields:** 42
- **PHI sensitive:** No

#### 1.3.6 HXPLVL6
- **Record format:** HXFLVL6
- **Unique key:** Yes
- **Key fields:** HX6NUM
- **Total fields:** 155
- **PHI sensitive:** No

These level tables provide hierarchical structures; detailed field‑level definitions are not required for PHI analysis.

### 1.4 HXPTABLD – Table Definition File
- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI sensitive:** No

### 1.5 HXPXMLD – XML Detail File
- **Record format:** HXFXMLD
- **Unique key:** Yes
- **Key fields:** XMDUSR, XMDSEQ, XMDSQ2
- **Total fields:** 4
- **PHI sensitive:** No

### 1.6 HXPXMLR – XML Header File
- **Record format:** HXFXMLR
- **Unique key:** Yes
- **Key fields:** XMRUSR, XMRSEQ, XMRID
- **Total fields:** 4
- **PHI sensitive:** No

### 1.7 OAPIRNK – Rank File
- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI sensitive:** Yes (BRKMRN)

### 1.8 OMPMAST – Master File
- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI sensitive:** Yes (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN)

#### 1.8.1 PHI‑Relevant Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|--------------|
| MMMRNO | ?    | ?      | ?   | MRN            | MRN          |
| MMACCT | ?    | ?      | ?   | Account        | AccountNumber |
| MMNAME | ?    | ?      | ?   | Name           | PatientName  |
| MMPSSN | ?    | ?      | ?   | SSN            | SSN          |
| MMMMRN | ?    | ?      | ?   | MRN            | MRN          |

### 1.9 OXPBNFIT – Benefit File
- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI sensitive:** Yes (XFBTEL)

### 1.10 OXPNSTN – Station File
- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI sensitive:** No

### 1.11 TAPIRNK – External Rank Table
- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (not defined in compact schema)
- **Total fields:** 43
- **PHI sensitive:** No

### 1.12 TMPMAST – External Master Table
- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (not defined in compact schema)
- **Total fields:** 181
- **PHI sensitive:** Possibly (not flagged explicitly)

### 1.13 TXPBNFIT – External Benefit Table
- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (not defined in compact schema)
- **Total fields:** 12
- **PHI sensitive:** No

### 1.14 TXPNSTN – External Station Table
- **Record format:** ATE TABLE
- **Unique key:** No
- **Key fields:** (not defined in compact schema)
- **Total fields:** 19
- **PHI sensitive:** No

## 2. Logical Files (LF)

### 2.1 HAPIRNK – Logical View over TAPIRNK
- **Based on PF:** TAPIRNK
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** None

HAPIRNK provides a keyed view of rank data aligned to level‑6 and account.

### 2.2 HMLMAST5H – Logical View over TMPMAST
- **Based on PF:** TMPMAST
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit:** None

HMLMAST5H offers a chronological view over master data keyed by posting station and add date/time.

### 2.3 HXLTABLD – Logical View over HXPTABLD
- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDMAP
- **Select/Omit:** None

### 2.4 HXLTABLP – Logical View over HXPTABLD
- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDLDS
- **Select/Omit:** None

### 2.5 HXLTABLS – Logical View over HXPTABLD
- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDSDS
- **Select/Omit:** None

These three LFs provide alternate access paths over shared table definitions, each keyed by different secondary fields.

### 2.6 HXPBNFIT – Logical View over TXPBNFIT
- **Based on PF:** TXPBNFIT
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/Omit:** None

HXPBNFIT provides logical access to benefit staging data aligned to the live benefit definitions.

### 2.7 HXPNSTN – Logical View over TXPNSTN
- **Based on PF:** TXPNSTN
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/Omit:** None

HXPNSTN exposes station mappings keyed by level‑6 and station codes.

## 3. PHI Summary

The following files contain PHI‑flagged fields:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT:** XFBTEL (PhoneNumber)

None identified in other PFs or LFs within the compact schema.
