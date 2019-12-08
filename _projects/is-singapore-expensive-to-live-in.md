---
title: "Is Singapore expensive to live in"
collection: projects
tags:
    - python
    - notebook
--- 
# "Is Singapore expensive to live in?"
Inspired by the following articles:


[CNBC - Singapore named the world's most expensive
city](https://www.cnbc.com/2018/03/14/singapore-named-the-worlds-most-expensive-
city-to-live-in.html) - Singapore was named the most expensive city in the world
to live on the 14th March, 2018. This had caused a lot of murmur amongst the
citizens as they quietly acknowledge the study and the article was passed around
on social media.

[gov.sg - Is Singapore the most expensive city to live
in?](https://www.gov.sg/factually/content/is-singapore-the-most-expensive-city-
to-live-in) - The number study was countered by the government in looking at.

As these two looked at more at the definition of 'basket of goods', I felt that
it was also important to look at purchasing power such as income and inflation
to paint a more inclusive picture. 

**In [76]:**

{% highlight python %}
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import statsmodels.api as sm
from statsmodels.sandbox.regression.predstd import wls_prediction_std

%matplotlib inline
{% endhighlight %}

    D:\Anaconda3\lib\site-packages\statsmodels\compat\pandas.py:56: FutureWarning: The pandas.core.datetools module is deprecated and will be removed in a future version. Please use the pandas.tseries module instead.
      from pandas.core import datetools

 
### Looking at household income data

From https://www.singstat.gov.sg/ , I've exported an excel file locally
containing household income between 2010 and 2017.

First, the file is read in to pandas using **pd.read_excel**.

Afterwards, **index_col** argument is used to indicate that we would like to use
the 'variables' column in the file as a key

**skiprows** argument states to skip the first 5 rows that had contained
information of the subject, title and topic, which will not be used in the
analysis.

**skip footer** arguments indicates to skip the number of rows from the bottom
(82 in this case) which contained definitions of each variable. These will not
be used in the analysis for now. 

**In [26]:**

{% highlight python %}
householdincome2010_2017 = pd.read_excel('Household income from work.xlsx',index_col=0,skiprows=5,skipfooter=82)
{% endhighlight %}

**In [31]:**

{% highlight python %}
householdincome2010_2017 = householdincome2010_2017.T
{% endhighlight %}

**In [54]:**

{% highlight python %}
householdincome2010_2017 = householdincome2010_2017.reset_index().rename({'index':'Year'},axis=1)
householdincome2010_2017['Year'] = householdincome2010_2017['Year'].astype(int)
{% endhighlight %}

**In [67]:**

{% highlight python %}
householdincome2010_2017.plot(kind = "line",x='Year',y='Gini Coefficient Based On Equivalised Household Income From Work After Accounting For Government Transfers And Taxes (Based On Square Root Scale)')
{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x1f5f7864588>



 
![png]({{ BASE_PATH }}/images/is-singapore-expensive-to-live-in_6_1.png) 


**In [68]:**

{% highlight python %}
householdincome2010_2017.plot(kind="line",x='Year',y='Median Monthly Household Income From Work Including Employer CPF Contributions (Dollar)')

{% endhighlight %}




    <matplotlib.axes._subplots.AxesSubplot at 0x1f5f789b978>



 
![png]({{ BASE_PATH }}/images/is-singapore-expensive-to-live-in_7_1.png) 


**In [107]:**

{% highlight python %}
X = householdincome2010_2017['Year']
X = sm.add_constant(X)
y = householdincome2010_2017['Median Monthly Household Income From Work Including Employer CPF Contributions (Dollar)']
{% endhighlight %}

**In [108]:**

{% highlight python %}
results = sm.OLS(y,X).fit()
{% endhighlight %}

**In [109]:**

{% highlight python %}
print(results.summary())
{% endhighlight %}

                                                                   OLS Regression Results                                                              
    ===================================================================================================================================================
    Dep. Variable:     Median Monthly Household Income From Work Including Employer CPF Contributions (Dollar)   R-squared:                       0.961
    Model:                                                                                                 OLS   Adj. R-squared:                  0.955
    Method:                                                                                      Least Squares   F-statistic:                     149.2
    Date:                                                                                     Sun, 12 Aug 2018   Prob (F-statistic):           1.83e-05
    Time:                                                                                             20:32:11   Log-Likelihood:                -52.555
    No. Observations:                                                                                        8   AIC:                             109.1
    Df Residuals:                                                                                            6   BIC:                             109.3
    Df Model:                                                                                                1                                         
    Covariance Type:                                                                                 nonrobust                                         
    ==============================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
    ------------------------------------------------------------------------------
    const      -7.479e+05   6.19e+04    -12.085      0.000   -8.99e+05   -5.96e+05
    Year         375.3810     30.734     12.214      0.000     300.177     450.585
    ==============================================================================
    Omnibus:                        1.633   Durbin-Watson:                   0.883
    Prob(Omnibus):                  0.442   Jarque-Bera (JB):                1.044
    Skew:                          -0.709   Prob(JB):                        0.593
    Kurtosis:                       1.940   Cond. No.                     1.77e+06
    ==============================================================================
    
    Warnings:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 1.77e+06. This might indicate that there are
    strong multicollinearity or other numerical problems.


    D:\Anaconda3\lib\site-packages\scipy\stats\stats.py:1390: UserWarning: kurtosistest only valid for n>=20 ... continuing anyway, n=8
      "anyway, n=%i" % int(n))

 
#### Reading Year coefficients
It seems that year is directly correlated to median household income. On
average, its likely that the median income per year is rising by approximately
$375 sgd (including cpf contributions) 

**In [116]:**

{% highlight python %}
print('Parameters: ',results.params)
print('R2: ', results.rsquared)
{% endhighlight %}

    Parameters:  const   -747874.047619
    Year        375.380952
    dtype: float64
    R2:  0.9613339084472377


**In [111]:**

{% highlight python %}
prstd, iv_l, iv_u = wls_prediction_std(results)
{% endhighlight %}

**In [114]:**

{% highlight python %}
fig, ax = plt.subplots(figsize=(8,6))

ax.plot(X['Year'], y, 'o', label="data")
ax.plot(X['Year'], results.fittedvalues, 'r--.', label="OLS")
ax.plot(X['Year'], iv_u, 'r--')
ax.plot(X['Year'], iv_l, 'r--')
ax.legend(loc='best');
{% endhighlight %}

 
![png]({{ BASE_PATH }}/images/is-singapore-expensive-to-live-in_14_0.png) 

 
 

**In [None]:**

{% highlight python %}

{% endhighlight %}
