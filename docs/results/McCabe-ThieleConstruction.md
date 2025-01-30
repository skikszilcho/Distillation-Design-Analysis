---
title: "McCabe-Thiele Construction"
parent: "result"
nav_order: 2
---
## Initial Feed Flow

| Flow Type   |   Ethanol |   Water |   Total |
|:------------|----------:|--------:|--------:|
| Feed        |  222.76   | 377.24  |     600 |
| Distillate  |  213.4    |  46.602 |     260 |
| Bottoms     |    9.3659 | 330.63  |     340 |

After initializing and organizing the necessary column data, an arbitrary ethanol distillate concentration was selected from the dataset to calculate the initial feed flow information. To refine these calculations, Vapor-Liquid (VL) equilibrium relationships for binary mixtures and general flash equations were applied. These equations helped determine the final feed flow rates, ensuring consistency with the assumed separation conditions.

```python
# Using VL Equilibrium for binary mixtures and General Flash Equations
wtr_feed = (((1 - temp_e_conc)*(psi*(Psat['water'](feed_T)/feed_P - 1) + 1)/(Psat['water'](feed_T)/feed_P)) + 0.34)*feed
   # The flowrate of water in feed. To ensure a ~37:63 ethanol:water ratio, 0.34 is used --> units: kmol/h
eth_feed = feed - wtr_feed  # Ethanol flowrate in feed --> units: kmol/h

# Other flows
eth_dist = temp_e_conc*dist
wtr_dist = dist - eth_dist

eth_botts = eth_feed - eth_dist
wtr_botts = wtr_feed - wtr_dist

# Lists
ethanol = [round(eth_feed, 2), round(eth_dist, 2) , round(eth_botts, 4)]
water = [round(wtr_feed, 2), round(wtr_dist, 3), round(wtr_botts, 2)]
totals = [feed,dist,botts]
names = ['Feed','Distillate','Bottoms']

# Inputing calculated information in separate dataframe for improved readability
df_col = pd.DataFrame(list(zip(names, ethanol, water, totals)), columns = ['Flow Type', 'Ethanol', 'Water', 'Total'])


# For 80% ethanol purity in distillate
eth_d_pty = 0.805

n_eth_dist = eth_d_pty*dist
n_wtr_dist = dist - n_eth_dist

n_eth_botts = eth_feed - n_eth_dist
n_wtr_botts = wtr_feed - n_wtr_dist

# Lists
n_ethanol = [round(eth_feed, 2), round(n_eth_dist, 2) , round(n_eth_botts, 4)]
n_water = [round(wtr_feed, 2), round(n_wtr_dist, 3), round(n_wtr_botts, 2)]

# Inputing calculated information in separate dataframe for improved readability
df_n_col = pd.DataFrame(list(zip(names, n_ethanol, n_water, totals)), columns = ['Flow Type', 'Ethanol', 'Water', 'Total'])
```

## Final Feed Flow

| Flow Type   |   Ethanol |   Water |   Total |
|:------------|----------:|--------:|--------:|
| Feed        |  222.76   |  377.24 |     600 |
| Distillate  |  209.3    |   50.7  |     260 |
| Bottoms     |   13.4635 |  326.54 |     340 |


