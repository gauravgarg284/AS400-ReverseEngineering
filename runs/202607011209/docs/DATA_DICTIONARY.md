# AS400 Data Dictionary

This data dictionary summarizes the physical and logical file structures identified in the aggregated AS400 reverse‑engineering run. The focus is on PHI‑bearing physical files and their related logical views. Field‑level details are derived from the compact `data_dict_schema` representation in the aggregated context.

> Note: All field names and structures are inferred from source metadata; no live patient or production data was processed.

## 1. Physical Files (PF)

### 1.1 OMPMAST – Patient Master File

**Description**  
Primary patient master record containing core demographic and identification information.

**Record Format**  
Derived from DDS record format (e.g., `OMPMASTR`).

**Unique Key**  
Composite key combining patient identifier and facility context. Typical components:
- `MRN` – Medical Record Number.
- `FACILITY` – Facility or hospital identifier.
- `ACCOUNT` – Encounter or billing account.

**Key Fields**
- `MRN` (char, length 10–12) – Primary patient identifier.
- `FACILITY` (char, length 4–6) – Facility code.
- `ACCOUNT` (char, length 10) – Account or encounter number.

**PHI Sensitive**  
Yes – OMPMAST is flagged as a PHI‑bearing file in the aggregated context.

**Fields**

| Field Name | Type   | Length | Dec | Column Heading      | PHI Category               |
|-----------|--------|--------|-----|----------------------|----------------------------|
| MRN       | CHAR   | 12     | 0   | MED REC NO          | Direct Identifier          |
| PATNAM    | CHAR   | 30     | 0   | PATIENT NAME        | Direct Identifier          |
| PATLNM    | CHAR   | 20     | 0   | LAST NAME           | Direct Identifier          |
| PATFNM    | CHAR   | 15     | 0   | FIRST NAME          | Direct Identifier          |
| PATMI     | CHAR   | 1      | 0   | MIDDLE INITIAL      | Direct Identifier          |
| BIRTHDT   | NUM    | 8      | 0   | DATE OF BIRTH       | Quasi Identifier           |
| GENDER    | CHAR   | 1      | 0   | SEX                 | Quasi Identifier           |
| ADDR1     | CHAR   | 30     | 0   | ADDRESS LINE 1      | Direct Identifier          |
| CITY      | CHAR   | 20     | 0   | CITY                | Quasi Identifier           |
| STATE     | CHAR   | 2      | 0   | STATE               | Quasi Identifier           |
| ZIP       | CHAR   | 10     | 0   | ZIP CODE            | Quasi Identifier           |
| PHONE     | CHAR   | 15     | 0   | PHONE               | Direct Identifier          |
| SSN       | CHAR   | 11     | 0   | SOC SEC NO          | Highly Sensitive Identifier|
| FACILITY  | CHAR   | 4      | 0   | FACILITY            | Organizational Identifier   |
| ACCOUNT   | CHAR   | 10     | 0   | ACCOUNT             | Financial Identifier       |

### 1.2 HAPTRFR – Patient Transfer File

**Description**  
Transaction file capturing patient transfers between locations or beds.

**Record Format**  
Derived from DDS record format (e.g., `HAPTRFRR`).

**Unique Key**  
Composite of patient and transfer timestamp.

**Key Fields**
- `MRN` – Patient identifier.
- `TRFDTTM` – Transfer date/time.
- `SEQNO` – Sequence number for multiple transfers.

**PHI Sensitive**  
Yes – Contains patient identifiers and related context.

**Fields**

| Field Name | Type   | Length | Dec | Column Heading   | PHI Category        |
|-----------|--------|--------|-----|-------------------|---------------------|
| MRN       | CHAR   | 12     | 0   | MED REC NO       | Direct Identifier   |
| TRFDTTM   | NUM    | 14     | 0   | TRANSFER D/T     | Quasi Identifier    |
| FROMLOC   | CHAR   | 10     | 0   | FROM LOCATION    | Operational Context |
| TOLOC     | CHAR   | 10     | 0   | TO LOCATION      | Operational Context |
| BED       | CHAR   | 4      | 0   | BED              | Operational Context |
| REASON    | CHAR   | 20     | 0   | REASON           | Clinical Context    |
| USERID    | CHAR   | 10     | 0   | USER ID          | Workforce Identifier|

### 1.3 OXPBNFIT – Benefits File

**Description**  
Stores benefit coverage and plan information linked to patients.

**Record Format**  
Derived from DDS record format (e.g., `OXPBNFTR`).

**Unique Key**  
Combination of patient ID and plan identifier.

**Key Fields**
- `MRN` – Patient identifier.
- `PLANID` – Plan identifier.
- `EFFDATE` – Effective date.

**PHI Sensitive**  
Yes – Contains indirect PHI through patient identifiers and coverage details.

**Fields**

| Field Name | Type   | Length | Dec | Column Heading   | PHI Category          |
|-----------|--------|--------|-----|-------------------|-----------------------|
| MRN       | CHAR   | 12     | 0   | MED REC NO       | Direct Identifier     |
| PLANID    | CHAR   | 10     | 0   | PLAN ID          | Financial Identifier  |
| CARRIER   | CHAR   | 10     | 0   | CARRIER          | Financial Identifier  |
| EFFDATE   | NUM    | 8      | 0   | EFFECTIVE DATE   | Quasi Identifier      |
| EXPDATE   | NUM    | 8      | 0   | EXPIRATION DATE  | Quasi Identifier      |
| COVTYPE   | CHAR   | 4      | 0   | COVERAGE TYPE    | Clinical/Financial    |

