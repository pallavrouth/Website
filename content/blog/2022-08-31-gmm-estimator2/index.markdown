---
title: 'Inner Workings of the GMM Estimator (Part 2)'
subtitle: "How does the GMM estimator work?"
excerpt: "This blog post demonstrates how the mechanism of GMM estimation. I construct a GMM estimator from scratch to demonstrate its basic principles"
author: Pallav Routh
date: '2022-05-12'
slug: 
  - gmm-estimator
categories:
  - Regression
  - Estimator
  - Optimization
tags:
  - Econometrics
---



This blog post is part II of an earlier blog post where I go behind the scenes of an MM estimator to illustrate the steps involved in this method. In this part, I extend the connection from an MM estimator to a GMM estimator. I demonstrate how GMM method can be applied in the instrumental variable regression context. This blog post will help you gain insights into some of the unique aspects of this estimation strategy - such as the role of weight matrix.

I start with a recap of the regression problem from part I. 

## A simple regression problem




Previously, we used an MM estimator to estimate the parameters of the following regression model - 

`$$mpg_i = \beta_0 + \beta_1cyl_i + \beta_2disp_i + \beta_3wt_i + \epsilon_i$$`

We defined the MM estimator as follows - 

`$$[1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top][\{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}]$$`

And we chose the value of betas that minimizes the value of the estimator above - 

`$$\mathbf{\hat{\beta}} = \text{argmin}_{\beta} [ 1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top][\{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}]$$`

Using the MM estimator we found out the values of beta that minimizes the sum of squares all the way to 0 -


```r
#one step estimator
one_step <- function(params){
  s_mom <- colMeans(x * as.vector(y - x %*% params))
  return(t(s_mom) %*% s_mom)
}

mm_1step_par <- optim(par = c(ones = 0, cyl = 0, disp = 0, wt = 0), 
                      fn = one_step,
                      method = "BFGS")$par

mm_1step_par
```

```
##         ones          cyl         disp           wt 
## 41.125286263 -1.787759994  0.007531794 -3.639819297
```


Finally, we showed that the estimates from the MM estimator is similar to the OLS estimates. 


```r
ols_par
```

```
## [1] 41.10768 -1.78494  0.00747 -3.63568
```

```r
round(mm_1step_par,5)
```

```
##     ones      cyl     disp       wt 
## 41.12529 -1.78776  0.00753 -3.63982
```


## From MM to GMM

The fact that our estimator converged to 0 for the above choice of betas, is not a coincidence. It is 0 because of the nature of problem we solved. If you take a closer look at our sample moments from part I, you will see that we have 4 unknowns (the 4 betas) and exactly 4 moment equations. In matrix algebra such an arrangement is referred to as a *determined* or *specified* system. Determined systems always give you a unique solution. 

Unfortunately, in many econometric problems this isn't the case. There are plenty of scenarios where we have more equations than unknowns. In matrix algebra, such an arrangement is referred to as a *over-determined* or *over-specified* system. Over-determined systems gives you many solutions. 

When this is the case, the MM estimator changes slightly. It incorporates an additional *weight matrix* `\(\mathbf{W}\)`[^1]. Therefore, the GMM estimator is a weighted sum of squares (of sample moments). In matrix notation, the estimator is given by - 

[^1]: Using weights to solve an over determined system is a common strategy in matrix algebra. In fact, GMM isn't the only method that uses such weights to find parameters of a system of equations. The weighted least squares (WLS) method also uses a similar strategy. In WLS, the estimator becomes `\((x^\top w x)^\top x^\top w y\)`. WLS is part of the FGLS strategy of estimation where weight matrices are used to *weigh* the residuals from a linear model.

`$$[ 1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top] \mathbf{W} [\{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}]$$`

This is the GMM estimator. The goal of this GMM estimator is the same as before - find values of betas that make this expression as low as possible. Mathematically this can be expressed as -

`$$\mathbf{\hat{\beta}} = \text{argmin}_{\beta}\{ 1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top \mathbf{W} \{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}$$`

