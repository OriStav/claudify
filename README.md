# 🧹 Claudify

![Claudify Animation Placeholder](.assets/claudify_animation.gif)

**Transform messy, research-style codebases into clean, well-documented repositories optimized for Claude Code workflows.**

LLMs are incredible at coding, but they choke on massive context windows, tangled dependencies, and disorganized "research-style" scripts. Claude Code needs clear boundaries, specific context, and single-responsibility modules to thrive.

**Claudify** is an 8-phase workflow that uses Opus (for planning) and Sonnet (for execution) to dissect, untangle, and restructure your project so AI can actually work with it.

## Quick Start
```bash
npx skills add oristav/claudify
```

## Why use Claudify?
- **Saves Context:** Breaks massive monoliths into digestible modules.
- **Safety First:** Operates on a dedicated branch, confirms read/write paths, and treats data/secrets as untouchable.
- **AI-Native Context:** Generates the perfect `CLAUDE.md` and repository inventory.
- **Builds Confidence:** Generates tests and Mermaid architecture diagrams (before & after).

## The 8 Phases

| Phase | Name | What it does | Model |
|-------|------|--------------|-------|
| 0 | Pre-flight | Audits the codebase, extracts dependencies & diagrams | Opus |
| 1 | Architect | Creates a step-by-step refactoring & module extraction plan | Opus |
| 2 | Refactor | Executes the plan safely via git commits | Sonnet |
| 3 | Cleanup | Removes dead code and empty directories with confirmation | Sonnet |
| 4 | CLAUDE.md | Writes the perfect AI-context guide tailored to the new repo | Sonnet |
| 5 | README.md | Generates human-readable docs with Mermaid architecture | Sonnet |
| 6 | Tests | Scaffolds test structures for the most critical modules | Sonnet |
| 7 | Validation | Runs diffs, checks logic, and proves the refactor worked | Sonnet |

## Outputs - Keep your hands on the wheel
*Saved to the `claudify/` directory.*

- `PLAN.md` — step-by-step work plan
- `PROTECTED_PATHS.md` — list of paths that must not be read or modified
- `INVENTORY.md` — structured snapshot of the codebase
- `DIAGRAMS.md` — architecture diagrams before, after, diff
- `CLEANUP_LOG.md` — log of removed files and directories
- `REFACTOR_LOG.md` — log of refactoring steps
- `CLAUDIFY_REPORT.md` — log of removed files and directories
- `CLAUDE.md` — AI-context guide tailored to the new repo


## Example: Before and After 
*Claudify Architecture Diagrams*

All architecture diagrams for this refactor session. Updated as each phase completes.

---

### Phase 0 — Architecture Before

```mermaid
graph TD
    subgraph Entry["Entry Points"]
        run_pipeline["run_pipeline.py<br>(CLI pipeline)"]
        report_app["report_app.py<br>(CLI reports)"]
        app["app.py<br>(Streamlit launcher)"]
        tp_root["transperant_pipeline.py<br>(Streamlit pipeline)"]
    end

    subgraph Pages["pages/ — Streamlit Pages"]
        player["player.py"]
        analysis["analysis.py"]
        tp_page["transperant_pipeline.py"]
        parts_tool["parts_clustering_tool.py"]
        editor["editor.py"]
        player_bk["player_bk.py<br>(backup)"]
        tech_review["technical_review.py<br>(stub)"]
    end

    subgraph Apps["apps/ — Sub-applications"]
        trends_app["trends.py"]
        distributions_app["distributions.py"]
        dictioner_app["dictioner.py"]
        miner_app["miner.py<br>(legacy)"]
        subgraph DictMethods["dictioner_methods/"]
            translator_dm["translator.py"]
            cleaner_dm["cleaner.py"]
            reviewer_dm["reviewer.py"]
        end
    end

    subgraph Methods["methods/ — Core Logic (~4,800 lines)"]
        subgraph Pipeline["Pipeline Orchestration"]
            transperancy["transperancy_utils.py<br>(403 lines)"]
            st_transperancy["st_transperancy_utils.py"]
            ppline["ppline_utils.py<br>(404 lines)"]
        end
        subgraph DataProcessing["Data Processing"]
            miner_u["miner_utils.py"]
            cleaner_u["cleaner_utils.py"]
            model_u["model_utils.py"]
            part_u["part_utils.py"]
            scrambler["scrambler_utils.py"]
            epidemics["epidemics_utils.py"]
        end
        subgraph Analysis["Analysis & Features"]
            features_t["features_tabler.py<br>(316 lines)"]
            features_u["features_utils.py"]
            outliers["outliers_utils.py"]
            screeners["screeners_utils.py"]
            screeners_d["screeners_distribution.py"]
            trends_u["trends_utils.py"]
        end
        subgraph IO["I/O & Utilities"]
            db_utils["db_utils.py"]
            utils["utils.py"]
            lng_utils["lng_utils.py"]
            static_t["static_tabler.py"]
            orikis["orikis_utils.py"]
            report_gen["report_generator.py<br>(471 lines)"]
        end
        research["research_utils.py<br>(orphan)"]
        app_utils["app_utils.py<br>(orphan)"]
    end

    subgraph Classes["classes/"]
        Dataset["Dataset.py<br>(legacy loader)"]
        ModelsTranslator["ModelsTranslator.py"]
        Status["Status.py (Log)"]
    end

    subgraph Config["proj_consts/"]
        consts["consts.py<br>(302 lines)"]
        paths["paths.py (Paths class)"]
        paths_root["paths_root.py"]
        mapping["mapping.py<br>(1086 lines, data)"]
    end

    subgraph Data["db/{company}/ (PROTECTED)"]
        new_data["new/ — Raw Excel"]
        cache_data["cache/ — Pickled pipeline"]
        output_data["output/ — Reports"]
    end

    run_pipeline --> transperancy
    run_pipeline --> ppline
    report_app --> ppline
    report_app --> report_gen
    app --> Pages
    tp_root --> transperancy
    player --> transperancy
    analysis --> transperancy
    tp_page --> st_transperancy
    parts_tool --> part_u
    editor --> static_t
    transperancy --> miner_u & cleaner_u & model_u & part_u & db_utils & lng_utils & utils
    model_u --> ModelsTranslator & Status & lng_utils
    cleaner_u --> lng_utils & Status
    paths --> paths_root --> consts
    miner_u -.->|reads| new_data
    db_utils -.->|read/write| cache_data
    report_gen -.->|writes| output_data

    classDef entry fill:#4a90d9,stroke:#2c5f8a,color:#fff
    classDef data fill:#f5a623,stroke:#c47d10,color:#fff
    class run_pipeline,report_app,app,tp_root entry
    class new_data,cache_data,output_data data
```

