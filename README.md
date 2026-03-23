# Oracle Odds AI

> **AI-powered sports prediction platform** — live odds, quantitative modelling, and machine learning that gets smarter with every settled match.

🌐 **[oracleodds.ai](https://oracleodds.ai)** &nbsp;|&nbsp; ⚡ Built with React, Supabase, Gemini AI

---

<div align="center">

![Oracle Odds AI](assets/soccer_player.png)

### The sports intelligence layer that professional bettors don't want you to have.

</div>

---

## What is Oracle?

Oracle Odds AI is a full-stack sports prediction platform that combines:

- **Live odds ingestion** from multiple bookmakers (The Odds API, BetExplorer, Pinnacle)
- **Gemini 2.5 Flash AI** with grounded web search for real-time injury/form context
- **Proprietary quant engine** — Poisson/Dixon-Coles for football, NBA pace model for basketball
- **Adaptive learning** — every settled match updates the model's league priors and team strength profiles
- **Tiered subscription system** with Stripe payments and Supabase RLS enforcement

---

## Core Features

### Match Intelligence
Analyse any upcoming match across Football, Basketball, and Tennis. Oracle runs a multi-layer pipeline:

1. **Live odds normalisation** — strips bookmaker vig to find true fair probability
2. **Quant baseline** — Poisson (football) or NBA pace model (basketball) derives expected scoring
3. **AI analysis** — Gemini 2.5 Flash searches live for injuries, form, H2H, team news
4. **Adaptive context** — league priors and team strength profiles from the settled match ledger
5. **Expert verdict** — confidence gauge, predicted scoreline, best bet, likely scorers

### Sports Coverage
| Sport | Competitions |
|-------|-------------|
| ⚽ Football | Premier League, La Liga, Serie A, Bundesliga, Ligue 1, UCL, Championship + 15 more |
| 🏀 Basketball | NBA |
| 🎾 Tennis | ATP, WTA, Grand Slams |

### QuickSlip — Multi-Match Batch Analysis
Add up to 4 matches to a slip and run AI analysis on all of them simultaneously. Results are saved as a receipt card with W/L settlement tracking.

### Accuracy Tracker
Every prediction is logged to a Supabase ledger. As matches settle the platform tracks:
- **Outcome accuracy** (correct winner/draw %)
- **Exact score rate**
- **Average goals off** per prediction
- **By-league breakdown** — know which competitions Oracle calls best
- **Monthly trend** — see the model improve over time

### Leaderboard
Global rankings by total predictions. Elite tier users get a private Telegram community.

---

## Subscription Tiers

| Tier | Daily Analyses | QuickSlip | Leagues |
|------|---------------|-----------|---------|
| Free | 1 | ✗ | Major only |
| Basic | 2 | 2 matches | + lower leagues |
| Premium | Unlimited | 4 matches | All |
| Elite | Unlimited | 4 matches | All + VIP Telegram |

---

## Tech Stack

### Frontend
- **React 18** + TypeScript
- **Tailwind CSS** with custom dark theme (`#04040a` base)
- **Lucide React** icons
- Animated particle background, skeleton loading, toast notifications
- Fully responsive — mobile-first with safe-area nav

### Backend & Infrastructure
- **Supabase** — Postgres database, Auth, Edge Functions, Row Level Security
- **Supabase Cron** — auto-settles predictions nightly via pg_cron
- **Gemini 2.5 Flash** — AI analysis with Google Search grounding
- **The Odds API** — live odds across 30+ sports
- **ESPN API** — live scores and match data
- **Football-Data.org** — European football fixtures

### Quantitative Engine
```
Odds Normalisation  →  Fair Probability (vig removed)
       ↓
Sport Detection     →  Football: Poisson/Dixon-Coles
                        Basketball: NBA Pace Model (2-way odds)
                        Tennis: Direct probability
       ↓
Team Strength       →  Attack/Defense λ from settled match history
       ↓
Adaptive Context    →  League priors (avg goals, home bias, calibration bias)
       ↓
AI Prompt           →  Quant baseline + web-grounded analysis
       ↓
Predictions Ledger  →  Settlement → model retraining loop
```

### Database Schema (Key Tables)
| Table | Purpose |
|-------|---------|
| `profiles` | User accounts, subscription tier, prediction counts |
| `predictions_ledger` | Every AI prediction with actual outcome after settlement |
| `match_history` | Settled match results |
| `league_priors` | Per-league learned averages (goals, home bias, calibration) |
| `team_strengths` | Per-team attack/defense lambdas |
| `model_versions` | Model performance snapshots (exact hit rate, avg error) |
| `math_cache` | Quant baselines cached per match |

---

## Machine Learning Loop

Oracle gets smarter every day through a closed feedback loop:

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

This means Oracle's accuracy **compounds** — more predictions = more settled data = better baselines.

---

## Platform Athletes

<div align="center">
<img src="assets/pulisic.png" width="150" alt="Pulisic" />
&nbsp;&nbsp;&nbsp;
<img src="assets/alexander_arnold.png" width="150" alt="Alexander-Arnold" />
&nbsp;&nbsp;&nbsp;
<img src="assets/basketball_player.png" width="150" alt="Basketball" />
&nbsp;&nbsp;&nbsp;
<img src="assets/tennis_player.png" width="150" alt="Tennis" />
</div>

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
│  │  (Quant AI) │  │  (Gemini AI)│  │ Service     │    │
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
Row Level Security lets us enforce subscription tiers at the database level — a free user literally cannot query premium data even if they bypass the frontend.

**Why Gemini 2.5 Flash with Search Grounding?**
Sports predictions go stale within hours. Grounded search means every analysis queries live injury reports, team news, and form — not static training data from months ago.

**Why a custom quant layer instead of just prompting the AI?**
LLMs hallucinate scores. The quant baseline anchors the AI to mathematically sound probability ranges derived from actual market odds. The AI adds context the quant can't see (injuries, psychology, weather). Neither alone is as good as both together.

**Why separate models for football vs basketball?**
Poisson distribution models goals well (0–5 range, rare events). Basketball scores are continuous, high-variance point totals. The NBA model uses a direct 2-way odds conversion to point expectations with home court advantage baked in — Poisson would produce nonsense like "predicted score: 2–1" for an NBA game.

---

## Roadmap

- [ ] In-play live predictions
- [ ] AI-powered value bet scanner across all markets
- [ ] Public accuracy leaderboard (platform-wide, not just user)
- [ ] Native mobile app (React Native)
- [ ] Odds comparison across bookmakers
- [ ] Bankroll management calculator with Kelly Criterion

---

## Contact

Built and maintained by [@pushthev1be](https://github.com/pushthev1be)

🌐 [oracleodds.ai](https://oracleodds.ai)