You can see why it is called 'generalized' - when the weight matrix, `\(\mathbf{W}\)` is a matrix of ones ([$\mathbf{1}$]), the GMM estimator is just the MM estimator. Intuitively, without the `\(\mathbf{W}\)` matrix, our MM estimator was giving equal importance to all moment equations. But, with the inclusion of `\(\mathbf{W}\)` matrix, some moment condition is given more preference than others. The weights not only try to minimize the estimator as much as possible, but it also plays a role in shrinking the standard errors of estimates - which helps with efficiency.

But, how do we estimate such a `\(\mathbf{W}\)` matrix? Or more importantly, what is the optimal `\(\mathbf{W}\)` matrix? Hansen (1982) provides a solution. He proposes that the optimal weight matrix is equal to the inverse of the covariance of `\(x\)` and `\(e\)`. With this, he proposes the *two step* GMM estimator, that proceeds as follows -

1. Compute `\(\mathbf{\beta}^1 = \text{argmin}_{\beta}\{ 1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top\{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}\)`  
2. Compute the weight matrix `\(\mathbf{W}\)` using `\(\mathbf{\beta}^1\)`  
3. Compute the 2 step estimate `\(\mathbf{\beta}^2 = \text{argmin}_{\beta}\{ 1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top \mathbf{W} \{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}\)`

## Hansen's two step estimator

Let's see what these steps look like in practice. We have already implemented the first step using `one_step` function above. Next, we need to figure out how to compute `\(\mathbf{W}\)` using our first step estimates. Mathematically, `\(\mathbf{W} = S^{-1}\)` where `\(S = Cov(\mathbf{x}e)\)`. 

How do we estimate `\(S\)` in practise? Considering the simplest case where we consider the errors in our structural equation to be iid, `\(S = Cov(\mathbf{x}e) = E(e^2\mathbf{zz^\top})\)`. For our sample, we can estimate `\(S = \frac{1}{n} \hat{e}^2 \mathbf{zz^\top}\)` where `\(\hat{e}\)` is the residual. 

Realize that `\(\hat{e}\)` can be expressed as a function of our first step estimates because it is equal to `\((y_i-\mathbf{x}_i^\top \beta)\)`. So, we first create `\(e\)` using our first step estimates. Then we multiply `\(\hat{e}\)` with `\(\mathbf{x}\)` and invert it to obtain `\(\mathbf{W}\)`. This completes step 2 above. 

What does the `\(S\)` matrix or `\(W\)` matrix look like? Lets construct these matrices by assuming our parameter estimates are all 0.1.


```r
params <- rep(0.1,4)
# sample moments for these parameters
s_mom <- colMeans(x * as.vector(y - x %*% params))
# predict residuals using these sample moments
e_hat <- as.vector(y - x %*% mm_1step_par)
deg_fdm <- nrow(x) - ncol(x)
# S matrix
S <- (as.numeric(crossprod(e_hat)/deg_fdm) * crossprod(x)) / nrow(x)
S
```

```
##            ones         cyl       disp         wt
## ones    6.73188    41.65351   1553.192   21.65814
## cyl    41.65351   278.53153  10912.462  142.92707
## disp 1553.19197 10912.46157 458530.953 5699.27033
## wt     21.65814   142.92707   5699.270   75.92321
```

```r
W <- solve(S)
W
```

```
##             ones          cyl          disp           wt
## ones  5.70500714 -0.896315223  1.884334e-02 -1.354597879
## cyl  -0.89631522  0.260263523 -3.674085e-03  0.041534369
## disp  0.01884334 -0.003674085  9.906659e-05 -0.005895335
## wt   -1.35459788  0.041534369 -5.895335e-03  0.763940223
```

The `\(W\)` matrix is very different from a matrix of 1s. So, including the weight matrix is the estimator gives a very different final result. Here is comparison of MM and GMM sum of squares


```r
t(s_mom) %*% W %*% s_mom
```

```
##          [,1]
## [1,] 48.38703
```

```r
t(s_mom) %*% s_mom
```

