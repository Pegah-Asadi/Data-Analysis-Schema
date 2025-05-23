# **Data Analysis Schema**

> A practical, repeatable framework for structured data analysis.

## "What's the **plan**?"

That’s the question I hear most from data analysts, myself included.  
While every project has its own goal, strong analyses tend to follow the same core structure. This schema breaks it down into **10 clear steps**, from business framing to storytelling and documentation.
Some steps are strategic, others are technical. Some include code, others are just mindset.

Whether you're new to the role or just looking to sharpen your workflow, I hope this guide brings structure and clarity to your process.

👇 Let’s dive in.

---

## 🧠 **0 | Frame the Business Question**

Before any SQL or Python, clarify:

✅ What’s the **business goal**? (e.g. retention, cost reduction, partner ROI)  
📊 What **metrics** define success? (e.g. MAUs, cost per engaged member, NPS)  
⚠️ What are the **constraints**? (e.g. privacy laws, sample size, data freshness)

🛡️ Especially in regulated domains:

→ Use anonymized or pseudonymized data only  
→ Keep audit trails clean  
→ Only use the minimum necessary fields

[💻 See code examples in SQL and Python](https://github.com/Pegah-Asadi/Data-Analysis-Schema/blob/main/Pseudonymization.md).

---

## 🤝 **1 | Stakeholder Alignment**

Once the business question is framed, the next step is aligning with stakeholders.  

Before touching data, I meet with each stakeholder group to understand their goals, constraints, and definitions of success.  
Here’s how I align with each:
| Role             | What they care about                    | What I deliver                     |
| ---------------- | --------------------------------------- | ---------------------------------- |
| Product Manager  | User experience & engagement            | 1-pager with problem & KPIs        |
| Legal/Compliance | Data privacy & governance (HIPAA/PHIPA) | Data-flow diagram, pseudonym steps |
| Engineering/Data | Feasibility, pipelines, event tracking	 | Clear data needs + tracking plan   |
| Executives       | Business impact (ROI, revenue, savings) | Summary slide with outcomes        |
| Clinical Ops     | Health outcomes, operational efficiency | KPI glossary, outcome definitions  |

---

## 🧾 **2 | Data Audit & Governance**

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
| events           | App/page interactions         | user\_id, event\_name     |
| support\_tickets | Customer issues & resolutions | ticket\_id, issue\_type   |

🧹 Check Data Quality

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

Common best practices:  
- Use pseudonymized or anonymized IDs when sharing data  
- Limit fields to the minimum necessary  
- Keep processing logs or audit trails if required  
- Understand any compliance frameworks (e.g., GDPR, SOC2, HIPAA if applicable)

📄 Privacy Checklist:  
- Personally identifiable info (PII) removed or masked  
- Only essential fields used  
- Shared files don’t expose sensitive data  
- Access is role-based and logged<br><br><br>

### 🗂️ Deliverables for This Step

📋 Data Inventory Table  
→ Clear list of all relevant tables, key fields, and purpose

📊 Data Quality Report  
→ Null rates, duplicate counts, basic sanity checks

🔐 Privacy & Governance Plan  
→ Fields masked or pseudonymized, access rationale, logging strategy

---

## **🧰 3 | Extraction & Preparation**

This is where raw data turns into something meaningful. Your goal: extract the right data, clean it, and shape it into an analysis-ready format.

### 🏗️ Step-by-Step Workflow

| Step          | Common Tools                              | Output                              |
| ------------- | ----------------------------------------- | ----------------------------------- |
| **Extract**   | SQL (BigQuery, Snowflake, Redshift, etc.) | Source tables filtered & joined     |
| **Transform** | Python (pandas), dbt, R, Spark            | Cleaned, structured dataset         |
| **Validate**  | Python (asserts, checks), SQL             | QC/validation report, sanity checks |<br><br><br>

### 🔄 Key Principles

✅ Write Modular SQL  
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

🧼 Transform in Code  
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

🧪 Validate Your Output  
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

### 🧹 Bonus: Ready-to-Use Python Scripts
Want practical code to speed up cleaning and validation?

📂 Check out the general [Python scripts](https://github.com/Pegah-Asadi/Data-Analysis-Schema/blob/main/Data_Cleaning_and_Validation.md) for data cleaning and validation

These cover:  
🔁 Removing duplicates  
🕳️ Handling missing values  
🚨 Detecting outliers  
✅ Validating schema, value ranges, and data integrity using assert statements  

## **🔍 4 | Exploratory Data Analysis (EDA) & Hypothesis Generation**

Before you build a model or design an experiment, get to know your data.  
EDA helps you spot patterns, uncover issues, and identify the relationships worth testing.

🎯 Goals of This Step  
- Understand distributions, trends, and anomalies  
- Spot correlations and patterns  
- Form hypotheses grounded in real data  
- Get business and technical teams aligned before deeper analysis

### ⚙️ EDA Workflow Summary

| Subtask                  | Tool(s)                  | Why It Matters                        | Example Action                      |
| ------------------------ | ------------------------ | ------------------------------------- | ----------------------------------- |
| Distributions & Outliers | Python (Seaborn, pandas) | Identify skewed data & extreme values | `sns.histplot()`, `sns.boxplot()`   |
| Time-based Trends        | Looker, Tableau          | Spot seasonal effects or churn spikes | Line charts by day/week/month       |
| Correlation Matrix       | Python (Seaborn)         | Understand variable relationships     | `df.corr()`, `sns.heatmap()`        |
| Variable Pair Trends     | Tableau or Python        | Visually inspect pairwise trends      | Scatter plots with trendlines       |
| Group Comparisons        | SciPy, statsmodels       | Check if group differences are real   | `ttest_ind()`, `chi2_contingency()` |<br><br><br>

### **🧭 1. Data Profiling: “What does the data look like?”**

Start by exploring:  
- Key metrics (e.g. # transactions, avg session duration, revenue per user)  
- Value ranges, missingness, outliers

📈 Sample: Time-based Trends (Looker/Tableau)  
- Plot engagement, revenue, or sign-ups over time  
- Slice by day-of-week, campaign, or product  
- Example: Identify drop-offs after onboarding, spikes during promotions<br><br><br>

### **🔗 2. Correlations & Patterns: “What moves together?”**

Look for relationships between variables.

Use this to:  
- Understand leading indicators of conversion, churn, or retention  
- Avoid multicollinearity in models

📂 [See sample code for EDA →](./EDA_Sample_Code.md) <br><br><br>

### **🧪 3. Statistical Testing: “Is this difference real?”**

Use tests like t-tests, chi-square, or Mann-Whitney U to validate patterns in data.

Use when comparing:  
- Conversion rates across landing pages  
- Engagement before/after a feature rollout  
- Retention between cohorts

📂 [See sample code for hypothesis testing →](./Hypothesis_Testing_Sample_Code.md)<br><br><br>

### **💡 4. Generate Hypotheses**

Good hypotheses are:
- Testable (you can collect or analyze the needed data)  
- Specific (clear variables, direction, expected impact)  
- Relevant (linked to a business goal)  

**💭 Generic Example Hypotheses:**
1. "Users who see the onboarding tutorial complete 25% more actions in week 1."  
2. "Product reviews with images have higher average ratings than those without."  
3. "Sending reminder emails reduces cart abandonment by 10%."  
4. "Customers acquired via referral have higher LTV than paid users."<br><br><br>

### 🧾 Deliverables for This Step  
- Visualizations (distributions, trends, correlation plots)  
- Group comparison test results (e.g. t-test output)  
- A short list of hypotheses to test next  
- Optional: a 1-pager summarizing key EDA findings for your stakeholders  
