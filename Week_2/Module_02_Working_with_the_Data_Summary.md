# Module 2: Working with the Data — What Was Done and What Came Out

This document covers the notebook `[2026]_Module_02_Colab_Working_with_the_data.ipynb`. It describes the path the analysis took and the results it reached. It doesn't explain the code.

---

## 1. The Goal of This Module

Module 1 showed **where** to get data. This module turns that raw data into **one large, clean table** that later modules can use to predict stock moves.

The end product is a single file. It has one row per stock per trading day, and each row holds everything known about that stock, the market and the economy on that day. It also records what happened to the stock's price **30 days later**, which is the answer a model will try to predict.

Along the way, the notebook also looks at IPOs, one stock's growth patterns, the euro-area bond market and one company's financial health.

---

## 2. The Path Followed

| Step | What was done |
|---|---|
| 1 | Scraped every **US IPO from 2023 to 2026** and studied how they performed |
| 2 | Took one stock (**Novo Nordisk, NVO**) and built growth measures, past and future |
| 3 | Pulled market and macro data: DAX, S&P 500, Dow Jones, India ETF, GDP, inflation, rates, **VIX**, **gold**, **oil**, **Bitcoin** |
| 4 | Pulled the **euro-area government bond yield curve** from Eurostat |
| 5 | Looked at **Walmart's** financial ratios over time |
| 6 | Downloaded full daily history for **33 large stocks** across the US, Europe and India |
| 7 | Added about **120 technical indicators and candlestick patterns** for every stock |
| 8 | Merged everything into one table and saved it |

---

## 3. What the Data Showed

### 3.1 US IPOs, 2023–2026

**966 IPOs** were collected:

| Year | Number of IPOs |
|---|---|
| 2023 | 154 |
| 2024 | 225 |
| 2025 | 349 |
| 2026 (to September) | 238 |

The IPO market has been **picking up** each year.

How the IPOs performed, from IPO price to today:

| Measure | Value |
|---|---|
| Average IPO price | ~$12 |
| **Median return** | **~0%** |
| Average return | +3% |
| 25% of IPOs lost at least | **50%** |
| Best performer | ~+886% |
| Worst performer | ~−100% (almost total loss) |

**Takeaway:** the **typical IPO went nowhere**, and a quarter of them lost half their value or more. A few big winners pull the average up. **Reddit (RDDT)** is one of those winners: it listed at $34 in March 2024 and was at about $160, roughly **+380%**.

About 3–4% of IPO records had missing prices or returns. Many of these were **blank-cheque companies (SPACs)** that list at a fixed $10 with no real trading history.

### 3.2 One stock in detail: Novo Nordisk (NVO)

NVO has **45 years** of daily history (from 1981). The notebook measured how its price moved over 1 day and over 30 days:

| Horizon | Share of days followed by a rise |
|---|---|
| Next 1 day | **48.7%** |
| Next 30 days | **61.6%** |

**Takeaway:** day-to-day moves are close to a coin flip, but over a month the long-term upward trend shows through. That's why later modules predict the **30-day** direction rather than the next day. Daily moves are usually within a few percent, while 30-day moves spread much wider, often ±30%.

At the time of the run, NVO was trading at about **$42.7**, down about 8% over the previous 30 trading days.

### 3.3 Market and macro snapshot (15 Sep 2026)

| Series | Latest | Change over the last 365 trading days* |
|---|---|---|
| S&P 500 | — | **+35%** |
| Dow Jones | — | +24% |
| DAX (Germany) | ~25,400 | +28% |
| India ETF (EPI) | ~$41.4 | **−5%** |
| Gold | ~$4,346 / oz | **+37%** |
| WTI oil | ~$103 / barrel | **+47%** (+32% over the last 30 trading days alone) |
| Brent oil | ~$109 / barrel | +45% |
| Bitcoin | ~$76,800 | **−33%** |
| VIX (fear index) | ~17.4 | Calm. Readings above 30 signal stress. |
| Fed Funds rate | 3.63% | |
| 1y / 5y / 10y Treasury yields | 4.35% / 4.78% / 4.96% | |
| Core inflation | 2.4% | |

*"365 trading days" is about 17 months for stocks and commodities, and exactly one year for Bitcoin, which trades every day.*

**Takeaway:** US and European stocks had a very strong run, gold and oil surged, Bitcoin fell sharply and Indian stocks lagged. Oil's sharp jump in the last month stands out.

### 3.4 The euro-area bond yield curve

Eurostat's daily yield-curve data was pulled for **1 September 2026**. It covers AAA-rated euro-area government bonds at every maturity from 3 months to 30 years.

| Maturity | Yield |
|---|---|
| 3 months | 2.42% |
| 1 year | 2.73% |
| 10 years | 3.36% |

The curve **slopes upward**: longer loans pay more, which is the normal, healthy shape. The chart matches the one published by the European Central Bank. Euro-area yields are about **1.5 percentage points lower** than US yields at the same maturities.

### 3.5 Company health: Walmart (WMT)

