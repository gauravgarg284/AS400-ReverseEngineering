# DATA_DICTIONARY – HABADTE Application

This data dictionary summarizes the physical and logical files used by the HABADTE application based on the aggregated compact schema. For each physical file (PF) we list record format, keys, uniqueness, PHI sensitivity, and a descriptive view of fields. For logical files (LF) we describe their base PF, key structure, and purpose.

> Note: Only fields and PHI categories explicitly present in the aggregated schema are documented. Where field-level details are not available, the description indicates the presence and nature of PHI without fabricating missing attributes.

---

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Patient Transfer File

- **Record format:** HAFTRFR  
- **Unique key:** Yes  
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE  
- **Total fields:** 28  
- **PHI-sensitive:** Yes  
- **PHI fields:** AFACCT (AccountNumber), AFMRNO (MRN)

**Field Definitions**

| Field   | Type  | Length | Dec | Column Heading | PHI Category   |
|---------|-------|--------|-----|----------------|----------------|
| AFLVL6  | ?     | ?      | ?   | Level 6        | –              |
| AFACCT  | ?     | ?      | ?   | Account        | AccountNumber  |
| AFTRDT  | ?     | ?      | ?   | Transfer Date  | –              |
| AFTRTM  | ?     | ?      | ?   | Transfer Time  | –              |
| AFTYPE  | ?     | ?      | ?   | Transfer Type  | –              |
| AFMRNO  | ?     | ?      | ?   | Medical Rec No | MRN            |
| ...     | ...   | ...    | ... | ...            | ...            |

Other non-PHI fields include status codes, audit attributes (user, job, timestamp), and control flags used by HABADTE to determine which records are eligible for XML generation and profile updates.

---

### 1.2 HXPDICT – Cross-Reference Dictionary

- **Record format:** HXFDICT  
- **Unique key:** No  
- **Key fields:** (none in schema)  
- **Total fields:** 2705  
- **PHI-sensitive:** Yes  
- **PHI fields:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME

**Selected PHI Field Definitions**

| Field   | Type | Length | Dec | Column Heading      | PHI Category   |
|---------|------|--------|-----|---------------------|----------------|
| CCMRNO  | ?    | ?      | ?   | Current MRN         | MRN            |
| XFBTEL  | ?    | ?      | ?   | Contact Phone       | PhoneNumber    |
| XCNAME  | ?    | ?      | ?   | Patient Name        | PatientName    |
| HXRMNO  | ?    | ?      | ?   | Room Number         | RoomNumber     |
| XFRMNO  | ?    | ?      | ?   | Alternate Room No   | RoomNumber     |
| HVACCT  | ?    | ?      | ?   | Hospital Account    | AccountNumber  |
| IMGMRN  | ?    | ?      | ?   | Imaging MRN         | MRN            |
| HXGMRN  | ?    | ?      | ?   | Global MRN          | MRN            |
| IHMRNO  | ?    | ?      | ?   | Historical MRN      | MRN            |
| IHACCT  | ?    | ?      | ?   | Historical Account  | AccountNumber  |
| WBDATE  | ?    | ?      | ?   | Birth Date          | DateOfBirth    |
| XMDMRN  | ?    | ?      | ?   | XML Detail MRN      | MRN            |
| ENNAME  | ?    | ?      | ?   | Encounter Name      | PatientName    |

The remaining fields in HXPDICT represent a wide range of attributes (codes, dates, flags, descriptions) that cross-reference patient, encounter, and facility information.

---

### 1.3 HXPLVL1 – HXPLVL6 – Level/Plan Tables

These physical files store hierarchical level or plan information.

- **HXPLVL1**  
  - Record format: HXFLVL1  
  - Unique key: Yes  
  - Key fields: HX1NUM  
  - Total fields: 36  
  - PHI-sensitive: No

- **HXPLVL2**  
  - Record format: HXFLVL2  
  - Unique key: Yes  
  - Key fields: HX2NUM  
  - Total fields: 39  
  - PHI-sensitive: No

- **HXPLVL3**  
  - Record format: HXFLVL3  
  - Unique key: Yes  
  - Key fields: HX3NUM  
  - Total fields: 39  
  - PHI-sensitive: No

- **HXPLVL4**  
  - Record format: HXFLVL4  
  - Unique key: Yes  
  - Key fields: HX4NUM  
  - Total fields: 39  
  - PHI-sensitive: No

- **HXPLVL5**  
  - Record format: HXFLVL5  
  - Unique key: Yes  
  - Key fields: HX5NUM  
  - Total fields: 42  
  - PHI-sensitive: No

- **HXPLVL6**  
  - Record format: HXFLVL6  
  - Unique key: Yes  
  - Key fields: HX6NUM  
  - Total fields: 155  
  - PHI-sensitive: No

