# Module 3: Time Series Modeling — What Was Done and What Came Out

This document covers the notebook `[2026]_Module_03_Colab_Time_Series_Modeling.ipynb`. It describes the path the analysis took and the results it reached. It doesn't explain the code.

---

## 1. The Question Being Asked

> **For a given stock on a given day, will its price be higher 30 trading days from now?**

This module is the **first attempt at prediction**. It takes the dataset built in Module 2 and tries three kinds of approach:

1. **Hand-written rules** ("if X, then predict up").
2. A **classic statistical time-series model** (ARIMA) that looks only at past prices.
3. A **machine-learning model** (decision tree) that looks at all available features.

Module 4 builds on this with more models and a trading simulation.

---

## 2. The Data

| Item | Value |
|---|---|
| Source | The combined dataset from Module 2, refreshed on **18 Sep 2026** |
| Universe | **33 large stocks** in the US, Europe and India |
| Period used | **Jan 2000 → Sep 2026** |
| Rows | **202,292** stock-days |
| Features | **239**: 184 numeric (price growth, technical indicators, candlestick patterns, macro) and 55 yes/no flags (month, weekday, ticker, region) |

### Time-based split

The data was split in date order, so models are always tested on a later period than they learned from:

| Split | Period | Share |
|---|---|---|
| Train | Jan 2000 → Sep 2018 | 68% |
| Validation | Sep 2018 → Sep 2022 | 16% |
| **Test** | **Sep 2022 → Sep 2026** | 16% |

The three periods have very similar patterns of 30-day returns. Each has an average gain of about **+2.2% to +2.5%** per 30 days, and a similar share of rising days. The training period has a few extreme outliers (one stock-day was followed by a ×5 gain), but the typical behaviour is consistent across time.

**Baseline:** in the test period, **57%** of stock-days were followed by a rise. Any useful prediction needs to beat that.

---

## 3. Step 1: Which Features Matter? (Correlation Check)

Each feature was checked for a simple link with the target.

**Most linked with a 30-day rise:**

| Direction | Feature | Correlation |
|---|---|---|
| Negative | 10-year Treasury yield | −0.071 |
| Negative | 5-year Treasury yield | −0.064 |
| Negative | Brent oil 365-day growth | −0.059 |
| Negative | Potential GDP growth | −0.049 |
| Negative | Monthly core inflation | −0.049 |
| Positive | Bitcoin 365-day growth | +0.038 |
| Positive | Month = October | +0.036 |
| Positive | Month = November | +0.035 |

**Most linked with the size of the 30-day move:**

- The Dow Jones, DAX and S&P 500 having risen a lot over the past 365 trading days points slightly to **smaller** future gains (≈ −0.06 to −0.08). This is a hint of mean reversion: big runs tend to cool off.
- Being **NVIDIA**, being in **October**, and **high recent volatility** each point slightly to larger future gains (≈ +0.05).

**Takeaway:** every link is **very weak** (below 0.08 on a scale where 1.0 is perfect). The clearest theme is that **high interest rates and rising oil are bad for stocks**, while autumn months and risk appetite (Bitcoin) are mildly good. No single feature predicts the market alone.

---

## 4. Step 2: Hand-Written Rules

Three simple rules were tested on the 2022–2026 test period:

| Rule | Precision (how often a "buy" was right) | Buy signals |
|---|---|---|
| CCI momentum indicator is very high (> 200) | 56.5% | 921 |
| Stock rose over the last 30 days | **57.9%** | 19,158 |
| Stock **and** S&P 500 both rose over the last 30 days | 57.3% | 15,244 |

**Result:** all three land almost exactly on the 57% baseline. Simple momentum rules don't give a meaningful edge on their own.

---

## 5. Step 3: ARIMA (Statistical Forecast from Price Alone)

ARIMA forecasts a price using only its own past values. It was tried on **one stock, Amazon (AMZN)**, using a fixed configuration (ARIMA 6,2,2).

