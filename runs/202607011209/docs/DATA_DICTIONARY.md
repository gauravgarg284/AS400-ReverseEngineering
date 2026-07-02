# Data Dictionary – HABADTE Application Suite

This data dictionary summarizes the physical and logical file structures used by the HABADTE patient management ecosystem. It is derived entirely from the compact schema in the aggregated context; no raw DDS source has been loaded.

## 1. Physical Files (PF)

Each subsection describes a physical file, its record format, key structure, uniqueness, and PHI‑sensitive fields. Where detailed field metadata is not available in the compact schema, fields are listed as known PHI attributes with their categories.

### 1.1 HAPTRFR – Patient Transfer/Transaction File

- **File Name:** HAPTRFR
- **Record Format:** HAFTRFR
- **Unique Key:** Yes
- **Key Fields:**
  - AFLVL6 – Level key (coverage/plan level)
  - AFACCT – Account number (PHI)
  - AFTRDT – Transaction date
  - AFTRTM – Transaction time
  - AFTYPE – Transaction type code
- **Total Fields:** 28
- **PHI Sensitive Flag:** Yes
- **PHI Fields:**
  - AFACCT – Category: AccountNumber
  - AFMRNO – Category: MRN

HAPTRFR contains per‑account transactions or transfers, keyed by level and timestamp. PHI is limited to account numbers and medical record numbers.

### 1.2 HXPDICT – Global Dictionary / Registry

- **File Name:** HXPDICT
- **Record Format:** HXFDICT
- **Unique Key:** No
- **Key Fields:** (not specified in compact schema)
- **Total Fields:** 2705
- **PHI Sensitive Flag:** Yes
- **PHI Fields:**
  - CCMRNO – Category: MRN
  - XFBTEL – Category: PhoneNumber
  - XCNAME – Category: PatientName
  - HXRMNO – Category: RoomNumber
  - XFRMNO – Category: RoomNumber
  - HVACCT – Category: AccountNumber
  - IMGMRN – Category: MRN
  - HXGMRN – Category: MRN
  - IHMRNO – Category: MRN
  - IHACCT – Category: AccountNumber
  - WBDATE – Category: DateOfBirth
  - XMDMRN – Category: MRN
  - ENNAME – Category: PatientName

Although individual field types and lengths are not given, HXPDICT clearly aggregates a wide set of identifiers and PHI attributes, including multiple MRNs, account numbers, names, phone numbers, room assignments, and dates of birth.

### 1.3 HXPLVL1–HXPLVL6 – Hierarchical Level Files

These files define level hierarchies, likely for benefits, plans, or organizational structures.

#### HXPLVL1

- **File Name:** HXPLVL1
- **Record Format:** HXFLVL1
- **Unique Key:** Yes
- **Key Fields:** HX1NUM
- **Total Fields:** 36
- **PHI Sensitive Flag:** No

#### HXPLVL2

- **File Name:** HXPLVL2
- **Record Format:** HXFLVL2
- **Unique Key:** Yes
- **Key Fields:** HX2NUM
- **Total Fields:** 39
- **PHI Sensitive Flag:** No

#### HXPLVL3

- **File Name:** HXPLVL3
- **Record Format:** HXFLVL3
- **Unique Key:** Yes
- **Key Fields:** HX3NUM
- **Total Fields:** 39
- **PHI Sensitive Flag:** No

#### HXPLVL4

- **File Name:** HXPLVL4
- **Record Format:** HXFLVL4
- **Unique Key:** Yes
- **Key Fields:** HX4NUM
- **Total Fields:** 39
- **PHI Sensitive Flag:** No

#### HXPLVL5

- **File Name:** HXPLVL5
- **Record Format:** HXFLVL5
- **Unique Key:** Yes
- **Key Fields:** HX5NUM
- **Total Fields:** 42
- **PHI Sensitive Flag:** No

#### HXPLVL6

