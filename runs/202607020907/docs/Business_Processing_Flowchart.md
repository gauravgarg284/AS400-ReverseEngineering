# Business Processing Flowchart – HABADTE Application

This document describes the HABADTE end to end processing logic using Mermaid flowcharts. Each section embeds a flowchart capturing program level behavior inferred from narratives and business rules.

## 1. Top Level Processing Flow

```mermaid
flowchart TD
    START([Start]) --> INPUT_RECEIPT[Receive input records]
    INPUT_RECEIPT --> COUNTER_INIT[Initialize counters and control fields]
    COUNTER_INIT --> PREF_LOADS[Load application preferences and configuration]
    PREF_LOADS --> HEADER_RESOLUTION[Resolve report and XML header context]
    HEADER_RESOLUTION --> MAIN_QUERY_LOOP[Main query loop over transfer records]
    MAIN_QUERY_LOOP --> COUNTER_ACCUM[Accumulate counters and apply validations]
    COUNTER_ACCUM --> RESPONSE_ASSEMBLY[Assemble response and XML payload]
    RESPONSE_ASSEMBLY --> END([End])
```

## 2. Record Filter Gate

```mermaid
flowchart TD
    START_FILTER([Start Filtering]) --> BR001[X equals zero? (BR-001)]
    BR001 -->|Yes - EXIT| EXIT1([Exit - no work])
    BR001 -->|No| BR002[X equals 40? (BR-002)]
    BR002 -->|Yes - EXIT| EXIT2([Exit - batch limit reached])
    BR002 -->|No| BR003[VYY less than 1800? (BR-003)]
    BR003 -->|Yes - EXCLUDE| EXCL_YEAR_LOW([Exclude record - invalid year])
    BR003 -->|No| BR004[VYY greater than 2100? (BR-004)]
    BR004 -->|Yes - EXCLUDE| EXCL_YEAR_HIGH([Exclude record - invalid year])
    BR004 -->|No| BR005[VMM less than 01? (BR-005)]
    BR005 -->|Yes - EXCLUDE| EXCL_MONTH_LOW([Exclude record - invalid month])
    BR005 -->|No| BR006[VMM greater than 12? (BR-006)]
    BR006 -->|Yes - EXCLUDE| EXCL_MONTH_HIGH([Exclude record - invalid month])
    BR006 -->|No| BR007[VDD less than 01? (BR-007)]
    BR007 -->|Yes - EXCLUDE| EXCL_DAY_LOW([Exclude record - invalid day])
    BR007 -->|No| BR008[VDD greater than DYS(VMM)? (BR-008)]
    BR008 -->|Yes - EXCLUDE| EXCL_DAY_HIGH([Exclude record - invalid day])
    BR008 -->|No| BR009[LDAMAP greater than 99? (BR-009/010/011)]
    BR009 -->|Yes - EXCLUDE| EXCL_MAP_RANGE([Exclude record - invalid mapping])
    BR009 -->|No| BR012[LDAMAP greater than 9999? (BR-012)]
    BR012 -->|Yes - EXCLUDE| EXCL_MAP_ABS([Exclude record - mapping overflow])
    BR012 -->|No| BR013[*IN79 on or active? (BR-013/014/015/016)]
    BR013 -->|Yes - EXIT| EXIT_TBL([Exit table lookup])
    BR013 -->|No| BR017[File indicator equals zero? (BR-017)]
    BR017 -->|Yes - EXCLUDE| EXCL_FILE_IND([Skip record - invalid file])
    BR017 -->|No| BR018[Flag indicator equals void? (BR-018)]
    BR018 -->|Yes - EXCLUDE| EXCL_VOID([Skip record - void transfer])
    BR018 -->|No| BR019[Inpatient flag equals outpatient? (BR-019)]
    BR019 -->|Yes - EXCLUDE| EXCL_OUTPATIENT([Skip record - outpatient])
    BR019 -->|No - INCLUDE| INCLUDE_REC([Include record in processing])
```

## 3. Data Enrichment Flow

```mermaid
flowchart TD
    subgraph HABADTE_Core
        H_CORE_START([Start HABADTE record loop])
        H_CORE_START --> H_CHECKS[Apply HABADTE eligibility checks]
        H_CHECKS --> H_ENRICH_LEVEL[Call XFXLDSC for level descriptions]
        H_ENRICH_LEVEL --> H_ENRICH_DICT[Call XFXTABL for table lookups]
        H_ENRICH_DICT --> H_MRN_ROLLOVER[Call XFXMRNROL for MRN rollover]
        H_MRN_ROLLOVER --> H_XML_WRITE[Write XML detail record]
    end

    subgraph Level_Enrichment
        L_START([XFXLDSC entry])
        L_START --> L_READ_LVL[Read HXPLVL1 to HXPLVL6]
        L_READ_LVL --> L_VALIDATE_MAP[Validate LDAMAP range]
        L_VALIDATE_MAP --> L_RETURN_DESC[Return level descriptions]
    end

    subgraph Table_Enrichment
        T_START([XFXTABL entry])
        T_START --> T_READ_DICT[Read HXPTABLD and table variants]
        T_READ_DICT --> T_CHECK_IND79[Check indicator 79]
        T_CHECK_IND79 --> T_RETURN_DESC[Return table descriptions]
    end

    subgraph MRN_Rollover
        M_START([XFXMRNROL entry])
        M_START --> M_PROFILE_LOOKUP[Call HXXAPPPRF for profiles]
        M_PROFILE_LOOKUP --> M_APPLY_MRN[Apply MRN rollover]
        M_APPLY_MRN --> M_RETURN_MRN[Return current MRN]
    end

    H_ENRICH_LEVEL --> L_START
    H_ENRICH_DICT --> T_START
    H_MRN_ROLLOVER --> M_START
    L_RETURN_DESC --> H_ENRICH_DICT
    T_RETURN_DESC --> H_MRN_ROLLOVER
    M_RETURN_MRN --> H_XML_WRITE
```

