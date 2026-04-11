# Machine Learning Cheat Sheet (scikit-learn)

A practical reference for machine learning workflows using scikit-learn.

> **Using Spark?** For the PySpark equivalent of everything in this file, see `pyspark_cheatsheet.md` — Spark ML section at the bottom.

---

## Setup

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, LabelEncoder, OneHotEncoder
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, r2_score,
    accuracy_score, f1_score, precision_score, recall_score,
    roc_auc_score, confusion_matrix, classification_report
)
from sklearn.pipeline import Pipeline
```

---

## Shared Workflow

These steps apply to regression, classification, and most clustering tasks.

### Load and prepare data
```python
df = pd.read_csv("data.csv")
X = df.drop(columns=["target"])
y = df["target"]
```

### Encode a categorical target column (classification)
```python
le = LabelEncoder()
y = le.fit_transform(y)
```

### Encode categorical feature columns
```python
from sklearn.preprocessing import OneHotEncoder
from sklearn.compose import ColumnTransformer

cat_cols = ["city", "department"]
num_cols = ["age", "salary"]

preprocessor = ColumnTransformer([
    ("num", StandardScaler(), num_cols),
    ("cat", OneHotEncoder(handle_unknown="ignore"), cat_cols)
])
```

### Train / test split
```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

### Scale numeric features
```python
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)
```

> Always fit the scaler on training data only, then transform both sets.

---

## Regression

Predicts a continuous numeric value.

### Available models

| Model | Import |
|-------|--------|
| Linear Regression | `from sklearn.linear_model import LinearRegression` |
| Ridge (L2 regularization) | `from sklearn.linear_model import Ridge` |
| Lasso (L1 regularization) | `from sklearn.linear_model import Lasso` |
| Decision Tree | `from sklearn.tree import DecisionTreeRegressor` |
| Random Forest | `from sklearn.ensemble import RandomForestRegressor` |
| Gradient Boosting | `from sklearn.ensemble import GradientBoostingRegressor` |
| Support Vector Regression | `from sklearn.svm import SVR` |

### Train a model
```python
from sklearn.ensemble import RandomForestRegressor

model = RandomForestRegressor(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
```

### Make predictions
```python
y_pred = model.predict(X_test)
```

### Evaluate
```python
rmse = mean_squared_error(y_test, y_pred, squared=False)
mae  = mean_absolute_error(y_test, y_pred)
r2   = r2_score(y_test, y_pred)

print(f"RMSE: {rmse:.4f} | MAE: {mae:.4f} | R²: {r2:.4f}")
```

---

## Classification

Predicts a discrete class label.

### Available models

| Model | Import |
|-------|--------|
| Logistic Regression | `from sklearn.linear_model import LogisticRegression` |
| Decision Tree | `from sklearn.tree import DecisionTreeClassifier` |
| Random Forest | `from sklearn.ensemble import RandomForestClassifier` |
| Gradient Boosting | `from sklearn.ensemble import GradientBoostingClassifier` |
| Support Vector Machine | `from sklearn.svm import SVC` |
| K-Nearest Neighbors | `from sklearn.neighbors import KNeighborsClassifier` |
| Naive Bayes | `from sklearn.naive_bayes import GaussianNB` |

### Train a model
```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
```

### Make predictions
```python
y_pred = model.predict(X_test)
y_prob = model.predict_proba(X_test)[:, 1]   # probability scores for AUC
```

### Evaluate
```python
accuracy  = accuracy_score(y_test, y_pred)
f1        = f1_score(y_test, y_pred, average="weighted")
precision = precision_score(y_test, y_pred, average="weighted")
recall    = recall_score(y_test, y_pred, average="weighted")
auc       = roc_auc_score(y_test, y_prob)    # binary classification only

print(f"Accuracy: {accuracy:.4f} | F1: {f1:.4f} | Precision: {precision:.4f} | Recall: {recall:.4f} | AUC: {auc:.4f}")
```

### Confusion matrix
```python
cm = confusion_matrix(y_test, y_pred)
print(cm)
```

### Full classification report
```python
print(classification_report(y_test, y_pred))
```

### Feature importance (tree-based models)
```python
importances = pd.Series(model.feature_importances_, index=X.columns)
importances.sort_values(ascending=False).head(10)
```

---

## Clustering

Groups data without a target label. No train/test split — use the full dataset.

### Available models

