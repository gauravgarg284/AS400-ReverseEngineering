# Business Processing Flowchart

This document describes the end to end business processing for the HABADTE inpatient transfer extract and its supporting DATA_MAINTENANCE utilities, using Mermaid flowcharts embedded inline.

## 1. Top Level Processing Flow

```mermaid
flowchart TD
    %% Top Level Processing Flow
    START([Start])
    INP_RECEIPT[Input receipt]
    CNTR_INIT[Counter init]
    PREF_LOADS[Preference loads]
    HDR_RES[Header resolution]
    MAIN_QUERY[Main query loop]
    CNT_ACCUM[Counter accumulation]
    RESP_ASM[Response assembly]
    END([End])

    START --> INP_RECEIPT
    INP_RECEIPT --> CNTR_INIT
    CNTR_INIT --> PREF_LOADS
    PREF_LOADS --> HDR_RES
    HDR_RES --> MAIN_QUERY
    MAIN_QUERY --> CNT_ACCUM
    CNT_ACCUM --> RESP_ASM
    RESP_ASM --> END
```

## 2. Record Filter Gate

```mermaid
flowchart TD
    %% Record Filter Gate using filter type rules
    START([Record in scope])

    D_BR002{BR 002 X equals 40}
    D_BR003{BR 003 VYY lt 1800}
    D_BR004{BR 004 VYY gt 2100}
    D_BR005{BR 005 VMM lt 01}
    D_BR006{BR 006 VMM gt 12}
    D_BR007{BR 007 VDD lt 01}
    D_BR008{BR 008 VDD gt DYS VMM}
    D_BR009{BR 009 LDAMAP gt 99}
    D_BR010{BR 010 LDAMAP gt 99}
    D_BR011{BR 011 LDAMAP gt 99}
    D_BR012{BR 012 LDAMAP gt 9999}
    D_BR013{BR 013 IN79 on}
    D_BR014{BR 014 IN79 on}
    D_BR015{BR 015 IN79 on}
    D_BR016{BR 016 IN79 on}
    D_BR018{BR 018 FLAG void}
    D_BR019{BR 019 IO flag outpatient}

    EXCLUDE([Exclude record])
    INCLUDE([Include record])

    START --> D_BR002
    D_BR002 -->|true| EXCLUDE
    D_BR002 -->|false| D_BR003

    D_BR003 -->|true| EXCLUDE
    D_BR003 -->|false| D_BR004

    D_BR004 -->|true| EXCLUDE
    D_BR004 -->|false| D_BR005

    D_BR005 -->|true| EXCLUDE
    D_BR005 -->|false| D_BR006

    D_BR006 -->|true| EXCLUDE
    D_BR006 -->|false| D_BR007

    D_BR007 -->|true| EXCLUDE
    D_BR007 -->|false| D_BR008

    D_BR008 -->|true| EXCLUDE
    D_BR008 -->|false| D_BR009

    D_BR009 -->|true| EXCLUDE
    D_BR009 -->|false| D_BR010

    D_BR010 -->|true| EXCLUDE
    D_BR010 -->|false| D_BR011

    D_BR011 -->|true| EXCLUDE
    D_BR011 -->|false| D_BR012

    D_BR012 -->|true| EXCLUDE
    D_BR012 -->|false| D_BR013

    D_BR013 -->|true| EXCLUDE
    D_BR013 -->|false| D_BR014

    D_BR014 -->|true| EXCLUDE
    D_BR014 -->|false| D_BR015

    D_BR015 -->|true| EXCLUDE
    D_BR015 -->|false| D_BR016

    D_BR016 -->|true| EXCLUDE
    D_BR016 -->|false| D_BR018

    D_BR018 -->|true| EXCLUDE
    D_BR018 -->|false| D_BR019

    D_BR019 -->|true| EXCLUDE
    D_BR019 -->|false| INCLUDE
```

## 3. Data Enrichment Flow

