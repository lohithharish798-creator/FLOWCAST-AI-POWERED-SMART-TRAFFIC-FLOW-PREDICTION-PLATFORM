# FLOWCAST-AI-POWERED-SMART-TRAFFIC-FLOW-PREDICTION-PLATFORM
FlowCast: Northline Corridor Traffic & Congestion Prediction
Meridian Mobility Systems — Data & AI Division
Dataset: Northline Corridor (NL-001 to NL-025, Jan 1 – May 31, 2025)
1. Project Overview
FlowCast is an end-to-end machine learning pipeline and inference API designed to forecast half-hourly traffic volume and classify road congestion levels across 25 segments of the Northline Corridor.
Modeling Targets
Primary Regression Target: traffic_volume (vehicle count per 30-minute window)
Primary Classification Target: congestion_level (Free-flow, Moderate, Heavy, Severe)
Secondary Risk Target: accident_count (incident occurrence indicator)
Data Pipeline & Quality Rules
The raw input tables reside in data/raw/ and are processed in the following order:
1. calendar_events.csv (Date grain): Loaded first; provides flags for public holidays, local events, and scheduled roadworks.
2. weather_observations.csv (Station × Hour grain):
Standardizes mixed date formats (DD/MM/YYYY \rightarrow YYYY-MM-DD).
Normalizes weather labels ('RAIN', 'rainy', etc.) into a controlled vocabulary (Clear, Cloudy, Rain, Fog).
3. traffic_sensor_log.csv (Segment × 30-min grain):
De-duplication: Drops duplicate detector retries on road_id + date + time.
Outlier Filtering: Removes negative traffic_volume, avg_speed > 200 km/h, and occupancy > 100%.
JSON Parsing: Extracts vehicle class shares (2W, Car, LCV, HCV) from vehicle_type_dist.
Target Imputation: Derives blank congestion_level rows (~15%) using the Volume-to-Capacity (V/C) ratio where 30-minute capacity is \text{road\_capacity} \div 2:
< 0.50: Free-flow
0.50 - 0.79: Moderate
0.80 - 0.99: Heavy
\ge 1.00: Severe
4. Temporal Alignment: Broadcasts hourly weather observations across both corresponding 30-minute traffic windows (HH:00 and HH:30).
4. Machine Learning Workflow
Feature Engineering: Generates cyclical/calendar features (hour_of_day, day_of_week, is_weekend, month) alongside historical lag features (traffic_vol_lag_30m, traffic_vol_lag_1h, and 2-hour rolling averages) grouped by road_id.
Validation Strategy: Uses a strict chronological split (Training: Jan–Apr 2025; Testing: May 2025) to prevent temporal data leakage.
Algorithms: LightGBM Regressor (LGBMRegressor) for volume forecasting and LightGBM Classifier (LGBMClassifier) for congestion banding.

Outlier Filtering: Removes negative traffic_volume, avg_speed > 200 km/h, and occupancy > 100%.
JSON Parsing: Extracts vehicle class shares (2W, Car, LCV, HCV) from vehicle_type_dist.
Target Imputation: Derives blank congestion_level rows (~15%) using the Volume-to-Capacity (V/C) ratio where 30-minute capacity is \text{road\_capacity} \div 2:
< 0.50: Free-flow
0.50 - 0.79: Moderate
0.80 - 0.99: Heavy
\ge 1.00: Severe
