# Module 4: Trading Simulations — What Was Done and What Came Out

This document covers the notebook `[2026]_Module_04_Colab_Trading_Simulations.ipynb`. It describes the path the analysis took and the results it reached. It doesn't explain the code.

---

## 1. The Question Being Asked

> **For a given stock on a given day, will its price be higher 30 trading days from now?**

That is a yes/no (binary) question. The notebook tries to answer it in several ways, from simple hand-written rules to machine-learning models. Each answer is then turned into a trading strategy: **invest $100 every time the prediction says "yes"**. The last step measures how much money each strategy would have made.

---

## 2. The Data

| Item | Value |
|---|---|
| Universe | **33 large-cap stocks** across the **US, Europe and India** (e.g. AAPL, MSFT, NVDA, LLY, MC.PA, SIE.DE, RELIANCE.NS, TCS.NS) |
| Period used | **Jan 2000 → 18 Sep 2026** (about 26 years) |
| Rows | **~202,000** stock-day observations |
| Features available | **301** per row |

The features came from earlier modules and fall into these groups:

- **Price momentum**: the stock's own growth over 1, 3, 7, 30, 90 and 365 trading days (365 trading days is about 17 months).
- **Market context**: the same trading-day growth windows for the S&P 500, Dow Jones, DAX, an India ETF, gold, oil (WTI & Brent) and Bitcoin.
- **Technical indicators**: about 55 classic ones (RSI, MACD, CCI, ADX, ATR, OBV, …).
- **Candlestick patterns**: 61 of them.
- **Macro-economics**: inflation (core CPI), Fed Funds rate, 1/5/10-year Treasury yields, potential GDP growth.
- **Calendar and identity flags**: month, weekday, ticker, region, and a new **"month + week-of-month"** flag (e.g. *October, week 4*).

### Calendar effects checked

The notebook tested whether a particular week of a particular month is linked to positive 30-day returns. The strongest link was **October week 4** (correlation ≈ **0.025**). That is very weak, so seasonality on its own carries almost no signal.

---

## 3. How the Data Was Split (by Time)

The data was split **in time order** so that models are always tested on a later period than they learned from:

| Split | Period | Share of rows |
|---|---|---|
| **Train** | Jan 2000 → Sep 2018 | ~68% |
| **Validation** | Sep 2018 → Sep 2022 | ~16% |
| **Test** | **Sep 2022 → Sep 2026** (~4 years, 1,040 trading days) | ~16% |

In the test period, **57%** of all stock-days were followed by a positive 30-day return. This is the **baseline**: a "always say yes" strategy would be right 57% of the time. A useful predictor has to beat it.

The key metric throughout is **precision**: *when the model says "buy", how often is it right?* Precision matters most because money is only put at risk on "buy" signals.

---

## 4. Step 1: Hand-Written Rules (Baseline)

Five simple rules were tried first:

| # | Rule (in plain words) | Precision on Test | Signals on Test |
|---|---|---|---|
| pred0 | CCI indicator is very high (> 200), i.e. strong short-term momentum | 56.5% | 921 |
| pred1 | Stock went up over the last 30 days | 57.9% | 19,158 |
| pred2 | Stock **and** S&P 500 both went up over the last 30 days | 57.3% | 15,244 |
| pred3 | 10-year yield ≤ 4.5% **and** 5-year yield ≤ 4% (low-rate environment) | **58.8%** | 15,910 |
| pred4 | 10-year yield > 4% **and** Fed Funds ≤ 4.795% | 50.3% | 15,921 |

**Result:** none of the manual rules beats the 57% baseline by much. The low-rate rule (pred3) does best at about 59%. The pred4 rule is no better than a coin flip. It also never fired during the validation period, because rates were low then.

---

## 5. Step 2: Decision Trees

A decision tree learns a sequence of yes/no questions about the features. The **depth** controls how many questions it can ask in a row. Every tree was trained on Train + Validation combined (2000–2022) and scored on Test.

### Depth tuning

Depths 1 to 20 were compared:

| Depth | 1 | 2 | 3 | **4** | 5 | 6 | **7** | 8 | 10 | 15 | 20 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Test precision | 57.0% | 62.3% | 57.2% | **62.9%** | 61.3% | 60.9% | **62.6%** | 61.3% | 59.6% | 58.1% | 59.7% |

**Pattern:** shallow-to-medium trees (depth 4–7) work best. Deeper trees **overfit**: they memorise the past and do worse on new data.

### Trees kept

