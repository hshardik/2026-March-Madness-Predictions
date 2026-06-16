# 🏀 March Madness Datathon 2026

**Team: Cushing Street Analytics**

A data-driven analysis of NCAA March Madness tournament performance (2008–2025) and prediction model for the 2026 tournament.

---

## Project Overview

This project analyzes 18 years of NCAA Tournament data to answer three questions:

1. **Which seeds do well or poorly?** Are there teams that consistently over- or under-perform their seeding?
2. **What attributes predict tournament success?** Which stats actually matter in March?
3. **Who will win the 2026 tournament?** Can we predict the Final Four from regular-season data alone?

We built a Gradient Boosting model trained on 1,152 team-year records across 34 features (SEED deliberately excluded) and validated it with Leave-One-Year-Out cross-validation across all 18 tournament years.

**Key result:** Mean Absolute Error of 0.75 wins — the model is off by less than one tournament round per team on average.

**2026 Final Four prediction:** Michigan (5.29), Duke (4.91), Illinois (4.41), Arizona (4.26). Champion: Michigan.

---

## Repository Structure

```
├── March_Madness_Datathon_v4_no_seed_live16.ipynb   # Main analysis notebook (Phases 1–5)
├── Cushing_Street_Analytics.pptx                     # Presentation deck (with speaker notes)
├── March_Madness_Executive_Summary.docx              # One-page executive summary
├── QA_Prep_Guide_Panel_Questions.docx                # Anticipated judge Q&A with responses
├── predictions_2026_live_16_no_seed.csv              # Model predictions for 16 live teams
├── seed_performance_2008_2025.csv                    # Seed-level historical performance
├── team_over_under_performers.csv                    # WAE rankings by program
├── march_madness/                                    # Raw Kaggle dataset (extracted)
│   ├── KenPom Barttorvik.csv                         # Primary dataset (103 cols × 1,215 rows)
│   ├── Resumes.csv                                   # Team resume metrics
│   ├── Team Results.csv                              # Program-level aggregated results
│   ├── Seed Results.csv                              # Seed-level aggregated results
│   ├── Coach Results.csv                             # Coach tournament performance
│   ├── Tournament Matchups.csv                       # Bracket structures with scores
│   └── ...                                           # Additional supporting files
└── README.md
```

---

## Data Source

