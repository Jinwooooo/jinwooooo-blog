---
layout: post
title:  "Reviewing Linear Regression - Part 3"
image: ''
date:   2018-09-15 00:12:00
tags:
- Review
- Linear Regression
- Stream of Progress
- Python
description: 'Reviewing Linear Regression for Practice. Using Kaggle Public Dataset : Skillcraft'
categories:
- Linear Regression
---

## Read me first

I'm going to do a simple linear regression on the variable provided in the dataset and try to visualize some problematic variables and try to make a final model with the most correlated variable. In the end, I will organize and make a kernel up on Kaggle.com if possible, so go to the link on the bottom for an organized report. 'Stream of Progress' tags are a progressive post that are going to talk about stuff that maybe aren't so worth your time.

**Finalized Post/Blog/Kernel**

* [For Kaggle](www.google.com)
* [For My Own Blog](www.google.com)

---

**Model Comparison**
{% highlight python %}
from statsmodels.stats.anova import anova_lm

# Comparing Linear Models with ANOVA
anova_result = anova_lm(model_2, model_4)
print(anova_result)
{% endhighlight %}

{% highlight text %}
   df_resid          ssr  df_diff     ss_diff          F  Pr(>F)
0    2657.0  2653.927933      0.0         NaN        NaN     NaN
1    2661.0  2765.202804     -4.0 -111.274872  26.770408     NaN
{% endhighlight %}

Not much to take from here. Shows that Model 2 is slightly better than Model 4, but not much! Model 4 wasn't very meaningful in the first place, so I will just keep using Model 2 from now on.

---

**Multicollinearity Test**

The OLS summary always stated there is a strong indication for multicollinearity or other numerical problems. So, let's check it out with VIF (Variation Inflation Factor).

I was looking through some VIF example up on google and found out that you can't just straight up OLS summary to get the VIF. Read this [post](https://stackoverflow.com/questions/42658379/variance-inflation-factor-in-python) for more information why I'm adding constant to figure out the VIF.

{% highlight python %}
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.tools.tools import add_constant

mc_x = add_constant(x_2)
pd.Series([variance_inflation_factor(mc_x.values, i)
          for i in range(mc_x.shape[1])],index=mc_x.columns)
{% endhighlight %}

{% highlight text %}
const               373.549982
Age                   1.125467
HoursPerWeek          1.097244
APM                  38.874221
SelectByHotkeys      12.762982
AssignToHotkeys       1.613080
UniqueHotkeys         1.273288
MinimapAttacks        1.126291
NumberOfPACs         14.632116
GapBetweenPACs        2.312456
ActionLatency         5.440825
ActionsInPAC          8.627256
TotalMapExplored      1.397277
WorkersMade           1.286370
{% endhighlight %}


Damn... there isn't a concrete value that says if it's over x, it's highly correlated, but usually if the values is near 1 it's good and if it's over 5, it's not.

By the way, APM having high VIF value is somewhat logical. It will correlate with most of the activity measured because the higher the APM, the higher number of keyboard inputs and mouse clicks. Therefore, resulting in higher numbers in other variables as well.

Usually, you'd do a stepwise regression, but I've read lots of posts stating that stepwise regression should be avoided if possible. Perhaps moving one or few of the variables with high VIF may help. Guess we're back to making more Models.

---

**Back to Making Models**

Before I go into making more models, it's quite common to go back and forth on analysis or test results in linear regression. Usually multiple models have to be made and compared. I've decided to reduce one variable that has the highest VIF and see how it goes.

**5th Model (reducing APM variable due to high VIF value)**

{% highlight python %}
# 5th Model : using Model 2 and reducing variables
removed_cols = ['GameID','LeagueIndex','TotalHours','MinimapRightClicks',
                'UniqueUnitsMade','ComplexUnitsMade','ComplexAbilitiesUsed']
removed_cols = removed_cols + ['APM']
x_5 = df_train.drop(removed_cols, axis=1)
model_5 = sm.OLS(y,x_5).fit()