| # | Model | Precision on Test | Buy signals on Test |
|---|---|---|---|
| pred5 | Tree, depth 10 | 59.6% | 9,861 |
| pred6 | Tree, depth 4 (best score) | **62.9%** | 6,677 |
| pred7 | Tree, depth 7 (chosen as "best": more complex but almost as precise) | 62.6% | 4,196 (243 nodes / 122 leaves) |

The depth-10 tree caught **1,344** winning trades on Test that **none** of the five manual rules caught. The ML model finds opportunities the simple rules miss.

### What the tree learned first

A depth-2 tree was drawn to show the top-level logic. **Its first splits use macro-economic variables, not price patterns:**

- If **core inflation ≤ 2%**, it predicts **"up"**.
- If **core inflation > 2%**, it predicts:
  - **"down"** when the Fed Funds rate ≤ 4.96%
  - **"up"** when the Fed Funds rate is higher

So, at the top level, the model mainly reads the **interest-rate / inflation regime**.

---

## 6. Step 3: Random Forest

A Random Forest averages many decision trees. A grid of depths (4–15) and tree counts (10–100) was compared.

- Precision ranged from **~55% to ~60%**.
- The best region was **deep trees (13–15) with many trees (100)**, at about **60%**.
- **Selected:** 100 trees, depth 13 (≈ 60.3% precision at the default 50% cut-off).

---

## 7. Step 4: Logistic Regression

A linear model was tried after the features were standardised.

- Precision stayed between **54.4% and 55.6%** for every setting tried.
- **Result:** it is **below the 57% baseline**. A linear model can't capture this problem well.

---

## 8. Step 5: Neural Network

A two-layer neural network (256 + 256 neurons, ~143k parameters) was set up, and a search over 1,024 configurations was defined. The training history of the best configuration showed:

- **Training** precision rose steadily from 63% → **73%**.
- **Validation** precision **fell** from 56% → **54%**, and validation loss more than doubled.

**Result:** classic **overfitting**. The network memorised the training years and got worse on unseen years. It was not used further.

---

## 9. Step 6: Tuning the "Confidence Threshold"

Instead of "buy if the model is more than 50% sure", the notebook used the model's **probability** and only bought above a higher cut-off. The goal is fewer but more reliable trades.

Four new strategies were created:

| # | Model | Buy only if probability ≥ | Precision on Test | Buy signals on Test |
|---|---|---|---|---|
| pred8 | Tree depth 7 | 0.56 | 62.7% | 4,183 |
| pred9 | Tree depth 7 | 0.66 | 59.3% | 2,185 |
| pred10 | Random Forest | 0.55 | **63.5%** | 3,111 |
| pred11 | Random Forest | 0.65 | **74.4%** | **39** |

**Results:**

- Raising the bar on the **Random Forest** clearly raised precision, up to **74%**. The cost is that it fired only **39 times in 4 years**, clustered in late 2022, early 2025 and March 2026.
- For the decision tree, raising the bar **did not help**. Its probabilities come in a few fixed steps (e.g. 0.57, 0.68, 0.74), so thresholds jump in big blocks.
- The models are **far more cautious on Test than on Train**. The depth-7 tree said "buy" on ~80% of training days but only **~13%** of test days. The Random Forest at 0.65 said "buy" on 43% of training days but **0.1%** of test days. The market regime of 2022–2026 (higher rates and inflation) looks unfamiliar to models trained mostly on 2000–2022.

---

## 10. Step 7: Trading Simulation

### The rules of the game

- Every time a strategy says "buy" for a stock on a day, invest **$100**.
- Hold for **30 trading days**, then sell.
- Pay **0.1% fees on the buy and 0.1% on the sell** (0.2% per round trip).
- **Capital needed** = $100 × 30 days × (number of buys on a busy day, 75th percentile), because positions overlap for 30 days.
- **CAGR** (compound annual growth rate) = the annualised return on that capital over the ~4-year test period.

### Results (Test period, Sep 2022 → Sep 2026)

