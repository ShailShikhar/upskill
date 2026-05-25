# NYC Taxi Notebook - Quick Reference & Troubleshooting Guide

## Quick Reference: What Each Section Does

### 1. Data Ingestion & Initial Inspection
**Purpose:** Load data and verify it's complete and in expected format
- Loads 3 datasets from different sources (Parquet, CSV)
- Checks dimensions: 3M rows × 19 cols (taxi), 265 × 4 (zones), 13K × 97 (weather)
- Verifies data types are appropriate for operations
- Handles timezone localization for NYC data
**Key Insight:** Large datasets require format (Parquet) for efficiency

### 2. Data Understanding & Semantic Validation  
**Purpose:** Verify data means what we think it means
- Confirms fare columns sum correctly
- Validates domain rules (distances ≥ 0, times go forward)
- Identifies leakage columns (data only known after outcome)
- Creates trip duration from timestamps
**Key Insight:** Column names aren't always self-explanatory; validate meanings

### 3. Data Quality Checks
**Purpose:** Identify data issues before analysis
- 0 missing values in taxi data (clean!)
- 0 exact duplicates (good data integrity)
- Some zones have no representation (coverage gaps)
- Trip duration outliers exist (15+ hour trips)
**Key Insight:** Real data always has issues; systematic checking finds them

### 4. Data Integration (Combining Datasets)
**Purpose:** Merge taxi, zone, and weather into single dataset
- Adds zone names to pickup/dropoff location IDs
- Matches weather to nearest hour (not exact second)
- Preserves all 3M taxi records (no loss)
- Final: 3M rows × 31 columns
**Key Insight:** Different data granularities require thoughtful merging

### 5. Target Variable Analysis
**Purpose:** Identify what we want to predict
- Fare amount: $2.50-$250+, right-skewed (most trips $10-20)
- Trip duration: 5 minutes to 6+ hours, depends on traffic
- Hourly demand: highly variable, seasonal spikes (rush hour)
**Key Insight:** Different targets need different modeling approaches

### 6-16. Data Preparation & Modeling
**Purpose:** Make data model-ready and build predictions
- Handle missing values (none needed here)
- Detect and treat outliers (long-distance airport trips vs errors)
- Engineer features (rush hour, weather interactions)
- Encode categories (zones, payment types)
- Select best features (correlation, importance)
- Split train/test respecting temporal order
**Key Insight:** 80% of time is preparation; 20% is modeling

---

## Errors & Fixes Quick Lookup

### Problem: "InvalidIndexError: Reindexing only valid with uniquely valued Index objects"
**When:** When trying to concatenate DataFrames side-by-side
```python
# ❌ WRONG: Non-unique indices crash concat
taxi_idx = df.set_index('datetime')  # Many rows per datetime!
side_by_side = pd.concat([taxi_idx, weather_idx], axis=1)  # CRASH

# ✅ RIGHT: Aggregate to unique values first
weather_hourly = weather.groupby(weather.index.floor('h')).mean()  # Now unique!
side_by_side = pd.concat([taxi_indexed, weather_hourly], axis=1)  # Works
```

### Problem: "MemoryError: Unable to allocate X GiB"
**When:** Dataset too large to fit in memory (3M rows × 97 cols)
```python
# ❌ WRONG: All data in memory
df_merged = pd.merge_asof(taxi_large, weather_huge, on='time')  # OOM!

# ✅ RIGHT: Reduce dimensions before merge
weather_key = weather[['DATE', 'TMP', 'AA1']]  # Select key cols only
weather_hourly = weather_key.groupby('hour').mean()  # Reduce rows
df_merged = taxi.merge(weather_hourly, on='pickup_hour', how='left')  # Fits!
```

### Problem: "TypeError: Could not convert string to numeric"
**When:** Numeric column stored as strings with special characters
```python
# ❌ WRONG: Try to sum/mean strings
weather['TMP'].mean()  # ERROR: string '+0100,5'

# ✅ RIGHT: Clean before conversion
weather['TMP'] = (
    weather['TMP']
    .str.replace('+', '', regex=False)
    .str.replace(',', '.')
    .astype(float)
)
weather['TMP'].mean()  # Now works!
```

