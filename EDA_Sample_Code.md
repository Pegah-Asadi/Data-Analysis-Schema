# 📊 Exploratory Data Analysis (EDA) – Sample Code

This file includes generalized Python snippets to help you explore data distributions, spot outliers, and identify variable relationships before moving into modeling or testing.

---

## 1️⃣ Visualize Distributions

Understand how a numeric variable is distributed across your dataset.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Load dataset
df = pd.read_csv('your_data.csv')

# Replace 'feature_name' with your column (e.g., 'session_duration', 'revenue')
sns.histplot(df['feature_name'], bins=30, kde=True)
plt.title('Distribution of Feature')
plt.xlabel('Feature Value')
plt.ylabel('Count')
plt.show()
```

## 2️⃣ Detect Outliers

Use boxplots to visually detect extreme values in a numeric column.

```python
# Replace 'feature_name' with the numeric column you want to analyze
sns.boxplot(x=df['feature_name'])
plt.title('Outliers in Feature')
plt.show()
```

## 3️⃣ Identify Correlations

Check how numeric variables are related using a correlation matrix.

```python
# Create a correlation matrix
corr_matrix = df.corr(numeric_only=True)

# Plot heatmap
plt.figure(figsize=(10, 8))
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', fmt=".2f")
plt.title('Correlation Between Numeric Features')
plt.show()
```

📝 Notes  
- Replace 'feature_name' with actual column names in your dataset.  
- These plots help you catch data issues and generate hypotheses for deeper testing.  
