# Business Processing Flowchart – HABADTE Project

Run ID: 202607011209

This document provides visual flowcharts for the HABADTE inpatient transfer XML export process and related Data Maintenance utilities.

## 1. Top-Level Processing Flow

```mermaid
flowchart TD
    START([START]) --> INP[input receipt]
    INP --> CNTINIT[counter init]
    CNTINIT --> PREFLOAD[preference loads]
    PREFLOAD --> HDRRES[header resolution]
    HDRRES --> MAINLOOP[main query loop]
    MAINLOOP --> CNTACC[counter accumulation]
    CNTACC --> RESPASM[response assembly]
    RESPASM --> END([END])
```

## 2. Record Filter Gate

```mermaid
flowchart TD
    START([Evaluate Transfer Record]) --> BR001{BR-001: X equals 0?}
    BR001 -- Yes EXCLUDE --> EXIT1[Exit loop]
    BR001 -- No INCLUDE --> BR002{BR-002: X equals 40?}
    BR002 -- Yes EXCLUDE --> EXIT2[Exit loop]
    BR002 -- No INCLUDE --> BR003{BR-003: VYY < 1800?}
    BR003 -- Yes EXCLUDE --> SKIP003[Skip record]
    BR003 -- No INCLUDE --> BR004{BR-004: VYY > 2100?}
    BR004 -- Yes EXCLUDE --> SKIP004[Skip record]
    BR004 -- No INCLUDE --> BR005{BR-005: VMM < 01?}
    BR005 -- Yes EXCLUDE --> SKIP005[Skip record]
    BR005 -- No INCLUDE --> BR006{BR-006: VMM > 12?}
    BR006 -- Yes EXCLUDE --> SKIP006[Skip record]
    BR006 -- No INCLUDE --> BR007{BR-007: VDD < 01?}
    BR007 -- Yes EXCLUDE --> SKIP007[Skip record]
    BR007 -- No INCLUDE --> BR008{BR-008: VDD > DYS VMM?}
    BR008 -- Yes EXCLUDE --> SKIP008[Skip record]
    BR008 -- No INCLUDE --> BR018{BR-018: FLAG equals void or voided?}
    BR018 -- Yes EXCLUDE --> SKIP018[Skip record]
    BR018 -- No INCLUDE --> BR019{BR-019: INPATIENT OUTPATIENT equals outpatient?}
    BR019 -- Yes EXCLUDE --> SKIP019[Skip record]
    BR019 -- No INCLUDE --> PASS[Record accepted]
```

## 3. Data Enrichment Flow

```mermaid
flowchart TD
    subgraph HABADTE_Enrichment
        HABREC[HABADTE transfer record] --> STNLOOK[Station lookup via HXPNSTN]
        HABREC --> BENLOOK[Benefit plan lookup via HXPBNFIT]
        HABREC --> RANKLOOK[Rank lookup via HAPIRNK OAPIRNK]
        HABREC --> PMLOOK[Patient master lookup via OMPMAST]
    end

    subgraph Data_Maintenance_Utilities
        CYMD[XFXCYMD date validation]
        CNTR[XFXCNTR counter control]
        LDSC[XFXLDSC level mapping]
        TABL[XFXTABL table configuration]
    end

    STNLOOK --> CYMD
    BENLOOK --> TABL
    RANKLOOK --> LDSC
    PMLOOK --> CNTR
```

## 4. Counter and Aggregation Logic

```mermaid
flowchart TD
    STARTCNT([Init counters]) --> APPLY017[Apply BR-017 file indicator check]
    APPLY017 --> BR017{MMPFIL equals 0?}
    BR017 -- Yes --> SKIPFILE[Skip account file]
    BR017 -- No --> LOOP[Per transfer loop]

    LOOP --> BR018{MMPFLG equals V void?}
    BR018 -- Yes --> SKIPVOID[Increment skippedVoided and continue]
    BR018 -- No --> BR019{MMIORO equals O outpatient?}
    BR019 -- Yes --> SKIPOP[Increment skippedOutpatient and continue]
    BR019 -- No --> CNTUPD[Increment totalTransfers]

    CNTUPD --> NEXTREC[Next transfer]
    NEXTREC --> LOOP
```

## 5. Application Preference Lookup Flow

```mermaid
flowchart TD
    PREFSTART([Preference lookup request]) --> PREFQ[Query preference tables via HXPTABLD and HXLTABLD]
    PREFQ --> PREFFOUND{Preference row found?}
    PREFFOUND -- Yes --> PREFAPPLY[Apply preference values to HABADTE context]
    PREFFOUND -- No --> PREFDEFAULT[Use default configuration]

    PREFAPPLY --> PREFEND([Preferences applied])
    PREFDEFAULT --> PREFEND
```

## 6. Org and Hierarchy Level Lookup Flow

```mermaid
flowchart TD
    ORGSTART([Station level code HX6NUM]) --> L1[Lookup HXPLVL1 facility]
    L1 --> L2[Lookup HXPLVL2 campus]
    L2 --> L3[Lookup HXPLVL3 building]
    L3 --> L4[Lookup HXPLVL4 unit]
    L4 --> L5[Lookup HXPLVL5 ward]
    L5 --> L6[Lookup HXPLVL6 station room]
    L6 --> ORGEND[Resolved facility unit station hierarchy]
```

## 7. End to End Summary Flow

```mermaid
flowchart LR
    subgraph Input
        INREQ[Inpatient transfer request]
    end

    subgraph "Setup Phase"
        SET1[Validate dates via XFXCYMD]
        SET2[Load preferences and tables via XFXTABL]
    end

    subgraph "Data Query"
        Q1[Read transfers from HAPTRFR]
    end

    subgraph "Per Record Enrichment Loop"
        E1[Apply BR-018 void filter]
        E2[Apply BR-019 outpatient filter]
        E3[Station enrichment via HXPNSTN]
        E4[Benefit enrichment via HXPBNFIT]
        E5[Rank and master enrichment via HAPIRNK and OMPMAST]
    end

    subgraph Counting
        C1[Update totalTransfers]
        C2[Update skippedVoided]
        C3[Update skippedOutpatient]
        C4[Update skippedByFileFlag]
    end

    subgraph Response
        R1[Build XML HXFXMLH and HXFXMLD]
        R2[Return inpatient transfer summary]
    end

    INREQ --> SET1
    SET1 --> SET2
    SET2 --> Q1
    Q1 --> E1
    E1 --> E2
    E2 --> E3
    E3 --> E4
    E4 --> E5
    E5 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> C4
    C4 --> R1
    R1 --> R2
```