| Model | Import | Best for |
|-------|--------|----------|
| K-Means | `from sklearn.cluster import KMeans` | Compact, spherical clusters |
| DBSCAN | `from sklearn.cluster import DBSCAN` | Irregular shapes, noise detection |
| Agglomerative | `from sklearn.cluster import AgglomerativeClustering` | Hierarchical structure |
| Gaussian Mixture | `from sklearn.mixture import GaussianMixture` | Soft/probabilistic clusters |

### Train and assign clusters
```python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, random_state=42, n_init=10)
labels = model.fit_predict(X)
```

### Evaluate — Silhouette score
```python
from sklearn.metrics import silhouette_score

score = silhouette_score(X, labels)
print(f"Silhouette Score: {score:.4f}")
# Ranges from -1 to 1. Closer to 1 means well-separated clusters.
```

### Find the best K — elbow method
```python
inertias = []
for k in range(2, 11):
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    km.fit(X)
    inertias.append((k, km.inertia_))

for k, inertia in inertias:
    print(f"k={k}  inertia={inertia:.2f}")
```

---

## Model Selection & Tuning

These work the same for both regression and classification — just swap the model and evaluator.

### Cross-validation
```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5, scoring="f1_weighted")
print(f"CV F1: {scores.mean():.4f} ± {scores.std():.4f}")

# Common scoring options:
# regression:     "r2", "neg_mean_squared_error", "neg_mean_absolute_error"
# classification: "accuracy", "f1_weighted", "roc_auc"
```

### GridSearchCV — exhaustive search over a parameter grid
```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    "n_estimators": [50, 100, 200],
    "max_depth": [3, 5, 10, None]
}

grid_search = GridSearchCV(
    estimator=RandomForestClassifier(random_state=42),
    param_grid=param_grid,
    cv=5,
    scoring="f1_weighted",
    n_jobs=-1
)
grid_search.fit(X_train, y_train)

print(grid_search.best_params_)
best_model = grid_search.best_estimator_
```

### RandomizedSearchCV — faster, samples random combinations
```python
from sklearn.model_selection import RandomizedSearchCV

param_dist = {
    "n_estimators": [50, 100, 200, 300],
    "max_depth": [3, 5, 10, None],
    "min_samples_split": [2, 5, 10]
}

random_search = RandomizedSearchCV(
    estimator=RandomForestClassifier(random_state=42),
    param_distributions=param_dist,
    n_iter=20,
    cv=5,
    scoring="f1_weighted",
    random_state=42,
    n_jobs=-1
)
random_search.fit(X_train, y_train)

print(random_search.best_params_)
```

---

## Pipelines

Chains preprocessing and model into one clean object. Prevents data leakage and makes deployment easier.

### Basic pipeline — scaler + model
```python
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", RandomForestClassifier(n_estimators=100, random_state=42))
])

pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)
```

### Pipeline with preprocessing for mixed column types
```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import RandomForestClassifier

num_cols = ["age", "salary"]
cat_cols = ["city", "department"]

preprocessor = ColumnTransformer([
    ("num", StandardScaler(), num_cols),
    ("cat", OneHotEncoder(handle_unknown="ignore"), cat_cols)
])

pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", RandomForestClassifier(n_estimators=100, random_state=42))
])

pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)
```

### Use a pipeline with GridSearchCV
```python
param_grid = {
    "model__n_estimators": [100, 200],
    "model__max_depth": [5, 10, None]
}

grid_search = GridSearchCV(pipeline, param_grid, cv=5, scoring="f1_weighted")
grid_search.fit(X_train, y_train)
```

> Note the double underscore `__` to reference parameters inside a pipeline step.

---

## Saving & Loading Models

### Save with joblib (recommended for sklearn)
```python
import joblib

joblib.dump(model, "models/my_model.pkl")
joblib.dump(pipeline, "models/my_pipeline.pkl")
```

### Load
```python
model = joblib.load("models/my_model.pkl")
pipeline = joblib.load("models/my_pipeline.pkl")
```

### Save with pickle
```python
import pickle

with open("models/my_model.pkl", "wb") as f:
    pickle.dump(model, f)

with open("models/my_model.pkl", "rb") as f:
    model = pickle.load(f)
```

> Prefer `joblib` over `pickle` for sklearn models — it handles large numpy arrays more efficiently.

---

*Last updated: 2026*
