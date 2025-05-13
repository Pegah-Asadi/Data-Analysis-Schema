# Data Analysis Schema

> A practical, repeatable framework for structured data analysis.

## "What's the plan?"

That’s the question I hear most from data analysts, myself included.  
While every project has its own goal, strong analyses tend to follow the same core structure. This schema breaks it down into 10 clear steps, from business framing to storytelling and documentation.
Some steps are strategic, others are technical. Some include code, others are just mindset.

Whether you're new to the role or just looking to sharpen your workflow, I hope this guide brings structure and clarity to your process.

👇 Let’s dive in.

## 🧠 0 | Frame the Business Question

Before any SQL or Python, clarify:

✅ What’s the business goal? (e.g. retention, cost reduction, partner ROI)  
📊 What metrics define success? (e.g. MAUs, cost per engaged member, NPS)  
⚠️ What are the constraints? (e.g. privacy laws, sample size, data freshness)

🛡️ Especially in regulated domains:

→ Use anonymized or pseudonymized data only  
→ Keep audit trails clean  
→ Only use the minimum necessary fields

[💻 See code examples in SQL and Python](https://github.com/Pegah-Asadi/Data-Analysis-Schema/blob/main/Pseudonymization.md).

## 🤝 1 | Stakeholder Alignment

Once the business question is framed, the next step is aligning with stakeholders.
Before touching data, I meet with each stakeholder group to understand their goals, constraints, and definitions of success.  
Here’s how I align with each:
| Role             | What they care about                    | What I deliver                     |
| ---------------- | --------------------------------------- | ---------------------------------- |
| Product Manager  | User experience & engagement            | 1-pager with problem & KPIs        |
| Legal/Compliance | Data privacy & governance (HIPAA/PHIPA) | Data-flow diagram, pseudonym steps |
| Engineering/Data | Feasibility, pipelines, event tracking	 | Clear data needs + tracking plan   |
| Executives       | Business impact (ROI, revenue, savings) | Summary slide with outcomes        |
