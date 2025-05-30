# 🧪 A/B Test Sample Size Calculator (Binary Metric)

```python
"""
Sample size calculator for binary outcomes (e.g., conversion rate)
––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
• Estimates how many users you need **per group** to detect a meaningful uplift.
• Assumes equal split (1:1) and uses normal approximation.
• For continuous metrics, use `TTestIndPower` instead.
"""

# 📦 Import required functions from statsmodels
from statsmodels.stats.power import NormalIndPower
from statsmodels.stats.proportion import proportion_effectsize

# -------------------------------------------------------------
# 🎯 1. Define your experiment parameters
# -------------------------------------------------------------

baseline_rate   = 0.10    # Current conversion rate (10%)
min_detectable  = 0.012   # Minimum uplift you care about (+1.2pp → 11.2%)
alpha           = 0.05    # Significance level (5%) → probability of false positive
power           = 0.80    # Statistical power (80%) → probability of detecting a real effect

# -------------------------------------------------------------
# 📐 2. Calculate effect size based on proportions
# -------------------------------------------------------------
# Cohen's h: standardized effect size for proportion differences
effect_size = proportion_effectsize(
    baseline_rate,
    baseline_rate + min_detectable
)

# -------------------------------------------------------------
# 🧮 3. Solve for required sample size per group
# -------------------------------------------------------------
# Assumes:
# - Independent groups
# - Equal group sizes (1:1 ratio)
# - Normal approximation holds for proportions
n_per_group = NormalIndPower().solve_power(
    effect_size,
    power=power,
    alpha=alpha,
    ratio=1
)

# -------------------------------------------------------------
# 📊 4. Output the result
# -------------------------------------------------------------
# Display the required number of users in each group
print(f"📏  Need ≈ {int(n_per_group):,} users per group (1:1 split)")
```
