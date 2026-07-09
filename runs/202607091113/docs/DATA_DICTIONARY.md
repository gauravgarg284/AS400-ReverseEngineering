# Data Dictionary

This data dictionary summarizes the physical and logical file structures discovered in the AS/400 codebase. It focuses on record formats, keys, uniqueness, PHI sensitivity, and field-level metadata as provided in the aggregated schema.

## 1. Physical Files (PF)

### 1.1 HAPTRFR

- Record format: **HAFTRFR**
- Unique key: **Yes**
- Key fields: `AFLVL6`, `AFACCT`, `AFTRDT`, `AFTRTM`, `AFTYPE`
- Total fields: **28**
- PHI-sensitive: **Yes**
- PHI fields:
  - `AFACCT` – category: AccountNumber
  - `AFMRNO` – category: MRN

**Field Table**

| Field  | Type* | Length* | Dec* | Column Heading* | PHI Category   |
|--------|-------|---------|------|-----------------|----------------|
| AFACCT | *char | *       | *    | *               | AccountNumber  |
| AFMRNO | *char | *       | *    | *               | MRN            |

> Note: `Type*`, `Length*`, `Dec*`, and `Column Heading*` are placeholders for underlying DDS attributes; they are not explicitly present in the compact schema, but the PHI categories are accurate.

---

### 1.2 HXPDICT

- Record format: **HXFDICT**
- Unique key: **No**
- Key fields: *(none in compact schema)*
- Total fields: **2705**
- PHI-sensitive: **Yes**
- PHI fields (subset from PHI registry):
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

**Field Table (PHI subset)**

| Field  | Type* | Length* | Dec* | Column Heading* | PHI Category  |
|--------|-------|---------|------|-----------------|---------------|
| CCMRNO | *char | *       | *    | *               | MRN           |
| XFBTEL | *char | *       | *    | *               | PhoneNumber   |
| XCNAME | *char | *       | *    | *               | PatientName   |
| HXRMNO | *char | *       | *    | *               | RoomNumber    |
| XFRMNO | *char | *       | *    | *               | RoomNumber    |
| HVACCT | *char | *       | *    | *               | AccountNumber |
| IMGMRN | *char | *       | *    | *               | MRN           |
| HXGMRN | *char | *       | *    | *               | MRN           |
| IHMRNO | *char | *       | *    | *               | MRN           |
| IHACCT | *char | *       | *    | *               | AccountNumber |
| WBDATE | *date | *       | *    | *               | DateOfBirth   |
| XMDMRN | *char | *       | *    | *               | MRN           |
| ENNAME | *char | *       | *    | *               | PatientName   |

---

### 1.3 HXPLVL1

- Record format: **HXFLVL1**
- Unique key: **Yes**
- Key fields: `HX1NUM`
- Total fields: **36**
- PHI-sensitive: **No**

**Field Table (summary)**

| Field  | Type* | Length* | Dec* | Column Heading* | PHI Category |
|--------|-------|---------|------|-----------------|--------------|
| HX1NUM | *     | *       | *    | *               | None         |

---

### 1.4 HXPLVL2

- Record format: **HXFLVL2**
- Unique key: **Yes**
- Key fields: `HX2NUM`
- Total fields: **39**
- PHI-sensitive: **No**

---

### 1.5 HXPLVL3

- Record format: **HXFLVL3**
- Unique key: **Yes**
- Key fields: `HX3NUM`
- Total fields: **39**
- PHI-sensitive: **No**

---

### 1.6 HXPLVL4

- Record format: **HXFLVL4**
- Unique key: **Yes**
- Key fields: `HX4NUM`
- Total fields: **39**
- PHI-sensitive: **No**

---

### 1.7 HXPLVL5

- Record format: **HXFLVL5**
- Unique key: **Yes**
- Key fields: `HX5NUM`
- Total fields: **42**
- PHI-sensitive: **No**

---

### 1.8 HXPLVL6

- Record format: **HXFLVL6**
- Unique key: **Yes**
- Key fields: `HX6NUM`
- Total fields: **155**
- PHI-sensitive: **No**

---

### 1.9 HXPTABLD

- Record format: **XFFTABLD**
- Unique key: **No**
- Key fields: `XFDTCD`, `XFDECD`
- Total fields: **7**
- PHI-sensitive: **No**

**Field Table (summary)**

| Field  | Type* | Length* | Dec* | Column Heading* | PHI Category |
|--------|-------|---------|------|-----------------|--------------|
| XFDTCD | *     | *       | *    | *               | None         |
| XFDECD | *     | *       | *    | *               | None         |

---

### 1.10 HXPXMLD

- Record format: **HXFXMLD**
- Unique key: **Yes**
- Key fields: `XMDUSR`, `XMDSEQ`, `XMDSQ2`
- Total fields: **4**
- PHI-sensitive: **No**

---

### 1.11 HXPXMLR

- Record format: **HXFXMLR**
- Unique key: **Yes**
- Key fields: `XMRUSR`, `XMRSEQ`, `XMRID`
- Total fields: **4**
- PHI-sensitive: **No**

---

### 1.12 OAPIRNK

- Record format: **HBFIRNK**
- Unique key: **Yes**
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Total fields: **33**
- PHI-sensitive: **Yes**
- PHI fields:
  - `BRKMRN` – MRN

---

### 1.13 OMPMAST

- Record format: **HMFMAST**
- Unique key: **Yes**
- Key fields: `MMPLV6`, `MMACCT`
- Total fields: **149**
- PHI-sensitive: **Yes**
- PHI fields:
  - `MMMRNO` – MRN
  - `MMACCT` – AccountNumber
  - `MMNAME` – PatientName
  - `MMPSSN` – SSN
  - `MMMMRN` – MRN

