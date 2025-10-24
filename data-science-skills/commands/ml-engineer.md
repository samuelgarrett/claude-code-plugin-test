# ML Engineer

**Description**: Build, train, and deploy machine learning models following MLOps best practices

## ML Project Workflow

1. **Define Problem** - Classification, regression, clustering?
2. **Prepare Data** - Clean, split, scale
3. **Choose Model** - Select appropriate algorithm
4. **Train Model** - Fit to training data
5. **Evaluate** - Test performance
6. **Tune** - Optimize hyperparameters
7. **Deploy** - Put model in production

## Data Preparation

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, LabelEncoder

# Load data
X = df[['feature1', 'feature2', 'feature3']]
y = df['target']

# Split data (80/20 train/test)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Scale features
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)  # Use same scaler!

# Encode categorical labels
label_encoder = LabelEncoder()
y_train_encoded = label_encoder.fit_transform(y_train)
```

## Classification Models

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix

# Train model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train_scaled, y_train)

# Make predictions
y_pred = model.predict(X_test_scaled)
y_pred_proba = model.predict_proba(X_test_scaled)

# Evaluate
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(f"Precision: {precision_score(y_test, y_pred, average='weighted'):.4f}")
print(f"Recall: {recall_score(y_test, y_pred, average='weighted'):.4f}")
print(f"F1 Score: {f1_score(y_test, y_pred, average='weighted'):.4f}")

print("\nClassification Report:")
print(classification_report(y_test, y_pred))

print("\nConfusion Matrix:")
print(confusion_matrix(y_test, y_pred))
```

## Regression Models

```python
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error

# Train model
model = GradientBoostingRegressor(n_estimators=100, random_state=42)
model.fit(X_train_scaled, y_train)

# Predict
y_pred = model.predict(X_test_scaled)

# Evaluate
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
mae = mean_absolute_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)

print(f"RMSE: {rmse:.4f}")
print(f"MAE: {mae:.4f}")
print(f"R² Score: {r2:.4f}")
```

## Model Selection & Comparison

```python
from sklearn.model_selection import cross_val_score

models = {
    'Logistic Regression': LogisticRegression(),
    'Random Forest': RandomForestClassifier(),
    'Gradient Boosting': GradientBoostingClassifier(),
}

for name, model in models.items():
    scores = cross_val_score(model, X_train_scaled, y_train, cv=5)
    print(f"{name}: {scores.mean():.4f} (+/- {scores.std():.4f})")
```

## Hyperparameter Tuning

```python
from sklearn.model_selection import GridSearchCV

# Define parameter grid
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [None, 10, 20],
    'min_samples_split': [2, 5, 10]
}

# Grid search
grid_search = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='f1_weighted',
    n_jobs=-1
)

grid_search.fit(X_train_scaled, y_train)

print(f"Best parameters: {grid_search.best_params_}")
print(f"Best score: {grid_search.best_score_:.4f}")

# Use best model
best_model = grid_search.best_estimator_
```

## Feature Importance

```python
# For tree-based models
importances = model.feature_importances_
feature_importance_df = pd.DataFrame({
    'feature': X.columns,
    'importance': importances
}).sort_values('importance', ascending=False)

print(feature_importance_df)

# Plot
plt.barh(feature_importance_df['feature'], feature_importance_df['importance'])
plt.xlabel('Importance')
plt.title('Feature Importance')
plt.show()
```

## Save & Load Models

```python
import joblib

# Save model
joblib.dump(model, 'model.pkl')
joblib.dump(scaler, 'scaler.pkl')

# Load model
loaded_model = joblib.load('model.pkl')
loaded_scaler = joblib.load('scaler.pkl')

# Make predictions
new_data = [[value1, value2, value3]]
new_data_scaled = loaded_scaler.transform(new_data)
prediction = loaded_model.predict(new_data_scaled)
```

## Best Practices

### 1. **Always Split Data Before Scaling**
```python
# ✅ Correct order
X_train, X_test = train_test_split(X, y)
scaler.fit(X_train)  # Fit only on training
X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)

# ❌ Wrong: Data leakage
scaler.fit(X)  # Don't fit on all data!
```

### 2. **Use Cross-Validation**
```python
from sklearn.model_selection import cross_validate

cv_results = cross_validate(
    model, X, y,
    cv=5,
    scoring=['accuracy', 'f1_weighted'],
    return_train_score=True
)
```

### 3. **Handle Imbalanced Data**
```python
from sklearn.utils.class_weight import compute_class_weight

# Compute class weights
class_weights = compute_class_weight(
    'balanced',
    classes=np.unique(y_train),
    y=y_train
)

# Use in model
model = RandomForestClassifier(class_weight='balanced')
```

## When to Use This Skill

- Building predictive models
- Training classification/regression models
- Evaluating model performance
- Tuning hyperparameters
- Deploying ML models

---

**Remember**: Good ML is methodical - clean data, proper splits, cross-validation, and thorough evaluation!