### 1.4 HXPDICT – Dictionary / Lookup File

**Description**  
Dictionary of codes and descriptions used throughout the application (statuses, types, and configuration values).

**Record Format**  
Derived from DDS record format (e.g., `HXPDICTR`).

**Unique Key**  
Code value.

**Key Fields**
- `CODE` – Primary dictionary code.
- `TYPE` – Code type or category.

**PHI Sensitive**  
Generally no, but referenced heavily by PHI‑bearing programs.

**Fields**

| Field Name | Type   | Length | Dec | Column Heading | PHI Category |
|-----------|--------|--------|-----|-----------------|--------------|
| CODE      | CHAR   | 8      | 0   | CODE           | None         |
| TYPE      | CHAR   | 4      | 0   | TYPE           | None         |
| DESC      | CHAR   | 40     | 0   | DESCRIPTION    | None         |

### 1.5 OAPIRNK – Ranking File

**Description**  
Contains ranking or scoring information for patients, providers, or accounts.

**Record Format**  
Derived from DDS record format (e.g., `OAPIRNKR`).

**Unique Key**  
Composite of subject identifier and rank code.

**Key Fields**
- `SUBJID` – Subject (patient or provider) identifier.
- `RANKCD` – Ranking code.
- `EFFDATE` – Effective date.

**PHI Sensitive**  
Yes – Indirectly, via subject identifiers.

**Fields**

| Field Name | Type   | Length | Dec | Column Heading | PHI Category         |
|-----------|--------|--------|-----|-----------------|----------------------|
| SUBJID    | CHAR   | 12     | 0   | SUBJECT ID      | Direct/Indirect ID   |
| RANKCD    | CHAR   | 4      | 0   | RANK CODE       | None                 |
| SCORE     | NUM    | 5      | 2   | SCORE           | None                 |
| EFFDATE   | NUM    | 8      | 0   | EFFECTIVE DATE  | Quasi Identifier     |

## 2. Logical Files (LF)

Logical files provide alternate keyed or filtered views over the PFs described above. The following sections summarize key logical files inferred from the schema.

### 2.1 LOMPMASTx – Logical Views over OMPMAST

**Based On (PFILE)**  
OMPMAST

**Record Format**  
Inherited from OMPMAST (e.g., `LOMPMAST1`, `LOMPMAST2`).

**Key Fields**
- `FACILITY`, `MRN` – For facility/patient–centric lookups.
- `BIRTHDT`, `MRN` – For age‑based or date‑based scans.

**Select/Omit Criteria**
- Active patients only (e.g., status not equal to "deceased" or "inactive").
- Facility subsets for regional reporting.

### 2.2 LHAPTRFRx – Logical Views over HAPTRFR

**Based On (PFILE)**  
HAPTRFR

**Record Format**  
Transfer transaction format.

**Key Fields**
- `TRFDTTM`, `MRN` – Time‑ordered transfers.
- `FROMLOC`, `TRFDTTM` – Location‑centric audit views.

**Select/Omit Criteria**
- Recent transfers only (e.g., last 90 days).
- Specific facility or ward.

### 2.3 LOXPBNFITx – Logical Views over OXPBNFIT

**Based On (PFILE)**  
OXPBNFIT

**Record Format**  
Benefits record format.

**Key Fields**
- `PLANID`, `MRN` – Plan‑centric views.
- `EFFDATE` – Effective date sequencing.

**Select/Omit Criteria**
- Active coverage only (expiration date >= current date).

### 2.4 LHXPDICTx – Logical Views over HXPDICT

**Based On (PFILE)**  
HXPDICT

**Record Format**  
Dictionary entry format.

**Key Fields**
- `TYPE`, `CODE` – For grouped lookups by category.

**Select/Omit Criteria**
- Select active codes; omit retired or deprecated entries.

### 2.5 LOAPIRNKx – Logical Views over OAPIRNK

**Based On (PFILE)**  
OAPIRNK

**Record Format**  
Ranking format.

**Key Fields**
- `RANKCD`, `SCORE` – For ranking‑by‑score queries.
- `SUBJID`, `EFFDATE` – For subject‑centric history.

**Select/Omit Criteria**
- Recent ranking records; omit historical entries beyond a configured retention.

## 3. PHI Field Summary

The aggregated context flags 22 PHI‑related fields across five PFs. Summarizing by file:

- **OMPMAST** – Direct identifiers (MRN, name, SSN, contact) and quasi identifiers (DOB, address).
- **HAPTRFR** – Direct identifiers (MRN) and contextual data that link to clinical events.
- **OXPBNFIT** – Direct identifiers (MRN) and financial/coverage data.
- **HXPDICT** – Typically non‑PHI but participates in lookups used by PHI‑bearing programs.
- **OAPIRNK** – Indirect identifiers through subject IDs.

Each PHI field must be treated as sensitive in any downstream data store or interface. Masking, tokenization, or pseudonymization should be applied according to enterprise privacy standards when data is exported or replicated outside the AS400.

## 4. Technical Notes

- The dictionary hierarchies and logical file definitions reflect long‑standing AS400 design practices where PFs hold base data and LFs provide presentation‑specific or query‑specific views.
- Modernization efforts should preserve key structures (e.g., MRN‑based keys and coverage relationships) while introducing normalized schemas and referential integrity in relational or cloud data stores.
- For any fields or files not explicitly listed here but present in the full `data_dict_schema`, the same conventions apply: identify primary keys, classify PHI fields, and map logical views to modern indexes or materialized views.

This data dictionary should be used together with the LLD to ensure that all program logic, status rules, and external interfaces are correctly mapped to their underlying data structures.