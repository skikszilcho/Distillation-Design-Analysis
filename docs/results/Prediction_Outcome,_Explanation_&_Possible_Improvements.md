---
title: "Prediction Outcome, Explanation & Possible Improvements"
parent: "result"
nav_order: 4
---

Predicting Ethanol Concentration Using a Random Forest Model
To analyze the relationship between column operating conditions and ethanol concentration, a Random Forest Regressor was trained using stage temperatures and key flowrate variables as input features.

The dataset includes measurements for temperatures at each stage of the distillation column, along with total liquid and vapor flow rates, distillate and bottoms flow rates, and feed flow rates. These features were selected based on their relevance to ethanol separation efficiency.

The workflow involves:

Defining Features and Target: The input features (X) include stage-wise temperature values and flowrate parameters, while the target variable (y) represents the ethanol concentration in the distillate.
Data Splitting: The dataset is divided into training and testing subsets using an 95%-5% split to ensure model generalization.
Training the Model: A Random Forest Regressor is used to capture nonlinear dependencies between input variables and ethanol concentration.
Feature Importance Analysis: After training, feature importance scores are extracted to determine which variables most significantly impact ethanol concentration predictions.
This approach provides valuable insights into the key factors affecting ethanol purity and can aid in process optimization by identifying the most influential operating conditions.







































































