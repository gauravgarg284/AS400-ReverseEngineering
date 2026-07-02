# Business Processing Flowcharts – HABADTE Patient Management Engine

This document summarizes the end to end HABADTE processing flow using Mermaid flowcharts. The flows are derived from semantic interpretations and the approved business rules catalog.

---

## 1. Top Level Processing Flow

```mermaid
flowchart TD
    START([Start])
    INPUT_RECEIPT[Input receipt and request validation]
    COUNTER_INIT[Counter initialization via XFXCNTR]
    PREF_LOAD[Preference and control indicator load via XFXTABL]
    HEADER_RESOLVE[Header resolution from OMPMAST and HAPTRFR]
    MAIN_QUERY_LOOP[Main query loop over encounter and transfer records]
    COUNTER_ACCUM[Counter accumulation and aggregation]
    RESPONSE_ASSEMBLY[Response assembly and formatting]
    END([End])

    START --> INPUT_RECEIPT
    INPUT_RECEIPT --> COUNTER_INIT
    COUNTER_INIT --> PREF_LOAD
    PREF_LOAD --> HEADER_RESOLVE
    HEADER_RESOLVE --> MAIN_QUERY_LOOP
    MAIN_QUERY_LOOP --> COUNTER_ACCUM
    COUNTER_ACCUM --> RESPONSE_ASSEMBLY
    RESPONSE_ASSEMBLY --> END
```

---

## 2. Record Filter Gate (Filter Rules)

```mermaid
flowchart TD
    START_FILTER([Start filter gate])
    BR003{{"BR-003: Year < 1800"}}
    BR004{{"BR-004: Year > 2100"}}
    BR005{{"BR-005: Month < 01"}}
    BR006{{"BR-006: Month > 12"}}
    BR007{{"BR-007: Day < 01"}}
    BR008{{"BR-008: Day > daysInMonth"}}
    BR009{{"BR-009: LDAMAP > 99"}}
    BR010{{"BR-010: LDAMAP > 99"}}
    BR011{{"BR-011: LDAMAP > 99"}}
    BR012{{"BR-012: LDAMAP > 9999"}}
    BR013{{"BR-013: IN79 on"}}
    BR014{{"BR-014: IN79 on"}}
    BR015{{"BR-015: IN79 on"}}
    BR016{{"BR-016: IN79 on"}}
    BR018{{"BR-018: Flag void"}}
    BR019{{"BR-019: Outpatient flag"}}

    EXCLUDE([Record excluded])
    INCLUDE([Record included])

    START_FILTER --> BR003
    BR003 -- "True" --> EXCLUDE
    BR003 -- "False" --> BR004
    BR004 -- "True" --> EXCLUDE
    BR004 -- "False" --> BR005
    BR005 -- "True" --> EXCLUDE
    BR005 -- "False" --> BR006
    BR006 -- "True" --> EXCLUDE
    BR006 -- "False" --> BR007
    BR007 -- "True" --> EXCLUDE
    BR007 -- "False" --> BR008
    BR008 -- "True" --> EXCLUDE
    BR008 -- "False" --> BR009

    BR009 -- "True" --> EXCLUDE
    BR009 -- "False" --> BR010
    BR010 -- "True" --> EXCLUDE
    BR010 -- "False" --> BR011
    BR011 -- "True" --> EXCLUDE
    BR011 -- "False" --> BR012
    BR012 -- "True" --> EXCLUDE
    BR012 -- "False" --> BR013

    BR013 -- "True" --> EXCLUDE
    BR013 -- "False" --> BR014
    BR014 -- "True" --> EXCLUDE
    BR014 -- "False" --> BR015
    BR015 -- "True" --> EXCLUDE
    BR015 -- "False" --> BR016
    BR016 -- "True" --> EXCLUDE
    BR016 -- "False" --> BR018

    BR018 -- "True" --> EXCLUDE
    BR018 -- "False" --> BR019
    BR019 -- "True" --> EXCLUDE
    BR019 -- "False" --> INCLUDE
```

