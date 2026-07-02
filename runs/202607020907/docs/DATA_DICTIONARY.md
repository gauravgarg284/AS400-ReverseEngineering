# Data Dictionary – HABADTE AS400 Application

This data dictionary describes the physical (PF) and logical (LF) files identified in the HABADTE application context. It is derived entirely from the aggregated compact data dictionary schema and PHI registry, ensuring that each structure and field-level sensitivity is documented for modernization and governance.

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Patient Transfer File

- **Record format:** HAFTRFR
- **Unique key:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI-sensitive fields:**
  - **AFACCT** – AccountNumber
  - **AFMRNO** – MRN

**Description:** HAPTRFR stores patient transfer records keyed by a level identifier, account number, transfer date and time, and transfer type. The presence of account and MRN fields makes this file a core PHI-bearing structure, typically used in HABADTE to manage or audit inpatient/outpatient transfers.

**Field Table (conceptual):**

| Field   | Type     | Length | Dec | Heading              | PHI Category   |
|---------|----------|--------|-----|----------------------|----------------|
| AFLVL6  | Numeric  | n/a    | n/a | Level 6 Identifier   | None           |
| AFACCT  | Numeric  | n/a    | n/a | Account Number       | AccountNumber  |
| AFTRDT  | Date     | n/a    | n/a | Transfer Date        | None           |
| AFTRTM  | Time     | n/a    | n/a | Transfer Time        | None           |
| AFTYPE  | Char     | n/a    | n/a | Transfer Type        | None           |
| AFMRNO  | Char     | n/a    | n/a | Medical Record No.   | MRN            |
| ...     | ...      | ...    | ... | Other attributes     | As applicable  |

### 1.2 HXPDICT – Dictionary / Reference File

- **Record format:** HXFDICT
- **Unique:** false
- **Total fields:** 2705
- **PHI-sensitive fields:**
  - **CCMRNO** – MRN
  - **XFBTEL** – PhoneNumber
  - **XCNAME** – PatientName
  - **HXRMNO** – RoomNumber
  - **XFRMNO** – RoomNumber
  - **HVACCT** – AccountNumber
  - **IMGMRN** – MRN
  - **HXGMRN** – MRN
  - **IHMRNO** – MRN
  - **IHACCT** – AccountNumber
  - **WBDATE** – DateOfBirth
  - **XMDMRN** – MRN
  - **ENNAME** – PatientName

**Description:** HXPDICT is a very large dictionary or cross-reference file. It aggregates patient identifiers, names, account numbers, room assignments, contact details, and date of birth values. Many MRN and account-related fields indicate that this file serves as a central repository of patient and encounter metadata.

**Field Table (selected fields):**

| Field   | Type     | Length | Dec | Heading                    | PHI Category   |
|---------|----------|--------|-----|----------------------------|----------------|
| CCMRNO  | Char     | n/a    | n/a | Current MRN                | MRN            |
| XFBTEL  | Char     | n/a    | n/a | Contact Phone              | PhoneNumber    |
| XCNAME  | Char     | n/a    | n/a | Patient Name               | PatientName    |
| HXRMNO  | Char     | n/a    | n/a | Room Number                | RoomNumber     |
| XFRMNO  | Char     | n/a    | n/a | Alternate Room Number      | RoomNumber     |
| HVACCT  | Numeric  | n/a    | n/a | Historical Account Number  | AccountNumber  |
| IMGMRN  | Char     | n/a    | n/a | Imaging MRN                | MRN            |
| HXGMRN  | Char     | n/a    | n/a | Global MRN                 | MRN            |
| IHMRNO  | Char     | n/a    | n/a | In-house MRN               | MRN            |
| IHACCT  | Numeric  | n/a    | n/a | In-house Account           | AccountNumber  |
| WBDATE  | Date     | n/a    | n/a | Date of Birth              | DateOfBirth    |
| XMDMRN  | Char     | n/a    | n/a | XML MRN                    | MRN            |
| ENNAME  | Char     | n/a    | n/a | Encounter Name             | PatientName    |
| ...     | ...      | ...    | ... | Additional reference data  | As applicable  |

### 1.3 HXPLVL1 – Level 1 Table