| Strategy | Trades | Net profit | Capital needed | 4-yr growth | **CAGR** | Avg profit / trade |
|---|---|---|---|---|---|---|
| pred0 – CCI > 200 | 921 | $1,656 | $6,000 | ×1.28 | 6.3% | $1.80 |
| pred1 – stock up last 30d | 19,158 | **$40,918** | $72,000 | ×1.57 | **11.9%** | $2.14 |
| pred2 – stock & S&P up | 15,244 | $30,854 | $75,000 | ×1.41 | 9.0% | $2.02 |
| pred3 – low rates | 15,910 | $37,542 | $99,000 | ×1.38 | 8.4% | $2.36 |
| pred4 – high 10y, moderate Fed | 15,921 | $15,547 | $99,000 | ×1.16 | 3.7% | $0.98 |
| pred5 – Tree depth 10 | 9,861 | $27,419 | $48,000 | **×1.57** | **12.0%** | $2.78 |
| pred6 – Tree depth 4 | 6,677 | $21,051 | $99,000 | ×1.21 | 4.9% | $3.15 |
| pred7 – Tree depth 7 | 4,196 | $15,064 | $51,000 | ×1.30 | 6.7% | $3.59 |
| pred8 – Tree d7, p ≥ 0.56 | 4,183 | $15,047 | $51,000 | ×1.30 | 6.7% | $3.60 |
| pred9 – Tree d7, p ≥ 0.66 | 2,185 | $6,397 | $24,000 | ×1.27 | 6.1% | $2.93 |
| pred10 – Random Forest, p ≥ 0.55 | 3,111 | $11,622 | $27,000 | ×1.43 | **9.4%** | $3.74 |
| pred11 – Random Forest, p ≥ 0.65 | 39 | $263 | $6,000 | ×1.04 | 1.1% | **$6.73** |

### What the simulation shows

1. **Every strategy made money** in this test window. Part of that is because 2022–2026 was mostly a rising market: 57% of stock-days went up.
2. **Best annual return:** the **depth-10 decision tree (pred5) at ~12% CAGR**, needing about $48k of capital. The very simple **"stock rose last month"** rule (pred1) was essentially tied at ~11.9% but needed 50% more capital. A plain momentum rule turned out to be a tough benchmark.
3. **Best efficiency:** the **Random Forest at 0.55 (pred10)**. It earned 9.4% CAGR with only $27k of capital and about 7 trades a day, with one of the highest profits per trade ($3.74).
4. **Higher precision doesn't mean higher total return.** The 74%-precise strategy (pred11) earned the most per trade ($6.73) but traded so rarely that it made only $263 in 4 years. Its capital sat idle almost all the time.
5. **Higher precision doesn't guarantee higher CAGR either.** The depth-4 tree had the best precision among the trees (62.9%), but it put all its trades into a few busy days with up to all 33 stocks at once. That pushed its capital requirement to $99k and pulled its CAGR down to 4.9%.
6. **Fees matter more for weak signals.** Fees ate 17% of gross profit for pred4 but only 3–5% for the more selective ML strategies.

---

## 11. Overall Takeaways

- **Predicting 30-day stock direction is hard.** The best models reach about **60–63% precision** against a 57% baseline. That's a small edge, but real, and it adds up over thousands of trades.
- **Tree-based models (Decision Tree, Random Forest) beat** the linear model and the neural network here. The neural network and deep trees overfit badly.
- **The macro regime (inflation, interest rates) is the most important input.** The trees split on it first, and the best hand-written rule was also a rate rule.
- **Trading economics depend on more than accuracy:** how often a strategy trades, how bunched its trades are, and how much capital it ties up all affect the final return.
- **Simple momentum is a strong benchmark.** Any model has to clearly beat "buy what went up last month" to justify its complexity.

---

## 12. Caveats and Things to Be Aware Of

These points affect how far the results can be trusted:

- **Model choice used the test set.** Depth and other settings were picked by looking at test-period precision, so the reported test numbers are somewhat **optimistic**. A cleaner approach would pick settings on Validation and touch Test only once at the end.
- **The probability-threshold table in Step 6 was computed on standardised (rescaled) test data**, which had been prepared for the logistic regression. The decision tree was trained on unscaled data, so that table and its precision-vs-threshold curve are unreliable. The final pred8/pred9 signals were computed on the original unscaled data, so their numbers in the simulation are fine.
- **Rule pred3 doesn't match its description.** The notebook text says "10y ≤ 4 and 5y ≤ 1", but the rule applied was "10y ≤ 4.5 and 5y ≤ 4". The ~58.8% precision shown is for the applied rule. An inline note in the notebook quotes 0.580, but the actual output is 0.588.
- **The capital and CAGR figures are approximations.** They assume a fixed "busy day" level of positions held for 30 days. They don't model cash reuse, compounding, taxes, slippage or currency effects across US, EU and Indian markets.
- **No "buy and hold everything" benchmark** was run in the simulation. Without one, it's hard to say how much of each strategy's profit came from skill and how much from a generally rising market.
- The final two cells look up strategy names that no longer exist (`…rule_92`, `…rule_84`) and return empty or partial tables. They are leftovers from an earlier version of the notebook.
