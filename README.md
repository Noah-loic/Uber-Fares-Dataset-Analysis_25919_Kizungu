

# Uber Trip Data Analysis Report
## Names: KIZUNGU SHYAKA NOAH LOIC
## ID: 25919
## Group: B

## Tools Used

- **Python (Pandas, Numpy, Seaborn, Matplotlib)**  
- **Power BI Desktop**  
- **Git & GitHub**

## 1. Objective
The goal of this notebook is to analyze Uber ride data to:
- Clean and preprocess the dataset.
- Understand fare patterns.
- Visualize trends and detect anomalies such as outliers or missing values.

## 2. Steps Performed

### Step 1: Import Libraries
Essential Python libraries such as `pandas`, `numpy`, `matplotlib`, and `seaborn` were imported for data handling and visualization. Warnings were suppressed to reduce clutter.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings('ignore')
```

### Step 2: Load Dataset
The dataset (`uber.csv`) was loaded using `pandas` and the unnamed index column was renamed to `Ride_ID` for clarity.
```python
df = pd.read_csv('uber.csv')
```
```python
print(df.rename(columns={'Unnamed: 0': 'Ride_ID'}, inplace=True))
```

### Step 3: Data Inspection
Several commands were used to examine the dataset:
- `.head()`: To view the first few rows.
- `.info()`: To get column types and non-null counts.
- `.isnull().sum()`: To detect missing values.
- `.describe()`: To understand statistical properties (e.g., min, max, mean, quartiles).

 ```python
iprint("First 5 rows:")
print(df.head())

print("\nDataset Info:")
print(df.info())

print("\nMissing Values:")
print(df.isnull().sum())

print("\nDescriptive Statistics:")
print(df.describe())
```

### Step 4: Data Cleaning
Key cleaning operations included:
- Dropping rows with missing or invalid critical values like `fare_amount`, `pickup_longitude`, `pickup_latitude`, `dropoff_longitude`, or `dropoff_latitude`.
- Removing duplicate rows.
- Converting `pickup_datetime` to proper datetime format and removing rows where conversion failed.
- Saving the cleaned dataset as `uber_cleaned.csv`.

```python
df_clean = df.dropna(subset=['fare_amount', 'pickup_longitude', 'pickup_latitude', 'dropoff_longitude', 'dropoff_latitude'])

df_clean = df_clean.drop_duplicates()
if 'pickup_datetime' in df_clean.columns:
    df_clean['pickup_datetime'] = pd.to_datetime(df_clean['pickup_datetime'], errors='coerce')
    
df_clean = df_clean.dropna(subset=['pickup_datetime'])

df_clean.to_csv('uber_cleaned.csv', index=False)