**Field Table (PHI subset)**

| Field  | Type* | Length* | Dec* | Column Heading* | PHI Category  |
|--------|-------|---------|------|-----------------|---------------|
| MMMRNO | *char | *       | *    | *               | MRN           |
| MMACCT | *char | *       | *    | *               | AccountNumber |
| MMNAME | *char | *       | *    | *               | PatientName   |
| MMPSSN | *char | *       | *    | *               | SSN           |
| MMMMRN | *char | *       | *    | *               | MRN           |

---

### 1.14 OXPBNFIT

- Record format: **XFFBNFIT**
- Unique key: **Yes**
- Key fields: `XFBUBN`, `XFBPLN`
- Total fields: **34**
- PHI-sensitive: **Yes**
- PHI fields:
  - `XFBTEL` – PhoneNumber

---

### 1.15 OXPNSTN

- Record format: **XFFNSTN**
- Unique key: **Yes**
- Key fields: `XFNLV6`, `XFNSST`
- Total fields: **23**
- PHI-sensitive: **No**

---

### 1.16 TAPIRNK

- Record format: **ATE TABLE**
- Unique key: **No**
- Key fields: *(not specified)*
- Total fields: **43**
- PHI-sensitive: **No**

---

### 1.17 TMPMAST

- Record format: **ATE TABLE**
- Unique key: **No**
- Key fields: *(not specified)*
- Total fields: **181**
- PHI-sensitive: **No**

---

### 1.18 TXPBNFIT

- Record format: **ATE TABLE**
- Unique key: **No**
- Key fields: *(not specified)*
- Total fields: **12**
- PHI-sensitive: **No**

---

### 1.19 TXPNSTN

- Record format: **ATE TABLE**
- Unique key: **No**
- Key fields: *(not specified)*
- Total fields: **19**
- PHI-sensitive: **No**

---

## 2. Logical Files (LF)

### 2.1 HAPIRNK

- Based on PF: **TAPIRNK**
- Record format: **HBFIRNK**
- Key fields: `BRKLV6`, `BRKACC`, `BRKSEQ`
- Select/omit: **None**

HAPIRNK provides an ordered access path over TAPIRNK by level, account, and sequence.

---

### 2.2 HMLMAST5H

- Based on PF: **TMPMAST**
- Record format: **HMFMAST**
- Key fields: `MMPNST`, `MMADDT`, `MMADTM`
- Select/omit: **None**

HMLMAST5H supports queries over TMPMAST by plan/state and admission date/time.

---

### 2.3 HXLTABLD

- Based on PF: **HXPTABLD**
- Record format: **XFFTABLD**
- Key fields: `XFDTCD`, `XFDMAP`
- Select/omit: **None**

HXLTABLD exposes mappings from data codes (`XFDTCD`) to map codes (`XFDMAP`).

---

### 2.4 HXLTABLP

- Based on PF: **HXPTABLD**
- Record format: **XFFTABLD**
- Key fields: `XFDTCD`, `XFDLDS`
- Select/omit: **None**

HXLTABLP provides a lookup path for data codes and load descriptors.

---

### 2.5 HXLTABLS

- Based on PF: **HXPTABLD**
- Record format: **XFFTABLD**
- Key fields: `XFDTCD`, `XFDSDS`
- Select/omit: **None**

HXLTABLS provides selection/description mappings for data codes.

---

### 2.6 HXPBNFIT

- Based on PF: **TXPBNFIT**
- Record format: **XFFBNFIT**
- Key fields: `XFBUBN`, `XFBPLN`
- Select/omit: **None**

HXPBNFIT presents benefit records keyed by UB number and plan.

---

### 2.7 HXPNSTN

- Based on PF: **TXPNSTN**
- Record format: **XFFNSTN**
- Key fields: `XFNLV6`, `XFNSST`
- Select/omit: **None**

HXPNSTN provides a keyed view of station/state-related records.

---

## 3. PHI Summary

The system contains PHI-related fields in several PFs, summarized below.

| File    | Field   | PHI Category   |
|---------|---------|----------------|
| HAPTRFR | AFACCT  | AccountNumber  |
| HAPTRFR | AFMRNO  | MRN            |
| HXPDICT | CCMRNO  | MRN            |
| HXPDICT | XFBTEL  | PhoneNumber    |
| HXPDICT | XCNAME  | PatientName    |
| HXPDICT | HXRMNO  | RoomNumber     |
| HXPDICT | XFRMNO  | RoomNumber     |
| HXPDICT | HVACCT  | AccountNumber  |
| HXPDICT | IMGMRN  | MRN            |
| HXPDICT | HXGMRN  | MRN            |
| HXPDICT | IHMRNO  | MRN            |
| HXPDICT | IHACCT  | AccountNumber  |
| HXPDICT | WBDATE  | DateOfBirth    |
| HXPDICT | XMDMRN  | MRN            |
| HXPDICT | ENNAME  | PatientName    |
| OAPIRNK | BRKMRN  | MRN            |
| OMPMAST | MMMRNO  | MRN            |
| OMPMAST | MMACCT  | AccountNumber  |
| OMPMAST | MMNAME  | PatientName    |
| OMPMAST | MMPSSN  | SSN            |
| OMPMAST | MMMMRN  | MRN            |
| OXPBNFIT| XFBTEL  | PhoneNumber    |

These PHI designations should guide access control, masking, and audit requirements in any modernized implementation.