```
##         [,1]
## [1,] 9362015
```

Inclusion of the `\(W\)` matrix has an effect of reducing the sum of squares of the sample moments.

Finally, we can construct our full two step estimator mentioned in step 3. I perform these steps below by encoding it within the function `two_step`.  


```r
two_step <- function(params){
  s_mom <- colMeans(x * as.vector(y - x %*% params))
  # step 2
  e_hat <- as.vector(y - x %*% mm_1step_par)
  deg_fdm <- nrow(x) - ncol(x)
  S <- (as.numeric(crossprod(e_hat)/deg_fdm) * crossprod(x)) / nrow(x)
  W <- solve(S)
  # step 3
  return(t(s_mom) %*% W %*% s_mom)
}
```

I implement the 2 step estimator on our simple regression problem[^2].

[^2]: Note that it is not necessary to do so (because we can simply use our MM estimator), but I implement it anyway


```r
mm_2step_par <- optim(par = c(ones = 0, cyl = 0, disp = 0, wt = 0), 
                      fn = two_step,
                      method = "BFGS")$par
```

These numbers represent the values of beta that minimizes our GMM estimator. Let's check how it compares with our OLS estimates and our one step estimator.


```r
mm_1step_par
```

```
##         ones          cyl         disp           wt 
## 41.125286263 -1.787759994  0.007531794 -3.639819297
```

```r
mm_2step_par
```

```
##         ones          cyl         disp           wt 
## 41.107692745 -1.784976967  0.007472892 -3.635614454
```

```r
ols_par
```

```
## [1] 41.10768 -1.78494  0.00747 -3.63568
```

Interestingly, the GMM estimate is closer to the OLS estimate compared to the MM estimate. But, let's also check the final value of our estimator for the choice of betas above.


```r
optim(par = c(ones = 0, cyl = 0, disp = 0, wt = 0), 
                      fn = two_step,
                      method = "BFGS")$value
```

```
## [1] 2.315574e-10
```

It is 0 much like our MM estimator. Thus, for this specific example, the estimator converged to a value of 0 with or without the presence of weight matrix `\(\mathbf{W}\)`. Next, we will see how this changes for a problem that is over specified.

## GMM IV estimator

The GMM estimator was specifically designed to tackle estimation problems that are over-specified. That is, there are more moment equations than unknowns parameters. But, when does this happen? A classic example of an overspecified estimation problem is instrumental variable (IV) estimation.  

For IV problems, `\(E(\mathbf{x}e) = 0\)` is no longer our moment equations. Instead, the moment equations are given by `\(E(\mathbf{z}e) = 0\)`, where `\(\mathbf{z}\)` is a matrix containing the instruments and the exogenous variables and `\(e\)` as before, represents the residuals from the regression of `\(y\)` on `\(x\)`. Let's use a concrete example to define the elements of `\(\mathbf{z}\)` and `\(e\)`.

The `CigaretteSW` data contains cigarette consumption for different US states over a period of 10 year. To keep the problem simple, let's ignore the panel nature of the data. Here is a glimpse of the data.


```r
data("CigarettesSW")
glimpse(CigarettesSW)
```

```
## Rows: 96
## Columns: 9
## $ state      <fct> AL, AR, AZ, CA, CO, CT, DE, FL, GA, IA, ID, IL, IN, KS, KY,…
## $ year       <fct> 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985,…
## $ cpi        <dbl> 1.076, 1.076, 1.076, 1.076, 1.076, 1.076, 1.076, 1.076, 1.0…
## $ population <dbl> 3973000, 2327000, 3184000, 26444000, 3209000, 3201000, 6180…
## $ packs      <dbl> 116.4863, 128.5346, 104.5226, 100.3630, 112.9635, 109.2784,…
## $ income     <dbl> 46014968, 26210736, 43956936, 447102816, 49466672, 60063368…
## $ tax        <dbl> 32.50000, 37.00000, 31.00000, 26.00000, 31.00000, 42.00000,…
## $ price      <dbl> 102.18167, 101.47500, 108.57875, 107.83734, 94.26666, 128.0…
## $ taxs       <dbl> 33.34834, 37.00000, 36.17042, 32.10400, 31.00000, 51.48333,…
```

