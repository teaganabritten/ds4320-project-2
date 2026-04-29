# pipeline.ipynb: This file contains the analysis and prediction for this project


```python
import numpy as np
import pandas as pd
import pymongo
import os
import logging
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor, RandomForestClassifier
from sklearn.metrics import mean_squared_error, r2_score, accuracy_score
import matplotlib.pyplot as plt
import matplotlib as mpl
import seaborn as sns
import warnings
from dotenv import load_dotenv
import plotly.express as px
```


```python
# Basic logging setup for notebook runs
logging.basicConfig(level=logging.INFO, format='%(asctime)s [%(levelname)s] %(message)s')
logger = logging.getLogger("air-quality-pipeline")

# Load from absolute path to ensure it's found
dotenv_loaded = load_dotenv('/home/teaganabritten/UVA/DataScience/ds4320/ds4320-project-2/.env')
if dotenv_loaded:
    logger.info("Loaded environment variables from .env file")
else:
    logger.warning("Could not load .env file; falling back to existing environment variables")

warnings.filterwarnings('ignore')
```

    2026-04-28 19:40:11,722 [INFO] Loaded environment variables from .env file



```python
username = os.getenv("MONGO_USERNAME")
password = os.getenv("MONGO_PASSWORD")

if not username or not password:
    raise ValueError(
        "Missing MongoDB credentials. Ensure MONGO_USERNAME and MONGO_PASSWORD are set in environment or .env."
    )

logger.info("Connecting to MongoDB Atlas cluster")

try:
    client = pymongo.MongoClient(
        f"mongodb+srv://{username}:{password}@cluster0.57c6i.mongodb.net/",
        serverSelectionTimeoutMS=10000
    )

    # Force initial server check so auth/network issues surface immediately
    client.admin.command("ping")
    logger.info("MongoDB connection established")

    db = client["DS4320"]
    collection = db["Project2"]

    logger.info("Fetching documents from DS4320.Project2")
    documents = list(collection.find({}))

    if len(documents) == 0:
        logger.warning("Query succeeded but returned 0 documents")
    else:
        logger.info(f"Loaded {len(documents)} documents")

except pymongo.errors.OperationFailure as e:
    logger.error(f"Authentication/authorization failed: {e}")
    raise
except pymongo.errors.ServerSelectionTimeoutError as e:
    logger.error(f"Could not reach MongoDB server: {e}")
    raise
except pymongo.errors.PyMongoError as e:
    logger.error(f"MongoDB error while importing data: {e}")
    raise
except Exception as e:
    logger.exception(f"Unexpected error during data import: {e}")
    raise
```

    2026-04-28 19:40:14,713 [INFO] Connecting to MongoDB Atlas cluster
    2026-04-28 19:40:15,035 [INFO] MongoDB connection established
    2026-04-28 19:40:15,039 [INFO] Fetching documents from DS4320.Project2
    2026-04-28 19:40:19,646 [INFO] Loaded 2185 documents