**Kaggle Dataset:** [March Madness Data](https://www.kaggle.com/datasets/nishaanamin/march-madness-data) by Nishaan Amin

The dataset contains 38 CSV files. Our primary analytical dataset is `KenPom Barttorvik.csv`, which provides 103 columns of advanced basketball statistics per team-year sourced from [KenPom](https://kenpom.com/) and [Barttorvik](https://barttorvik.com/).

### Why 2008–2025 (not 1985)?

We deliberately used the 2008–2025 window rather than the full history back to 1985. The game has fundamentally changed:

- The three-point line moved (19'9" → 20'9" in 2008 → 22'1.75" in 2019)
- The shot clock shortened (45s → 35s in 1993 → 30s in 2015)
- The transfer portal (2018) and NIL (2021) reshaped roster construction
- The one-and-done rule (2006) changed which players appear in college
- The advanced metrics we rely on didn't exist in reliable form before ~2008

Eighteen years of consistent, modern-era data produces better predictions than forty years of mixed-era data from a sport that no longer resembles its 1985 version.

---

## Methodology

### Phase 1 — Data and Target Variable

We identified KenPom Barttorvik as our primary dataset — 103 columns of advanced stats per team-year. We supplemented it with five supporting datasets: team resume metrics, program-level results, seed-level results, coach tournament performance, and bracket structures with scores.

From this, we derived our target variable: **WINS**. The tournament uses a ROUND encoding that tells us where each team was eliminated, and we converted that into a simple win count from 0 to 6. When we looked at the WINS distribution across all completed years (2008–2025), we found that 603 out of 1,147 team-years — more than half — don't advance past the first round. That tells you how hard it is to win even a single game in this tournament.

| ROUND | Meaning | WINS |
|:-----:|---------|:----:|
| 64 | Lost in Round of 64 | 0 |
| 32 | Lost in Round of 32 | 1 |
| 16 | Lost in Sweet 16 | 2 |
| 8 | Lost in Elite 8 | 3 |
| 4 | Lost in Final Four | 4 |
| 2 | Lost in Finals | 5 |
| 1 | Champion | 6 |

### Phase 2 — Seed Performance and WAE

We performed a seed-level performance analysis. For each seed 1 through 16, we calculated average wins per tournament, advancement rates by round, and championship rate. The results confirmed what basketball fans suspect but the data makes undeniable: 97% of 1-seeds advance past the Round of 64, 82.4% make the Sweet 16, and 19.1% win it all. 2-seeds start strong but drop off sharply after the Sweet 16.

To identify which specific schools over- or under-perform, we created a metric called **Wins Above Expectation (WAE)** — actual wins minus the historical average for that seed. This separates programs that consistently exceed their seeding from programs that consistently disappoint.

| Over-performers | WAE | Under-performers | WAE |
|----------------|:---:|-----------------|:---:|
| UConn | +13.3 | Georgetown | −8.1 |
| Michigan St. | +10.1 | Virginia | −7.9 |
| Butler | +9.2 | Vanderbilt | −6.3 |
| Michigan | +8.5 | Pittsburgh | −4.8 |
| North Carolina | +8.2 | Kansas | −3.3 |

We verified our WAE calculation against the PASE (Performance Against Seed Expectation) column already present in the Barttorvik dataset. Both methods produced the same rank-order results.

### Phase 3 — Feature Selection and Importance

From the 103 columns available, we selected the 24 most relevant features based on efficiency, tempo, win rate, and team composition. We also pulled the 7 strongest features from the team resume dataset. On top of those, we engineered custom metrics for conference strength, prior tournament experience, and historical team performance — giving us **35 total features**.

We then trained a Random Forest to predict how far a team advances and extracted the feature importance scores. We also ran a correlation analysis to understand each feature's linear relationship with tournament wins.

**Key finding:** In-game shooting stats (EFG%, three-point rate, free throw rate, turnover rate, tempo) rank near the bottom of feature importance. What actually drives deep runs is the résumé — who you played, whether you beat them, and how consistently you did it. The tournament doesn't reward style. It rewards preparation.

**Top predictive features:**
- Resume Score (R SCORE) — who you played and whether you beat them
- Wins Above Bubble (WAB) — quality wins against good opponents
- BARTHAG — probability of beating an average D1 team
- Adjusted Efficiency Margin (KADJ EM / BADJ EM) — points per 100 possessions, adjusted for opponent
- Conference strength and prior tournament experience (engineered)

### Phase 4 — Model and Validation

We used a **Gradient Boosting Regressor** to predict tournament wins. SEED was deliberately removed from the feature set to force the model to learn from actual basketball performance rather than the Selection Committee's ranking — leaving 34 input features.

To make sure the model isn't just memorizing past results, we validated it with **Leave-One-Year-Out cross-validation**: train on 17 years, predict the 1 year left out, repeat 18 times. Each iteration produces a separate accuracy score, and we average all 18 to get our final metric. The score is not a fluke — it's the average of 18 independent tests.

| Metric | Value | Interpretation |
|--------|:-----:|----------------|
| Mean Absolute Error | 0.75 wins | Prediction off by less than one round per team |
| R² | 0.33 | Model explains 33% of variance in tournament outcomes |
| Cross-validation | 18-fold LOYO | Tested on every year independently |

**Note on MAE interpretation:** MAE of 0.75 means the model's prediction is off by 0.75 wins on average. This is not the same as saying it "predicts 75% of outcomes correctly" — those are different metrics. If the model predicts a team will win 3 games, they typically win 2 or 4.

**Why R² = 0.33 is meaningful:** Single-elimination tournaments are inherently high-variance. One bad shooting night ends your season. A model explaining 33% of outcomes from pre-tournament data alone is extracting real signal from an extremely noisy system. If R² were 0.80, the tournament would be predictable — and that contradicts everything we know about March Madness.

### Phase 5 — 2026 Prediction

We trained the final model on all 1,152 historical rows and ran it on the **16 teams still alive** in the 2026 tournament. Predictions use the same feature pipeline as training: engineer conference strength and prior experience, merge resume data, apply the same scaler (not refitted), and predict.

**Predicted Final Four:**

| Region | Team | Seed | Predicted Wins |
|--------|------|:----:|:--------------:|
| East | **Michigan** | #1 | 5.29 |
| Midwest | Duke | #1 | 4.91 |
| South | Illinois | #3 | 4.41 |
| West | Arizona | #1 | 4.26 |

**Predicted champion:** Michigan over Duke.

**Upset watch:**
- **#3 Illinois** — highest Wins Above Expectation of any live team; the model sees them as a 1-seed in disguise
- **#5 St. John's** — projected above their seed line
- **#4 Alabama** — projected to outperform a typical 4-seed

**Vulnerable:**
- **#3 Michigan State** — the only live team projected below their seed expectation despite program pedigree
- **#9 Iowa** — projected for an early exit

---

## Key Insight

> **The tournament doesn't reward style. It rewards preparation.**
>
> Teams that scheduled hard, won the games they were supposed to win, and built a body of work against quality competition are the ones that go deep in March. In-game shooting stats — the things fans and commentators obsess over — rank near the bottom of what actually predicts tournament success.

---

## Model Limitations

Our model is our best guess based on stats we think are good at describing the game. It cannot:

- Predict a player getting hot from three in the second half
- Account for injuries, suspensions, or team chemistry
- Capture the crowd, the pressure, or the emotional momentum of a single-elimination game
- Guarantee anything — Florida was in our original Final Four and they've already been eliminated

Sports will forever be unpredictable because there are too many variables. That's what makes March Madness worth watching.

---

## Technical Details

| Component | Detail |
|-----------|--------|
| Language | Python 3.10+ |
| Core libraries | pandas, numpy, scikit-learn, matplotlib, seaborn |
| Model | `GradientBoostingRegressor(n_estimators=300, max_depth=4, learning_rate=0.1)` |
| Validation | `LeaveOneGroupOut` (groups = tournament years) |
| Features | 34 (24 statistical + 4 engineered + 7 resume − 1 SEED removed) |
| Target | WINS (0–6, derived from ROUND) |
| Training set | 1,152 team-years (2008–2025) |
| Prediction set | 16 live teams (2026 tournament) |

---

## How to Run

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/nishaanamin/march-madness-data)
2. Extract the ZIP file into a folder
3. Open `March_Madness_Datathon_v4_no_seed_live16.ipynb`
4. Update the `DATA_PATH` variable in the setup cell to point to your extracted folder
5. Run all cells top to bottom — the setup cell loads everything in one shot, so no dependency errors

---

## Team

**Cushing Street Analytics** — Brandeis International Business School, Datathon 2026
