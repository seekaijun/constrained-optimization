## Constrained Optimization (Energy Efficiency on Buildings)

### Dataset Information
The dataset is obtained from UCI Machine Learning Repository: https://archive.ics.uci.edu/ml/datasets/Energy+efficiency

The optimization objective is to minimize the heat energy needed for maintaining the temperature in building, with similar heating load and cooling load.

*Note:
<br>
1) Heating load is the amount of heat energy that would need to be added to a space to maintain the temperature in an acceptable range.

2) Cooling load is the amount of heat energy that would need to be removed from a space (cooling) to maintain the temperature in an acceptable range.

### Variable Information
1) X1: Relative Compactness

2) X2: Surface Area - m²

3) X3: Wall Area - m²

4) X4: Roof Area - m²

5) X5: Overall Height - m

6) X6: Orientation - 2:North, 3:East, 4:South, 5:West

7) X7: Glazing Area - 0%, 10%, 25%, 40% (of floor area)

8) X8: Glazing Area Distribution (Variance) - 1:Uniform, 2:North, 3:East, 4:South, 5:West

9) Y1: Heating Load - kWh/m²

10) Y2: Cooling Load - kWh/m²


```python
import pandas as pd
import statsmodels.formula.api as smf

decimal = 4
```


```python
# Load excel data into dataframe
df_init = pd.read_excel('../data/ENB2012_data.xlsx')

display(df_init[:5])
print('There\'re %d rows and %d variables in dataframe.' % (df_init.shape))
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X6</th>
      <th>X7</th>
      <th>X8</th>
      <th>Y1</th>
      <th>Y2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>2</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>3</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>4</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>5</td>
      <td>0.0</td>
      <td>0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>2</td>
      <td>0.0</td>
      <td>0</td>
      <td>20.84</td>
      <td>28.28</td>
    </tr>
  </tbody>
</table>
</div>


    There're 768 rows and 10 variables in dataframe.
    


```python
# Drop categorical fields
df = df_init.drop(columns=['X6', 'X8'])
X_var = ['X1', 'X2', 'X3', 'X4', 'X5', 'X7']

display(df[:5])
print('There\'re %d rows and %d variables in dataframe.' % (df.shape))
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>Y1</th>
      <th>Y2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>20.84</td>
      <td>28.28</td>
    </tr>
  </tbody>
</table>
</div>


    There're 768 rows and 8 variables in dataframe.
    


```python
# Create fitted model (ordinary least squares regression) for variable Y1
lm1 = smf.ols(formula='Y1 ~ X1 + X2 + X3 + X4 + X5 + X7', data=df).fit()

display(lm1.summary())
```


<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>           <td>Y1</td>        <th>  R-squared:         </th> <td>   0.915</td>
</tr>
<tr>
  <th>Model:</th>                   <td>OLS</td>       <th>  Adj. R-squared:    </th> <td>   0.915</td>
</tr>
<tr>
  <th>Method:</th>             <td>Least Squares</td>  <th>  F-statistic:       </th> <td>   1646.</td>
</tr>
<tr>
  <th>Date:</th>             <td>Tue, 21 Jul 2020</td> <th>  Prob (F-statistic):</th>  <td>  0.00</td> 
</tr>
<tr>
  <th>Time:</th>                 <td>21:38:33</td>     <th>  Log-Likelihood:    </th> <td> -1916.8</td>
</tr>
<tr>
  <th>No. Observations:</th>      <td>   768</td>      <th>  AIC:               </th> <td>   3846.</td>
</tr>
<tr>
  <th>Df Residuals:</th>          <td>   762</td>      <th>  BIC:               </th> <td>   3873.</td>
</tr>
<tr>
  <th>Df Model:</th>              <td>     5</td>      <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>      <td>nonrobust</td>    <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>         <th>coef</th>     <th>std err</th>      <th>t</th>      <th>P>|t|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>Intercept</th> <td>   84.3865</td> <td>   19.112</td> <td>    4.415</td> <td> 0.000</td> <td>   46.869</td> <td>  121.904</td>
</tr>
<tr>
  <th>X1</th>        <td>  -64.7734</td> <td>   10.334</td> <td>   -6.268</td> <td> 0.000</td> <td>  -85.059</td> <td>  -44.488</td>
</tr>
<tr>
  <th>X2</th>        <td>   -0.0626</td> <td>    0.013</td> <td>   -4.650</td> <td> 0.000</td> <td>   -0.089</td> <td>   -0.036</td>
</tr>
<tr>
  <th>X3</th>        <td>    0.0361</td> <td>    0.004</td> <td>    9.346</td> <td> 0.000</td> <td>    0.029</td> <td>    0.044</td>
</tr>
<tr>
  <th>X4</th>        <td>   -0.0494</td> <td>    0.008</td> <td>   -6.540</td> <td> 0.000</td> <td>   -0.064</td> <td>   -0.035</td>
</tr>
<tr>
  <th>X5</th>        <td>    4.1700</td> <td>    0.339</td> <td>   12.285</td> <td> 0.000</td> <td>    3.504</td> <td>    4.836</td>
</tr>
<tr>
  <th>X7</th>        <td>   20.4380</td> <td>    0.799</td> <td>   25.588</td> <td> 0.000</td> <td>   18.870</td> <td>   22.006</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td>20.756</td> <th>  Durbin-Watson:     </th> <td>   0.646</td>
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.000</td> <th>  Jarque-Bera (JB):  </th> <td>  44.998</td>
</tr>
<tr>
  <th>Skew:</th>          <td>-0.002</td> <th>  Prob(JB):          </th> <td>1.69e-10</td>
</tr>
<tr>
  <th>Kurtosis:</th>      <td> 4.186</td> <th>  Cond. No.          </th> <td>2.95e+16</td>
