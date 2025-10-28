# NYC Taxi Surge Pricing Model

This project develops a dynamic surge pricing model for taxi services using the NYC Yellow Taxi trip dataset. The system is built on a two-part machine learning approach:

1.  **Base Fare Prediction:** An XGBoost model predicts the standard fare for a trip based on its characteristics (distance, duration, location, etc.).
2.  **Demand Forecasting:** A second XGBoost model forecasts the ride demand (number of rides) for a specific zone and time.

These two predictions are then combined with a supply heuristic to calculate a dynamic surge multiplier and determine the final, surge-adjusted price for a ride.

## 🗃️ Dataset

* **Source:** NYC Taxi & Limousine Commission (TLC) Trip Data
* **File Used:** `yellow_tripdata_2015-01.csv`
* **Initial Size:** 12,748,986 records

## ⚙️ Project Workflow

The project follows a comprehensive data science pipeline:

### 1. Data Cleaning
* Loaded the dataset (12.7M rows).
* Removed rows with null values.
* Filtered out invalid records where key metrics (e.g., `trip_distance`, `passenger_count`, `total_amount`) were zero.
* Removed trips with latitude or longitude coordinates set to 0.

### 2. Exploratory Data Analysis (EDA) & Outlier Removal
* Used box plots to visualize the distribution of `trip_distance` and `total_amount` and identify extreme outliers.
* Applied the **Interquartile Range (IQR)** method to filter out outliers from `trip_distance` and `fare_amount`.
* Filtered coordinates to ensure all pickup and dropoff locations are within the geographical bounds of New York City (latitudes 40-42, longitudes -75 to -72).

### 3. Feature Engineering
Several new features were created to improve model performance:

* **Trip Duration:** Calculated `trip_duration_min` from the pickup and dropoff timestamps.
* **Categorical Duration:** Binned `trip_duration_min` into `short`, `medium`, and `long` categories.
* **Time-Based Features:** Extracted `pickup_hour`, `pickup_day_of_week`, `pickup_month`, and a boolean `is_weekend` from the pickup timestamp.
* **Geospatial Clustering (Zone Creation):**
    * Used `sklearn.cluster.KMeans` with `n_clusters=50` on the pickup and dropoff coordinates.
    * Created `pickup_zone_d` and `dropoff_zone_d` to represent geographical zones instead of raw coordinates.
* **Demand Feature:** Created the target variable `demand_1` by grouping data by `pickup_zone_d`, `pickup_date`, and `pickup_hour` and counting the number of rides.

### 4. Modeling

Two separate models were trained using `XGBRegressor`.

#### Model 1: Base Fare Prediction
* **Objective:** Predict the `fare_amount` for a trip.
* **Features:** `passenger_count`, `trip_distance`, `trip_duration_min`, time-based features, and zone IDs.
* **Results:**
    * **R² Score:** 0.986
    * **Mean Absolute Error (MAE):** 0.24

#### Model 2: Demand Forecasting
* **Objective:** Predict the `demand_1` (number of rides) for a given zone at a specific hour.
* **Features:** `pickup_zone_d`, `pickup_hour`, `pickup_day_of_week`, `pickup_month`, `is_weekend`.
* **Results:**
    * **R² Score:** 0.895
    * **Mean Absolute Error (MAE):** 45.01

### 5. Surge Price Calculation

The final surge price is calculated using the outputs of both models:

1.  **Get Base Fare:** `predicted_fare = model.predict(...)`
2.  **Get Demand:** `predicted_demand = xgb_demand.predict(...)`
3.  **Estimate Supply:** A heuristic is used to estimate supply based on demand and time of day (e.g., during peak hours, supply is 70% of predicted demand).
    ```python
    if pickup_hour in [8,9,18,19,20]: # Peak hours
        supply = predicted_demand * 0.7
    else: # Off-peak
        supply = predicted_demand * 1.1
    ```
4.  **Calculate Multiplier:** The surge multiplier is calculated based on the demand-supply gap.
    ```python
    surge_multiplier = 1 + (predicted_demand - supply) / supply
    surge_multiplier = max(1, surge_multiplier) # Multiplier cannot be less than 1
    ```
5.  **Calculate Final Price:**
    ```python
    surge_price = predicted_fare * surge_multiplier
    ```

## 🛠️ Installation & Usage

To run this project, you'll need the following libraries.

```bash
pip install pandas matplotlib scikit-learn xgboost

# ===== USER INPUT SAMPLE =====
user_input = {
    'passenger_count': 2,
    'trip_distance': 5.3,
    'pickup_longitude': -73.985,
    'pickup_latitude': 40.748,
    'dropoff_longitude': -73.975,
    'dropoff_latitude': 40.758,
    'trip_duration_min': 12,
    'trip_duration_category': 1,  # 1=medium
    'pickup_hour': 18,
    'pickup_day_of_week': 3,  # 3=Thursday
    'pickup_month': 10,
    'is_weekend': 0,
    'pickup_zone_d': 12,
    'dropoff_zone_d': 45
}

# ... (prediction logic) ...

# ===== PREDICTION RESULT =====
# Base Fare Price      : $17.11
# Predicted Demand     : 4.42
# Estimated Supply     : 3.09
# Surge Multiplier     : 1.43
# Final Surge Price    : $24.44