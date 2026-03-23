# Oracle Odds AI — System Architecture

## AI Pipeline Flow

```mermaid
flowchart TD
    A[INPUT DATA] --> B[Odds Data]
    A --> C[Match Info]
    A --> D[User History]

    B --> E[1. Odds Normalisation\nStrip vig → fair probability]
    E --> F[2. Poisson λ Derivation\nor NBA Pace Model]
    F --> G[3. Fair Probabilities\nhome / draw / away]
    B --> H[4. Player Z-Scores]
    H --> I[Player Props]

    C --> J[Team History]
    C --> K[Similar Matches]
    D --> L[Training Examples]

    J --> M[Sport Detection]
    K --> M
    L --> M

    G --> N[GEMINI PROMPT\ngeminiService.ts]
    M --> O[Search Grounding\nLive web — injuries, form, news]
    O --> P[Match Analysis]
    P --> Q[Prediction Synthesis]

    N --> Q
    I --> Q

    Q --> R[Predicted Score]
    Q --> S[Reasoning]
    Q --> T[Confidence %]

    style E fill:#7c3aed,color:#fff
    style F fill:#7c3aed,color:#fff
    style G fill:#7c3aed,color:#fff
    style H fill:#7c3aed,color:#fff
    style M fill:#f59e0b,color:#fff
    style O fill:#f59e0b,color:#fff
    style P fill:#f59e0b,color:#fff
    style Q fill:#f59e0b,color:#fff
    style R fill:#06b6d4,color:#fff
    style S fill:#06b6d4,color:#fff
    style T fill:#06b6d4,color:#fff
```

---

## Adaptive Learning Loop

```mermaid
flowchart LR
    A[User runs analysis] --> B[Prediction stored\nin predictions_ledger]
    B --> C[Match finishes\nauto-detected via cron]
    C --> D[Actual score vs predicted\nresiduals calculated]
    D --> E[league_priors updated\navg goals, home bias drift]
    D --> F[team_strengths updated\nattack / defense λ]
    E --> G[model_versions snapshot]
    F --> G
    G --> H{Best model\nwins tournament}
    H -->|Promoted| I[Next prediction uses\nupdated priors]
    I --> A

    style A fill:#10b981,color:#fff
    style H fill:#10b981,color:#fff
    style I fill:#10b981,color:#fff
```

---

## Settlement Flow (nightly pg_cron)

```mermaid
sequenceDiagram
    participant C as pg_cron (03:00 UTC)
    participant F as settle-predictions<br/>Edge Function
    participant P as predictions_ledger
    participant M as match_history
    participant L as league_priors
    participant V as model_versions

    C->>F: HTTP POST (trigger)
    F->>P: Fetch unsettled predictions (settled_at IS NULL)
    F->>M: Fetch actual scores for each match_id
    F->>P: Write actual scores + residuals + prediction_status
    F->>L: updateLeaguePriorsFromSettledResults()
    F->>V: refreshModelVersionSnapshot()
    V->>V: Tournament — promote best avg_total_error model
```

---

## Quant Engine — Sport Branching

```mermaid
flowchart TD
    A[Match odds input] --> B{detectSport}
    B -->|Football| C[Poisson / Dixon-Coles]
    B -->|Basketball| D[NBA Pace Model]
    B -->|Tennis| E[Direct probability]

    C --> C1[Remove vig\nfair probability]
    C1 --> C2[Reconstruct expected goals λ]
    C2 --> C3[Apply league priors\navg goals + home bias]
    C3 --> C4[Apply team strength λ\nattack / defense]
    C4 --> C5[Dixon-Coles correction\nlow-score interdependence]
    C5 --> C6[Monte Carlo\nWin/Draw/Loss %]

    D --> D1[2-way odds normalisation\nno draw in NBA]
    D1 --> D2[Home win prob\n→ implied point spread]
    D2 --> D3[League avg PPG 110\n± spread + home court 3.5pts]
    D3 --> D4[Adjust by team\nscoring history if available]
    D4 --> D5[homeLambda pts\nawayLambda pts\npredictedTotal]

    style C fill:#7c3aed,color:#fff
    style D fill:#2563eb,color:#fff
    style E fill:#059669,color:#fff
```

---

## Infrastructure Overview

```mermaid
graph TB
    subgraph CLIENT["CLIENT — React + TypeScript + Tailwind"]
        L[Lobby\nMatch cards]
        B[BetSlip\nAnalysis output]
        H[History Vault\nReceipts + W/L]
        AC[Accuracy\nDashboard]
        LD[Live Dashboard\nGlobal Pulse]
    end

    subgraph SERVICES["SERVICE LAYER"]
        MS[mathService\nQuant engine]
        GS[geminiService\nGemini 2.5 Flash]
        CS[cacheService\nSupabase queries]
        LS[liveDataService\nMatch polling]
    end

    subgraph SUPABASE["SUPABASE"]
        AUTH[Auth + RLS\nTier enforcement]
        DB[(Postgres\nDatabase)]
        EF[Edge Functions\nsettle / brain-sync\nodds-aggregator / monitor]
        CRON[pg_cron\nNightly settlement]
    end

    subgraph EXTERNAL["EXTERNAL APIs"]
        OA[The Odds API\n30+ sports]
        ESPN[ESPN API\nLive NBA scores]
        FD[Football-Data.org\nEU fixtures]
        GEM[Gemini 2.5 Flash\nSearch grounding]
    end

    CLIENT --> SERVICES
    SERVICES --> SUPABASE
    SUPABASE --> EXTERNAL
    CRON --> EF
```
