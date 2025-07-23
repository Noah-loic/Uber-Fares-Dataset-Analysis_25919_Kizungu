## Uber Trip Data Analysis Report

## 1. Objective
The goal of this notebook is to analyze Uber ride data to:
- Clean and preprocess the dataset.
- Understand fare patterns.
- Visualize trends and detect anomalies such as outliers or missing values.

## 2. Steps Performed

### Step 1: Import Libraries
Essential Python libraries such as `pandas`, `numpy`, `matplotlib`, and `seaborn` were imported for data handling and visualization. Warnings were suppressed to reduce clutter.

### Step 2: Load Dataset
The dataset (`uber.csv`) was loaded using `pandas` and the unnamed index column was renamed to `Ride_ID` for clarity.

### Step 3: Data Inspection
Several commands were used to examine the dataset:
- `.head()`: To view the first few rows.
- `.info()`: To get column types and non-null counts.
- `.isnull().sum()`: To detect missing values.
- `.describe()`: To understand statistical properties (e.g., min, max, mean, quartiles).

### Step 4: Data Cleaning
Key cleaning operations included:
- Dropping rows with missing or invalid critical values like `fare_amount`, `pickup_longitude`, `pickup_latitude`, `dropoff_longitude`, or `dropoff_latitude`.
- Removing duplicate rows.
- Converting `pickup_datetime` to proper datetime format and removing rows where conversion failed.
- Saving the cleaned dataset as `uber_cleaned.csv`.

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
**Purpose:** To detect outliers and understand fare spread.  
**Observation:** There are several extreme high-value fares that lie far outside the interquartile range.  
**Conclusion:** The dataset contains outliers which may represent:
- Long-distance trips.
- Data entry errors.
- Unusual ride patterns (e.g., surge pricing or airport trips).

### Scatter Plot: Fare vs Distance
**Code Used:**
```python
if 'distance' in df_clean.columns:
    plt.figure(figsize=(8,5))
    sns.scatterplot(x='distance', y='fare_amount', data=df_clean)
    plt.title('Fare vs Distance')
    plt.xlabel('Distance (miles)')
    plt.ylabel('Fare Amount ($)')
    plt.show()
```


**Interpretation:**
- The scatter plot shows the relationship between distance traveled and fare amount.
- **General Trend:** As the distance increases, the fare also tends to increase.
- **Outliers Present:**
  - Very high fares for short distances → possibly due to surge pricing, waiting time, or data entry errors.
  - Low fares for long distances → might be anomalies or discounts.
- **Conclusion:** There is a positive correlation between distance and fare amount, but it is not perfectly linear due to external factors like traffic, tolls, and pricing algorithms.

### Heatmap: Correlation Between Numeric Variables
**Code Used:**
```python
numeric_cols = df_clean.select_dtypes(include=[np.number])
plt.figure(figsize=(10,8))
sns.heatmap(numeric_cols.corr(), annot=True, cmap='coolwarm', fmt=".2f")
plt.title('Correlation Heatmap')
plt.show()
```

**Interpretation:**
- The heatmap shows pairwise correlations between all numeric variables in the dataset.
- **Key Insight:** A moderate to strong correlation between `distance` and `fare_amount` (e.g., ~0.6 to 0.8), confirming the trend seen in the scatter plot.
- Other variables (like latitudes and longitudes) typically show weak or no correlation with fare since they are spatial rather than numerical magnitudes.
- **Conclusion:** The heatmap confirms that fare is mainly influenced by distance, validating the scatter plot. Other features may be useful for location-based clustering or advanced modeling but do not directly correlate with fare.



