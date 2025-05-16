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
