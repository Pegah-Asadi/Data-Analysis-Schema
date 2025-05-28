# 🧪 Hypothesis Testing – Sample Code

Compare two independent groups (A vs B). Includes normality + equal-variance checks and effect size.

> **Requires:** `pandas`, `scipy`  (install via `pip install scipy`)

---

```python
"""
Two-group comparison workflow
––––––––––––––––––––––––––––––––––––––––––––––––
1. Assumption checks      · normality (Shapiro) + variance (Levene)
2. Choose proper test     · t-test  ↔ Mann–Whitney U
3. Report effect size     · Cohen’s d
"""

import pandas as pd
from scipy import stats
import numpy as np

# -------------------------------------------------------------
# 0 · LOAD DATA
# -------------------------------------------------------------
df = pd.read_csv("your_data.csv")                 # <-- edit this line with your file

# Define column names: one for grouping (e.g. "variant") and one for the numeric metric (e.g. "conversion")
group_col  = "<group_column>"                     # e.g. "variant" with values like "A" and "B"
metric_col = "<metric_column>"                    # e.g. "conversion"

# Filter the DataFrame for each group, and drop missing values from the metric
a = df[df[group_col] == "A"][metric_col].dropna()
b = df[df[group_col] == "B"][metric_col].dropna()

# Ensure both groups have data — fail early if not
assert len(a) and len(b), "Both groups must have data!"

# -------------------------------------------------------------
# 1 · ASSUMPTION CHECKS
# -------------------------------------------------------------
# Test for normality using the Shapiro–Wilk test
# For large samples (>5000), take a sample to avoid performance issues
p_norm_a = stats.shapiro(a.sample(5000, replace=False) if len(a) > 5000 else a).pvalue
p_norm_b = stats.shapiro(b.sample(5000, replace=False) if len(b) > 5000 else b).pvalue

# Both groups are considered normal if both p-values > 0.05
normal   = (p_norm_a > 0.05) and (p_norm_b > 0.05)

# Test for equal variances using Levene's test
p_levene = stats.levene(a, b).pvalue
equal_var = p_levene > 0.05

# Print results of assumption checks
print(f"Normal? {normal}  |  Equal variances? {equal_var}")

# -------------------------------------------------------------
# 2 · SELECT & RUN TEST
# -------------------------------------------------------------
# If both groups are normally distributed, run an independent t-test
# Otherwise, use the non-parametric Mann–Whitney U test
if normal:
    t_stat, p_val = stats.ttest_ind(a, b, equal_var=equal_var)
    test_name = "Independent t-test"
else:
    t_stat, p_val = stats.mannwhitneyu(a, b, alternative="two-sided")
    test_name = "Mann–Whitney U"

# -------------------------------------------------------------
# 3 · EFFECT SIZE (Cohen’s d) – always useful
# -------------------------------------------------------------
# Define a function to calculate Cohen's d (standardized effect size)
def cohens_d(x, y):
    nx, ny      = len(x), len(y)
    pooled_std  = np.sqrt(((nx - 1)*x.var(ddof=1) + (ny - 1)*y.var(ddof=1)) / (nx + ny - 2))
    return (x.mean() - y.mean()) / pooled_std

# Compute effect size
d = cohens_d(a, b)

# -------------------------------------------------------------
# 4 · REPORT
# -------------------------------------------------------------
# Print test results and interpretation
print(f"\n📊  {test_name}")
print(f"   test statistic = {t_stat:.3f}")
print(f"   p-value        = {p_val:.4f}")
print(f"   Cohen’s d      = {d:.3f}")

# Interpret the p-value at α = 0.05 (standard threshold for significance)
alpha = 0.05
if p_val < alpha:
    print("✅ Statistically significant difference.")
else:
    print("🟡 No significant difference detected.")
```

🗒️ Adapting this template:  
- Categorical outcomes → chi2_contingency on a 2×2 table.  
- Paired samples (before/after) → stats.ttest_rel or Wilcoxon.  
- Multiple groups → ANOVA + post-hoc (Tukey HSD).  
- Large, skewed metrics (e.g. revenue) often break normality—go straight to Mann-Whitney or bootstrap. 