</tr>
</table><br/><br/>Warnings:<br/>[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.<br/>[2] The smallest eigenvalue is 5.23e-25. This might indicate that there are<br/>strong multicollinearity problems or that the design matrix is singular.



```python
# Construct linear regression formula for variable Y1
lm1_formula = f'Y1 = {round(lm1.params[0], decimal)}'

for var in X_var:
    if lm1.params[var] < 0:
        lm1_formula += f' - {abs(round(lm1.params[var], decimal))} {var}'
    else:
        lm1_formula += f' + {round(lm1.params[var], decimal)} {var}'

print('Linear regression formula for Y1:')
print(lm1_formula)
```

    Linear regression formula for Y1:
    Y1 = 84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7
    


```python
# Create fitted model (ordinary least squares regression) for variable Y2
lm2 = smf.ols(formula='Y2 ~ X1 + X2 + X3 + X4 + X5 + X7', data=df).fit()

display(lm2.summary())
```


<table class="simpletable">
<caption>OLS Regression Results</caption>
<tr>
  <th>Dep. Variable:</th>           <td>Y2</td>        <th>  R-squared:         </th> <td>   0.888</td>
</tr>
<tr>
  <th>Model:</th>                   <td>OLS</td>       <th>  Adj. R-squared:    </th> <td>   0.887</td>
</tr>
<tr>
  <th>Method:</th>             <td>Least Squares</td>  <th>  F-statistic:       </th> <td>   1203.</td>
</tr>
<tr>
  <th>Date:</th>             <td>Tue, 21 Jul 2020</td> <th>  Prob (F-statistic):</th>  <td>  0.00</td> 
</tr>
<tr>
  <th>Time:</th>                 <td>21:38:40</td>     <th>  Log-Likelihood:    </th> <td> -1980.2</td>
</tr>
<tr>
  <th>No. Observations:</th>      <td>   768</td>      <th>  AIC:               </th> <td>   3972.</td>
</tr>
<tr>
  <th>Df Residuals:</th>          <td>   762</td>      <th>  BIC:               </th> <td>   4000.</td>
</tr>
<tr>
  <th>Df Model:</th>              <td>     5</td>      <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>      <td>nonrobust</td>    <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>         <th>coef</th>     <th>std err</th>      <th>t</th>      <th>P>|t|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>Intercept</th> <td>   97.7618</td> <td>   20.756</td> <td>    4.710</td> <td> 0.000</td> <td>   57.015</td> <td>  138.508</td>
</tr>
<tr>
  <th>X1</th>        <td>  -70.7877</td> <td>   11.223</td> <td>   -6.307</td> <td> 0.000</td> <td>  -92.819</td> <td>  -48.756</td>
</tr>
<tr>
  <th>X2</th>        <td>   -0.0661</td> <td>    0.015</td> <td>   -4.520</td> <td> 0.000</td> <td>   -0.095</td> <td>   -0.037</td>
</tr>
<tr>
  <th>X3</th>        <td>    0.0225</td> <td>    0.004</td> <td>    5.366</td> <td> 0.000</td> <td>    0.014</td> <td>    0.031</td>
</tr>
<tr>
  <th>X4</th>        <td>   -0.0443</td> <td>    0.008</td> <td>   -5.405</td> <td> 0.000</td> <td>   -0.060</td> <td>   -0.028</td>
</tr>
<tr>
  <th>X5</th>        <td>    4.2838</td> <td>    0.369</td> <td>   11.620</td> <td> 0.000</td> <td>    3.560</td> <td>    5.008</td>
</tr>
<tr>
  <th>X7</th>        <td>   14.8180</td> <td>    0.867</td> <td>   17.082</td> <td> 0.000</td> <td>   13.115</td> <td>   16.521</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Omnibus:</th>       <td>104.896</td> <th>  Durbin-Watson:     </th> <td>   1.095</td>
</tr>
<tr>
  <th>Prob(Omnibus):</th> <td> 0.000</td>  <th>  Jarque-Bera (JB):  </th> <td> 232.225</td>
</tr>
<tr>
  <th>Skew:</th>          <td> 0.766</td>  <th>  Prob(JB):          </th> <td>3.74e-51</td>
</tr>
<tr>
  <th>Kurtosis:</th>      <td> 5.215</td>  <th>  Cond. No.          </th> <td>2.95e+16</td>
</tr>
</table><br/><br/>Warnings:<br/>[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.<br/>[2] The smallest eigenvalue is 5.23e-25. This might indicate that there are<br/>strong multicollinearity problems or that the design matrix is singular.



```python
# Construct linear regression formula for variable Y2
lm2_formula = f'Y2 = {round(lm2.params[0], decimal)}'

for var in X_var:
    if lm2.params[var] < 0:
        lm2_formula += f' - {abs(round(lm2.params[var], decimal))} {var}'
    else:
        lm2_formula += f' + {round(lm2.params[var], decimal)} {var}'

print('Linear regression formula for Y2:')
print(lm2_formula)
```

    Linear regression formula for Y2:
    Y2 = 97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7
    


```python
# Calculate difference between X2 and (X3 + X4), X3 and X4, Y1 and Y2
df['X2 - (X3 + X4)'] = round(df['X2'] - (df['X3'] + df['X4']), decimal)
df['X3 - X4'] = round(df['X3'] - df['X4'], decimal)
df['|Y1 - Y2|'] = abs(round(df['Y1'] - df['Y2'], decimal))
df['Avg(Y1, Y2)'] = round(df[['Y1', 'Y2']].mean(axis=1), decimal)

display(df[:5])
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>Y1</th>
      <th>Y2</th>
      <th>X2 - (X3 + X4)</th>
      <th>X3 - X4</th>
      <th>|Y1 - Y2|</th>
      <th>Avg(Y1, Y2)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
      <td>110.25</td>
      <td>183.75</td>
      <td>5.78</td>
      <td>18.44</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
      <td>110.25</td>
      <td>183.75</td>
      <td>5.78</td>
      <td>18.44</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
      <td>110.25</td>
      <td>183.75</td>
      <td>5.78</td>
      <td>18.44</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.98</td>
      <td>514.5</td>
      <td>294.0</td>
      <td>110.25</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>15.55</td>
      <td>21.33</td>
      <td>110.25</td>
      <td>183.75</td>
      <td>5.78</td>
      <td>18.44</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.90</td>
      <td>563.5</td>
      <td>318.5</td>
      <td>122.50</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>20.84</td>
      <td>28.28</td>
      <td>122.50</td>
      <td>196.00</td>
      <td>7.44</td>
      <td>24.56</td>
    </tr>
  </tbody>
</table>
</div>



```python
# Display descriptive statistics of variables
display(df.describe())
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>Y1</th>
      <th>Y2</th>
      <th>X2 - (X3 + X4)</th>
      <th>X3 - X4</th>
      <th>|Y1 - Y2|</th>
      <th>Avg(Y1, Y2)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.00000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
      <td>768.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.764167</td>
      <td>671.708333</td>
      <td>318.500000</td>
      <td>176.604167</td>
      <td>5.25000</td>
      <td>0.234375</td>
      <td>22.307195</td>
      <td>24.587760</td>
      <td>176.604167</td>
      <td>141.895833</td>
      <td>2.677622</td>
      <td>23.447478</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.105777</td>
      <td>88.086116</td>
      <td>43.626481</td>
      <td>45.165950</td>
      <td>1.75114</td>
      <td>0.133221</td>
      <td>10.090204</td>
      <td>9.513306</td>
      <td>45.165950</td>
      <td>71.380754</td>
      <td>1.730804</td>
      <td>9.742477</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.620000</td>
      <td>514.500000</td>
      <td>245.000000</td>
      <td>110.250000</td>
      <td>3.50000</td>
      <td>0.000000</td>
      <td>6.010000</td>
      <td>10.900000</td>
      <td>110.250000</td>
      <td>24.500000</td>
      <td>0.000000</td>
      <td>8.475000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.682500</td>
      <td>606.375000</td>
      <td>294.000000</td>
      <td>140.875000</td>
      <td>3.50000</td>
      <td>0.100000</td>
      <td>12.992500</td>
      <td>15.620000</td>
      <td>140.875000</td>
      <td>91.875000</td>
      <td>1.280000</td>
      <td>14.375000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.750000</td>
      <td>673.750000</td>
      <td>318.500000</td>
      <td>183.750000</td>
      <td>5.25000</td>
      <td>0.250000</td>
      <td>18.950000</td>
      <td>22.080000</td>
      <td>183.750000</td>
      <td>147.000000</td>
      <td>2.660000</td>
      <td>20.485000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.830000</td>
      <td>741.125000</td>
      <td>343.000000</td>
      <td>220.500000</td>
      <td>7.00000</td>
      <td>0.400000</td>
      <td>31.667500</td>
      <td>33.132500</td>
      <td>220.500000</td>
      <td>186.812500</td>
      <td>3.480000</td>
      <td>32.167500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>0.980000</td>
      <td>808.500000</td>
      <td>416.500000</td>
      <td>220.500000</td>
      <td>7.00000</td>
      <td>0.400000</td>
      <td>43.100000</td>
      <td>48.030000</td>
      <td>220.500000</td>
      <td>294.000000</td>
      <td>10.690000</td>
      <td>44.975000</td>
    </tr>
  </tbody>
</table>
</div>



```python
# Construct constraints based on variable sign in formula and probability distribution of variables
print('Linear regression formula for Y1 and Y2:')
print('1) ' + lm1_formula)
print('2) ' + lm2_formula)

print('\nAs this is a minimization problem of Y1 and Y2,')
print('i) Use <= sign for variables with negative coefficient in formula.')
print('ii) Use >= sign for variables with positive coefficient in formula.')

print('\nFor example,')
print(f'The larger the value of X1 (coefficient = {round(lm1.params["X1"], decimal)}), the smaller the value of Y1.')
print('Hence constraint for X1 is using <= sign to limit its value.')

print('\nBased on probability distribution of variables,')
print('iii) Use maximum value for constraint from i).')
print('iv) Use minimum value for constraint from ii).')
print('v) Use median value for constraint involving two or more variables.')

print('\nConstraints:')
X1_upper_constr = df['X1'].max()
print(f'3) X1 <= {X1_upper_constr}')
X2_upper_constr = df['X2'].max()
print(f'4) X2 <= {X2_upper_constr}')
X5_lower_constr = df['X5'].min()
print(f'5) X5 >= {X5_lower_constr}')
X7_lower_constr = df['X7'].min()
print(f'6) X7 >= {X7_lower_constr}')

print('\nAs X2 (Surface Area) is always greater than sum of X3 (Wall Area) and X4 (Roof Area),')
X2X3X4_constr = df['X2 - (X3 + X4)'].median()
print(f'7) X3 + X4 + {X2X3X4_constr} <= X2')

print('\nAs X3 (Wall Area) is always greater than X4 (Roof Area),')
X3X4_constr = df['X3 - X4'].median()
print(f'8) X4 + {X3X4_constr} <= X3')

print('\nAs value of Y1 and Y2 needs to be similar to each other,')
Y1Y2_constr = df['|Y1 - Y2|'].median()
print(f'9) Y1 - Y2 <= {Y1Y2_constr}')
print(f'10) Y2 - Y1 <= {Y1Y2_constr}')

print('\nOther constraints with their minimum or maximum:')
X4_lower_constr = df['X4'].min()
print(f'11) X4 >= {X4_lower_constr}')
X5_upper_constr = df['X5'].max()
print(f'12) X5 <= {X5_upper_constr}')
X7_upper_constr = df['X7'].max()
print(f'13) X7 <= {X7_upper_constr}')
Y1Y2_lower_constr = df['Avg(Y1, Y2)'].min()
print(f'14) Y1, Y2 >= {Y1Y2_lower_constr}')
print('15) X1 >= 0')
```

    Linear regression formula for Y1 and Y2:
    1) Y1 = 84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7
    2) Y2 = 97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7
    
    As this is a minimization problem of Y1 and Y2,
    i) Use <= sign for variables with negative coefficient in formula.
    ii) Use >= sign for variables with positive coefficient in formula.
    
    For example,
    The larger the value of X1 (coefficient = -64.7734), the smaller the value of Y1.
    Hence constraint for X1 is using <= sign to limit its value.
    
    Based on probability distribution of variables,
    iii) Use maximum value for constraint from i).
    iv) Use minimum value for constraint from ii).
    v) Use median value for constraint involving two or more variables.
    
    Constraints:
    3) X1 <= 0.98
    4) X2 <= 808.5
    5) X5 >= 3.5
    6) X7 >= 0.0
    
    As X2 (Surface Area) is always greater than sum of X3 (Wall Area) and X4 (Roof Area),
    7) X3 + X4 + 183.75 <= X2
    
    As X3 (Wall Area) is always greater than X4 (Roof Area),
    8) X4 + 147.0 <= X3
    
    As value of Y1 and Y2 needs to be similar to each other,
    9) Y1 - Y2 <= 2.66
    10) Y2 - Y1 <= 2.66
    
    Other constraints with their minimum or maximum:
    11) X4 >= 110.25
    12) X5 <= 7.0
    13) X7 <= 0.4
    14) Y1, Y2 >= 8.475
    15) X1 >= 0
    


```python
# Optimization problem
print('Minimize Z = Y1 + Y2, subject to,')
print('1) ' + lm1_formula)
print('2) ' + lm2_formula)
print(f'3) X1 <= {X1_upper_constr}')
print(f'4) X2 <= {X2_upper_constr}')
print(f'5) X5 >= {X5_lower_constr}')
print(f'6) X7 >= {X7_lower_constr}')
print(f'7) X3 + X4 + {X2X3X4_constr} <= X2')
print(f'8) X4 + {X3X4_constr} <= X3')
print(f'9) Y1 - Y2 <= {Y1Y2_constr}')
print(f'10) Y2 - Y1 <= {Y1Y2_constr}')
print(f'11) X4 >= {X4_lower_constr}')
print(f'12) X5 <= {X5_upper_constr}')
print(f'13) X7 <= {X7_upper_constr}')
print(f'14) Y1, Y2 >= {Y1Y2_lower_constr}')
print('15) X1 >= 0')
```

    Minimize Z = Y1 + Y2, subject to,
    1) Y1 = 84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7
    2) Y2 = 97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7
    3) X1 <= 0.98
    4) X2 <= 808.5
    5) X5 >= 3.5
    6) X7 >= 0.0
    7) X3 + X4 + 183.75 <= X2
    8) X4 + 147.0 <= X3
    9) Y1 - Y2 <= 2.66
    10) Y2 - Y1 <= 2.66
    11) X4 >= 110.25
    12) X5 <= 7.0
    13) X7 <= 0.4
    14) Y1, Y2 >= 8.475
    15) X1 >= 0
    