```python
# Feature engineering: flatten readings and create aggregated features
data_records = []

for doc in documents:
    day = pd.to_datetime(doc['day'])
    day_of_year = day.dayofyear
    site_id = doc['site_id']
    
    # Aggregate readings by parameter
    readings_dict = {
        'day_of_year': day_of_year,
        'site_id': site_id,
        'date': day
    }
    
    # Extract values for each parameter type
    for reading in doc.get('readings', []):
        param = reading.get('parameter', '')
        value = reading.get('value')
        if pd.notna(value):
            key = f"{param}_value"
            if key not in readings_dict:
                readings_dict[key] = []
            readings_dict[key].append(value)
    
    # Aggregate the readings (mean, std, min, max)
    # Convert to list to avoid "dictionary changed size during iteration" error
    for key, values in list(readings_dict.items()):
        if isinstance(values, list) and values:
            readings_dict[f"{key}_mean"] = np.mean(values)
            readings_dict[f"{key}_std"] = np.std(values)
            readings_dict[f"{key}_min"] = np.min(values)
            readings_dict[f"{key}_max"] = np.max(values)
            readings_dict[f"{key}_count"] = len(values)
            del readings_dict[key]
    
    data_records.append(readings_dict)

# Convert to DataFrame
df = pd.DataFrame(data_records)
print(f"Created dataset with shape: {df.shape}")
print(f"\nFeatures: {df.columns.tolist()}")
print(f"Day of year range: {df['day_of_year'].min()} - {df['day_of_year'].max()}")
print(f"\nFirst few rows:")
print(df.head())
```

    Created dataset with shape: (2185, 78)
    
    Features: ['day_of_year', 'site_id', 'date', 'NO2_value_mean', 'NO2_value_std', 'NO2_value_min', 'NO2_value_max', 'NO2_value_count', 'OZONE_value_mean', 'OZONE_value_std', 'OZONE_value_min', 'OZONE_value_max', 'OZONE_value_count', 'BARPR_value_mean', 'BARPR_value_std', 'BARPR_value_min', 'BARPR_value_max', 'BARPR_value_count', 'CO_value_mean', 'CO_value_std', 'CO_value_min', 'CO_value_max', 'CO_value_count', 'PM2.5_value_mean', 'PM2.5_value_std', 'PM2.5_value_min', 'PM2.5_value_max', 'PM2.5_value_count', 'PRECIP_value_mean', 'PRECIP_value_std', 'PRECIP_value_min', 'PRECIP_value_max', 'PRECIP_value_count', 'RHUM_value_mean', 'RHUM_value_std', 'RHUM_value_min', 'RHUM_value_max', 'RHUM_value_count', 'RWS_value_mean', 'RWS_value_std', 'RWS_value_min', 'RWS_value_max', 'RWS_value_count', 'TEMP_value_mean', 'TEMP_value_std', 'TEMP_value_min', 'TEMP_value_max', 'TEMP_value_count', 'PM10_value_mean', 'PM10_value_std', 'PM10_value_min', 'PM10_value_max', 'PM10_value_count', 'PMC_value_mean', 'PMC_value_std', 'PMC_value_min', 'PMC_value_max', 'PMC_value_count', 'RWD_value_mean', 'RWD_value_std', 'RWD_value_min', 'RWD_value_max', 'RWD_value_count', 'WS_value_mean', 'WS_value_std', 'WS_value_min', 'WS_value_max', 'WS_value_count', 'NO_value_mean', 'NO_value_std', 'NO_value_min', 'NO_value_max', 'NO_value_count', 'NOX_value_mean', 'NOX_value_std', 'NOX_value_min', 'NOX_value_max', 'NOX_value_count']
    Day of year range: 1 - 365
    
    First few rows:
       day_of_year    site_id       date  NO2_value_mean  NO2_value_std  \
    0          109  110010050 2025-04-19           3.670       1.550838   
    1          109  110010051 2025-04-19           8.390       3.318870   
    2          109  110010041 2025-04-19           2.445       0.655343   
    3          109  110010043 2025-04-19             NaN            NaN   
    4          110  110010043 2025-04-20             NaN            NaN   
    
       NO2_value_min  NO2_value_max  NO2_value_count  OZONE_value_mean  \
    0            1.9            7.7             20.0         45.473684   
    1            3.9           17.1             20.0               NaN   
    2            1.4            3.9             20.0         44.894737   
    3            NaN            NaN              NaN         44.600000   
    4            NaN            NaN              NaN         40.869565   
    
       OZONE_value_std  ...  NO_value_mean  NO_value_std  NO_value_min  \
    0         4.816293  ...            NaN           NaN           NaN   
    1              NaN  ...            NaN           NaN           NaN   
    2         4.722786  ...            NaN           NaN           NaN   
    3         6.036555  ...            NaN           NaN           NaN   
    4         6.278357  ...            NaN           NaN           NaN   
    
       NO_value_max  NO_value_count  NOX_value_mean  NOX_value_std  NOX_value_min  \
    0           NaN             NaN             NaN            NaN            NaN   
    1           NaN             NaN             NaN            NaN            NaN   
    2           NaN             NaN             NaN            NaN            NaN   
    3           NaN             NaN             NaN            NaN            NaN   
    4           NaN             NaN             NaN            NaN            NaN   
    
       NOX_value_max  NOX_value_count  
    0            NaN              NaN  
    1            NaN              NaN  
    2            NaN              NaN  
    3            NaN              NaN  
    4            NaN              NaN  
    
    [5 rows x 78 columns]



