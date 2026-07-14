# ARIMA vs VAR Forecasting — Walmart Stock Price Analysis

**Course:** Digital Banking and Fintech  
**Authors:** Shota Xu · May Moh Moh Saesar Thaw  
**Tool:** Orange3 Data Mining (Time Series Add-on)  
**Data Source:** [Investing.com](https://www.investing.com)

---

## What This Project Is About

For our FinTech midterm, we compared two time series forecasting models — **ARIMA** and **VAR** — to see which one does a better job at predicting Walmart's stock price.

We used Walmart's daily stock data from 2019 to 2023, and paired it with the S&P 500 index as a second variable for the VAR model. The idea is simple — since Walmart is one of the biggest companies in the US, its stock price is likely influenced by how the overall market moves.

We tested both models at three forecast horizons: **5 days, 10 days, and 15 days**, then compared their accuracy using RMSE and MAE.

---

## Files in This Repository

| File | Description |
|------|-------------|
| `WMT.csv` | Walmart daily stock prices (Jan 2019 – Dec 2023) |
| `SP500.csv` | S&P 500 daily index data (same date range) |
| `WMT_S_P500_Workflow.ows` | Orange3 workflow — open this to run the full analysis |
| `README.md` | This file |

---

## Data Overview

| | WMT | S&P 500 |
|---|---|---|
| Date Range | Jan 2, 2019 – Dec 29, 2023 | Jan 2, 2019 – Dec 29, 2023 |
| Observations | 1,258 trading days | 1,258 trading days |
| Missing Values | None | None |
| Source | Investing.com | Investing.com |

---

## How to Run This

### Step 1 — Install Orange3
Download it free at 👉 https://orangedatamining.com/download/

### Step 2 — Install the Time Series Add-on
1. Open Orange3
2. Go to **Options → Add-ons**
3. Find **Time Series**, check the box, click OK
4. Restart Orange when prompted

### Step 3 — Open the Workflow
1. Download all files from this repository into the same folder
2. Open Orange3
3. Go to **File → Open** and select `WMT_S_P500_Workflow.ows`
4. The full workflow loads automatically — double-click any widget to explore the results

---

## What We Did — Step by Step

### 1. Loaded the Data
We loaded WMT.csv into Orange using the File widget and converted it to a time series using the Form Timeseries widget.

### 2. Checked for Stationarity
We plotted a Line Chart of WMT's price — it showed a clear upward trend from ~$31 in 2019 to ~$55 in 2023. This means the data is **non-stationary**, which is a problem because ARIMA and VAR need stationary data to work properly.

### 3. Differencing (d = 1)
We used the Difference widget to calculate the daily change in price instead of the actual price. For example:
- Day 1 price: $31.11
- Day 2 price: $30.95
- Difference: **−$0.16**

This removed the upward trend completely. We only needed to do this once — so **d = 1**.

### 4. Confirmed Stationarity with Correlogram
The Correlogram showed bars near zero and within the 95% confidence bands — confirming the data was now stationary and ready for modelling.

### 5. Built the ARIMA Model
We used **ARIMA(1,1,1)**:
- p = 1 → look at yesterday's price change
- d = 1 → differenced once
- q = 1 → correct for yesterday's forecast error

### 6. Built the VAR Model
We merged WMT and S&P 500 data together and ran the VAR model using both series simultaneously. VAR asks whether the S&P 500's movement today helps predict Walmart's price tomorrow.

### 7. Evaluated Both Models
We used Model Evaluation widgets set to 5, 10, and 15 forecast steps and recorded RMSE and MAE for each.

---

## Workflow Structure

<img width="955" height="903" alt="Screenshot 2026-07-14 at 2 55 33 PM" src="https://github.com/user-attachments/assets/8a833b16-7b3a-4676-b7de-d911ebe01f9b" />

### ARIMA Workflow
```
File (WMT.csv)
  → Form Timeseries
      → Line Chart              ← BEFORE differencing (non-stationary)
      → Correlogram             ← PROOF stationarity confirmed (d=1)
      → Difference
          → Line Chart (1)      ← AFTER differencing (stationary)
      → ARIMA Model (1,1,1)
          → Model Evaluation    ← 5 days
          → Model Evaluation    ← 10 days
          → Model Evaluation    ← 15 days
```

### VAR Workflow
```
File (WMT.csv)   ──┐
                    → Merge Data → Select Columns
File (SP500.csv) ──┘
  → Form Timeseries (1)
  → Difference (1)
      → Line Chart (2)          ← AFTER differencing (stationary)
      → Correlogram (1)         ← PROOF stationarity confirmed
      → VAR Model
          → Model Evaluation    ← 5 days
          → Model Evaluation    ← 10 days
          → Model Evaluation    ← 15 days
```

---

## Results

### ARIMA(1,1,1)

| Forecast Horizon | RMSE | MAE |
|-----------------|------|-----|
| 5 Days | 0.696 | 0.549 |
| 10 Days | 2.207 | 0.933 |
| 15 Days | 2.300 | 0.823 |

### VAR (WMT + S&P 500)

| Forecast Horizon | RMSE | MAE |
|-----------------|------|-----|
| 5 Days | 0.628 | 0.472 |
| 10 Days | 0.962 | 0.723 |
| 15 Days | 0.898 | 0.602 |

> **RMSE** = Root Mean Square Error — punishes big mistakes more heavily. Lower is better.  
> **MAE** = Mean Absolute Error — average error in dollars. Lower is better.  
> Both metrics closer to 0 = more accurate and reliable forecast.

---

## Verdict — VAR Wins

**VAR outperformed ARIMA at all three forecast horizons.**

The biggest difference was at 10 days — ARIMA's RMSE was 2.207 while VAR's was only 0.962. VAR was more than twice as accurate. At 15 days, VAR's RMSE of 0.898 was 61% lower than ARIMA's 2.300.

This makes sense — Walmart is genuinely influenced by the broader market. When the S&P 500 moves, Walmart tends to follow. By including this second variable, VAR had more useful information to work with, which led to better and more stable forecasts — especially at longer horizons.

---

## Limitations

- We only used one external variable (S&P 500) — factors like competitor performance, inflation, and consumer sentiment were not included
- Neither model can predict sudden real-world events like COVID-19, earnings surprises, or policy changes
- Both models assume linear relationships — real stock markets are often more complex and non-linear
- The 15-day forecast should be treated as a general direction, not an exact prediction

## What We Would Do Differently

- Add competitor stock data (Costco, Target) as additional VAR variables
- Include the Consumer Price Index (CPI) to account for inflation effects
- Test different ARIMA parameter combinations beyond (1,1,1)
- Compare results against more advanced models like LSTM neural networks

---

## References

Forbes. (n.d.). *Walmart*. https://www.forbes.com/companies/walmart/

LinkedIn. (n.d.). *What are the advantages and disadvantages of using ARIMA models for forecasting?* https://www.linkedin.com/advice/0/what-advantages-disadvantages-using-arima

LinkedIn. (n.d.). *What are the advantages and disadvantages of using vector autoregression models for forecasting?* https://www.linkedin.com/advice/0/what-advantages-disadvantages-using-vector-autoregression

Wikipedia. (2026). *Walmart*. https://en.wikipedia.org/wiki/Walmart