The model was re-fitted each day and asked to forecast the price 30 days ahead. If the forecast was higher than today's price, it signalled "buy". Because this is slow (about **1.5 minutes for just 10 forecasts**), only the **first 10 days of the test period** (mid-to-late September 2022) were run.

| Result | Value |
|---|---|
| Days forecast | 10 |
| "Buy" signals | 7 |
| Correct "buy" signals | **0** |
| Precision | **0%** |

On those days Amazon was around $114–$125, and the model expected small rises. In reality Amazon **fell to about $86–$111** over the next 30 days, a 10–25% drop during the late-2022 sell-off.

**Takeaway:** ARIMA follows the recent trend and **can't anticipate a market turn**. It's also too slow to run across 33 stocks and thousands of days. Ten forecasts is far too small a sample to judge it fairly, but it wasn't promising, and it isn't carried forward.

---

## 6. Step 4: Decision Trees (Machine Learning on All Features)

Two decision trees were trained on 2000–2022 data, using all 239 features, and tested on 2022–2026:

| Tree depth | Precision on Test | Overall accuracy | Buy signals (of 33,163) |
|---|---|---|---|
| 20 (very detailed) | 57.3% | 47.3% | 9,841 |
| **10 (moderate)** | **60.2%** | 48.7% | 9,298 |

**Results:**

- The **depth-10 tree** beat the baseline, with about **60% precision**. That is the first real edge in this module.
- The **depth-20 tree** did worse, landing right on the baseline. Deeper trees **memorise** the training years and generalise poorly. This is classic overfitting.
- **Overall accuracy is below 50%** for both trees. That's because they predict "down" most of the time (about 72% of days), while 57% of days actually went up. The trees are **cautious**: they rarely say "buy", but are right more often when they do. For trading, that matters more than overall accuracy.

### What the trees found important

**Depth-10 tree, top 10 features:**

1. Core inflation (year over year)
2. Gold's 365-day growth
3. Core inflation (month over month)
4. WTI oil's 365-day growth
5. Fed Funds rate
6. Accumulation/Distribution line (volume-based)
7. On-Balance Volume
8. TRIX (momentum)
9. Bitcoin's 365-day growth
10. Dow Jones 90-day growth

**Depth-20 tree, top features:** On-Balance Volume, Accumulation/Distribution, TRIX, the stock's own 365-day growth, volatility, and core inflation.

**Takeaway:** the moderate tree leans mostly on the **macro regime** (inflation, interest rates, gold, oil). The deeper tree leans more on **stock-specific volume and momentum signals**. Those signals look strong in training but don't hold up as well on new data.

The tree predictions were then joined back to the main table, ready for the trading simulations in Module 4.

---

## 7. Overall Takeaways

- **Predicting the 30-day direction is hard.** Simple rules don't beat the 57% "always up" baseline.
- **Price-only forecasting (ARIMA) failed** on the small sample tried, and it is too slow to scale.
- **A moderately deep decision tree** reached about **60% precision**, a modest but real improvement on the baseline.
- **More complexity isn't better:** the depth-20 tree overfit and lost its edge.
- **Macro-economics leads.** Inflation, interest rates and commodity trends are the strongest signals in both the correlation check and the trees.

---

## 8. Caveats and Things to Be Aware Of

- **The trees were trained without a fixed random seed,** so re-running the notebook can give slightly different precision and feature rankings. Module 4 fixes the seed.
- **The tree depths (10 and 20) were picked by hand,** not tuned. Module 4 runs a proper depth search.
- **Validation data was merged into training,** so there was no separate check for choosing settings before testing.
- **The ARIMA test covered just 10 days of one stock,** all in a falling market. The 0% precision says more about that period than about ARIMA in general.
- **Correlations only capture straight-line links.** A feature with near-zero correlation can still be useful when combined with others, which is what the decision tree exploits.
- **The precision figures are all very close to the baseline,** so small differences (e.g. 57.3% vs 57.9%) aren't meaningful.