```python
# Data cleaning and preparation
# Keep site_id/day_of_year as separate targets
model_df = df.drop(['date'], axis=1).copy()

feature_cols = [c for c in model_df.columns if c not in ['day_of_year', 'site_id']]
X = model_df[feature_cols]
X = X.fillna(X.mean())

# Targets for two tasks
y_day = model_df['day_of_year']
y_site = model_df['site_id'].astype(str)

print(f"Features shape: {X.shape}")
print(f"Day target range: {y_day.min():.0f} - {y_day.max():.0f}")
print(f"Site classes: {y_site.nunique()}")

# Split for day-of-year regression
X_train_day, X_test_day, y_train_day, y_test_day = train_test_split(
    X, y_day, test_size=0.2, random_state=42
)

# Split for site classification
X_train_site, X_test_site, y_train_site, y_test_site = train_test_split(
    X, y_site, test_size=0.2, random_state=42, stratify=y_site
)

# Scale features (separate scalers for each task)
scaler_day = StandardScaler()
X_train_day_scaled = scaler_day.fit_transform(X_train_day)
X_test_day_scaled = scaler_day.transform(X_test_day)

scaler_site = StandardScaler()
X_train_site_scaled = scaler_site.fit_transform(X_train_site)
X_test_site_scaled = scaler_site.transform(X_test_site)

print(f"Regression train/test sizes: {X_train_day_scaled.shape[0]} / {X_test_day_scaled.shape[0]}")
print(f"Classification train/test sizes: {X_train_site_scaled.shape[0]} / {X_test_site_scaled.shape[0]}")
```

    Features shape: (2185, 75)
    Day target range: 1 - 365
    Site classes: 6
    Regression train/test sizes: 1748 / 437
    Classification train/test sizes: 1748 / 437



```python
# Model 1: Day-of-year regression
rf_day_model = RandomForestRegressor(n_estimators=150, max_depth=15, random_state=42, n_jobs=-1)
rf_day_model.fit(X_train_day_scaled, y_train_day)

y_train_day_pred = rf_day_model.predict(X_train_day_scaled)
y_test_day_pred = rf_day_model.predict(X_test_day_scaled)

train_day_r2 = r2_score(y_train_day, y_train_day_pred)
test_day_r2 = r2_score(y_test_day, y_test_day_pred)
train_day_rmse = np.sqrt(mean_squared_error(y_train_day, y_train_day_pred))
test_day_rmse = np.sqrt(mean_squared_error(y_test_day, y_test_day_pred))

print("Day-of-Year Model (Regression)")
print(f"Training R² Score: {train_day_r2:.4f}")
print(f"Test R² Score: {test_day_r2:.4f}")
print(f"Training RMSE: {train_day_rmse:.2f} days")
print(f"Test RMSE: {test_day_rmse:.2f} days")

# Model 2: Site prediction classification
rf_site_model = RandomForestClassifier(n_estimators=200, max_depth=20, random_state=42, n_jobs=-1)
rf_site_model.fit(X_train_site_scaled, y_train_site)

y_train_site_pred = rf_site_model.predict(X_train_site_scaled)
y_test_site_pred = rf_site_model.predict(X_test_site_scaled)

train_site_acc = accuracy_score(y_train_site, y_train_site_pred)
test_site_acc = accuracy_score(y_test_site, y_test_site_pred)

print("\nSite Model (Classification)")
print(f"Training Accuracy: {train_site_acc:.4f}")
print(f"Test Accuracy: {test_site_acc:.4f}")

# Feature importance from both models
feature_importance_day = pd.DataFrame({
    'feature': feature_cols,
    'importance': rf_day_model.feature_importances_
}).sort_values('importance', ascending=False)

feature_importance_site = pd.DataFrame({
    'feature': feature_cols,
    'importance': rf_site_model.feature_importances_
}).sort_values('importance', ascending=False)

print("\nTop 10 Day-of-Year Features:")
print(feature_importance_day.head(10))

print("\nTop 10 Site-Prediction Features:")
print(feature_importance_site.head(10))
```

    Day-of-Year Model (Regression)
    Training R² Score: 0.7543
    Test R² Score: 0.1912
    Training RMSE: 52.13 days
    Test RMSE: 94.16 days
    
    Site Model (Classification)
    Training Accuracy: 0.9989
    Test Accuracy: 0.9336
    
    Top 10 Day-of-Year Features:
                 feature  importance
    20  PM2.5_value_mean    0.094111
    5   OZONE_value_mean    0.090464
    8    OZONE_value_max    0.088182
    6    OZONE_value_std    0.077585
    3      NO2_value_max    0.060501
    2      NO2_value_min    0.054161
    0     NO2_value_mean    0.051268
    21   PM2.5_value_std    0.047611
    23   PM2.5_value_max    0.044522
    22   PM2.5_value_min    0.043297
    
    Top 10 Site-Prediction Features:
                   feature  importance
    2        NO2_value_min    0.059692
    21     PM2.5_value_std    0.055355
    24   PM2.5_value_count    0.051902
    0       NO2_value_mean    0.047107
    23     PM2.5_value_max    0.046227
    3        NO2_value_max    0.043560
    20    PM2.5_value_mean    0.042743
    22     PM2.5_value_min    0.042578
    29  PRECIP_value_count    0.035457
    1        NO2_value_std    0.033367



