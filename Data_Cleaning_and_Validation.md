# 🧹 General Python Scripts for Data Cleaning & Validation

These reusable scripts help you clean, validate, and sanity-check your data before moving on to analysis or modeling.

---

## 🧼 1. Data Cleaning Script

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# Load your dataset
df = pd.read_csv('your_data.csv')  # Replace with your file or source

### 1️⃣ CHECK FOR DUPLICATES ###

# Find duplicate rows
duplicates = df[df.duplicated()]
print(f"Found {len(duplicates)} duplicate rows.")

# Remove duplicates
df = df.drop_duplicates()

### 2️⃣ CHECK FOR MISSING VALUES ###

# Summary of missing values per column
missing_summary = df.isnull().sum()
print("Missing values per column:\n", missing_summary)

# Fill missing values based on context:
df['numeric_column'] = df['numeric_column'].fillna(df['numeric_column'].mean())      # Numeric: mean
df['numeric_column'] = df['numeric_column'].fillna(df['numeric_column'].median())    # Numeric: median
df['category_column'] = df['category_column'].fillna(df['category_column'].mode()[0])  # Categorical: mode
df['category_column'] = df['category_column'].fillna('Unknown')                      # Categorical: constant

# Drop rows with too many missing fields (e.g., >50%)
df = df.dropna(thresh=int(df.shape[1] * 0.5))

### 3️⃣ DETECT OUTLIERS (IQR METHOD) ###

def detect_outliers_iqr(column):
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    lower = Q1 - 1.5 * IQR
    upper = Q3 + 1.5 * IQR
    return df[(df[column] < lower) | (df[column] > upper)]

outliers = detect_outliers_iqr('numeric_column')  # Replace with your column
print(f"Outliers found:\n{outliers}")

# Visualize with a boxplot
sns.boxplot(x=df['numeric_column'])
plt.show()

### 4️⃣ SPOT DATA ANOMALIES ###

# Category distribution
print("Category value counts:\n", df['category_column'].value_counts())

# Unexpected or invalid values
print("Invalid ages found:\n", df[df['age'] <= 0])

# Correlation matrix
corr_matrix = df.corr(numeric_only=True)
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm')
plt.show()

### 5️⃣ FINAL SUMMARY ###

print("Final dataset overview:")
print(df.info())
print(df.describe())

```

## ✅ 2. Data Validation Script (with assert statements)

This script ensures your cleaned data meets expectations by validating:
- Column types  
- Value ranges  
- Uniqueness  
- Completeness  
- Business rules

```python
import pandas as pd

# Load your dataset
df = pd.read_csv('your_data.csv')  # Replace with your file or source

### 1️⃣ ASSERT NO MISSING VALUES IN CRITICAL COLUMNS ###
assert df['user_id'].isnull().sum() == 0, "Missing user_id values!"
assert df['signup_date'].isnull().sum() == 0, "Missing signup dates!"

### 2️⃣ ASSERT DATA TYPES ARE CORRECT ###
assert pd.api.types.is_numeric_dtype(df['age']), "Age should be numeric!"
assert pd.api.types.is_datetime64_any_dtype(pd.to_datetime(df['signup_date'], errors='coerce')), "Signup date should be datetime!"

### 3️⃣ ASSERT VALUE RANGES ###
assert df['age'].between(0, 120).all(), "Invalid ages (must be 0–120)!"
assert df['conversion_score'].between(0, 1).all(), "Conversion score should be between 0 and 1!"

### 4️⃣ ASSERT UNIQUENESS CONSTRAINTS ###
assert df['transaction_id'].is_unique, "Duplicate transaction IDs!"
assert not df['user_id'].is_unique, "Users can have multiple records—unexpected uniqueness!"

### 5️⃣ ASSERT VALID CATEGORICAL VALUES ###
expected_statuses = {'active', 'inactive', 'pending'}
actual_statuses = set(df['status'].dropna().unique())
assert actual_statuses.issubset(expected_statuses), f"Unexpected status values: {actual_statuses - expected_statuses}"

### 6️⃣ ASSERT NO DUPLICATE ROWS ###
assert not df.duplicated().any(), "Duplicate rows detected!"

### 7️⃣ ASSERT POSITIVE VALUES ###
assert (df['revenue'] > 0).all(), "Revenue column has non-positive values!"

### 8️⃣ OPTIONAL CUSTOM ASSERTIONS ###
# Example: All users must have at least one interaction
interactions_per_user = df.groupby('user_id').size()
assert (interactions_per_user >= 1).all(), "Some users have no interactions!"

### ✅ FINAL STATUS ###
print("✅ All validations passed. Data looks good!")
```

💡 Notes for Customization  
- Replace column names like 'user_id', 'age', 'revenue', 'signup_date', etc., with those relevant to your dataset.  
- Add business-specific rules as needed.  
- Use these scripts as checkpoints in your ETL or analysis pipeline.

