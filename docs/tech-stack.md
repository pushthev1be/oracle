# Oracle Odds AI — Full Tech Stack

## Frontend

| Technology | Version | Role |
|-----------|---------|------|
| React | 18 | UI framework |
| TypeScript | 5 | Type safety |
| Tailwind CSS | 4 | Styling |
| Vite | 5 | Build tool |
| Lucide React | latest | Icons |

### Frontend Architecture
- **Component-driven** — BetSlip, QuickSlip, AccuracyDashboard, Leaderboard all isolated
- **Context-based auth** — `useAuth()` hook wraps Supabase session state globally
- **Toast notification system** — module-level event emitter, zero dependencies, no re-renders
- **Skeleton loading** — shimmer placeholders for every async data boundary
- **Mobile-first** — 44px+ touch targets, safe-area insets, `touch-action: manipulation`
- **PKCE auth** — native Navigator Locks in production, no-op in dev (StrictMode safe)
- **Stable storage keys** — `oracle_vault_id_{uuid}` prevents slip loss across profile states

---

## Backend

| Technology | Role |
|-----------|------|
| Supabase Postgres | Primary database |
| Supabase Auth | Email OTP · Password · Google OAuth · PKCE flow |
| Supabase Edge Functions | Serverless compute (Deno) |
| Supabase RLS | Per-row security enforcement by subscription tier |
| pg_cron | Nightly automated settlement + scheduled social posts |

### Edge Functions

| Function | Trigger | Purpose |
|---------|---------|---------|
| `settle-predictions` | pg_cron 03:00 UTC | Settles pending predictions, updates league priors |
| `brain-sync` | On demand | Syncs league priors and team strengths from settled data |
| `odds-aggregator` | Scheduled | Pulls and caches odds from multiple sources |
| `precompute-analyses` | Scheduled | Pre-runs AI on upcoming fixtures for instant QuickSlip |
| `daily-social-post` | Cron daily | Posts picks to Twitter and Telegram automatically |
| `telegram-bot` | Webhook | Commands, account linking, pick delivery |
| `stripe-checkout` | Client-triggered | Creates Stripe checkout sessions |
| `stripe-webhook` | Stripe POST | Processes payments, updates subscription tier |
| `monitor-oracle` | Scheduled | Platform health checks and alerting |
| `betexplorer-scraper` | Scheduled | Scrapes supplementary odds data |
| `basketball-odds` | Scheduled | NBA-specific odds fetch and normalisation |
| `player-props` | On demand | Fetches and caches player prop markets |

---

## Payments

| Technology | Role |
|-----------|------|
| Stripe | Subscription billing |
| Stripe Webhooks | Tier updates on payment events |
| Supabase RLS | Tier enforcement at DB layer |

Idempotency guards on the webhook handler prevent duplicate tier changes from replayed events.

---

## Social & Notifications

| Service | Role |
|---------|------|
| Telegram Bot API | Daily picks delivery, account linking, VIP channel |
| Twitter API | Automated daily pick posts via `daily-social-post` |
| global_cache table | Audit log of every social post (`social_picks_YYYY-MM-DD` keys) |

---

## AI & Data

| Service | Usage |
|---------|-------|
| Gemini 2.5 Flash | Primary AI analysis with Google Search grounding |
| The Odds API | Live odds across 30+ sports/competitions |
| ESPN API | Live NBA scores and match events |
| Football-Data.org | European football fixtures and results |
| BetExplorer | Supplementary odds data |
| Pinnacle | Sharp odds reference |

### Multi-Key Rotation
Oracle manages a pool of Gemini API keys with:
- Per-project concurrency limits to avoid rate limits
- Cooldown tracking per key after 429 responses
- Progressive backoff with smart key selection
- Transparent to the user — QuickSlip's 4 parallel analyses never surface key failures

---

## Quant Engine

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
5. Outputs: homeLambda (points), awayLambda (points), predictedTotal, impliedSpread
```

### Tennis
```
1. Direct probability from moneyline odds
2. No score model — confidence derived from implied probability gap
```

### Adaptive Learning Loop
```
Prediction stored → Match settles → Residuals calculated
→ league_priors updated (avg goals drift, home bias, calibration)
→ team_strengths updated (attack/defense λ)
→ model_versions snapshot → best model promoted via tournament
```
