# Oracle Odds AI

> **AI-powered sports prediction platform** — live odds, quantitative modelling, and machine learning that gets smarter with every settled match.

🌐 **[oracleai.live](https://oracleai.live)** &nbsp;|&nbsp; 📱 **[Telegram Channel](https://t.me/oracleoddsai)** &nbsp;|&nbsp; 🐦 **[@oracleoddsai](https://twitter.com/oracleoddsai)** &nbsp;|&nbsp; ⚡ React · Supabase · Gemini 2.5 Flash

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
| Top cities | Lagos (63) · Chicago (35) · Abuja (29) |
| Geo spread | Nigeria · USA · Ghana · UK |

*Source: Google Analytics, March 2026*

---

## What is Oracle?

Oracle Odds AI is a full-stack sports prediction platform built for bettors who want an edge backed by maths, not gut feel. It combines:

- **Live odds ingestion** from multiple bookmakers (The Odds API, BetExplorer, Pinnacle) — normalised and de-vigged before any analysis runs
- **Gemini 2.5 Flash AI** with Google Search grounding for real-time injury reports, team news, and form context — not stale training data
- **Proprietary quant engine** — Poisson/Dixon-Coles for football, NBA pace model for basketball, direct probability for tennis
- **Adaptive learning** — every settled match recalibrates league priors and team strength profiles, compounding accuracy over time
- **Tiered subscription** with Stripe payments and Supabase RLS enforcement: free users literally cannot query premium data even if they bypass the frontend
- **Telegram bot** — daily high-confidence picks and QuickSlip results delivered directly to subscribers, no app required

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
- Confidence gauge (0–100%) — composite of quant probability + AI grounding certainty
- Predicted scoreline per sport (realistic range enforced — AI can't output "2–1" for an NBA game)
- Best bet recommendation (moneyline / spread / O/U)
- Narrative signals — *Bayern's Dominance · Public Favorite Bias · UCL Knockout Stage*
- Oracle QuickPicks with verified confidence bars
- Odds movement arrows (↑ green / ↓ red) between refreshes

### QuickSlip — Batch Analysis
Add up to 4 matches to a slip and run AI analysis on all simultaneously:
- Parallel Gemini analyses (multi-key rotation handles the concurrency without rate limit failures)
- **Partial saves** — if 3/4 matches have precomputed analysis, the slip saves immediately with those 3; re-analyze to fill the 4th
- Results save as receipt cards in History Vault with W/L settlement per leg
- Kick-off date and time shown per game on the receipt

### History Vault
- Receipt-style cards with OracleOdds brand header
- Win/Loss/Pending badge per slip and per leg
- Predicted winner displayed ("Team X to win · 2–1") — not generic fallback text
- Match kickoff date under each game
- Confidence trend sparkline (last 10 analyses)
- Win rate, best streak, recent form dots
- Delete any slip with the trash icon
- Auto-settlement via pg_cron nightly at 03:00 UTC
- Syncs across devices via Supabase DB

### Global Pulse Dashboard
- Platform-wide total analyses counter
- Confidence yield simulation
- System edge vs closing line value
- Live analysis stream feed

### Accuracy Dashboard
Proof-of-performance tracker built for trust and transparency:

**Your Stats (personal)**
- Win rate % with colour-coded indicator
- Best win streak
- Recent form (last 10 as W/L dots)
- By-league win rate breakdown

**Platform Stats (from predictions_ledger)**
- Outcome accuracy % — correct winner/draw predictions
- Exact score rate % — how often the exact scoreline matched
- Average goals off per prediction
- Monthly trend bar chart
- By-league breakdown with progress bars

### Leaderboard
- Ranked by total predictions
- Top 3 podium display
- Tier badges (Free / Basic / Premium / Elite)
- "YOU" indicator for the signed-in user

### Telegram Integration
- Daily high-confidence picks posted automatically via `daily-social-post` edge function
- QuickSlip results delivered to subscribers when analyses complete
- Inline sport emoji tags and confidence scores in every post
- Connect from the in-app header icon — link your account for personalised alerts

---

## Sports Coverage

| Sport | Competitions |
|-------|-------------|
| Football | Premier League · La Liga · Serie A · Bundesliga · Ligue 1 · UCL · Championship + 15 more |
| Basketball | NBA |
| Tennis | ATP · WTA · Grand Slams |

---

## Subscription Tiers

| | Free | Starter Pro | Elite Data | Oracle Elite |
|-|------|-------------|------------|--------------|
| **Price** | $0/mo | $2/mo | $10/mo | $20/mo |
| **AI Predictions/day** | 1 | 3 | 10 | Unlimited |
| **QuickSlip** | — | 2 slots | Unlimited | Unlimited |
| **Leagues** | Major only | Mid-tier | All sports | All sports |
| **Extras** | — | — | — | High-confidence toggle · Telegram Alpha · VIP Bankroll |

Tier enforcement is applied at **three layers**: frontend UI gates → application logic → Supabase RLS. A free user literally cannot query premium data even if they bypass both of the first two layers.

---

## Tech Stack

### Frontend
- **React 18** + TypeScript — component-driven, context-based auth
- **Tailwind CSS 4** with custom dark theme (`#04040a` base, green/black/slate brand)
- **Vite 5** build tool
- Zero-dependency toast system (module-level event emitter)
- Skeleton loading for every async data boundary
- Mobile-first: 44px+ touch targets, safe-area insets, `touch-action: manipulation`
- Chrome mobile PKCE auth fix — native Navigator Locks in production, prevents concurrent code exchange invalidation

### Backend & Infrastructure
- **Supabase** — Postgres, Auth (PKCE flow), Edge Functions (Deno), Row Level Security
- **pg_cron** — nightly auto-settlement of predictions at 03:00 UTC
- **Stripe** — subscription billing with webhook-driven tier updates and idempotency guards

### AI & Data
- **Gemini 2.5 Flash** — primary AI analysis with Google Search grounding
- **The Odds API** — live odds across 30+ sports
- **ESPN API** — live NBA scores and events
- **Football-Data.org** — European football fixtures
- **BetExplorer + Pinnacle** — sharp odds reference for line comparison

### Multi-Key Rotation
Oracle manages a pool of Gemini API keys with per-project concurrency limits, cooldown tracking per key after 429 responses, and progressive backoff with smart key selection — QuickSlip's 4 parallel analyses run without hitting rate limits.

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
| `profiles` | User accounts, subscription tier, prediction counts, Telegram ID |
| `predictions_ledger` | Every AI prediction with actual outcome after settlement |
| `match_history` | Settled match results with team names and sport source |
| `league_priors` | Per-league learned averages (goals, home bias, calibration bias) |
| `team_strengths` | Per-team attack/defense lambdas from settled match history |
| `model_versions` | Model performance snapshots (exact hit rate, avg error, promotion status) |
| `math_cache` | Quant baselines cached per match — avoids recomputing expensive Poisson steps |
| `user_slips` | QuickSlip batch history synced from localStorage for cross-device access |
| `global_cache` | Shared odds/analysis cache, social post logs, precomputed analyses |

See [full schema docs →](docs/database-schema.md)

---

## Edge Functions

| Function | Trigger | Purpose |
|---------|---------|---------|
| `settle-predictions` | pg_cron 03:00 UTC | Settles pending predictions against actual scores, updates league priors |
| `brain-sync` | On demand | Syncs league priors and team strengths from settled match data |
| `odds-aggregator` | Scheduled | Pulls and caches odds from The Odds API, BetExplorer, Pinnacle |
| `precompute-analyses` | Scheduled | Pre-runs AI analysis on upcoming high-profile matches for instant QuickSlip |
| `daily-social-post` | Cron daily | Posts high-confidence picks to Twitter and Telegram automatically |
| `telegram-bot` | Webhook | Handles Telegram commands, account linking, and pick delivery |
| `stripe-checkout` | Client-triggered | Creates Stripe checkout sessions for tier upgrades |
| `stripe-webhook` | Stripe POST | Processes payment events, updates `profiles.subscription_tier` |
| `monitor-oracle` | Scheduled | Platform health checks, odds freshness monitoring, alerting |
| `betexplorer-scraper` | Scheduled | Scrapes supplementary odds data from BetExplorer |
| `basketball-odds` | Scheduled | NBA-specific odds fetch and normalisation |
| `player-props` | On demand | Fetches and caches player prop markets |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                        CLIENT                           │
│  React + TypeScript + Tailwind                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │  Lobby   │ │QuickSlip │ │ History  │ │Accuracy  │  │
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
│  │  + RLS   │  │  (Data)  │  │ settle / social-post │  │
│  └──────────┘  └──────────┘  │ telegram / stripe    │  │
│                               └──────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                   EXTERNAL APIs                         │
│  The Odds API │ ESPN │ Football-Data │ Gemini Search    │
│  Stripe │ Telegram Bot API │ BetExplorer │ Pinnacle     │
└─────────────────────────────────────────────────────────┘
```

---

## Platform Scale

| Metric | Detail |
|--------|--------|
| Database tables | 9 core tables + RLS policies on all |
| Edge functions | 11 deployed functions |
| Sports covered | Football (20+ leagues) · Basketball (NBA) · Tennis (ATP/WTA/Slams) |
| Odds sources | 3+ (The Odds API · BetExplorer · Pinnacle) |
| Cron schedule | Nightly settlement + daily social posts |
| Concurrency | QuickSlip: 4 parallel Gemini analyses per request |
| API key pool | Multi-key rotation with per-key cooldown after 429 |
| Tier enforcement layers | 3 (frontend → app logic → Supabase RLS) |
| Auth flows | OTP (magic link) · Password · Google OAuth · Forgot/Reset |

---

## Engineering Challenges

### The Basketball Score Bug
**Problem:** Users were getting NBA predictions like "predicted score: 2–1". The model was broken for basketball.

**Root cause:** The Poisson distribution is designed for low-count rare events (football goals average 1.2–1.8 per team). Feeding NBA moneyline odds into the same Poisson pipeline produced lambdas of ~1.8 — a soccer score, not a basketball score.

**Fix:** Built a completely separate `deriveBasketballBaseline()` function using a pace model:
```typescript
// mathService.ts — NBA pace model
const spread = (homeWinProb - 0.5) * AVG_TOTAL * 0.12;
const homeLambda = Math.round(avgPerTeam + spread + HOME_BIAS / 2);
const awayLambda = Math.round(avgPerTeam - spread - HOME_BIAS / 2);
// Result: 108 vs 103 instead of 2 vs 1
```

---

### Chrome Mobile PKCE Auth Failure
**Problem:** Users on Chrome mobile were redirected back to the sign-in screen after authenticating. Other browsers worked fine.

**Root cause:** The Supabase client had a no-op lock override to prevent React StrictMode double-mount deadlocks in development. In production on Chrome mobile, this allowed two concurrent PKCE code exchanges to run (tab restore + service worker). The second invalidated the first — session came back null.

**Fix:** Restore native `navigator.locks` in production; keep the no-op only in dev:
```typescript
const isDev = process.env.NODE_ENV === 'development';
const authLock = isDev
  ? (_name, _timeout, fn) => fn()  // no-op for StrictMode safety
  : undefined;                      // native locks in prod
```

---

### Slip Disappearing on Mobile Refresh
**Problem:** Users' most recent slip vanished after a page refresh on mobile.

**Root cause:** The localStorage key was `oracle_vault_${username}`. On mobile, the profile sometimes loads transiently (username derived from email prefix) before the DB profile arrives (with a different username). The key changed between sessions — slips stored under one key were invisible when read under another.

**Fix:** Switched to a stable UUID-based key (`oracle_vault_id_${user.id}`) with a one-time migration from the legacy key.

---

### Supabase Auth Lock Contention
**Problem:** Concurrent requests during QuickSlip (4 parallel analyses) hit Supabase auth lock contention — session token refresh collided with itself under concurrency.

**Fix:** Separated the Supabase client into two instances — one with auth (user-scoped queries) and one without (`supabaseNoAuth`) for read-only public data. Parallel analyses use the no-auth client, eliminating the contention.

---

### LLM Score Hallucination
**Problem:** Without constraints, Gemini occasionally produced wildly off-base scorelines.

**Fix:** Quant layer runs first, producing mathematically grounded lambdas injected as hard anchors in the Gemini prompt:
```
QUANT BASELINE (treat as ground truth):
- Home expected goals: 1.42 | Away: 0.98
- Fair probabilities: Home 58% | Draw 22% | Away 20%
- Your predicted score MUST be within ±0.8 goals of this baseline
```
Basketball adds: *"BASKETBALL SCORES MUST BE 95–130 per team — outputting '2–1' is a CRITICAL ERROR."*

---

### Fixture Loading (45s → <5s)
**Problem:** The match list took 45+ seconds to load on first visit.

**Root cause:** Three external API calls (odds, scraped odds, aggregated odds) were sequential `await` calls — each waited for the previous before starting.

**Fix:** `Promise.all` with hard timeouts — all three run in parallel with 5s cutoffs each:
```typescript
const [oddsMap, scrapedOdds, aggregatedOdds, syncCache] = await Promise.all([
  withTimeout(fetchAllOdds(force), 5_000, new Map()),
  withTimeout(fetchScrapedOdds(force), 5_000, {}),
  withTimeout(fetchAggregatedOdds(force), 5_000, {}),
  withTimeout(getCachedFixtures(), 5_000, null),
]);
```

---

### Zero-Dependency Toast System
**Problem:** `alert()` blocks the main thread. React Context-based toasts add re-render overhead.

**Fix:** Module-level event emitter:
```typescript
// services/toast.ts
let _handler: ToastHandler | null = null;

export const toast = {
  success: (msg: string) => _handler?.(msg, 'success'),
  error:   (msg: string) => _handler?.(msg, 'error'),
  warning: (msg: string) => _handler?.(msg, 'warning'),
};
// ToastContainer.tsx registers _handler once on mount
// Any module can call toast.error() with zero imports
```

---

### Tier Enforcement at 3 Independent Layers
Free users bypassing the frontend still can't access premium data:
```sql
CREATE POLICY "tier_gate_league_priors"
ON league_priors FOR SELECT
USING (
  is_major_league(league)
  OR
  (SELECT subscription_tier FROM profiles WHERE id = auth.uid())
    IN ('basic', 'premium', 'elite')
);
```
The frontend shows an upgrade prompt, the application checks the tier before running analysis, and the database refuses the query regardless. All three must be bypassed simultaneously.

---

## Key Engineering Decisions

**Why Supabase over Firebase?**
Row Level Security enforces subscription tiers at the database level. Firebase has no equivalent per-row security primitive.

**Why Gemini 2.5 Flash with Search Grounding?**
Sports predictions go stale within hours. Static training data is useless for "who's injured tonight". Grounded search queries live sources on every call.

**Why a custom quant layer instead of just prompting AI?**
LLMs hallucinate scores. The quant baseline anchors the AI to mathematically sound probability ranges. Neither alone is as accurate as both together.

**Why separate models for football vs basketball?**
Poisson models goals well (0–5 range). Basketball scores are continuous, high-variance point totals. Proved empirically when Poisson started outputting NBA scores of "2–1".

**Why stable UUID storage keys?**
Username-derived localStorage keys break when profile state is transient (especially on mobile Chrome where the DB profile loads asynchronously after auth). UUID never changes.

---

## Code Highlights

### Adaptive Learning — League Prior Update
```typescript
async function updateLeaguePriors(league: string, residuals: Residual[]) {
  const avgError = residuals.reduce((s, r) => s + r.totalError, 0) / residuals.length;
  const homeBiasDelta = residuals.reduce((s, r) => s + r.homeResidual, 0) / residuals.length;

  await supabase.from('league_priors').upsert({
    league,
    avg_total_error: avgError,
    home_bias: existingPrior.home_bias + homeBiasDelta * LEARNING_RATE,
    calibration_bias: computeCalibrationBias(residuals),
    sample_size: existingPrior.sample_size + residuals.length,
    last_updated: new Date().toISOString(),
  });
}
```

### Odds Normalisation — Vig Removal
```typescript
function removeVig(homeOdds: number, drawOdds: number, awayOdds: number) {
  const impliedHome = 1 / homeOdds;
  const impliedDraw = drawOdds ? 1 / drawOdds : 0;
  const impliedAway = 1 / awayOdds;
  const overround = impliedHome + impliedDraw + impliedAway; // e.g. 1.06 = 6% vig

  return {
    fairHome: impliedHome / overround,
    fairDraw: impliedDraw / overround,
    fairAway: impliedAway / overround,
  };
}
```

### Model Version Tournament
```typescript
async function promotebestModel() {
  const { data: versions } = await supabase
    .from('model_versions')
    .select('*')
    .gte('sample_size', MIN_SAMPLE_FOR_PROMOTION)
    .order('avg_total_error', { ascending: true })
    .limit(1);

  if (versions?.[0] && !versions[0].is_active) {
    await supabase.from('model_versions')
      .update({ is_active: false }).neq('version_id', versions[0].version_id);
    await supabase.from('model_versions')
      .update({ is_active: true, promoted_at: new Date() })
      .eq('version_id', versions[0].version_id);
  }
}
```

---

## Roadmap

- [ ] In-play live predictions
- [ ] AI-powered value bet scanner across all markets
- [ ] Native mobile app (React Native)
- [ ] Odds comparison across bookmakers
- [ ] Bankroll management calculator with Kelly Criterion
- [ ] Public accuracy leaderboard (platform-wide)
- [ ] NFL and MLB coverage expansion
- [ ] Historical back-testing interface

---

## Docs

- [Tech Stack →](docs/tech-stack.md)
- [Feature Reference →](docs/features.md)
- [Database Schema →](docs/database-schema.md)
- [Architecture Diagrams →](docs/architecture.md)

---

Built by [@pushthev1be](https://github.com/pushthev1be) &nbsp;|&nbsp; 🌐 [oracleai.live](https://oracleai.live) &nbsp;|&nbsp; 📱 [Telegram](https://t.me/oracleoddsai) &nbsp;|&nbsp; 🐦 [@oracleoddsai](https://twitter.com/oracleoddsai)
