# Retail-Demand-Inventory-Optimizer
A two-stage decision analytics project: forecast product demand, then use that forecast to make an inventory decision — how much safety stock to hold and when to reorder, at the lowest cost that still meets a target service level.

# Business Problem

Retailers need to decide how much stock to keep on hand for each item at each store. Too little, and they stock out and lose sales. Too much, and they tie up cash in inventory that costs money to hold. This project answers that question using real historical sales data rather than guesswork.

# Data

Store Item Demand Forecasting Challenge (Kaggle) — 5 years (2013–2017) of daily sales across 10 stores and 50 items, 913,000 rows total. Raw data is not included in this repo per the competition's data-sharing terms; see Reproducing below.

# Approach
# 1. Demand Forecasting (01_ForestDemand.ipynb)
Engineered time-based features (day of week, week of year, month) and lag/rolling-window features (previous day's sales, previous week's sales, 7-day rolling average) to capture trend and seasonality
Trained a Random Forest Regressor on an 80/20 time-based split (not random — this is a forecasting problem, so the model is only evaluated on data that comes after everything it was trained on)
Result: Test MAE of 6.53, RMSE of 8.52, against an average daily demand of ~52 units per store-item — roughly 12-13% relative error, a reasonable result for a first-pass model with no external features (no pricing, promotions, or holiday data were available in this dataset)
Feature importance showed recent lagged sales (lag_1, rolling_mean_7) as the strongest predictors, ahead of calendar features — consistent with demand being driven more by short-term momentum than by pure seasonality in this dataset

<img width="867" height="575" alt="image" src="https://github.com/user-attachments/assets/018f0431-273e-481c-8eab-6f1b60f32d72" />


# 2. Inventory Optimization (02_optimize_inventory.ipynb)

Takes the forecast's mean and standard deviation of demand per store-item and applies the standard reorder-point / safety-stock model used in supply chain planning:

Safety Stock   = z × σ(daily demand) × √(lead time)
Reorder Point  = (avg daily demand × lead time) + Safety Stock

Business assumptions (stated explicitly, not hidden in code): 3-day lead time, 95% target service level, ₹5 unit cost, 20% annual holding cost rate.

Result: ₹18,668 total annual holding cost across all 500 store-item combinations to maintain a 95% service level
Plotted cost against service level from 80% to 99% — cost rises sharply as the target approaches 99%, a clear diminishing-returns pattern. Recommendation: pushing every item to 99% service level is not cost-efficient; a 95% target is a more defensible default, with 99% reserved selectively for the highest-volume or highest-margin items.

<img width="1166" height="760" alt="image" src="https://github.com/user-attachments/assets/9889a159-ac1a-4e48-96ce-60e9816ddb60" />

# Files
| File | Description |
|---|---|
| 01_ForecastDemand.ipynb | Loads data, engineers features, trains and evaluates the forecasting model |
| 02_Optimize_Inventory.ipynb | Loads the forecast, computes reorder points and safety stock, plots the cost tradeoff |
| demand_forecast_summary.csv | Model output: predicted mean/std daily demand per store-item (feeds notebook 2) |
| inventory_policy.csv | Final output: safety stock, reorder point, and holding cost per store-item |
| feature_importance.png | Which features drove the forecasting model's predictions |
| cost_vs_service_level.png | The cost/service-level tradeoff chart |

# Tools

Python, Pandas, NumPy, Scikit-learn (Random Forest), SciPy (statistical functions for the inventory model), Matplotlib.
