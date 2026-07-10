# Business Processing Flowchart

Auto-generated from pipeline artifacts. Covers all 7 processing phases.

## 1. Top-Level Processing Flow

End-to-end order of program execution, derived from Agent-2's program narratives (interpretations.json). Shows the overall request-to-response path.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart TD
    START([START]) --> InputReceipt
    InputReceipt[Input Receipt and Validation]
    InputReceipt --> Step0
    Step0["XFXCNTR: RPGLE program in domain 'Data Maintenance'. Contains 2 rule("]
    Step0 --> Step1
    Step1["XFXCYMD: RPGLE program in domain 'Data Maintenance'. Contains 6 rule("]
    Step1 --> Step2
    Step2["XFXLDSC: RPGLE program in domain 'Data Maintenance'. Contains 4 rule("]
    Step2 --> Step3
    Step3["XFXTABL: RPGLE program in domain 'Data Maintenance'. Contains 4 rule("]
    Step3 --> Step4
    Step4["HABADTE: RPGLE program in domain 'Patient Management'. Contains 3 rul"]
    Step4 --> Step5
    Step5["HXXAPPPRF: SQLRPGLE program in domain 'Data Maintenance'. Contains 3 ru"]
    Step5 --> Step6
    Step6["XFXGETID: RPGLE program in domain 'Data Maintenance'. Cyclomatic compl"]
    Step6 --> Step7
    Step7["XFXLEAP: RPGLE program in domain 'Data Maintenance'. Cyclomatic compl"]
    Step7 --> Step8
    Step8["XFXMRNROL: RPGLE program in domain 'Data Maintenance'. Calls programs: "]
    Step8 --> END([END])
```

## 2. Record Filter Gate

Extracted CABEQ/CAB decision rules that decide whether a record is included or excluded from processing, in evaluation order.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart TD
    RecordIn([Record Enters Filter]) --> F0
    F0{"BR-017: When -FILE INDICATOR equals zero, branch to 'SKIP' (CABEQ)"}
    F0 -->|EXCLUDE| SKIP0([Skip Record])
    F0 -->|INCLUDE| F1
    F1{"BR-018: When -FLAG INDICATOR equals void/voided, branch to (CABEQ)"}
    F1 -->|EXCLUDE| SKIP1([Skip Record])
    F1 -->|INCLUDE| F2
    F2{"BR-019: When -INPATIENT/OUTPATIENT FLAG equals outpatient, (CABEQ)"}
    F2 -->|EXCLUDE| SKIP2([Skip Record])
    F2 -->|INCLUDE| RecordOut([Record Accepted])
```

## 3. Data Enrichment Flow

Per-program key business rules that enrich a record after it passes the filter gate, grouped by the program that applies them.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart TD
    subgraph XFXCNTR ["XFXCNTR Enrichment"]
        XFXCNTR_0["BR-001: When X equals zero, branch to 'EXIT'"]
        XFXCNTR_1["BR-002: When X equals 40, branch to 'EXIT'"]
    end
    subgraph XFXCYMD ["XFXCYMD Enrichment"]
        XFXCYMD_0["BR-008: When VDD is greater than DYS(VMM), branch to 'EXIT"]
        XFXCYMD_1["BR-003: When VYY is less than 1800, branch to 'EXIT'"]
        XFXCYMD_2["BR-004: When VYY is greater than 2100, branch to 'EXIT'"]
        XFXCYMD_3["BR-005: When VMM is less than 01, branch to 'EXIT'"]
    end
    subgraph XFXLDSC ["XFXLDSC Enrichment"]
        XFXLDSC_0["BR-009: When LDAMAP is greater than 99, branch to 'EXIT'"]
        XFXLDSC_1["BR-010: When LDAMAP is greater than 99, branch to 'EXIT'"]
        XFXLDSC_2["BR-011: When LDAMAP is greater than 99, branch to 'EXIT'"]
        XFXLDSC_3["BR-012: When LDAMAP is greater than 9999, branch to 'EXIT'"]
    end
    subgraph XFXTABL ["XFXTABL Enrichment"]
        XFXTABL_0["BR-013: When *IN79 equals on/active, branch to 'EXIT'"]
        XFXTABL_1["BR-014: When *IN79 equals on/active, branch to 'EXIT'"]
        XFXTABL_2["BR-015: When *IN79 equals on/active, branch to 'EXIT'"]
        XFXTABL_3["BR-016: When *IN79 equals on/active, branch to 'EXIT'"]
    end
    subgraph HABADTE ["HABADTE Enrichment"]
        HABADTE_0["BR-018: When -FLAG INDICATOR equals void/voided, branch to"]
        HABADTE_1["BR-019: When -INPATIENT/OUTPATIENT FLAG equals outpatient,"]
        HABADTE_2["BR-017: When -FILE INDICATOR equals zero, branch to 'SKIP'"]
    end
    subgraph HXXAPPPRF ["HXXAPPPRF Enrichment"]
        HXXAPPPRF_0["BR-020: SQL program accesses table 'HXPAPPPRF'"]
        HXXAPPPRF_1["BR-021: SQL program accesses table 'HXPAPPL6'"]
        HXXAPPPRF_2["BR-022: SQL program accesses table 'HXPAPPPRF'"]
    end
    XFXCNTR_0 --> XFXCYMD_0
    XFXCYMD_0 --> XFXLDSC_0
    XFXLDSC_0 --> XFXTABL_0
    XFXTABL_0 --> HABADTE_0
    HABADTE_0 --> HXXAPPPRF_0