---

## 3. Data Enrichment Flow

```mermaid
flowchart TD
    subgraph "Level Hierarchy Enrichment"
        LSTART[Record after filters]
        LMAP_CHECK[Validate LDAMAP mapping code]
        LLOOKUPS[Lookup HXPLVL1 to HXPLVL6 hierarchy]
        LHIER_FLAGS[Set hierarchy flags and descriptors]
        LSTART --> LMAP_CHECK --> LLOOKUPS --> LHIER_FLAGS
    end

    subgraph "Benefit Plan Enrichment"
        BSTART[Record with hierarchy]
        BLOOKUP[Lookup benefit plan and bundle]
        BFLAGS[Set benefit flags hasActiveBenefits or missingBenefits]
        BSTART --> BLOOKUP --> BFLAGS
    end

    subgraph "Status Enrichment"
        SSTART[Record with benefits]
        SLOOKUP[Lookup inpatient status from HXPNSTN]
        SFLAGS[Set status flags isAdmitted isDischarged isPending]
        SSTART --> SLOOKUP --> SFLAGS
    end

    LHIER_OUT[Hierarchy enriched record]
    BEN_OUT[Benefits enriched record]
    STATUS_OUT[Status enriched record]

    LSTART --> LHIER_OUT
    LHIER_OUT --> BSTART
    BFLAGS --> BEN_OUT
    BEN_OUT --> SSTART
    SFLAGS --> STATUS_OUT
```

---

## 4. Counter and Aggregation Logic

```mermaid
flowchart TD
    CNT_START([Start counting])
    INIT_COUNTERS[Initialize counters totalRecordsRead totalRecordsSkipped totalInvalidDates totalInvalidMappings totalInpatientKept]
    LOOP_RECORD[Iterate over retained records]

    CBR001{{"BR-001: X equals 0"}}
    CBR002{{"BR-002: X equals 40"}}
    DATE_INVALID{{"Date invalid BR-003 to BR-008"}}
    MAP_INVALID{{"LDAMAP invalid BR-009 to BR-012"}}
    INPATIENT_KEPT{{"Inpatient kept BR-019 inverse"}}

    UPDATE_SUMMARY[Update summary counters]
    CNT_END([End counting])

    CNT_START --> INIT_COUNTERS --> LOOP_RECORD
    LOOP_RECORD --> CBR001
    LOOP_RECORD --> CBR002
    LOOP_RECORD --> DATE_INVALID
    LOOP_RECORD --> MAP_INVALID
    LOOP_RECORD --> INPATIENT_KEPT

    CBR001 -- "Exit loop" --> UPDATE_SUMMARY
    CBR002 -- "Exit loop" --> UPDATE_SUMMARY
    DATE_INVALID -- "Increment totalInvalidDates" --> UPDATE_SUMMARY
    MAP_INVALID -- "Increment totalInvalidMappings" --> UPDATE_SUMMARY
    INPATIENT_KEPT -- "Increment totalInpatientKept" --> UPDATE_SUMMARY

    UPDATE_SUMMARY --> CNT_END
```

---

## 5. Application Preference Lookup Flow

```mermaid
flowchart TD
    PREF_START([Start preference lookup])
    LOAD_PREF_REQ[Load preference request context]
    TABLE_CHECK{{"BR-013 to BR-016: IN79 on"}}
    PREF_QUERY[Query configuration and preference tables]
    PREF_FOUND[Preference found]
    PREF_NOT_FOUND[Preference not found]
    PREF_APPLY[Apply preferences to processing context]
    PREF_WARN[Log preference warning and continue]

    PREF_START --> LOAD_PREF_REQ --> TABLE_CHECK
    TABLE_CHECK -- "IN79 on" --> PREF_NOT_FOUND
    TABLE_CHECK -- "IN79 off" --> PREF_QUERY

    PREF_QUERY --> PREF_FOUND
    PREF_QUERY --> PREF_NOT_FOUND

    PREF_FOUND --> PREF_APPLY
    PREF_NOT_FOUND --> PREF_WARN
```

