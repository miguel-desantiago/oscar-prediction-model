# Academy Award Statistical Prediction Model

A prediction model that picks Academy Award winners for the event's five main categories, Best Picture, Best Actor, Best Actress, Best Supporting Actor, and Best Supporting Actress. The model uses historical precursor-award results, and backtests itself against 11 real ceremonies to show how accurate that approach actually is.

## Why I built this

The Academy Awards as well as the entirety of awards season has always been something my family and I follow consistently every single year. And without fail, every single year we all sit around and try to make guesses on who is going to win each award show, and slowly, build up to who we think is going to win the Academy Award. 

So, when the 2026 Awards Season came around, we once again followed this same tradition and my mind got to thinking, how can we predict this systematically?

That is how this project came to be. I wanted to combine something I actively follow and really enjoy with the field that I am pursuing. It was a way for me to collect and structure a dataset by hand and design a reasonable approach for modeling.

## What it does

Every year, there are plenty of award shows prior to the Academy Awards, that many consider to be strong predictors of who will end up winning the Oscar for a particular film. These awards often include the Producers Guild, the Directors Guild, the Screen Actors Guild, BAFTA, Golden Globes, and Critics' Choice. 

This model takes that idea and treats precursor awards as a way to support a prediction for a year's Academy Award winners. It measures how often that award's winner has gone on to win the Oscar, and uses that rate as a weight for creating a probability score. A nominee's final score is constructed from the weights of every precursor they won, along with a small fixed contribution from the total number of Oscar nominations for their film in that year.

With only 11 years of historical data as of now, there is not enough data to fit something like a linear regression model, without the risk of overfitting.

## Data

All data was researched and compiled by hand in Google Sheets, then exported to CSV — nothing here is pulled from an API or a pre-built dataset. It covers:

- **`awards_historical_data/`** — every nominee in all five categories, 2015–2025, with their precursor-award results and whether they won the Oscar.
- **`2026_oscar_nominees/`** — the current field being predicted. `won_oscar` is blank here since the outcome isn't known yet.

## Methodology

For Best Picture, the precursor signals ranked (highest to lowest reliability): **PGA (~72%)**, then a near-tie between **DGA and Critics' Choice (~54%)**, with **BAFTA (~18%)** the weakest. PGA members and Academy members tend to evaluate films on similar criteria, while BAFTA is a separate national body with its own taste profile. Interestingly, that ordering does *not* hold for the acting categories. BAFTA is actually one of the *strongest* signals for the lead acting categories, which is a good reminder that "this award predicts well" doesn't generalize across categories.

Total nomination count is included as a very light signal (weight: `0.04` per nomination) rather than a measured rate. It was seen that the gap between winner and non-winner average nomination counts is small and inconsistent year to year, so it's treated as a minor tiebreaker rather than real evidence.

## Results

The model is backtested using **leave-one-year-out (LOYO) cross-validation**: for every ceremony, its precursor rates are computed using only the *other* years, so a year's own outcome never leaks into the numbers used to predict it.

| Category | Accuracy |
|---|---|
| **Overall** | **78.2%** (43 / 55) |
| Best Supporting Actor | 90.9% |
| Best Supporting Actress | 90.9% |
| Best Actor | 81.8% |
| Best Actress | 72.7% |
| Best Picture | 54.5% |

Best Picture is consistently the hardest category to call. It has historically had more genuine precursor disagreement (different guilds pick different winners more often), while the supporting-acting categories tend to have a clearer front-runner earlier in the season.

**Honest limitation:** every rate above comes from only 11 ceremonies, so treat these numbers as a reasonable estimate of the model's performance, not a guarantee. 

## Project structure

```
oscars_prediction_model/
├── awards_historical_data/        # 2015-2025 results per category
├── 2026_oscar_nominees/           # current field being predicted
├── oscars_prediction_system.ipynb # the model + 2026 predictions
├── testing/
│   └── oscar_model_backtest.ipynb # self-contained LOYO backtest
└── requirements.txt
```

The backtest notebook duplicates the statistics/scoring logic rather than importing it, so either one can be read and run entirely on its own.

## Running it locally

```
python3 -m venv .venv
source .venv/bin/activate      # .venv\Scripts\activate on Windows
pip install -r requirements.txt
jupyter notebook
```

Then open `oscars_prediction_system.ipynb` and run all cells for the 2026 predictions, or `testing/oscar_model_backtest.ipynb` to see the backtest.

## Future work

- A second, fitted model (logistic regression) evaluated with the same LOYO backtest, compared directly against this heuristic baseline
- More features as more data becomes available (prior nomination counts, release timing, critics' aggregator scores)