print("\nCleaned data saved to 'data/uber_cleaned.csv'")
```
## 3. Data Visualization and Interpretation

### Histogram of Fare Amounts
**Code Used:**
```python
# Histogram of fares
plt.figure(figsize=(10,5))
sns.histplot(df_clean['fare_amount'], bins=50, kde=True)
plt.title('Fare Amount Distribution')
plt.xlabel('Fare Amount ($)')
plt.ylabel('Frequency')
plt.xlim(0,100)  # Limit x-axis for better visibility
plt.show()
```
<img width="890" height="470" alt="true history" src="https://github.com/user-attachments/assets/801b8c68-891b-48ab-8708-0252c409ad64" />

**Purpose:** To view the distribution of fare amounts.  
**Observation:** Most fares are clustered below $50, with a peak in the $5–$15 range.  
**Conclusion:** The majority of Uber trips are relatively inexpensive, suggesting short-distance or urban travel.


###  Boxplot of Fare Amounts

**Code Used:**
```python
plt.figure(figsize=(8,4))
sns.boxplot(x=df_clean['fare_amount'])
plt.title('Fare Amount Boxplot')
plt.show()
```
<img width="644" height="393" alt="boxplot" src="https://github.com/user-attachments/assets/5b9941e1-6e81-4194-9326-019b00adbba5" />

**Purpose:** To detect outliers and understand fare spread.  
**Observation:** There are several extreme high-value fares that lie far outside the interquartile range.  
**Conclusion:** The dataset contains outliers which may represent:
- Long-distance trips.
- Data entry errors.
- Unusual ride patterns (e.g., surge pricing or airport trips).

## Scatter Plot: Fare vs Distance

### Calculating distance using haversine formula

#### Feature Engineering: Trip Distance Calculation (Haversine Formula)

The original dataset includes pickup and dropoff coordinates but does **not** contain a direct `distance` column.  
To accurately measure trip lengths, we calculated the straight-line (great-circle) distance between the pickup and dropoff points using the **Haversine formula**.

---

#### Why This Matters

Understanding trip distance is crucial for analyzing:

- How **distance affects fare amount**
- Identifying **short vs long trips**
- Spotting **potential data anomalies** (e.g., high fare for short distance)
```python
def haversine_distance(lat1, lon1, lat2, lon2):
    """
    Calculate the great-circle distance between two points 
    on the Earth (specified in decimal degrees).
    Returns distance in miles.
    """
    # Convert degrees to radians
    lat1, lon1, lat2, lon2 = map(np.radians, [lat1, lon1, lat2, lon2])
    
    # Haversine formula
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    
    a = np.sin(dlat/2.0)**2 + np.cos(lat1) * np.cos(lat2) * np.sin(dlon/2.0)**2
    c = 2 * np.arcsin(np.sqrt(a))
    
    # Radius of Earth in miles
    r = 3956  
    return c * r)
```
### Creating and adding distance column to the dataset
```python
# Create a new 'distance' column
df_clean['distance'] = haversine_distance(
    df_clean['pickup_latitude'], df_clean['pickup_longitude'],
    df_clean['dropoff_latitude'], df_clean['dropoff_longitude']
)
df_clean.to_csv('uber_cleaned_with_feature.csv', index=False)

print("\nEnhanced dataset saved to 'uber_cleaned_with_feature.csv'")
print("\nFinal columns in dataset:", df_clean.columns.tolist())
```
**Code Used to plot :**
```python
if 'distance' in df_clean.columns:
    plt.figure(figsize=(8,5))
    sns.scatterplot(x='distance', y='fare_amount', data=df_clean)
    plt.title('Fare vs Distance')
    plt.xlabel('Distance (miles)')
    plt.ylabel('Fare Amount ($)')
    plt.show()
```
<img width="695" height="470" alt="scatterplot" src="https://github.com/user-attachments/assets/495421cd-aa65-4993-9089-9627572b54d4" />


**Interpretation:**
- The scatter plot shows the relationship between distance traveled and fare amount.
- **General Trend:** As the distance increases, the fare also tends to increase.
- **Outliers Present:**
  - Very high fares for short distances → possibly due to surge pricing, waiting time, or data entry errors.
  - Low fares for long distances → might be anomalies or discounts.
- **Conclusion:** There is a positive correlation between distance and fare amount, but it is not perfectly linear due to external factors like traffic, tolls, and pricing algorithms.

## 4 Feature Engineering: Time-Based Features

To enhance the dataset for temporal analysis, we extracted new columns from the `pickup_datetime` field and created a peak-hour indicator. These engineered features help identify trends in rider behavior and support powerful time-based visualizations in Power BI.

**Code Used**

```python
# Extract date/time features
df_clean['hour'] = df_clean['pickup_datetime'].dt.hour
df_clean['day'] = df_clean['pickup_datetime'].dt.day
df_clean['month'] = df_clean['pickup_datetime'].dt.month
df_clean['day_of_week'] = df_clean['pickup_datetime'].dt.dayofweek

# Peak/off-peak indicator (Example: 7–9 AM & 5–7 PM)
def peak_hour(hour):
    return 1 if hour in [7, 8, 9, 17, 18, 19] else 0