| Fiscal year ending | Earnings per share | Net profit margin | EBITDA margin |
|---|---|---|---|
| Jan 2023 | $1.43 | 1.9% | 4.9% |
| Jan 2024 | $1.92 | 2.4% | 5.6% |
| Jan 2025 | $2.42 | 2.9% | 6.2% |
| Jan 2026 | **$2.74** | **3.1%** | **6.5%** |

**Takeaway:** Walmart has **nearly doubled its earnings per share in three years** and widened its margins every year. Revenue for the latest year was about **$611 billion**. Its next earnings date is 19 November 2026. Earnings surprises over the past few years ranged from about −12% to +40% versus analyst estimates.

### 3.6 The 33-stock universe

The notebook downloaded full daily history for 33 of the largest companies in three regions:

| Region | Stocks | Daily rows |
|---|---|---|
| **US** | MSFT, AAPL, GOOG, NVDA, AMZN, META, BRK-B, LLY, AVGO, V, JPM | 87,218 |
| **Europe** | NVO, MC.PA (LVMH), ASML, RMS.PA (Hermès), OR.PA (L'Oréal), SAP, ACN, TTE, SIE.DE (Siemens), IDEXY, CDI.PA (Dior) | 83,615 |
| **India** | RELIANCE, TCS, HDB, BHARTIARTL, IBN, SBIN, LICI, INFY, ITC, HINDUNILVR, LT | 69,818 |

The longest history is **Eli Lilly** (from 1972). The shortest is **LIC of India**, which listed in 2022.

Average growth over the previous 365 trading days (about 17 months) for stocks in each region, by year:

| Year | US | Europe | India |
|---|---|---|---|
| 2020 | ×1.48 | ×1.25 | ×1.10 |
| 2021 | ×1.80 | ×1.58 | ×1.55 |
| 2022 | ×1.18 | ×1.19 | ×1.38 |
| 2023 | ×1.20 | ×1.25 | ×1.21 |
| 2024 | **×2.16** | ×1.41 | ×1.36 |
| 2025 | ×1.58 | ×1.06 | ×1.17 |
| 2026 | ×1.36 | ×1.05 | **×0.94** |

**Takeaway:** US stocks led most years, helped in 2024 by NVIDIA's surge. India was strongest in 2022, but its stocks are, on average, **down** over the past year in 2026.

### 3.7 Technical indicators

For every stock and day, the notebook added:

- **About 55 technical indicators** covering momentum (RSI, MACD, CCI), volume (OBV, A/D line), volatility (ATR) and market cycles.
- **61 candlestick patterns**, such as "three black crows" and "morning star", each flagged when it appears.
- A few custom measures: 10- and 20-day moving averages, daily price range and recent volatility.

### 3.8 The final dataset

Everything was joined into one table:

| Item | Value |
|---|---|
| Rows | **240,651** (one per stock per trading day) |
| Columns | **203** |
| Covers | Stock prices, the stock's own growth, technical indicators, candlestick patterns, index growth (S&P 500, Dow, DAX, India), commodity and crypto growth (gold, WTI, Brent, Bitcoin), VIX, GDP, inflation, Fed rate, Treasury yields |
| Answers to predict | The stock's growth over the **next 30 days**, and whether that growth is positive |
| Saved as | `stocks_df_combined_2026_09_15.parquet.brotli` |

This file is the input for Modules 3 and 4. They use a refreshed version dated 18 Sep 2026.

---

## 4. Overall Takeaways

- **IPOs are a lottery for most buyers:** the median IPO returned nothing, and a quarter lost half their value.
- **The 30-day horizon is more predictable than the 1-day one.** Daily direction is close to 50/50, while monthly direction leans clearly upward for good stocks.
- **The markets in September 2026:** strong stocks in the US and Europe, booming gold and oil, falling Bitcoin and lagging India.
- **The main outcome is the dataset:** about 240k rows by 203 columns, joining prices, technical signals and the macro environment for 33 global leaders.

---

## 5. Caveats and Things to Be Aware Of

- **Monthly macro data is attached to every day of its month.** For example, August's inflation figure is attached to every day in August, even though it's only published in mid-September. Models trained on this data could "see" inflation figures slightly before they were really public. This is a mild look-ahead bias.
- **The latest values are carried forward.** Where this month's macro figure isn't out yet, last month's value is used. That's reasonable, but the newest rows rely on older data.
- **The last day has incomplete prices.** The notebook ran during the trading day on 15 Sep 2026, so some stocks have blank prices for that day.
- **The VIX column ended up with an unclear name** (`Close_y`) after the merge, and is easy to confuse with the stock's own close price (`Close_x`). Module 4 drops it.
- **Regions are labelled by company, not by listing.** ACN (Accenture) and IDEXY are tagged as European, and HDB, IBN and INFY as Indian, even though they trade in the US. Their prices are in US dollars.
- **Average growth by region is a simple average across stocks,** so a single huge winner (like NVIDIA) can pull a region's figure up a lot.
- **The TA-Lib installation printed some download errors,** but the library still installed and every indicator was calculated for all 33 stocks.