- **Record format:** HXFLVL1
- **Unique key:** HX1NUM
- **Total fields:** 36
- **PHI-sensitive fields:** none

**Description:** HXPLVL1 represents level 1 configuration records, keyed by a numeric level identifier (HX1NUM). It is used for hierarchical classification of institutions, products, or statuses and contains only configuration data without PHI.

### 1.4 HXPLVL2 – Level 2 Table

- **Record format:** HXFLVL2
- **Unique key:** HX2NUM
- **Total fields:** 39
- **PHI-sensitive fields:** none

**Description:** HXPLVL2 extends the hierarchy with level 2 values, again keyed by a numeric identifier. It stores configuration attributes, ranges, or descriptions used by the XFXLDSC program.

### 1.5 HXPLVL3 – Level 3 Table

- **Record format:** HXFLVL3
- **Unique key:** HX3NUM
- **Total fields:** 39
- **PHI-sensitive fields:** none

**Description:** Level 3 classification table, similar in structure to HXPLVL2. The combination of HXPLVL1–3 provides multi-tier classifications.

### 1.6 HXPLVL4 – Level 4 Table

- **Record format:** HXFLVL4
- **Unique key:** HX4NUM
- **Total fields:** 39
- **PHI-sensitive fields:** none

**Description:** Level 4 configuration. Together with lower levels, HXPLVL4 supports complex mapping logic for institutional or status hierarchies.

### 1.7 HXPLVL5 – Level 5 Table

- **Record format:** HXFLVL5
- **Unique key:** HX5NUM
- **Total fields:** 42
- **PHI-sensitive fields:** none

**Description:** Level 5 table used in multi-level mapping. Additional fields beyond levels 1–4 suggest more detailed attributes or extended configuration.

### 1.8 HXPLVL6 – Level 6 Table

- **Record format:** HXFLVL6
- **Unique key:** HX6NUM
- **Total fields:** 155
- **PHI-sensitive fields:** none

**Description:** HXPLVL6 significantly expands the number of attributes per record (155 fields), representing the richest level of configuration. It is heavily read by XFXLDSC for description and mapping logic.

### 1.9 HXPTABLD – Table Definition File

- **Record format:** XFFTABLD
- **Unique:** false
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI-sensitive fields:** none

**Description:** HXPTABLD is a compact table definition file. XFDTCD holds a data or table code, while XFDECD holds a description or extended code. This PF is the base for multiple logical files used to access specific views of the dictionary.

### 1.10 HXPXMLD – XML Detail File

- **Record format:** HXFXMLD
- **Unique key:** XMDUSR, XMDSEQ, XMDSQ2
- **Total fields:** 4
- **PHI-sensitive fields:** none flagged in compact schema (actual payload may contain PHI depending on XML content).

**Description:** HXPXMLD stores XML detail or payload segments keyed by user and sequence fields. It is written by HABADTE during XML message creation.

### 1.11 HXPXMLR – XML Header File

- **Record format:** HXFXMLR
- **Unique key:** XMRUSR, XMRSEQ, XMRID
- **Total fields:** 4

**Description:** HXPXMLR is the XML header record file, capturing user-based sequence identifiers for XML messages. XFXGETID reads from HXFXMLR to drive ID assignments.

### 1.12 OAPIRNK – Break/Rank File

- **Record format:** HBFIRNK
- **Unique key:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI-sensitive fields:**
  - **BRKMRN** – MRN

**Description:** OAPIRNK contains break or rank records keyed by level, account, and sequence. It is closely related to transfer ranking and holds MRN as a sensitive identifier.

### 1.13 OMPMAST – Patient Master File

- **Record format:** HMFMAST
- **Unique key:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI-sensitive fields:**
  - **MMMRNO** – MRN
  - **MMACCT** – AccountNumber
  - **MMNAME** – PatientName
  - **MMPSSN** – SSN
  - **MMMMRN** – MRN

**Description:** OMPMAST is a central patient master file, keyed by level and account. Multiple MRN fields and the presence of SSN and patient name indicate that this file holds a comprehensive set of patient identifiers and attributes.

### 1.14 OXPBNFIT – Benefit File

