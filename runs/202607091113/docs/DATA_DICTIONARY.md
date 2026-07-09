# Data Dictionary

Run ID: 202607091113  
System: Legacy AS/400 (IBM i)

This data dictionary summarizes the logical and physical file structures discovered during reverse engineering, including PHI indicators. Field‑level information is derived from the compact `data_dict_schema` representation and PHI registry.

---

## 1. Physical Files (PF)

Each subsection describes a physical file (PF), its record format, key structure, uniqueness, and PHI sensitivity. Field tables focus on fields explicitly flagged as PHI or structurally significant; the full column set is available in the underlying DDS.

### 1.1 HAPTRFR

- Record format: `HAFTRFR`
- Unique key: `Yes`
- Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- Total fields: 28
- PHI sensitive: `Yes` (AFACCT, AFMRNO)

#### PHI Fields

| Field  | Type    | Length | Dec | Column Heading | PHI Category   |
|--------|---------|--------|-----|----------------|----------------|
| AFACCT | (DDS)   | n/a    | n/a | Account        | AccountNumber  |
| AFMRNO | (DDS)   | n/a    | n/a | MRN            | MRN            |

Design notes: HAPTRFR stores transfer records keyed by level‑6 entity, account, date/time, and type, with direct ties to member accounts and MRNs.

---

### 1.2 HXPDICT

- Record format: `HXFDICT`
- Unique key: `No`
- Key fields: none (dictionary/lookup structure)
- Total fields: 2705
- PHI sensitive: `Yes` (multiple fields)

#### Selected PHI Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category   |
|--------|-------|--------|-----|----------------|----------------|
| CCMRNO | (DDS) | n/a    | n/a | MRN            | MRN            |
| XFBTEL | (DDS) | n/a    | n/a | Phone          | PhoneNumber    |
| XCNAME | (DDS) | n/a    | n/a | Name           | PatientName    |
| HXRMNO | (DDS) | n/a    | n/a | Room           | RoomNumber     |
| XFRMNO | (DDS) | n/a    | n/a | Room           | RoomNumber     |
| HVACCT | (DDS) | n/a    | n/a | Account        | AccountNumber  |
| IMGMRN | (DDS) | n/a    | n/a | MRN            | MRN            |
| HXGMRN | (DDS) | n/a    | n/a | MRN            | MRN            |
| IHMRNO | (DDS) | n/a    | n/a | MRN            | MRN            |
| IHACCT | (DDS) | n/a    | n/a | Account        | AccountNumber  |
| WBDATE | (DDS) | n/a    | n/a | Birth Date     | DateOfBirth    |
| XMDMRN | (DDS) | n/a    | n/a | MRN            | MRN            |
| ENNAME | (DDS) | n/a    | n/a | Name           | PatientName    |

Design notes: HXPDICT is a dense dictionary that mixes code values with PHI, suggesting it functions as a cross‑reference or configuration table with embedded identity data.

---

### 1.3 HXPLVL1

- Record format: `HXFLVL1`
- Unique key: `Yes`
- Key fields: `HX1NUM`
- Total fields: 36
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX1NUM | (DDS) | n/a    | n/a | Level 1 Key    | None        |

---

### 1.4 HXPLVL2

- Record format: `HXFLVL2`
- Unique key: `Yes`
- Key fields: `HX2NUM`
- Total fields: 39
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX2NUM | (DDS) | n/a    | n/a | Level 2 Key    | None        |

---

### 1.5 HXPLVL3

- Record format: `HXFLVL3`
- Unique key: `Yes`
- Key fields: `HX3NUM`
- Total fields: 39
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX3NUM | (DDS) | n/a    | n/a | Level 3 Key    | None        |

---

### 1.6 HXPLVL4

- Record format: `HXFLVL4`
- Unique key: `Yes`
- Key fields: `HX4NUM`
- Total fields: 39
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX4NUM | (DDS) | n/a    | n/a | Level 4 Key    | None        |

