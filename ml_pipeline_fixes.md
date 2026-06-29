# 🛠️ Machine Learning Pipeline Refactoring & Bug Fixes

This document details the code defects found in the original Jupyter Notebook [phishing_data_analytics.ipynb](file:///Users/user/Documents/Bus_Data_Project/emai_phising_notebook/phishing_data_analytics.ipynb) and how we refactored the pipeline to ensure statistical validity, correct prediction behavior, and clean execution.

---

## 1. StandardScaler Data Leakage (Fixed)

### The Defect
```python
# StandardScaler fit on the ENTIRE dataset before train-test split
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

X_train, X_test, y_train, y_test = train_test_split(X_scaled, y, test_size=0.2, random_state=42)
```
- **Consequence**: The mean and standard deviation of the test split leaked into the model training set, introducing data leakage and resulting in overly optimistic model validation metrics.

### Refactored Solution
```python
# Split the raw data first to isolate the test set
X_train_raw, X_test_raw, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Fit scaler ONLY on training data, then transform both
scaler = StandardScaler()
X_train = pd.DataFrame(scaler.fit_transform(X_train_raw), columns=X.columns)
X_test = pd.DataFrame(scaler.transform(X_test_raw), columns=X.columns)
```

---

## 2. Production Pipeline Inference Scaling Bug (Fixed)

### The Defect
```python
# The final pipeline scaler was fit on X_train (which was ALREADY scaled)
production_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', best_model)
])
production_pipeline.fit(X_train, y_train)
```
- **Consequence**: The pipeline's internal scaler fit on scaled data (mean of ~0 and std of ~1), making it a dummy scaler. When raw features (e.g. `urgency_score = 8`) were submitted to `predict_phishing_email` in production, they were not scaled properly. The model received raw data while expecting scaled variables, yielding a **0.00% phishing probability** for high-risk emails.

### Refactored Solution
```python
# Train the production pipeline on RAW training data
production_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', best_model)
])
production_pipeline.fit(X_train_raw, y_train)
```
- **Result**: Incoming raw features are scaled correctly using training set parameters before inference, restoring accurate classifications.

---

## 3. Discarded GridSearchCV Hyperparameter Tuning (Fixed)

### The Defect
- **Consequence**: `GridSearchCV` was executed for Random Forest and XGBoost to find the best parameters, but the tuned models (`rf_grid.best_estimator_`, `xgb_grid.best_estimator_`) were discarded. The notebook continued comparing only the un-tuned models, saving a default baseline model.

### Refactored Solution
- Added the optimized models back to the `results` evaluation dictionary:
```python
results['Random Forest (Tuned)'] = {
    'model': rf_grid.best_estimator_,
    'y_pred': rf_grid.best_estimator_.predict(X_test)
}
results['XGBoost (Tuned)'] = {
    'model': xgb_grid.best_estimator_,
    'y_pred': xgb_grid.best_estimator_.predict(X_test)
}
```

---

## 4. Multi-Process Permutation Importance Warning Spam (Fixed)

### The Defect
```python
perm_importance = permutation_importance(..., n_jobs=-1)
```
- **Consequence**: Running permutation importance with `n_jobs=-1` spawns parallel subprocesses via `joblib`. These subprocesses bypass Python's main thread warning filters, flooding the notebook logs with `RuntimeWarning: divide by zero encountered in matmul` warnings (caused by the perfectly separable data).

### Refactored Solution
- Configured execution to run sequentially (`n_jobs=1`) inside a localized `warnings.catch_warnings()` context block:
```python
import warnings

with warnings.catch_warnings():
    warnings.filterwarnings('ignore', category=RuntimeWarning)
    perm_importance = permutation_importance(
        best_model, X_test, y_test, 
        n_repeats=10, 
        random_state=42, 
        n_jobs=1  # Run sequentially to respect warnings filter
    )
```
- **Result**: Sub-second execution with completely silenced warnings.
