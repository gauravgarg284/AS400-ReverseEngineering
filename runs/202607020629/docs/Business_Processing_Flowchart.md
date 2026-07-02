# Business Processing Flowchart – HABADTE Run 202607020629

This document describes the HABADTE patient management batch flow using Mermaid flowcharts. All diagrams are generated from AS400 RPG metadata and semantic interpretations.

## 1. Top-Level Processing Flow

```mermaid
flowchart TD
    START([Start Batch]) --> IN_RECV[Input receipt from HAPTRFR]
    IN_RECV --> CTR_INIT[Counter init via XFXCNTR]
    CTR_INIT --> PREF_LOAD[Preference loads and configuration]
    PREF_LOAD --> HDR_RES[Header resolution and level/table lookups]
    HDR_RES --> MAIN_LOOP[Main transaction query loop]
    MAIN_LOOP --> CTR_ACC[Counter accumulation and validations]
    CTR_ACC --> RESP_ASM[Response assembly into XML headers and details]
    RESP_ASM --> END([End Batch])
```

## 2. Record Filter Gate

```mermaid
flowchart TD
    START([Start Record]) --> BR017{BR-017: FILE INDICATOR = 0?}
    BR017 -- "Yes: SKIP" --> SKIP[Skip record]
    BR017 -- "No" --> BR018{BR-018: FLAG = void/voided?}
    BR018 -- "Yes: SKIP" --> SKIP
    BR018 -- "No" --> BR019{BR-019: INPATIENT_OUTPATIENT_FLAG = 'O'?}
    BR019 -- "Yes: SKIP" --> SKIP
    BR019 -- "No" --> INCLUDE[Include record for enrichment]
```

## 3. Data Enrichment Flow

```mermaid
flowchart TD
    subgraph "Level Description Enrichment"
        L_START[Record enters level enrichment] --> L_RULES[Apply LDAMAP rules BR-009 to BR-012]
        L_RULES --> L_READ[Read HXPLVL1 to HXPLVL6]
        L_READ --> L_OUT[Attach level descriptions]
    end

    subgraph "Table Dictionary Enrichment"
        T_START[Record enters table enrichment] --> T_RULES[Apply indicator *IN79 rules BR-013 to BR-016]
        T_RULES --> T_LOOKUP[Lookup HXPTABLD entries]
        T_LOOKUP --> T_OUT[Attach table driven attributes]
    end

    subgraph "Date Validation Enrichment"
        D_START[Record enters date validation] --> D_RULES[Apply date rules BR-003 to BR-008]
        D_RULES --> D_CHECK[Validate calendar date with XFXCYMD and XFXLEAP]
        D_CHECK --> D_OUT[Mark valid or invalid dates]
    end

    subgraph "XML Routing Enrichment"
        R_START[Record enters routing enrichment] --> R_READ[Read HXFXMLR via XFXGETID]
        R_READ --> R_OUT[Attach routing identifiers]
    end

    L_OUT --> T_START
    T_OUT --> D_START
    D_OUT --> R_START
    R_OUT --> ENRICHED[Record fully enriched]
```

## 4. Counter / Aggregation Logic

```mermaid
flowchart TD
    C_START[Begin counter processing] --> C_BR001{BR-001: X = 0?}
    C_BR001 -- "Yes" --> C_EXIT[Exit counter routine]
    C_BR001 -- "No" --> C_BR002{BR-002: X = 40?}
    C_BR002 -- "Yes" --> C_EXIT
    C_BR002 -- "No" --> C_CONT[Continue counting and aggregation]

    C_CONT --> VAL_START[Begin validation counters]
    VAL_START --> V_BR003{BR-003: Year < 1800?}
    V_BR003 -- "Yes" --> V_EXIT[Flag invalid date]
    V_BR003 -- "No" --> V_BR004{BR-004: Year > 2100?}
    V_BR004 -- "Yes" --> V_EXIT
    V_BR004 -- "No" --> V_BR005{BR-005: Month < 1?}
    V_BR005 -- "Yes" --> V_EXIT
    V_BR005 -- "No" --> V_BR006{BR-006: Month > 12?}
    V_BR006 -- "Yes" --> V_EXIT
    V_BR006 -- "No" --> V_BR007{BR-007: Day < 1?}
    V_BR007 -- "Yes" --> V_EXIT
    V_BR007 -- "No" --> V_BR008{BR-008: Day > DYS(Month)?}
    V_BR008 -- "Yes" --> V_EXIT
    V_BR008 -- "No" --> V_OK[Increment valid date counters]
```