## 4. Counter and Aggregation Logic

```mermaid
flowchart TD
    CNT_START([Start counters]) --> CNT_INIT[Initialize counters]
    CNT_INIT --> CNT_LOOP[Loop over eligible records]

    CNT_LOOP --> CNT_X_ZERO[X equals zero? (BR-001)]
    CNT_X_ZERO -->|Yes| CNT_EXIT_ZERO[Exit loop - no items]
    CNT_X_ZERO -->|No| CNT_X_FORTY[X equals 40? (BR-002)]
    CNT_X_FORTY -->|Yes| CNT_EXIT_LIMIT[Exit loop - batch limit]
    CNT_X_FORTY -->|No| CNT_INC[Increment processedTransfers]

    CNT_INC --> CNT_DATE_VAL[Validate date via XFXCYMD]
    CNT_DATE_VAL --> DATE_INVALID[Date invalid?]
    DATE_INVALID -->|Yes| CNT_DATE_FAIL[Increment dateValidationFailures]
    CNT_DATE_FAIL --> CNT_NEXT[Move to next record]
    DATE_INVALID -->|No| CNT_NEXT_VALID[Continue processing]
    CNT_NEXT_VALID --> CNT_NEXT

    CNT_NEXT --> CNT_LOOP
```

## 5. Application Preference Lookup Flow

```mermaid
flowchart TD
    PREF_START([Start preferences]) --> PREF_LOAD_LDA[Copy HXXLDA configuration]
    PREF_LOAD_LDA --> PREF_LOAD_LEVEL[Copy HXXLEVEL configuration]
    PREF_LOAD_LEVEL --> PREF_LOAD_XMLCFG[Copy HXXXML and CXXXMLP]
    PREF_LOAD_XMLCFG --> PREF_QUERY[Query application profile via HXXAPPPRF]

    PREF_QUERY --> PREF_FOUND[Profile found?]
    PREF_FOUND -->|Yes| PREF_APPLY[Apply preference settings]
    PREF_FOUND -->|No| PREF_DEFAULT[Apply default preferences]

    PREF_APPLY --> PREF_DONE([Preferences ready])
    PREF_DEFAULT --> PREF_DONE
```

## 6. Org and Hierarchy Level Lookup Flow

```mermaid
flowchart TD
    ORG_START([Start org lookup]) --> L1[Lookup Level 1 in HXPLVL1]
    L1 --> L2[Lookup Level 2 in HXPLVL2]
    L2 --> L3[Lookup Level 3 in HXPLVL3]
    L3 --> L4[Lookup Level 4 in HXPLVL4]
    L4 --> L5[Lookup Level 5 in HXPLVL5]
    L5 --> L6[Lookup Level 6 in HXPLVL6]
    L6 --> ORG_DONE([Org hierarchy resolved])
```

## 7. End to End Summary Flow

```mermaid
flowchart LR
    subgraph Input
        IN_START([Start])
        IN_START --> IN_READ[Read transfer and context inputs]
    end

    subgraph "Setup Phase"
        SET_PREF[Load preferences and profiles]
        SET_HDR[Resolve header and sequence]
    end

    subgraph "Data Query"
        QRY_HAPTRFR[Query HAPTRFR transfer records]
        QRY_MASTER[Access OMPMAST and related data]
    end

    subgraph "Per-Record Enrichment Loop"
        ENR_FILTER[Apply HABADTE eligibility filters]
        ENR_DATE[Validate dates via XFXCYMD]
        ENR_LEVEL[Enrich levels via XFXLDSC]
        ENR_DICT[Enrich dictionary via XFXTABL]
        ENR_MRN[Apply MRN rollover]
    end

    subgraph Counting
        CNT_TRACK[Track counters and totals]
    end

    subgraph Response
        RESP_XML[Write XML header and detail]
        RESP_OUT[Return response payload]
    end

    IN_READ --> SET_PREF
    SET_PREF --> SET_HDR
    SET_HDR --> QRY_HAPTRFR
    QRY_HAPTRFR --> ENR_FILTER
    ENR_FILTER --> ENR_DATE
    ENR_DATE --> ENR_LEVEL
    ENR_LEVEL --> ENR_DICT
    ENR_DICT --> ENR_MRN
    ENR_MRN --> CNT_TRACK
    CNT_TRACK --> RESP_XML
    RESP_XML --> RESP_OUT
```
