---
title: Test Data Operating & Analysis
parent: Result
nav_order: 3
---

## Test Data Cleaning

The test data contained inconsistencies in the number of rows across different columns, as well as variations in some column names. To ensure proper data handling, these discrepancies were addressed when loading the `CSV` file. After importing the data, the column names and index were standardized to maintain consistency with the structure of the designed column data. This ensured uniformity in formatting and facilitated seamless data processing.

```python
columns = ['T3', 'T4', 'T5', 'T6', 'T7', 'T8', 'T9', 'T10', 'T11', 'T12', 'T13', 'T14',
          'L', 'V', 'D', 'B', 'F', 'Ethanol concentration']  # Adjusting columns based on irregular column sizes
# Load Dataframe from excel file at given location into pandas dataframe
filename = "/kaggle/input/distillation-column/dataset_distill.csv"        
#df_data_test = pd.read_csv(filename, names = columns, engine = 'python')
# Load DataFrame, ensuring proper parsing
# Load DataFrame, ensuring proper parsing
df_data_test = pd.read_csv(
    filename,
    names = columns,       # Define column names
    engine = 'python',     # Use Python engine for flexibility
    skiprows = 1,          # Skip the first row (header row, if needed)
    delimiter = ';',       # Specify the correct delimiter (adjust if it's not a comma)
    skipinitialspace = True,  # Handle spaces after delimiters
)
df_data_test.set_index('T3', inplace=True)
df_data_test.reset_index(inplace=True)

df_data_test.rename(columns = {'T3':'Stage 1',
                              'T4': 'Stage 2',
                              'T5':'Stage 3',
                              'T6':'Stage 4',
                              'T7':'Stage 5',
                              'T8':'Stage 6',
                              'T9':'Stage 7',
                              'T10':'Stage 8',
                              'T11':'Stage 9',
                              'T12':'Stage 10',
                              'T13':'Stage 11',
                              'T14':'Stage 12'}, inplace = True)

df_data_test.drop('Stage 11', axis = 1, inplace = True)
df_data_test.rename(columns = {'Stage 12':'Stage 11'}, inplace = True)


# Identify columns with non-numeric data types
non_numeric_columns = df_data_test.select_dtypes(include=['object']).columns

# Display the columns and sample values
for col in non_numeric_columns:
    print(f"Column: {col}")
    print(df_data_test[col].unique())
    print()

# Identify columns with non-numeric data types
non_numeric_columns = df_data_test.select_dtypes(include=['object']).columns

# Display the columns and sample values
for col in non_numeric_columns:
    df_data_test[col] = df_data_test[col].str.replace(',', '').astype(float)
    print(f"Column: {col}")
    print(df_data_test[col].unique())
    print()
```

<iframe src="Test_case_data.html" width="100%" height="400px"></iframe>

Then using `.describe()` python function on the test data to generate summary statistics, providing insights into key statistical measures. Since the size of the column the data was obtained from had a greater number of stages, the information obtained from the summary statistics was used along with the main research paper which used the dataset for soft-sensors validation and a new feature selection algorithm validation to help identify which columns exhibited greater consistency, allowing for a justified selection of stable features for further processing.


|       |    Stage 1 |    Stage 2 |    Stage 3 |    Stage 4 |    Stage 5 |    Stage 6 |    Stage 7 |    Stage 8 |    Stage 9 |   Stage 10 |   Stage 11 |              L |              V |        D |        B |        F |   Ethanol concentration |
|:------|-----------:|-----------:|-----------:|-----------:|-----------:|-----------:|-----------:|-----------:|-----------:|-----------:|-----------:|---------------:|---------------:|---------:|---------:|---------:|------------------------:|
| count | 4408       | 4408       | 4408       | 4408       | 4408       | 4408       | 4408       | 4408       | 4408       | 4408       | 4408       | 4408           | 4408           | 4408     | 4408     | 4408     |            4408         |
| mean  |  352.066   |  352.829   |  353.91    |  355.214   |  356.454   |  357.54    |  358.484   |  359.301   |  360.049   |  360.884   |  367.717   |    7.1992e+08  |    7.19919e+09 |  251.824 |  293.412 |  545.236 |               0.792205  |
| std   |    2.67119 |    3.78842 |    4.98645 |    6.25103 |    7.11985 |    7.62026 |    7.88216 |    7.97929 |    7.89919 |    7.60403 |    6.63942 |    2.88766e+09 |    2.88766e+10 |   71.048 |  108.551 |  119.193 |               0.0764407 |
| min   |  350.77    |  350.79    |  350.8     |  350.82    |  350.86    |  350.91    |  351       |  351.16    |  351.5     |  352.43    |  354.52    |   75           |  225           |  150     |   90     |  350     |               0.53819   |
| 25%   |  350.9     |  350.95    |  351       |  351.07    |  351.16    |  351.28    |  351.44    |  351.72    |  352.22    |  353.32    |  362.92    |  450           |  600           |  150     |  200     |  350     |               0.76189   |
| 50%   |  351.1     |  351.26    |  351.475   |  351.84    |  352.37    |  352.93    |  353.74    |  354.6     |  356.785   |  361.47    |  372.81    |  780           | 1040           |  260     |  320     |  600     |               0.81752   |
| 75%   |  351.822   |  352.41    |  353.74    |  356.868   |  361.64    |  364.75    |  366.88    |  367.42    |  367.522   |  367.91    |  373.01    | 1050           | 1400           |  260     |  352.5   |  600     |               0.85028   |
| max   |  364.97    |  368.6     |  369.06    |  372.57    |  372.97    |  373.01    |  373.01    |  373.01    |  373.01    |  373.01    |  373.07    |    1.23e+10    |    1.23e+11    |  350     |  450     |  650     |               0.89176   |


