# Data Dictionary

PIPELINE_RUN-ID: 202607011449

This data dictionary summarizes the physical and logical file structures identified in the reverse-engineered AS400 codebase. It focuses on metadata available from the aggregated context: record formats, key structures, field counts, and PHI classifications.

## 1. Physical Files (PF)

Each subsection describes a physical file, its key structure, and PHI sensitivity. Field-level detail is constrained to available metadata. Where specific field attributes (length, type, decimals, column headings) are not present in the aggregated context, they are omitted rather than inferred.

### 1.1 HAPTRFR

- Record format: **HAFTRFR**
- Unique key: **Yes**
- Key fields: **AFLVL6, AFACCT, AFTRDT, AFTRTM, AFTYPE**
- Total fields: **28**
- PHI-sensitive: **Yes**
- PHI fields:
  - AFACCT (AccountNumber)
  - AFMRNO (MRN)

HAPTRFR holds transfer records keyed by a level identifier, account, date, time, and transfer type. The presence of account and MRN fields classifies this file as PHI-bearing.

### 1.2 HXPDICT

- Record format: **HXFDICT**
- Unique key: **No**
- Key fields: *(not specified)*
- Total fields: **2705**
- PHI-sensitive: **Yes**
- PHI fields:
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

HXPDICT is a large dictionary PF carrying multiple PHI attributes. It likely stores dictionary-style records for patients, accounts, rooms, or events.

### 1.3 HXPLVL1

- Record format: **HXFLVL1**
- Unique key: **Yes**
- Key fields: **HX1NUM**
- Total fields: **36**
- PHI-sensitive: **No**

HXPLVL1 is the first level in a hierarchical level structure. No PHI fields are reported.

### 1.4 HXPLVL2

- Record format: **HXFLVL2**
- Unique key: **Yes**
- Key fields: **HX2NUM**
- Total fields: **39**
- PHI-sensitive: **No**

HXPLVL2 extends the level hierarchy with additional attributes.

### 1.5 HXPLVL3

- Record format: **HXFLVL3**
- Unique key: **Yes**
- Key fields: **HX3NUM**
- Total fields: **39**
- PHI-sensitive: **No**

HXPLVL3 continues the level model. No PHI fields are flagged.

### 1.6 HXPLVL4

- Record format: **HXFLVL4**
- Unique key: **Yes**
- Key fields: **HX4NUM**
- Total fields: **39**
- PHI-sensitive: **No**

HXPLVL4 adds further level attributes.

### 1.7 HXPLVL5

- Record format: **HXFLVL5**
- Unique key: **Yes**
- Key fields: **HX5NUM**
- Total fields: **42**
- PHI-sensitive: **No**

HXPLVL5 extends the structure with additional fields.

### 1.8 HXPLVL6

- Record format: **HXFLVL6**
- Unique key: **Yes**
- Key fields: **HX6NUM**
- Total fields: **155**
- PHI-sensitive: **No**

HXPLVL6 is the most complex level PF, containing 155 fields. It is heavily used by the XFXLDSC program.

### 1.9 HXPTABLD

- Record format: **XFFTABLD**
- Unique key: **No**
- Key fields: **XFDTCD, XFDECD**
- Total fields: **7**
- PHI-sensitive: **No**

HXPTABLD is a generic table PF representing code-to-description or mapping relationships.

### 1.10 HXPXMLD

- Record format: **HXFXMLD**
- Unique key: **Yes**
- Key fields: **XMDUSR, XMDSEQ, XMDSQ2**
- Total fields: **4**
- PHI-sensitive: **No**

HXPXMLD stores XML detail records keyed by user and sequence identifiers.

### 1.11 HXPXMLR

- Record format: **HXFXMLR**
- Unique key: **Yes**
- Key fields: **XMRUSR, XMRSEQ, XMRID**
- Total fields: **4**
- PHI-sensitive: **No**

HXPXMLR holds XML header or request records.

### 1.12 OAPIRNK

- Record format: **HBFIRNK**
- Unique key: **Yes**
- Key fields: **BRKLV6, BRKACC, BRKSEQ**
- Total fields: **33**
- PHI-sensitive: **Yes**
- PHI fields:
  - BRKMRN (MRN)

OAPIRNK contains break or ranking records associated with MRNs and accounts.

### 1.13 OMPMAST

- Record format: **HMFMAST**
- Unique key: **Yes**
- Key fields: **MMPLV6, MMACCT**
- Total fields: **149**
- PHI-sensitive: **Yes**
- PHI fields:
  - MMMRNO (MRN)
  - MMACCT (AccountNumber)
  - MMNAME (PatientName)
  - MMPSSN (SSN)
  - MMMMRN (MRN)

OMPMAST is a master file for patients or accounts, storing multiple identifiers and demographic attributes.

### 1.14 OXPBNFIT

- Record format: **XFFBNFIT**
- Unique key: **Yes**
- Key fields: **XFBUBN, XFBPLN**
- Total fields: **34**
- PHI-sensitive: **Yes**
- PHI fields:
  - XFBTEL (PhoneNumber)

OXPBNFIT stores benefit-related data, keyed by benefit number and plan.

### 1.15 OXPNSTN

- Record format: **XFFNSTN**
- Unique key: **Yes**
- Key fields: **XFNLV6, XFNSST**
- Total fields: **23**
- PHI-sensitive: **No**