```python
# Convert the objective function and constraints with respect to variable Y1 and Y2
print(lm1_formula)
print(lm2_formula)


print('\nConverting objective function:')
print('Z = Y1 + Y2')
print('Z = (' + lm1_formula[5:] + ') + (' + lm2_formula[5:] + ')')

# Calculate Y1 + Y2
Z_intercept = round(round(lm1.params[0], decimal) + round(lm2.params[0], decimal), decimal)
X1_coef = round(round(lm1.params['X1'], decimal) + round(lm2.params['X1'], decimal), decimal)
X2_coef = round(round(lm1.params['X2'], decimal) + round(lm2.params['X2'], decimal), decimal)
X3_coef = round(round(lm1.params['X3'], decimal) + round(lm2.params['X3'], decimal), decimal)
X4_coef = round(round(lm1.params['X4'], decimal) + round(lm2.params['X4'], decimal), decimal)
X5_coef = round(round(lm1.params['X5'], decimal) + round(lm2.params['X5'], decimal), decimal)
X7_coef = round(round(lm1.params['X7'], decimal) + round(lm2.params['X7'], decimal), decimal)
X_var_coef = [X1_coef, X2_coef, X3_coef, X4_coef, X5_coef, X7_coef]

lm1_plus_lm2 = f'{Z_intercept}'
for var, coef in dict(zip(X_var, X_var_coef)).items():
    if coef < 0:
        lm1_plus_lm2 += f' - {abs(coef)} {var}'
    else:
        lm1_plus_lm2 += f' + {coef} {var}'

print('Z = ' + lm1_plus_lm2)


print('\nConverting constraint:')
print(f'Y1 - Y2 <= {Y1Y2_constr}')
print('(' + lm1_formula[5:] + ') - (' + lm2_formula[5:] + f') <= {Y1Y2_constr}')

# Calculate Y1 - Y2
lm1_minus_lm2 = f'{round(round(lm1.params[0], decimal) - round(lm2.params[0], decimal), decimal)}'
for var in X_var:
    if round(lm1.params[var], decimal) - round(lm2.params[var], decimal) < 0:
        lm1_minus_lm2 += ' - ' + \
            f'{abs(round(round(lm1.params[var], decimal) - round(lm2.params[var], decimal), decimal))} {var}'
    else:
        lm1_minus_lm2 += ' + ' + \
            f'{round(round(lm1.params[var], decimal) - round(lm2.params[var], decimal), decimal)} {var}'

print(lm1_minus_lm2 + f' <= {Y1Y2_constr}')
print(lm1_minus_lm2[7 + decimal:] + ' <= ' + \
      f'{round(Y1Y2_constr + -round(round(lm1.params[0], decimal) - round(lm2.params[0], decimal), decimal), decimal)}')


print('\nConverting constraint:')
print(f'Y2 - Y1 <= {Y1Y2_constr}')
print('(' + lm2_formula[5:] + ') - (' + lm1_formula[5:] + f') <= {Y1Y2_constr}')

# Calculate Y2 - Y1
lm2_minus_lm1 = f'{round(round(lm2.params[0], decimal) - round(lm1.params[0], decimal), decimal)}'
for var in X_var:
    if round(lm2.params[var], decimal) - round(lm1.params[var], decimal) < 0:
        lm2_minus_lm1 += ' - ' + \
            f'{abs(round(round(lm2.params[var], decimal) - round(lm1.params[var], decimal), decimal))} {var}'
    else:
        lm2_minus_lm1 += ' + ' + \
            f'{round(round(lm2.params[var], decimal) - round(lm1.params[var], decimal), decimal)} {var}'

print(lm2_minus_lm1 + f' <= {Y1Y2_constr}')
print(lm2_minus_lm1[4 + decimal:] + ' <= ' + \
      f'{round(Y1Y2_constr + -round(round(lm2.params[0], decimal) - round(lm1.params[0], decimal), decimal), decimal)}')


print('\nConverting constraint:')
print(f'Y1 >= {Y1Y2_lower_constr}')
print(lm1_formula[5:] + f' >= {Y1Y2_lower_constr}')
print(lm1_formula[9 + decimal:] + f' >= {Y1Y2_lower_constr + -round(lm1.params[0], decimal)}')


print('\nConverting constraint:')
print(f'Y2 >= {Y1Y2_lower_constr}')
print(lm2_formula[5:] + f' >= {Y1Y2_lower_constr}')
print(lm2_formula[9 + decimal:] + f' >= {Y1Y2_lower_constr + -round(lm2.params[0], decimal)}')
```

    Y1 = 84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7
    Y2 = 97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7
    
    Converting objective function:
    Z = Y1 + Y2
    Z = (84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7) + (97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7)
    Z = 182.1483 - 135.5611 X1 - 0.1287 X2 + 0.0586 X3 - 0.0937 X4 + 8.4538 X5 + 35.256 X7
    
    Converting constraint:
    Y1 - Y2 <= 2.66
    (84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7) - (97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7) <= 2.66
    -13.3753 + 6.0143 X1 + 0.0035 X2 + 0.0136 X3 - 0.0051 X4 - 0.1138 X5 + 5.62 X7 <= 2.66
    6.0143 X1 + 0.0035 X2 + 0.0136 X3 - 0.0051 X4 - 0.1138 X5 + 5.62 X7 <= 16.0353
    
    Converting constraint:
    Y2 - Y1 <= 2.66
    (97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7) - (84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7) <= 2.66
    13.3753 - 6.0143 X1 - 0.0035 X2 - 0.0136 X3 + 0.0051 X4 + 0.1138 X5 - 5.62 X7 <= 2.66
    - 6.0143 X1 - 0.0035 X2 - 0.0136 X3 + 0.0051 X4 + 0.1138 X5 - 5.62 X7 <= -10.7153
    
    Converting constraint:
    Y1 >= 8.475
    84.3865 - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7 >= 8.475
    - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7 >= -75.9115
    
    Converting constraint:
    Y2 >= 8.475
    97.7618 - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7 >= 8.475
    - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7 >= -89.2868
    


```python
# Optimization problem after variable Y1 and Y2 conversion
print(f'Minimize Z = ' + lm1_plus_lm2 + ', subject to,')
print(f'1) X1 <= {X1_upper_constr}')
print(f'2) X2 <= {X2_upper_constr}')
print(f'3) X5 >= {X5_lower_constr}')
print(f'4) -X2 + X3 + X4 <= {-X2X3X4_constr}')
print(f'5) -X3 + X4 <= {-X3X4_constr}')
print('6) ' + lm1_minus_lm2[7 + decimal:] + ' <= ' + \
      f'{round(Y1Y2_constr + -round(round(lm1.params[0], decimal) - round(lm2.params[0], decimal), decimal), decimal)}')
print('7) ' + lm2_minus_lm1[4 + decimal:] + ' <= ' + \
      f'{round(Y1Y2_constr + -round(round(lm2.params[0], decimal) - round(lm1.params[0], decimal), decimal), decimal)}')
print(f'8) X4 >= {X4_lower_constr}')
print(f'9) X5 <= {X5_upper_constr}')
print(f'10) X7 <= {X7_upper_constr}')
print('11) ' + lm1_formula[9 + decimal:] + f' >= {Y1Y2_lower_constr + -round(lm1.params[0], decimal)}')
print('12) ' + lm2_formula[9 + decimal:] + f' >= {Y1Y2_lower_constr + -round(lm2.params[0], decimal)}')
print('13) X1, X7 >= 0')
```

    Minimize Z = 182.1483 - 135.5611 X1 - 0.1287 X2 + 0.0586 X3 - 0.0937 X4 + 8.4538 X5 + 35.256 X7, subject to,
    1) X1 <= 0.98
    2) X2 <= 808.5
    3) X5 >= 3.5
    4) -X2 + X3 + X4 <= -183.75
    5) -X3 + X4 <= -147.0
    6) 6.0143 X1 + 0.0035 X2 + 0.0136 X3 - 0.0051 X4 - 0.1138 X5 + 5.62 X7 <= 16.0353
    7) - 6.0143 X1 - 0.0035 X2 - 0.0136 X3 + 0.0051 X4 + 0.1138 X5 - 5.62 X7 <= -10.7153
    8) X4 >= 110.25
    9) X5 <= 7.0
    10) X7 <= 0.4
    11) - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7 >= -75.9115
    12) - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7 >= -89.2868
    13) X1, X7 >= 0
    


