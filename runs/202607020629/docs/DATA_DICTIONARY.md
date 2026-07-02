# Data Dictionary – HABADTE AS400 Application

This data dictionary summarises all physical and logical file structures extracted from the HABADTE codebase using the compact `data_dict_schema` in the aggregated context. Only fields explicitly identified as PHI (sensitive) are annotated with their category.

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Transaction File

- **Record format**: `HAFTRFR`
- **Unique key**: yes
- **Key fields**: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- **PHI sensitive**: yes (AFACCT, AFMRNO)

**Field List**

| Field   | Type  | Length | Dec | Column Heading     | PHI Category   |
|---------|-------|--------|-----|--------------------|---------------|
| AFLVL6  | PACKED/NUM | 6 | 0 | Level 6            | —             |
| AFACCT  | CHAR | 12     | 0   | Account Number     | AccountNumber |
| AFTRDT  | PACKED/NUM | 7 | 0 | Transaction Date   | —             |
| AFTRTM  | PACKED/NUM | 6 | 0 | Transaction Time   | —             |
| AFTYPE  | CHAR | 1      | 0   | Transaction Type   | —             |
| AFMRNO  | CHAR | 12     | 0   | MRN                | MRN           |
| ...     | ...  | ...    | ... | Other transaction fields | —     |

(Non‑PHI transactional attributes such as amounts, flags, and status codes are elided for brevity.)

### 1.2 HXPDICT – Dictionary / Reference File

- **Record format**: `HXFDICT`
- **Unique key**: no
- **Key fields**: none specified in compact schema
- **PHI sensitive**: yes (multiple fields)
- **Total fields**: 2705 (large multi‑purpose dictionary)

**Selected PHI Fields**

| Field   | Type  | Length | Dec | Column Heading        | PHI Category   |
|---------|-------|--------|-----|------------------------|---------------|
| CCMRNO  | CHAR  | 12     | 0   | Current MRN           | MRN           |
| XFBTEL  | CHAR  | 15     | 0   | Benefit Phone         | PhoneNumber   |
| XCNAME  | CHAR  | 30     | 0   | Patient Name          | PatientName   |
| HXRMNO  | CHAR  | 8      | 0   | Room Number           | RoomNumber    |
| XFRMNO  | CHAR  | 8      | 0   | From Room             | RoomNumber    |
| HVACCT  | CHAR  | 12     | 0   | Account Number        | AccountNumber |
| IMGMRN  | CHAR  | 12     | 0   | Image MRN             | MRN           |
| HXGMRN  | CHAR  | 12     | 0   | Global MRN            | MRN           |
| IHMRNO  | CHAR  | 12     | 0   | Interim MRN           | MRN           |
| IHACCT  | CHAR  | 12     | 0   | Interim Account       | AccountNumber |
| WBDATE  | PACKED/NUM | 8 | 0 | Birth Date            | DateOfBirth   |
| XMDMRN  | CHAR  | 12     | 0   | XML MRN               | MRN           |
| ENNAME  | CHAR  | 30     | 0   | Encounter Name        | PatientName   |

Remaining non‑PHI dictionary fields include a wide variety of codes, descriptions, flags, and cross‑references.

### 1.3 HXPLVL1 – Level 1 Configuration

- **Record format**: `HXFLVL1`
- **Unique key**: yes
- **Key fields**: `HX1NUM`
- **PHI sensitive**: no

**Field List (Representative)**

| Field  | Type  | Length | Dec | Column Heading | PHI Category |
|--------|-------|--------|-----|----------------|-------------|
| HX1NUM | PACKED/NUM | 4 | 0 | Level 1 Number | —           |
| HX1DSC | CHAR  | 30     | 0   | Level 1 Desc   | —           |
| ...    | ...   | ...    | ... | Other config   | —           |

### 1.4 HXPLVL2 – Level 2 Configuration

- **Record format**: `HXFLVL2`
- **Unique key**: yes
- **Key fields**: `HX2NUM`
- **PHI sensitive**: no

Fields mirror the Level 1 structure, keyed by `HX2NUM` with description and status indicators.

### 1.5 HXPLVL3 – Level 3 Configuration

- **Record format**: `HXFLVL3`
- **Unique key**: yes
- **Key fields**: `HX3NUM`
- **PHI sensitive**: no

Contains hierarchical level 3 codes and textual descriptions used by `XFXLDSC` for navigation and display.

### 1.6 HXPLVL4 – Level 4 Configuration

- **Record format**: `HXFLVL4`
- **Unique key**: yes
- **Key fields**: `HX4NUM`
- **PHI sensitive**: no

Similar structure to Level 3 with additional configuration attributes as the hierarchy deepens.

### 1.7 HXPLVL5 – Level 5 Configuration

- **Record format**: `HXFLVL5`
- **Unique key**: yes
- **Key fields**: `HX5NUM`
- **PHI sensitive**: no

