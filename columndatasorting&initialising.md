---
title: Column Data Sorting & Initialising
parent: Result
nav_order: 1
url: https://skikszilcho.github.io/Distillation-Design-Analysis/Results/#Column-Data-Sorting
---

The [Antoine constants] are first defined based on the initialization of assumed and given data. Subsequently, necessary calculations are performed to gather the required information for the column design. Relevant Python modules are imported, and the compound data, along with any assumptions, are initialized. Using this foundational information, key parameters such as the vaporization fraction (`psi`), q-value (`q`), boil-up ratio (`RB`), and separation efficiency (`alpha`) are calculated. The results are then organized for enhanced clarity and readability.

```python
import sys
import random
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)
import math    #Various math expressions
import seaborn as sns
import matplotlib.pyplot as plt
from scipy.optimize import fsolve
from IPython.display import display
%matplotlib inline

# Compound information
eth_mw = 46.07   # units: kg/kmol
wtr_mw = 18.016  # units: kg/mol

# Column Information
feed = 600 # units: kmol/h
dist = 260 # units: kmol/h
botts = feed - dist # units: kmol/h
feed_P = 1.01 # units: bar
feed_T = 350 # units: K
T_liq = 780 # total liquid flowrate --> units: kmol/h
T_vap = 1040 # total vapour flowrate --> units: kmol/h
T_feed = T_liq + T_vap  # total feed flowrate in column --> units: kmol/h
RR = T_liq/dist  # The reflux ratio
psi = T_vap/T_feed  # fraction of vapourisation
q = 1 - psi   # The q-line value
RB = (T_vap-(1-q)*feed)/botts  # The boil-up ratio
temp_e_conc = 0.82076  # The taken temporary ethanol concentration in the distillate
alpha = Psat['ethanol'](feed_T)/Psat['water'](feed_T)  # Separation efficiency or relative volatility

names = ['Light substance','Heavy substance','Ethanol molecular weight','Water molecular weight',
         'Ethanol Vapour Pressure (bar)', 'Water Vapour Pressure (bar)', 'Feed Flow (kmol/h)', 'Distillate Flow (kmol/h)',
         'Bottoms Flow (kmol/h)','Total Column Liquid Flow (kmol/h)', 'Total Column Vapour Flow (kmol/h)',
         'Feed Temperature (K)','Column Operating Pressure (bar)', 'Vapourisation Fraction (\u03A8)',
         'Feed State/phase (q)', 'Reflux Ratio', 'Boil-up Ratio', 'Relative Volatility (\u03B1)',
         'Distillate Temporary Ethanol Conc.']
info_sep = ['Ethanol', 'Water', eth_mw, round(wtr_mw, 2), round(Psat['ethanol'](feed_T), 3), round(Psat['water'](feed_T), 3),
            feed, dist, botts, T_liq, T_vap, feed_T, feed_P, round(psi, 3), round(q, 3), RR, round(RB, 2),
           round(alpha, 2), temp_e_conc]

# Inputing calculated information in separate dataframe for improved readability
df_sep = pd.DataFrame(list(zip(names, info_sep)), columns = ['Variable', 'Value'])
```

  | Variable                           | Value   |
  |:-----------------------------------|:--------|
  | Light substance                    | Ethanol |
  | Heavy substance                    | Water   |
  | Ethanol molecular weight           | 46.07   |
  | Water molecular weight             | 18.02   |
  | Ethanol Vapour Pressure (bar)      | 0.955   |
  | Water Vapour Pressure (bar)        | 0.416   |
  | Feed Flow (kmol/h)                 | 600     |
  | Distillate Flow (kmol/h)           | 260     |
  | Bottoms Flow (kmol/h)              | 340     |
  | Total Column Liquid Flow (kmol/h)  | 780     |
  | Total Column Vapour Flow (kmol/h)  | 1040    |
  | Feed Temperature (K)               | 350     |
  | Column Operating Pressure (bar)    | 1.01    |
  | Vapourisation Fraction (Ψ)         | 0.571   |
  | Feed State/phase (q)               | 0.429   |
  | Reflux Ratio                       | 3.0     |
  | Boil-up Ratio                      | 2.05    |
  | Relative Volatility (α)            | 2.29    |
  | Distillate Temporary Ethanol Conc. | 0.82076 |

Using the `lambda`, `def`, and `dict()` python functions, an automation for the automatic calculation of saturated pressure at desired temperatures in the desired units for pressure for each component was created.

```python
Psat = dict()
# Define the Antoine constants for different temperature ranges
def Psat_ethanol(T):
    if T < 364:  # Adjust this range as needed
        # Constants for T range (K): 270-369-->P (bar)
        return (np.exp(18.5242 - 3578.91/(T + (-50.5)))) / 7.5006157584566 / 100
    elif 364 <= T < 514:
        # Constants for temperature between 300 and 513.91
        return (10**(4.92531 - 1432.526/(T + (-61.819))))  # from webbook website

Psat['ethanol'] = lambda T: Psat_ethanol(T)
Psat['water'] = lambda T: (np.exp(18.3036 -  3816.44/(T +  (-46.13))))/7.5006157584566/100    # T range (K): 284-441 --> P (bar)
E = 'ethanol'
W = 'water'
#feed_P = 1.01 # units: bar
#feed_T = 350 # units: K
```