```python
# Visualization for both models
fig, axes = plt.subplots(2, 2, figsize=(16, 12))
fig.suptitle('Air Quality Prediction Models: Day-of-Year & Site Classification', 
             fontsize=16, fontweight='bold', y=0.995)

# Plot 1: Day model - Actual vs Predicted (Test set)
axes[0, 0].scatter(y_test_day, y_test_day_pred, alpha=0.6, s=40, color='#2E86AB', edgecolors='white', linewidth=0.5)
axes[0, 0].plot([y_test_day.min(), y_test_day.max()], [y_test_day.min(), y_test_day.max()], 
                'r--', lw=2.5, label='Perfect Prediction', alpha=0.8)
axes[0, 0].set_xlabel('Actual Day of Year', fontweight='bold')
axes[0, 0].set_ylabel('Predicted Day of Year', fontweight='bold')
axes[0, 0].set_title(f'Day-of-Year Regression (R² = {test_day_r2:.4f}, RMSE = {test_day_rmse:.2f} days)', 
                     fontweight='bold', pad=10)
axes[0, 0].grid(True, alpha=0.3, linestyle='--')
axes[0, 0].legend(loc='upper left', framealpha=0.95)

# Plot 2: Day model residuals
residuals_day = y_test_day - y_test_day_pred
axes[0, 1].scatter(y_test_day_pred, residuals_day, alpha=0.6, s=40, color='#A23B72', edgecolors='white', linewidth=0.5)
axes[0, 1].axhline(y=0, color='r', linestyle='--', lw=2.5, label='Zero Error', alpha=0.8)
axes[0, 1].fill_between(axes[0, 1].get_xlim(), -10, 10, alpha=0.1, color='red')
axes[0, 1].set_xlabel('Predicted Day of Year', fontweight='bold')
axes[0, 1].set_ylabel('Residuals (days)', fontweight='bold')
axes[0, 1].set_title(f'Residual Plot (Mean = {residuals_day.mean():.2f}, Std = {residuals_day.std():.2f})', 
                     fontweight='bold', pad=10)
axes[0, 1].grid(True, alpha=0.3, linestyle='--')
axes[0, 1].legend(loc='upper left', framealpha=0.95)

# Plot 3: Feature Importance (day model)
top_day_features = feature_importance_day.head(12)
colors_day = sns.color_palette("Blues_r", len(top_day_features))
bars_day = axes[1, 0].barh(range(len(top_day_features)), top_day_features['importance'], 
                            color=colors_day, edgecolor='white', linewidth=1.5)
axes[1, 0].set_yticks(range(len(top_day_features)))
axes[1, 0].set_yticklabels(top_day_features['feature'])
axes[1, 0].set_xlabel('Importance Score', fontweight='bold')
axes[1, 0].set_title('Top 12 Features: Day-of-Year Model', fontweight='bold', pad=10)
axes[1, 0].invert_yaxis()
axes[1, 0].grid(True, alpha=0.3, axis='x', linestyle='--')

# Plot 4: Site-model feature importance
top_site_features = feature_importance_site.head(12)
colors_site = sns.color_palette("Oranges_r", len(top_site_features))
bars_site = axes[1, 1].barh(range(len(top_site_features)), top_site_features['importance'], 
                             color=colors_site, edgecolor='white', linewidth=1.5)
axes[1, 1].set_yticks(range(len(top_site_features)))
axes[1, 1].set_yticklabels(top_site_features['feature'])
axes[1, 1].set_xlabel('Importance Score', fontweight='bold')
axes[1, 1].set_title(f'Top 12 Features: Site Classification (Accuracy = {test_site_acc:.4f})', 
                     fontweight='bold', pad=10)
axes[1, 1].invert_yaxis()
axes[1, 1].grid(True, alpha=0.3, axis='x', linestyle='--')

# Adjust layout with proper spacing
plt.tight_layout()
plt.savefig('air_quality_models.png', dpi=300, bbox_inches='tight', facecolor='white')
plt.show()
print("Figure saved as 'air_quality_models.png'")
```


    
![png](pipeline2_files/pipeline2_7_0.png)
    


    Figure saved as 'air_quality_models.png'


## Time Series Model