mc_x = add_constant(x_5)
pd.Series([variance_inflation_factor(mc_x.values, i) for i in range(mc_x.shape[1])],index=mc_x.columns)
{% endhighlight %}

{% highlight text %}
const               328.535526
Age                   1.124034
HoursPerWeek          1.096839
SelectByHotkeys       1.464873
AssignToHotkeys       1.611555
UniqueHotkeys         1.271804
MinimapAttacks        1.122639
NumberOfPACs          5.682806
GapBetweenPACs        2.312431
ActionLatency         5.292601
ActionsInPAC          2.029783
TotalMapExplored      1.396693
WorkersMade           1.262277
{% endhighlight %}

Wow. This is great. By reducing 1 APM variable, the VIF values have gotten lower significantly. I'm still not liking that there are VIF values over 5, so again, I'm going to reduce the highest VIF value variable : NumberOfPACs.

**6th Model (reducing NumberOfPACs)**

{% highlight python %}
# reducing 1 more variable
removed_cols = removed_cols + ['NumberOfPACs']
x_6 = df_train.drop(removed_cols, axis=1)
model_6 = sm.OLS(y,x_6).fit()

# testing with VIF again
mc_x = add_constant(x_6)
pd.Series([variance_inflation_factor(mc_x.values, i) for i in range(mc_x.shape[1])],index=mc_x.columns)
{% endhighlight %}

{% highlight text %}
const               97.021079
Age                  1.123118
HoursPerWeek         1.090223
SelectByHotkeys      1.430194
AssignToHotkeys      1.592759
UniqueHotkeys        1.263051
MinimapAttacks       1.122631
GapBetweenPACs       2.294757
ActionLatency        2.812819
ActionsInPAC         1.288095
TotalMapExplored     1.300130
WorkersMade          1.189928
{% endhighlight %}

Lovely. Just in case let me reduce ActionLatency instead of NumberOfPACs to see if the other model is better (ActionLatency also had VIF value over 5)

{% highlight python %}
# reducing the other variable
removed_cols = ['GameID','LeagueIndex','TotalHours','MinimapRightClicks',
                'UniqueUnitsMade','ComplexUnitsMade','ComplexAbilitiesUsed']
removed_cols = removed_cols + ['APM','ActionLatency']
x_7 = df_train.drop(removed_cols, axis=1)
model_7 = sm.OLS(y,x_7).fit()

# testing with VIF again
mc_x = add_constant(x_7)
pd.Series([variance_inflation_factor(mc_x.values, i) for i in range(mc_x.shape[1])],index=mc_x.columns)
{% endhighlight %}

{% highlight text %}
const               157.123643
Age                   1.104197
HoursPerWeek          1.096788
SelectByHotkeys       1.462628
AssignToHotkeys       1.611384
UniqueHotkeys         1.270006
MinimapAttacks        1.122171
NumberOfPACs          3.020198
GapBetweenPACs        1.933692
ActionsInPAC          1.824861
TotalMapExplored      1.392885
WorkersMade           1.261149
{% endhighlight %}

Hmm... while Model 6 has 2 variables with VIF over 2, Model 7 has 1 variable with VIF over 3 but others are lower than 2. Perhaps we should check this with ANOVA. Before going into ANOVA, let's check out their model summary!

{% highlight python %}
print(model_6.summary())
{% endhighlight %}