---

## 6. Org Hierarchy Level Lookup Flow

```mermaid
flowchart TD
    ORG_START([Start hierarchy lookup])
    LOAD_LVL_KEY[Load level6 key from encounter]
    LVL1_BRANCH{{"Level 1 HXPLVL1"}}
    LVL2_BRANCH{{"Level 2 HXPLVL2"}}
    LVL3_BRANCH{{"Level 3 HXPLVL3"}}
    LVL4_BRANCH{{"Level 4 HXPLVL4"}}
    LVL5_BRANCH{{"Level 5 HXPLVL5"}}
    LVL6_BRANCH{{"Level 6 HXPLVL6"}}
    LVL_COMPLETE[Hierarchy complete]
    LVL_INCOMPLETE[Hierarchy incomplete]

    ORG_START --> LOAD_LVL_KEY
    LOAD_LVL_KEY --> LVL1_BRANCH
    LOAD_LVL_KEY --> LVL2_BRANCH
    LOAD_LVL_KEY --> LVL3_BRANCH
    LOAD_LVL_KEY --> LVL4_BRANCH
    LOAD_LVL_KEY --> LVL5_BRANCH
    LOAD_LVL_KEY --> LVL6_BRANCH

    LVL1_BRANCH --> LVL_COMPLETE
    LVL2_BRANCH --> LVL_COMPLETE
    LVL3_BRANCH --> LVL_COMPLETE
    LVL4_BRANCH --> LVL_COMPLETE
    LVL5_BRANCH --> LVL_COMPLETE
    LVL6_BRANCH --> LVL_COMPLETE

    LVL1_BRANCH --> LVL_INCOMPLETE
    LVL2_BRANCH --> LVL_INCOMPLETE
    LVL3_BRANCH --> LVL_INCOMPLETE
    LVL4_BRANCH --> LVL_INCOMPLETE
    LVL5_BRANCH --> LVL_INCOMPLETE
    LVL6_BRANCH --> LVL_INCOMPLETE
```

---

## 7. End to End Summary Flow

```mermaid
flowchart LR
    subgraph Input
        IN_START([Start])
        IN_REQ[Receive request and validate]
        IN_START --> IN_REQ
    end

    subgraph "Setup Phase"
        SET_COUNTER[Initialize counters via XFXCNTR]
        SET_PREF[Load preferences and control indicators via XFXTABL]
        SET_COUNTER --> SET_PREF
    end

    subgraph "Data Query"
        QRY_HEADER[Read patient header from OMPMAST]
        QRY_TRANS[Read transfer records from HAPTRFR]
        QRY_FILTERS[Apply filter rules BR-003 to BR-019]
        QRY_HEADER --> QRY_TRANS --> QRY_FILTERS
    end

    subgraph "Per Record Enrichment Loop"
        ENR_HIER[Enrich hierarchy using HXPLVL1 to HXPLVL6]
        ENR_BEN[Enrich benefits using OXPBNFIT or HXPBNFIT]
        ENR_STATUS[Enrich status using HXPNSTN]
        ENR_HIER --> ENR_BEN --> ENR_STATUS
    end

    subgraph Counting
        CNT_INIT[Initialize aggregation counters]
        CNT_UPDATE[Update counts totalRecordsRead totalRecordsSkipped totalInvalidDates totalInvalidMappings totalInpatientKept]
        CNT_INIT --> CNT_UPDATE
    end

    subgraph Response
        RESP_BUILD[Assemble header detail and summary]
        RESP_WRITE[Write output to HXFXMLD and HXFXMLH]
        RESP_RETURN[Return response to caller]
        RESP_BUILD --> RESP_WRITE --> RESP_RETURN
    end

    IN_REQ --> SET_COUNTER
    SET_PREF --> QRY_HEADER
    QRY_FILTERS --> ENR_HIER
    ENR_STATUS --> CNT_INIT
    CNT_UPDATE --> RESP_BUILD
```