**Common Field Patterns (illustrative)**

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|--------------|
| HXxNUM | ?    | ?      | ?   | Level Number   | –            |
| HXxDSC | ?    | ?      | ?   | Level Desc     | –            |
| ...    | ...  | ...    | ... | ...            | ...          |

These tables are used by XFXLDSC to resolve human-readable descriptions and attributes associated with level codes.

---

### 1.4 HXPTABLD – Generic Table Dictionary

- **Record format:** XFFTABLD  
- **Unique key:** No  
- **Key fields:** XFDTCD, XFDECD  
- **Total fields:** 7  
- **PHI-sensitive:** No

**Field Definitions (logical)**

| Field  | Type | Length | Dec | Column Heading   | PHI Category |
|--------|------|--------|-----|------------------|--------------|
| XFDTCD | ?    | ?      | ?   | Table Code       | –            |
| XFDECD | ?    | ?      | ?   | Entry Code       | –            |
| XFDESC | ?    | ?      | ?   | Entry Description| –            |
| ...    | ...  | ...    | ... | ...              | ...          |

---

### 1.5 HXPXMLD – XML Detail Table

- **Record format:** HXFXMLD  
- **Unique key:** Yes  
- **Key fields:** XMDUSR, XMDSEQ, XMDSQ2  
- **Total fields:** 4  
- **PHI-sensitive:** No (no specific PHI fields flagged)

**Field Definitions**

| Field  | Type | Length | Dec | Column Heading      | PHI Category |
|--------|------|--------|-----|---------------------|--------------|
| XMDUSR | ?    | ?      | ?   | User                | –            |
| XMDSEQ | ?    | ?      | ?   | Sequence            | –            |
| XMDSQ2 | ?    | ?      | ?   | Detail Sequence     | –            |
| XMDTXT | ?    | ?      | ?   | XML Detail Payload  | –            |

---

### 1.6 HXPXMLR – XML Record Header Table

- **Record format:** HXFXMLR  
- **Unique key:** Yes  
- **Key fields:** XMRUSR, XMRSEQ, XMRID  
- **Total fields:** 4  
- **PHI-sensitive:** No (no specific PHI fields flagged)

**Field Definitions**

| Field  | Type | Length | Dec | Column Heading   | PHI Category |
|--------|------|--------|-----|------------------|--------------|
| XMRUSR | ?    | ?      | ?   | User             | –            |
| XMRSEQ | ?    | ?      | ?   | Sequence         | –            |
| XMRID  | ?    | ?      | ?   | Record Id        | –            |
| XMRHDR | ?    | ?      | ?   | XML Header Data  | –            |

---

### 1.7 OAPIRNK – Rank File

- **Record format:** HBFIRNK  
- **Unique key:** Yes  
- **Key fields:** BRKLV6, BRKACC, BRKSEQ  
- **Total fields:** 33  
- **PHI-sensitive:** Yes  
- **PHI fields:** BRKMRN (MRN)

**Selected Field Definitions**

| Field   | Type | Length | Dec | Column Heading      | PHI Category |
|---------|------|--------|-----|---------------------|--------------|
| BRKLV6  | ?    | ?      | ?   | Level 6             | –            |
| BRKACC  | ?    | ?      | ?   | Account             | –            |
| BRKSEQ  | ?    | ?      | ?   | Sequence            | –            |
| BRKMRN  | ?    | ?      | ?   | Medical Rec Number  | MRN          |
| ...     | ...  | ...    | ... | ...                 | ...          |

---

### 1.8 OMPMAST – Patient Master File

- **Record format:** HMFMAST  
- **Unique key:** Yes  
- **Key fields:** MMPLV6, MMACCT  
- **Total fields:** 149  
- **PHI-sensitive:** Yes  
- **PHI fields:** MMMRNO (MRN), MMACCT (AccountNumber), MMNAME (PatientName), MMPSSN (SSN), MMMMRN (MRN)

**Selected PHI Field Definitions**

| Field   | Type | Length | Dec | Column Heading      | PHI Category   |
|---------|------|--------|-----|---------------------|----------------|
| MMPLV6  | ?    | ?      | ?   | Plan Level          | –              |
| MMACCT  | ?    | ?      | ?   | Account             | AccountNumber  |
| MMNAME  | ?    | ?      | ?   | Patient Name        | PatientName    |
| MMPSSN  | ?    | ?      | ?   | SSN                 | SSN            |
| MMMRNO  | ?    | ?      | ?   | Primary MRN         | MRN            |
| MMMMRN  | ?    | ?      | ?   | Merged MRN          | MRN            |
| ...     | ...  | ...    | ... | ...                 | ...            |