```python
# Iteration functions
def check_optimal(Cj_minus_Zj, maximization):
    if maximization:
        if all(val <= 0 for val in Cj_minus_Zj):
            print(f'{Cj_minus_Zj} are <= 0, optimal solution is found successfully!')
            return True
        else:
            print(f'Any of {Cj_minus_Zj} is not <= 0, optimal solution is not found yet.')
            print('Proceed to next step.')
            return False
    else:
        if all(val >= 0 for val in Cj_minus_Zj):
            print(f'{Cj_minus_Zj} are >= 0, optimal solution is found successfully!')
            return True
        else:
            print(f'Any of {Cj_minus_Zj} is not >= 0, optimal solution is not found yet.')
            print('Proceed to next step.')
            return False


def simplex_method(df_init, calc_cols, basic_var, maximize, max_iter_cnt, decimal=4):
    iter_cnt = 0
    print(f'Iteration {iter_cnt}:')
    
    cols = ['CBi', 'Basic Variable'] + calc_cols + ['Solution']
    df = df_init.loc[df_init['Basic Variable'].isin(['Cj'] + basic_var), cols].copy()
    display(df)

    # Calculate Zj = sum(CBi * aij) and Cj - Zj
    print('Calculate Zj and Cj - Zj:')

    df = df.append({'Basic Variable': 'Zj'}, ignore_index=True)
    df.loc[df['Basic Variable']=='Zj', calc_cols] = \
        round(df.loc[df['Basic Variable'].isin(basic_var), calc_cols].mul(
        df.loc[df['Basic Variable'].isin(basic_var), 'CBi'], axis=0
        ).sum(), decimal).values.tolist()

    df = df.append({'Basic Variable': 'Cj - Zj'}, ignore_index=True)
    df.loc[df['Basic Variable']=='Cj - Zj', calc_cols] = \
        round(df.loc[df['Basic Variable']=='Cj', calc_cols] - \
        df.loc[df['Basic Variable']=='Zj', calc_cols].values, decimal).values[0].tolist()

    display(df)

    # Check the optimality condition for Cj - Zj
    Cj_minus_Zj = df.loc[df['Basic Variable']=='Cj - Zj', calc_cols]
    if check_optimal(Cj_minus_Zj.values[0].tolist(), maximize):
        df_final = df.loc[df['Basic Variable'].isin(basic_var), ['Basic Variable', 'Solution']] \
            .set_index('Basic Variable')
        return df_final, iter_cnt

    # Iterations
    while iter_cnt < max_iter_cnt:
        iter_cnt += 1
        print(f'\nIteration {iter_cnt}:')
        
        # a) Find key column
        if maximize:
            key_col = Cj_minus_Zj.idxmax(axis=1).values[0]
            key_col_val = Cj_minus_Zj.max(axis=1).values[0]
            print(f'a) Key column is {key_col} because {key_col_val} is the largest value of Cj - Zj.')
        else:
            key_col = Cj_minus_Zj.idxmin(axis=1).values[0]
            key_col_val = Cj_minus_Zj.min(axis=1).values[0]
            print(f'a) Key column is {key_col} because {key_col_val} is the smallest value of Cj - Zj.')

        # b) Find key row
        df.loc[(df['Basic Variable'].isin(basic_var)) & (df[key_col] != 0), 'Ratio'] = \
            round(df['Solution'] / df[key_col], decimal)
        #display(df)
        Ratio = df.loc[(df['Basic Variable'].isin(basic_var)) & (df['Ratio'] >= 0), 'Ratio']
        if len(Ratio.values) == 0:
            print('All values of Ratio are negative or empty, there is no solution.')
            return None, iter_cnt
        key_row = df.loc[Ratio.idxmin(), 'Basic Variable']
        key_row_val = Ratio.min()
        print(f'b) Key row is {key_row} because {key_row_val} is the smallest non-negative value of Ratio.')

        # c) Find key element
        # d) Identify entering and departing variables
        key_elmnt_val = df.loc[df['Basic Variable']==key_row, key_col].values[0]
        print(f'c) Key element is {key_elmnt_val} located at ({key_row}, {key_col}).')
        print(f'd) {key_col} is entering variable while {key_row} is departing variable.')

        # e) Filling in the elements
        print('e) Filling in the elements:')
        df_i = df[cols].copy()
        df_i.loc[df_i['Basic Variable']==key_row] = \
            [df.loc[df['Basic Variable']=='Cj', key_col].values[0], key_col] + \
            round((df.loc[df['Basic Variable']==key_row, calc_cols + ['Solution']] / \
            key_elmnt_val), decimal).values[0].tolist()

        basic_var = [key_col if var==key_row else var for var in basic_var]
        basic_var_fill = [var for var in basic_var if var!=key_col]
        for var in basic_var_fill:
            df_i.loc[df_i['Basic Variable']==var, calc_cols + ['Solution']] = \
                round((df.loc[df['Basic Variable']==var, calc_cols + ['Solution']] - \
                df.loc[df['Basic Variable']==var, key_col].values[0] * \
                df.loc[df['Basic Variable']==key_row, calc_cols + ['Solution']].values[0] / \
                key_elmnt_val), decimal)

        #display(df_i)
        
        # Calculate Zj = sum(CBi * aij) and Cj - Zj
        #print('Calculate Zj and Cj - Zj:')

        df_i.loc[df_i['Basic Variable']=='Zj', calc_cols] = \
            round(df_i.loc[df_i['Basic Variable'].isin(basic_var), calc_cols].mul(
            df_i.loc[df_i['Basic Variable'].isin(basic_var), 'CBi'], axis=0
            ).sum(), decimal).values.tolist()

        df_i.loc[df_i['Basic Variable']=='Cj - Zj', calc_cols] = \
            round(df_i.loc[df_i['Basic Variable']=='Cj', calc_cols] - \
            df_i.loc[df_i['Basic Variable']=='Zj', calc_cols].values, decimal).values[0].tolist()

        display(df_i)
        
        # Check the optimality condition for Cj - Zj
        Cj_minus_Zj = df_i.loc[df_i['Basic Variable']=='Cj - Zj', calc_cols]
        if check_optimal(Cj_minus_Zj.values[0].tolist(), maximize):
            df_final = df_i.loc[df_i['Basic Variable'].isin(basic_var), ['Basic Variable', 'Solution']] \
                .set_index('Basic Variable')
            return df_final, iter_cnt
        
        df = df_i.copy()
    
    print(f'Iteration has reached the maximum limit {max_iter_cnt}.')
    return None, iter_cnt
```


```python
# Solution
# Convert the objective function and constraints to standard form
print(f'Minimize Z = ' + lm1_plus_lm2 + \
      ' + 0 S1 + 0 S2 + 0 S3 + 0 S4 + 0 S5 + 0 S6 + 0 S7 + 0 S8 + 0 S9 + 0 S10 + 0 S11 + 0 S12' + \
      ', subject to,')
print(f'1) X1 + 0 X2 + 0 X3 + 0 X4 + 0 X5 + 0 X7 + S1 = {X1_upper_constr}')
print(f'2) 0 X1 + X2 + 0 X3 + 0 X4 + 0 X5 + 0 X7 + S2 = {X2_upper_constr}')
print(f'3) 0 X1 + 0 X2 + 0 X3 + 0 X4 + X5 + 0 X7 - S3 = {X5_lower_constr}')
print(f'4) 0 X1 - X2 + X3 + X4 + 0 X5 + 0 X7 + S4 = {-X2X3X4_constr}')
print(f'5) 0 X1 + 0 X2 - X3 + X4 + 0 X5 + 0 X7 + S5 = {-X3X4_constr}')
print('6) ' + lm1_minus_lm2[7 + decimal:] + ' + S6 = ' + \
      f'{round(Y1Y2_constr + -round(round(lm1.params[0], decimal) - round(lm2.params[0], decimal), decimal), decimal)}')
print('7) ' + lm2_minus_lm1[4 + decimal:] + ' + S7 = ' + \
      f'{round(Y1Y2_constr + -round(round(lm2.params[0], decimal) - round(lm1.params[0], decimal), decimal), decimal)}')
print(f'8) 0 X1 + 0 X2 + 0 X3 + X4 + 0 X5 + 0 X7 - S8 = {X4_lower_constr}')
print(f'9) 0 X1 + 0 X2 + 0 X3 + 0 X4 + X5 + 0 X7 + S9 = {X5_upper_constr}')
print(f'10) 0 X1 + 0 X2 + 0 X3 + 0 X4 + 0 X5 + X7 + S10 = {X7_upper_constr}')
print('11) ' + lm1_formula[9 + decimal:] + f' - S11 = {Y1Y2_lower_constr + -round(lm1.params[0], decimal)}')
print('12) ' + lm2_formula[9 + decimal:] + f' - S12 = {Y1Y2_lower_constr + -round(lm2.params[0], decimal)}')
print('13) X1, X7, S1, S2, S3, S4, S5, S6, S7, S8, S9, S10, S11, S12 >= 0')
```

    Minimize Z = 182.1483 - 135.5611 X1 - 0.1287 X2 + 0.0586 X3 - 0.0937 X4 + 8.4538 X5 + 35.256 X7 + 0 S1 + 0 S2 + 0 S3 + 0 S4 + 0 S5 + 0 S6 + 0 S7 + 0 S8 + 0 S9 + 0 S10 + 0 S11 + 0 S12, subject to,
    1) X1 + 0 X2 + 0 X3 + 0 X4 + 0 X5 + 0 X7 + S1 = 0.98
    2) 0 X1 + X2 + 0 X3 + 0 X4 + 0 X5 + 0 X7 + S2 = 808.5
    3) 0 X1 + 0 X2 + 0 X3 + 0 X4 + X5 + 0 X7 - S3 = 3.5
    4) 0 X1 - X2 + X3 + X4 + 0 X5 + 0 X7 + S4 = -183.75
    5) 0 X1 + 0 X2 - X3 + X4 + 0 X5 + 0 X7 + S5 = -147.0
    6) 6.0143 X1 + 0.0035 X2 + 0.0136 X3 - 0.0051 X4 - 0.1138 X5 + 5.62 X7 + S6 = 16.0353
    7) - 6.0143 X1 - 0.0035 X2 - 0.0136 X3 + 0.0051 X4 + 0.1138 X5 - 5.62 X7 + S7 = -10.7153
    8) 0 X1 + 0 X2 + 0 X3 + X4 + 0 X5 + 0 X7 - S8 = 110.25
    9) 0 X1 + 0 X2 + 0 X3 + 0 X4 + X5 + 0 X7 + S9 = 7.0
    10) 0 X1 + 0 X2 + 0 X3 + 0 X4 + 0 X5 + X7 + S10 = 0.4
    11) - 64.7734 X1 - 0.0626 X2 + 0.0361 X3 - 0.0494 X4 + 4.17 X5 + 20.438 X7 - S11 = -75.9115
    12) - 70.7877 X1 - 0.0661 X2 + 0.0225 X3 - 0.0443 X4 + 4.2838 X5 + 14.818 X7 - S12 = -89.2868
    13) X1, X7, S1, S2, S3, S4, S5, S6, S7, S8, S9, S10, S11, S12 >= 0
    


