# Data Dictionary – HABADTE System

Run ID: 202607161154  
Project: HABADTE (Patient admission and census reporting utilities)

This document captures the structural data dictionary for the HABADTE application as inferred from the aggregated data dictionary schema. It focuses on physical files (PF) and logical files (LF) relevant to patient admission, census reporting, organizational hierarchy and reference data.

---

## 1. Physical Files (PF)

### 1.1 HAPTRFR – Transfer History File

**Record Format:** HAFTRFR  
**Unique Key:** Yes  
**Key Fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE  
**Total Fields:** 28  
**PHI Sensitive Flag:** Yes (contains MRN and account fields)

**Description:** Stores patient transfer history across units and rooms. Each record represents a movement event for a patient account.

**Key Fields:**
- **AFLVL6** – Organizational level 6 (e.g., ward/unit identifier).  
- **AFACCT** – Patient account number (PHI category: AccountNumber).  
- **AFTRDT** – Transfer date.  
- **AFTRTM** – Transfer time.  
- **AFTYPE** – Transfer type code (e.g., admission, transfer, discharge).

**PHI Fields:**
- **AFACCT** – Account number used to link to patient master records. *(Category: AccountNumber)*
- **AFMRNO** – Medical Record Number for the patient. *(Category: MRN)*

These fields are used by HABADTE to determine current room, census participation and organizational context.

---

### 1.2 HXPDICT – Dictionary / Crosswalk File

**Record Format:** HXFDICT  
**Unique Key:** No  
**Key Fields:** none (operates as generic dictionary)  
**Total Fields:** 2705  
**PHI Sensitive Flag:** Yes (multiple PHI fields)

**Description:** A large multi-purpose dictionary table containing patient-related identifiers, contact details, room mappings and other configuration or crosswalk data.

**Selected PHI Fields:**
- **CCMRNO** – Medical Record Number. *(Category: MRN)*
- **XFBTEL** – Telephone number. *(Category: PhoneNumber)*
- **XCNAME** – Patient name. *(Category: PatientName)*
- **HXRMNO** – Room number. *(Category: RoomNumber)*
- **XFRMNO** – Alternate room number. *(Category: RoomNumber)*
- **HVACCT** – Account number. *(Category: AccountNumber)*
- **IMGMRN** – Imaging MRN. *(Category: MRN)*
- **HXGMRN** – General MRN. *(Category: MRN)*
- **IHMRNO** – MRN variant. *(Category: MRN)*
- **IHACCT** – Account number variant. *(Category: AccountNumber)*
- **WBDATE** – Date of birth. *(Category: DateOfBirth)*
- **XMDMRN** – MRN associated with XML detail records. *(Category: MRN)*
- **ENNAME** – Patient name (alternative field). *(Category: PatientName)*

This file forms a central PHI and configuration hub across the HABADTE ecosystem.

---

### 1.3 HXPLVL1–HXPLVL6 – Organizational Level Tables

These tables define hierarchical organizational levels.

#### HXPLVL1

**Record Format:** HXFLVL1  
**Unique Key:** Yes  
**Key Fields:** HX1NUM  
**Total Fields:** 36  
**PHI Sensitive Flag:** No

**Description:** Organizational hierarchy level 1 (top-level facility grouping). Keys are numeric level codes.

#### HXPLVL2

**Record Format:** HXFLVL2  
**Unique Key:** Yes  
**Key Fields:** HX2NUM  
**Total Fields:** 39  
**PHI Sensitive Flag:** No

#### HXPLVL3

**Record Format:** HXFLVL3  
**Unique Key:** Yes  
**Key Fields:** HX3NUM  
**Total Fields:** 39  
**PHI Sensitive Flag:** No

#### HXPLVL4

**Record Format:** HXFLVL4  
**Unique Key:** Yes  
**Key Fields:** HX4NUM  
**Total Fields:** 39  
**PHI Sensitive Flag:** No

#### HXPLVL5

**Record Format:** HXFLVL5  
**Unique Key:** Yes  
**Key Fields:** HX5NUM  
**Total Fields:** 42  
**PHI Sensitive Flag:** No

#### HXPLVL6

**Record Format:** HXFLVL6  
**Unique Key:** Yes  
**Key Fields:** HX6NUM  
**Total Fields:** 155  
**PHI Sensitive Flag:** No

**Usage:** XFXLDSC reads these tables to resolve organizational level codes into descriptions used by HABADTE for census filtering and labeling.

---

### 1.4 HXPTABLD – Dictionary Table

**Record Format:** XFFTABLD  
**Unique Key:** No  
**Key Fields:** XFDTCD, XFDECD  
**Total Fields:** 7  
**PHI Sensitive Flag:** No