## McCabe-Thiele Construction Automation
The calculated data was used to apply the McCabe-Thiele method for distillation column design. To automate the calculations, Python’s `lambda` function was employed to `define` key equations for the rectifying (`TOL`) and stripping (`BOL`) operating lines, the q-line, the equilibrium curve, and the feed lines. Additionally, random ethanol liquid fraction values were generated and sorted using the `def`, `random`, and `sort` functions. These values were then mapped to their corresponding equilibrium, q-line, rectifying, and stripping line data points, allowing for a more systematic and automated analysis of the distillation process.
```python
# Equations & useful info for McCabe-Thiele Construction
xD = n_eth_dist/dist
zF = eth_feed/feed
xB = n_eth_botts/botts
Eqcurve = lambda x: ((alpha*x)/(1 + x*(alpha - 1)))
TOL = lambda x: ((RR/(RR + 1))*x + (1/(RR + 1))*xD)
BOL = lambda y: (RB/(RB + 1))*y + (1/(RB + 1))*xB
BOL_2 = lambda x: ((RB + 1)/RB)*x - (1/(RB))*xB
xq = ((1/(RR + 1))*xD + (1/(q - 1))*zF)/((q/(q - 1)) - (RR/(RR + 1)))
yq = (q/(q - 1))*xq - (1/(q - 1))*zF  # q-line/ Feed line

yq_2 = lambda x: (q/(q - 1))*x - (1/(q - 1))*zF  # q-line/ Feed line extra use

# This is to generate random numbers between 0 & 1 for the McCabe-Thiele and then
#   ensure specific values in this generated list
def generate_numbers(num_values, specific_numbers=None):
    if specific_numbers is None:
        specific_numbers = []
    
    # Generate random numbers
    random_numbers = [random.random() for _ in range(num_values - len(specific_numbers))]
    
    # Add specific numbers to the list
    numbers = random_numbers + specific_numbers
    
    # Sort the list
    numbers.sort()
    
    return numbers

# Use of the function generate_numbers
num_values = 19  # Total number of values to 
n_xB = xB + 0.0000005
spcfc_nums = [0, zF, xD, n_xB, xq, 1]  # Numbers to be in the result

gen_val = generate_numbers(num_values, spcfc_nums)
# print(gen_val)

# Function to create a dictionary with x values and corresponding y values
def create_xy_dict(x_values,function):
    # Calculate y values using the Eqcurve lambda function
    y_values = [function(x) for x in x_values]
    
    # Create a dictionary that maps x values to y values
    xy_dict = dict(zip(x_values, y_values))
    
    return xy_dict


eqcurve_dict = create_xy_dict(gen_val, Eqcurve)
tol_dict = create_xy_dict(gen_val, TOL)
bol_dict = create_xy_dict(gen_val, BOL_2)
q_dict = create_xy_dict(gen_val, yq_2)


df_bol = pd.DataFrame(list(zip(gen_val, bol_dict.values())), columns = ['x\u2099', 'BOL'])
df_bol = df_bol[(df_bol['x\u2099'] > xB) & (df_bol['x\u2099'] <= xq)]
ndf_bol_dict = {key: value for key, value in bol_dict.items() if xB < key <= xq}

df_tol = pd.DataFrame(list(zip(gen_val, tol_dict.values())), columns = ['x\u2099', 'TOL'])
df_tol = df_tol[(df_tol['x\u2099'] >= xq) & (df_tol['x\u2099'] < xD)]
ndf_tol_dict = {key: value for key, value in tol_dict.items() if (key <= xD) & (key >= xq)}

df_mct = pd.DataFrame(list(zip(gen_val, eqcurve_dict.values(), tol_dict.values(), bol_dict.values())), 
                     columns = ['x\u2099', 'EQ Curve', 'TOL', 'BOL'])

df_q_dict = {key: value for key, value in q_dict.items() if (key == xq) | (key == zF)}
```


|        xₙ |   EQ Curve |      TOL |         BOL |
|----------:|-----------:|---------:|------------:|
| 0         |  0         | 0.20125  | -0.0193124  |
| 0.011895  |  0.026877  | 0.210171 | -0.00161616 |
| 0.0358863 |  0.0786799 | 0.228165 |  0.0340758  |
| 0.039599  |  0.0864231 | 0.230949 |  0.0395993  |
| 0.12538   |  0.247497  | 0.295285 |  0.167216   |
| 0.174861  |  0.327145  | 0.332396 |  0.240829   |
| 0.221272  |  0.394644  | 0.367204 |  0.309876   |
| 0.298985  |  0.494574  | 0.425488 |  0.425488   |
| 0.371272  |  0.57534   | 0.479704 |  0.533032   |
| 0.384585  |  0.589114  | 0.489689 |  0.552837   |
| 0.457121  |  0.658923  | 0.544091 |  0.660749   |
| 0.464269  |  0.665359  | 0.549452 |  0.671383   |
| 0.625589  |  0.79311   | 0.670442 |  0.911379   |
| 0.774932  |  0.887635  | 0.782449 |  1.13356    |
| 0.805     |  0.904502  | 0.805    |  1.17829    |
| 0.807799  |  0.906039  | 0.807099 |  1.18245    |
| 0.82494   |  0.915337  | 0.819955 |  1.20796    |
| 0.863612  |  0.935599  | 0.848959 |  1.26549    |
| 1         |  1         | 0.95125  |  1.46839    |


