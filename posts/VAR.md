---
date: '2013-03-12'
title: Vector autoregression (VAR) in R
categories: ['R', 'finance']
tags: [R, replicating, finance, return predictability]
---




In this post, I want to show how to run a vector autoregression (VAR) in `R`. First, I'm gonna explain with the help of a finance example when this method comes in handy and then I'm gonna run one with the help of the `vars` package.

Some theory
=========================

So what exactly is a VAR? Without going into too much detail here, it's basically just a generalization of a univariate autoregression (AR) model. An AR model explains *one* variable linearly with its own previous values, while a VAR explains a *vector* of variables with the vector's previous values. The VAR model is a statistical tool in the sense that it just fits the coefficients that best describe the data at hand. You still should have some economic intuition on why you put the variables in your vector. For instance, you could easily estimate a VAR with a time-series of the number of car sales in Germany and the temperature in Australia. However, it's hard to sell to someone *why* you are doing this, even if you would find that one variable helps explaining the other...

Let's make an example of a VAR often applied in finance (starting with [Campbell/Ammer, 1993](http://schwert.ssb.rochester.edu/f532/jf93_ca.pdf)). Concretely, I implement an approach to decompose unexpected returns into two parts: cash flow (CF) news and discount rate (DR) news. This is an important issue, as pointed out for instance by [Chen/Zhao (2009): Return decomposition](http://www.kent.edu/business/about/upload/ChenZhao_RFS2009.pdf), which notation I'm going to use here as well:

> Naturally, ﬁnancial economists place keen interest in the relative importance of CF news and DR news—the
two fundamental components of asset valuation—in determining the time-series and cross-sectional variations of stock returns. Relatively speaking, CF news is more related to ﬁrm fundamentals because of its link to production; DR
news can reﬂect time-varying risk aversion or investor sentiment. Their relative importance thus helps greatly to understand how the ﬁnancial market works, and provides the empirical basis for theoretical modeling.

<div>

We start with the following decomposition of unexpected equity return $e_{t+1}$, based on the seminal work by Campbell/Shiller (1988):

$$ e_{t+1} = r_{t+1} - E_t r_{t+1} $$
$$ = (E_{t+1} - E_t) \sum_{j=0}^{\infty} \rho^j \Delta d_{t+1+j} - (E_{t+1} - E_t) \sum_{j=1}^{\infty} \rho^j r_{t+1+j} $$
$$ = e_{CF,t+1} - e_{DR, t+1} $$

I'm not going into details here about the notation because this is explained for instance in Chen/Zhao (2009) and tons of other papers. However, just a short motivation on what is done here. Basically, investors expect a return for the next period ($E_t r_{t+1}$). However, there is uncertainty in this world and hence, you normally don't get what you expect, but what actually happens, i.e. $r_{t+1}$. For example, investor at the beginning of 2008 most definitely expected a positive return on their stocks, otherwise they wouldn't have had invested in them. But in the end, they ended up with a high negative return because negative news came in. So the unexpected return $e_{t+1}$ is just the difference between the actual realization $r_{t+1}$ and the expected return $E_t r_{t+1}$.

</div>

However, financial economists are also interested on *why* returns didn't turn out to be the same as expected. Well obviously, some news must have arrived in period $t+1$ which led to a revisal and adjustment of the stock price, which in turn leads to a different return. The Campbell/Shiller decomposition shows that there are only two relevant parameters: news about future expected cash flows and news about future expected returns. As the above quote already shows, separating between these two is an important issue in financial research.

<div>

Now, let's introduce a VAR process. Concretely, we will assume that there is a vector of state variables $z_t$ that follows a first-order VAR. This means that every state variable in period $t+1$ can be explained by a linear combination of the state variables in $t$ and a constant. Surpressing the constant, we can write

$$ z_{t+1} = \Gamma z_t + u_{t+1} $$


We further assume that the first element of the state variable vector $z_{t+1}$ is the equity return $r_{t+1}$. We can then write the discount rate news as follows:


$$ -e_{DR,t+1} =  - (E_{t+1} - E_t) \sum_{j=1}^{\infty} \rho^j r_{t+1+j} $$ 
$$  =  - E_{t+1} \sum_{j=1}^{\infty} \rho^j r_{t+1+j} + E_t \sum_{j=1}^{\infty} \rho^j r_{t+1+j} $$ 
$$  =  - e1^{\prime} \sum_{j=1}^{\infty} \rho^j \Gamma^j z_{t+1} + \sum_{j=1}^{\infty} \rho^j \Gamma^{j+1} z_{t} $$ 
$$  =  - e1^{\prime} \sum_{j=1}^{\infty} \rho^j \Gamma^j (\Gamma z_{t} + u_{t+1}) + \sum_{j=1}^{\infty} \rho^j \Gamma^{j+1} z_{t} $$
$$  =  - e1^{\prime} \sum_{j=1}^{\infty} \rho^j \Gamma^j u_{t+1} $$
$$  =  - e1^{\prime} \rho \Gamma (I - \rho \Gamma)^{-1} u_{t+1} $$
$$  =  - e1^{\prime} \lambda u_{t+1} $$

where $\lambda = \rho \Gamma (I - \rho \Gamma)^{-1}$ and $e1$ is a vector whose first element is equal to one and zero otherwise.

</div>

This derivation looks more complicated than it is. Basically, it's just the application of a perpetuity or infinite geometric series. Why do we have to apply a perpetuity here? Well, the VAR tells us that returns today are explained by returns from last period multiplied by a persistence factor and a random shock. However, returns last period were explained by returns two periods ago and so on. So this means that every shock is not transitory (which means it only has relevance for one period), but is persistent. 

