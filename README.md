# **Data Analysis Schema**

> A practical, repeatable framework for structured, confident analysis.

## 🚀 Why This Exists
  
While every project has its own goal, strong analyses tend to follow the same core structure. This schema breaks it down into **10 clear steps**, from business framing to storytelling and documentation.
Some steps are strategic, others are technical. Some include code, others are just mindset.

Whether you're new to the role or just looking to sharpen your workflow, I hope this guide brings structure and clarity to your process.

👇 Let’s dive in.

---

## **1 | 🧠 Frame the Business Question**

Before touching data, clarify:

🎯 What’s the **business goal**? (e.g. increase retention, reduce costs, partner ROI)  
📊 How will success be measured? (e.g. MAUs, NPS, churn rate)  
⚠️ What are the **constraints**? (e.g. privacy laws, sample size, timelines)

🛡️ Especially in regulated domains:

→ Use anonymized or pseudonymized data only  
→ Keep audit trails clean  
→ Only use the minimum necessary fields

[➡️ See pseudonymization code (SQL + Python)](./Pseudonymization.md).

---

## **2 | 🤝 Align with Stakeholders**

Before touching data, I meet with each stakeholder group to understand their goals, constraints, and definitions of success.  
Here’s how I align with each:

| Role             | What they care about                    | What I deliver                     |
| ---------------- | --------------------------------------- | ---------------------------------- |
| Product Manager  | User experience, engagement, retention  | 1-pager with problem & KPIs        |
| Legal/Compliance | Data use & governance                   | Data-flow diagram, pseudonym steps |
| Engineering/Data | Feasibility, pipelines, event tracking	 | Clear data needs + tracking plan   |
| Executives       | Business impact (ROI, revenue, savings) | Clear outcomes in a slide          |

---

## **3 | 🧾 Data Audit & Governance**

Before analysis begins, you need a clear picture of **what data you have**, **how clean it is**, and **whether you can use it**.  
This step ensures your foundation is solid, technically and ethically.

### 📦 Inventory Your Data

List all relevant data sources and tables connected to your business question.
Examples include:

- CRM exports (customer info, interactions)  
- App logs (clickstream, events)  
- Financial or billing systems  
- Third-party data (e.g., marketing platforms, vendor feeds)

📄 Build a simple inventory table:

| Table Name       | Description                   | Key Fields                |
| ---------------- | ----------------------------- | ------------------------- |
| customers        | User profiles, demographics   | customer\_id, age, region |
| transactions     | Purchase history              | txn\_id, amount, date     |
| events           | App usage                     | user\_id, event\_name     |
| support\_tickets | Customer issues & resolutions | ticket\_id, issue\_type   |

&nbsp;

### 🧹 Check Data Quality

Clean data saves you from rework and false conclusions.

Look for:  
- Missing values (null % per column)  
- Duplicates  
- Invalid ranges (e.g., negative ages, future timestamps)  
- Mismatched joins or keys

✅ Tip: Automate basic checks if possible  
(Saved me 20% time by catching issues early.)<br><br><br>

### 🔐 Review Access & Privacy

Make sure your data access aligns with legal, contractual, or internal governance policies.

📄 Privacy Checklist:  
- Use pseudonymized or anonymized IDs when sharing data  
- Limit fields to the minimum necessary  
- Keep processing logs or audit trails if required  
- Understand any compliance frameworks (e.g., GDPR, SOC2, HIPAA if applicable)
- Access is role-based and logged<br><br><br>

### 🗂️ Deliverables for This Step  

📋 Data Inventory Table  
→ Clear list of all relevant tables, key fields, and purpose

📊 Data Quality Report  
→ Null rates, duplicate counts, basic sanity checks

🔐 Privacy & Governance Plan  
→ Fields masked or pseudonymized, access rationale, logging strategy

---

## **4 | 🧰 Extraction & Preparation**

