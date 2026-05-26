# Predicting the 2026 World Cup — First-Pass Approach

Status: design doc (v1, 2026-05-26). Companion to the `data/` dataset.

## 1. Objective

Two outputs:

1. **Match model** — for any pairing (A vs B) at a given venue, output
   probabilities of A win / draw / B win, plus an expected scoreline.
2. **Tournament forecast** — for each of the 48 teams, the probability of
   finishing top-2 in its group, reaching each knockout round, and winning the
   trophy. This comes from running the match model through a Monte Carlo
   simulation of the whole bracket.

Keep the two layers separate. The match model is the hard part; the tournament
forecast is "run the match model 50,000 times and count."

## 2. Architecture

```
 data/ (current state)        history corpus (to acquire)
        |                              |
        v                              v
   feature builder  <----------  Elo / ratings engine
        |                              |
        +--------------+---------------+
                       v
                 MATCH MODEL  (P(W/D/L), expected goals)
                       |
                       v
            TOURNAMENT SIMULATOR (Monte Carlo)
                       |
                       v
     per-team stage probabilities + champion odds  ->  dashboard
```

## 3. The match model — three options, in order of effort

### A. Elo baseline (start here)
Each team has a single strength rating. Win probability is a logistic function
of the rating gap plus a venue term:

```
P(A beats B) = 1 / (1 + 10^(-(EloA - EloB + H) / 400))
```

`H` is a home/neutral adjustment (host nations get a bump; everyone else plays
on neutral US/CA/MX soil). Draws are handled by splitting the win/loss curve or
with an ordered model. Pros: trivial to implement, generalizes to ANY pairing
(critical for knockout sims where we do not know matchups in advance), needs
only historical results to seed. Cons: ignores goals and squad detail.

### B. Bivariate Poisson goals model (recommended target)
Model each team's goals as Poisson with rates driven by attack and defence
strengths:

```
lambda_A = exp(mu + attackA - defenceB + home_adj)
lambda_B = exp(mu + attackB - defenceA)
```

Fit attack/defence per team (Dixon-Coles is the standard refinement, with a
correction for low-scoring draws and time-decay weighting so recent matches
count more). This gives a full scoreline distribution, so W/D/L, clean-sheet
and over/under all fall out. It is interpretable and a genuinely strong
baseline for football.

### C. Gradient-boosted classifier (the ML layer)
XGBoost/LightGBM predicting P(W/D/L) (or two regressors for each team's goals)
from the engineered feature vector below. Use it to beat the Poisson baseline
and to absorb features Elo/Poisson cannot (squad value, injuries, rest, travel).
Train on historical internationals, validate by Ranked Probability Score.

**Recommendation:** ship Elo end-to-end first (fastest path to a real champion
number), then upgrade the match model to Poisson + a GBM that blends in our
squad/form/H2H features. Treat "DL or agents" (the original ask) as a later
layer once the baseline is calibrated.

## 4. Features and where they come from

Per-match features are built from two team feature vectors plus match context.

| Feature | Source | Status |
|---|---|---|
| FIFA ranking | `team.json.fifa_ranking` | have |
| Squad market value (sum + top-XI) | `squad.json` aggregate | have |
| Squad age / caps / experience | `squad.json` aggregate | have |
| Share of players in top-5 leagues | `squad.json.club_league` | have (derivable) |
| Available vs injured ratio | `squad.json.status` | have |
| Recent form (PPG, GF, GA, streak) | `data/<slug>/form.json` | **adding now** |
| Head-to-head record | `data/h2h/<group>.json` | **adding now** |
| Elo rating | ratings engine over history | gap |
| Match venue / host advantage | `team.json` fixtures | partial |
| Travel distance, rest days | fixtures + venue coords | gap (derivable) |
| Altitude (Mexico City) / heat | venue metadata | gap |
| WC pedigree / past tournament runs | history corpus | gap |

The squad-derived aggregates (value, age, experience, top-league share,
availability) are the features that justify all the per-player data we already
collected, and they are computed by `scripts/build_features.py`.

## 5. Training data (the biggest gap)

The current `data/` is a **snapshot of the present** (who is in form, who is
worth what). To *train* a model you need **labelled history**: thousands of past
international results with goals. The standard, free option is the Kaggle
"International football results from 1872 to present" dataset (results,
goalscorers, shootouts). That corpus is used to:

- seed Elo ratings,
- fit Poisson attack/defence strengths,
- train and cross-validate the GBM,
- backtest on the 2018 and 2022 World Cups.

Acquiring and cleaning that corpus is the next real engineering task.

## 6. Tournament simulation (Monte Carlo)

One simulated tournament:

1. **Group stage** — simulate all 72 group matches with the match model.
   Rank each group by points, then the official tiebreakers (goal difference,
   goals for, head-to-head, then fair-play / drawing of lots). Take the top 2
   from each group plus the 8 best third-placed teams (32 advance).
2. **Knockout** — build the Round of 32 bracket from the fixed advancement map,
   simulate single matches; on a drawn 90 minutes, go to extra time then a
   penalty model (roughly a coin flip, optionally tilted by a team's shootout
   history). Advance the winner; repeat through the Final.
3. Record how far every team got.

Run ~50,000 times. Each team's champion probability is the fraction of
simulations it won; same for "reached QF", "won group", etc. Variance shrinks
with more runs; 50k gives stable two-decimal probabilities.

## 7. Evaluation

- **Match level:** Ranked Probability Score (RPS, the football-standard metric
  for ordered W/D/L), plus log-loss and Brier. Calibration plots (do 30%
  predictions happen ~30% of the time?).
- **Tournament level:** compare simulated champion odds to the betting market
  (a sanity check, not ground truth). Backtest the whole pipeline on WC 2018 and
  2022 using only pre-tournament data and see if the favourites land in the
  right probability band.
- Always hold out a time-based test set (train on pre-2022, test on 2022+); never
  shuffle-split football matches.

## 8. Known modelling pitfalls

- **Draws are hard.** A plain win-prob model under-predicts draws; use an ordered
  or Poisson model that produces a real draw probability.
- **Friendlies lie.** Weight competitive matches above friendlies; time-decay so
  2026 form outweighs 2023.
- **Neutral venues.** Almost every 2026 match is on neutral ground except the
  three hosts; do not bake in generic home advantage.
- **Squad churn.** Our squads are provisional; injuries (e.g. Xavi Simons out)
  must flow into the availability feature, not just be a footnote.
- **H2H is sparse.** Many pairings have few or zero meetings, so H2H is a small
  adjustment feature, never the backbone. Ratings carry the load for unseen
  pairings.

## 9. Roadmap

- [x] Phase 1 — current-state data: 48 squads, club stats, market values.
- [x] Phase 2 — recent form + group head-to-head.
- [ ] Phase 3 — acquire historical results corpus; build Elo engine; assemble the
      per-match feature matrix (`scripts/build_features.py` covers the team side).
- [ ] Phase 4 — Elo baseline + Monte Carlo simulator → first champion odds.
- [ ] Phase 5 — Poisson/Dixon-Coles + GBM; calibrate; backtest 2018/2022 vs market.
- [ ] Phase 6 (stretch) — DL / agentic ensemble, live in-tournament updating.

## 10. Immediate next steps

1. Run `scripts/build_features.py` to produce `data/features.csv` (team-level
   strength features from the data we already have).
2. Pull the historical results corpus and stand up the Elo engine.
3. Write the Monte Carlo group+knockout simulator against an Elo match model to
   get an end-to-end champion probability, however rough, then iterate on the
   match model.