## 5. Application Preference Lookup Flow

```mermaid
flowchart TD
    P_START[Start preference lookup] --> P_REQ[Build preference request from HXXLDA and HXXLEVEL]
    P_REQ --> P_CALL[Call XFXLDSC and XFXTABL]
    P_CALL --> P_QUERY[Query level and table configuration]
    P_QUERY --> P_FOUND{Preference rows found?}
    P_FOUND -- "Yes" --> P_USE[Apply preferences to processing context]
    P_FOUND -- "No" --> P_DEFAULT[Apply default preferences]
    P_USE --> P_DONE[Preference lookup complete]
    P_DEFAULT --> P_DONE
```

## 6. Org / Hierarchy Level Lookup Flow

```mermaid
flowchart TD
    H_START[Start hierarchy lookup] --> H_L1[Lookup Level 1 in HXPLVL1]
    H_L1 --> H_L2{Level 2 exists?}
    H_L2 -- "Yes" --> H_L2_READ[Lookup Level 2 in HXPLVL2]
    H_L2 -- "No" --> H_OUT1[Return Level 1 only]

    H_L2_READ --> H_L3{Level 3 exists?}
    H_L3 -- "Yes" --> H_L3_READ[Lookup Level 3 in HXPLVL3]
    H_L3 -- "No" --> H_OUT2[Return Levels 1 to 2]

    H_L3_READ --> H_L4{Level 4 exists?}
    H_L4 -- "Yes" --> H_L4_READ[Lookup Level 4 in HXPLVL4]
    H_L4 -- "No" --> H_OUT3[Return Levels 1 to 3]

    H_L4_READ --> H_L5{Level 5 exists?}
    H_L5 -- "Yes" --> H_L5_READ[Lookup Level 5 in HXPLVL5]
    H_L5 -- "No" --> H_OUT4[Return Levels 1 to 4]

    H_L5_READ --> H_L6{Level 6 exists?}
    H_L6 -- "Yes" --> H_L6_READ[Lookup Level 6 in HXPLVL6]
    H_L6 -- "No" --> H_OUT5[Return Levels 1 to 5]

    H_L6_READ --> H_OUT6[Return Levels 1 to 6]
```

## 7. End-to-End Summary Flow

```mermaid
flowchart LR
    subgraph "Input"
        IN1[Receive transactions from HAPTRFR]
    end

    subgraph "Setup Phase"
        S1[Load counters and context]
        S2[Load preferences and copybooks]
    end

    subgraph "Data Query"
        Q1[Iterate over HAPTRFR records]
        Q2[Apply filter gate BR-017 to BR-019]
    end

    subgraph "Per-Record Enrichment Loop"
        E1[Level description via XFXLDSC]
        E2[Table dictionary via XFXTABL]
        E3[Date validation via XFXCYMD]
        E4[XML routing via XFXGETID]
    end

    subgraph "Counting"
        C1[Counter exits via XFXCNTR]
        C2[Maintain totals and invalid date counts]
    end

    subgraph "Response"
        R1[Build XML header HXFXMLH]
        R2[Write XML detail HXFXMLD]
        R3[Update statement status XFFNSTN]
    end

    IN1 --> S1
    S1 --> S2
    S2 --> Q1
    Q1 --> Q2
    Q2 --> E1
    E1 --> E2
    E2 --> E3
    E3 --> E4
    E4 --> C1
    C1 --> C2
    C2 --> R1
    R1 --> R2
    R2 --> R3
```