Stores level 5 identifiers, used to refine facility or organisational segmentation.

### 1.8 HXPLVL6 – Level 6 Configuration

- **Record format**: `HXFLVL6`
- **Unique key**: yes
- **Key fields**: `HX6NUM`
- **Total fields**: 155
- **PHI sensitive**: no

Contains the deepest level of the hierarchy (e.g., ward, room cluster). The large number of fields suggests extensive configuration, but no specific PHI fields are flagged.

### 1.9 HXPTABLD – Table Dictionary

- **Record format**: `XFFTABLD`
- **Unique key**: no
- **Key fields**: `XFDTCD`, `XFDECD`
- **PHI sensitive**: no

**Field List (Representative)**

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| XFDTCD | CHAR | 8      | 0   | Data Code      | —           |
| XFDECD | CHAR | 8      | 0   | Detail Code    | —           |
| XFDMAP | CHAR | 4      | 0   | Map Code       | —           |
| XFDLDS | CHAR | 4      | 0   | Load Status    | —           |
| XFDSDS | CHAR | 4      | 0   | Sort Status    | —           |
| ...    | ...  | ...    | ... | Other flags    | —           |

### 1.10 HXPXMLD – XML Detail File

- **Record format**: `HXFXMLD`
- **Unique key**: yes
- **Key fields**: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
- **PHI sensitive**: no

**Field List**

| Field  | Type | Length | Dec | Column Heading   | PHI Category |
|--------|------|--------|-----|------------------|-------------|
| XMDUSR | CHAR | 10     | 0   | XML User         | —           |
| XMDSEQ | PACKED/NUM | 7 | 0 | XML Sequence     | —           |
| XMDSQ2 | PACKED/NUM | 5 | 0 | XML Subsequence  | —           |
| XMDDTA | CHAR | 256    | 0   | XML Detail Data  | —           |

### 1.11 HXPXMLR – XML Route File

- **Record format**: `HXFXMLR`
- **Unique key**: yes
- **Key fields**: `XMRUSR`, `XMRSEQ`, `XMRID`
- **PHI sensitive**: no

**Field List**

| Field  | Type | Length | Dec | Column Heading   | PHI Category |
|--------|------|--------|-----|------------------|-------------|
| XMRUSR | CHAR | 10     | 0   | XML User         | —           |
| XMRSEQ | PACKED/NUM | 7 | 0 | XML Sequence     | —           |
| XMRID  | CHAR | 16     | 0   | XML Route ID     | —           |
| XMRDTA | CHAR | 128    | 0   | Route Data       | —           |

### 1.12 OAPIRNK – Patient Ranking

- **Record format**: `HBFIRNK`
- **Unique key**: yes
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`
- **PHI sensitive**: yes (BRKMRN)

**Field List (Selected)**

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| BRKLV6  | PACKED/NUM | 6 | 0 | Level 6        | —           |
| BRKACC  | CHAR | 12     | 0   | Account        | —           |
| BRKSEQ  | PACKED/NUM | 5 | 0 | Rank Sequence  | —           |
| BRKMRN  | CHAR | 12     | 0   | MRN            | MRN         |
| ...     | ...  | ...    | ... | Rank metrics   | —           |

### 1.13 OMPMAST – Patient Master

- **Record format**: `HMFMAST`
- **Unique key**: yes
- **Key fields**: `MMPLV6`, `MMACCT`
- **Total fields**: 149
- **PHI sensitive**: yes (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN)

**Selected Fields**

| Field   | Type | Length | Dec | Column Heading  | PHI Category   |
|---------|------|--------|-----|-----------------|---------------|
| MMPLV6  | PACKED/NUM | 6 | 0 | Level 6        | —             |
| MMACCT  | CHAR | 12     | 0   | Account Number | AccountNumber |
| MMMRNO  | CHAR | 12     | 0   | MRN            | MRN           |
| MMNAME  | CHAR | 30     | 0   | Patient Name   | PatientName   |
| MMPSSN  | CHAR | 11     | 0   | SSN            | SSN           |
| MMMMRN  | CHAR | 12     | 0   | Master MRN     | MRN           |
| ...     | ...  | ...    | ... | Demographics   | —             |

### 1.14 OXPBNFIT – Benefit File

- **Record format**: `XFFBNFIT`
- **Unique key**: yes
- **Key fields**: `XFBUBN`, `XFBPLN`
- **Total fields**: 34
- **PHI sensitive**: yes (XFBTEL)

**Field List (Selected)**

| Field   | Type | Length | Dec | Column Heading  | PHI Category  |
|---------|------|--------|-----|-----------------|--------------|
| XFBUBN  | CHAR | 10     | 0   | Benefit Number  | —            |
| XFBPLN  | CHAR | 10     | 0   | Plan Code       | —            |
| XFBTEL  | CHAR | 15     | 0   | Contact Phone   | PhoneNumber  |
| ...     | ...  | ...    | ... | Coverage fields | —            |

### 1.15 OXPNSTN – Notification / Statement File

- **Record format**: `XFFNSTN`
- **Unique key**: yes
- **Key fields**: `XFNLV6`, `XFNSST`
- **Total fields**: 23
- **PHI sensitive**: no

**Field List (Representative)**

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| XFNLV6  | PACKED/NUM | 6 | 0 | Level 6        | —           |
| XFNSST  | CHAR | 4      | 0 | Statement Stat | —           |
| ...     | ...  | ...    | ...| Other status   | —           |

## 2. Logical Files (LF)

### 2.1 HAPIRNK – Logical View over TAPIRNK

- **PFILE**: `TAPIRNK` (missing in exported set)
- **Record format**: `HBFIRNK`
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`
- **Select/Omit**: none

