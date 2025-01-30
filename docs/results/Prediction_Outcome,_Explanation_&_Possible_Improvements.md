---
title: "Prediction Outcome, Explanation & Possible Improvements"
parent: "result"
nav_order: 4
---

## Training Ethanol Distillate Concentration Prediction Model Using a Random Forest Model

A Random Forest Regressor was trained using stage temperatures and key flowrate variables as input features to analyze the relationship between column operating conditions and distillate ethanol concentration.

The temperatures at each stage of the distillation column, along with total liquid and vapor flow rates, distillate and bottoms flow rates, and feed flow rates in the dataset were selected as features based on their relevance to ethanol separation efficiency. This provides valuable insights into the key factors affecting ethanol purity and can aid in process optimization by identifying the most influential operating conditions.

The model set-up and training involves:

  [Defining Features and Target:] The input features `X` include stage-wise temperature values and flowrate parameters, while the target variable `y` represents the ethanol concentration in the distillate.
  
  [Data Splitting:] The dataset is divided into training and testing subsets using an 95%-5% split to ensure model generalization.
  
  [Training the Model:] A Random Forest Regressor is used to capture nonlinear dependencies between input variables and ethanol concentration.
  
  [Feature Importance Analysis:] After training, feature importance scores are extracted to determine which variables most significantly impact ethanol concentration predictions.

  ```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split

# Define features and target
X = df_data_test[['Stage 1', 'Stage 2', 'Stage 3', 'Stage 4',
        'Stage 5', 'Stage 6', 'Stage 7', 'Stage 8', 'Stage 9',
                 'Stage 10', 'Stage 11', 'L', 'V', 'D', 'B', 'F']]
y = df_data_test['Ethanol concentration']

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.05, random_state = 42)

# Train Random Forest
rf = RandomForestRegressor(random_state = 42)
rf.fit(X_train, y_train)

# Feature importance
feature_importance = pd.DataFrame({
    'Variable': X.columns,
    'Importance': rf.feature_importances_
}).sort_values(by='Importance', ascending = False)

print(feature_importance)
```

| Variable   |   Importance |
|:-----------|-------------:|
| Stage 1    |  0.974168    |
| B          |  0.00523747  |
| Stage 2    |  0.00435971  |
| Stage 9    |  0.00207201  |
| Stage 11   |  0.00194979  |
| Stage 8    |  0.00175818  |
| L          |  0.00153181  |
| V          |  0.00152963  |
| Stage 3    |  0.001473    |
| Stage 5    |  0.00132003  |
| Stage 7    |  0.000996146 |
| Stage 6    |  0.0009147   |
| Stage 4    |  0.000819819 |
| F          |  0.000807538 |
| Stage 10   |  0.000775016 |
| D          |  0.000286951 |










































