We are interested in estimating cigarette consumption (`packs`) as a function of per capita income (`rincome`) and per capita price (`rprice`). In particular we are interested in the following log-log relationship -

`$$logpacks_i = \beta_0 + \beta_1logrincome_i + \beta_2logrprice_i + e_{i}$$`
As it turns out `rprice` is endogenous. An OLS would lead to biased estimates of `\(\beta_2\)`. And so, we need to use instrument variable methods to eliminate the bias. We decide to use per capita sales tax (`salestax`) and consumer price index (`cpi`) as instruments for `rprice`. 

Lets define what constitutes our vector of dependent variable `\(y\)`, matrix of independent variables `\(\mathbf{x}\)` and matrix of instruments `\(\mathbf{z}\)` :  
- `\(y\)` is a `\(96 \times 1\)` vector that includes values of cigarette consumption   
- `\(\mathbf{x}\)` is a `\(96 \times 3\)` matrix that includes the intercept, endogeneous variable per capita price and exogeneous variable per capita income  
- `\(\mathbf{z}\)` is a `\(96 \times 4\)` matrix that includes not only the intercept and instruments (sales tax and consumer price index) but also the exogeneous variable from `\(x\)` : per capita income

Now that we know what constitues `\(\mathbf{z}\)`, how would we compute `\(e\)`? We obtain residuals `\(e\)` by expressing it as `\(y - x^\top\mathbf{\beta}\)`. This is identical to the regression example before.

Given `\(e\)` and `\(\mathbf{z}\)`, we can finally express our moment conditions mathematically. Recall, the moment conditions are given by `\(E(\mathbf{z}e)\)`. So, for this example, the moments would look like -

`$$E\left(\begin{array}
{rl}
(logpacks_i - \beta_0 + \beta_1logrincome_i + \beta_2logrprice_i)&\times 1 \\
(logpacks_i - \beta_0 + \beta_1logrincome_i + \beta_2logrprice_i)&\times salestax_i \\
(logpacks_i - \beta_0 + \beta_1logrincome_i + \beta_2logrprice_i)&\times cpi_i \\
(logpacks_i - \beta_0 + \beta_1logrincome_i + \beta_2logrprice_i)&\times logrincome_i
\end{array}\right) = 0$$`


Now you can see why this estimation is overspecified - we have 4 equations above but 3 unknowns - `\(\beta_0\)`, `\(\beta_1\)` and `\(\beta_2\)`. Thus, over-specification is likely to arise in IV problems, when there are more than one instruments for an endogeneous variable[^3]. Note that in this example, if there was one instrument for our endogenous variable, our problem be exactly specified (because we would have 3 moment equations and 3 unknowns).

[^3]: For instrument variable regressions, when there is more than one instrument for an endogenous variable, we also refer to such a situation as *overspecification*

Let's see how we can use GMM to estimate the coefficients `\(\beta_0\)`, `\(\beta_1\)` and `\(\beta_2\)`. First, I create all the variables required for estimation -


```r
CigarettesSW$rincome <- with(CigarettesSW, income / population / cpi)
CigarettesSW$rprice <- with(CigarettesSW, price / cpi)
CigarettesSW$salestax <- with(CigarettesSW, (taxs - tax) / cpi)
```

Next, I initialize my vector `\(y\)`, matrix `\(\mathbf{x}\)` and matrix `\(\mathbf{z}\)` -


```r
# y
y <- log(CigarettesSW$packs)
# x = intercept,rprice and rincome
ones <- rep(1,length(y))
x <- as.matrix(cbind(ones,log(CigarettesSW[,c('rprice','rincome')])))
rownames(x) <- NULL
# z = intercept,salestax,cpi and rincome
z <- as.matrix(cbind(ones,CigarettesSW[,c('salestax','rincome','cpi')]))
z[,3] <- log(z[,3]) #log rincome
rownames(z) <- NULL
```