- **Record format:** XFFBNFIT
- **Unique key:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI-sensitive fields:**
  - **XFBTEL** – PhoneNumber

**Description:** OXPBNFIT stores benefit-related records keyed by benefit and plan codes. Phone number fields provide contact information for benefits or providers.

### 1.15 OXPNSTN – Institution Status File

- **Record format:** XFFNSTN
- **Unique key:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI-sensitive fields:** none

**Description:** OXPNSTN models institution status or location attributes using level and status identifiers. It contributes structural data without direct PHI.

## 2. Logical Files (LF)

Logical files provide alternate views and access paths over PFs.

### 2.1 HAPIRNK – Logical View over TAPIRNK

- **Base PF (PFILE):** TAPIRNK (not present in code drop)
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** none

**Description:** HAPIRNK is a logical file exposing transfer or break records over TAPIRNK. It keys records by level, account, and sequence. TAPIRNK must be obtained from production libraries or configuration to complete the schema.

### 2.2 HMLMAST5H – Logical View over TMPMAST

- **Base PF (PFILE):** TMPMAST (missing)
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM

**Description:** HMLMAST5H provides a historical or posting-oriented view over TMPMAST, keyed by posting state and add date/time. It is likely used to track changes to patient master records over time.

### 2.3 HXLTABLD – Logical View over HXPTABLD (Map View)

- **Base PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDMAP
- **Select/Omit:** none

**Description:** HXLTABLD presents table dictionary records keyed by data code and map identifier, giving access to mapping-specific configurations.

### 2.4 HXLTABLP – Logical View over HXPTABLD (Long Description View)

- **Base PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDLDS

**Description:** HXLTABLP exposes the same dictionary but keyed by data code and long description, used when extended descriptive text is required.

### 2.5 HXLTABLS – Logical View over HXPTABLD (Short Description View)

- **Base PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDSDS

**Description:** HXLTABLS provides a compact view keyed by short descriptions. Together, the three LFs form a triad of access paths over the table dictionary.

### 2.6 HXPBNFIT – Logical View over TXPBNFIT

- **Base PF:** TXPBNFIT (missing)
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN

**Description:** HXPBNFIT exposes benefit records from TXPBNFIT. It is functionally parallel to OXPBNFIT but may include a different subset or historical representation of benefits.

### 2.7 HXPNSTN – Logical View over TXPNSTN

- **Base PF:** TXPNSTN (missing)
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST

**Description:** HXPNSTN presents institution status records for TXPNSTN using level and status keys, likely for fast lookup of institutional attributes.

## 3. PHI Sensitivity Summary

To support compliance and security design, PHI usage across files is summarized below.

### 3.1 PHI by File

- **HAPTRFR:** AccountNumber (AFACCT), MRN (AFMRNO).
- **HXPDICT:** multiple MRN fields (CCMRNO, IMGMRN, HXGMRN, IHMRNO, XMDMRN), AccountNumber fields (HVACCT, IHACCT), PatientName fields (XCNAME, ENNAME), RoomNumber fields (HXRMNO, XFRMNO), PhoneNumber (XFBTEL), DateOfBirth (WBDATE).
- **OAPIRNK:** MRN (BRKMRN).
- **OMPMAST:** MRN fields (MMMRNO, MMMMRN), AccountNumber (MMACCT), PatientName (MMNAME), SSN (MMPSSN).
- **OXPBNFIT:** PhoneNumber (XFBTEL).

### 3.2 Governance Notes

- Files with MRN, SSN, account numbers, names, phone numbers, or dates of birth must be classified as high-sensitivity in any target platform.
- Logical files over PHI-bearing PFs (e.g., HMLMAST5H over TMPMAST, HAPIRNK over TAPIRNK) inherit PHI sensitivity even if specific fields are not explicitly enumerated in this compact schema.
- XML header/detail files (HXPXMLD/R and related HXFXML* implementations) may embed PHI within their payloads; while compact schema does not list PHI fields, modernization must assume potential PHI until full content analysis is performed.

This data dictionary serves as the structural reference for HABADTE’s persistence layer, informing schema migration, data masking, and access control strategies in a modernized environment.