### Problem: "ValueError: Merge on datetime64[ns, America/New_York] and datetime64[ns]"
**When:** Timezone mismatch on merge keys
```python
# ❌ WRONG: Different timezone states
left_col = df['time']  # datetime64[ns, America/New_York]
right_col = weather['datetime']  # datetime64[ns] (no timezone)
pd.merge(df, weather, left_on=left_col, right_on=right_col)  # ERROR

# ✅ RIGHT: Harmonize timezones
left_col = df['time'].dt.tz_localize(None)  # Remove timezone
pd.merge(df, weather, left_on=left_col, right_on=right_col)  # Works
```

---

## Common Preprocessing Checklist

Use this before modeling:

### Data Loading ✓
- [ ] File exists and path is correct
- [ ] Data loads without encoding errors
- [ ] Row count matches expected (3M for taxi data)
- [ ] Column names are readable

### Data Types ✓
- [ ] datetime columns are datetime64, not object
- [ ] Numeric columns are int64/float64, not object
- [ ] Categorical columns are object or category
- [ ] All expected columns present

### Missing Values ✓
- [ ] Checked with `.isna().sum()`
- [ ] Decided: drop, impute, or flag
- [ ] Imputation strategy documented
- [ ] No unintended silent NaNs created

### Duplicates ✓
- [ ] Checked for exact row duplicates (`.duplicated().sum()`)
- [ ] Checked for business logic duplicates (same trip twice)
- [ ] Decided: keep, remove, or investigate
- [ ] Documented why duplicates exist (if any)

### Outliers ✓
- [ ] Calculated IQR and identified outliers
- [ ] Visualized distribution (histogram)
- [ ] Decided: keep, cap, transform, or remove
- [ ] Documented impact on statistics

### Joins ✓
- [ ] Verified join key data types match
- [ ] Checked for null in join keys
- [ ] Verified uniqueness assumptions
- [ ] Used `indicator=True` to diagnose joins
- [ ] Validated row count after join

### Encoding ✓
- [ ] All categorical variables identified
- [ ] Chosen encoding method per variable
- [ ] Verified output is numeric
- [ ] Checked for unexpected NaNs post-encoding

### Scaling ✓
- [ ] Fit scaler on train data only
- [ ] Applied to both train and test
- [ ] Verified mean≈0, std≈1 (if standardized)
- [ ] Documented scaling for deployment

### Splits ✓
- [ ] Used time-based split (for time-series)
- [ ] No future data in training set
- [ ] Test set is truly unseen
- [ ] Stratified if needed (for imbalanced data)

### Final Validation ✓
- [ ] X and y are separate
- [ ] No leakage columns in X
- [ ] No NaN values remaining
- [ ] Summary statistics look reasonable
- [ ] Ready to model!

---

## Performance Optimization Tips

### Reading Large Files
```python
# ✓ FAST: Parquet with column selection
df = pd.read_parquet('taxi.parquet', columns=['pickup_time', 'fare', 'distance'])

# ✗ SLOW: Read all CSV columns
df = pd.read_csv('taxi.csv')
```

### Memory-Efficient Merges
```python
# ✓ Smart: Aggregate first, then join
weather_hourly = weather.groupby('hour').agg({'temp': 'mean'})  # 13K → 744 rows
taxi = taxi.merge(weather_hourly, on='hour', how='left')  # Much smaller!

# ✗ Wasteful: Direct merge of large tables
taxi = taxi.merge(weather, on='exact_time')  # MemoryError!
```

### Type Optimization
```python
# ✓ Memory-efficient: Use category for low cardinality
df['payment_type'] = df['payment_type'].astype('category')  # 6 values

# ✗ Memory waste: Keep as object
df['payment_type'] = df['payment_type'].astype('object')  # Duplicated strings
```

### Filtering Early
```python
# ✓ FAST: Filter before operations
jan_data = df[df['date'].dt.month == 1]  # Remove 11/12 of data
jan_stats = jan_data['fare'].mean()

# ✗ SLOW: Compute on all data
all_stats = df['fare'].mean()  # Includes non-Jan months
jan_stats = all_stats  # Wrong!
```

---

## Debugging Tips

### "My merge lost data!"
**Diagnose with:**
```python
# Check join key coverage
merged = df1.merge(df2, on='key', how='left', indicator=True)
print(merged['_merge'].value_counts())

# Output:
# left_only  = rows in df1 not found in df2
# both       = matched rows
# right_only = rows in df2 not found in df1 (shouldn't exist for left join)
```

