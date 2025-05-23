# 🧪 Hypothesis Testing – Sample Code

Use this script to statistically test differences between two groups, such as different user segments or product versions.

---

## 🎯 Example Use Case

"Do users who receive an onboarding email convert more in the first 7 days compared to those who don’t?"

---

## 🔍 T-Test for Comparing Two Groups

```python
import pandas as pd
from scipy.stats import ttest_ind

# Load your data
df = pd.read_csv('your_data.csv')

# Replace with your actual group and metric columns
group_a = df[df['group_column'] == 'A']['metric_column']
group_b = df[df['group_column'] == 'B']['metric_column']

# Run an independent t-test
t_stat, p_value = ttest_ind(group_a, group_b, nan_policy='omit')

# Print results
print(f"T-statistic: {t_stat:.2f}")
print(f"P-value: {p_value:.4f}")

# Interpretation
if p_value < 0.05:
    print("Result is statistically significant. Difference is likely real.")
else:
    print("No significant difference found.")
```


📝 Notes
- group_column should be a binary label (e.g., 'A' vs 'B', 'with_email' vs 'no_email').  
- metric_column is the numeric outcome you're comparing (e.g., engagement, spend, retention).  
- nan_policy='omit' ensures missing values don't affect results.  
- This test assumes equal variances. For unequal variances, use ttest_ind(..., equal_var=False).  