{% highlight text %}
OLS Regression Results                            
==============================================================================
Dep. Variable:            LeagueIndex   R-squared:                       0.934
Model:                            OLS   Adj. R-squared:                  0.934
Method:                 Least Squares   F-statistic:                     3424.
Date:                Sat, 15 Sep 2018   Prob (F-statistic):               0.00
Time:                        12:00:00   Log-Likelihood:                -4099.3
No. Observations:                2670   AIC:                             8221.
Df Residuals:                    2659   BIC:                             8285.
Df Model:                          11                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
Age                  0.0733      0.005     15.017      0.000       0.064       0.083
HoursPerWeek         0.0203      0.002     10.712      0.000       0.017       0.024
SelectByHotkeys     47.3744      5.555      8.528      0.000      36.482      58.267
AssignToHotkeys   1659.0953    127.314     13.031      0.000    1409.450    1908.741
UniqueHotkeys        0.0643      0.010      6.159      0.000       0.044       0.085
MinimapAttacks     775.5311    153.265      5.060      0.000     475.001    1076.061
GapBetweenPACs      -0.0058      0.002     -3.068      0.002      -0.010      -0.002
ActionLatency       -0.0118      0.002     -6.619      0.000      -0.015      -0.008
ActionsInPAC         0.1476      0.015      9.633      0.000       0.118       0.178
TotalMapExplored     0.0335      0.003     10.983      0.000       0.028       0.040
WorkersMade        483.4725     44.362     10.898      0.000     396.485     570.460
==============================================================================
Omnibus:                       16.776   Durbin-Watson:                   2.032
Prob(Omnibus):                  0.000   Jarque-Bera (JB):               16.702
Skew:                          -0.178   Prob(JB):                     0.000236
Kurtosis:                       2.849   Cond. No.                     6.12e+05
==============================================================================

Warnings:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 6.12e+05. This might indicate that there are
strong multicollinearity or other numerical problems.
{% endhighlight %}

{% highlight python %}
print(model_7.summary())
{% endhighlight %}

{% highlight text %}
OLS Regression Results                            
==============================================================================
Dep. Variable:            LeagueIndex   R-squared:                       0.947
Model:                            OLS   Adj. R-squared:                  0.947
Method:                 Least Squares   F-statistic:                     4361.
Date:                Sat, 15 Sep 2018   Prob (F-statistic):               0.00
Time:                        12:00:00   Log-Likelihood:                -3795.7
No. Observations:                2670   AIC:                             7613.
Df Residuals:                    2659   BIC:                             7678.
Df Model:                          11                                         
Covariance Type:            nonrobust                                         
====================================================================================
                       coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------------
Age                  0.0212      0.004      5.184      0.000       0.013       0.029
HoursPerWeek         0.0099      0.002      5.710      0.000       0.006       0.013
SelectByHotkeys     31.3489      4.939      6.347      0.000      21.664      41.034
AssignToHotkeys    874.1684    117.459      7.442      0.000     643.848    1104.489
UniqueHotkeys        0.0297      0.009      3.155      0.002       0.011       0.048
MinimapAttacks    1050.7058    137.136      7.662      0.000     781.802    1319.610
NumberOfPACs       692.5465     25.562     27.093      0.000     642.423     742.670
GapBetweenPACs      -0.0104      0.001     -9.035      0.000      -0.013      -0.008
ActionsInPAC         0.1599      0.013     12.540      0.000       0.135       0.185
TotalMapExplored    -0.0069      0.003     -2.209      0.027      -0.013      -0.001
WorkersMade        185.1212     41.172      4.496      0.000     104.389     265.853
==============================================================================
Omnibus:                       10.968   Durbin-Watson:                   1.999
Prob(Omnibus):                  0.004   Jarque-Bera (JB):               10.836
Skew:                          -0.139   Prob(JB):                      0.00444
Kurtosis:                       2.856   Cond. No.                     3.97e+05
==============================================================================

Warnings:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The condition number is large, 3.97e+05. This might indicate that there are
strong multicollinearity or other numerical problems.
{% endhighlight %}

Just by skimming through the summaries, seems like Model 7 is a better pick. They both still have the warning about multicollinearity, but we've checked it through VIF, so I'm going to ignore this error moving forward.

{% highlight python %}
anova_result = anova_lm(model_6, model_7)
print(anova_result)
{% endhighlight %}

{% highlight text %}
   df_resid          ssr  df_diff     ss_diff    F  Pr(>F)
0    2659.0  3369.798241      0.0         NaN  NaN     NaN
1    2659.0  2684.315046     -0.0  685.483195 -inf     NaN
{% endhighlight %}

The only values that we can use is SSR. Model 7 is slighly better in this respect, so we'll use Model 7 as our final model, and do **residual analysis** and **predicting with test set**!

<img src="../uploads/clapping.gif">