```python
# Solution
# Transform the standard form to initial simplex table
basic_var = ['S1', 'S2', 'S3', 'S4', 'S5', 'S6', 'S7', 'S8', 'S9', 'S10', 'S11', 'S12'] # Cannot have 'Zj'
calc_cols = X_var + basic_var
cols = ['CBi', 'Basic Variable'] + calc_cols + ['Solution'] # Must have 'CBi', 'Basic Variable',
                                                            # 'Solution' columns
basic_row_Cj = [None, 'Cj', X1_coef, X2_coef, X3_coef, X4_coef, X5_coef, X7_coef,
                0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, None] # Must have 'Cj' row
basic_row_S1 = [0, basic_var[0], 1, 0, 0, 0, 0, 0,
                1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, X1_upper_constr]
basic_row_S2 = [0, basic_var[1], 0, 1, 0, 0, 0, 0,
                0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, X2_upper_constr]
basic_row_S3 = [0, basic_var[2], 0, 0, 0, 0, 1, 0,
                0, 0, - 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, X5_lower_constr]
basic_row_S4 = [0, basic_var[3], 0, - 1, 1, 1, 0, 0,
                0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, -X2X3X4_constr]
basic_row_S5 = [0, basic_var[4], 0, 0, - 1, 1, 0, 0,
                0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, -X3X4_constr]
basic_row_S6 = [0, basic_var[5],
                round(round(lm1.params['X1'], decimal) - round(lm2.params['X1'], decimal), decimal),
                round(round(lm1.params['X2'], decimal) - round(lm2.params['X2'], decimal), decimal),
                round(round(lm1.params['X3'], decimal) - round(lm2.params['X3'], decimal), decimal),
                round(round(lm1.params['X4'], decimal) - round(lm2.params['X4'], decimal), decimal),
                round(round(lm1.params['X5'], decimal) - round(lm2.params['X5'], decimal), decimal),
                round(round(lm1.params['X7'], decimal) - round(lm2.params['X7'], decimal), decimal),
                0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0,
                round(Y1Y2_constr + -round(round(lm1.params[0], decimal) - round(lm2.params[0], decimal), decimal), decimal)]
basic_row_S7 = [0, basic_var[6],
                round(round(lm2.params['X1'], decimal) - round(lm1.params['X1'], decimal), decimal),
                round(round(lm2.params['X2'], decimal) - round(lm1.params['X2'], decimal), decimal),
                round(round(lm2.params['X3'], decimal) - round(lm1.params['X3'], decimal), decimal),
                round(round(lm2.params['X4'], decimal) - round(lm1.params['X4'], decimal), decimal),
                round(round(lm2.params['X5'], decimal) - round(lm1.params['X5'], decimal), decimal),
                round(round(lm2.params['X7'], decimal) - round(lm1.params['X7'], decimal), decimal),
                0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0,
                round(Y1Y2_constr + -round(round(lm2.params[0], decimal) - round(lm1.params[0], decimal), decimal), decimal)]
basic_row_S8 = [0, basic_var[7], 0, 0, 0, 1, 0, 0,
                0, 0, 0, 0, 0, 0, 0, - 1, 0, 0, 0, 0, X4_lower_constr]
basic_row_S9 = [0, basic_var[8], 0, 0, 0, 0, 1, 0,
                0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, X5_upper_constr]
basic_row_S10 = [0, basic_var[9], 0, 0, 0, 0, 0, 1,
                 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, X7_upper_constr]
basic_row_S11 = [0, basic_var[10],
                 round(lm1.params['X1'], decimal),
                 round(lm1.params['X2'], decimal),
                 round(lm1.params['X3'], decimal),
                 round(lm1.params['X4'], decimal),
                 round(lm1.params['X5'], decimal),
                 round(lm1.params['X7'], decimal),
                 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, - 1, 0,
                 Y1Y2_lower_constr + -round(lm1.params[0], decimal)]
basic_row_S12 = [0, basic_var[11],
                 round(lm2.params['X1'], decimal),
                 round(lm2.params['X2'], decimal),
                 round(lm2.params['X3'], decimal),
                 round(lm2.params['X4'], decimal),
                 round(lm2.params['X5'], decimal),
                 round(lm2.params['X7'], decimal),
                 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, - 1,
                 Y1Y2_lower_constr + -round(lm2.params[0], decimal)]
data = [basic_row_Cj, basic_row_S1, basic_row_S2, basic_row_S3, basic_row_S4, basic_row_S5, basic_row_S6, basic_row_S7,
       basic_row_S8, basic_row_S9, basic_row_S10, basic_row_S11, basic_row_S12]
df_simpx_init = pd.DataFrame(data, columns=cols)

maximize = False
max_iter_cnt = 20

df_simpx_final, iter_cnt = simplex_method(df_simpx_init, calc_cols, basic_var, maximize, max_iter_cnt, decimal * 2)
print(f'{iter_cnt} iterations taken.')
print('Final solution:')
display(df_simpx_final)
```

    Iteration 0:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0</td>
      <td>S1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.9800</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0</td>
      <td>1</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>808.5000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3.5000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.0</td>
      <td>S4</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-183.7500</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-147.0000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0</td>
      <td>S6</td>
      <td>6.0143</td>
      <td>0.0035</td>
      <td>0.0136</td>
      <td>-0.0051</td>
      <td>-0.1138</td>
      <td>5.620</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>16.0353</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0</td>
      <td>S7</td>
      <td>-6.0143</td>
      <td>-0.0035</td>
      <td>-0.0136</td>
      <td>0.0051</td>
      <td>0.1138</td>
      <td>-5.620</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-10.7153</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>110.2500</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>7.0000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.0</td>
      <td>S10</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0.4000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.0</td>
      <td>S11</td>
      <td>-64.7734</td>
      <td>-0.0626</td>
      <td>0.0361</td>
      <td>-0.0494</td>
      <td>4.1700</td>
      <td>20.438</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-1</td>
      <td>0</td>
      <td>-75.9115</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0</td>
      <td>S12</td>
      <td>-70.7877</td>
      <td>-0.0661</td>
      <td>0.0225</td>
      <td>-0.0443</td>
      <td>4.2838</td>
      <td>14.818</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-1</td>
      <td>-89.2868</td>
    </tr>
  </tbody>
