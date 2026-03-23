# Oracle Odds AI

> **AI-powered sports prediction platform** — live odds, quantitative modelling, and machine learning that gets smarter with every settled match.

🌐 **[oracleai.live](https://oracleai.live)** &nbsp;|&nbsp; ⚡ React · Supabase · Gemini 2.5 Flash

---

<div align="center">

<img src="assets/pulisic.png" width="130" alt="Pulisic" />
&nbsp;&nbsp;
<img src="assets/alexander_arnold.png" width="130" alt="Alexander-Arnold" />
&nbsp;&nbsp;
<img src="assets/basketball_player.png" width="130" alt="Basketball" />
&nbsp;&nbsp;
<img src="assets/tennis_player.png" width="130" alt="Tennis" />

### The sports intelligence layer that professional bettors don't want you to have.

</div>

---

## Live Traction

> Organic growth — zero paid advertising.

| Metric | Value |
|--------|-------|
| Active users (last 30 days) | **295** |
| Sessions | **686** |
| Google organic | 28 users / 110 sessions |
| Twitter referral | 12 users / 83 sessions |
| Stripe checkout sessions | 10 |
| Top city | Lagos (63), Chicago (35), Abuja (29) |
| Geo spread | Nigeria · USA · Ghana · UK |

*Source: Google Analytics, March 2026*

---

## Screenshots

| Global Pulse Dashboard | Match QuickPicks |
|------------------------|-----------------|
| ![Dashboard](assets/screenshots/dashboard.png) | ![QuickPicks](assets/screenshots/quickpicks.png) |

| Pricing Plans | AI Pipeline Diagram |
|---------------|-------------------|
| ![Pricing](assets/screenshots/pricing.png) | ![Pipeline](assets/screenshots/pipeline.png) |

---

## What is Oracle?

Oracle Odds AI is a full-stack sports prediction platform that combines:

- **Live odds ingestion** from multiple bookmakers (The Odds API, BetExplorer, Pinnacle)
- **Gemini 2.5 Flash AI** with grounded web search for real-time injury/form context
- **Proprietary quant engine** — Poisson/Dixon-Coles for football, NBA pace model for basketball
- **Adaptive learning** — every settled match recalibrates league priors and team strength profiles
- **Tiered subscription system** with Stripe payments and Supabase RLS enforcement at the database layer

---

## Analysis Pipeline

Every match analysis runs a 5-stage pipeline:

```
Stage 1: Odds Normalisation
  → Strip bookmaker vig → find true fair probability per outcome

Stage 2: Quant Baseline
  → Football: Poisson/Dixon-Coles expected goals
  → Basketball: NBA pace model point expectations
  → Tennis: Direct probability from odds

Stage 3: Adaptive Context Fetch (Supabase)
  → League priors: avg goals, home bias, calibration bias
  → Team strengths: attack/defense λ from settled history

Stage 4: AI Analysis (Gemini 2.5 Flash + Search Grounding)
  → Injuries, form, H2H, weather, team news — live from the web
  → Quant baseline anchors the AI's score prediction

Stage 5: Verdict + Storage
  → Confidence gauge, scoreline, best bet, narrative signals
  → Prediction stored to ledger for future model training
```

See [full architecture diagrams →](docs/architecture.md)

---

## Core Features

### Match Intelligence
- Confidence gauge (0–100%)
- Predicted scoreline per sport
- Best bet recommendation (moneyline / spread / O/U)
- Narrative signals — *Bayern's Dominance · Public Favorite Bias · UCL Knockout Stage*
- Oracle QuickPicks with verified confidence bars

### QuickSlip — Batch Analysis
Add up to 4 matches to a slip and run AI analysis on all simultaneously. Results save as receipt cards with W/L settlement tracking.

### History Vault
- Receipt cards with Win/Loss/Pending badges
- Confidence trend sparkline (last 10 analyses)
- Win rate, best streak, recent form dots
- Auto-settlement via pg_cron nightly

### Global Pulse Dashboard
- Platform-wide total analyses
- Confidence yield simulation
- System edge vs closing line value
- Live analysis stream feed

### Accuracy Dashboard
Proof-of-performance tracker:
- Outcome accuracy % (correct winner/draw)
- Exact score rate %
- Average goals off per prediction
- By-league breakdown with progress bars
- Monthly trend chart

### Leaderboard
Ranked by total predictions. Elite tier gets private Telegram community.

---

## Sports Coverage

| Sport | Competitions |
|-------|-------------|
| Football | Premier League, La Liga, Serie A, Bundesliga, Ligue 1, UCL, Championship + 15 more |
| Basketball | NBA |
| Tennis | ATP, WTA, Grand Slams |

---

## Subscription Tiers

| | Free Tier | Starter Pro | Elite Data | Oracle Elite |
|-|-----------|-------------|------------|--------------|
| **Price** | $0/mo | $2/mo | $10/mo | $20/mo |
| **AI Predictions/day** | 1 | 3 | 10 | Unlimited |
| **QuickSlip** | — | 2 slots | Unlimited | Unlimited |
| **Leagues** | Major only | Mid-tier | All sports | All sports |
| **Extras** | — | — | — | High-confidence toggle · Telegram Alpha · VIP Bankroll |

Tier enforcement is applied at **three layers**: frontend UI gates → application logic → Supabase RLS (free users literally cannot query premium data even if they bypass the frontend).

---

## Tech Stack

### Frontend
- **React 18** + TypeScript — component-driven, context-based auth
- **Tailwind CSS 4** with custom dark theme (`#04040a` base)
- **Vite 5** build tool
- Toast notification system (module-level event emitter, zero dependencies)
- Skeleton loading, match countdown timers, odds movement arrows
- Mobile-first: 44px+ touch targets, safe-area insets, `touch-action: manipulation`

### Backend & Infrastructure
- **Supabase** — Postgres, Auth, Edge Functions (Deno), Row Level Security
- **pg_cron** — nightly auto-settlement of predictions at 03:00 UTC
- **Stripe** — subscription billing with webhook-driven tier updates

### AI & Data
- **Gemini 2.5 Flash** — primary AI analysis with Google Search grounding
- **The Odds API** — live odds across 30+ sports
- **ESPN API** — live NBA scores
- **Football-Data.org** — European football fixtures
- **BetExplorer + Pinnacle** — sharp odds reference

### Multi-Key Rotation
Oracle manages a pool of Gemini API keys with per-project concurrency limits, cooldown tracking per key after 429 responses, and progressive backoff with smart key selection.

---

## Quantitative Engine

### Football — Poisson/Dixon-Coles
```
1. Remove bookmaker vig → fair probability
2. Reconstruct expected goals (λ) from fair probability
3. Apply league priors (avg goals, home bias) from DB
4. Apply team strength adjustments (attack/defense λ)
5. Dixon-Coles correction for low-score interdependence
6. Monte Carlo simulation → Win/Draw/Loss probabilities
```

### Basketball — NBA Pace Model
```
1. 2-way odds normalisation (no draw in NBA)
2. Home win probability → implied point spread
3. League average PPG (110) ± spread + home court advantage (3.5 pts)
4. Adjusted by adaptive team scoring history if available
5. Outputs: homeLambda (pts), awayLambda (pts), predictedTotal, impliedSpread
```

### Adaptive Learning Loop
```
Prediction stored → Match settles → Residuals calculated
→ league_priors updated (avg goals drift, home bias, calibration)
→ team_strengths updated (attack/defense λ)
→ model_versions snapshot → best model promoted via tournament
```

---

## Machine Learning Loop

```
User runs analysis
       ↓
Prediction stored in predictions_ledger
       ↓
Match finishes (auto-detected via cron)
       ↓
Actual score vs predicted → residuals calculated
       ↓
league_priors updated (avg goals, home bias drift)
       ↓
team_strengths updated (attack/defense λ)
       ↓
model_versions snapshot → best model promoted
       ↓
Next prediction uses updated priors ✓
```

Oracle's accuracy **compounds** — more predictions = more settled data = better baselines.

---

## Database Schema (Key Tables)

| Table | Purpose |
|-------|---------|
| `profiles` | User accounts, subscription tier, prediction counts |
| `predictions_ledger` | Every AI prediction with actual outcome after settlement |
| `match_history` | Settled match results |
| `league_priors` | Per-league learned averages (goals, home bias, calibration bias) |
| `team_strengths` | Per-team attack/defense lambdas |
| `model_versions` | Model performance snapshots (exact hit rate, avg error) |
| `math_cache` | Quant baselines cached per match |

See [full schema docs →](docs/database-schema.md)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                        CLIENT                           │
│  React + TypeScript + Tailwind                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │  Lobby   │ │ BetSlip  │ │ History  │ │Accuracy  │  │
│  │ (Matches)│ │(Analysis)│ │(Receipts)│ │Dashboard │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                    SERVICE LAYER                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  mathService│  │geminiService│  │ liveData    │    │
│  │  (Quant AI) │  │ (Gemini AI) │  │  Service    │    │
│  └─────────────┘  └─────────────┘  └─────────────┘    │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                     SUPABASE                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │
│  │   Auth   │  │ Postgres │  │   Edge Functions     │  │
│  │  + RLS   │  │  (Data)  │  │ settle / brain-sync  │  │
│  └──────────┘  └──────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                   EXTERNAL APIs                         │
│   The Odds API │ ESPN │ Football-Data │ Gemini Search   │
└─────────────────────────────────────────────────────────┘
```

---

## Key Engineering Decisions

**Why Supabase over Firebase?**
Row Level Security enforces subscription tiers at the database level — a free user literally cannot query premium data even if they bypass the frontend. No application-layer trust required.

**Why Gemini 2.5 Flash with Search Grounding?**
Sports predictions go stale within hours. Grounded search means every analysis queries live injury reports, team news, and form — not static training data from months ago.

**Why a custom quant layer instead of just prompting the AI?**
LLMs hallucinate scores. The quant baseline anchors the AI to mathematically sound probability ranges derived from actual market odds. The AI adds context the quant can't see (injuries, psychology, weather). Neither alone is as accurate as both together.

**Why separate models for football vs basketball?**
Poisson distribution models goals well (0–5 range, rare events). Basketball scores are continuous, high-variance point totals. Discovered this the hard way — Poisson was outputting "NBA score: 2–1". The NBA pace model uses a direct 2-way odds conversion to point expectations with home court advantage baked in.

**Why multi-key Gemini rotation?**
QuickSlip runs 2–4 analyses in parallel. A single API key hits rate limits immediately. The rotation pool with per-key cooldown tracking handles concurrency transparently.

---

## Roadmap

- [ ] In-play live predictions
- [ ] AI-powered value bet scanner across all markets
- [ ] Native mobile app (React Native)
- [ ] Odds comparison across bookmakers
- [ ] Bankroll management calculator with Kelly Criterion
- [ ] Public accuracy leaderboard (platform-wide)

---

## Docs

- [Tech Stack →](docs/tech-stack.md)
- [Feature Reference →](docs/features.md)
- [Database Schema →](docs/database-schema.md)
- [Architecture Diagrams →](docs/architecture.md)

---

Built by [@pushthev1be](https://github.com/pushthev1be) &nbsp;|&nbsp; 🌐 [oracleai.live](https://oracleai.live)
