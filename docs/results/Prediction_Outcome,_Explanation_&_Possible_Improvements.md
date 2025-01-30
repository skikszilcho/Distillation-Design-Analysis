---
title: Prediction Outcome, Explanation & Possible Improvements
parent: Result
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

## Test Data & Predictions
<iframe src="X_test_data.html" width="100%" height="400px"></iframe>
Based on the test data used the model predictions are given as:
<iframe src="X_test_data_2.html" width="100%" height="400px"></iframe>

## Model Analysis

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np

# Mean Absolute Error (MAE)
mae = mean_absolute_error(y_test, y_pred)
print(f'Mean Absolute Error: {mae}')

# Mean Squared Error (MSE)
mse = mean_squared_error(y_test, y_pred)
print(f'Mean Squared Error: {mse}')

# Root Mean Squared Error (RMSE)
rmse = np.sqrt(mse)
print(f'Root Mean Squared Error: {rmse}')

# R² (Coefficient of Determination)
r2 = r2_score(y_test, y_pred)
print(f'R-squared: {r2}')

df_col_cmp_tst = df_col_cmp.iloc[:,:-1]
y_pred = rf.predict(df_col_cmp_tst)
df_col_cmp_2 = df_col_cmp.copy()
df_col_cmp_2['Predicted Ethanol concentration'] = y_pred
display(df_col_cmp_2)

# Mean Absolute Error (MAE)
mae = mean_absolute_error(df_col_cmp_2['Predicted Ethanol concentration'], df_col_cmp_2['Ethanol concentration'])
print(f'Mean Absolute Error: {mae}')

# Mean Squared Error (MSE)
mse = mean_squared_error(df_col_cmp_2['Predicted Ethanol concentration'], df_col_cmp_2['Ethanol concentration'])
print(f'Mean Squared Error: {mse}')

# Root Mean Squared Error (RMSE)
rmse = np.sqrt(mse)
print(f'Root Mean Squared Error: {rmse}')
```

The feature importance analysis is used to help identify which variables have the most significant impact on ethanol concentration predictions, identifying the more influential variables which would allow engineers to focus on key process parameters to improve ethanol recovery and purity. This analysis also helps in improving machine learning model performance by eliminating redundant or irrelevant variables and reducing overfitting. A statistical analysis was conducted between the model predictions and the actual test values. These include the mean absolute error `MAE`, mean squared error `MSE`, root mean squared error `RMSE`, and the coefficient of determination `R^2`.

| Variable   |          Value           |
|:-----------|-------------------------:|
| MAE        |  0.0006031714677405955   |
| MSE        |  1.3275055321942725e-06  |
| RMSE       |  0.0011521742629456155   |
| R^2        |  0.9997963086875911      |

Lower values indicate better model performance, while coefficient of determination values closer to 1 indicate a better fit by the model. Based on the observed values, the model performance is extremely good. The use of the model on the designed column provides the following results:

|   Stage 1 |   Stage 2 |   Stage 3 |   Stage 4 |   Stage 5 |   Stage 6 |   Stage 7 |   Stage 8 |   Stage 9 |   Stage 10 |   Stage 11 |   L |    V |   D |   B |   F |   Ethanol concentration |   Predicted Ethanol concentration |
|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|-----------:|-----------:|----:|-----:|----:|----:|----:|------------------------:|----------------------------------:|
|    354.36 |    357.15 |    360.22 |    362.93 |    365.66 |    365.67 |    367.02 |    368.46 |    369.82 |     370.99 |     371.89 | 780 | 1040 | 260 | 340 | 600 |                   0.805 |                          0.701458 |

The statistical analysis of predictions of the model on the designed column information is conducted. Based on the values statistical results, the model's predictions are off by about 0.1035 units from the actual ethanol concentration. This results in the squared differences between predicted and actual values to the value of 0.0107 with the `RMSE` indicating deviation between predicted and actual values.

| Variable   |          Value         |
|:-----------|-----------------------:|
| MAE        |  0.10354227500000013   |
| MSE        |  0.010721002712175652  |
| RMSE       |  0.10354227500000013   |

## Project Conclusion and Recommendations

The large differences in the errors of the model predictions on the training & test case to the actual designed column data indicate that the model is overfitting the data and may not generalise well to new, unseen data. The model can be improved through the design of a column that utilises a ethanol distillate concentration that does not fall outside outlier teritory. Outlier data in the used data could be handled using improved methods, since no outlier values handling techniques were utilised. Hyperparameters ( e.g. tree depth, number of trees) to improve upon model performane. A suggestion would be to further automate the code such that it takes and appropriately handles user input for distillate ethanol concentration. 




































