---

### 1.7 HXPLVL5

- Record format: `HXFLVL5`
- Unique key: `Yes`
- Key fields: `HX5NUM`
- Total fields: 42
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX5NUM | (DDS) | n/a    | n/a | Level 5 Key    | None        |

---

### 1.8 HXPLVL6

- Record format: `HXFLVL6`
- Unique key: `Yes`
- Key fields: `HX6NUM`
- Total fields: 155
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX6NUM | (DDS) | n/a    | n/a | Level 6 Key    | None        |

---

### 1.9 HXPTABLD

- Record format: `XFFTABLD`
- Unique key: `No`
- Key fields: `XFDTCD`, `XFDECD`
- Total fields: 7
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| XFDTCD | (DDS) | n/a    | n/a | Data Code      | None        |
| XFDECD | (DDS) | n/a    | n/a | Detail Code    | None        |

---

### 1.10 HXPXMLD

- Record format: `HXFXMLD`
- Unique key: `Yes`
- Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
- Total fields: 4
- PHI sensitive: `No` (no PHI fields flagged)

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| XMDUSR | (DDS) | n/a    | n/a | XML User       | None        |
| XMDSEQ | (DDS) | n/a    | n/a | XML Seq        | None        |
| XMDSQ2 | (DDS) | n/a    | n/a | XML Sub Seq    | None        |

---

### 1.11 HXPXMLR

