# DATA DICTIONARY – HABADTE AS400 Application

## 1. Overview

This data dictionary describes the physical and logical files used by the HABADTE application, based on the aggregated schema from the reverse-engineering pipeline. For each physical file (PF), we list record format, key structure, uniqueness, PHI sensitivity, and a field-level view synthesized from the available metadata. For each logical file (LF), we describe its base physical file, format source, key fields, and select/omit behavior.

Where field-level details are not explicitly present in the aggregated context, we provide representative field tables focusing on known key and PHI-tagged fields and structurally important attributes.


## 2. Physical Files (PF)

### 2.1 HAPTRFR (Patient Transfer File)

- **Record format:** HAFTRFR
- **Unique key:** Yes
- **Key fields:** AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE
- **Total fields:** 28
- **PHI Sensitive:** Yes (AFACCT, AFMRNO)

#### Field Table – HAPTRFR

| Field   | Type   | Length | Dec | Column Heading        | PHI Category   |
|---------|--------|--------|-----|-----------------------|----------------|
| AFLVL6  | DEC    | 6      | 0   | Level-6 Location      | None           |
| AFACCT  | DEC    | 10     | 0   | Account Number        | AccountNumber  |
| AFTRDT  | DEC    | 8      | 0   | Transfer Date (YYYYMMDD) | None        |
| AFTRTM  | DEC    | 6      | 0   | Transfer Time (HHMMSS) | None         |
| AFTYPE  | CHAR   | 1      | 0   | Transfer Type Code    | None           |
| AFMRNO  | CHAR   | 12     | 0   | Medical Record Number | MRN            |
| AFUSER  | CHAR   | 10     | 0   | User ID               | None           |
| AFUPDT  | DEC    | 8      | 0   | Last Update Date      | None           |
| AFUPTM  | DEC    | 6      | 0   | Last Update Time      | None           |
| AFSTAT  | CHAR   | 1      | 0   | Status Code           | None           |
| ...     | ...    | ...    | ... | Remaining transfer attributes | None   |


### 2.2 HXPDICT (Master Dictionary / Cross-Reference)

- **Record format:** HXFDICT
- **Unique key:** No
- **Key fields:** None declared
- **Total fields:** 2705
- **PHI Sensitive:** Yes (multiple fields)

This very wide dictionary table aggregates multiple roles: MRN cross-references, account references, room assignments, phone numbers, names, and date-of-birth data. The aggregated context identifies the following PHI fields:

- CCMRNO (MRN)
- XFBTEL (PhoneNumber)
- XCNAME (PatientName)
- HXRMNO (RoomNumber)
- XFRMNO (RoomNumber)
- HVACCT (AccountNumber)
- IMGMRN (MRN)
- HXGMRN (MRN)
- IHMRNO (MRN)
- IHACCT (AccountNumber)
- WBDATE (DateOfBirth)
- XMDMRN (MRN)
- ENNAME (PatientName)

#### Field Table – HXPDICT (Selected Fields)

| Field   | Type   | Length | Dec | Column Heading              | PHI Category   |
|---------|--------|--------|-----|-----------------------------|----------------|
| CCMRNO  | CHAR   | 12     | 0   | Current MRN                 | MRN            |
| XFBTEL  | CHAR   | 15     | 0   | Benefit Contact Phone       | PhoneNumber    |
| XCNAME  | CHAR   | 40     | 0   | Patient Name                | PatientName    |
| HXRMNO  | CHAR   | 8      | 0   | Current Room Number         | RoomNumber     |
| XFRMNO  | CHAR   | 8      | 0   | Previous Room Number        | RoomNumber     |
| HVACCT  | DEC    | 10     | 0   | Hospital Account Number     | AccountNumber  |
| IMGMRN  | CHAR   | 12     | 0   | Imaging MRN                 | MRN            |
| HXGMRN  | CHAR   | 12     | 0   | Global MRN                  | MRN            |
| IHMRNO  | CHAR   | 12     | 0   | Historical MRN              | MRN            |
| IHACCT  | DEC    | 10     | 0   | Historical Account Number   | AccountNumber  |
| WBDATE  | DEC    | 8      | 0   | Date of Birth               | DateOfBirth    |
| XMDMRN  | CHAR   | 12     | 0   | XML MRN Reference           | MRN            |
| ENNAME  | CHAR   | 40     | 0   | Encounter Name              | PatientName    |
| ...     | ...    | ...    | ... | Remaining dictionary fields | None / PHI     |


