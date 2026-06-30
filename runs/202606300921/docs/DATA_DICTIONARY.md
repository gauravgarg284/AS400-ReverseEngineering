# Data Dictionary — HABADTE

## 1. Physical Files (PF)

### 1.1 HAPTRFR

- Record Format: HAPTRFRR
- Unique Key: AFACCT + AFMRNO + transfer sequence.
- Key Fields: AFACCT, AFMRNO, transfer date/time.
- PHI Sensitive: Yes.

#### 1.1.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| AFACCT | CHAR | 10     | 0   | Account        | Account Identifier |
| AFMRNO | CHAR | 10     | 0   | MRN            | Medical Record Number |
| ...    | ...  | ...    | ... | ...            | ... |

---

### 1.2 OMPMAST

- Record Format: OMPMASTR
- Unique Key: MMMRNO or MMACCT.
- Key Fields: MMMRNO, MMACCT.
- PHI Sensitive: Yes.

#### 1.2.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| MMMRNO | CHAR | 10     | 0   | MRN            | Medical Record Number |
| MMACCT | CHAR | 10     | 0   | Account        | Account Identifier |
| MMNAME | CHAR | 30     | 0   | Name           | Patient Name |
| MMPSSN | CHAR | 11     | 0   | SSN            | National ID / SSN |
| ...    | ...  | ...    | ... | ...            | ... |

---

### 1.3 OXPBNFIT

- Record Format: OXPBNFITR
- Unique Key: Benefit code.
- PHI Sensitive: Yes (XFBTEL).

#### 1.3.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| XFBTEL | CHAR | 15     | 0   | Phone          | Contact Phone |
| ...    | ...  | ...    | ... | ...            | ... |

---

### 1.4 OXPNSTN

- Record Format: OXPNSTNR
- Unique Key: Institution code.
- PHI Sensitive: No.

#### 1.4.1 Fields

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| INSTCD  | CHAR | 6      | 0   | Institution    | None |
| INSTNAM | CHAR | 40     | 0   | Name           | None |
| ...     | ...  | ...    | ... | ...            | ... |

---

### 1.5 HXPDICT

- Record Format: HXPDICTR
- Unique Key: CCMRNO/HXRMNO.
- PHI Sensitive: Yes.

#### 1.5.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| CCMRNO | CHAR | 10     | 0   | Cross MRN      | Medical Record Number |
| HXRMNO | CHAR | 10     | 0   | HABMRN         | Medical Record Number |
| XCNAME | CHAR | 30     | 0   | Name           | Patient Name |
| XFBTEL | CHAR | 15     | 0   | Phone          | Contact Phone |
| ...    | ...  | ...    | ... | ...            | ... |

---

### 1.6 OAPIRNK

- Record Format: OAPIRNKR
- Unique Key: Account + rank.
- PHI Sensitive: Yes.

#### 1.6.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| AFACCT | CHAR | 10     | 0   | Account        | Account Identifier |
| AFMRNO | CHAR | 10     | 0   | MRN            | Medical Record Number |
| ...    | ...  | ...    | ... | ...            | ... |

---

### 1.7 HXPTABLD

- Record Format: HXPTABLDR
- Unique Key: TBLID + KEYVAL.
- PHI Sensitive: No.

#### 1.7.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| TBLID  | CHAR | 8      | 0   | Table ID       | None |
| KEYVAL | CHAR | 10     | 0   | Key Value      | None |
| DESC   | CHAR | 40     | 0   | Description    | None |

---

### 1.8 HXPXMLD

- Record Format: HXPXMLDR
- Unique Key: XMLID.
- PHI Sensitive: Possibly (depending on payload).

#### 1.8.1 Fields

| Field  | Type | Length | Dec | Column Heading | PHI Category |
|--------|------|--------|-----|----------------|-------------|
| XMLID  | CHAR | 20     | 0   | XML ID         | None |
| XMLDEF | CLOB | 32767  | 0   | Definition     | May Contain PHI |

---

### 1.9 HXPXMLR

- Record Format: HXPXMLRR
- Unique Key: XMLID + timestamp.
- PHI Sensitive: Yes (runtime payload).

#### 1.9.1 Fields

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| XMLID   | CHAR | 20     | 0   | XML ID         | None |
| XMLTEXT | CLOB | 32767  | 0   | XML Text       | May Contain PHI |

---

### 1.10 HXPLVL1–HXPLVL6

- Record Format: HXPLVLR
- Unique Key: LVLKEY.
- PHI Sensitive: No.

#### 1.10.1 Fields

| Field   | Type | Length | Dec | Column Heading | PHI Category |
|---------|------|--------|-----|----------------|-------------|
| LVLKEY  | CHAR | 10     | 0   | Level Key      | None |
| LVLNAME | CHAR | 40     | 0   | Level Name     | None |

---

## 2. Logical Files (LF)

### 2.1 HAPIRNK

- Based On PF: OAPIRNK / TAPIRNK.
- Format Source: OAPIRNK.
- Key Fields: Rank, AFACCT.
- Select/Omit: Likely selects active ranks; may omit inactive or void entries.

### 2.2 HMLMAST5H

- Based On PF: TMPMAST (missing).
- Format Source: TMPMAST.
- Key Fields: MMMRNO, MMACCT.
- Select/Omit: HABADTE-specific subset (e.g., only inpatients or certain account types).

### 2.3 HXPBNFIT

- Based On PF: OXPBNFIT / TXPBNFIT.
- Format Source: OXPBNFIT.
- Key Fields: Benefit code.
- Select/Omit: Active benefits; may omit expired or inactive ones.

### 2.4 HXPNSTN

- Based On PF: OXPNSTN / TXPNSTN.
- Format Source: OXPNSTN.
- Key Fields: Institution code.

### 2.5 HXLTABLS

- Based On PF: HXPTABLD.
- Format Source: HXPTABLD.
- Key Fields: TBLID, KEYVAL.
- Select/Omit: Status-based selection (e.g., only active entries).

### 2.6 HXLTABLP

- Based On PF: HXPTABLD.
- Format Source: HXPTABLD.
- Key Fields: TBLID, KEYVAL, level.
- Select/Omit: Preference-level specific entries.

### 2.7 HXLTABLD

- Based On PF: HXPTABLD.
- Format Source: HXPTABLD.
- Key Fields: TBLID, KEYVAL.
- Select/Omit: Alternate key sequence or filter; may be used for dictionary-style lookups.

---

## 3. PHI Sensitivity Summary

| File     | PHI Fields                          |
|----------|-------------------------------------|
| HAPTRFR  | AFACCT, AFMRNO                      |
| OXPBNFIT | XFBTEL                              |
| OMPMAST  | MMMRNO, MMACCT, MMNAME, MMPSSN      |
| HXPDICT  | CCMRNO, XFBTEL, XCNAME, HXRMNO      |
| OAPIRNK  | AFACCT, AFMRNO                      |

These fields must be handled with strict access control, encryption/masking, and auditing in any migrated solution.
