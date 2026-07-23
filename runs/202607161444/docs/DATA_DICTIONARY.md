# Data Dictionary – HABADTE Application

This data dictionary summarizes the physical and logical files used by the HABADTE Active Patients by Date and Time report and its supporting utilities. It is derived from the compact `data_dict_schema` and focuses on structure, keys, and PHI-sensitive fields.

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Patient Transfer File

- **Record format**: `HAFTRFR`  
- **Unique key**: Yes  
- **Key fields**: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`  
- **Total fields**: 28  
- **PHI sensitive flag**: Yes (contains PHI fields)

**Field Overview**

| Field Name | Type      | Length | Dec | Column Heading     | PHI Category   |
|-----------|-----------|--------|-----|--------------------|----------------|
| AFLVL6    | Numeric   | 6      | 0   | Facility Level 6   | None           |
| AFACCT    | Numeric   | N/A    | N/A | Account Number     | AccountNumber  |
| AFTRDT    | Numeric   | 8      | 0   | Transfer Date      | None           |
| AFTRTM    | Numeric   | 6      | 0   | Transfer Time      | None           |
| AFTYPE    | Alpha     | N/A    | N/A | Transfer Type      | None           |
| AFMRNO    | Numeric   | N/A    | N/A | Medical Record #   | MRN            |
| ...       | ...       | ...    | ... | Additional fields  | None / As coded|

This file is used by HABADTE to locate room assignments within a requested census date/time window and to determine current occupancy and leave status.

### 1.2 HXPDICT – Extended Patient Dictionary

- **Record format**: `HXFDICT`  
- **Unique key**: No  
- **Key fields**: *(none in compact schema)*  
- **Total fields**: 2705  
- **PHI sensitive flag**: Yes

**Key PHI Fields**

| Field Name | Type    | Length | Dec | Column Heading     | PHI Category   |
|-----------|---------|--------|-----|--------------------|----------------|
| CCMRNO    | Numeric | N/A    | N/A | Central MRN        | MRN            |
| XFBTEL    | Alpha   | N/A    | N/A | Phone Number       | PhoneNumber    |
| XCNAME    | Alpha   | N/A    | N/A | Patient Name       | PatientName    |
| HXRMNO    | Alpha   | N/A    | N/A | Room Number        | RoomNumber     |
| XFRMNO    | Alpha   | N/A    | N/A | Room Number        | RoomNumber     |
| HVACCT    | Numeric | N/A    | N/A | Account Number     | AccountNumber  |
| IMGMRN    | Numeric | N/A    | N/A | MRN (Image)        | MRN            |
| HXGMRN    | Numeric | N/A    | N/A | MRN (Global)       | MRN            |
| IHMRNO    | Numeric | N/A    | N/A | MRN (History)      | MRN            |
| IHACCT    | Numeric | N/A    | N/A | Account Number     | AccountNumber  |
| WBDATE    | Numeric | N/A    | N/A | Date of Birth      | DateOfBirth    |
| XMDMRN    | Numeric | N/A    | N/A | MRN (XML)          | MRN            |
| ENNAME    | Alpha   | N/A    | N/A | Patient Name       | PatientName    |

The remainder of the 2705 fields include additional demographic, contact, billing, and configuration attributes not explicitly listed here. Many are non-PHI or supporting codes.

### 1.3 HXPLVL1 – Level 1 Hierarchy

- **Record format**: `HXFLVL1`  
- **Unique key**: Yes  
- **Key fields**: `HX1NUM`  
- **Total fields**: 36  
- **PHI sensitive flag**: No

This table defines top-level corporate hierarchy values (e.g., corporation or system-level identifiers). XFXLDSC may reference this as part of level description resolution.

### 1.4 HXPLVL2 – Level 2 Hierarchy

- **Record format**: `HXFLVL2`  
- **Unique key**: Yes  
- **Key fields**: `HX2NUM`  
- **Total fields**: 39  
- **PHI sensitive flag**: No

Represents the second tier of hierarchy (e.g., regions or networks).

### 1.5 HXPLVL3 – Level 3 Hierarchy

- **Record format**: `HXFLVL3`  
- **Unique key**: Yes  
- **Key fields**: `HX3NUM`  
- **Total fields**: 39  
- **PHI sensitive flag**: No

Defines third-level corporate entities.

### 1.6 HXPLVL4 – Level 4 Hierarchy

- **Record format**: `HXFLVL4`  
- **Unique key**: Yes  
- **Key fields**: `HX4NUM`  
- **Total fields**: 39  
- **PHI sensitive flag**: No

### 1.7 HXPLVL5 – Level 5 Hierarchy

- **Record format**: `HXFLVL5`  
- **Unique key**: Yes  
- **Key fields**: `HX5NUM`  
- **Total fields**: 42  
- **PHI sensitive flag**: No

### 1.8 HXPLVL6 – Level 6 Hierarchy

- **Record format**: `HXFLVL6`  
- **Unique key**: Yes  
- **Key fields**: `HX6NUM`  
- **Total fields**: 155  
- **PHI sensitive flag**: No

Level 6 typically represents facility-level identifiers (e.g., individual hospitals). HABADTE uses level 6 values (MMPLV6, AFLVL6) to scope census reports.

### 1.9 HXPTABLD – Generic Table Definition

- **Record format**: `XFFTABLD`  
- **Unique key**: No  
- **Key fields**: `XFDTCD`, `XFDECD`  
- **Total fields**: 7  
- **PHI sensitive flag**: No

This table stores generic code values and descriptions used for mapping categories such as room classes and other reference codes.

### 1.10 HXPXMLD – XML Detail File

- **Record format**: `HXFXMLD`  
- **Unique key**: Yes  
- **Key fields**: `XMDUSR`, `XMDSEQ`, `XMDSQ2`  
- **Total fields**: 4  
- **PHI sensitive flag**: No

Each record represents an XML fragment tied to a user and sequence. HABADTE writes detail rows such as patient line items and summary totals.

### 1.11 HXPXMLR – XML Report Header File

- **Record format**: `HXFXMLR`  
- **Unique key**: Yes  
- **Key fields**: `XMRUSR`, `XMRSEQ`, `XMRID`  
- **Total fields**: 4  
- **PHI sensitive flag**: No

Contains XML report-level data such as header records, report identifiers, and run metadata.

### 1.12 OAPIRNK – Inpatient Ranking File

- **Record format**: `HBFIRNK`  
- **Unique key**: Yes  
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`  
- **Total fields**: 33  
- **PHI sensitive flag**: Yes