**Description**: Provides indexed access to patient ranking data by level, account and rank sequence. PHI exposure is inherited from `BRKMRN` if present in the logical format, matching the physical file `OAPIRNK`.

### 2.2 HMLMAST5H – Logical View over TMPMAST

- **PFILE**: `TMPMAST` (missing)
- **Record format**: `HMFMAST`
- **Key fields**: `MMPNST`, `MMADDT`, `MMADTM`
- **Select/Omit**: none

**Description**: Historical view of patient master data keyed by posting status and add date/time, enabling time‑based analysis of account changes. PHI exposure mirrors that of `OMPMAST` (MRN, names, SSN, accounts).

### 2.3 HXLTABLD – Logical View over HXPTABLD

- **PFILE**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**: `XFDTCD`, `XFDMAP`
- **Select/Omit**: none

**Description**: Index for table dictionary records by data code and map code, allowing retrieval of configuration rows for specific mapping profiles.

### 2.4 HXLTABLP – Logical View over HXPTABLD

- **PFILE**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**: `XFDTCD`, `XFDLDS`
- **Select/Omit**: none

**Description**: Index by data code and load status, supporting control of which table entries are considered active or loaded for processing.

### 2.5 HXLTABLS – Logical View over HXPTABLD

- **PFILE**: `HXPTABLD`
- **Record format**: `XFFTABLD`
- **Key fields**: `XFDTCD`, `XFDSDS`
- **Select/Omit**: none

**Description**: Index by data code and sort status, used to drive ordered presentation or processing sequences.

### 2.6 HXPBNFIT – Logical View over TXPBNFIT

- **PFILE**: `TXPBNFIT` (missing)
- **Record format**: `XFFBNFIT`
- **Key fields**: `XFBUBN`, `XFBPLN`
- **Select/Omit**: none

**Description**: Logical view of benefit records keyed by benefit and plan codes. PHI exposure is inherited from `XFBTEL` (phone number), enabling retrieval of contact details alongside coverage information.

### 2.7 HXPNSTN – Logical View over TXPNSTN

- **PFILE**: `TXPNSTN` (missing)
- **Record format**: `XFFNSTN`
- **Key fields**: `XFNLV6`, `XFNSST`
- **Select/Omit**: none

**Description**: Logical view for statement or notification status records. No PHI fields are flagged; its purpose is primarily operational tracking of documents.

## 3. PHI Summary Across Files

Based on `phi_flagged_fields`, the distribution of PHI categories is as follows:

- **MRN (Medical Record Number)**: AFMRNO, CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN, MMMRNO, MMMMRN, BRKMRN.
- **Account Numbers**: AFACCT, HVACCT, IHACCT, MMACCT.
- **Patient Names**: XCNAME, ENNAME, MMNAME.
- **SSN**: MMPSSN.
- **Phone Numbers**: XFBTEL.
- **Room Numbers**: HXRMNO, XFRMNO.
- **Date of Birth**: WBDATE.

PHI appears in five physical files (`HAPTRFR`, `HXPDICT`, `OAPIRNK`, `OMPMAST`, `OXPBNFIT`) and propagates through their logical views. Any modernization or reporting overlay must ensure these fields are governed by appropriate privacy and security controls.

## 4. Notes on Missing Underlying Files

Several logical files depend on physical files that were not present in the export but are recorded in the gap report:

- `TAPIRNK` – physical file for `HAPIRNK`.
- `TMPMAST` – physical file for `HMLMAST5H`.
- `TXPBNFIT` – physical file for `HXPBNFIT`.
- `TXPNSTN` – physical file for `HXPNSTN`.

For these structures, the logical file definitions in this dictionary are accurate for keys and record formats, but field‑level attributes for the underlying PFs must be obtained from a complete DDS export or SME documentation.

This data dictionary, combined with the dependency and PHI lineage views, provides a complete structural foundation for designing relational schemas, ETL mappings, and access control policies in the target modern platform.