**Description:** Base dictionary file for status and type codes. Logical files HXLTABLD, HXLTABLP and HXLTABLS provide different keyed views over the same records.

**Key Fields:**
- **XFDTCD** – Data type code (e.g., table category).
- **XFDECD** – Detail code within the data type.

**Field Table (compact):**
- XFDTCD – Code – length unknown – description: data type identifier.  
- XFDECD – Code – length unknown – description: detail code.  
- Other fields include description and mapping/special-status indicators used by logical files.

---

### 1.5 HXPXMLD – XML Detail File

**Record Format:** HXFXMLD  
**Unique Key:** Yes  
**Key Fields:** XMDUSR, XMDSEQ, XMDSQ2  
**Total Fields:** 4  
**PHI Sensitive Flag:** No

**Description:** Holds XML detail payload records associated with census export.

**Key Fields:**
- **XMDUSR** – User ID or process identifier associated with the XML batch.  
- **XMDSEQ** – Primary sequence number for XML detail records.  
- **XMDSQ2** – Secondary sequence or segment number.

---

### 1.6 HXPXMLR – XML Reference File

**Record Format:** HXFXMLR  
**Unique Key:** Yes  
**Key Fields:** XMRUSR, XMRSEQ, XMRID  
**Total Fields:** 4  
**PHI Sensitive Flag:** No (although link fields may indirectly reference PHI)

**Description:** Holds XML reference or header records, used for control and identification.

**Key Fields:**
- **XMRUSR** – User or process identifier.  
- **XMRSEQ** – Sequence number.  
- **XMRID** – XML record identifier.

XFXGETID reads HXFXMLR to obtain IDs necessary for writing corresponding detail records in HXPXMLD.

---

### 1.7 OAPIRNK – Operational APIRNK File

**Record Format:** HBFIRNK  
**Unique Key:** Yes  
**Key Fields:** BRKLV6, BRKACC, BRKSEQ  
**Total Fields:** 33  
**PHI Sensitive Flag:** Yes

**PHI Field:**
- **BRKMRN** – Medical Record Number. *(Category: MRN)*

This file stores operational records keyed by organizational level, patient account and sequence. It is related to TAPIRNK/HAPIRNK, which provide parallel table-based and logical views.

---

### 1.8 OMPMAST – Operational Patient Master File

**Record Format:** HMFMAST  
**Unique Key:** Yes  
**Key Fields:** MMPLV6, MMACCT  
**Total Fields:** 149  
**PHI Sensitive Flag:** Yes

**PHI Fields:**
- **MMMRNO** – Medical Record Number. *(Category: MRN)*
- **MMACCT** – Account Number. *(Category: AccountNumber)*
- **MMNAME** – Patient Name. *(Category: PatientName)*
- **MMPSSN** – Social Security Number. *(Category: SSN)*
- **MMMMRN** – Additional MRN field. *(Category: MRN)*

OMPMAST is the canonical patient master file. TMPMAST and HMLMAST5H provide table-based and logical views optimized for census calculations.

---

### 1.9 OXPBNFIT – Operational Benefit File

**Record Format:** XFFBNFIT  
**Unique Key:** Yes  
**Key Fields:** XFBUBN, XFBPLN  
**Total Fields:** 34  
**PHI Sensitive Flag:** Yes

**PHI Field:**
- **XFBTEL** – Telephone number associated with benefit information. *(Category: PhoneNumber)*

This file stores benefit plans keyed by benefit number and plan ID. TXPBNFIT and HXPBNFIT mirror it with table-based and logical views.

---

### 1.10 OXPNSTN – Operational Nursing Station File

**Record Format:** XFFNSTN  
**Unique Key:** Yes  
**Key Fields:** XFNLV6, XFNSST  
**Total Fields:** 23  
**PHI Sensitive Flag:** No

**Description:** Contains nursing station reference data (names, types, attributes) keyed by organizational level and station code. Used by HABADTE and related utilities to resolve station names and classifications.

---

### 1.11 TAPIRNK – Table-Based APIRNK File

**Record Format:** ATE TABLE  
**Unique Key:** No  
**Key Fields:** none  
**Total Fields:** 43  
**PHI Sensitive Flag:** No

**Description:** Table-based version of APIRNK data, used together with HAPIRNK logical file for keyed access.

---

### 1.12 TMPMAST – Table-Based Patient Master File

**Record Format:** ATE TABLE  
**Unique Key:** No  
**Key Fields:** none  
**Total Fields:** 181  
**PHI Sensitive Flag:** No (PHI fields are present but flagged primarily in OMPMAST in compact schema)

**Description:** Table-based patient master used to drive logical views like HMLMAST5H. HABADTE likely iterates TMPMAST records filtered via logical file keys.

---

