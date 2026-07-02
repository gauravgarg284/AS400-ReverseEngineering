# Business Processing Flowchart - HABADTE Project

## 1. Top-Level Processing Flow

```mermaid
flowchart TD
    START([Start]) --> INPUT_RECEIPT[Input receipt and context load]
    INPUT_RECEIPT --> COUNTER_INIT[Counter initialization]
    COUNTER_INIT --> PREF_LOADS[Preference and profile loads]
    PREF_LOADS --> HEADER_RESOLUTION[Header and hierarchy resolution]
    HEADER_RESOLUTION --> MAIN_QUERY[Main census query loop]
    MAIN_QUERY --> COUNTER_ACCUM[Counter accumulation]
    COUNTER_ACCUM --> RESP_ASSEMBLY[Response and XML assembly]
    RESP_ASSEMBLY --> END([End])
```

## 2. Record Filter Gate

```mermaid
flowchart TD
    START_FILTER([Start filter evaluation]) --> BR001{BR-001: X equals 0}
    BR001 -- "Exclude (EXIT)" --> EXIT1[Exit centering]
    BR001 -- "Include" --> BR002

    BR002{BR-002: X equals 40} -- "Exclude (EXIT)" --> EXIT2[Exit centering]
    BR002 -- "Include" --> BR003

    BR003{BR-003: VYY < 1800} -- "Exclude (EXIT)" --> EXIT3[Exit date routine]
    BR003 -- "Include" --> BR004

    BR004{BR-004: VYY > 2100} -- "Exclude (EXIT)" --> EXIT4[Exit date routine]
    BR004 -- "Include" --> BR005

    BR005{BR-005: VMM < 01} -- "Exclude (EXIT)" --> EXIT5[Exit date routine]
    BR005 -- "Include" --> BR006

    BR006{BR-006: VMM > 12} -- "Exclude (EXIT)" --> EXIT6[Exit date routine]
    BR006 -- "Include" --> BR007

    BR007{BR-007: VDD < 01} -- "Exclude (EXIT)" --> EXIT7[Exit date routine]
    BR007 -- "Include" --> BR008

    BR008{BR-008: VDD > DYS(VMM)} -- "Exclude (EXIT)" --> EXIT8[Exit date routine]
    BR008 -- "Include" --> FINAL_OK[Record passes date filters]

    FINAL_OK --> HAB017

    HAB017{BR-017: file indicator equals 0} -- "Exclude (SKIP)" --> SKIP1[Skip record]
    HAB017 -- "Include" --> HAB018

    HAB018{BR-018: flag indicator equals void} -- "Exclude (SKIP)" --> SKIP2[Skip record]
    HAB018 -- "Include" --> HAB019

    HAB019{BR-019: inpatient outpatient flag equals outpatient} -- "Exclude (SKIP)" --> SKIP3[Skip record]
    HAB019 -- "Include" --> CENSUS_OK[Record included in census]
```

## 3. Data Enrichment Flow

```mermaid
flowchart TD
    subgraph "Nursing station enrichment"
        NS_START[Record from master file] --> NS_KEY[Build station key]
        NS_KEY --> NS_LOOKUP[Lookup station in HXPNSTN]
        NS_LOOKUP --> NS_FOUND{Station found}
        NS_FOUND -- "Yes" --> NS_APPLY[Apply station name]
        NS_FOUND -- "No" --> NS_DEFAULT[Use station code]
    end

    subgraph "Room class enrichment"
        RC_START[Record after station enrichment] --> RC_CODE[Derive room class code]
        RC_CODE --> RC_CALL[XFXTABL table lookup]
        RC_CALL --> RC_IND{*IN79 on}
        RC_IND -- "Yes" --> RC_EXIT[Exit table processing]
        RC_IND -- "No" --> RC_APPLY[Apply room class description and leave flag]
    end

    subgraph "Transfer history enrichment"
        TR_START[Record after room class] --> TR_KEY[Build transfer key]
        TR_KEY --> TR_READ[Read HAPTRFR history]
        TR_READ --> TR_FOUND{Latest transfer found}
        TR_FOUND -- "Yes" --> TR_APPLY[Overlay room from transfer]
        TR_FOUND -- "No" --> TR_KEEP[Keep master room]
    end

    NS_APPLY --> RC_START
    NS_DEFAULT --> RC_START
    RC_APPLY --> TR_START
    RC_EXIT --> TR_START
    TR_APPLY --> ENRICH_DONE[Enriched record]
    TR_KEEP --> ENRICH_DONE
```

## 4. Counter and Aggregation Logic

```mermaid
flowchart TD
    CNT_START[Begin per record evaluation] --> CNT_FILTERS[Apply BR-017 to BR-019]
    CNT_FILTERS --> CNT_PASS{Record passes filters}

    CNT_PASS -- "Yes" --> CNT_CLASS{Room class leave flag}
    CNT_PASS -- "No" --> CNT_SKIP[Do not count]

    CNT_CLASS -- "Leave" --> CNT_LEAVE[Increment totalLeave]
    CNT_CLASS -- "Not leave" --> CNT_ACTIVE[Increment totalActive]

    CNT_LEAVE --> CNT_TOTAL[Increment totalBeds]
    CNT_ACTIVE --> CNT_TOTAL
    CNT_TOTAL --> CNT_NEXT[Move to next record]
```