```mermaid
flowchart TD
    %% Data Enrichment Flow using key rules per program

    subgraph "Date Validation"
        DV_IN[Input date parts]
        DV_BR003[BR 003 year lower bound]
        DV_BR004[BR 004 year upper bound]
        DV_BR005[BR 005 month lower bound]
        DV_BR006[BR 006 month upper bound]
        DV_BR008[BR 008 day max per month]
        DV_OUT[Valid date]

        DV_IN --> DV_BR003 --> DV_BR004 --> DV_BR005 --> DV_BR006 --> DV_BR008 --> DV_OUT
    end

    subgraph "Level Description Lookup"
        LD_IN[Input level code]
        LD_BR009[BR 009 map gt 99]
        LD_BR010[BR 010 map gt 99]
        LD_BR011[BR 011 map gt 99]
        LD_BR012[BR 012 map gt 9999]
        LD_OUT[Level description]

        LD_IN --> LD_BR009 --> LD_BR010 --> LD_BR011 --> LD_BR012 --> LD_OUT
    end

    subgraph "Table Preference Evaluation"
        TP_IN[Input table state]
        TP_BR013[BR 013 IN79 on]
        TP_BR014[BR 014 IN79 on]
        TP_BR015[BR 015 IN79 on]
        TP_BR016[BR 016 IN79 on]
        TP_OUT[Preference applied]

        TP_IN --> TP_BR013 --> TP_BR014 --> TP_BR015 --> TP_BR016 --> TP_OUT
    end

    subgraph "Transfer Eligibility"
        TE_IN[Transfer record]
        TE_BR018[BR 018 flag void]
        TE_BR019[BR 019 io outpatient]
        TE_BR017[BR 017 file indicator zero]
        TE_OUT[Eligible transfer]

        TE_IN --> TE_BR018 --> TE_BR019 --> TE_BR017 --> TE_OUT
    end

    DV_OUT --> LD_IN
    LD_OUT --> TP_IN
    TP_OUT --> TE_IN
    TE_OUT --> DV_IN
```

## 4. Counter and Aggregation Logic

```mermaid
flowchart TD
    %% Counter and aggregation logic using validation type rules
    START([Counter process start])
    CNT_INIT[Counter init]
    D_BR001{BR 001 X equals zero}
    D_BR017{BR 017 file indicator zero}

    CNT_EXIT([Exit without increment])
    CNT_NEXT[Proceed to count]

    START --> CNT_INIT --> D_BR001
    D_BR001 -->|true| CNT_EXIT
    D_BR001 -->|false| D_BR017
    D_BR017 -->|true| CNT_EXIT
    D_BR017 -->|false| CNT_NEXT
```

## 5. Application Preference Lookup Flow

```mermaid
flowchart TD
    %% Application preference lookup using XFXTABL narrative
    REQ_START([Preference request])
    PREF_QUERY[Read table preferences]
    PREF_FOUND{Preference row found}

    PREF_APPLY[Apply table driven config]
    PREF_DEFAULT[Apply default behavior]

    REQ_START --> PREF_QUERY --> PREF_FOUND
    PREF_FOUND -->|true| PREF_APPLY
    PREF_FOUND -->|false| PREF_DEFAULT
```

## 6. Org Hierarchy Level Lookup Flow

```mermaid
flowchart TD
    %% Org hierarchy lookup across HXPLVL1 to HXPLVL6
    ORG_START([Hierarchy lookup request])

    L1[Level1 lookup HXPLVL1]
    L2[Level2 lookup HXPLVL2]
    L3[Level3 lookup HXPLVL3]
    L4[Level4 lookup HXPLVL4]
    L5[Level5 lookup HXPLVL5]
    L6[Level6 lookup HXPLVL6]

    ORG_START --> L1 --> L2 --> L3 --> L4 --> L5 --> L6
```

## 7. End to End Summary Flow

```mermaid
flowchart LR
    %% End to end summary from input to response

    subgraph "Input"
        I_IN[Request params level6 and account]
    end

    subgraph "Setup Phase"
        S_CNT[XFXCNTR counter checks]
        S_PREF[XFXTABL preferences]
    end

    subgraph "Data Query"
        Q_TRFR[Query HAPTRFR transfers]
    end

    subgraph "Per Record Enrichment Loop"
        E_DATE[XFXCYMD date validation]
        E_LEVEL[XFXLDSC level description]
        E_STATUS[Status lookup HXPNSTN]
        E_BENEFIT[Benefit lookup HXPBNFIT]
    end

    subgraph "Counting"
        C_SKIP[Count skipped by BR 017 to BR 019]
        C_INC[Count included]
    end

    subgraph "Response"
        R_OUT[Assemble transfer extract]
    end

    I_IN --> S_CNT
    S_CNT --> S_PREF
    S_PREF --> Q_TRFR
    Q_TRFR --> E_DATE
    E_DATE --> E_LEVEL
    E_LEVEL --> E_STATUS
    E_STATUS --> E_BENEFIT
    E_BENEFIT --> C_SKIP
    E_BENEFIT --> C_INC
    C_SKIP --> R_OUT
    C_INC --> R_OUT
```