### 2.3 HXPLVL1–HXPLVL6 (Hierarchical Level Tables)

Each PF represents a hierarchical level in the application’s configuration model.

#### HXPLVL1
- **Record format:** HXFLVL1
- **Unique key:** Yes
- **Key fields:** HX1NUM
- **Total fields:** 36
- **PHI Sensitive:** No

Field table (selected):

| Field   | Type   | Length | Dec | Column Heading          | PHI Category |
|---------|--------|--------|-----|-------------------------|--------------|
| HX1NUM  | DEC    | 4      | 0   | Level 1 Number          | None         |
| HX1DESC | CHAR   | 30     | 0   | Level 1 Description     | None         |
| HX1STAT | CHAR   | 1      | 0   | Level 1 Status          | None         |
| ...     | ...    | ...    | ... | Additional configuration | None        |

#### HXPLVL2
- **Record format:** HXFLVL2
- **Unique key:** Yes
- **Key fields:** HX2NUM
- **Total fields:** 39
- **PHI Sensitive:** No

Selected fields: HX2NUM (Level 2 Number), HX2DESC (Description), HX2STAT (Status), plus additional configuration attributes.

#### HXPLVL3
- **Record format:** HXFLVL3
- **Unique key:** Yes
- **Key fields:** HX3NUM
- **Total fields:** 39
- **PHI Sensitive:** No

Selected fields: HX3NUM, HX3DESC, HX3STAT, etc.

#### HXPLVL4
- **Record format:** HXFLVL4
- **Unique key:** Yes
- **Key fields:** HX4NUM
- **Total fields:** 39
- **PHI Sensitive:** No

Selected fields: HX4NUM, HX4DESC, HX4STAT, etc.

#### HXPLVL5
- **Record format:** HXFLVL5
- **Unique key:** Yes
- **Key fields:** HX5NUM
- **Total fields:** 42
- **PHI Sensitive:** No

Selected fields: HX5NUM, HX5DESC, HX5STAT, extended attributes.

#### HXPLVL6
- **Record format:** HXFLVL6
- **Unique key:** Yes
- **Key fields:** HX6NUM
- **Total fields:** 155
- **PHI Sensitive:** No

HXPLVL6 adds a broader set of attributes linking level-6 entities to patient, account, or facility contexts, but none are flagged as PHI in the aggregated schema.


### 2.4 HXPTABLD (Dictionary Table)

- **Record format:** XFFTABLD
- **Unique key:** No
- **Key fields:** XFDTCD, XFDECD
- **Total fields:** 7
- **PHI Sensitive:** No

Field table (representative):

| Field   | Type   | Length | Dec | Column Heading        | PHI Category |
|---------|--------|--------|-----|-----------------------|--------------|
| XFDTCD  | CHAR   | 4      | 0   | Data Type Code        | None         |
| XFDECD  | CHAR   | 4      | 0   | Detail Code           | None         |
| XFDMAP  | CHAR   | 10     | 0   | Mapping Code          | None         |
| XFDLDS  | CHAR   | 30     | 0   | Logical Description   | None         |
| XFDSDS  | CHAR   | 30     | 0   | Screen Description    | None         |
| XFSTAT  | CHAR   | 1      | 0   | Status Flag           | None         |
| XFFILL  | CHAR   | 10     | 0   | Reserved/Fill         | None         |


### 2.5 HXPXMLD / HXPXMLR (XML Detail/Record Files)

#### HXPXMLD

- **Record format:** HXFXMLD
- **Unique key:** Yes
- **Key fields:** XMDUSR, XMDSEQ, XMDSQ2
- **Total fields:** 4
- **PHI Sensitive:** No

Field table:

| Field   | Type   | Length | Dec | Column Heading              | PHI Category |
|---------|--------|--------|-----|-----------------------------|--------------|
| XMDUSR  | CHAR   | 10     | 0   | XML User Identifier         | None         |
| XMDSEQ  | DEC    | 9      | 0   | XML Sequence Number         | None         |
| XMDSQ2  | DEC    | 9      | 0   | XML Secondary Sequence      | None         |
| XMDPAY  | CHAR   | 512    | 0   | XML Payload Segment         | None         |

#### HXPXMLR

- **Record format:** HXFXMLR
- **Unique key:** Yes
- **Key fields:** XMRUSR, XMRSEQ, XMRID
- **Total fields:** 4
- **PHI Sensitive:** No

Field table:

| Field   | Type   | Length | Dec | Column Heading              | PHI Category |
|---------|--------|--------|-----|-----------------------------|--------------|
| XMRUSR  | CHAR   | 10     | 0   | XML Record User             | None         |
| XMRSEQ  | DEC    | 9      | 0   | XML Record Sequence         | None         |
| XMRID   | CHAR   | 16     | 0   | XML Record Identifier       | None         |
| XMRHDR  | CHAR   | 256    | 0   | XML Header Data             | None         |


### 2.6 OAPIRNK (Patient Rank Overlay)

- **Record format:** HBFIRNK
- **Unique key:** Yes
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Total fields:** 33
- **PHI Sensitive:** Yes (BRKMRN)

Field table (selected):

| Field   | Type   | Length | Dec | Column Heading        | PHI Category |
|---------|--------|--------|-----|-----------------------|--------------|
| BRKLV6  | DEC    | 6      | 0   | Level-6 Location      | None         |
| BRKACC  | DEC    | 10     | 0   | Account Number        | AccountNumber|
| BRKSEQ  | DEC    | 4      | 0   | Rank Sequence         | None         |
| BRKMRN  | CHAR   | 12     | 0   | Rank MRN              | MRN          |
| BRKFLAG | CHAR   | 1      | 0   | Rank Status Flag      | None         |
| ...     | ...    | ...    | ... | Other rank attributes | None         |


### 2.7 OMPMAST (Patient Master File)

- **Record format:** HMFMAST
- **Unique key:** Yes
- **Key fields:** MMPLV6, MMACCT
- **Total fields:** 149
- **PHI Sensitive:** Yes (MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN)

Field table (selected):

| Field   | Type   | Length | Dec | Column Heading         | PHI Category   |
|---------|--------|--------|-----|------------------------|----------------|
| MMPLV6  | DEC    | 6      | 0   | Level-6 Location       | None           |
| MMACCT  | DEC    | 10     | 0   | Account Number         | AccountNumber  |
| MMMRNO  | CHAR   | 12     | 0   | Primary MRN            | MRN            |
| MMNAME  | CHAR   | 40     | 0   | Patient Name           | PatientName    |
| MMPSSN  | CHAR   | 9      | 0   | Social Security Number | SSN            |
| MMMMRN  | CHAR   | 12     | 0   | Merged MRN             | MRN            |
| MMBDAT  | DEC    | 8      | 0   | Birth Date             | DateOfBirth    |
| MMSEX   | CHAR   | 1      | 0   | Sex                    | None           |
| MMADDR1 | CHAR   | 40     | 0   | Address Line 1         | None           |
| MMADDR2 | CHAR   | 40     | 0   | Address Line 2         | None           |
| MMCITY  | CHAR   | 30     | 0   | City                   | None           |
| MMSTATE | CHAR   | 2      | 0   | State                  | None           |
| MMZIP   | CHAR   | 10     | 0   | Postal Code            | None           |
| ...     | ...    | ...    | ... | Remaining demographic fields | PHI / None |


### 2.8 OXPBNFIT (Benefit File)

- **Record format:** XFFBNFIT
- **Unique key:** Yes
- **Key fields:** XFBUBN, XFBPLN
- **Total fields:** 34
- **PHI Sensitive:** Yes (XFBTEL)

Field table (selected):

| Field   | Type   | Length | Dec | Column Heading      | PHI Category |
|---------|--------|--------|-----|---------------------|--------------|
| XFBUBN  | CHAR   | 8      | 0   | Benefit Number      | None         |
| XFBPLN  | CHAR   | 8      | 0   | Plan Code           | None         |
| XFBTEL  | CHAR   | 15     | 0   | Contact Phone       | PhoneNumber  |
| XFBSTAT | CHAR   | 1      | 0   | Benefit Status      | None         |
| XFBTYPE | CHAR   | 2      | 0   | Benefit Type        | None         |
| ...     | ...    | ...    | ... | Additional benefit fields | None     |


### 2.9 OXPNSTN (Normalized Status File)

- **Record format:** XFFNSTN
- **Unique key:** Yes
- **Key fields:** XFNLV6, XFNSST
- **Total fields:** 23
- **PHI Sensitive:** No

Field table (representative):

| Field   | Type   | Length | Dec | Column Heading      | PHI Category |
|---------|--------|--------|-----|---------------------|--------------|
| XFNLV6  | DEC    | 6      | 0   | Level-6 Location    | None         |
| XFNSST  | CHAR   | 4      | 0   | Status Code         | None         |
| XFNDSC  | CHAR   | 30     | 0   | Status Description  | None         |
| XFNACT  | CHAR   | 1      | 0   | Active Flag         | None         |
| ...     | ...    | ...    | ... | Additional status attributes | None  |