df_clean['peak_hour'] = df_clean['hour'].apply(peak_hour)
```

 **Purpose: Extract Hour, Day, Month, and Day of Week**

These new columns allow us to analyze ride patterns across different timescales:
- **hour:** For hourly trends (e.g., busiest hours)
- **day:** For daily activity trends
- **month:** For monthly or seasonal trends
- **day_of_week:** To distinguish between weekdays and weekends

### Create a `peak_hour` Flag
Identifies whether a ride occurred during peak hours (7–9 AM or 5–7 PM).

**Encoded as:**
- `1` = Peak hour
- `0` = Off-peak

  <img width="344" height="171" alt="check for new features" src="https://github.com/user-attachments/assets/aad121bc-e806-4578-8368-39487e52d7bd" />


### barplot: fare_amount VS Hour of the day
**Code Used:**
```python
Explain the observation and purpose of the infor(code) and image i provided:
# Group data by hour and calculate average fare
hourly_fare = df_clean.groupby('hour')['fare_amount'].mean().reset_index()

# Plot
plt.figure(figsize=(10,5))
sns.barplot(x='hour', y='fare_amount', data=hourly_fare, palette='viridis')
plt.title('Average Fare Amount by Hour of Day')
plt.xlabel('Hour of Day (0-23)')
plt.ylabel('Average Fare Amount ($)')
plt.xticks(range(0,24))
plt.show()
```
<img width="880" height="483" alt="Screenshot 2025-07-23 23 40 45" src="https://github.com/user-attachments/assets/47663019-005f-4ebc-acc2-e823a449ce4d" />

**Purpose:**
The code and chart aim to analyze and visualize how fare prices vary throughout the day, likely to identify patterns or peak pricing periods. This could be useful for pricing strategies, demand forecasting, or understanding customer behavior based on time of day. The use of a bar plot with a 'viridis' palette enhances readability and highlights trends effectively.

**Observations:**
- The highest average fare occurs around 5 AM, reaching approximately $16.
- Fares are relatively high from midnight (0) to 6 AM, ranging from $10 to $16, suggesting peak pricing during early morning hours.
- The average fare drops to around $10-$12 from 6 AM to midday (12 PM), indicating a more stable rate during morning and late morning.
- From noon (12 PM) to evening (around 18 PM), fares remain fairly consistent, hovering between $10 and $12.
- A slight increase is observed in the late evening and night (18 PM to 23 PM), with the fare peaking again at around $12-$13 by 23 PM.




### Heatmap: Correlation Between Numeric Variables
**Code Used:**
```python
numeric_cols = df_clean.select_dtypes(include=[np.number])
plt.figure(figsize=(10,8))
sns.heatmap(numeric_cols.corr(), annot=True, cmap='coolwarm', fmt=".2f")
plt.title('Correlation Heatmap')
plt.show()
```
<img width="893" height="790" alt="heatmap" src="https://github.com/user-attachments/assets/dce855c4-9ab6-4f4e-9f95-ade69140188e" />


**Interpretation:**
- The heatmap shows pairwise correlations between all numeric variables in the dataset.
- **Key Insight:** A moderate to strong correlation between `distance` and `fare_amount` (e.g., ~0.6 to 0.8), confirming the trend seen in the scatter plot.
- Other variables (like latitudes and longitudes) typically show weak or no correlation with fare since they are spatial rather than numerical magnitudes.
- **Conclusion:** The heatmap confirms that fare is mainly influenced by distance, validating the scatter plot. Other features may be useful for location-based clustering or advanced modeling but do not directly correlate with fare
- 
# For more in depth view and understanding you can refer to the attach jupyterNotebook file


# Power BI Visualization Insights

### Dashboard
<img width="659" height="365" alt="dashboard" src="https://github.com/user-attachments/assets/c556120c-86c9-4d8b-8000-0d51481bb6eb" />


### 1. Sum of Fare Amount by Hour
<img width="340" height="205" alt="sum of fare by hour" src="https://github.com/user-attachments/assets/aa925e9a-ed3c-4b3a-8dab-6e13108b54db" />

**Observation:** Fare amounts peak between hours 6–10 and 16–20, with a noticeable dip around midnight to early morning (0–5).
**Insight:** Peak hours likely correspond to morning (7–9 AM) and evening (5–7 PM) rush hours, indicating higher demand or longer trips during commute times.


### 2. Passenger Count by Fare Amount
<img width="355" height="239" alt="sum of passenger count by fare amount" src="https://github.com/user-attachments/assets/a60acb83-ad51-4bd2-a5b9-140735e74a97" />

**Observation:** 
- Most rides have fares between **$0–$50**.
- Very few rides exceed $100, and extreme fares (e.g., $400+) are outliers.

**Insight:** Most rides with passengers are low-cost, with occasional high-fare rides possibly involving multiple passengers or long distances.

### 3. Sum of Fare by Day of the Week
<img width="535" height="299" alt="sum of fare by day of the week" src="https://github.com/user-attachments/assets/2032da9a-b17c-4200-98d2-769454e46902" />

**Observation:** 
- **Wednesdays, Thursdays, and Fridays** are the most lucrative days for Uber fares.
- **Sunday** has the lowest fare total — potentially due to reduced commuting.
- The total fare amount is relatively consistent across all days of the week, with a slight peak around day 3 (Wednesday) and day 4 (Thursday).
- **Insight:** Mid-week days might see higher ride activity or longer trips, though the variation is minimal, suggesting stable demand throughout the week.

### 4. Sum of fare_amount by Year, Quarter, Month and Day
<img width="398" height="227" alt="sum of fare by year" src="https://github.com/user-attachments/assets/65b07eb4-fdba-4a05-9e7a-07bceab3925c" />

- **Observation:** The fare amount shows significant fluctuations over time, with peaks and troughs from 2010 to 2014, followed by a sharp decline.
- **Insight:** There may have been a surge in Uber usage or longer trips around 2012–2013, with a possible drop-off or data collection change after 2014.

### 5. Fare Distribution
<img width="644" height="377" alt="fare distribution" src="https://github.com/user-attachments/assets/2c55bdcc-871c-4c69-9cff-303b648f3052" />

- **Observation:** The majority of fares are concentrated below $100, with a steep drop-off beyond that, and a few outliers up to $500.
- **Insight:** Most Uber trips are short and inexpensive, with rare high-value trips possibly due to long distances, surge pricing, or errors.

### 6. Sum of fare_amount by month
<img width="410" height="215" alt="sum of by month" src="https://github.com/user-attachments/assets/01cb2bf3-479b-4014-99af-aea52b6bcb20" />

- **Observation:** Fare amounts vary slightly across months, with a peak around months 2–4 and a dip around month 7.
- **Insight:** Seasonal trends might influence ride demand, with higher activity in early months (possibly winter or early spring) and lower in mid-year.

### 7. Sum of fare_amount by day
<img width="520" height="304" alt="sum of fare by the day" src="https://github.com/user-attachments/assets/f765c1cb-7975-44fb-8e06-abb98246a843" />

- **Observation:** Fare amounts are relatively stable throughout the month, with a slight increase around days 15–25.
- **Insight:** Mid-month periods might see higher ride activity, possibly due to pay cycles or increased urban movement.

### 8. Sum of fare_amount by peak_hour
- **Observation:** Approximately 71.01% of the total fare amount occurs during peak hours (1), while 28.99% occurs off-peak (0).
- **Insight:** Peak hours (7–9 AM and 5–7 PM) generate significantly more revenue, reflecting higher demand or surge pricing during these times.

### 9. Sum of passenger_count by peak_hour
- **Observation:** About 70.59% of passenger counts occur during peak hours (1), while 29.41% occur off-peak (0).
- **Insight:** Peak hours not only increase fare amounts but also involve more passengers, suggesting crowded rides or higher utilization during commute times.
<img width="250" height="309" alt="pie chart" src="https://github.com/user-attachments/assets/34ff2caa-15ff-4b33-8626-02bac43ef49c" />