- **File Name:** HXPLVL6
- **Record Format:** HXFLVL6
- **Unique Key:** Yes
- **Key Fields:** HX6NUM
- **Total Fields:** 155
- **PHI Sensitive Flag:** No

None of these level files contain PHI‑tagged fields in the aggregated registry. They model structural metadata used by level description logic.

### 1.4 HXPTABLD – Generic Table File

- **File Name:** HXPTABLD
- **Record Format:** XFFTABLD
- **Unique Key:** No
- **Key Fields:**
  - XFDTCD – Data code
  - XFDECD – Extended code
- **Total Fields:** 7
- **PHI Sensitive Flag:** No

HXPTABLD is the base physical file for several logical views that present mappings and descriptions. It does not itself contain PHI.

### 1.5 HXPXMLD – XML Detail File

- **File Name:** HXPXMLD
- **Record Format:** HXFXMLD
- **Unique Key:** Yes
- **Key Fields:**
  - XMDUSR – User identifier
  - XMDSEQ – Primary sequence
  - XMDSQ2 – Secondary sequence
- **Total Fields:** 4
- **PHI Sensitive Flag:** No

HXPXMLD stores XML detail segments keyed by user and sequence numbers. PHI is not explicitly tagged here.

### 1.6 HXPXMLR – XML Record File

- **File Name:** HXPXMLR
- **Record Format:** HXFXMLR
- **Unique Key:** Yes
- **Key Fields:**
  - XMRUSR – User identifier
  - XMRSEQ – Sequence
  - XMRID – Record ID
- **Total Fields:** 4
- **PHI Sensitive Flag:** No

HXPXMLR stores XML record headers or control records. It is used by XFXGETID to derive identifiers.

### 1.7 OAPIRNK – Patient Ranking File

- **File Name:** OAPIRNK
- **Record Format:** HBFIRNK
- **Unique Key:** Yes
- **Key Fields:**
  - BRKLV6 – Level key
  - BRKACC – Account number
  - BRKSEQ – Ranking sequence
- **Total Fields:** 33
- **PHI Sensitive Flag:** Yes
- **PHI Fields:**
  - BRKMRN – Category: MRN

OAPIRNK contains ranked patient records keyed by level and account, with MRN as sensitive data.

### 1.8 OMPMAST – Patient Master File

- **File Name:** OMPMAST
- **Record Format:** HMFMAST
- **Unique Key:** Yes
- **Key Fields:**
  - MMPLV6 – Level key
  - MMACCT – Account number
- **Total Fields:** 149
- **PHI Sensitive Flag:** Yes
- **PHI Fields:**
  - MMMRNO – Category: MRN
  - MMACCT – Category: AccountNumber
  - MMNAME – Category: PatientName
  - MMPSSN – Category: SSN
  - MMMMRN – Category: MRN

OMPMAST is the primary patient master file with multiple identifiers and sensitive attributes, including SSN and MRNs.

### 1.9 OXPBNFIT – Benefit Plan File

- **File Name:** OXPBNFIT
- **Record Format:** XFFBNFIT
- **Unique Key:** Yes
- **Key Fields:**
  - XFBUBN – Benefit number
  - XFBPLN – Plan code
- **Total Fields:** 34
- **PHI Sensitive Flag:** Yes
- **PHI Fields:**
  - XFBTEL – Category: PhoneNumber

OXPBNFIT describes benefit plans and associates them with contact numbers, which are PHI.

### 1.10 OXPNSTN – Station/Namespace File

- **File Name:** OXPNSTN
- **Record Format:** XFFNSTN
- **Unique Key:** Yes
- **Key Fields:**
  - XFNLV6 – Level key
  - XFNSST – Namespace or station status
- **Total Fields:** 23
- **PHI Sensitive Flag:** No

OXPNSTN contains station or namespace information; no PHI‑tagged fields are recorded.

## 2. Logical Files (LF)