```

## 4. Counter and Aggregation Logic

Business rules that increment, sum, or tally values during processing (counters, totals, running sums), in the order they're applied.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart TD
    START_AGG([Begin Aggregation Pass]) --> Init
    Init[Initialize Counters to Zero]
    Init --> MainLoop[Main Processing Loop]
    MainLoop --> CountRec[Increment Record Counter]
    CountRec --> WriteTotal[Write Total Count to Output]
    WriteTotal --> AGG_END([Aggregation Complete])
```

## 5. Application Preference Lookup Flow

How a configurable setting is resolved at runtime: look up the preference table first, fall back to a system default if no entry is found.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart TD
    PrefStart([Preference Lookup Start]) --> QueryPref
    QueryPref[Query Application Preference Table]
    QueryPref --> FoundQ{Preference Found?}
    FoundQ -->|YES| UseValue[Use Configured Value]
    FoundQ -->|NO| UseDefault[Use System Default Value]
    UseValue --> PrefDone([Preference Resolved])
    UseDefault --> PrefDone
    %% Program: XFXLDSC
    %% When LDAMAP is greater than 99, branch to 'EXIT'
    %% When LDAMAP is greater than 99, branch to 'EXIT'
```

## 6. Org and Hierarchy Level Lookup Flow

How a record is resolved to its organizational level (facility, department, region, etc.) — one branch per level the program checks.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart TD
    OrgStart([Org Lookup Request]) --> CheckLevel
    CheckLevel{Org Level?}
    CheckLevel -->|Level 1| L0["XFXCYMD lookup"]
    L0 --> OrgDone
    OrgDone([Org Level Resolved])
```

## 7. End-to-End Summary Flow

One-page rollup of sections 1-6: input, setup (preferences + org lookup), query, per-record enrichment loop, and response assembly.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 60, "rankSpacing": 70}}}%%
flowchart LR
    subgraph inp ["Input Phase"]
        INP[Receive Request Parameters]
    end
    subgraph setup ["Setup Phase"]
        PREF[Load Preferences]
        ORG[Resolve Org Hierarchy]
    end
    subgraph query ["Data Query"]
        QSQL[Execute Main SQL Query]
        FILT[Apply Record Filters]
    end
    subgraph enrich ["Per-Record Enrichment Loop"]
        ENRCH[Enrich Each Record]
        CNT[Update Counters]
    end
    subgraph resp ["Response"]
        ASMBL[Assemble Response]
        OUT[Return to Caller]
    end
    INP --> PREF
    PREF --> ORG
    ORG --> QSQL
    QSQL --> FILT
    FILT --> ENRCH
    ENRCH --> CNT
    CNT --> ASMBL
    ASMBL --> OUT
```
