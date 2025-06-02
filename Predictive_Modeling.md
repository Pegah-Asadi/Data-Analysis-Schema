# 🤖 Predictive Modeling – Binary Classification Starter
Target example: churn prediction (0/1)
Swap out the estimator for regression or multiclass tasks as needed.

> **Requires:** `pandas`, `scikit-learn`, `matplotlib`

```python

# -------------------------------------------------------------
# 📦 0 · LOAD DATA + BASIC SPLIT
# -------------------------------------------------------------
from pathlib import Path
import pandas as pd
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.metrics import (
    roc_auc_score, RocCurveDisplay,
    classification_report, ConfusionMatrixDisplay
)
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
import matplotlib.pyplot as plt

# Load the dataset (replace this with your actual path/source)
df = pd.read_parquet("data/churn.parquet")  # 🧩 Replace loader if needed

# Split into features (X) and target (y)
y = df["churned"]             # 🎯 Binary target: 1 = churned, 0 = not churned
X = df.drop(columns="churned")

# Train-test split (80/20) — stratify to preserve class ratio
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.20, stratify=y, random_state=42
)

# -------------------------------------------------------------
# 🛠️ 1 · PREPROCESSING PIPELINE
# -------------------------------------------------------------

# Automatically detect numeric and categorical columns
num_cols = X.select_dtypes(include="number").columns
cat_cols = X.select_dtypes(exclude="number").columns

# Preprocessing steps:
# - Standardize numeric columns
# - One-hot encode categorical columns (ignore unknowns during inference)
preprocessor = ColumnTransformer([
    ("num", StandardScaler(), num_cols),
    ("cat", OneHotEncoder(handle_unknown="ignore"), cat_cols)
])

# -------------------------------------------------------------
# 🤖 2 · MODEL SELECTION + CROSS-VALIDATION
# -------------------------------------------------------------

# Define candidate models
models = {
    "logreg": LogisticRegression(max_iter=200, n_jobs=-1),
    "rf": RandomForestClassifier(
        n_estimators=300,
        max_depth=None,
        n_jobs=-1,
        class_weight="balanced"  # ✅ useful for imbalanced datasets
    )
}

# Train and evaluate each model using 5-fold cross-validation
# Scoring: ROC-AUC (suitable for binary classification, esp. with imbalance)
for name, est in models.items():
    pipe = Pipeline([("prep", preprocessor), ("model", est)])
    
    cv_auc = cross_val_score(pipe, X_train, y_train,
                             cv=5, scoring="roc_auc")
    
    print(f"{name}  |  CV ROC-AUC = {cv_auc.mean():.3f} ± {cv_auc.std():.3f}")

# -------------------------------------------------------------
# ✅ 3 · FINALIZE BEST MODEL & EVALUATE ON TEST SET
# -------------------------------------------------------------

# Pick the best-performing model based on cross-validation
# (Here we're choosing random forest manually — update if logistic wins)
best = Pipeline([
    ("prep", preprocessor),
    ("model", models["rf"])
]).fit(X_train, y_train)

# Generate predictions
y_pred = best.predict(X_test)                   # Predicted classes
y_prob = best.predict_proba(X_test)[:, 1]       # Predicted probabilities (for ROC)

# Classification report: precision, recall, F1, etc.
print(classification_report(y_test, y_pred))

# Evaluate using ROC-AUC — good for overall ranking quality
roc = roc_auc_score(y_test, y_prob)
print(f"Test ROC-AUC = {roc:.3f}")

# Plot ROC curve
RocCurveDisplay.from_predictions(y_test, y_prob)
plt.show()

# -------------------------------------------------------------
# 📊 4 · INTERPRETABILITY — FEATURE IMPORTANCE (Tree-based models only)
# -------------------------------------------------------------

# Extract feature importances from the trained Random Forest
# Works only if the model has the `feature_importances_` attribute
feat_imp = pd.Series(
    best.named_steps["model"].feature_importances_,
    index=best.named_steps["prep"].get_feature_names_out()
).sort_values(ascending=False).head(20)

# Display top 20 contributing features
print("Top predictors:\n", feat_imp)
```