Logical files provide alternate key sequences, filtered views, or projections over PFs. The compact schema identifies key relationships and select/omit behavior.

### 2.1 HAPIRNK – Logical View over TAPIRNK

- **Logical File Name:** HAPIRNK
- **Based On PF:** TAPIRNK (missing from harvested set)
- **Record Format:** HBFIRNK
- **Key Fields:** BRKLV6, BRKACC, BRKSEQ
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** Yes (via MRN in underlying structures)

HAPIRNK presents ranked patient data from TAPIRNK. Although TAPIRNK’s DDS is missing, HAPIRNK’s format name matches OAPIRNK’s record format, suggesting similar MRN‑bearing structures.

### 2.2 HMLMAST5H – Logical View over TMPMAST

- **Logical File Name:** HMLMAST5H
- **Based On PF:** TMPMAST (missing)
- **Record Format:** HMFMAST
- **Key Fields:** MMPNST, MMADDT, MMADTM
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** Yes (inherits PHI from master data)

HMLMAST5H provides a temporal view over patient master records keyed by posting status and add date/time. Fields such as MRNs, names, accounts, and SSNs exist in the underlying HMFMAST format.

### 2.3 HXLTABLD – Logical Mapping View

- **Logical File Name:** HXLTABLD
- **Based On PF:** HXPTABLD
- **Record Format:** XFFTABLD
- **Key Fields:** XFDTCD, XFDMAP
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** No

HXLTABLD exposes mapping codes for data codes, likely used by XFXTABL to translate codes into internal/normalized values.

### 2.4 HXLTABLP – Logical Long Description View

- **Logical File Name:** HXLTABLP
- **Based On PF:** HXPTABLD
- **Record Format:** XFFTABLD
- **Key Fields:** XFDTCD, XFDLDS
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** No

HXLTABLP provides long descriptions (LDS) for table entries, likely used in reports and user displays.

### 2.5 HXLTABLS – Logical Short Description View

- **Logical File Name:** HXLTABLS
- **Based On PF:** HXPTABLD
- **Record Format:** XFFTABLD
- **Key Fields:** XFDTCD, XFDSDS
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** No

HXLTABLS exposes short descriptions (SDS) for table entries, suitable for compact print layouts and UI fields.

### 2.6 HXPBNFIT – Logical Benefit Plan View

- **Logical File Name:** HXPBNFIT
- **Based On PF:** TXPBNFIT (missing)
- **Record Format:** XFFBNFIT
- **Key Fields:** XFBUBN, XFBPLN
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** Yes (inherited XFBTEL)

HXPBNFIT is a logical view used by application logic to access benefit plan definitions. It inherits PHI from XFBTEL, which holds phone numbers.

### 2.7 HXPNSTN – Logical Station/Namespace View

- **Logical File Name:** HXPNSTN
- **Based On PF:** TXPNSTN (missing)
- **Record Format:** XFFNSTN
- **Key Fields:** XFNLV6, XFNSST
- **Select/Omit Criteria:** None documented
- **PHI Sensitive Flag:** No

HXPNSTN presents station/namespace information keyed by level and status. No PHI fields are tagged.

## 3. PHI Field Catalogue

For quick reference, the following table consolidates all PHI‑flagged fields across PFs.

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

These fields must be treated as sensitive in any modernization effort. Access controls, audit logging, and masking strategies should be applied around the files and programs that consume or emit them.

## 4. Notes on Missing Base Files

Several logical files in this dictionary reference PFs that are not present in the harvested set:

- TAPIRNK (base for HAPIRNK)
- TMPMAST (base for HMLMAST5H)
- TXPBNFIT (base for HXPBNFIT)
- TXPNSTN (base for HXPNSTN)

Although their logical views provide insight into key structures, the full column definitions, types, and lengths of underlying fields are not available. Any reverse‑engineering or migration must either retrieve these DDS definitions from the source system or reconstruct them from downstream databases and runtime traces.