---

### 1.9 OXPBNFIT – Benefit File

- **Record format:** XFFBNFIT  
- **Unique key:** Yes  
- **Key fields:** XFBUBN, XFBPLN  
- **Total fields:** 34  
- **PHI-sensitive:** Yes  
- **PHI fields:** XFBTEL (PhoneNumber)

**Selected Field Definitions**

| Field   | Type | Length | Dec | Column Heading   | PHI Category |
|---------|------|--------|-----|------------------|--------------|
| XFBUBN  | ?    | ?      | ?   | Benefit Code     | –            |
| XFBPLN  | ?    | ?      | ?   | Plan Code        | –            |
| XFBTEL  | ?    | ?      | ?   | Contact Phone    | PhoneNumber  |
| ...     | ...  | ...    | ... | ...              | ...          |

---

### 1.10 OXPNSTN – Status File

- **Record format:** XFFNSTN  
- **Unique key:** Yes  
- **Key fields:** XFNLV6, XFNSST  
- **Total fields:** 23  
- **PHI-sensitive:** No

This file stores status codes associated with a particular level. No PHI fields are flagged.

---

### 1.11 TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN – Staging Tables

Each of these is labeled "ATE TABLE" in the schema, with no unique key fields.

- **TAPIRNK** – 43 fields, PHI-sensitive: No (by explicit PHI list).  
- **TMPMAST** – 181 fields, PHI-sensitive: No (PHI is captured in OMPMAST instead).  
- **TXPBNFIT** – 12 fields, PHI-sensitive: No (PHI is in OXPBNFIT via XFBTEL).  
- **TXPNSTN** – 19 fields, PHI-sensitive: No.

These files appear to be temporary or external representations of rank, patient master, benefit, and status data.

---

## 2. Logical Files (LF)

### 2.1 HAPIRNK – Rank Logical File

- **Based on PF:** TAPIRNK  
- **Record format:** HBFIRNK  
- **Key fields:** BRKLV6, BRKACC, BRKSEQ  
- **Select/Omit:** None

**Description:** Provides a keyed view over TAPIRNK compatible with OAPIRNK’s layout, used by programs that expect the HBFIRNK format for ranking records.

---

### 2.2 HMLMAST5H – Patient Master Logical File

- **Based on PF:** TMPMAST  
- **Record format:** HMFMAST  
- **Key fields:** MMPNST, MMADDT, MMADTM  
- **Select/Omit:** None

**Description:** Supplies an access path over patient master staging data ordered by patient status and admission datetime, used for history or audit views.

---

### 2.3 HXLTABLD / HXLTABLP / HXLTABLS – Generic Table Logical Files

All three logical files share the record format XFFTABLD and base file HXPTABLD.

- **HXLTABLD**  
  - Key fields: XFDTCD, XFDMAP  
  - Select/Omit: None  
  - Description: Access path by table code and “map” code, typically used when mapping application values.

- **HXLTABLP**  
  - Key fields: XFDTCD, XFDLDS  
  - Select/Omit: None  
  - Description: Access path by table code and "load" descriptor; likely used when loading configuration values.

- **HXLTABLS**  
  - Key fields: XFDTCD, XFDSDS  
  - Select/Omit: None  
  - Description: Access path by table code and "status" descriptor; used when retrieving status-specific entries.

These LFs provide multiple index strategies on the same seven-field dictionary HXPTABLD, enabling XFXTABL and related logic to perform efficient lookups.

---

### 2.4 HXPBNFIT – Benefit Logical File

- **Based on PF:** TXPBNFIT  
- **Record format:** XFFBNFIT  
- **Key fields:** XFBUBN, XFBPLN  
- **Select/Omit:** None

**Description:** Indexed view over benefit staging data, aligned with the structure of OXPBNFIT so that programs can read staged and live benefit data using a common format.

---

### 2.5 HXPNSTN – Status Logical File

- **Based on PF:** TXPNSTN  
- **Record format:** XFFNSTN  
- **Key fields:** XFNLV6, XFNSST  
- **Select/Omit:** None

**Description:** Indexed view over status staging data, aligned to OXPNSTN’s layout and keyed by level and status.

---

## 3. PHI Sensitivity Overview

Across all tables:

- PHI-bearing files: HAPTRFR, HXPDICT, OAPIRNK, OMPMAST, OXPBNFIT
- PHI categories encountered: MRN, AccountNumber, PatientName, PhoneNumber, RoomNumber, DateOfBirth, SSN

These files should be subject to enhanced access controls, auditing, and encryption at rest/in transit in any modernized architecture.

---

End of DATA_DICTIONARY for HABADTE.