This is where raw data turns into something meaningful.  
🎯 Goal: extract the right data, clean it, and shape it into an analysis-ready format.

### 🏗️ Step-by-Step Workflow

| Step          | Common Tools                              | Output                              |
| ------------- | ----------------------------------------- | ----------------------------------- |
| **Extract**   | SQL (BigQuery, Snowflake, Redshift, etc.) | Source tables filtered & joined     |
| **Transform** | Python (pandas), dbt, R, Spark            | Cleaned, structured dataset         |
| **Validate**  | Python (asserts, checks), SQL             | QC/validation report, sanity checks |

&nbsp;

### 🔄 Key Principles

#### 1. Write Modular SQL  
- Use CTEs (Common Table Expressions) to break down logic  
- Keep queries readable and reusable
- Example:
```SQL
WITH filtered_events AS (
  SELECT *
  FROM events
  WHERE event_date >= '2024-01-01'
)
SELECT user_id, COUNT(*) AS event_count
FROM filtered_events
GROUP BY user_id;
```

&nbsp;

#### 2. Transform in Code  
Use pandas or dbt to:
- Handle missing values  
- Create new features  
- Normalize and filter data  

- Example:
```Python
df['signup_date'] = pd.to_datetime(df['signup_date'])
df['days_since_signup'] = (today - df['signup_date']).dt.days
```

&nbsp;

#### 3. Validate Your Output  
- Add assertions to catch edge cases and silent errors early:  
```Pyhton
assert df['user_id'].isnull().sum() == 0, "Missing user IDs!"
assert df['age'].between(0, 120).all(), "Invalid age values!"
```
- Create a QC report: null %s, duplicate counts, type mismatches, etc.<br><br><br>

### 📦 Deliverables for This Step  
- Clean dataset — ready for EDA or modeling  
- Modular extraction SQL — readable, reusable, and easy to debug  
- Validation report or notebook — with assertions, checks, and summary stats

&nbsp;

[➡️ Full Python cleaning & validation snippets](./Data_Cleaning_and_Validation.md)

These cover:  
🔁 Removing duplicates  
🕳️ Handling missing values  
🚨 Detecting outliers  
✅ Validating schema, value ranges, and data integrity using assert statements  

---

## **5 | 🔍 Exploratory Data Analysis (EDA) & Hypothesis Generation**

This is where ideas form and problems surface.

🎯 Goals of This Step:  
- Understand distributions, trends, and gaps  
- Spot correlations and patterns  
- Form testable hypotheses  

### ⚙️ EDA Workflow Summary

| Subtask                  | Tool(s)                  | Why It Matters                        | Example Action                      |
| ------------------------ | ------------------------ | ------------------------------------- | ----------------------------------- |
| Distributions & Outliers | Python (Seaborn, pandas) | Identify skewed data & extreme values | `sns.histplot()`, `sns.boxplot()`   |
| Time-based Trends        | Looker, Tableau          | Spot seasonal effects or churn spikes | Line charts by day/week/month       |
| Correlation Matrix       | Python (Seaborn)         | Understand variable relationships     | `df.corr()`, `sns.heatmap()`        |
| Variable Pair Trends     | Tableau or Python        | Visually inspect pairwise trends      | Scatter plots with trendlines       |
| Group Comparisons        | SciPy, statsmodels       | Check if group differences are real   | `ttest_ind()`, `chi2_contingency()` |

&nbsp;

### ** 1. 🧭 Data Profiling: “What does the data look like?”**