OXPNSTN is a station-level reference PF.

## 2. Logical Files (LF)

Logical files are views over PFs. They present alternate key structures and, in some cases, selection/omission criteria.

### 2.1 HAPIRNK

- Logical file name: **HAPIRNK**
- Based on PF: **TAPIRNK** (missing)
- Record format: **HBFIRNK**
- Key fields: **BRKLV6, BRKACC, BRKSEQ**
- Select/omit description: *(none specified)*
- PHI-sensitive: **Yes (inherited)**

HAPIRNK inherits MRN-related PHI from its underlying PF and shares its key structure.

### 2.2 HMLMAST5H

- Logical file name: **HMLMAST5H**
- Based on PF: **TMPMAST** (missing)
- Record format: **HMFMAST**
- Key fields: **MMPNST, MMADDT, MMADTM**
- Select/omit description: *(none specified)*
- PHI-sensitive: **Yes (inferred from HMFMAST)**

HMLMAST5H provides a chronological index over master records using posting date and time. PHI is inherited from HMFMAST.

### 2.3 HXLTABLD

- Logical file name: **HXLTABLD**
- Based on PF: **HXPTABLD**
- Record format: **XFFTABLD**
- Key fields: **XFDTCD, XFDMAP**
- Select/omit description: *(none specified)*
- PHI-sensitive: **No**

HXLTABLD presents an alternate key for HXPTABLD emphasizing mapping.

### 2.4 HXLTABLP

- Logical file name: **HXLTABLP**
- Based on PF: **HXPTABLD**
- Record format: **XFFTABLD**
- Key fields: **XFDTCD, XFDLDS**
- Select/omit description: *(none specified)*
- PHI-sensitive: **No**

HXLTABLP highlights a “loads” or list field, offering another logical perspective on HXPTABLD.

### 2.5 HXLTABLS

- Logical file name: **HXLTABLS**
- Based on PF: **HXPTABLD**
- Record format: **XFFTABLD**
- Key fields: **XFDTCD, XFDSDS**
- Select/omit description: *(none specified)*
- PHI-sensitive: **No**

HXLTABLS provides a subset-focused view over HXPTABLD.

### 2.6 HXPBNFIT (LF)

- Logical file name: **HXPBNFIT**
- Based on PF: **TXPBNFIT** (missing)
- Record format: **XFFBNFIT**
- Key fields: **XFBUBN, XFBPLN**
- Select/omit description: *(none specified)*
- PHI-sensitive: **Yes (inferred from XFFBNFIT)**

HXPBNFIT overlays a missing benefits PF. PHI is present via phone number fields.

### 2.7 HXPNSTN

- Logical file name: **HXPNSTN**
- Based on PF: **TXPNSTN** (missing)
- Record format: **XFFNSTN**
- Key fields: **XFNLV6, XFNSST**
- Select/omit description: *(none specified)*
- PHI-sensitive: **No**

HXPNSTN overlays station-level data with an alternate key.

## 3. PHI Field Summary

The following fields are explicitly flagged as PHI across the schema:

- **AccountNumber**:
  - HAPTRFR.AFACCT
  - HXPDICT.HVACCT
  - HXPDICT.IHACCT
  - OMPMAST.MMACCT

- **MRN**:
  - HAPTRFR.AFMRNO
  - HXPDICT.CCMRNO
  - HXPDICT.IMGMRN
  - HXPDICT.HXGMRN
  - HXPDICT.IHMRNO
  - HXPDICT.XMDMRN
  - OAPIRNK.BRKMRN
  - OMPMAST.MMMRNO
  - OMPMAST.MMMMRN

- **PatientName**:
  - HXPDICT.XCNAME
  - HXPDICT.ENNAME
  - OMPMAST.MMNAME

- **PhoneNumber**:
  - HXPDICT.XFBTEL
  - OXPBNFIT.XFBTEL

- **RoomNumber**:
  - HXPDICT.HXRMNO
  - HXPDICT.XFRMNO

- **DateOfBirth**:
  - HXPDICT.WBDATE

- **SSN**:
  - OMPMAST.MMPSSN

These fields require special consideration for masking, encryption, and access control in any modernization or reporting effort.

## 4. Orphan and Missing Base Files

Several LFs rely on base PFs that were not recovered:

- TAPIRNK (PF) – base for HAPIRNK
- TMPMAST (PF) – base for HMLMAST5H
- TXPBNFIT (PF) – base for HXPBNFIT
- TXPNSTN (PF) – base for HXPNSTN

In each case, the logical file record format is known, but detailed field definitions and constraints from the PF are missing. These base PFs must be obtained from the original system or reconstructed from backups to complete the data dictionary.

## 5. Usage Overview

- HAPTRFR, OAPIRNK, OMPMAST, OXPBNFIT, and OXPNSTN form the core operational PF set.
- HXPDICT acts as a wide, shared dictionary feeding multiple programs.
- HXPLVL1–HXPLVL6 and HXPTABLD, together with their logical overlays, provide configuration and classification data.
- HXPXMLD and HXPXMLR store XML headers and details for integrations.

This data dictionary provides a consolidated view of the AS400 schema as reconstructed from the supplied source metadata, enabling downstream agents and modernization teams to reason about data flows and PHI handling without direct access to the original DDS source.