```python
# Time-series model: forecast a daily air-quality signal
# Build a daily series from the first available "*_mean" feature.
mean_feature_cols = [c for c in df.columns if c.endswith('_mean')]
if not mean_feature_cols:
    raise ValueError("No aggregated mean features found for time-series modeling.")

ts_feature = mean_feature_cols[0]
print(f"Using time-series target feature: {ts_feature}")

# Aggregate to one value per day across sites and sort chronologically
ts_df = (
    df[['date', ts_feature]]
    .dropna()
    .groupby('date', as_index=False)[ts_feature]
    .mean()
    .sort_values('date')
)

if len(ts_df) < 30:
    raise ValueError("Not enough daily observations for a reliable time-series split.")

series = ts_df.set_index('date')[ts_feature].asfreq('D')
series = series.interpolate(limit_direction='both')

# Hold out last 20% of days for out-of-sample evaluation
split_idx = int(len(series) * 0.8)
train_ts = series.iloc[:split_idx]
test_ts = series.iloc[split_idx:]

if len(test_ts) == 0:
    raise ValueError("Time-series test set is empty after split.")

try:
    from statsmodels.tsa.holtwinters import ExponentialSmoothing

    ts_model = ExponentialSmoothing(
        train_ts,
        trend='add',
        seasonal=None,
        initialization_method='estimated'
    ).fit(optimized=True)

    ts_forecast = ts_model.forecast(steps=len(test_ts))
    model_name = 'ExponentialSmoothing (additive trend)'
except Exception as e:
    # Safe fallback if statsmodels is unavailable or fit fails
    print(f"Statsmodels model unavailable/failed ({e}); falling back to naive forecast.")
    ts_forecast = pd.Series(train_ts.iloc[-1], index=test_ts.index)
    model_name = 'Naive (last-value)'

# Evaluate
rmse_ts = np.sqrt(mean_squared_error(test_ts, ts_forecast))
mae_ts = np.mean(np.abs(test_ts - ts_forecast))

print(f"Time-Series Model: {model_name}")
print(f"Train size: {len(train_ts)}, Test size: {len(test_ts)}")
print(f"Test RMSE: {rmse_ts:.4f}")
print(f"Test MAE: {mae_ts:.4f}")

# Publication-quality plot
fig, ax = plt.subplots(figsize=(14, 7))

# Plot training data
ax.plot(train_ts.index, train_ts.values, label='Training Data', linewidth=2.5, 
        color='#2E86AB', alpha=0.85, marker='o', markersize=4)

# Plot test data
ax.plot(test_ts.index, test_ts.values, label='Actual Test Data', linewidth=2.5, 
        color='#A23B72', alpha=0.85, marker='s', markersize=4)

# Plot forecast
ax.plot(ts_forecast.index, ts_forecast.values, label='Forecast', linewidth=2.5, 
        color='#F18F01', alpha=0.85, linestyle='--', marker='^', markersize=4)

# Shade the test period
ax.axvspan(test_ts.index[0], test_ts.index[-1], alpha=0.1, color='gray', label='Test Period')

# Formatting
ax.set_xlabel('Date', fontweight='bold', fontsize=12)
ax.set_ylabel(ts_feature, fontweight='bold', fontsize=12)
ax.set_title(f'Time-Series Forecast: {ts_feature}\n{model_name} (RMSE={rmse_ts:.4f}, MAE={mae_ts:.4f})', 
             fontweight='bold', fontsize=14, pad=15)
ax.legend(loc='best', framealpha=0.95, fontsize=11, edgecolor='black')
ax.grid(True, alpha=0.3, linestyle='--')

# Improve x-axis date formatting
fig.autofmt_xdate(rotation=45, ha='right')

plt.tight_layout()
plt.savefig('time_series_forecast.png', dpi=300, bbox_inches='tight', facecolor='white')
plt.show()
print("Figure saved as 'time_series_forecast.png'")
```

    Using time-series target feature: NO2_value_mean
    Time-Series Model: ExponentialSmoothing (additive trend)
    Train size: 292, Test size: 74
    Test RMSE: 8.0382
    Test MAE: 6.9785



    
![png](pipeline2_files/pipeline2_9_1.png)
    


    Figure saved as 'time_series_forecast.png'


## Visualisation Rationale & Discussion

I chose to use these plots of the models to represent their strengths, weaknesses, and heaviest impacts most clearly. The scatterplots in the above section demonstrate the performance of the random forest regressor, indicating in the first plot how the prediction for each day compared to the actual value and in the second plot what the error in prediction was for each day. This shows how the Random Forest regression performed relative to expectations, with a perfect prediction marked in the middle. The plots below describe the biggest features of the Random Forest Regression and Classification models, used to predict day of the year and the site of measurement respectively. These show which of the observed quantities are most informative of what separates either the day or the site from others. The measured quantities that influence decisions the most are notably different for predicting the day of the year and for predicting the measurement site. The final plot represents the results of the third model, plotting the time series data as it performed on the train and test partitions of the year of data collected. 