## Final Column Data Information

Since the test data included temperature measurements for each stage of the column, as well as total liquid and vapor flow rates, distillate and bottoms flow rates, feed flow rates, and the distillate ethanol concentration, the temperature profile for each stage of the designed column was calculated accordingly. To ensure consistency, the column data was structured in a format similar to the test dataset, allowing for a direct comparison and validation of the design.

```python
# Operating pressure in bar
feed_P_pa = feed_P * 100000  # Convert to Pascals

# Function to solve for temperature using Raoult's Law
def solve_temperature(x_ethanol, P):
    def equation(T):
        y_ethanol = x_ethanol * (Psat['ethanol'](T) * 100000) / P
        y_water = (1 - x_ethanol) * (Psat['water'](T) * 100000) / P
        return y_ethanol + y_water - 1  # Raoult's Law
    # Initial guess for temperature (in K)
    T_guess = 272.15 + 78  # Close to ethanol's boiling point
    return fsolve(equation, T_guess)[0]  # Return temperature in K

# Input data: x-y values for rectifying and stripping sections
x_y_rectifying = create_stairs_plot_D(xD, xq, yq, 'dct')
x_y_stripping = create_stairs_plot_B(xB, xq, yq, 'dct')

# Number of stages
#num_stages = len(list(create_stairs_plot_D(xD, xq, yq, 'dct').values())) 
#+ len(list(create_stairs_plot_B(xB, xq, yq, 'dct').values()))*0

# Calculate total number of stages
num_stages_rectifying = len(list(create_stairs_plot_D(xD, xq, yq, 'dct').values()))
num_stages_stripping = len(list(create_stairs_plot_B(xB, xq, yq, 'dct').values()))
total_stages = num_stages_rectifying + num_stages_stripping

# Initialize dictionary to store stage temperatures
stage_temperatures = {}

# Iterative procedure for all stages
for stage in range(1, total_stages + 1):
    if stage <= num_stages_rectifying:
        # Rectifying section
        x_y_rectifying = list(create_stairs_plot_D(xD, xq, yq, 'dct').keys())
        x_y_rectifying.sort(reverse = True)
        x_ethanol = x_y_rectifying[stage - 1]
    else:
        # Stripping section
        x_y_stripping = list(create_stairs_plot_B(xB, xq, yq, 'dct').keys())
        x_y_stripping.sort(reverse = True)
        x_ethanol = x_y_stripping[stage - num_stages_rectifying - 1]

    # Solve for temperature at the current stage
    T_stage = solve_temperature(x_ethanol, feed_P_pa)
    stage_temperatures[f"Stage {stage}"] = [T_stage]

# Convert to DataFrame
df_temperatures = pd.DataFrame(stage_temperatures)

# Rename columns to match stage numbers
#df_temperatures.columns = [f"Stage {stage}" for stage in range(1, num_stages + 1)]

df_col_cmp = df_temperatures.copy()
df_col_cmp .iloc[:, 0:11] = df_col_cmp .iloc[:, 0:11].round(2)
df_col_cmp['L'] =  T_liq
df_col_cmp["V"] = T_vap
df_col_cmp["D"] = dist
df_col_cmp["B"] = botts
df_col_cmp["F"] = feed
df_col_cmp["Ethanol concentration"] = eth_d_pty

# Display the resulting DataFrame
display(df_col_cmp)
```

|   Stage 1 |   Stage 2 |   Stage 3 |   Stage 4 |   Stage 5 |   Stage 6 |   Stage 7 |   Stage 8 |   Stage 9 |   Stage 10 |   Stage 11 |   L |    V |   D |   B |   F |   Ethanol concentration |
|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|-----------:|-----------:|----:|-----:|----:|----:|----:|------------------------:|
|    354.36 |    357.15 |    360.22 |    362.93 |    365.66 |    365.67 |    367.02 |    368.46 |    369.82 |     370.99 |     371.89 | 780 | 1040 | 260 | 340 | 600 |                   0.805 |

