Also, maybe some of you are like me and get a headache when dealing with matrix multiplication. For those, I want to explain the computation of the $\lambda$ a little longer. This is a generalization of a geometric series which is called a [Neumann series](http://en.wikipedia.org/wiki/Neumann_series) in mathematics. It states that

$$ (I - A)^{-1} = \sum_{j=0}^{\infty} A^j $$

This formula only works if the sum of each row is smaller than 1. There are two subleties to note though:

* $A^j$ does not mean an element-wise operation, but a $j$-times multiplication of the matrix with itself. In `R`, though, if you just write $A^j$, you get the former, not the latter. If you want the latter, you have to use a special operator from the `expm` package (see discussion on [SO](http://stackoverflow.com/questions/3274818/matrix-power-in-r)). (I don't want to confuse you, so to be clear: you don't need that package here because the above formula is so much easier than applying the sum-formula. But if you want to check that the formula is correct, you can't just call $A^j$ in `R`.) 
* $A^{-1}$ in `R` is not identical to what is meant here! In `R`, it just returns the reciprocal of each element. In mathematics, it means that the inverse of a matrix is needed ($A A^{-1} = I$). 

The big takeaway is that you have to be really careful when implementing matrix formulas in `R`. I don't have a mathematical background, so I always start the most obvious way, i.e. just type $A^j$ and $A^{-1}$ and get completely non-sensical results.

So let's check that the Neumann series formula actually works. Here, I start with $j=1$ instead of $j=0$, so the formula has to be $A (I - A)^{-1}$. 


```r
library(expm)
```

```
## Loading required package: Matrix
```

```
## Loading required package: lattice
```

```
## Attaching package: 'expm'
```

```
## The following object(s) are masked from 'package:Matrix':
## 
## expm
```

```r

mat <- matrix(c(.1, .2, .3, .4), nrow=2)

res <- matrix(0, nrow=2, ncol=2)
for (j in 1:1000) {
  res[1,1] <- res[1,1] + (mat %^% j)[1,1]
  res[2,1] <- res[2,1] + (mat %^% j)[2,1]
  res[1,2] <- res[1,2] + (mat %^% j)[1,2]
  res[2,2] <- res[2,2] + (mat %^% j)[2,2]
}

res
```

```
##        [,1]  [,2]
## [1,] 0.2500 0.625
## [2,] 0.4167 0.875
```

```r
mat %*% solve(diag(2) - mat)
```

```
##        [,1]  [,2]
## [1,] 0.2500 0.625
## [2,] 0.4167 0.875
```


As you can see, applying the Neumann series formula or doing it the hard way lead to the same results.

<div>

Continuing with our example, the CF news can now easily be backed out as the difference between the total unexpected return, which is just the random shock $u_{t+1}$ and the DR news:

$$ e_{CF,t+1} = (e1^{\prime} + e1^{\prime} \lambda) u_{t+1} $$

</div>

(This, by the way, is the big objection Chen/Zhao (2009) have against the return decomposition approach. Most studies model the DR news part directly and the CF news part is backed out. So every modeling error one makes ends up in the residual which is nothing else than the CF news part. Hence, one cannot distinguish anymore between modeling noise and true CF news. They support their argument by two nice examples. First, they show that the return decomposition approach results in high CF news for government bonds although those securities do not have any CF news part by definition. Second , they show that this approach yields very different results for stocks, subject to the state variable that is used. This supports the hypothesis that the CF news is mostly modeling noise. If you are interested in this literature, make sure to also read Engsted/Pedersen/Tanggaard (2012): Pitfalls in VAR based return decompositions: A clarification. They respond to the critique brought forward by Chen/Zhao and defend the VAR based return decomposition approach.)

Alternatively, if we include log dividend growth in the state vector as the second element, we can compute the CF news part directly as

<div>

$$ e_{CF,t+1} = e2^{\prime} (I + \lambda) u_{t+1} = e2^{\prime} (I - \rho \Gamma)^{-1} u_{t+1}  $$

</div>

where $e2$ is a vector where the second element is 1 and the rest 0.

Implementation
=========================

Let's try to replicate the results in table 4 of [Chen/Da/Zhao (2009): What Drives Stock Price Movements?](http://www3.nd.edu/~zda/CDZ.pdf) because they use state variables that are all available in the [data set](http://www.hec.unil.ch/agoyal/) of Amit Goyal.

They use the following vector of state variables:

<div>

$$ z_{t+1} = [r_t \Delta d_t dp_t eqis_t]^{\prime} $$

where $r_t$ is long annual return, $\Delta d_t$ is log dividend growth, $dp_t$ is log dividend yield, and $eqis_t$ is the ratio of equity issuing activity as a fraction of total issuing activity.

</div>

OK, let's read in the data. (You find more information about the data set on my Goyal/Welch (2008) replication post).


```r
intYear <- 1927
#Use that to check out the other time period
#intYear <- 1946
library(data.table)
library(ggplot2)
library(lubridate)
library(vars)
library(reshape2)
annual  <- read.csv2("/home/christophj/Dropbox/FinanceIssues/ReturnPredictability/Data/annual_update_2010.csv", 
                     na.strings=c("NaN", "NA"), stringsAsFactors=FALSE)
annual <- as.data.table(annual)
annual <- annual[, IndexDiv := Index + D12]
annual <- annual[, dp := log(D12) - log(Index)]
annual <- annual[, ep := log(E12) - log(Index)]
vec_dy <- c(NA, annual[2:nrow(annual), log(D12)] - annual[1:(nrow(annual)-1), log(Index)])
annual <- annual[, dy := vec_dy]
annual <- annual[, logret   :=c(NA,diff(log(Index)))]
vec_logretdiv <- c(NA, annual[2:nrow(annual), log(IndexDiv)] - annual[1:(nrow(annual)-1), log(Index)])
annual <- annual[, logretdiv:=vec_logretdiv]
annual <- annual[, logRfree := log(Rfree + 1)]
annual <- annual[, rp_div   := logretdiv - logRfree]
annual <- annual[, div_growth := c(NA, diff(log(D12)))]
vec_state <- annual[yyyy >= intYear, list(logretdiv, div_growth, dp, eqis)]
```


So now that we have the vector of state variables, we can estimate the VAR. To do so, we use the package `vars` in `R`. 


```r
var_est <- VAR(vec_state,   p=1, type="const")
summary(var_est)
```

```
## 
## VAR Estimation Results:
## ========================= 
## Endogenous variables: logretdiv, div_growth, dp, eqis 
## Deterministic variables: const 
## Sample size: 83 
## Log Likelihood: 531.442 
## Roots of the characteristic polynomial:
## 0.949 0.419 0.419 0.407
## Call:
## VAR(y = vec_state, p = 1, type = "const")
## 
## 
## Estimation results for equation logretdiv: 
## ========================================== 
## logretdiv = logretdiv.l1 + div_growth.l1 + dp.l1 + eqis.l1 + const 
## 
##               Estimate Std. Error t value Pr(>|t|)   
## logretdiv.l1    0.1231     0.1078    1.14   0.2572   
## div_growth.l1  -0.2007     0.1762   -1.14   0.2581   
## dp.l1           0.1160     0.0477    2.43   0.0174 * 
## eqis.l1        -0.5069     0.1954   -2.59   0.0113 * 
## const           0.5719     0.1722    3.32   0.0014 **
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## 
## Residual standard error: 0.185 on 78 degrees of freedom
## Multiple R-Squared: 0.137,	Adjusted R-squared: 0.0932 
## F-statistic: 3.11 on 4 and 78 DF,  p-value: 0.0199 
## 
## 
## Estimation results for equation div_growth: 
## =========================================== 
## div_growth = logretdiv.l1 + div_growth.l1 + dp.l1 + eqis.l1 + const 
## 
##               Estimate Std. Error t value Pr(>|t|)    
## logretdiv.l1    0.3484     0.0525    6.64  3.8e-09 ***
## div_growth.l1   0.2034     0.0857    2.37     0.02 *  
## dp.l1          -0.0199     0.0232   -0.86     0.39    
## eqis.l1        -0.1282     0.0951   -1.35     0.18    
## const          -0.0409     0.0838   -0.49     0.63    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## 
## Residual standard error: 0.0899 on 78 degrees of freedom
## Multiple R-Squared: 0.439,	Adjusted R-squared: 0.41 
## F-statistic: 15.3 on 4 and 78 DF,  p-value: 2.94e-09 
## 
## 
## Estimation results for equation dp: 
## =================================== 
## dp = logretdiv.l1 + div_growth.l1 + dp.l1 + eqis.l1 + const 
## 
##               Estimate Std. Error t value Pr(>|t|)    
## logretdiv.l1    0.2267     0.1212    1.87    0.065 .  
## div_growth.l1   0.4237     0.1979    2.14    0.035 *  
## dp.l1           0.8928     0.0536   16.65   <2e-16 ***
## eqis.l1         0.3895     0.2196    1.77    0.080 .  
## const          -0.4819     0.1935   -2.49    0.015 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## 
## Residual standard error: 0.208 on 78 degrees of freedom
## Multiple R-Squared: 0.81,	Adjusted R-squared:  0.8 
## F-statistic:   83 on 4 and 78 DF,  p-value: <2e-16 
## 
## 
## Estimation results for equation eqis: 
## ===================================== 
## eqis = logretdiv.l1 + div_growth.l1 + dp.l1 + eqis.l1 + const 
## 
##               Estimate Std. Error t value Pr(>|t|)    
## logretdiv.l1   0.17269    0.05241    3.30   0.0015 ** 
## div_growth.l1  0.00365    0.08562    0.04   0.9661    
## dp.l1          0.04280    0.02320    1.85   0.0688 .  
## eqis.l1        0.46124    0.09498    4.86    6e-06 ***
## const          0.23049    0.08369    2.75   0.0073 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## 
## Residual standard error: 0.0898 on 78 degrees of freedom
## Multiple R-Squared: 0.36,	Adjusted R-squared: 0.328 
## F-statistic:   11 on 4 and 78 DF,  p-value: 4.04e-07 
## 
## 
## 
## Covariance matrix of residuals:
##            logretdiv div_growth        dp      eqis
## logretdiv    0.03415   0.001279 -0.034346  0.001159
## div_growth   0.00128   0.008078  0.006991  0.000426
## dp          -0.03435   0.006991  0.043096 -0.000821
## eqis         0.00116   0.000426 -0.000821  0.008064
## 
## Correlation matrix of residuals:
##            logretdiv div_growth     dp    eqis
## logretdiv     1.0000     0.0770 -0.895  0.0699
## div_growth    0.0770     1.0000  0.375  0.0527
## dp           -0.8953     0.3747  1.000 -0.0440
## eqis          0.0699     0.0527 -0.044  1.0000
```


The function call is pretty much self-explonatory. We estimate a VAR with only one lag. However, let's explain the output results of the `summary` function a little.

There are basically four summary outputs of regressions stacked up. This makes sense if you check the definition of a VAR further above again; a VAR basically wants to explain every current value of a variable with its previous value (in the case of `p=1`, otherwise with its previous value**s**) and the previous values of the other variables in the vector. Since we only want to allow for linear relations between those variables, we are basically estimating an OLS for every variable in the vector. So we can easily replicate the results by running the OLS ourselves. Let's do that for the `eqis` variable in the data set. (I will use the package  `dyn` for that because we have to lag the independent variables. To use the package, I have to transform the vector of state variables into a time-series object.)


```r
library(dyn)
summary(dyn$lm(eqis ~ lag(logretdiv, -1) + 
                      lag(div_growth, -1) + 
                      lag(dp, -1) + 
                      lag(eqis, -1), 
               data=ts(vec_state)))
```

```
## 
## Call:
## lm(formula = dyn(eqis ~ lag(logretdiv, -1) + lag(div_growth, 
##     -1) + lag(dp, -1) + lag(eqis, -1)), data = ts(vec_state))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.2830 -0.0504 -0.0152  0.0311  0.3425 
## 
## Coefficients:
##                     Estimate Std. Error t value Pr(>|t|)    
## (Intercept)          0.23049    0.08369    2.75   0.0073 ** 
## lag(logretdiv, -1)   0.17269    0.05241    3.30   0.0015 ** 
## lag(div_growth, -1)  0.00365    0.08562    0.04   0.9661    
## lag(dp, -1)          0.04280    0.02320    1.85   0.0688 .  
## lag(eqis, -1)        0.46124    0.09498    4.86    6e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.0898 on 78 degrees of freedom
##   (2 observations deleted due to missingness)
## Multiple R-squared: 0.36,	Adjusted R-squared: 0.328 
## F-statistic:   11 on 4 and 78 DF,  p-value: 4.04e-07
```


As you can see, we are getting exactly the same coefficients this way. 

Next, let's calculate $\lambda$. To do so, $\rho$ is set to $0.96$.


```r
rho <- 0.96
Gamma <- t(sapply(coef(var_est), FUN=function(df) {df[1:4, 1]}))
lambda <- (rho * Gamma) %*% solve(diag(4) - rho * Gamma)
```


A short explanation on how the `Gamma` is computed. First, remember that $\Gamma$ is the matrix of coefficients that basically completely describes the VAR. So for instance, to explain the log equity returns which is the first element of the state vector, we use the first row of $\Gamma$. The first element in this row is the OLS regression coefficient of the previous log equity return regressed on the current equity return, the second element is the coefficient of the previous log dividend growth on the current loq equity return, and so forth. 

However, the function `coef` applied on a `vars` object doesn't return such a matrix, but a list of results, where each element of the list is basically the results of one OLS. So we want to loop through those list elements and get the coefficients, which are the first four rows in the first column of each list object. This is exactly what is done in the `sapply` call.

Now we have all the ingredients to compute both the DR and the CF news. Also, the return news is just the residual:


```r
u <- resid(var_est)[, 1]
DR_news <- as.vector(c(1,0,0,0) %*% lambda %*% t(resid(var_est)))
CF_news <- as.vector(c(0,1,0,0) %*% (diag(4) + lambda) %*% t(resid(var_est)))
#Alternatively, backed out
CF_news_backed <- as.vector((c(1,0,0,0) + c(1,0,0,0) %*% lambda) %*% t(resid(var_est)))
#Other ways of writing that
#CF_news_backed <- as.vector(c(1,0,0,0) %*% t(resid(var_est)) + c(1,0,0,0) %*% lambda %*% t(resid(var_est)))
#CF_news_backed <- u + DR_news
```



```r
#Regression coefficients as reported in table 4 of Chen/Da/Zhao (2013)
summary(lm(DR_news ~ u))$coef[2, 1]*-1
```

```
## [1] 0.5612
```

```r
summary(lm(CF_news_backed ~ u))$coef[2, 1]
```

```
## [1] 0.4388
```

```r
summary(lm(CF_news ~ u))$coef[2, 1]
```

```
## [1] 0.4038
```

```r
#Variance decomposition; terms have to add to 1
var(DR_news)/var(u)
```

```
## [1] 0.586
```

```r
var(CF_news_backed)/var(u)
```

```
## [1] 0.4637
```

```r
-2*cov(CF_news_backed, DR_news)/var(u)
```

```
## [1] -0.0497
```


Those results are pretty similar to Chen/Da/Zhao (2013). So for the time period $1927-2010$, DR and CF news seem to be equally important. If you set `intYear <- 1946`, however, the regression coefficient of discount rate news on unexpected return is over 1, while CF news has a negative coefficient. This means that positive news on cash flows has a negative impact on returns, which is counterintuitive. As you can see, this approach is quite sensitive to the time period. 

<!---
Finally, let's also run regressions for longer periods. To do so, we have to implement the formulas from appendix A.3 in their paper:
-->






