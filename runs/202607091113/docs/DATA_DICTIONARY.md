# Data Dictionary – HABADTE Project

This data dictionary summarizes the physical and logical files used in the HABADTE project based on the compact schema from the reverse‑engineering pipeline. Each physical file (PF) section lists its key structure, uniqueness, and PHI sensitivity. Each logical file (LF) section documents its base PF, key fields, and any select/omit behavior.

---

## 1. Physical Files (PF)

### 1.1 HAPTRFR (Transfer File)

- Record format: `HAFTRFR`
- Unique key: **Yes**
- Key fields (in order):
  - `AFLVL6` – Level 6 identifier
  - `AFACCT` – Account number (**PHI: AccountNumber**)
  - `AFTRDT` – Transfer date
  - `AFTRTM` – Transfer time
  - `AFTYPE` – Transfer type code
- Total fields: 28
- PHI-sensitive fields:
  - `AFACCT` – categorized as AccountNumber
  - `AFMRNO` – categorized as MRN

**Description**  
HAPTRFR stores transfer transactions at level 6, keyed by account and date/time, capturing patient or account-level movement, possibly between units or financial buckets. PHI is present through account and MRN identifiers.

**Field Catalog (selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category   |
|---------|------|--------|-----|----------------|----------------|
| AFLVL6  | NUM  | 6      | 0   | LVL6          | –              |
| AFACCT  | NUM  | 12     | 0   | ACCOUNT       | AccountNumber  |
| AFMRNO  | NUM  | 12     | 0   | MRN           | MRN            |
| AFTRDT  | NUM  | 8      | 0   | TRANS DATE    | –              |
| AFTRTM  | NUM  | 6      | 0   | TRANS TIME    | –              |
| AFTYPE  | CHAR | 1      | –   | TYPE          | –              |

---

### 1.2 HXPDICT (Dictionary File)

- Record format: `HXFDICT`
- Unique key: **No**
- Key fields: none explicitly defined in compact schema
- Total fields: 2705
- PHI-sensitive fields:
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

**Description**  
HXPDICT is a large dictionary and cross-reference file that consolidates patient identifiers, account numbers, contact details, and other reference attributes. It likely serves as a central lookup pool for multiple business processes.

**Field Catalog (selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category  |
|---------|------|--------|-----|----------------|---------------|
| CCMRNO  | NUM  | 12     | 0   | MRN            | MRN           |
| XFBTEL  | CHAR | 10     | –   | PHONE          | PhoneNumber   |
| XCNAME  | CHAR | 30     | –   | NAME           | PatientName   |
| HXRMNO  | CHAR | 8      | –   | ROOM           | RoomNumber    |
| XFRMNO  | CHAR | 8      | –   | ROOM2          | RoomNumber    |
| HVACCT  | NUM  | 12     | 0   | ACCOUNT        | AccountNumber |
| IMGMRN  | NUM  | 12     | 0   | MRN IMG        | MRN           |
| HXGMRN  | NUM  | 12     | 0   | MRN G          | MRN           |
| IHMRNO  | NUM  | 12     | 0   | MRN H          | MRN           |
| IHACCT  | NUM  | 12     | 0   | ACCOUNT H      | AccountNumber |
| WBDATE  | NUM  | 8      | 0   | BIRTH DATE     | DateOfBirth   |
| XMDMRN  | NUM  | 12     | 0   | MRN XML        | MRN           |
| ENNAME  | CHAR | 30     | –   | NAME2          | PatientName   |

---

### 1.3 HXPLVL1 – HXPLVL6 (Level Tables)

#### HXPLVL1
- Record format: `HXFLVL1`
- Unique key: **Yes**
- Key fields: `HX1NUM`
- Total fields: 36
- PHI: none

#### HXPLVL2
- Record format: `HXFLVL2`
- Unique key: **Yes**
- Key fields: `HX2NUM`
- Total fields: 39

#### HXPLVL3
- Record format: `HXFLVL3`
- Unique key: **Yes**
- Key fields: `HX3NUM`
- Total fields: 39

#### HXPLVL4
- Record format: `HXFLVL4`
- Unique key: **Yes**
- Key fields: `HX4NUM`
- Total fields: 39

#### HXPLVL5
- Record format: `HXFLVL5`
- Unique key: **Yes**
- Key fields: `HX5NUM`
- Total fields: 42

#### HXPLVL6
- Record format: `HXFLVL6`
- Unique key: **Yes**
- Key fields: `HX6NUM`
- Total fields: 155

**Description**  
The HXPLVLx family represents hierarchical level tables that classify data (e.g., facility levels, benefit levels). They are heavily used by the XFXLDSC program to resolve codes into descriptions.

**Example Field Catalog (HXPLVL6, selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| HX6NUM  | NUM  | 6      | 0   | LEVEL6        | –           |
| HX6DSC  | CHAR | 40     | –   | DESC          | –           |
| HX6STS  | CHAR | 1      | –   | STATUS        | –           |

---

### 1.4 HXPTABLD (Table Dictionary)

- Record format: `XFFTABLD`
- Unique key: **No**
- Key fields: `XFDTCD`, `XFDECD`
- Total fields: 7
- PHI: none

**Description**  
HXPTABLD is a generic table dictionary storing code and description pairs. It is shared by several logical files that define alternative key sequences.

**Field Catalog (selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| XFDTCD  | CHAR | 4      | –   | TABLE CODE    | –           |
| XFDECD  | CHAR | 4      | –   | ENTRY CODE    | –           |
| XFDESC  | CHAR | 40     | –   | DESCRIPTION   | –           |

---

### 1.5 HXPXMLD (XML Detail) and HXPXMLR (XML Header)

#### HXPXMLD
- Record format: `HXFXMLD`
- Unique key: **Yes**
- Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
- Total fields: 4
- PHI: none flagged in schema

#### HXPXMLR
- Record format: `HXFXMLR`
- Unique key: **Yes**
- Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`
- Total fields: 4

**Description**  
These files store XML header (`HXPXMLR`) and detail (`HXPXMLD`) records, keyed by user and sequence identifiers. They are used in conjunction with HABADTE and XFXGETID to construct or track XML payloads.

---

### 1.6 OAPIRNK (Broker/Rank File)

- Record format: `HBFIRNK`
- Unique key: **Yes**
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Total fields: 33
- PHI-sensitive fields:
  - `BRKMRN` – MRN

**Field Catalog (selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| BRKLV6  | NUM  | 6      | 0   | LVL6          | –           |
| BRKACC  | NUM  | 12     | 0   | ACCOUNT       | –           |
| BRKSEQ  | NUM  | 4      | 0   | SEQ           | –           |
| BRKMRN  | NUM  | 12     | 0   | MRN           | MRN         |

---

### 1.7 OMPMAST (Master File)

- Record format: `HMFMAST`
- Unique key: **Yes**
- Key fields: `MMPLV6`, `MMACCT`
- Total fields: 149
- PHI-sensitive fields:
  - `MMMRNO`, `MMMMRN` – MRN
  - `MMACCT` – AccountNumber
  - `MMNAME` – PatientName
  - `MMPSSN` – SSN

**Description**  
OMPMAST is the primary master data file for members/patients, storing demographic and financial attributes.

**Field Catalog (selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category   |
|---------|------|--------|-----|----------------|----------------|
| MMPLV6  | NUM  | 6      | 0   | LVL6          | –              |
| MMACCT  | NUM  | 12     | 0   | ACCOUNT       | AccountNumber  |
| MMMRNO  | NUM  | 12     | 0   | MRN           | MRN            |
| MMMMRN  | NUM  | 12     | 0   | MRN2          | MRN            |
| MMNAME  | CHAR | 30     | –   | NAME          | PatientName    |
| MMPSSN  | CHAR | 11     | –   | SSN           | SSN            |

---

### 1.8 OXPBNFIT (Benefit File)

- Record format: `XFFBNFIT`
- Unique key: **Yes**
- Key fields: `XFBUBN`, `XFBPLN`
- Total fields: 34
- PHI-sensitive fields:
  - `XFBTEL` – PhoneNumber

**Field Catalog (selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| XFBUBN  | CHAR | 10     | –   | BENEFIT       | –           |
| XFBPLN  | CHAR | 10     | –   | PLAN          | –           |
| XFBTEL  | CHAR | 10     | –   | PHONE         | PhoneNumber |

---

### 1.9 OXPNSTN (Station/State File)

- Record format: `XFFNSTN`
- Unique key: **Yes**
- Key fields: `XFNLV6`, `XFNSST`
- Total fields: 23
- PHI: none

---

### 1.10 TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN (Table Files)

These PFs are described in compact form as "ATE TABLE" with no explicit keys or PHI flags.

- **TAPIRNK**
  - Record format: `ATE TABLE`
  - Unique key: **No**
  - Total fields: 43
  - PHI: none identified in compact schema.

- **TMPMAST**
  - Record format: `ATE TABLE`
  - Unique key: **No**
  - Total fields: 181
  - PHI: none explicitly flagged in compact schema; however, as a table counterpart to OMPMAST, it may contain PHI.

- **TXPBNFIT**
  - Record format: `ATE TABLE`
  - Unique key: **No**
  - Total fields: 12

- **TXPNSTN**
  - Record format: `ATE TABLE`
  - Unique key: **No**
  - Total fields: 19

**Description**  
These tables act as base PFs for logical overlays (HAPIRNK, HMLMAST5H, HXPBNFIT, HXPNSTN) and provide normalized, possibly denormalized, reference datasets.

---

## 2. Logical Files (LF)

### 2.1 HAPIRNK

- PFILE: `TAPIRNK`
- Record format: `HBFIRNK`
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Select/omit: none
- PHI-sensitive fields: inherits `BRKMRN` (MRN) via base PF structure.

**Description**  
Provides an ordered view over TAPIRNK, typically for broker or rank-oriented processing keyed by level, account, and sequence.

---

### 2.2 HMLMAST5H

- PFILE: `TMPMAST`
- Record format: `HMFMAST`
- Key fields: `MMPNST`, `MMADDT`, `MMADTM`
- Select/omit: none

**Description**  
Logical file for master records, keyed by posting state and add date/time, supporting chronological or state-based queries.

---

### 2.3 HXLTABLD

- PFILE: `HXPTABLD`
- Record format: `XFFTABLD`
- Key fields: `XFDTCD`, `XFDMAP`
- Select/omit: none

**Description**  
Defines an access path over the table dictionary emphasizing mapping codes.

---

### 2.4 HXLTABLP

- PFILE: `HXPTABLD`
- Record format: `XFFTABLD`
- Key fields: `XFDTCD`, `XFDLDS`
- Select/omit: none

**Description**  
Provides an access path ordered by table code and description, suited for user-facing lists.

---

### 2.5 HXLTABLS

- PFILE: `HXPTABLD`
- Record format: `XFFTABLD`
- Key fields: `XFDTCD`, `XFDSDS`
- Select/omit: none

**Description**  
Alternate view over HXPTABLD focusing on status or sub-description fields.

---

### 2.6 HXPBNFIT

- PFILE: `TXPBNFIT`
- Record format: `XFFBNFIT`
- Key fields: `XFBUBN`, `XFBPLN`
- Select/omit: none

**Description**  
Logical overlay for benefit records, providing efficient retrieval by benefit and plan.

---

### 2.7 HXPNSTN

- PFILE: `TXPNSTN`
- Record format: `XFFNSTN`
- Key fields: `XFNLV6`, `XFNSST`
- Select/omit: none

**Description**  
Logical view over station/state data, keyed by level and state, used by HABADTE and related utilities for state-based validation or routing.

---

## 3. PHI Field Summary

This section summarizes all PHI-sensitive fields across the schema.

| File     | Field   | PHI Category   |
|----------|---------|----------------|
| HAPTRFR  | AFACCT  | AccountNumber  |
| HAPTRFR  | AFMRNO  | MRN            |
| HXPDICT  | CCMRNO  | MRN            |
| HXPDICT  | XFBTEL  | PhoneNumber    |
| HXPDICT  | XCNAME  | PatientName    |
| HXPDICT  | HXRMNO  | RoomNumber     |
| HXPDICT  | XFRMNO  | RoomNumber     |
| HXPDICT  | HVACCT  | AccountNumber  |
| HXPDICT  | IMGMRN  | MRN            |
| HXPDICT  | HXGMRN  | MRN            |
| HXPDICT  | IHMRNO  | MRN            |
| HXPDICT  | IHACCT  | AccountNumber  |
| HXPDICT  | WBDATE  | DateOfBirth    |
| HXPDICT  | XMDMRN  | MRN            |
| HXPDICT  | ENNAME  | PatientName    |
| OAPIRNK  | BRKMRN  | MRN            |
| OMPMAST  | MMMRNO  | MRN            |
| OMPMAST  | MMACCT  | AccountNumber  |
| OMPMAST  | MMNAME  | PatientName    |
| OMPMAST  | MMPSSN  | SSN            |
| OMPMAST  | MMMMRN  | MRN            |
| OXPBNFIT | XFBTEL  | PhoneNumber    |

PHI must be handled with appropriate access controls and auditing whenever these files are accessed or exposed in downstream systems.