**Key PHI Field**

| Field Name | Type    | Length | Dec | Column Heading | PHI Category |
|-----------|---------|--------|-----|----------------|--------------|
| BRKMRN    | Numeric | N/A    | N/A | MRN            | MRN          |

This file indexes or ranks inpatient accounts for reporting or analytical purposes.

### 1.13 OMPMAST – Inpatient Master File

- **Record format**: `HMFMAST`  
- **Unique key**: Yes  
- **Key fields**: `MMPLV6`, `MMACCT`  
- **Total fields**: 149  
- **PHI sensitive flag**: Yes

**Key PHI Fields**

| Field Name | Type    | Length | Dec | Column Heading     | PHI Category   |
|-----------|---------|--------|-----|--------------------|----------------|
| MMMRNO    | Numeric | N/A    | N/A | MRN                | MRN            |
| MMACCT    | Numeric | N/A    | N/A | Account Number     | AccountNumber  |
| MMNAME    | Alpha   | N/A    | N/A | Patient Name       | PatientName    |
| MMPSSN    | Numeric | N/A    | N/A | SSN                | SSN            |
| MMMMRN    | Numeric | N/A    | N/A | Secondary MRN      | MRN            |

### 1.14 OXPBNFIT – Benefit Plan File

- **Record format**: `XFFBNFIT`  
- **Unique key**: Yes  
- **Key fields**: `XFBUBN`, `XFBPLN`  
- **Total fields**: 34  
- **PHI sensitive flag**: Yes

**Key PHI Field**

| Field Name | Type  | Length | Dec | Column Heading | PHI Category   |
|-----------|-------|--------|-----|----------------|----------------|
| XFBTEL    | Alpha | N/A    | N/A | Phone Number   | PhoneNumber    |

This file holds benefit plan definitions and related contact info.

### 1.15 OXPNSTN – Nursing Station File

- **Record format**: `XFFNSTN`  
- **Unique key**: Yes  
- **Key fields**: `XFNLV6`, `XFNSST`  
- **Total fields**: 23  
- **PHI sensitive flag**: No

Defines nursing stations, their identifiers, and descriptive names. HABADTE uses this via the HXPNSTN LF to show station names in the report and XML.