### 1.13 TXPBNFIT – Table-Based Benefit File

**Record Format:** ATE TABLE  
**Unique Key:** No  
**Key Fields:** none  
**Total Fields:** 12  
**PHI Sensitive Flag:** No

**Description:** Table-based version of benefit data, with HXPBNFIT providing keyed logical access.

---

### 1.14 TXPNSTN – Table-Based Nursing Station File

**Record Format:** ATE TABLE  
**Unique Key:** No  
**Key Fields:** none  
**Total Fields:** 19  
**PHI Sensitive Flag:** No

**Description:** Table-based nursing station master; HXPNSTN is the logical view providing keyed access.

---

## 2. Logical Files (LF)

### 2.1 HAPIRNK – Logical APIRNK

**Based On (PFILE):** TAPIRNK  
**Record Format:** HBFIRNK  
**Key Fields:** BRKLV6, BRKACC, BRKSEQ  
**Select/Omit Description:** None (all records presented).

**Purpose:** Provides keyed access to APIRNK records by organizational level, account and sequence, aligning with the PF OAPIRNK and its MRN content.

---

### 2.2 HMLMAST5H – Logical Patient Master View

**Based On (PFILE):** TMPMAST  
**Record Format:** HMFMAST  
**Key Fields:** MMPNST, MMADDT, MMADTM  
**Select/Omit Description:** None.

**Purpose:** Filters and indexes patient master records by nursing station and admission date/time. HABADTE uses this view to select patients eligible for inclusion on a given census date.

---

### 2.3 HXLTABLD – Logical Dictionary (Mapping View)

**Based On (PFILE):** HXPTABLD  
**Record Format:** XFFTABLD  
**Key Fields:** XFDTCD, XFDMAP  
**Select/Omit Description:** None.

**Purpose:** Provides mapping-oriented dictionary view, where XFDMAP is a mapping code associated with each data type (XFDTCD). Used by XFXTABL to resolve mappings for codes.

---

### 2.4 HXLTABLP – Logical Dictionary (Logical Description View)

**Based On (PFILE):** HXPTABLD  
**Record Format:** XFFTABLD  
**Key Fields:** XFDTCD, XFDLDS  
**Select/Omit Description:** None.

**Purpose:** Presents logical description codes (XFDLDS) for each data type. XFXTABL uses this view to translate codes into more user-friendly logical descriptions.

---

### 2.5 HXLTABLS – Logical Dictionary (Status Description View)

**Based On (PFILE):** HXPTABLD  
**Record Format:** XFFTABLD  
**Key Fields:** XFDTCD, XFDSDS  
**Select/Omit Description:** None.

**Purpose:** Provides status description codes (XFDSDS) associated with data types. This view supports status-based logic such as active/inactive flags, void indicators and other stateful behaviors.

---

### 2.6 HXPBNFIT – Logical Benefit View

**Based On (PFILE):** TXPBNFIT  
**Record Format:** XFFBNFIT  
**Key Fields:** XFBUBN, XFBPLN  
**Select/Omit Description:** None.

**Purpose:** Supplies a keyed view of benefit records by benefit number and plan, mirroring the operational OXPBNFIT PF with a table-based backing. PHI field XFBTEL appears here as well.

---

### 2.7 HXPNSTN – Logical Nursing Station View

**Based On (PFILE):** TXPNSTN  
**Record Format:** XFFNSTN  
**Key Fields:** XFNLV6, XFNSST  
**Select/Omit Description:** None.

**Purpose:** Provides keyed nursing station reference data for HABADTE and related utilities.

---

## 3. PHI Sensitivity Overview

Based on aggregated PHI flags:

- **Files with PHI:** HAPTRFR, HXPDICT, OAPIRNK, OMPMAST, OXPBNFIT.  
- **Common PHI categories:** MRN, AccountNumber, PatientName, PhoneNumber, DateOfBirth, SSN, RoomNumber.

For modernization and compliance:
- Access to these files should be encapsulated behind services with strict role-based access control.
- Field-level masking or tokenization may be required for MRN, account numbers and SSN when exposed in UI or external APIs.
- Audit logging should be enabled for queries touching PHI fields.

---

## 4. Summary

The HABADTE data model combines operational and table-based master files (OMPMAST/TMPMAST, OAPIRNK/TAPIRNK, OXPBNFIT/TXPBNFIT, OXPNSTN/TXPNSTN) with specialized logical views (HMLMAST5H, HAPIRNK, HXLTABL*, HXPBNFIT, HXPNSTN) and large configuration dictionaries (HXPDICT, HXPLVL*, HXPTABLD). This structured dictionary is central to understanding patient census behavior, organizational filtering and reference data resolution, and must be preserved or carefully transformed during legacy modernization.