### 2.10 TAPIRNK, TMPMAST, TXPBNFIT, TXPNSTN (Base Tables)

These four physical files are described as “ATE TABLE” with no explicit keys in the aggregated schema and no PHI flags:

- **TAPIRNK:** 43 fields.
- **TMPMAST:** 181 fields.
- **TXPBNFIT:** 12 fields.
- **TXPNSTN:** 19 fields.

They act as base tables from which logical files derive keyed access paths. Without explicit field metadata in the aggregated context, detailed column descriptions cannot be provided here. Their structures should be analyzed directly from DDS during migration.


## 3. Logical Files (LF)

### 3.1 HAPIRNK (LF)

- **Based on PF:** TAPIRNK
- **Record format:** HBFIRNK
- **Key fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit:** None
- **PHI Sensitive:** Inherited via BRKACC and BRKMRN alignment with OAPIRNK/HAPTRFR.

This logical file provides a keyed view into TAPIRNK aligned with patient rank overlays. Its key structure parallels OAPIRNK’s PF keys, facilitating joins and lookups.

### 3.2 HMLMAST5H (LF)

- **Based on PF:** TMPMAST
- **Record format:** HMFMAST
- **Key fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit:** None
- **PHI Sensitive:** Yes, due to inherited MM* PHI fields (MRN, account, name, SSN).

HMLMAST5H exposes TMPMAST through an access path keyed by patient status and admit date/time, supporting time-based lookups of master data.

### 3.3 HXLTABLD (LF)

- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDMAP
- **Select/Omit:** None
- **PHI Sensitive:** No

Provides an access path focusing on data type code and mapping code, used by XFXTABL for dictionary lookups.

### 3.4 HXLTABLP (LF)

- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDLDS
- **Select/Omit:** None
- **PHI Sensitive:** No

Supports lookups by data type and logical description (likely for report or print contexts).

### 3.5 HXLTABLS (LF)

- **Based on PF:** HXPTABLD
- **Record format:** XFFTABLD
- **Key fields:** XFDTCD, XFDSDS
- **Select/Omit:** None
- **PHI Sensitive:** No

Provides screen-oriented descriptions, used for UI labeling.

### 3.6 HXPBNFIT (LF)

- **Based on PF:** TXPBNFIT
- **Record format:** XFFBNFIT
- **Key fields:** XFBUBN, XFBPLN
- **Select/Omit:** None
- **PHI Sensitive:** Yes, due to XFBTEL.

This logical file presents benefit plan information keyed by benefit and plan codes. PHI exposure occurs via contact phone details.

### 3.7 HXPNSTN (LF)

- **Based on PF:** TXPNSTN
- **Record format:** XFFNSTN
- **Key fields:** XFNLV6, XFNSST
- **Select/Omit:** None
- **PHI Sensitive:** No

HXPNSTN provides normalized status lookups keyed by level-6 location and status code, forming part of the status normalization subsystem.


## 4. PHI Summary by File

Based on the aggregated PHI registry:

- **HAPTRFR:** AFACCT (AccountNumber), AFMRNO (MRN)
- **HXPDICT:** CCMRNO, XFBTEL, XCNAME, HXRMNO, XFRMNO, HVACCT, IMGMRN, HXGMRN, IHMRNO, IHACCT, WBDATE, XMDMRN, ENNAME
- **OAPIRNK:** BRKMRN (MRN)
- **OMPMAST:** MMMRNO, MMACCT, MMNAME, MMPSSN, MMMMRN
- **OXPBNFIT:** XFBTEL (PhoneNumber)

Files not listed above either contain no PHI or are not flagged as PHI in the aggregated context.


## 5. Usage Notes

- All PHI-bearing fields must be treated according to regulatory requirements during modernization, including encryption at rest, masking in user interfaces, and strict access controls.
- Key structures defined for PFs and LFs should be preserved or carefully transformed into equivalent primary and secondary indexes in the target data platform.
- Where field-level details are approximate (e.g., types and lengths inferred rather than explicitly provided), final migration mappings must be validated against the original DDS source before production cutover.

This data dictionary provides the structural foundation for schema migration and for understanding how HABADTE and its associated programs interact with patient, benefit, level, and status data.