- Record format: `HXFXMLR`
- Unique key: `Yes`
- Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`
- Total fields: 4
- PHI sensitive: `No` (no PHI fields flagged)

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| XMRUSR | (DDS) | n/a    | n/a | XML User       | None        |
| XMRSEQ | (DDS) | n/a    | n/a | XML Seq        | None        |
| XMRID  | (DDS) | n/a    | n/a | XML ID         | None        |

---

### 1.12 OAPIRNK

- Record format: `HBFIRNK`
- Unique key: `Yes`
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Total fields: 33
- PHI sensitive: `Yes` (BRKMRN)

#### PHI Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| BRKMRN | (DDS) | n/a    | n/a | MRN            | MRN         |

---

### 1.13 OMPMAST

- Record format: `HMFMAST`
- Unique key: `Yes`
- Key fields: `MMPLV6`, `MMACCT`
- Total fields: 149
- PHI sensitive: `Yes` (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN)

#### PHI Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category   |
|--------|-------|--------|-----|----------------|----------------|
| MMMRNO | (DDS) | n/a    | n/a | MRN            | MRN            |
| MMACCT | (DDS) | n/a    | n/a | Account        | AccountNumber  |
| MMNAME | (DDS) | n/a    | n/a | Name           | PatientName    |
| MMPSSN | (DDS) | n/a    | n/a | SSN            | SSN            |
| MMMMRN | (DDS) | n/a    | n/a | MRN            | MRN            |

---

### 1.14 OXPBNFIT

- Record format: `XFFBNFIT`
- Unique key: `Yes`
- Key fields: `XFBUBN`, `XFBPLN`
- Total fields: 34
- PHI sensitive: `Yes` (XFBTEL)

#### PHI Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| XFBTEL | (DDS) | n/a    | n/a | Phone          | PhoneNumber |

---

### 1.15 OXPNSTN

- Record format: `XFFNSTN`
- Unique key: `Yes`
- Key fields: `XFNLV6`, `XFNSST`
- Total fields: 23
- PHI sensitive: `No`

#### Key Fields

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| XFNLV6 | (DDS) | n/a    | n/a | Level 6 Key    | None        |
| XFNSST | (DDS) | n/a    | n/a | State/Status   | None        |

---

### 1.16 TAPIRNK

- Record format: `ATE TABLE`
- Unique key: `No`
- Key fields: none
- Total fields: 43
- PHI sensitive: `No` (no PHI fields flagged)

Note: TAPIRNK is the physical base for LF HAPIRNK.

---

### 1.17 TMPMAST

- Record format: `ATE TABLE`
- Unique key: `No`
- Key fields: none
- Total fields: 181
- PHI sensitive: `No` (no PHI fields flagged)

Note: TMPMAST is the physical base for LF HMLMAST5H.

---

### 1.18 TXPBNFIT

- Record format: `ATE TABLE`
- Unique key: `No`
- Key fields: none
- Total fields: 12
- PHI sensitive: `No` (PHI appears in logical view HXPBNFIT via OXPBNFIT, not flagged here)

---

### 1.19 TXPNSTN

- Record format: `ATE TABLE`
- Unique key: `No`
- Key fields: none
- Total fields: 19
- PHI sensitive: `No`

---

## 2. Logical Files (LF)

Logical files present alternate key sequences and, in some cases, filtered subsets of their physical files.

### 2.1 HAPIRNK

- Based on PF: `TAPIRNK`
- Record format: `HBFIRNK`
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Select/Omit: none

Description: Provides a keyed view over TAPIRNK allowing retrieval by level‑6 break line and account sequence. No PHI fields are explicitly flagged at the LF level.

---

### 2.2 HMLMAST5H

- Based on PF: `TMPMAST`
- Record format: `HMFMAST`
- Key fields: `MMPNST`, `MMADDT`, `MMADTM`
- Select/Omit: none

Description: Logical view providing chronological ordering of master records by plan/state and add date/time.

---

### 2.3 HXLTABLD

- Based on PF: `HXPTABLD`
- Record format: `XFFTABLD`
- Key fields: `XFDTCD`, `XFDMAP`
- Select/Omit: none

Description: Logical view used for data code-to-map lookups.

---

### 2.4 HXLTABLP

- Based on PF: `HXPTABLD`
- Record format: `XFFTABLD`
- Key fields: `XFDTCD`, `XFDLDS`
- Select/Omit: none

Description: Logical view used for data code-to-load descriptor mapping.

---

### 2.5 HXLTABLS

- Based on PF: `HXPTABLD`
- Record format: `XFFTABLD`
- Key fields: `XFDTCD`, `XFDSDS`
- Select/Omit: none

Description: Logical view used for data code-to-sub descriptor mapping.

---

### 2.6 HXPBNFIT

- Based on PF: `TXPBNFIT`
- Record format: `XFFBNFIT`
- Key fields: `XFBUBN`, `XFBPLN`
- Select/Omit: none

Description: Logical view exposing benefit details from TXPBNFIT, keyed by benefit and plan numbers. While TXPBNFIT itself is not flagged PHI, benefit phone/contact details are PHI in related PF OXPBNFIT.

---

### 2.7 HXPNSTN

- Based on PF: `TXPNSTN`
- Record format: `XFFNSTN`
- Key fields: `XFNLV6`, `XFNSST`
- Select/Omit: none

Description: Logical view exposing state/plan or status information, keyed by level‑6 code and status/state.

---

## 3. PHI Summary by File

This section summarizes PHI presence per file for quick reference.

### 3.1 PHI‑Bearing Physical Files

- **HAPTRFR** – AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT** – CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK** – BRKMRN (MRN)
- **OMPMAST** – MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT** – XFBTEL (PhoneNumber)

### 3.2 Non‑PHI Physical Files

- HXPLVL1–HXPLVL6 (hierarchy levels)
- HXPTABLD (code table)
- HXPXMLD, HXPXMLR (XML control/detail indices)
- OXPNSTN, TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN (no PHI flags in registry)

---

## 4. Usage Notes

- The dictionary reflects **schema as discovered**; some fields may hold PHI logically even if not flagged, especially free‑form character columns.
- Logical file PHI is typically inherited from the underlying PF; masking and access controls should be enforced at both PF and LF levels in a modernized system.
- Files with record format `ATE TABLE` appear to be generalized tables whose detailed layouts must be inspected in the original DDS for full migration.
