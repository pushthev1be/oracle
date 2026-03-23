# Oracle Odds AI — Database Schema

## Core Tables

### `profiles`
Extends Supabase auth.users with Oracle-specific data.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid | Foreign key to auth.users |
| `username` | text | Display name |
| `email` | text | User email |
| `avatar_url` | text | Profile picture URL |
| `subscription_tier` | text | `free` / `basic` / `premium` / `elite` |
| `total_predictions` | int | Lifetime prediction count (leaderboard) |
| `predictions_today` | int | Daily count (resets via cron) |
| `last_prediction_at` | timestamptz | Last analysis timestamp |

**RLS policies:** Users can only read/write their own row. Leaderboard data (username, tier, total_predictions) is publicly readable.

---

### `predictions_ledger`
The core intelligence table. Every AI prediction is stored here.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid | Primary key |
| `match_id` | text | Reference to match |
| `model_version` | text | Which model version made this prediction |
| `predicted_home_goals` | float | Predicted home score |
| `predicted_away_goals` | float | Predicted away score |
| `actual_home_score` | float | Actual result (null until settled) |
| `actual_away_score` | float | Actual result (null until settled) |
| `home_residual` | float | actual - predicted (home) |
| `away_residual` | float | actual - predicted (away) |
| `total_error` | float | \|home_residual\| + \|away_residual\| |
| `prediction_status` | text | `exact` / `directional` / `miss` / null |
| `settled_at` | timestamptz | When the match result was confirmed |
| `created_at` | timestamptz | When the prediction was made |

**`prediction_status` values:**
- `exact` — predicted the exact scoreline
- `directional` — predicted the correct outcome (home win / draw / away win)
- `miss` — predicted the wrong outcome

---

### `match_history`
Settled match results used for model training.

| Column | Type | Description |
|--------|------|-------------|
| `id` | uuid | Primary key |
| `home_team_id` | text | Home team name |
| `away_team_id` | text | Away team name |
| `home_score` | int | Final home score |
| `away_score` | int | Final away score |
| `league` | text | Competition name |
| `sport` | text | `Soccer` / `Basketball` / `Tennis` |
| `match_date` | date | Match date |

---

### `league_priors`
Learned statistical priors per competition. Updated after every settlement.

| Column | Type | Description |
|--------|------|-------------|
| `league` | text | Primary key — competition name |
| `sport` | text | Sport type |
| `avg_goals` | float | Average total goals/points per match |
| `home_bias` | float | Home team scoring advantage |
| `draw_bias` | float | Draw frequency |
| `calibration_bias` | float | Model over/under-prediction bias |
| `avg_total_error` | float | Running average prediction error |
| `sample_size` | int | Number of settled matches used |
| `model_version` | text | Model that generated these priors |
| `last_updated` | timestamptz | Last recalibration timestamp |

---

### `team_strengths`
Per-team attack and defense ratings derived from match history.

| Column | Type | Description |
|--------|------|-------------|
| `team_id` | text | Team name (primary key) |
| `sport` | text | Sport type |
| `attack_lambda` | float | Expected scoring rate (goals or points) |
| `defense_lambda` | float | Expected conceding rate |
| `last_updated` | timestamptz | Last update |

---

### `model_versions`
Tracks performance of each model version for automatic promotion.

| Column | Type | Description |
|--------|------|-------------|
| `version_id` | text | Model version string (e.g. `v1.2-adaptive-learning`) |
| `avg_total_error` | float | Average prediction error across all settled matches |
| `exact_hit_rate` | float | Rate of exact score predictions |
| `sample_size` | int | Number of predictions in this version |
| `is_active` | bool | Whether this is the currently active model |
| `promoted_at` | timestamptz | When this version was promoted to active |
| `parameters_snapshot` | jsonb | Snapshot of league priors at promotion time |

---

### `math_cache`
Cached quant baselines per match to avoid recomputation.

| Column | Type | Description |
|--------|------|-------------|
| `match_id` | text | Primary key |
| `home_lambda` | float | Expected home scoring rate |
| `away_lambda` | float | Expected away scoring rate |
| `fair_home_prob` | float | Vig-removed home win probability |
| `fair_draw_prob` | float | Vig-removed draw probability |
| `fair_away_prob` | float | Vig-removed away win probability |
| `market_edge` | float | Oracle edge vs market |
| `created_at` | timestamptz | Cache timestamp |

---

## Settlement Flow

```sql
-- Nightly pg_cron job (runs at 03:00 UTC)
SELECT cron.schedule(
  'settle-predictions',
  '0 3 * * *',
  $$ SELECT net.http_post(url := 'https://<project>.supabase.co/functions/v1/settle-predictions') $$
);
```

1. Edge function fetches all `predictions_ledger` rows where `settled_at IS NULL`
2. For each, fetches actual result from `match_history`
3. Calculates residuals and `prediction_status`
4. Updates `predictions_ledger` with actuals
5. Triggers `updateLeaguePriorsFromSettledResults` for affected leagues
6. Triggers `refreshModelVersionSnapshot` to update accuracy metrics
7. Promotes best-performing model version
