# Oracle Odds AI — Feature Reference

## Match Analysis Pipeline

Every analysis runs a 5-stage pipeline:

```
Stage 1: Odds Normalisation
  → Strip vig, find true fair probability per outcome

Stage 2: Quant Baseline
  → Football: Poisson expected goals
  → Basketball: NBA pace model point expectations
  → Tennis: Direct probability from odds

Stage 3: Adaptive Context Fetch (Supabase)
  → League priors: avg goals, home bias, calibration bias
  → Team strengths: attack/defense λ from settled history

Stage 4: AI Analysis (Gemini 2.5 Flash + Search)
  → Injuries, form, H2H, weather, team news — live from the web
  → Quant baseline anchors the AI's score prediction

Stage 5: Verdict + Storage
  → Confidence gauge, scoreline, best bet, quick picks
  → Prediction stored to ledger for future model training
```

---

## Subscription Tiers & Enforcement

Tiers are enforced at **three layers**:

1. **Frontend** — UI gates show upgrade prompt
2. **Application logic** — daily limit checked before running analysis
3. **Database (RLS)** — policies prevent direct API access to premium data

```
Free:    1 analysis/day  |  Major leagues only  |  No QuickSlip
Basic:   2 analyses/day  |  + lower leagues     |  QuickSlip (2 matches)
Premium: Unlimited       |  All leagues         |  QuickSlip (4 matches)
Elite:   Unlimited       |  All leagues         |  QuickSlip (4) + Telegram VIP
```

---

## QuickSlip — Batch Analysis

- Add 1–4 matches to a slip
- AI analyses all matches in parallel (rate-limited key rotation handles concurrency)
- Results saved as a receipt card to History vault
- Each match in the batch independently W/L settled

---

## History Vault

Every analysis is saved locally (localStorage) and synced to Supabase:
- **Receipt cards** — win/loss/pending badge, competition, scoreline, best bet
- **Confidence trend sparkline** — SVG chart of your last 10 analysis confidence scores
- **Win rate** — live-calculated from settled slips
- **Auto-settlement** — pg_cron settles pending slips nightly using official scores

---

## Accuracy Dashboard

Proof-of-performance tracker built for trust and transparency:

**Your Stats (personal)**
- Win rate % with colour-coded performance indicator
- Best win streak
- Recent form (last 10 as W/L dots)
- By-league win rate breakdown

**Platform Stats (from predictions_ledger)**
- Outcome accuracy % — correct winner/draw prediction
- Exact score rate % — how often the exact scoreline was called
- Average goals off per prediction
- Monthly trend bar chart
- By-league with progress bars

---

## Leaderboard

- Ranked by total predictions (DB-native count)
- Top 3 podium display
- Tier badges (Free/Basic/Premium/Elite)
- "YOU" indicator for current user
- Auto-refreshes on view navigation

---

## Mobile Experience

- Bottom navigation bar with safe-area padding (notch support)
- BetSlip as a full-screen bottom-sheet modal (90vh, scrollable)
- Match cards with 44px+ touch targets
- `touch-action: manipulation` removes 300ms tap delay globally
- Active states on all interactive elements (no hover-only interactions)
- Filter pills, sport category selector all horizontally scrollable

---

## Live Data

- Match list auto-refreshes every 2 minutes
- Live matches shown with red indicator strip
- Odds movement arrows (↑ green / ↓ red) when odds change between refreshes
- Match countdown timers for upcoming fixtures
- Status filter: All / Live / Upcoming / Finished
