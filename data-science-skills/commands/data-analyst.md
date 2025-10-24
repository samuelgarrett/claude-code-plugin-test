# Data Analyst

**Description**: Perform systematic data analysis using Python, pandas, and visualization libraries

## Core Workflow

1. **Load & Inspect Data**
2. **Clean & Transform Data**
3. **Explore & Visualize**
4. **Analyze & Model**
5. **Report Findings**

## Data Loading & Inspection

```python
import pandas as pd
import numpy as np

# Load data
df = pd.read_csv('data.csv')
# df = pd.read_excel('data.xlsx')
# df = pd.read_sql(query, connection)
# df = pd.read_json('data.json')

# Initial inspection
print(df.shape)              # (rows, columns)
print(df.info())             # Data types, null counts
print(df.describe())         # Statistical summary
print(df.head())             # First 5 rows
print(df.columns.tolist())   # Column names
print(df.dtypes)             # Data types

# Check for missing data
print(df.isnull().sum())
print(df.duplicated().sum())
```

## Data Cleaning

```python
# Handle missing values
df = df.dropna()                           # Drop rows with any null
df = df.dropna(subset=['important_col'])   # Drop if specific column null
df['col'] = df['col'].fillna(0)            # Fill with value
df['col'] = df['col'].fillna(df['col'].mean())  # Fill with mean

# Remove duplicates
df = df.drop_duplicates()
df = df.drop_duplicates(subset=['id'], keep='first')

# Fix data types
df['date'] = pd.to_datetime(df['date'])
df['price'] = df['price'].astype(float)
df['category'] = df['category'].astype('category')

# Handle outliers
from scipy import stats
df = df[(np.abs(stats.zscore(df['value'])) < 3)]  # Remove outliers

# Rename columns
df = df.rename(columns={'old_name': 'new_name'})
df.columns = df.columns.str.lower().str.replace(' ', '_')
```

## Data Transformation

```python
# Create new columns
df['total'] = df['quantity'] * df['price']
df['month'] = df['date'].dt.month
df['year'] = df['date'].dt.year

# Apply functions
df['normalized'] = df['value'] / df['value'].max()
df['category_encoded'] = df['category'].map({'A': 1, 'B': 2, 'C': 3})

# Groupby aggregations
summary = df.groupby('category').agg({
    'sales': ['sum', 'mean', 'count'],
    'profit': 'sum'
}).round(2)

# Pivot tables
pivot = df.pivot_table(
    values='sales',
    index='category',
    columns='region',
    aggfunc='sum',
    fill_value=0
)

# Merge datasets
merged = pd.merge(df1, df2, on='id', how='left')
```

## Exploratory Data Analysis

```python
# Distribution analysis
print(df['value'].value_counts())
print(df['value'].value_counts(normalize=True))  # Percentages

# Correlation
correlation = df[['col1', 'col2', 'col3']].corr()

# Cross-tabulation
pd.crosstab(df['category'], df['region'], normalize='index')
```

## Visualization

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Set style
sns.set_style('whitegrid')
plt.rcParams['figure.figsize'] = (12, 6)

# Histogram
plt.hist(df['value'], bins=30, edgecolor='black')
plt.xlabel('Value')
plt.ylabel('Frequency')
plt.title('Distribution of Values')
plt.show()

# Boxplot
sns.boxplot(data=df, x='category', y='value')
plt.title('Value Distribution by Category')
plt.show()

# Time series
df.set_index('date')['sales'].plot()
plt.title('Sales Over Time')
plt.show()

# Heatmap
sns.heatmap(correlation, annot=True, cmap='coolwarm', center=0)
plt.title('Correlation Matrix')
plt.show()

# Scatter plot
sns.scatterplot(data=df, x='x_value', y='y_value', hue='category')
plt.title('X vs Y by Category')
plt.show()
```

## Statistical Analysis

```python
from scipy import stats

# T-test
group1 = df[df['category'] == 'A']['value']
group2 = df[df['category'] == 'B']['value']
t_stat, p_value = stats.ttest_ind(group1, group2)
print(f"T-statistic: {t_stat:.4f}, P-value: {p_value:.4f}")

# Chi-square test
contingency_table = pd.crosstab(df['category'], df['outcome'])
chi2, p_value, dof, expected = stats.chi2_contingency(contingency_table)
print(f"Chi-square: {chi2:.4f}, P-value: {p_value:.4f}")

# Linear regression
from sklearn.linear_model import LinearRegression
X = df[['feature1', 'feature2']]
y = df['target']
model = LinearRegression()
model.fit(X, y)
print(f"R² Score: {model.score(X, y):.4f}")
print(f"Coefficients: {model.coef_}")
```

## Best Practices

### 1. **Reproducibility**
```python
# Set random seed
np.random.seed(42)

# Save processed data
df.to_csv('cleaned_data.csv', index=False)

# Document transformations
# Use Jupyter notebooks with markdown cells
```

### 2. **Data Quality Checks**
```python
def validate_data(df):
    """Validate data quality"""
    checks = {
        'null_values': df.isnull().sum().sum() == 0,
        'duplicates': df.duplicated().sum() == 0,
        'date_range_valid': df['date'].min() > pd.Timestamp('2000-01-01'),
        'no_negative_prices': (df['price'] >= 0).all()
    }
    return all(checks.values()), checks

is_valid, results = validate_data(df)
print(f"Data valid: {is_valid}")
print(results)
```

### 3. **Performance**
```python
# Use vectorized operations
df['result'] = df['a'] * df['b']  # ✅ Fast

# Avoid loops
for i in range(len(df)):  # ❌ Slow
    df.loc[i, 'result'] = df.loc[i, 'a'] * df.loc[i, 'b']

# Use appropriate data types
df['category'] = df['category'].astype('category')  # Saves memory

# Chunk large files
for chunk in pd.read_csv('large_file.csv', chunksize=10000):
    process(chunk)
```

## Analysis Workflow Template

```python
# 1. LOAD DATA
df = pd.read_csv('data.csv')

# 2. INITIAL INSPECTION
print(f"Shape: {df.shape}")
print(f"Columns: {df.columns.tolist()}")
print(f"Nulls: \n{df.isnull().sum()}")

# 3. CLEAN DATA
df = df.drop_duplicates()
df = df.dropna(subset=['critical_column'])
df['date'] = pd.to_datetime(df['date'])

# 4. TRANSFORM
df['month'] = df['date'].dt.month
df['total'] = df['quantity'] * df['price']

# 5. EXPLORE
print(df['category'].value_counts())
print(df[['sales', 'profit']].describe())

# 6. VISUALIZE
df.groupby('month')['sales'].sum().plot(kind='bar')
plt.title('Monthly Sales')
plt.show()

# 7. ANALYZE
monthly_avg = df.groupby('month')['sales'].mean()
print(f"Average monthly sales: ${monthly_avg.mean():.2f}")

# 8. EXPORT RESULTS
results = df.groupby('category').agg({
    'sales': 'sum',
    'profit': 'sum'
})
results.to_csv('results.csv')
```

## When to Use This Skill

- Analyzing CSV/Excel datasets
- Creating data visualizations
- Performing statistical analysis
- Cleaning and transforming data
- Generating data reports
- Exploratory data analysis

---

**Remember**: Good data analysis is methodical - inspect, clean, explore, analyze, visualize, report!
