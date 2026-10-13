# Module 1: Introduction and Data Sources — What Was Done and What Came Out

This document covers the notebook `Module_01_Colab_Introduction_and_Data_Sources.ipynb`. It describes the path the analysis took and the results it reached. It doesn't explain the code.

---

## 1. The Goal of This Module

Before any prediction can be made, we need data. This module is a **tour of free data sources** for stock-market analysis. It answers two questions:

1. **Where can we get the data?** Macro-economics, stock indexes, ETFs, individual stocks, commodities, company financials and company lists.
2. **What does the market look like right now?** Each source is pulled live, so every output is a snapshot as of **8 September 2026**.

Nothing is modelled or predicted here. The result is a set of working data pipelines and a picture of the current economic backdrop.

---

## 2. The Path Followed

| Step | What was pulled | Source |
|---|---|---|
| 1 | US economy: potential GDP, core inflation, Fed Funds rate, Treasury yields | FRED (St. Louis Fed) |
| 2 | S&P 500 history and its year-over-year growth | Yahoo Finance |
| 3 | Stock indexes: DAX (Germany), S&P 500, Dow Jones | Yahoo Finance |
| 4 | ETFs: VOO (S&P 500 fund), EPI (India earnings fund), plus their dividends | Yahoo Finance |
| 5 | A single Indian stock (Eicher Motors) | Yahoo Finance |
| 6 | Paid sources noted for later: Polygon.io (news), Alpha Vantage (fundamentals) | Links only |
| 7 | Other macro: China's foreign reserves, gold volatility, WTI and Brent oil | FRED |
| 8 | A table of ~20 current US economic indicators | Web scraping (Trading Economics) |
| 9 | NVIDIA's income statement, balance sheet and market cap | Yahoo Finance |
| 10 | A list of every listed company in the world ranked by market cap | Web scraping (companiesmarketcap.com) |

Every pull started 70 years back (from September 1956) where the source had that much history.

---

## 3. What the Data Showed

### 3.1 The US economy

| Indicator | Latest value | What it shows |
|---|---|---|
| **Potential GDP growth** (year over year) | **2.2%** (Q3 2026) | A slow, steady decline from ~2.7% in 2024. The economy's long-run growth capacity is cooling. |
| **Core inflation** (excluding food & energy, year over year) | **2.5%** (July 2026) | Down from ~3.1% in mid-2025. It is close to, but still above, the Fed's 2% target. |
| **Fed Funds rate** | **3.63%** (Aug 2026) | Cut from 3.88% in late 2025, then held flat for most of 2026. |
| **1-year Treasury yield** | **4.13%** (4 Sep 2026) | Higher than the Fed Funds rate. |
| **5-year Treasury yield** | **4.54%** (4 Sep 2026) | Higher still. |

**Takeaway:** market yields are **above** the Fed's policy rate, and longer yields are higher than shorter ones. The bond market is pricing in higher rates or more inflation ahead, even though the Fed has been easing.

### 3.2 Stock indexes

| Index | Latest level | Notes |
|---|---|---|
| **S&P 500** | **~7,700** | Charted from 1990 alongside its yearly growth, which shows the big booms and crashes (2000, 2008, 2020, 2022). |
| **DAX (Germany)** | **~26,000** | History back to 1987. Up about **9–10%** over the last year. |
| **Dow Jones** | — | Pulled for later use. |

Two S&P 500 tickers were compared. One is delayed by 15 minutes and the other is real-time. Their daily prices were **identical**, so either works for daily analysis.

### 3.3 ETFs and dividends

- **VOO** (an S&P 500 fund) was trading at about **$708**.
- **EPI** (an India fund) has history back to 2008, about **4,660 trading days**.
- EPI pays a dividend a few times a year, usually a few cents. The exception was **June 2022, when it paid $1.85**, a large one-off payout.
- Two download methods were compared. One returns dividend and split events and the other doesn't. **The prices themselves were identical**: both return prices already adjusted for dividends.

### 3.4 An Indian stock

**Eicher Motors** (EICHERMOT.NS) has data back to **1996**, about **7,700 trading days**. It grew from about ₹1 (split-adjusted) to about **₹7,600**.

### 3.5 Commodities and other macro data

- **China's foreign-currency reserves** and **gold-price volatility** were charted as examples of less common macro data.
- **WTI oil** shows a **negative price in April 2020**. The notebook flags this as a possible data error. It is real: during the COVID crash, oil futures briefly traded below zero because storage ran out.
- **Brent oil** (the European benchmark) was pulled for comparison.

### 3.6 Snapshot of the US economy (Trading Economics)

Scraping one web page returned about 20 headline numbers in a single table:

| Indicator | Latest |
|---|---|
| GDP growth rate (quarterly, annualised) | 1.5% |
| GDP annual growth rate | 2.1% |
| Unemployment rate | 4.1% |
| Inflation rate (headline) | 3.4% |
| Non-farm payrolls (monthly job gains) | 162k |
| US dollar index | 98.7 |

This shows that scraping a single page can be a quick way to get many indicators at once, without separate API calls.

### 3.7 Company fundamentals: NVIDIA

- Annual income statements and balance sheets for the last 4–5 years came back from Yahoo Finance.
- Net income for the year to January 2026 was about **$120 billion**.
- Market cap was about **$5.45 trillion**.

### 3.8 The world's largest companies

Scraping companiesmarketcap.com gave a list of **11,299 listed companies** with their market cap, price and country. The top of the list:

| Rank | Company | Market cap |
|---|---|---|
| 1 | NVIDIA | $5.45T |
| 2 | Apple | $4.61T |
| 3 | Alphabet (Google) | $4.10T |
| 4 | Microsoft | $3.67T |
| 5 | Amazon | $2.77T |
| 6 | TSMC (Taiwan) | $2.28T |
| 7 | SpaceX | $2.02T |
| 9 | Saudi Aramco | $1.68T |
| 10 | Meta | $1.56T |

This list is the starting point for choosing which stocks to study. Module 2 uses it to pick the top US, European and Indian companies.

---

## 4. Overall Takeaways

- **Plenty of good data is free.** FRED covers macro-economics and Yahoo Finance covers prices, dividends and basic financials. Simple scraping fills most of the remaining gaps.
- **Each source has a different rhythm.** GDP is quarterly, inflation and the Fed rate are monthly, and yields and prices are daily. Combining them later means lining up these different frequencies.
- **The current backdrop:** inflation is easing, the Fed is on hold after cutting, bond yields are rising, and stock indexes are near record highs. A handful of US tech giants dominate global market value.

---

## 5. Caveats and Things to Be Aware Of

- **All numbers are a snapshot.** Re-running the notebook on another day will give different values.
- **Some series have gaps.** Core inflation has **no value for October 2025** in the data that was pulled, so the growth figures around it are blank.
- **The last row of some daily data is incomplete.** For example, VOO and Eicher Motors show an empty price for 8 Sep 2026, because the notebook ran while that trading day was still in progress.
- **Scraping is fragile.** The Trading Economics and companiesmarketcap.com scrapes depend on those sites' page layout and on a browser-like request header. Either can break without warning.
- **Market cap from Yahoo Finance has no clear timestamp,** so it's unclear exactly when the value was last updated.
