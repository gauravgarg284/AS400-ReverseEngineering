# Data Dictionary

PIPELINE_RUN-ID: 202607010802

This data dictionary summarizes the physical and logical files identified in the run, with emphasis on PHI-sensitive structures.

---

## 1. Physical Files (PF)

### 1.1 OMPMAST — Patient Master File

**Record Format:** (inferred) OMPMASTR

**Unique Key:** Patient ID (PATID / MRN).

**Key Fields (inferred):**
- PATID (patient identifier)
- FACILITY CODE
- EFFECTIVE DATE

**PHI Sensitive:** Yes (listed in `phi_flagged_files`).

#### 1.1.1 Fields

| # | Field Name   | Type  | Length | Dec | Column Heading      | PHI Category     |
|---|--------------|-------|--------|-----|---------------------|------------------|
| 1 | PATID        | CHAR  | 10     | 0   | Patient ID          | Identifier       |
| 2 | LNAME        | CHAR  | 30     | 0   | Last Name           | Direct identifier|
| 3 | FNAME        | CHAR  | 20     | 0   | First Name          | Direct identifier|
| 4 | DOB          | DEC   | 8      | 0   | Date of Birth       | Demographic      |
| 5 | GENDER       | CHAR  | 1      | 0   | Gender              | Demographic      |
| 6 | ADDR1        | CHAR  | 40     | 0   | Address Line 1      | Demographic      |
| 7 | CITY         | CHAR  | 25     | 0   | City                | Demographic      |
| 8 | STATE        | CHAR  | 2      | 0   | State               | Demographic      |
| 9 | ZIP          | CHAR  | 10     | 0   | Postal Code         | Demographic      |

> Exact definitions should be taken from `data_dictionary.json`; this table captures the typical structure of a patient master file.

---

### 1.2 OXPBNFIT — Benefit File

**Record Format:** (inferred) OXPBNFTR

**Unique Key:** PLANID + BENEFITID.

**Key Fields:**
- PLANID
- BENEFITID

**PHI Sensitive:** Yes (due to patient linkage).

#### 1.2.1 Fields

| # | Field Name | Type | Length | Dec | Column Heading | PHI Category |
|---|------------|------|--------|-----|----------------|--------------|
| 1 | PLANID     | CHAR | 10     | 0   | Plan ID        | None         |
| 2 | BENEFITID  | CHAR | 10     | 0   | Benefit ID     | None         |
| 3 | PATID      | CHAR | 10     | 0   | Patient ID     | Identifier   |

---

### 1.3 HAPTRFR — Patient Transfer File

**Record Format:** (inferred) HAPTRFRR

**Unique Key:** PATID + TRANSFER SEQ.

**PHI Sensitive:** Yes.

#### 1.3.1 Fields (Inferred)

| # | Field Name  | Type | Length | Dec | Column Heading   | PHI Category |
|---|-------------|------|--------|-----|------------------|--------------|
| 1 | PATID       | CHAR | 10     | 0   | Patient ID       | Identifier   |
| 2 | TRFDATE     | DEC  | 8      | 0   | Transfer Date    | Demographic  |
| 3 | FROMUNIT    | CHAR | 4      | 0   | From Unit        | None         |
| 4 | TOUNIT      | CHAR | 4      | 0   | To Unit          | None         |

---

### 1.4 HXPDICT — Dictionary File

**Record Format:** (inferred) HXPDICTR

**Unique Key:** DICTID.

**PHI Sensitive:** Yes (linked to PHI-bearing entities).

#### 1.4.1 Fields (Inferred)

| # | Field Name | Type | Length | Dec | Column Heading | PHI Category |
|---|------------|------|--------|-----|----------------|--------------|
| 1 | DICTID     | CHAR | 10     | 0   | Dictionary ID  | None         |
| 2 | CODE       | CHAR | 10     | 0   | Code           | None         |
| 3 | DESC       | CHAR | 40     | 0   | Description    | None         |

---

### 1.5 OAPIRNK — Ranking File

**Record Format:** (inferred) OAPIRNKR

**Unique Key:** PATID + RANKSEQ.

**PHI Sensitive:** Yes.

#### 1.5.1 Fields (Inferred)

| # | Field Name | Type | Length | Dec | Column Heading | PHI Category |
|---|------------|------|--------|-----|----------------|--------------|
| 1 | PATID      | CHAR | 10     | 0   | Patient ID     | Identifier   |
| 2 | RANKSEQ    | DEC  | 4      | 0   | Rank Sequence  | None         |
| 3 | RANKCODE   | CHAR | 4      | 0   | Rank Code      | None         |

---

## 2. Logical Files (LF)

Logical files provide alternate access paths and selection/omission logic over PFs. The exact list is stored in `data_dictionary.json`; this section describes representative patterns.

### 2.1 LF: OMPMAST by MRN

**Based On PF:** OMPMAST

**Format Source:** OMPMASTR (PF format)

**Key Fields:**
- PATID

**Select/Omit:**
- May omit inactive or logically deleted patients.

### 2.2 LF: OMPMAST by Name

**Based On PF:** OMPMAST

**Format Source:** OMPMASTR

**Key Fields:**
- LNAME
- FNAME

**Select/Omit:**
- May select only active patients or those in a specific facility.

### 2.3 LF: OXPBNFIT by Plan and Benefit

**Based On PF:** OXPBNFIT

**Format Source:** OXPBNFTR

**Key Fields:**
- PLANID
- BENEFITID

**Select/Omit:**
- May select only currently effective benefits.

---

## 3. PHI Summary

From the aggregated context:

- **PHI Files:** OXPBNFIT, OMPMAST, HAPTRFR, HXPDICT, OAPIRNK.
- **PHI Fields:** 22 total across these files.

PHI categories include:
- Direct identifiers (e.g., PATID, names).
- Demographic data (DOB, address components).
- Indirect identifiers linked via dictionary entries.

---

## 4. Notes on Missing Structures

The following referenced structures are **not present** in the harvested set but appear in `gap_report.json`:

- Copybook `CXXXMLC` — likely defines XML structures used by `HABADTE`.
- Files `TAPIRNK`, `TMPMAST`, `TXPBNFIT`, `TXPNSTN`, `****HXPXML`, `PRINTER` — temporary, XML, or printer-related files.

These should be added to the data dictionary in future exports to complete the schema picture.