---

### Phase 5 — Architecture After

```mermaid
graph TD
    subgraph Entry Points
        APP[app.py<br>Streamlit dashboard]
        RUN[run_pipeline.py<br>Headless pipeline]
        REP[report_app.py<br>Report viewer]
        TP[transperant_pipeline.py<br>Step-by-step UI]
    end

    subgraph Config
        CFG[config/settings.py<br>Toggles]
        CNS[config/constants.py<br>Domain constants]
        PTH[config/paths.py<br>Paths class]
    end

    subgraph Pipeline
        MINE[mining.py<br>standard_import]
        CLEAN[cleaning.py<br>raw_cleaner]
        MCLUST[model_clustering.py]
        PCLUST[part_clustering.py]
        FREQ[frequency_analysis.py]
        ORCH[orchestrator.py<br>run_all]
    end

    subgraph Reporting
        AGR[aggregator.py]
        GEN[generator.py]
        EPI[epidemics.py]
    end

    subgraph Analysis
        TRD[trends.py]
        DST[distributions.py]
        SCR[screeners.py]
    end

    subgraph UI
        STP[st_pipeline.py]
        HLP[helpers.py]
        IO[io.py]
    end

    subgraph Pages
        PG1[player.py]
        PG2[analysis.py]
        PG3[pipeline.py]
        PG4[parts_clustering_tool.py]
        PG5[editor.py]
    end

    subgraph Data
        RAW[(db/new/<br>Excel files)]
        CACHE[(db/cache/<br>pkl caches)]
    end

    RAW --> MINE --> CLEAN --> MCLUST & PCLUST --> FREQ
    FREQ --> AGR --> CACHE
    ORCH --> MINE & CLEAN & MCLUST & PCLUST & FREQ
    CFG & PTH --> Pipeline
    APP --> Pages --> UI --> Pipeline & Analysis
    STP --> ORCH
    RUN --> ORCH
    REP --> GEN

    style APP fill:#e1f5fe,stroke:#01579b
    style RUN fill:#e1f5fe,stroke:#01579b
    style REP fill:#e1f5fe,stroke:#01579b
    style TP fill:#e1f5fe,stroke:#01579b
    style RAW fill:#fff9c4,stroke:#f57f17
    style CACHE fill:#fff9c4,stroke:#f57f17
```

---

### Phase 7 — Before vs After Diff

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '#e8f5e9', 'secondaryColor': '#ffebee'}}}%%
graph TD

    subgraph BEFORE ["BEFORE — monolithic methods/ + proj_consts/"]
        style BEFORE fill:#ffebee,stroke:#c62828

        B_CONSTS["proj_consts/consts.py<br>god module: settings + constants<br>+ aggregations + paths"]
        B_CLASSES["classes/<br>Status, ModelsTranslator, Dataset"]
        B_METHODS["methods/<br>19 mixed-purpose files<br>pipeline + ui + reporting + analysis"]
        B_APP["app.py / run_pipeline.py"]
        B_PAGES["pages/ (transperant_pipeline.py)"]
        B_APPS["apps/"]

        B_APP --> B_CONSTS & B_CLASSES & B_METHODS
        B_PAGES --> B_CONSTS & B_METHODS
        B_APPS --> B_CONSTS & B_METHODS
    end

    subgraph AFTER ["AFTER — domain-organised packages"]
        style AFTER fill:#e8f5e9,stroke:#2e7d32

        A_CONFIG["config/<br>settings · constants · aggregations · paths"]
        A_DOMAIN["domain/<br>log · translator · dataset"]
        A_PIPELINE["pipeline/<br>language · mining · cleaning<br>model_clustering · part_clustering<br>feature_engineering · frequency_analysis<br>orchestrator · scrambler"]
        A_REPORTING["reporting/<br>aggregator · generator · epidemics"]
        A_ANALYSIS["analysis/<br>trends · distributions · screeners · outliers"]
        A_UI["ui/<br>st_pipeline · helpers · io<br>components · static_editor"]
        A_APP["app.py / run_pipeline.py"]
        A_PAGES["pages/<br>player · analysis · pipeline<br>parts_clustering_tool · editor"]
        A_APPS["apps/"]
        A_TESTS["tests/<br>test_cleaning · test_language<br>test_constants (51 tests)"]

        A_APP --> A_CONFIG & A_DOMAIN & A_PIPELINE & A_UI
        A_PAGES --> A_CONFIG & A_UI & A_ANALYSIS
        A_APPS --> A_CONFIG
        A_UI --> A_PIPELINE & A_REPORTING
        A_REPORTING --> A_PIPELINE
        A_TESTS -.->|covers| A_PIPELINE & A_CONFIG
    end
```

## License
MIT