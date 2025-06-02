# ✳️ A/B Test – End-to-End Template
Deterministic assignment → sample-size calc → real-time monitoring → final stats.

> **Requires:** `pandas`, `numpy`, `scipy`, `statsmodels`

```python
"""
0 · CONFIG - Define experiment parameters ---------------------------
Edit these values based on your business or experiment needs.
These can later be wired into DBT, Airflow macros, or config files.
"""
BASELINE_RATE      = 0.10     # 📌 Current conversion rate (10%)
MIN_DETECTABLE_UPL = 0.012    # 🧩 Smallest meaningful uplift (1.2pp)
ALPHA              = 0.05     # 🧪 Significance level (false positive threshold)
POWER              = 0.80     # 🎯 Statistical power (chance of detecting a true uplift)
SPLIT              = 0.5      # ⚖️  50-50 traffic split between control and test

# -------------------------------------------------------------------
# 1 · SAMPLE-SIZE (Calculate users needed per group) -----------------
# -------------------------------------------------------------------
from statsmodels.stats.power import NormalIndPower
from statsmodels.stats.proportion import proportion_effectsize

# Calculate standardized effect size using baseline and uplift
h = proportion_effectsize(BASELINE_RATE, BASELINE_RATE + MIN_DETECTABLE_UPL)

# Solve for required sample size per group (assumes 1:1 ratio)
n_each = NormalIndPower().solve_power(h, power=POWER, alpha=ALPHA, ratio=1)

print(f"📏  Need ≈ {int(n_each):,} users per group (1:1 split)")

# ℹ️ Use TTestIndPower for continuous metrics instead of proportions.

# -------------------------------------------------------------------
# 2 · RANDOMISATION HASH (BigQuery) ----------------------------------
# -------------------------------------------------------------------
"""
SELECT
    user_id,
    MOD(FARM_FINGERPRINT(CAST(user_id AS STRING)), 100) AS bucket_id,
    CASE WHEN bucket_id < 100 * {SPLIT}
         THEN 'control' ELSE 'test' END                AS group
FROM `project.dataset.users`
"""
/* ✅ Deterministic fingerprinting ensures stable group assignment over time. */
/* 🧩 {SPLIT} lets you dynamically change the group allocation. */

# -------------------------------------------------------------------
# 3 · LIVE DASHBOARD (pseudo-query) ---------------------------------
# -------------------------------------------------------------------
"""
WITH latest AS (
  SELECT group,
         COUNT(*)                          AS users,
         SUM(conversion)                   AS converts
  FROM   `project.dataset.ab_events`
  WHERE  event_date BETWEEN @start AND CURRENT_DATE()
  GROUP  BY group
)
SELECT group,
       converts / users AS conversion_rate
FROM   latest;
"""
/* ⚡ Useful for Looker or dashboarding tools during the live experiment. */

# -------------------------------------------------------------------
# 4 · POST-EXPERIMENT ANALYSIS --------------------------------------
# -------------------------------------------------------------------
import pandas as pd
from scipy import stats
from statsmodels.stats.api import DescrStatsW

# Load experiment results from file
df = pd.read_csv("ab_results.csv")

# Split into control and test groups
ctrl = df[df.group == "control"]["metric"]
test = df[df.group == "test"]["metric"]

# 🎯 Welch's t-test (safe for unequal variances)
t, p = stats.ttest_ind(ctrl, test, equal_var=False)
print(f"🧪 Welch t-test | t = {t:.3f}  |  p = {p:.4f}")

# 🧾 Calculate 95% confidence interval for the difference in means
dsc_c = DescrStatsW(ctrl)
dsc_t = DescrStatsW(test)

# Mean difference between groups
cm = dsc_t.mean - dsc_c.mean

# Standard error of the difference
se = (dsc_t.var / len(test) + dsc_c.var / len(ctrl)) ** 0.5

# Confidence interval using pooled degrees of freedom
ci_lo, ci_hi = stats.t.interval(0.95, df=len(ctrl) + len(test) - 2, loc=cm, scale=se)

print(f"📉 Mean diff = {cm:.4f} (95% CI: {ci_lo:.4f}, {ci_hi:.4f})")

# ✅ Welch’s t-test is robust to unequal variances.
# 🧠 Confidence intervals show practical significance, not just p-values.

# -------------------------------------------------------------------
# 5 · REPORT ---------------------------------------------------------
# Summary statistics for each group: mean, std, count, etc.
print(df.groupby("group")["metric"].describe())

"""
```
