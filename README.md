# Ride-Hailing Pricing & Demand Analysis

> Data analysis and predictive modeling of a real online-taxi dataset: demand patterns, unfulfilled requests, surge pricing, trip cancellations, hypothesis testing, and price/acceptance prediction.

**Course:** Transportation Planning (برنامه‌ریزی حمل و نقل) — Faculty of Industrial Engineering
**Instructor:** Dr. Sadeghi
**Author:** Kian Keivani 
---

## Table of Contents

1. [Overview](#overview)
2. [Dataset](#dataset)
3. [Project Questions](#project-questions)
4. [Repository Structure](#repository-structure)
5. [Getting Started](#getting-started)
6. [Methodology](#methodology)
7. [Results & Analysis](#results--analysis)
8. [Strategic Recommendations](#strategic-recommendations)
9. [Limitations & Future Work](#limitations--future-work)

---

## Overview

Imagine you are studying an online taxi market and want to evaluate how well its pricing system has performed. Using a real dataset from an online taxi application, containing trip price checks and requests from passengers, this project analyzes:

- **Spatial demand** — where demand originates and where it fails to convert into completed trips
- **Temporal & pricing behavior** — demand, price level, volatility and subsidies across the day
- **Cancellation behavior** — what drives passenger and driver cancellations
- **Prediction** — trip price (linear regression) and driver acceptance (logistic regression)

All code is in the Jupyter notebook; the analysis and conclusions are documented in this README.

## Dataset

The dataset (`data.csv`) holds a set of trip price checks/requests with 11 columns:

| Column | Description |
|---|---|
| `price_check_time` | Time the passenger requested a price quote |
| `passenger` | Passenger ID |
| `origin` | Origin zone ID |
| `destination` | Destination zone ID |
| `price` | Offered trip price (before discount) |
| `subsidy` | Discount amount |
| `distance` | Trip distance (meters) |
| `expected_duration` | Estimated trip duration (minutes) |
| `req_time` | Time the trip request was submitted |
| `driver` | ID of the driver who accepted the trip |
| `status` | Request outcome (see below) |

**`status` codes**

| Value | Meaning |
|---|---|
| `1` | Trip completed |
| `2` | Cancelled by passenger |
| `3` | Cancelled by driver |
| `4` | Driver did not accept the request |
| *blank* | No request registered — the passenger asked for a price but never submitted a trip request |


## Project Questions

The assignment is split into three parts. First, inspect the data generally and **preprocess** it (remove outliers and missing values, normalize columns, etc.).

### Part A — Data Analysis

**1. OD matrix analysis**
- Extract and plot the trip **demand** matrix.
- Extract and plot the **completed trips** matrix.

**2. Unfulfilled demand**
- What percentage of requests were left without a driver (`status = 4`)? How does this percentage change across hours of the day?
- Which origin–destination pairs have the highest share of demand?
- For which origin–destination pairs does **price per kilometer** fluctuate the most?
- During peak hours (06:00–08:00 and 16:00–19:00), what is the average ratio of subsidy to trip price? Compare it with non-peak hours.

**3. Temporal analysis**
- Plot the distribution of the number of requests across hours of the day. At which hours are requests highest and lowest?
- How do price level and price fluctuation vary across the day?

**4. Cancellation analysis**
- Plot the effect of **distance, price and estimated duration** on **passenger** cancellation (no 3D plots).
- Repeat the analysis from the **driver's** side.

### Part B — Passenger & Driver Behavior (Hypothesis Testing)

Using the `fitter` library and `scipy.stats`, answer the following. Use the **Fisher** and **t** tests at a **95% confidence level**.

1. Does the offered trip price have a significant effect on **passenger cancellation**?
2. Does trip distance have a significant effect on **driver cancellation**?
3. Are requests with a discount (`subsidy > 0`) more likely to be accepted than requests without one?

### Part C — Behavior Prediction

**1. Linear regression — predict trip price**
- Feed one record to the model and compare the predicted price with the real price.
- Compute the difference as an absolute value and as a percentage.
- Check whether this difference is acceptable relative to the model's average error (**MAE**).

**2. Logistic regression — predict whether the driver will accept the trip** (regardless of whether it is later cancelled)

*Suggested structure:*
- Numerical features: `hour`, `distance`, `subsidy`
- Categorical features: `origin`, `destination` → one-hot encoding

*Regression evaluation metrics:* MAE, RMSE, MAPE, R²

*Analysis required:*
- Evaluate the linear regression using the metrics above.
- Identify which features matter most for price prediction (**coefficients**).
- Check whether the model performs differently on long trips (**residual vs. distance** plot).
- Using the logistic regression coefficients, determine whether a discount increases the probability of a request. What about increasing the trip distance?
---
## Methodology

### 1. Preprocessing
- Missing-value inspection; rows without `distance` or `expected_duration` are dropped.
- `price_check_time` parsed to datetime; `hour` extracted.
- **Peak-hour flag** (`is_peak`): 06:00–08:00 and 16:00–19:00.
- Derived features: `price_per_km`, `discount_ratio = subsidy / price`.

### 2. OD Analysis
- Heatmaps of requested trips and completed trips (`status = 1`).
- Unfulfilled demand (`status = 4`), overall and per hour.
- Most-demanded OD pair and its share of all requests.
- Price-per-km volatility per OD pair via the **coefficient of variation** (pairs with ≥ 5 requests).
- Mean subsidy-to-price ratio, peak vs. off-peak.

### 3. Temporal Analysis
- Requests per hour (bar chart); mean price and price standard deviation per hour.

### 4. Cancellation Analysis
- Histograms of distance, price and duration for completed vs. passenger-cancelled trips, plus cancellation rate by distance quartile.
- Box plots comparing completed vs. driver-cancelled trips.

### 5. Hypothesis Testing
- Independent-samples **t-test** (price vs. passenger cancellation; distance vs. driver cancellation).
- **Fisher's exact test** (subsidy vs. acceptance), 95% confidence.

### 6. Predictive Modeling
- **Linear regression**: `price` ~ `hour`, `distance`, `subsidy`, one-hot `origin`/`destination`.
- **Logistic regression**: driver acceptance from the same features.
- 80/20 train–test split (`random_state=42`), `StandardScaler`.

## Results & Analysis

### 1. Data description and correlation structure

The data were cleaned and the relationships between the main variables examined. About **33%** of rows have no `status`; these are users who only requested a price quote and never entered the request cycle.

| Pair | Correlation | Interpretation |
|---|---|---|
| Price ↔ Distance | **0.81** | Very strong relationship — the base pricing algorithm tracks distance accurately |
| Price ↔ Travel time | **0.79** | Direct effect of traffic on pricing |
| Subsidy ↔ Price | **0.74** | The system grants larger discounts to more expensive trips |

### 2. Linear regression results (price prediction)

| Metric | Value | Interpretation |
|---|---|---|
| MAE | ≈ 9,079 | On average the model's absolute error is about 9 thousand units |
| RMSE | ≈ 16,935 | More sensitive to large errors; in some cases the gap between real and predicted price is considerable |
| MAPE | ≈ 14.55% | Average relative error is about 15% of the real price — acceptable for real transportation data |
| R² | ≈ 0.80 | About 80% of price variance is explained by the model's features — fairly high explanatory power |

Overall, the model performs well statistically.

**Individual record check.** To assess practical performance, several real records were compared with predictions:
- In some records (e.g., predicted prices around 60,000 or 80,000) the difference was below the MAE, so the prediction is acceptable.
- In others the absolute difference exceeded the MAE, so accuracy is not uniform. Percentage error was below 10% in some records and above 20% in others.
- This variation may come from non-linear pricing, the effect of variables not included, or different system behavior on particular trips.

### 3. OD matrices and unfulfilled demand

Geographic analysis showed that demand is concentrated in specific areas of the city.

- **Supply–demand gap:** Comparing the demand matrix (requests) with the completed-trip matrix shows that, in some zones, conversion into trips is low despite high demand. This may be caused by a shortage of drivers in those zones or by heavy traffic.
- **Status 4 (no driver found):** The "not accepted by driver" rate peaks at **06:00–08:00** and **16:00–19:00**.
- **Pricing strategy:** Fluctuation in price per kilometer during these hours indicates **peak/surge pricing** being applied to balance the market.

### 4. Statistical hypothesis tests (95% confidence)

| Test | Result | Conclusion |
|---|---|---|
| Price vs. passenger cancellation (t-test) | Mean price of cancelled trips **61,972** vs. completed **61,067**; **p = 0.15** | Price alone is not a significant factor in passenger cancellation |
| Distance vs. driver cancellation (t-test) | **p = 0.96** | Drivers show no tendency to cancel based on distance alone |
| Discount vs. acceptance (Fisher's exact) | **p = 0.83** | Offering a discount did not lead to a statistically significant increase in acceptance; passenger behavior depends more on other parameters (e.g., waiting time) |

### 5. Logistic regression (acceptance & behavior)

**Performance and data-imbalance challenge.** The logistic regression reaches high overall accuracy (**≈ 98.8%**), but the confusion matrix shows:
- The model identified all accepted trips (class 1) correctly but did not predict **any** sample of class 0 (not accepted).
- The reason is severe **class imbalance**, which biases the model toward the dominant class. Accuracy alone is therefore misleading.

**Coefficient analysis.**

| Feature | Coefficient | Interpretation |
|---|---|---|
| Hour of day (`hour`) | **−0.11** | As the day progresses toward night, the probability of a request and acceptance decreases |
| Distance (`distance`) | **+0.07** | For longer trips, passengers are more likely to request and drivers more likely to accept (probably due to higher profitability and the difficulty of finding a replacement) |
| Subsidy (`subsidy`) | **−0.06** | A weak, negative effect: increasing the discount did not necessarily raise the driver's acceptance probability, since discounts mainly aim to attract passengers rather than raise driver income |

## Strategic Recommendations

1. **Revise the discount policy.** Given the high p-value (0.83) and the negative regression coefficient, the discount allocation model should become more targeted and based on **passenger waiting time**.
2. **Manage supply during peak hours.** The sharp rise in status 4 at specific hours shows that current driver incentives during peak hours are insufficient.
3. **Improve the predictive model.** Add variables such as the driver's **estimated time of arrival (ETA)**. To resolve the class imbalance in the logistic regression, methods such as **class weighting** are necessary.
4. **Focus on long trips.** Given the positive distance coefficient, the system can prioritize and retain long trips, since these users are more inclined to finalize their request.