## 5. Application Preference Lookup Flow

```mermaid
flowchart TD
    PREF_START[Start preferences] --> PROF_CALL[XFXMRNROL call]
    PROF_CALL --> HXXAPP[HXXAPPPRF SQL profile read]
    HXXAPP --> PROF_MODE[Determine rollup mode]
    PROF_MODE --> MODE_FOUND{Profile found}

    MODE_FOUND -- "Yes" --> MODE_SET[Set MRN rollup mode]
    MODE_FOUND -- "No" --> MODE_DEFAULT[Default to account based]

    MODE_SET --> PREF_DONE[Preferences ready]
    MODE_DEFAULT --> PREF_DONE
```

## 6. Org and Hierarchy Level Lookup Flow

```mermaid
flowchart TD
    ORG_START[Start hierarchy resolution] --> LVL_KEYS[Load level codes from LDA]
    LVL_KEYS --> LVL1[XFXLDSC read HXPLVL1]
    LVL_KEYS --> LVL2[XFXLDSC read HXPLVL2]
    LVL_KEYS --> LVL3[XFXLDSC read HXPLVL3]
    LVL_KEYS --> LVL4[XFXLDSC read HXPLVL4]
    LVL_KEYS --> LVL5[XFXLDSC read HXPLVL5]
    LVL_KEYS --> LVL6[XFXLDSC read HXPLVL6]

    LVL1 --> LVL1_MAP{LDAMAP range ok}
    LVL2 --> LVL2_MAP{LDAMAP range ok}
    LVL3 --> LVL3_MAP{LDAMAP range ok}
    LVL4 --> LVL4_MAP{LDAMAP range ok}
    LVL5 --> LVL5_MAP{LDAMAP range ok}
    LVL6 --> LVL6_MAP{LDAMAP range ok}

    LVL1_MAP -- "Yes" --> LVL1_DESC[Apply level 1 description]
    LVL1_MAP -- "No" --> LVL1_EXIT[Exit level 1]

    LVL2_MAP -- "Yes" --> LVL2_DESC[Apply level 2 description]
    LVL2_MAP -- "No" --> LVL2_EXIT[Exit level 2]

    LVL3_MAP -- "Yes" --> LVL3_DESC[Apply level 3 description]
    LVL3_MAP -- "No" --> LVL3_EXIT[Exit level 3]

    LVL4_MAP -- "Yes" --> LVL4_DESC[Apply level 4 description]
    LVL4_MAP -- "No" --> LVL4_EXIT[Exit level 4]

    LVL5_MAP -- "Yes" --> LVL5_DESC[Apply level 5 description]
    LVL5_MAP -- "No" --> LVL5_EXIT[Exit level 5]

    LVL6_MAP -- "Yes" --> LVL6_DESC[Apply level 6 description]
    LVL6_MAP -- "No" --> LVL6_EXIT[Exit level 6]

    LVL1_DESC --> ORG_DONE[Hierarchy complete]
    LVL2_DESC --> ORG_DONE
    LVL3_DESC --> ORG_DONE
    LVL4_DESC --> ORG_DONE
    LVL5_DESC --> ORG_DONE
    LVL6_DESC --> ORG_DONE
```

## 7. End to End Summary Flow

```mermaid
flowchart LR
    subgraph Input
        IN_START[Input receipt and LDA load]
    end

    subgraph "Setup Phase"
        SET_PREF[Profile and preferences]
        SET_HDR[Hierarchy and header]
    end

    subgraph "Data Query"
        QRY_MAIN[Master and transfer reads]
        QRY_FILTER[Apply census filters]
    end

    subgraph "Per Record Enrichment Loop"
        ENR_NS[Enrich nursing station]
        ENR_RC[Enrich room class and leave]
        ENR_TR[Apply transfer history]
    end

    subgraph Counting
        CNT_ACTIVE[Increment active count]
        CNT_LEAVE[Increment leave count]
        CNT_BEDS[Increment bed count]
    end

    subgraph Response
        RESP_PRINT[Print report lines]
        RESP_XML[Write XML header and detail]
        RESP_API[Return structured response]
    end

    IN_START --> SET_PREF
    SET_PREF --> SET_HDR
    SET_HDR --> QRY_MAIN
    QRY_MAIN --> QRY_FILTER
    QRY_FILTER --> ENR_NS
    ENR_NS --> ENR_RC
    ENR_RC --> ENR_TR
    ENR_TR --> CNT_ACTIVE
    ENR_TR --> CNT_LEAVE
    ENR_TR --> CNT_BEDS
    CNT_BEDS --> RESP_PRINT
    CNT_BEDS --> RESP_XML
    RESP_XML --> RESP_API
```