## McCabe-Thiele Diagram

The automation code was extended by defining new functions to construct the stepwise stages (stairs) for both the rectifying and stripping sections of the McCabe-Thiele diagram. To enhance readability, the code also calculates and displays the total number of stages required for separation while clearly marking the feed stage location, ensuring a structured and intuitive visualization of the distillation process.

```python
def create_stairs_plot_B(xb, xq, yq, ui):
    bol_lst_x = list()
    bol_lst_y = list()
    bol_2 = dict()
    #tol_lst_x = list()
    #tol_lst_y = list()
    n = 0
    if n == 0:
        yb = Eqcurve(xb)
        bol_lst_x.append(xb)
        bol_lst_y.append(yb)
        #bol_2[xb] = yb
        #yd = TOL(xd)
        #tol_lst_x.append(xd)
        #tol_lst_y.append(yd)
        n += 1
    while yb <= yq:
        x = BOL(yb)
        yb = Eqcurve(x)
        if yb <= yq:
            bol_lst_x.append(x)
            bol_lst_y.append(yb)
            #bol_2[x] = yb
            n += 1
        else:
            bol_lst_x.append(x)
            #ybb = bol_lst_y[-1]
            bol_lst_y.append(yb)
            #bol_2[x] = ybb
            n =+ 1
    if ui == 'x_y':
        bolo = [bol_lst_x, bol_lst_y]
        return bolo
    elif ui == 'dct':
        n_2 = len(bol_lst_y)
        for i in range(n_2):
            bol_2[bol_lst_x[i]] = bol_lst_y[i]
        return bol_2
    elif ui == 'N':
        return n

# Similar stairs function as above but for rectifying section
def create_stairs_plot_D(xd, xq, yq, ui):
    tol_lst_x = list()
    tol_lst_y = list()
    tol_2 = dict()
    n = 0
    if n == 0:
        yd = TOL(xd)
        tol_lst_x.append(xd)
        tol_lst_y.append(yd)
        n += 1
    while yd >= yq:
        x = yd/(alpha - yd*(alpha - 1))
        yd = TOL(x)
        if yd >= yq:
            tol_lst_x.append(x)
            tol_lst_y.append(yd)
            n += 1
        else:
            tol_lst_x.append(x)
            ydd = tol_lst_y[-1]
            tol_lst_y.append(ydd)
            n =+ 1
    #return bol_lst_x, bol_lst_y, tol_lst_x, tol_lst_y
    if ui == 'x_y':
        #tolo = [tol_lst_x, tol_lst_y]
        return tol_lst_x, tol_lst_y
    elif ui == 'dct':
        n_2 = len(tol_lst_y)
        for i in range(n_2):
            tol_2[tol_lst_x[i]] = tol_lst_y[i]
        return tol_2
    elif ui == 'N':
        return n

# Create a figure and axis
fig, ax = plt.subplots(figsize = (10, 6))
ax.set_xlim(0, 1)
ax.set_ylim(0, 1)

# Adding the Equilibrium Curve & diagonal line
# 45-degree line range
x = np.linspace(0, 1, num_values)
y = x  # 45-degree line (y = x)
ax.plot(x, y, linestyle = '-', color = 'black', label = 'y = x line')

#for label, data in eqcurve_dict.items():
x = list(eqcurve_dict.keys())
y = list(eqcurve_dict.values())
ax.plot(x, y, linestyle = '-', color = 'orange', label = 'V-L EQ line')


# Adding BOL, TOL & q-lines
# BOL
#for label, data in ndf_bol_dict.items():
xb = list(ndf_bol_dict.keys())
yb = list(ndf_bol_dict.values())
ax.plot(xb, yb, linestyle = '-', color = 'deepskyblue', label = 'Stripping Section')
# TOL
#for label, data in ndf_tol_dict.items():
xd = list(ndf_tol_dict.keys())
yd = list(ndf_tol_dict.values())
ax.plot(xd, yd, linestyle = '-', color = 'green', label = 'Rectifying Section')
# q-line
x = list(df_q_dict.keys())
y = list(df_q_dict.values())
ax.plot(x, y, linestyle = '-', color = 'gold', label = 'q line')

# Create stairs plot using a step plot (each line represents a step)
#   Starting with Stripping Section
x = list(create_stairs_plot_B(xB, xq, yq, 'dct').keys())
y = list(create_stairs_plot_B(xB, xq, yq, 'dct').values())
ax.step(x, y, where = 'post', linestyle = '-', color = 'rebeccapurple', label = 'Stripping Section Trays')
#plt.step(x_values, y_values, where='post', label='McCabe-Thiele Steps', color='b')

#   Starting with Rectifying Section
x = list(create_stairs_plot_D(xD, xq, yq, 'dct').keys())
y = list(create_stairs_plot_D(xD, xq, yq, 'dct').values())
ax.step(x, y, where = 'post', linestyle = '-', color = 'rebeccapurple', label = 'Rectifying Section Trays')

# Extra for better improved
x = [xB, n_xB]
y = [Eqcurve(xB), BOL_2(n_xB)]
ax.plot(x, y, linestyle = '-', color = 'rebeccapurple', label = None)


ax.plot([xB,xB],[0,xB], linestyle = '--', color = 'slategray')
ax.scatter(xB,xB, color = 'lightsteelblue', zorder = 1.8)
ax.text(xB + 0.01,0.02,'$x_B$ = {:0.3f}'.format(float(xB)))

ax.plot([zF,zF,zF],[0,zF,zF], linestyle = '--', color = 'slategray')
ax.scatter([zF,zF],[zF,zF], color = 'lightsteelblue', zorder = 1.9)
ax.text(zF + 0.01,0.02,'$z_F$ = {:0.3f}'.format(float(zF)))

ax.plot([xD,xD],[0,xD], linestyle = '--', color = 'slategray')
ax.scatter(xD,xD, color = 'lightsteelblue', zorder = 1.9)
ax.text(xD - 0.11,0.02,'$x_D$ = {:0.3f}'.format(float(xD)))

#   Starting with Rectifying Section stage numbering
x = list(create_stairs_plot_D(xD, xq, yq, 'dct').keys())
y = list(create_stairs_plot_D(xD, xq, yq, 'dct').values())
ex = len(y)
for i in range(len(y) - 1):
    if i < len(y):
        n_x = y[i]/(alpha - y[i]*(alpha - 1))
        ax.scatter(n_x, y[i], color = 'plum', zorder = 1.9)
        ax.text(n_x - 0.06, y[i] + 0.01,'N = {:0.0f}'.format(float(i+1)))
    elif i == len(y):
        break
#   Starting with Stripping Section stage numbering
x = list(create_stairs_plot_B(xB, xq, yq, 'dct').keys())
y = list(create_stairs_plot_B(xB, xq, yq, 'dct').values())
y.sort(reverse = True)
for i in range(len(y)):
    if i < len(y):
        n_x = y[i]/(alpha - y[i]*(alpha - 1))
        ax.scatter(n_x, y[i], color = 'plum', zorder = 1.9)
        ax.text(n_x - 0.06, y[i] + 0.01,'N = {:0.0f}'.format(float(ex + i - 1)))
    elif i == len(y):
        break


plt.title('Ethanol-Water McCabe-Thiele Diagram)')
plt.xlabel('x = mol frac. of ethanol in liquid phase')
plt.ylabel('y = mol. frac. of ethanol in vapour phase')

# Show the plot
ax.legend()
ax.grid(True, linestyle='--', alpha = 0.7)
plt.show()
```
























