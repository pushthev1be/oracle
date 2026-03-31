# Oracle Odds AI — Feature Reference

## Match Analysis Pipeline

Every analysis runs a 5-stage pipeline:

```
Stage 1: Odds Normalisation
  → Strip vig, find true fair probability per outcome

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

---

## Subscription Tiers & Enforcement

Tiers are enforced at **three independent layers**:

1. **Frontend** — UI gates show upgrade prompt
2. **Application logic** — daily limit checked before running analysis
3. **Database (RLS)** — policies block direct API access to premium data

```
Free:    1 analysis/day  |  Major leagues only  |  No QuickSlip
Basic:   3 analyses/day  |  + lower leagues     |  QuickSlip (2 matches)
Premium: 10 analyses/day |  All leagues         |  QuickSlip (4 matches)
Elite:   Unlimited       |  All leagues         |  QuickSlip (4) + Telegram Alpha + VIP Bankroll
```

---

## QuickSlip — Batch Analysis

- Add 1–4 matches to a slip
- AI analyses all matches in parallel (multi-key rotation handles concurrency transparently)
- **Partial saves**: if only 3/4 matches have precomputed analysis, the slip saves immediately with those 3. Re-analyze to fill the rest — the partial receipt is replaced automatically
- Results saved as receipt cards in History Vault
- Each leg independently W/L settled by the nightly cron

---

## History Vault

Every analysis is saved locally (stable UUID-based key in localStorage) and synced to Supabase `user_slips`:

- **Receipt cards** — OracleOdds logo header, win/loss/pending badge, competition name
- **Winner display** — "Team X to win · 2–1" derived from predicted scores, not generic fallback text
- **Match dates** — kickoff date/time shown per game in batch slips
- **Per-leg W/L badges** — W (green) · L (red) · TBD (slate) on each game in a multi-match slip
- **Delete button** — remove any slip from history and DB with the trash icon
- **Confidence trend sparkline** — SVG chart of last 10 analysis confidence scores
- **Win rate** — live-calculated from settled slips
- **Auto-settlement** — pg_cron settles pending slips nightly using official scores
- **Cross-device sync** — DB is authoritative; localStorage is a fast-load cache

---

## Accuracy Dashboard

Proof-of-performance tracker:

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
- Tier badges (Free / Basic / Premium / Elite)
- "YOU" indicator for current user
- Auto-refreshes on view navigation

---

## Telegram Integration

- Daily high-confidence picks posted automatically at a scheduled time
- QuickSlip results delivered directly to linked subscribers
- Inline sport emoji tags, confidence scores, and predicted scores in every message
- Account linking from the in-app header icon
- Elite tier gets access to VIP Alpha channel with higher-confidence picks only
- Powered by the `telegram-bot` and `daily-social-post` edge functions

---

## Social Media

Daily posts to [@oracleoddsai](https://twitter.com/oracleoddsai) on Twitter via the `daily-social-post` edge function:
- Top picks for the day with confidence scores
- Sport-specific emojis and formatting
- Logged to `global_cache` with `social_picks_YYYY-MM-DD` keys for audit trail

---

## Auth Flows

| Method | Description |
|--------|-------------|
| OTP (Email Code) | 6-digit code sent to inbox — works without a password |
| Password | Sign up / sign in with email + password |
| Google OAuth | One-click Google sign-in via Supabase OAuth |
| Forgot Password | Reset link emailed, opens in-app reset form |
| Reset Password | Set new password after clicking email link |

PKCE flow with native Navigator Locks in production — Chrome mobile compatible.

---

## Mobile Experience

- Bottom navigation bar with safe-area padding (notch support)
- BetSlip as a full-screen bottom-sheet modal (scrollable)
- Match cards with 44px+ touch targets
- `touch-action: manipulation` removes 300ms tap delay globally
- Active states on all interactive elements (no hover-only interactions)
- Filter pills and sport category selector horizontally scrollable
- Chrome mobile sign-in fixed — PKCE lock contention resolved

---

## Live Data

- Match list auto-refreshes every 2 minutes
- Parallel fixture fetches with 5–8s hard timeouts (worst-case load under 10s vs 45s previously)
- Live matches shown with red indicator strip
- Odds movement arrows (↑ green / ↓ red) when odds change between refreshes
- Match countdown timers for upcoming fixtures
- Status filter: All / Live / Upcoming / Finished