</table>
<p>13 rows × 21 columns</p>
</div>


    Calculate Zj and Cj - Zj:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.0</td>
      <td>S1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.9800</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>808.5000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.5000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.0</td>
      <td>S4</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-183.7500</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-147.0000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0</td>
      <td>S6</td>
      <td>6.0143</td>
      <td>0.0035</td>
      <td>0.0136</td>
      <td>-0.0051</td>
      <td>-0.1138</td>
      <td>5.620</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>16.0353</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0</td>
      <td>S7</td>
      <td>-6.0143</td>
      <td>-0.0035</td>
      <td>-0.0136</td>
      <td>0.0051</td>
      <td>0.1138</td>
      <td>-5.620</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-10.7153</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>110.2500</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.0000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.0</td>
      <td>S10</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.4000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.0</td>
      <td>S11</td>
      <td>-64.7734</td>
      <td>-0.0626</td>
      <td>0.0361</td>
      <td>-0.0494</td>
      <td>4.1700</td>
      <td>20.438</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>-75.9115</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0</td>
      <td>S12</td>
      <td>-70.7877</td>
      <td>-0.0661</td>
      <td>0.0225</td>
      <td>-0.0443</td>
      <td>4.2838</td>
      <td>14.818</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>-89.2868</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [-135.5611, -0.1287, 0.0586, -0.0937, 8.4538, 35.256, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 1:
    a) Key column is X1 because -135.5611 is the smallest value of Cj - Zj.
    b) Key row is S1 because 0.98 is the smallest non-negative value of Ratio.
    c) Key element is 1.0 located at (S1, X1).
    d) X1 is entering variable while S1 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>808.500000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.0000</td>
      <td>S4</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-183.750000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-147.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0035</td>
      <td>0.0136</td>
      <td>-0.0051</td>
      <td>-0.1138</td>
      <td>5.620</td>
      <td>-6.0143</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>10.141286</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>-0.0035</td>
      <td>-0.0136</td>
      <td>0.0051</td>
      <td>0.1138</td>
      <td>-5.620</td>
      <td>6.0143</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-4.821286</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>110.250000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.0000</td>
      <td>S10</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.0000</td>
      <td>S11</td>
      <td>0.0000</td>
      <td>-0.0626</td>
      <td>0.0361</td>
      <td>-0.0494</td>
      <td>4.1700</td>
      <td>20.438</td>
      <td>64.7734</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>-12.433568</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>-0.0661</td>
      <td>0.0225</td>
      <td>-0.0443</td>
      <td>4.2838</td>
      <td>14.818</td>
      <td>70.7877</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>-19.914854</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-135.5611</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>135.5611</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, -0.1287, 0.0586, -0.0937, 8.4538, 35.256, 135.5611, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 2:
    a) Key column is X2 because -0.1287 is the smallest value of Cj - Zj.
    b) Key row is S4 because 183.75 is the smallest non-negative value of Ratio.
    c) Key element is -1.0 located at (S4, X2).
    d) X2 is entering variable while S4 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>1.0</td>
      <td>...</td>
      <td>1.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>624.750000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>-0.0000</td>
      <td>1.0000</td>
      <td>-1.0000</td>
      <td>-1.0000</td>
      <td>-0.0000</td>
      <td>-0.000</td>
      <td>-0.0000</td>
      <td>-0.0</td>
      <td>...</td>
      <td>-1.0000</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>183.750000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-147.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0171</td>
      <td>-0.0016</td>
      <td>-0.1138</td>
      <td>5.620</td>
      <td>-6.0143</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0035</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>9.498161</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.0171</td>
      <td>0.0016</td>
      <td>0.1138</td>
      <td>-5.620</td>
      <td>6.0143</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.0035</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-4.178161</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>110.250000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.0000</td>
      <td>S10</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.0000</td>
      <td>S11</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.0265</td>
      <td>-0.1120</td>
      <td>4.1700</td>
      <td>20.438</td>
      <td>64.7734</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.0626</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>-0.930818</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.0436</td>
      <td>-0.1104</td>
      <td>4.2838</td>
      <td>14.818</td>
      <td>70.7877</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.0661</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>-7.768979</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.1287</td>
      <td>0.1287</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-135.5611</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.1287</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.0701</td>
      <td>-0.2224</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>135.5611</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.1287</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, -0.0701, -0.2224, 8.4538, 35.256, 135.5611, 0.0, 0.0, -0.1287, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 3:
    a) Key column is X4 because -0.2224 is the smallest value of Cj - Zj.
    b) Key row is S11 because 8.310875 is the smallest non-negative value of Ratio.
    c) Key element is -0.112 located at (S11, X4).
    d) X4 is entering variable while S11 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.058600</td>
      <td>-0.0937</td>
      <td>8.453800</td>
      <td>35.256000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.763393</td>
      <td>0.0000</td>
      <td>37.232143</td>
      <td>182.482143</td>
      <td>578.333929</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.441071</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-8.928571</td>
      <td>0.0</td>
      <td>616.439125</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>-0.0000</td>
      <td>1.0000</td>
      <td>-0.763393</td>
      <td>0.0000</td>
      <td>-37.232143</td>
      <td>-182.482143</td>
      <td>-578.333929</td>
      <td>-0.0</td>
      <td>...</td>
      <td>-0.441071</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>8.928571</td>
      <td>-0.0</td>
      <td>192.060875</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.236607</td>
      <td>0.0000</td>
      <td>37.232143</td>
      <td>182.482143</td>
      <td>578.333929</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.558929</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-8.928571</td>
      <td>0.0</td>
      <td>-155.310875</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.017479</td>
      <td>0.0000</td>
      <td>-0.173371</td>
      <td>5.328029</td>
      <td>-6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.004394</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.014286</td>
      <td>0.0</td>
      <td>9.511458</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.017479</td>
      <td>0.0000</td>
      <td>0.173371</td>
      <td>-5.328029</td>
      <td>6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.014286</td>
      <td>0.0</td>
      <td>-4.191458</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.236607</td>
      <td>0.0000</td>
      <td>37.232143</td>
      <td>182.482143</td>
      <td>578.333929</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.558929</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-8.928571</td>
      <td>0.0</td>
      <td>101.939125</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.0000</td>
      <td>S10</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>0.236607</td>
      <td>1.0000</td>
      <td>-37.232143</td>
      <td>-182.482143</td>
      <td>-578.333929</td>
      <td>-0.0</td>
      <td>...</td>
      <td>0.558929</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>8.928571</td>
      <td>-0.0</td>
      <td>8.310875</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.017479</td>
      <td>0.0000</td>
      <td>0.173371</td>
      <td>-5.328029</td>
      <td>6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.985714</td>
      <td>-1.0</td>
      <td>-6.851458</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.076079</td>
      <td>-0.0937</td>
      <td>8.280429</td>
      <td>40.584029</td>
      <td>-6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.985714</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.017479</td>
      <td>0.0000</td>
      <td>0.173371</td>
      <td>-5.328029</td>
      <td>6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.985714</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, -0.01747857, 0.0, 0.17337143, -5.32802857, 6.93963429, 0.0, 0.0, -0.00439429, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.98571429, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 4:
    a) Key column is X7 because -5.32802857 is the smallest value of Cj - Zj.
    b) Key row is S10 because 0.4 is the smallest non-negative value of Ratio.
    c) Key element is 1.0 located at (S10, X7).
    d) X7 is entering variable while S10 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.058600</td>
      <td>-0.0937</td>
      <td>8.453800</td>
      <td>35.256</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.763393</td>
      <td>0.0000</td>
      <td>37.232143</td>
      <td>0.000</td>
      <td>578.333929</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.441071</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-182.482143</td>
      <td>-8.928571</td>
      <td>0.0</td>
      <td>543.446268</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>1.000000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>-0.763393</td>
      <td>0.0000</td>
      <td>-37.232143</td>
      <td>0.000</td>
      <td>-578.333929</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.441071</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>182.482143</td>
      <td>8.928571</td>
      <td>0.0</td>
      <td>265.053732</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.236607</td>
      <td>0.0000</td>
      <td>37.232143</td>
      <td>0.000</td>
      <td>578.333929</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.558929</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-182.482143</td>
      <td>-8.928571</td>
      <td>0.0</td>
      <td>-228.303732</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.017479</td>
      <td>0.0000</td>
      <td>-0.173371</td>
      <td>0.000</td>
      <td>-6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.004394</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-5.328029</td>
      <td>0.014286</td>
      <td>0.0</td>
      <td>7.380247</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.017479</td>
      <td>0.0000</td>
      <td>0.173371</td>
      <td>0.000</td>
      <td>6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.328029</td>
      <td>-0.014286</td>
      <td>0.0</td>
      <td>-2.060247</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.236607</td>
      <td>0.0000</td>
      <td>37.232143</td>
      <td>0.000</td>
      <td>578.333929</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.558929</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>-182.482143</td>
      <td>-8.928571</td>
      <td>0.0</td>
      <td>28.946268</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>1.000000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>1.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.236607</td>
      <td>1.0000</td>
      <td>-37.232143</td>
      <td>0.000</td>
      <td>-578.333929</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.558929</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>182.482143</td>
      <td>8.928571</td>
      <td>0.0</td>
      <td>81.303732</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.017479</td>
      <td>0.0000</td>
      <td>0.173371</td>
      <td>0.000</td>
      <td>6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.328029</td>
      <td>0.985714</td>
      <td>-1.0</td>
      <td>-4.720247</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.076079</td>
      <td>-0.0937</td>
      <td>8.280429</td>
      <td>35.256</td>
      <td>-6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-5.328029</td>
      <td>-1.985714</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.017479</td>
      <td>0.0000</td>
      <td>0.173371</td>
      <td>0.000</td>
      <td>6.939634</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.004394</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.328029</td>
      <td>1.985714</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, -0.01747857, 0.0, 0.17337143, 0.0, 6.93963429, 0.0, 0.0, -0.00439429, 0.0, 0.0, 0.0, 0.0, 0.0, 5.32802857, 1.98571429, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 5:
    a) Key column is X3 because -0.01747857 is the smallest value of Cj - Zj.
    b) Key row is S7 because 117.87274188 is the smallest non-negative value of Ratio.
    c) Key element is -0.01747857 located at (S7, X3).
    d) X3 is entering variable while S7 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.453800e+00</td>
      <td>35.256</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000e+00</td>
      <td>0.000</td>
      <td>1.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>4.480430e+01</td>
      <td>0.000</td>
      <td>8.814289e+02</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.249147</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>43.675933</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.022448e+01</td>
      <td>-9.552513</td>
      <td>0.0</td>
      <td>453.463058</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000e+00</td>
      <td>0.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-4.480430e+01</td>
      <td>0.000</td>
      <td>-8.814289e+02</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.249147</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-43.675933</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-5.022448e+01</td>
      <td>9.552513</td>
      <td>0.0</td>
      <td>355.036942</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>2.496613e+01</td>
      <td>0.000</td>
      <td>8.735547e+01</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.248033</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>-70.749903</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-5.594396e+02</td>
      <td>-7.917859</td>
      <td>0.0</td>
      <td>-82.541458</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000e+00</td>
      <td>0.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>5.320000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>1.0000</td>
      <td>-0.0000</td>
      <td>-9.919085e+00</td>
      <td>-0.000</td>
      <td>-3.970367e+02</td>
      <td>-0.0</td>
      <td>...</td>
      <td>0.251410</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-57.212918</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>-3.048321e+02</td>
      <td>0.817327</td>
      <td>-0.0</td>
      <td>117.872742</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>3.488522e+01</td>
      <td>0.000</td>
      <td>4.843922e+02</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.499443</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-13.536985</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>-2.546076e+02</td>
      <td>-8.735186</td>
      <td>0.0</td>
      <td>56.835800</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000e+00</td>
      <td>0.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000e+00</td>
      <td>1.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>-3.488522e+01</td>
      <td>0.000</td>
      <td>-4.843922e+02</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.499443</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>13.536985</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.546076e+02</td>
      <td>8.735186</td>
      <td>0.0</td>
      <td>53.414200</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000e+00</td>
      <td>0.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>1.000000</td>
      <td>-1.0</td>
      <td>-2.660000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.453800e+00</td>
      <td>35.256</td>
      <td>8.200000e-07</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6.300000e-07</td>
      <td>-2.000000</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-2.000000e-08</td>
      <td>0.000</td>
      <td>-8.200000e-07</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-6.300000e-07</td>
      <td>2.000000</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, -2e-08, 0.0, -8.2e-07, 0.0, 0.0, 0.0, 0.0, 0.0, -1.00000012, 0.0, 0.0, -6.3e-07, 2.0, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 6:
    a) Key column is S7 because -1.00000012 is the smallest value of Cj - Zj.
    b) Key row is S5 because 1.16666531 is the smallest non-negative value of Ratio.
    c) Key element is -70.74990345 located at (S5, S7).
    d) S7 is entering variable while S5 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.453800</td>
      <td>35.256</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>0.000</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>60.216607</td>
      <td>0.000</td>
      <td>935.355959</td>
      <td>1.0</td>
      <td>...</td>
      <td>0.096029</td>
      <td>0.617329</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-295.133575</td>
      <td>-14.440433</td>
      <td>0.0</td>
      <td>402.507862</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-60.216607</td>
      <td>0.000</td>
      <td>-935.355959</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.096029</td>
      <td>-0.617329</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>295.133575</td>
      <td>14.440433</td>
      <td>0.0</td>
      <td>405.992138</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.352879</td>
      <td>-0.000</td>
      <td>-1.234708</td>
      <td>-0.0</td>
      <td>...</td>
      <td>0.003506</td>
      <td>-0.014134</td>
      <td>-0.0</td>
      <td>1.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>7.907285</td>
      <td>0.111913</td>
      <td>-0.0</td>
      <td>1.166665</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.352879</td>
      <td>0.000</td>
      <td>1.234708</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.003506</td>
      <td>0.014134</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-7.907285</td>
      <td>-0.111913</td>
      <td>0.0</td>
      <td>4.153335</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>1.0000</td>
      <td>-0.0000</td>
      <td>-30.108303</td>
      <td>-0.000</td>
      <td>-467.677979</td>
      <td>-0.0</td>
      <td>...</td>
      <td>0.451986</td>
      <td>-0.808664</td>
      <td>-0.0</td>
      <td>0.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>147.566787</td>
      <td>7.220217</td>
      <td>-0.0</td>
      <td>184.621069</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>30.108303</td>
      <td>0.000</td>
      <td>467.677979</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.451986</td>
      <td>-0.191336</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.0</td>
      <td>0.0</td>
      <td>-147.566787</td>
      <td>-7.220217</td>
      <td>0.0</td>
      <td>72.628931</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000</td>
      <td>1.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>-30.108303</td>
      <td>0.000</td>
      <td>-467.677979</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.451986</td>
      <td>0.191336</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>147.566787</td>
      <td>7.220217</td>
      <td>0.0</td>
      <td>37.621069</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.352879</td>
      <td>0.000</td>
      <td>-1.234708</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.003506</td>
      <td>-0.014134</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.907285</td>
      <td>1.111913</td>
      <td>-1.0</td>
      <td>-1.493335</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.806679</td>
      <td>35.256</td>
      <td>1.234709</td>
      <td>0.0</td>
      <td>...</td>
      <td>-0.003506</td>
      <td>0.014134</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-7.907285</td>
      <td>-2.111913</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-0.352879</td>
      <td>0.000</td>
      <td>-1.234709</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.003506</td>
      <td>-0.014134</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>7.907285</td>
      <td>2.111913</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, -0.35287872, 0.0, -1.23470898, 0.0, 0.0, 0.00350578, -0.0141343, 0.0, 0.0, 0.0, 0.0, 7.90728529, 2.11191336, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 7:
    a) Key column is S1 because -1.23470898 is the smallest value of Cj - Zj.
    b) Key row is S8 because 0.15529688 is the smallest non-negative value of Ratio.
    c) Key element is 467.67797942 located at (S8, S1).
    d) S1 is entering variable while S8 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.453800e+00</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-6.437828e-02</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>9.664500e-04</td>
      <td>0.000409</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.002138</td>
      <td>0.0</td>
      <td>3.155308e-01</td>
      <td>1.543844e-02</td>
      <td>0.0</td>
      <td>0.824703</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000e-08</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>...</td>
      <td>1.000000e+00</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.000000</td>
      <td>0.0</td>
      <td>-6.000000e-08</td>
      <td>1.000000e-08</td>
      <td>0.0</td>
      <td>257.250000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000e+00</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-1.000000e-08</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.000000e+00</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-2.000000</td>
      <td>0.0</td>
      <td>6.000000e-08</td>
      <td>-1.000000e-08</td>
      <td>0.0</td>
      <td>551.250000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-2.733903e-01</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>2.312490e-03</td>
      <td>-0.014639</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>-0.002640</td>
      <td>0.0</td>
      <td>7.517697e+00</td>
      <td>9.285139e-02</td>
      <td>0.0</td>
      <td>1.358412</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>2.733903e-01</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>-2.312490e-03</td>
      <td>0.014639</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.002640</td>
      <td>0.0</td>
      <td>-7.517697e+00</td>
      <td>-9.285139e-02</td>
      <td>0.0</td>
      <td>3.961588</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>-1.000000e-08</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.000000e-08</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>7.000000e-08</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>257.250000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.0000</td>
      <td>S1</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>6.437828e-02</td>
      <td>0.000</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>-9.664500e-04</td>
      <td>-0.000409</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.002138</td>
      <td>0.0</td>
      <td>-3.155308e-01</td>
      <td>-1.543844e-02</td>
      <td>0.0</td>
      <td>0.155297</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000000e+00</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>1.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000000e+00</td>
      <td>1.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000000e+00</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>110.250000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-2.733903e-01</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>2.312490e-03</td>
      <td>-0.014639</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.002640</td>
      <td>0.0</td>
      <td>7.517697e+00</td>
      <td>1.092851e+00</td>
      <td>-1.0</td>
      <td>-1.301588</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.727190e+00</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>-2.313020e-03</td>
      <td>0.014639</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.002641</td>
      <td>0.0</td>
      <td>-7.517697e+00</td>
      <td>-2.092852e+00</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>-2.733905e-01</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>...</td>
      <td>2.313020e-03</td>
      <td>-0.014639</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.002641</td>
      <td>0.0</td>
      <td>7.517697e+00</td>
      <td>2.092852e+00</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, -0.27339045, 0.0, 0.0, 0.0, 0.0, 0.00231302, -0.01463924, 0.0, 0.0, -0.00264054, 0.0, 7.51769691, 2.09285191, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 8:
    a) Key column is X5 because -0.27339045 is the smallest value of Cj - Zj.
    b) Key row is S1 because 2.41225581 is the smallest non-negative value of Ratio.
    c) Key element is 0.06437828 located at (S1, X5).
    d) X5 is entering variable while S1 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>-0.000000e+00</td>
      <td>0.0</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-1.600000e-07</td>
      <td>1.0</td>
      <td>...</td>
      <td>1.000000e+00</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.000000</td>
      <td>0.0</td>
      <td>-1.000000e-08</td>
      <td>1.000000e-08</td>
      <td>0.0</td>
      <td>257.250000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-1.553319e+01</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.501205e-02</td>
      <td>0.006355</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.033213</td>
      <td>0.0</td>
      <td>4.901199e+00</td>
      <td>2.398082e-01</td>
      <td>0.0</td>
      <td>1.087744</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.600000e-07</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.000000e+00</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-2.000000</td>
      <td>0.0</td>
      <td>1.000000e-08</td>
      <td>-1.000000e-08</td>
      <td>0.0</td>
      <td>551.250000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>4.246623e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.791660e-03</td>
      <td>-0.016377</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>-0.011720</td>
      <td>0.0</td>
      <td>6.177756e+00</td>
      <td>2.729016e-02</td>
      <td>0.0</td>
      <td>2.017899</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-4.246623e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.791660e-03</td>
      <td>0.016377</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.011720</td>
      <td>0.0</td>
      <td>-6.177756e+00</td>
      <td>-2.729016e-02</td>
      <td>0.0</td>
      <td>3.302101</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.600000e-07</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.000000e-08</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>2.000000e-08</td>
      <td>-0.000000e+00</td>
      <td>0.0</td>
      <td>257.250000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.4538</td>
      <td>X5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>1.553319e+01</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.501205e-02</td>
      <td>-0.006355</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.033213</td>
      <td>0.0</td>
      <td>-4.901199e+00</td>
      <td>-2.398082e-01</td>
      <td>0.0</td>
      <td>2.412256</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-1.553319e+01</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.501205e-02</td>
      <td>0.006355</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.033213</td>
      <td>1.0</td>
      <td>4.901199e+00</td>
      <td>2.398082e-01</td>
      <td>0.0</td>
      <td>4.587744</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>110.250000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S12</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>4.246623e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.791660e-03</td>
      <td>-0.016377</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.011720</td>
      <td>0.0</td>
      <td>6.177756e+00</td>
      <td>1.027290e+00</td>
      <td>-1.0</td>
      <td>-0.642101</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>-4.246626e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.791130e-03</td>
      <td>0.016377</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.011721</td>
      <td>0.0</td>
      <td>-6.177756e+00</td>
      <td>-2.027291e+00</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>4.246626e+00</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.791130e-03</td>
      <td>-0.016377</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.011721</td>
      <td>0.0</td>
      <td>6.177756e+00</td>
      <td>2.027291e+00</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 4.24662566, 0.0, 0.0, -0.00179113, -0.01637661, 0.0, 0.0, -0.01172073, 0.0, 6.17775594, 2.02729064, 0.0] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 9:
    a) Key column is S5 because -0.01637661 is the smallest value of Cj - Zj.
    b) Key row is S12 because 39.20794709 is the smallest non-negative value of Ratio.
    c) Key element is -0.01637681 located at (S12, S5).
    d) S5 is entering variable while S12 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>259.307089</td>
      <td>1.0</td>
      <td>...</td>
      <td>8.905977e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.284336e+00</td>
      <td>0.0</td>
      <td>377.225875</td>
      <td>62.728343</td>
      <td>-61.061953</td>
      <td>218.042053</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-13.885308</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.431681e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.866538e-02</td>
      <td>0.0</td>
      <td>7.298447</td>
      <td>0.638443</td>
      <td>-0.388045</td>
      <td>0.838580</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-259.307089</td>
      <td>0.0</td>
      <td>...</td>
      <td>-8.905977e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.284336e+00</td>
      <td>0.0</td>
      <td>-377.225875</td>
      <td>-62.728343</td>
      <td>61.061953</td>
      <td>590.457947</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-259.307089</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.094023e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-2.843362e-01</td>
      <td>0.0</td>
      <td>-377.225875</td>
      <td>-62.728343</td>
      <td>61.061953</td>
      <td>296.457947</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.4538</td>
      <td>X5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>13.885308</td>
      <td>0.0</td>
      <td>...</td>
      <td>-1.431681e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-2.866538e-02</td>
      <td>0.0</td>
      <td>-7.298447</td>
      <td>-0.638443</td>
      <td>0.388045</td>
      <td>2.661420</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-13.885308</td>
      <td>0.0</td>
      <td>...</td>
      <td>1.431681e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.866538e-02</td>
      <td>1.0</td>
      <td>7.298447</td>
      <td>0.638443</td>
      <td>-0.388045</td>
      <td>4.338580</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000e+00</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>110.250000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.000</td>
      <td>-259.307089</td>
      <td>-0.0</td>
      <td>...</td>
      <td>1.094023e-01</td>
      <td>1.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>7.156638e-01</td>
      <td>-0.0</td>
      <td>-377.225875</td>
      <td>-62.728343</td>
      <td>61.061953</td>
      <td>39.207947</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>-0.000055</td>
      <td>0.0</td>
      <td>...</td>
      <td>-5.500000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.800000e-07</td>
      <td>0.0</td>
      <td>-0.000076</td>
      <td>-1.000013</td>
      <td>-0.999988</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000055</td>
      <td>0.0</td>
      <td>...</td>
      <td>5.500000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-5.800000e-07</td>
      <td>0.0</td>
      <td>0.000076</td>
      <td>1.000013</td>
      <td>0.999988</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 5.503e-05, 0.0, 0.0, 5.5e-07, 0.0, 0.0, 0.0, -5.8e-07, 0.0, 7.556e-05, 1.00001316, 0.99998765] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 10:
    a) Key column is S8 because -5.8e-07 is the smallest value of Cj - Zj.
    b) Key row is S3 because 29.25410513 is the smallest non-negative value of Ratio.
    c) Key element is 0.02866538 located at (S3, S8).
    d) S8 is entering variable while S3 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S2</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>881.430497</td>
      <td>1.0</td>
      <td>...</td>
      <td>2.491411e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>50.223079</td>
      <td>34.123261</td>
      <td>-43.675813</td>
      <td>180.469947</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-484.392956</td>
      <td>0.0</td>
      <td>...</td>
      <td>4.994460e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>254.608408</td>
      <td>22.272270</td>
      <td>-13.537063</td>
      <td>29.254105</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-881.430497</td>
      <td>0.0</td>
      <td>...</td>
      <td>-2.491411e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-50.223079</td>
      <td>-34.123261</td>
      <td>43.675813</td>
      <td>628.030053</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-397.037541</td>
      <td>0.0</td>
      <td>...</td>
      <td>2.514129e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-304.831488</td>
      <td>-56.395531</td>
      <td>57.212876</td>
      <td>304.775948</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.4538</td>
      <td>X5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-484.392956</td>
      <td>0.0</td>
      <td>...</td>
      <td>4.994460e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>254.608408</td>
      <td>22.272270</td>
      <td>-13.537063</td>
      <td>139.504105</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.000</td>
      <td>87.355414</td>
      <td>-0.0</td>
      <td>...</td>
      <td>-2.480332e-01</td>
      <td>1.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>0.0</td>
      <td>-0.0</td>
      <td>-559.439896</td>
      <td>-78.667800</td>
      <td>70.749939</td>
      <td>18.271843</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.000225</td>
      <td>0.0</td>
      <td>...</td>
      <td>-8.400000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.000223</td>
      <td>-1.000026</td>
      <td>-0.999980</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-0.000225</td>
      <td>0.0</td>
      <td>...</td>
      <td>8.400000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000223</td>
      <td>1.000026</td>
      <td>0.999980</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, -0.00022504, 0.0, -2.017e-05, 8.4e-07, 0.0, 0.0, 0.0, 0.0, 0.0, 0.00022277, 1.00002603, 0.99997982] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 11:
    a) Key column is S1 because -0.00022504 is the smallest value of Cj - Zj.
    b) Key row is S2 because 0.20474666 is the smallest non-negative value of Ratio.
    c) Key element is 881.43049723 located at (S2, S1).
    d) S1 is entering variable while S2 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>-1.134520e-03</td>
      <td>...</td>
      <td>-2.826600e-04</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.056979</td>
      <td>-0.038713</td>
      <td>0.049551</td>
      <td>0.775253</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S1</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.0</td>
      <td>1.134520e-03</td>
      <td>...</td>
      <td>2.826600e-04</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.056979</td>
      <td>0.038713</td>
      <td>-0.049551</td>
      <td>0.204747</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>5.495532e-01</td>
      <td>...</td>
      <td>6.363623e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>282.208663</td>
      <td>41.024817</td>
      <td>-37.539246</td>
      <td>128.431943</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>1.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>808.500000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>4.504468e-01</td>
      <td>...</td>
      <td>3.636377e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-282.208663</td>
      <td>-41.024817</td>
      <td>37.539246</td>
      <td>386.068057</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.4538</td>
      <td>X5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>5.495532e-01</td>
      <td>...</td>
      <td>6.363623e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>282.208663</td>
      <td>41.024817</td>
      <td>-37.539246</td>
      <td>238.681943</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S5</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.000</td>
      <td>0.0</td>
      <td>-9.910641e-02</td>
      <td>...</td>
      <td>-2.727247e-01</td>
      <td>1.0</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>0.0</td>
      <td>-0.0</td>
      <td>-564.417325</td>
      <td>-82.049634</td>
      <td>75.078492</td>
      <td>0.386114</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>-1.700000e-07</td>
      <td>...</td>
      <td>-2.800000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.000235</td>
      <td>-1.000035</td>
      <td>-0.999969</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>1.700000e-07</td>
      <td>...</td>
      <td>2.800000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000235</td>
      <td>1.000035</td>
      <td>0.999969</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.7e-07, -8.39e-06, 2.8e-07, 0.0, 0.0, 0.0, 0.0, 0.0, 0.00023526, 1.00003501, 0.99996905] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 12:
    a) Key column is S3 because -8.39e-06 is the smallest value of Cj - Zj.
    b) Key row is S5 because 0.01881122 is the smallest non-negative value of Ratio.
    c) Key element is 20.52572988 located at (S5, S3).
    d) S3 is entering variable while S5 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>-1.379950e-03</td>
      <td>...</td>
      <td>-9.580600e-04</td>
      <td>2.476480e-03</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.454746e+00</td>
      <td>-2.419075e-01</td>
      <td>0.235481</td>
      <td>0.776210</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S1</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.0</td>
      <td>1.379950e-03</td>
      <td>...</td>
      <td>9.580600e-04</td>
      <td>-2.476480e-03</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.454746e+00</td>
      <td>2.419075e-01</td>
      <td>-0.235481</td>
      <td>0.203790</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>5.000000e-01</td>
      <td>...</td>
      <td>5.000000e-01</td>
      <td>5.000000e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>-1.000000e-08</td>
      <td>1.000000e-08</td>
      <td>-0.000000</td>
      <td>128.625000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>1.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>808.500000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>1.000000e+00</td>
      <td>-1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>5.000000e-01</td>
      <td>...</td>
      <td>5.000000e-01</td>
      <td>-5.000000e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.000000e-08</td>
      <td>1.000000e-08</td>
      <td>0.000000</td>
      <td>385.875000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.4538</td>
      <td>X5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>-4.828400e-03</td>
      <td>...</td>
      <td>-1.328697e-02</td>
      <td>4.871934e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-2.749804e+01</td>
      <td>-3.997404e+00</td>
      <td>3.657775</td>
      <td>3.518811</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>4.828400e-03</td>
      <td>...</td>
      <td>1.328697e-02</td>
      <td>-4.871934e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>2.749804e+01</td>
      <td>3.997404e+00</td>
      <td>-3.657775</td>
      <td>3.481189</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>5.000000e-01</td>
      <td>...</td>
      <td>5.000000e-01</td>
      <td>5.000000e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.000000e-08</td>
      <td>1.000000e-08</td>
      <td>-0.000000</td>
      <td>238.875000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.0000</td>
      <td>-0.000</td>
      <td>0.0</td>
      <td>-4.828400e-03</td>
      <td>...</td>
      <td>-1.328697e-02</td>
      <td>4.871934e-02</td>
      <td>-0.0</td>
      <td>-0.0</td>
      <td>0.0</td>
      <td>-0.0</td>
      <td>-2.749804e+01</td>
      <td>-3.997404e+00</td>
      <td>3.657775</td>
      <td>0.018811</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.0</td>
      <td>-7.900000e-07</td>
      <td>...</td>
      <td>2.800000e-07</td>
      <td>-8.000000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-4.780000e-06</td>
      <td>-1.000002e+00</td>
      <td>-1.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.0</td>
      <td>7.900000e-07</td>
      <td>...</td>
      <td>-2.800000e-07</td>
      <td>8.000000e-07</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.780000e-06</td>
      <td>1.000002e+00</td>
      <td>1.000000</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    Any of [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 7.9e-07, 0.0, -2.8e-07, 8e-07, 0.0, 0.0, 0.0, 0.0, 4.78e-06, 1.00000155, 1.00000033] is not >= 0, optimal solution is not found yet.
    Proceed to next step.
    
    Iteration 13:
    a) Key column is S4 because -2.8e-07 is the smallest value of Cj - Zj.
    b) Key row is S1 because 212.71158383 is the smallest non-negative value of Ratio.
    c) Key element is 0.00095806 located at (S1, S4).
    d) S4 is entering variable while S1 is departing variable.
    e) Filling in the elements:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CBi</th>
      <th>Basic Variable</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X4</th>
      <th>X5</th>
      <th>X7</th>
      <th>S1</th>
      <th>S2</th>
      <th>...</th>
      <th>S4</th>
      <th>S5</th>
      <th>S6</th>
      <th>S7</th>
      <th>S8</th>
      <th>S9</th>
      <th>S10</th>
      <th>S11</th>
      <th>S12</th>
      <th>Solution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Cj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-135.5611</td>
      <td>X1</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>S4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>1043.775964</td>
      <td>1.440359</td>
      <td>...</td>
      <td>1.0</td>
      <td>-2.584890e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1518.428533</td>
      <td>252.497276</td>
      <td>-245.789648</td>
      <td>212.711584</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>S8</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-521.887982</td>
      <td>-0.220179</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.792445e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>-759.214266</td>
      <td>-126.248638</td>
      <td>122.894824</td>
      <td>22.269208</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.1287</td>
      <td>X2</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>808.500000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.0000</td>
      <td>S7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.0000</td>
      <td>S6</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.0586</td>
      <td>X3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-521.887982</td>
      <td>-0.220179</td>
      <td>...</td>
      <td>0.0</td>
      <td>7.924452e-01</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-759.214266</td>
      <td>-126.248638</td>
      <td>122.894824</td>
      <td>279.519208</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.4538</td>
      <td>X5</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.000</td>
      <td>13.868620</td>
      <td>0.014310</td>
      <td>...</td>
      <td>-0.0</td>
      <td>1.437398e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-7.322725</td>
      <td>-0.642480</td>
      <td>0.391975</td>
      <td>6.345104</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.0000</td>
      <td>S9</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-13.868620</td>
      <td>-0.014310</td>
      <td>...</td>
      <td>0.0</td>
      <td>-1.437398e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>7.322725</td>
      <td>0.642480</td>
      <td>-0.391975</td>
      <td>0.654896</td>
    </tr>
    <tr>
      <th>10</th>
      <td>35.2560</td>
      <td>X7</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>-0.0937</td>
      <td>X4</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>1.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>-521.887982</td>
      <td>-0.220179</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.792445e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-759.214266</td>
      <td>-126.248638</td>
      <td>122.894824</td>
      <td>132.519208</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.0000</td>
      <td>S3</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>13.868620</td>
      <td>0.014310</td>
      <td>...</td>
      <td>-0.0</td>
      <td>1.437398e-02</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-7.322725</td>
      <td>-0.642480</td>
      <td>0.391975</td>
      <td>2.845104</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>Zj</td>
      <td>-135.5611</td>
      <td>-0.1287</td>
      <td>0.0586</td>
      <td>-0.0937</td>
      <td>8.4538</td>
      <td>35.256</td>
      <td>-0.000293</td>
      <td>-0.000001</td>
      <td>...</td>
      <td>0.0</td>
      <td>-7.000000e-08</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-0.000431</td>
      <td>-1.000072</td>
      <td>-0.999931</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>Cj - Zj</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.0000</td>
      <td>0.000</td>
      <td>0.000293</td>
      <td>0.000001</td>
      <td>...</td>
      <td>0.0</td>
      <td>7.000000e-08</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000431</td>
      <td>1.000072</td>
      <td>0.999931</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>15 rows × 21 columns</p>