### "My dtype changed unexpectedly"
**Check with:**
```python
df.dtypes  # See all column types
df['column'].dtype  # Check specific column
df.info()  # Comprehensive summary
```

### "Which values are NaN?"
**Locate with:**
```python
df[df['column'].isna()]  # Show all rows with NaN in column
df.isna().sum()  # Count NaNs per column
df.isna().sum(axis=1)  # Count NaNs per row
```

### "Which rows have duplicates?"
**Find with:**
```python
# Find exact duplicates
dupes = df[df.duplicated(keep=False)]  # All duplicate rows (keep=False shows all)

# Find composite key duplicates
dupes = df[df.duplicated(subset=['col1', 'col2'], keep=False)]
```

---

## Useful Code Snippets

### Quick EDA
```python
df.describe()  # Statistical summary
df.info()  # Data types and non-null counts
df.dtypes  # All column types
df.isna().sum()  # Missing values per column
df.duplicated().sum()  # Number of duplicate rows
df.shape  # Rows × columns
```

### Data Validation
```python
# Check for nulls
assert df.isna().sum().sum() == 0, "Found NaN values!"

# Check for duplicates
assert not df.duplicated().any(), "Found duplicates!"

# Check ranges
assert (df['fare'] >= 0).all(), "Negative fares found!"
assert (df['distance'] >= 0).all(), "Negative distances found!"

# Check types
assert df['pickup_time'].dtype == 'datetime64[ns]', "Wrong dtype!"
```

### Summary by Group
```python
# Group and aggregate
df.groupby('zone').agg({
    'fare': ['mean', 'median', 'std', 'count'],
    'duration': 'mean'
})

# Count by group
df['payment_type'].value_counts()  # With counts
df['payment_type'].value_counts(normalize=True)  # With percentages
```

### Data Cleaning
```python
# Remove duplicates
df = df.drop_duplicates()

# Remove rows with NaN
df = df.dropna()

# Replace outliers
Q1 = df['fare'].quantile(0.25)
Q3 = df['fare'].quantile(0.75)
IQR = Q3 - Q1
mask = (df['fare'] >= Q1 - 1.5*IQR) & (df['fare'] <= Q3 + 1.5*IQR)
df = df[mask]

# Rename columns
df.rename(columns={'old_name': 'new_name'}, inplace=True)
```

---

## When to Use Each Solution

| Problem | Quick Fix | Robust Solution | Advanced Solution |
|---------|-----------|-----------------|------------------|
| Low memory | Select columns | Aggregate dimensions | Use Dask/Polars |
| Slow merge | Create index | Reduce rows first | Use DuckDB/Spark |
| Type error | astype('float') | Clean before conversion | Custom parser |
| Timezone issue | .tz_localize(None) | Standardize to UTC | Database-level handling |
| Missing data | dropna() | Strategy per column | Model-based imputation |
| Duplicates | drop_duplicates() | Composite key check | Deduplication rules |

---

## Further Reading

For each operation in this notebook, consult:

1. **Data Loading:** Pandas IO tools documentation
2. **Timezone handling:** pytz documentation
3. **Merging:** Pandas merge/join documentation
4. **Type conversion:** Pandas astype and pd.to_numeric
5. **Validation:** Pandas assert_frame_equal and custom validators
6. **Scaling:** scikit-learn preprocessing module
7. **Modeling:** scikit-learn, XGBoost, LightGBM documentation

---

## Final Checklist Before Modeling

- [ ] Data is loaded and verified
- [ ] All data types are correct
- [ ] Missing values handled
- [ ] Duplicates removed
- [ ] Outliers handled (decision documented)
- [ ] Datasets merged correctly
- [ ] Target variable identified and analyzed
- [ ] Features engineered and selected
- [ ] Categorical variables encoded
- [ ] Scaling/normalization applied
- [ ] Train/test split respects temporal order
- [ ] No leakage columns remain
- [ ] Summary statistics reviewed
- [ ] Ready to model!

---

**Remember:** Data preparation is 80% of the work, but it's often the most important 80%.  
Get this right, and modeling becomes straightforward. ✓
