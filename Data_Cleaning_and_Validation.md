# 🧹 General Python Scripts for Data Cleaning & Validation

These reusable scripts help you clean, validate, and sanity-check your data before moving on to analysis or modeling.

> **Dependencies:** `pandas`, `numpy`, `matplotlib`  
> (`seaborn` optional—commented out)

---

## 🧼 1. Data Cleaning Script

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
# import seaborn as sns  # optional

# ---------------------------------------------------------
# LOAD DATA (edit path/source)
# ---------------------------------------------------------
df = pd.read_csv("your_data.csv")          # 👈 replace
print(f"🔹 Loaded {len(df):,} rows, {df.shape[1]} columns")

# ---------------------------------------------------------
# 1️⃣  Duplicates
# ---------------------------------------------------------
# Count duplicate rows
dupes = df.duplicated().sum()
print(f"🔸 Duplicates found: {dupes:,}")

# Display and drop duplicate rows if any
if dupes:
    print(df[df.duplicated()].head())  # Show a few for inspection
    df.drop_duplicates(inplace=True)
    print("✅ Duplicates removed.")

# ---------------------------------------------------------
# 2️⃣  Missing values
# ---------------------------------------------------------
# Check missing values per column
missing = df.isna().sum().sort_values(ascending=False)
print("🔸 Missing per column:\n", missing.head())

# Separate numeric and categorical columns
num_cols  = df.select_dtypes(include="number").columns
cat_cols  = df.select_dtypes(exclude="number").columns

# Fill numeric columns with median (robust to outliers)
df[num_cols] = df[num_cols].fillna(df[num_cols].median())     # (Optional: use mean instead by changing `.median()` to `.mean()`)

# Fill categorical columns with a constant (e.g., "Unknown")
df[cat_cols] = df[cat_cols].fillna("Unknown")     # (Optional: use mode per column instead if more meaningful)

# Drop rows where more than 50% of fields are missing
row_thresh = int(df.shape[1] * 0.5)
df.dropna(thresh=row_thresh, inplace=True)

# ---------------------------------------------------------
# 3️⃣  Outlier detection (IQR) for any numeric column
# ---------------------------------------------------------
def iqr_outlier_mask(series, k=1.5):
    q1, q3 = series.quantile([0.25, 0.75])
    iqr = q3 - q1
    low, high = q1 - k * iqr, q3 + k * iqr
    return (series < low) | (series > high)

outliers = df[num_cols].apply(iqr_outlier_mask).any(axis=1)     # a row is flagged if any of its numeric values are outliers
print(f"🔸 Outlier rows: {outliers.sum():,}")

# Optional viz with a boxplot
# plt.boxplot(df["numeric_column"]); plt.show()

# ---------------------------------------------------------
# 4️⃣  Quick anomaly checks
# ---------------------------------------------------------
print("🔸 Category counts:\n", df["category_column"].value_counts().head())
print("🔸 Invalid ages:\n", df.loc[df["age"] <= 0, ["user_id", "age"]])     # replace 'age' with your numeric column you want to inspect

# ---------------------------------------------------------
# 5️⃣  Final summary
# ---------------------------------------------------------
print(df.info(memory_usage="deep"))
print(df.describe(include="all").T)

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

df = pd.read_csv("your_data.csv")  # 👈 replace

# ---------------------------------------------------------
# 1️⃣  Critical null checks
# ---------------------------------------------------------
for col in ["user_id", "signup_date"]:
    assert df[col].notna().all(), f"❌  Nulls in {col}"

# ---------------------------------------------------------
# 2️⃣  Type enforcement
# ---------------------------------------------------------
df["signup_date"] = pd.to_datetime(df["signup_date"], errors="raise")
assert pd.api.types.is_numeric_dtype(df["age"]), "❌  age not numeric"

# ---------------------------------------------------------
# 3️⃣  Value ranges
# ---------------------------------------------------------
assert df["age"].between(0, 120).all(),   "❌  age outside 0-120"
assert df["score"].between(0, 1).all(),   "❌  score outside 0-1"

# ---------------------------------------------------------
# 4️⃣  Uniqueness rules
# ---------------------------------------------------------
assert df["transaction_id"].is_unique, "❌  duplicate transaction_id"
# Users may repeat ⇒ non-unique allowed
assert not df["user_id"].is_unique, "❌  Users can have multiple records"

# ---------------------------------------------------------
# 5️⃣  Categorical whitelist
# ---------------------------------------------------------
# Define the expected valid values for the 'status' column
expected = {"active", "inactive", "pending"}

# Get the unique non-null values in the 'status' column,
# then subtract the expected values to find any unexpected entries
bad = set(df["status"].dropna()) - expected

# If any unexpected values are found, raise an error and print them
assert not bad, f"❌  unexpected status values: {bad}"

# ---------------------------------------------------------
# 6️⃣  NO DUPLICATE ROWS
# ---------------------------------------------------------
assert not df.duplicated().any(), "Duplicate rows detected!"

# ---------------------------------------------------------
# 7️⃣  POSITIVE VALUES
# ---------------------------------------------------------
assert (df['revenue'] > 0).all(), "Revenue column has non-positive values!"

# ---------------------------------------------------------
# 8️⃣  OPTIONAL CUSTOM ASSERTIONS
# ---------------------------------------------------------
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

