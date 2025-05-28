# 📊 Exploratory Data Analysis (EDA) – Sample Code

This file includes generalized Python snippets to help you explore data distributions, spot outliers, and identify variable relationships before moving into modeling or testing.

> **Requires:** `pandas`, `matplotlib`, `seaborn` (install via `pip install seaborn`)

---

```python
"""
EDA quick-start template
––––––––––––––––––––––––––––––––––––––––––––––––
• Works on any tabular CSV / parquet / SQL extract.
• Replace all placeholder column names in <angle brackets>.
• Keep figures lightweight—use a Notebook for deep dives.
"""

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_style("whitegrid")  # ✨ nice default look

# ------------------------------------------------------------------
# 0 · LOAD DATA -----------------------------------------------------
# ------------------------------------------------------------------
df = pd.read_csv("your_data.csv")          # <-- edit path / loader
print(f"Rows: {len(df):,} | Columns: {df.shape[1]}")

# ------------------------------------------------------------------
# 1 · DISTRIBUTIONS -------------------------------------------------
# ------------------------------------------------------------------
def plot_distribution(column: str, bins: int = 30) -> None:
    """Histogram + KDE for a numeric column."""
    sns.histplot(df[column], bins=bins, kde=True)
    plt.title(f"Distribution – {column}")
    plt.xlabel(column)
    plt.ylabel("Count")
    plt.show()

plot_distribution("<numeric_column>")      # e.g. "revenue"

# ------------------------------------------------------------------
# 2 · OUTLIER CHECK (IQR) ------------------------------------------
# ------------------------------------------------------------------
def iqr_outliers(series: pd.Series, k: float = 1.5) -> pd.Series:
    """Return Boolean mask of outliers via IQR rule."""
    q1, q3 = series.quantile([0.25, 0.75])
    iqr = q3 - q1
    low, high = q1 - k * iqr, q3 + k * iqr
    return (series < low) | (series > high)

num_col = "<numeric_column>"
outliers_mask = iqr_outliers(df[num_col])
print(f"Outliers in {num_col}: {outliers_mask.sum():,}")

# Optional visual
sns.boxplot(x=df[num_col])
plt.title(f"Outliers – {num_col}")
plt.show()

# ------------------------------------------------------------------
# 3 · CORRELATION MATRIX -------------------------------------------
# ------------------------------------------------------------------
# Check how numeric variables are related using a correlation matrix.
corr = df.corr(numeric_only=True)

# Plot heatmap
plt.figure(figsize=(10, 8))
sns.heatmap(corr, annot=True, cmap="coolwarm", fmt=".2f")
plt.title("Correlation (numeric features)")
plt.show()

# ------------------------------------------------------------------
# 4 · PAIRWISE SCATTER (auto) --------------------------------------
# ------------------------------------------------------------------
# Select a small set of columns to visualize relationships between them
# '<x>' and '<y>' should be numeric columns, '<hue_optional>' is an optional categorical column for coloring
pair_cols = ["<x>", "<y>", "<hue_optional>"]  # small subset keeps plot readable

# Create a pairplot to show pairwise relationships between selected columns
# 'hue' adds color to the plots based on the values in the categorical column
sns.pairplot(df[pair_cols], hue="<hue_optional>", corner=True)

plt.show()

# ------------------------------------------------------------------
# 5 · QUICK SUMMARY -------------------------------------------------
# ------------------------------------------------------------------
print(df.describe(include="all").T)        # categorical + numeric
print(df.isna().mean().sort_values(ascending=False).head())  # null %
```

📝 Notes  
- Swap <numeric_column> / <x> / <y> for real columns.  
- These plots help you catch data issues and generate hypotheses for deeper testing.  