</div>


    [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.00029275, 1.21e-06, 0.0, 0.0, 7e-08, 0.0, 0.0, 0.0, 0.0, 0.00043067, 1.00007235, 0.99993142] are >= 0, optimal solution is found successfully!
    13 iterations taken.
    Final solution:
    


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Solution</th>
    </tr>
    <tr>
      <th>Basic Variable</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>X1</th>
      <td>0.980000</td>
    </tr>
    <tr>
      <th>S4</th>
      <td>212.711584</td>
    </tr>
    <tr>
      <th>S8</th>
      <td>22.269208</td>
    </tr>
    <tr>
      <th>X2</th>
      <td>808.500000</td>
    </tr>
    <tr>
      <th>S7</th>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>S6</th>
      <td>2.660000</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>279.519208</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>6.345104</td>
    </tr>
    <tr>
      <th>S9</th>
      <td>0.654896</td>
    </tr>
    <tr>
      <th>X7</th>
      <td>0.400000</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>132.519208</td>
    </tr>
    <tr>
      <th>S3</th>
      <td>2.845104</td>
    </tr>
  </tbody>
</table>
</div>



```python
# Solution
X1 = round(df_simpx_final.loc['X1', 'Solution'], 2)
X2 = round(df_simpx_final.loc['X2', 'Solution'], 2)
X3 = round(df_simpx_final.loc['X3', 'Solution'], 2)
X4 = round(df_simpx_final.loc['X4', 'Solution'], 2)
X5 = round(df_simpx_final.loc['X5', 'Solution'], 2)
X7 = round(df_simpx_final.loc['X7', 'Solution'], 2)

print('Variables:')
print(f'X1 = {X1}')
print(f'X2 = {X2}')
print(f'X3 = {X3}')
print(f'X4 = {X4}')
print(f'X5 = {X5}')
print(f'X7 = {X7}')


Y1 = round(round(lm1.params[0], decimal) + round(lm1.params['X1'], decimal) * X1 + round(lm1.params['X2'], decimal) * X2 + \
     round(lm1.params['X3'], decimal) * X3 + round(lm1.params['X4'], decimal) * X4 + round(lm1.params['X5'], decimal) * X5 + \
     round(lm1.params['X7'], decimal) * X7, decimal)
Y2 = round(round(lm2.params[0], decimal) + round(lm2.params['X1'], decimal) * X1 + round(lm2.params['X2'], decimal) * X2 + \
     round(lm2.params['X3'], decimal) * X3 + round(lm2.params['X4'], decimal) * X4 + round(lm2.params['X5'], decimal) * X5 + \
     round(lm2.params['X7'], decimal) * X7, decimal)
Z = round(Z_intercept + X1_coef * X1 + X2_coef * X2 + X3_coef * X3 + X4_coef * X4 + X5_coef * X5 + X7_coef * X7, 2)

print('\nSolution:')
print(f'Y1 = {Y1}')
print(f'Y2 = {Y2}')
print(f'Z = {Z}')
```

    Variables:
    X1 = 0.98
    X2 = 808.5
    X3 = 279.52
    X4 = 132.52
    X5 = 6.35
    X7 = 0.4
    
    Solution:
    Y1 = 8.4954
    Y2 = 8.4959
    Z = 16.99
    