Here is a look at the first five rows of `\(\mathbf{x}\)` and the first five rows of `\(\mathbf{z}\)` -


```r
x[1:5,]
```

```
##      ones   rprice  rincome
## [1,]    1 4.553502 2.376195
## [2,]    1 4.546562 2.348339
## [3,]    1 4.614225 2.551822
## [4,]    1 4.607374 2.754509
## [5,]    1 4.472877 2.662089
```

```r
z[1:5,]
```

```
##      ones  salestax  rincome   cpi
## [1,]    1 0.7884121 2.376195 1.076
## [2,]    1 0.0000000 2.348339 1.076
## [3,]    1 4.8052211 2.551822 1.076
## [4,]    1 5.6728627 2.754509 1.076
## [5,]    1 0.0000000 2.662089 1.076
```

Before using GMM, I will use the popular 2SLS method to estimate `\(\beta_0\)`, `\(\beta_1\)` and `\(\beta_2\)`. These will serve as my benchmark.


```r
# 1st stage : predicted endogeneous variable
s1 <- z %*% round(as.numeric(solve( t(z) %*% z ) %*% t(z) %*% x[,'rprice']),5)
x_new <- as.matrix(cbind(x[,'ones'],s1,x[,'rincome']))
# 2nd stage : y ~ f(predicted endogenous,exogeneous)
colnames(x_new) <- c('ones','rprice','rincome')
tsls_par <- solve(t(x_new) %*% x_new ) %*% t(x_new) %*% y
```

Now, I will use the one step estimator `one_step` to estimate the same coefficients. We will use the one step estimates for comparison purposes and for using it within the two step estimator.


```r
#one step estimator
one_step <- function(param){
  s_mom <- colMeans( z * as.vector(y - x %*% param) ) 
  t(s_mom) %*% s_mom
}

mm_1step_par <- optim(par = c(ones = 0, rprice = 0, rincome = 0),
                      method = "BFGS",
                      fn = one_step)$par
```

Next, I initialize the two step estimator `two_step` and estimate the parameters of the model -


```r
two_step <- function(param){
  s_mom <- colMeans( z * as.vector(y - x %*% param) )
  e_hat <- as.vector(y - x %*% mm_1step_par)
  deg_fdm <- nrow(z) - ncol(z)
  S <- (as.numeric(crossprod(e_hat)/deg_fdm) * crossprod(z)) / nrow(z)
  W <- solve(S)	
  t(s_mom) %*% W %*% s_mom
}

mm_2step_par <- optim(par = c(ones = 0, rprice = 0, rincome = 0), 
                      method = "BFGS",
                      fn = two_step)$par
```

Lets check how the 2SLS, one step and two step estimates compare against each other -


```r
round(c(tsls_par[1,],tsls_par[2,],tsls_par[3,]),5)
```

```
##     ones   rprice  rincome 
##  9.87641 -1.27357  0.28282
```

```r
round(mm_1step_par,5)
```

```
##     ones   rprice  rincome 
##  5.84765 -1.84354  2.83618
```

```r
round(mm_2step_par,5)
```

```
##     ones   rprice  rincome 
##  9.87688 -1.27371  0.28287
```

The estimates from the two step estimator is much closer to the 2SLS method. This illustrates that the GMM method is another way of estimating the parameters of a 2SLS model. 

Lets check the estimator value as it iteraters over different values of betas -


```r
optim(par = c(ones = 0, rprice = 0, rincome = 0), 
      fn = two_step,
      method = "BFGS",
      control = list(trace = TRUE,
                     maxit = 100))$value
```

```
## initial  value 144.625400 
## iter  10 value 0.006539
## iter  20 value 0.000841
## iter  30 value 0.000797
## iter  40 value 0.000797
## final  value 0.000797 
## converged
```

```
## [1] 0.0007971827
```

The final value is not exactly 0 but close to it. Unlike the MM estimator, the GMM estimator does not find an exact solution. This means the estimator will not be exactly 0. However, the weights try to make it as close to 0 as possible. 