Start by exploring:  
- Key metrics (e.g. # transactions, avg session duration, revenue per user)  
- Value ranges, missingness, outliers

📈 Sample: Time-based Trends (Looker/Tableau)  
- Plot engagement, revenue, or sign-ups over time  
- Slice by day-of-week, campaign, or product  
- Example: Identify drop-offs after onboarding, spikes during promotions<br><br><br>

### **2. 🔗 Correlations & Patterns: “What moves together?”**

Look for relationships between variables.

Use this to:  
- Understand leading indicators of conversion, churn, or retention  
- Avoid multicollinearity in models

📂 [See sample code for EDA →](./EDA_Sample_Code.md) <br><br><br>

### **3. 🧪 Statistical Testing: “Is this difference real?”**

Use tests like t-tests, chi-square, or Mann-Whitney U to validate patterns in data.

Use when comparing:  
- Conversion rates across landing pages  
- Engagement before/after a feature rollout  
- Retention between cohorts

📂 [See sample code for hypothesis testing →](./Hypothesis_Testing_Sample_Code.md)<br><br><br>

### **4. 💡 Generate Hypotheses**

Good hypotheses are:
- Testable (you can collect or analyze the needed data)  
- Specific (clear variables, direction, expected impact)  
- Relevant (linked to a business goal)  

**💭 Generic Example Hypotheses:**
1. "Users who see onboarding complete 25% more actions in week 1."  
2. "Paid users churn less than organic users."  
3. "Feature X increases checkout rate by 10%."<br><br><br>

### 🧾 Deliverables for This Step  
- Visualizations (distributions, trends, correlation plots)  
- Group comparison test results (e.g. t-test output)  
- A short list of hypotheses to test next  
- Optional: a 1-pager summarizing key EDA findings for your stakeholders

---

## **6 | 🧪 Experiment or Modeling Design**

This step is about how you validate your hypotheses. Based on the question, you’ll either:  
- Test a causal relationship with an experiment (e.g., A/B test)  
- Predict an outcome with a machine learning model  

### 🎯 When to Use What?

| Question Type                            | Use This Method       |
| ---------------------------------------- | --------------------- |
| What is the impact of doing X?           | A/B Test (Experiment) |
| Can we predict who will do Y?            | Predictive Modeling   |
| Should we show version A or B of a page? | A/B Test              |
| Who are the high-risk users for Z?       | Modeling              |

### **A. ✳️ A/B Testing (Experiment Design)**

Use this when you want to test cause and effect.

**Common Use Cases:**  
- Does a new feature increase retention?  
- Do email reminders reduce drop-off?  
- Which landing page converts better?  

&nbsp;

**Key Steps for A/B Testing**   
1. Define Groups
   - Control Group: No change
   - Test Group: Receives the change (e.g., new feature, message)
2. Choose Metrics
   - Primary Metric: What you're testing (e.g., conversion rate, engagement)
   - Guardrails: What should not be negatively impacted (e.g., churn, cost)
3. Calculate Sample Size
Use power analysis to ensure you can detect a meaningful difference.

📂 Example code snippet for calculating sample size:
```python
from statsmodels.stats.power import tt_ind_solve_power

# Parameters
effect_size = 0.2  # Small effect size (standardized difference)
alpha = 0.05       # 5% significance level
power = 0.8        # 80% chance of detecting an effect

# Calculate
sample_size_per_group = tt_ind_solve_power(effect_size=effect_size, alpha=alpha, power=power)
print(f"Required sample size per group: {int(sample_size_per_group)} users")
```

4. Randomization Plan
Ensure fair, unbiased assignment using hashing or row numbers.

📂 Example code snippet for hashing:
```python
MOD(FARM_FINGERPRINT(CAST(user_id AS STRING)), 100)
# FARM_FINGERPRINT(...): creates a hash of the user ID. This generates a deterministic but seemingly random number — perfect for consistent random assignment.
# MOD(..., 100): gets the last two digits (like a bucket from 0 to 99). So we have 100 buckets total.
```

📂 Example code snippet in case you want an exact count:
```python
ROW_NUMBER() OVER (ORDER BY RAND()) AS row_num
```

5. Run & Monitor
   - Duration: 2–4 weeks (depending on volume)
   - Track metrics in real time via dashboard
6. Analyze Results
Use statistical tests (like t-tests) to validate outcomes.

📂 Example code snippet:
```python
import scipy.stats as stats

# Assume you have two arrays: control_engagement, test_engagement
t_stat, p_value = stats.ttest_ind(control_engagement, test_engagement)

print(f"T-statistic: {t_stat:.3f}, p-value: {p_value:.3f}")
```

### **B. 🤖 Predictive Modeling**

Use this when you want to forecast outcomes based on patterns.

**Common Use Cases:**  
- Predict customer churn  
- Identify high-value leads  
- Forecast sales or engagement  

&nbsp;

**Key Steps for Modeling**  
1. Define the Target  
   - Classification (e.g., churn: yes/no)
   - Regression (e.g., expected spend)
2. Feature Engineering
   - Create variables from raw data (e.g., logins, tenure)
   - Scale or encode if needed
3. Model Selection
   - Logistic Regression: Simple classification
   - Random Forest: More flexible and powerful
4. Train & Validate
   - Use K-Fold cross-validation
   - Track accuracy, precision, recall, ROC-AUC
5. Interpret Results
   - Examine feature importance
   - Validate business relevance

📂 See predictive modeling code example →<br><br><br>

### **📝 Deliverables for This Step**
1. For A/B Testing:
   - Experiment plan (groups, metrics, success criteria)
   - Sample size justification
   - Clean statistical results

2. For Modeling:
   - Feature list and preprocessing steps
   - Model performance metrics
   - Key insights (top predictors, expected outcomes)

---

## **7 | 🚀 Implementation & Monitoring**

This is where your analysis goes live, whether it’s a new experiment, a predictive model, or a business rule change.

**Your job is to make sure:**  
- The right data is collected  
- Stakeholders can track what’s happening  
- Issues are caught early and actioned fast  

### **1. 🔧 Instrumentation — Set Up the Right Tracking**

Goal: Ensure your product or system logs the right data to measure outcomes and impact.

What to do:  
Work with engineers or product owners to define key events

Clarify:  
- What to track (e.g., button clicks, transactions, feature usage)  
- When to trigger it (e.g., after confirmation, on success)  
- Which properties to include (e.g., user ID, timestamp, session length)  

Example Events:
`feature_shown`, `feature_clicked`, `conversion_completed`
`experiment_group_assigned`, `email_opened`, `app_crash`

✅ Tip: Always log a unique ID and timestamp for joinability and time-based analysis

### **2. 📊 Live Dashboards — Monitor Metrics in Real Time**

Goal: Keep all stakeholders informed of the impact and performance.

What to do:  
1. Build interactive dashboards in Looker, Tableau, Power BI, or Metabase
2. Segment by key dimensions (e.g., country, user type, traffic source)
3. Make dashboards user-friendly for:
   - PMs/Leads: Focused on conversion, engagement, adoption
   - Data Teams: More granular metrics, segment breakdowns, time trends

✅ Tip: Always include a filterable control/test toggle when tracking A/B experiments

### **3. 🚨 Alerting — Catch Problems Early**

Goal: Detect and act on unexpected changes before they escalate.

What to do:  
1. Set alerts on key performance indicators or guardrail metrics
2. Use built-in alert systems (Looker/Metabase), or custom tools like Slack alerts via webhook

Examples:
- Engagement drops >10% vs baseline  
- Daily revenue dips below expected range  
- Error rate > threshold in model predictions  

🧭 Example Workflow (Generalized)

1. Analyst defines the success metrics and key events to track  
2. Engineer implements tracking in the product  
3. Data flows to warehouse (e.g., BigQuery, Snowflake)  
4. Dashboard shows real-time performance (Tableau, Looker)  
5. Alerts are triggered if drop-off, errors, or spikes are detected<br><br><br>

📝 Deliverables from This Step

| Deliverable            | Description                                             |
| ---------------------- | ------------------------------------------------------- |
| ✅ Instrumented Events | Event specs implemented in product/system               |
| 📊 Dashboards          | Real-time visibility into performance                   |
| 🚨 Alerts              | Monitors for guardrail metrics and unexpected anomalies |

[➡️ Monitoring checklist here](./Implementation_Monitoring_Checklist.md)

---

## **8 | 📈 Results Analysis**

Once the experiment or model runs, it’s time to answer three key questions:  
- Did it work?  
- How much did it help?  
- Who benefited the most—and is the result trustworthy?  

&nbsp;

### 1. ✅ Lift & ROI Analysis — "What Was the Impact?"

Goal: Quantify the business value created by your change.

Key metrics to evaluate:  
- Lift → Difference between test and control (e.g., +12% conversion rate)  
- ROI → Return on investment  

`ROI = (Gain from Improvement − Cost of Change)/Cost of Change`
​
💡 Common Metrics:  
- Conversion Uplift: +12% in test group  
- Revenue Impact: +$50,000 increase in monthly revenue  
- Cost Reduction: –$5 per user in support costs  
- Efficiency Gains: Reduced time-to-resolution by 20%  

✅ Tip: Always compare absolute values and relative % change for context.

&nbsp;

### 2. 👥 Segment Deep-Dive — "Who Did It Work Best For?"

Goal: Identify which user segments saw the biggest impact.

🔍 Example Segmentation Dimensions:  
- User type: New vs Returning  
- Geography: North America, EU, APAC  
- Traffic source: Organic, Paid, Referral  
- Product usage: Light vs Power users  
- Subscription tier: Free vs Premium  

What to do:  
1. Slice your primary metric by these segments  
2. Compare performance across groups  
3. Identify high-performing or underperforming cohorts  

&nbsp;

### 3. 🧪 Statistical Confidence — "Can We Trust the Results?"

Goal: Make sure your findings aren’t just random noise.

Key Tools:
- T-Test / Chi-Square Test → Compare group outcomes
- P-Value < 0.05 → Indicates statistically significant result
- Pre/Post Checks → Validate that groups were balanced before the experiment
- CUPED (if available) → Reduces variance for faster and more accurate results

✅ Tip: Always check for external events that could bias your results.<br><br><br>

### 📝 Deliverables from This Step

| Deliverable             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| 📊 Impact Summary       | % lift, \$ value, efficiency gains                      |
| 👥 Segment Analysis     | Segment-level comparisons with takeaways                |
| 📈 ROI Calculation      | Business value gained vs cost of implementation         |
| 🧪 Statistical Evidence | P-values, confidence intervals, pre-test balance checks |

---

## **9 | 🧠 Tell the Story & Recommend**

The analysis is complete—now it’s time to communicate.  
This step turns findings into action by helping stakeholders understand what happened, why it matters, and what to do next.

🎯 Goals of This Step  
- Summarize the problem, method, and key results  
- Visualize the business impact clearly  
- Recommend what to do next with data-driven confidence  
- Align stakeholders across product, business, and tech  

&nbsp;

### 1. 📊 Build an Executive Dashboard — “Show the Story Visually”

Purpose: Give stakeholders a clear, visual snapshot of results and impact.

What to include:  
- Key KPIs: before vs. after comparison  
- Line/bar charts for trends  
- KPI cards for impact ($, %, time savings)  
- Clear labels + filters (e.g., date, user segment, experiment group)  
- Tools: Tableau, Looker, Power BI, Metabase   

🎨 Tip: Keep it clean, consistent, and filterable.

&nbsp;

### 2. 📝 Create a “TL;DR” Slide — “One Slide, One Story”

Purpose: Quickly convey the essence of your analysis in 30 seconds or less.

Template Structure:

| Section         | Description                                   |
| --------------- | --------------------------------------------- |
| **Problem**     | What were you solving for?                    |
| **Method**      | What approach did you take? (EDA, A/B, model) |
| **Result**      | What was the outcome? (impact, significance)  |
| **Next Action** | What should we do next?                       |

💡 Tip: Use this slide for team meetings, exec updates, or investor recaps.

&nbsp;

### 3. 📌 Recommend a Clear Action Plan — “Now What?”

Purpose: Turn insights into decisions and execution.

What to include:  
- Your recommended action (scale, iterate, pause)  
- Rationale based on business impact  
- Timeline and owners  
- Optional: next test ideas or variations  

Example Recommendations:  
- ✅ Roll out to all users by end of Q2  
- 🧪 Test variant B for segment X next  
- 🕒 Monitor KPIs for 3 months post-launch  
- ❌ Pause rollout due to low impact and high cost  

🔁 Tip: Be realistic. Tie recommendations to business goals and capacity.<br><br><br>

### 📦 Deliverables from This Step

| Deliverable            | Description                                         |
| ---------------------- | --------------------------------------------------- |
| 📊 Executive Dashboard | Before/after metrics, trends, visual impact         |
| 🧾 TL;DR Slide         | 1-slide summary: problem → method → result → action |
| 🧭 Action Plan         | Next steps, rationale, timeline, ownership          |

---

## **10 | 📚 Document & Keep Learning**

Great analysis doesn’t end with insights, it lives on through documentation, reuse, and monitoring.  
This step ensures your work is understandable, repeatable, and future-proof.

🎯 Goals of This Step  
- Document the logic, decisions, and results behind your analysis  
- Keep metric definitions and logic aligned across the org  
- Feed insights back into the product or data platform  
- Plan for long-term tracking and maintenance  

&nbsp;

### 1. 🗂️ Document Queries & Code — “Make It Reusable”

Goal: Ensure others can read, rerun, and adapt your work.

What to do:
1. Save SQL queries, Python notebooks, and assumptions in a shared space (e.g., GitHub, Confluence, internal wiki)  
2. Add comments explaining:  
   - What the query/script does  
   - Why filters or logic were applied  
5. Write a short README.md or inline summary for each file  

✅ Tip: Include input assumptions, test periods, and variable definitions directly in your notebooks or scripts.

&nbsp;

### 2. 📖 Update the Data Dictionary — “Keep Metrics Consistent”

Goal: Maintain a shared understanding of key fields, metrics, and calculations.

What to do:  
1. Use an internal tool (e.g., Notion, Confluence, Looker Dictionary, Google Sheets)  
2. Define:  
   - What each metric means (e.g., conversion_rate)  
   - How it’s calculated (e.g., unique_conversions / unique_visitors)  
   - Which tables/fields are involved  

&nbsp;

### 3. 🔁 Feed Learnings Back into the System — “Make It Smarter”

Goal: Use what you learned to improve tracking, data models, or user targeting.

What to do:  
1. Feed back new features or labels into your datasets  
   - E.g., add a high_risk_user flag based on model output  
   - Tag users who received a feature or treatment for future comparison  
2. Share findings with product, marketing, or engineering to refine:  
   - Event tracking  
   - User segmentation  
   - Model inputs and data pipelines  

&nbsp;

### 4. 📅 Schedule Audits — “Check Back Later”

Goal: Ensure your models, KPIs, and metrics remain accurate and relevant over time.

What to do:  
1. Set a reminder for a quarterly review of:  
   - Model performance (e.g., accuracy, drift, recalibration)  
   - Metric health (e.g., drop-off, new patterns)  
2. Ask:  
   - Is the ROI/lift still holding?  
   - Have user behaviors changed?  
   - Does the model still generalize?<br><br><br>

### 📦 Deliverables from This Step

| Deliverable                  | Description                                              |
| ---------------------------- | -------------------------------------------------------- |
| 📁 Documented Code & Queries | With comments, assumptions, and summaries                |
| 📖 Updated Data Dictionary   | Clear definitions and logic for key metrics              |
| 🔁 System Feedback           | New fields, labels, or product insights fed into systems |
| 📅 Monitoring Plan           | Future audit schedule with owners and timing             |

---