### 1.16 TAPIRNK – Table-based Ranking PF

- **Record format**: `ATE TABLE`  
- **Unique key**: No  
- **Key fields**: *(none)*  
- **Total fields**: 43  
- **PHI sensitive flag**: No

Base table for ranking data, surfaced through HAPIRNK.

### 1.17 TMPMAST – Table-based Master PF

- **Record format**: `ATE TABLE`  
- **Unique key**: No  
- **Key fields**: *(none)*  
- **Total fields**: 181  
- **PHI sensitive flag**: No (PHI-bearing fields are flagged in OMPMAST instead).

### 1.18 TXPBNFIT – Table-based Benefit PF

- **Record format**: `ATE TABLE`  
- **Unique key**: No  
- **Key fields**: *(none)*  
- **Total fields**: 12  
- **PHI sensitive flag**: No

### 1.19 TXPNSTN – Table-based Nursing Station PF

- **Record format**: `ATE TABLE`  
- **Unique key**: No  
- **Key fields**: *(none)*  
- **Total fields**: 19  
- **PHI sensitive flag**: No

## 2. Logical Files (LF)

### 2.1 HAPIRNK – Logical View of TAPIRNK

- **Based on PF**: `TAPIRNK`  
- **Record format**: `HBFIRNK`  
- **Key fields**: `BRKLV6`, `BRKACC`, `BRKSEQ`  
- **Select/Omit**: None documented

Provides an ordered view of inpatient ranking or index data by facility (level 6) and account.

### 2.2 HMLMAST5H – Logical View of TMPMAST

- **Based on PF**: `TMPMAST`  
- **Record format**: `HMFMAST`  
- **Key fields**: `MMPNST`, `MMADDT`, `MMADTM`  
- **Select/Omit**: None documented

Supports views of patient master data sorted by nursing station and admission date/time.

### 2.3 HXLTABLD – Logical View of HXPTABLD (Data Code + Map)

- **Based on PF**: `HXPTABLD`  
- **Record format**: `XFFTABLD`  
- **Key fields**: `XFDTCD`, `XFDMAP`  
- **Select/Omit**: None documented

Used when lookups require mapping codes associated with a data code (e.g., determining room class mapping).

### 2.4 HXLTABLP – Logical View of HXPTABLD (Data Code + Long Description)

- **Based on PF**: `HXPTABLD`  
- **Record format**: `XFFTABLD`  
- **Key fields**: `XFDTCD`, `XFDLDS`  
- **Select/Omit**: None documented

Provides fast access by data code and long description.

### 2.5 HXLTABLS – Logical View of HXPTABLD (Data Code + Short Description)

- **Based on PF**: `HXPTABLD`  
- **Record format**: `XFFTABLD`  
- **Key fields**: `XFDTCD`, `XFDSDS`  
- **Select/Omit**: None documented

Supports lookups by data code and short description.

### 2.6 HXPBNFIT – Logical View of TXPBNFIT

- **Based on PF**: `TXPBNFIT`  
- **Record format**: `XFFBNFIT`  
- **Key fields**: `XFBUBN`, `XFBPLN`  
- **Select/Omit**: None documented

Creates a keyed access path over benefit plan table data.

### 2.7 HXPNSTN – Logical View of TXPNSTN

- **Based on PF**: `TXPNSTN`  
- **Record format**: `XFFNSTN`  
- **Key fields**: `XFNLV6`, `XFNSST`  
- **Select/Omit**: None documented

Provides a keyed view of nursing station definitions for use in HABADTE’s room/station resolution logic.

## 3. PHI Sensitivity Summary

The following physical files contain PHI-tagged fields and should be treated as sensitive data sources:

- **HAPTRFR** – Account numbers and MRNs.  
- **HXPDICT** – Extensive identifiers, names, room numbers, phone numbers, dates of birth.  
- **OAPIRNK** – MRNs in ranking/index records.  
- **OMPMAST** – MRNs, account numbers, patient names, SSNs.  
- **OXPBNFIT** – Phone numbers.

Logical files inherit PHI sensitivity from their base PFs where applicable. For example, HMLMAST5H uses TMPMAST, which indirectly relates to the inpatient master data represented in OMPMAST.

In production environments, all access to these tables through HABADTE and XFX* utilities should be audited and controlled, ensuring that only authorized users and services can view or export PHI.

