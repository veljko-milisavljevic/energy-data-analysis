# Energy Data Analysis

Two university projects on electricity data, written in Python in Google Colab: a statistical study of electricity use in European countries and an hourly load forecast with SARIMA. Notebook text and comments are in Serbian.

| Notebook | What it does |
|---|---|
| [`01_data_preparation.ipynb`](01_data_preparation.ipynb) | Keeps the country rows of Ember's monthly electricity data (drops aggregates without an ISO code), joins them with Eurostat GDP by country and year (up to 2024), drops rows without GDP and saves `ember_with_demo.csv` |
| [`02_eu_electricity_statistics.ipynb`](02_eu_electricity_statistics.ipynb) | Statistical analysis of demand, generation, clean and renewable share and CO₂ intensity |
| [`03_sarima_load_forecast.ipynb`](03_sarima_load_forecast.ipynb) | Forecasts six months of hourly electricity load for the AEP region (USA) with SARIMA |

## European electricity statistics

Final project for a probability and statistics course. It covers:

- **Descriptive statistics** over 3,802 country-months of demand (TWh), generation (TWh), clean and renewable share (%) and CO₂ intensity (gCO₂e/kWh).
- **Covariance and correlation.** Demand is strongly correlated with GDP (r ≈ 0.95). The clean and renewable shares barely depend on GDP. Over time the renewable share rises (r ≈ 0.23) and CO₂ intensity falls (r ≈ −0.18).
- **Probabilities.** The distribution of CO₂ intensity and the conditional probability of low emissions given a high renewable share.
- **Confidence intervals** for the variance of the clean-energy share, from 50 random samples of 1,000 records.
- **Regression.** Demand ~ GDP is linear with R² = 0.903, and a quadratic model fits slightly better (R² = 0.907), so each extra unit of GDP adds less demand. A multiple regression of GDP on demand, renewable share and CO₂ intensity reaches R² = 0.904; CO₂ intensity is not significant in it (p = 0.80).

## SARIMA load forecast

- Model identification uses the last year of the series: an ADF test shows it is stationary (p < 0.001), and ACF and PACF suggest p = 2 and q = 0.
- Data for the model: the last two years of hourly load, resampled to a regular hourly index with gaps filled forward. The last 180 days are the test set (4,320 hours), the rest is the training set (13,201 hours, about 18 months).
- Model: SARIMA(2, 0, 0)(1, 1, 0, 24), with a daily season.
- Result on the test set: **MAE 2,119 MW, RMSE 2,531 MW, MAPE 15.3 %.**

## Data

The data is not in this repository. Put the files next to the notebooks before running them.

| File | Source |
|---|---|
| `ember.csv` | [Ember – Monthly Electricity Data](https://ember-energy.org/data/monthly-electricity-data/), Europe (CC BY 4.0) |
| `EU GDP.csv` | [Eurostat `nama_10_gdp`](https://ec.europa.eu/eurostat/databrowser/view/nama_10_gdp/default/table), GDP at market prices (`B1GQ`) in chain-linked volumes (`CLV10_MEUR`), reshaped by hand to the columns `Area`, `year`, `unit`, `Unit of measure`, `gdp` |
| `ember_with_demo.csv` | Output of notebook 01, input of notebook 02 |
| `AEP_hourly.csv` | [Hourly Energy Consumption](https://www.kaggle.com/datasets/robikscube/hourly-energy-consumption) by Rob Mulla, PJM data, Kaggle (CC0) |

## Running

Open a notebook in Colab with `https://colab.research.google.com/github/veljko-milisavljevic/energy-data-analysis/blob/main/<notebook>.ipynb` and upload the CSV files, or run it locally:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

## Known limitations

- The SARIMA model has only a daily season. Weekly and yearly patterns are not modelled, which is the main source of the error over a 180-day horizon.
- In notebook 03 the forecast band is labelled "80 %", but `summary_frame()` returns a 95 % interval by default.
- In notebook 02 the plot of renewable share over time reuses the fitted values of the demand model, so its line is wrong. The regression table above it is correct. The demand ~ GDP plot also draws one stray line (`plt.plot(demand.Value, ...)`) next to the correct orange fit.
- Notebook 01, as saved, writes the GDP column as `gdp`, while notebook 02 reads `GDP`. Rename the column before running notebook 02.
- The confidence-interval samples in notebook 02 have no random seed, so that plot changes on every run, and its y-axis is labelled "demand variance" although it shows the variance of the clean-energy share.
- A code comment in notebook 03 says "2 years train", but the training set is about 18 months (two years minus the 180-day test set).

## License

Code is under the [MIT License](LICENSE). The datasets belong to their publishers: Ember data is under [CC BY 4.0](https://ember-energy.org/creative-commons/), Eurostat data is reused under the [Eurostat copyright notice](https://ec.europa.eu/eurostat/help/copyright-notice) (source: Eurostat, `nama_10_gdp`), and the Kaggle dataset is CC0.
