---
title: 'Inner Workings of the GMM Estimator'
subtitle: "How does the GMM estimator work?"
excerpt: "This blog post demonstrates how the mechanism of GMM estimation. I construct a GMM estimator from scratch to demonstrate its basic principles"
author: Pallav Routh
date: '2022-05-26'
slug: 
  - gmm-estimator
categories:
  - Regression
  - Estimator
  - Optimization
tags:
  - Econometrics
---



Generalized method of moments (GMM) is a workhorse of modern econometrics. It is one of the oldest estimation methods known and it actually predates other popular methods such as OLS and MLE. Even today, many econometric models such as dynamic panel models or simultaneous equations model rely on GMM style estimators. 

When we use this method in practice, the steps involved in GMM estimation are hard to see because software platforms only show you the final result. This creates a gap between how we learn the theoretical underpinnings of this method and how it actually works on real life data. 

In this blog post, I go behind the scenes of a GMM estimator to illustrate the steps involved in this method. This will help you draw a connection between how this estimator is described in textbooks and how it really delivers results. I have split this topic into two parts. In this part I illustrate how an MM estimator works in the light of a regression problem. 

## A simple problem

We will start by focusing on a simple econometric problem where we don't need to use the generalized version of method of moments. Instead, a method of moments (MM) estimator is sufficient. We will build an MM estimator for obtaining the estimates of a multiple regression model.

We will use the `mtcars` data to estimate the following simple model -

`$$mpg_i = \beta_0 + \beta_1cyl_i + \beta_2disp_i + \beta_3wt_i + \epsilon_i$$`

Below, I create relevant dependent and independent variable (DV and IV) from the `mtcars` data.


```r
# DV
y <- mtcars$mpg
# intercept
ones <- rep(1,length(y))
# IVs including intercept
x <- as.matrix(cbind(ones,mtcars[,c('cyl','disp','wt')]))
rownames(x) <- NULL
```

Our DV `mpg` represented by `y` is a `\(32\times1\)` column vector. Whereas our IVs, represented by `x` is a matrix of dimension `\(32\times4\)`. The 4 columns corresponds to the 4 IVs including the intercept. Here is a look at the first 5 rows of IV matrix `x`.


```r
x[1:5,]
```

```
##      ones cyl disp    wt
## [1,]    1   6  160 2.620
## [2,]    1   6  160 2.875
## [3,]    1   4  108 2.320
## [4,]    1   6  258 3.215
## [5,]    1   8  360 3.440
```

First, I use ordinary least squares (OLS) method[^1]  to calculate the coefficients of the above model. I perform this matrix operation below to obtain my OLS estimates.

[^1]: The coefficients of the OLS model is estimated by performing the following matrix operation - `\((x^\top x)^{-1}x^\top y\)`, where `\(x\)` and `\(y\)` represent the matrices of IV and DV.


```r
ols_par <- round(as.numeric(solve( t(x) %*% x ) %*% t(x) %*% y),5)
names(ols_par) <- c("ones","cyl","disp","wt")
ols_par
```

```
##     ones      cyl     disp       wt 
## 41.10768 -1.78494  0.00747 -3.63568
```

Now, we will see how to estimate the above parameters using an MM estimator.

## The MM OLS estimator  

Before getting to an MM estimator, first lets take a look at what the moment conditions (or moment equations or simply moments) look like for a linear regression problem. The *population moments*[^2] in a regression problem is defined as `\(E(\mathbf{x}e) = 0\)` where `\(\mathbf{x}\)` represents all predictors and `\(e\)` represents residuals. Let's understand the moving parts of this simple equation in the light of our current model.

[^2]: Sometimes the population moments are referred to as *theoretical* moments because they are only realized in theory.

The first aspect of the definition is the matrix of predictors `\(\mathbf{x}\)` which we have already created in one of the code chunks above. The other aspect of the definition, is the matrix of residuals. In a linear model `\(y = \mathbf{x^\top \beta} + e\)`, realize that residuals can be expressed as a function of `\(y\)`, `\(\beta\)` and `\(x\)` : `\(e = y - \mathbf{x^\top \beta}\)`. We use the same expression for representing `\(e\)` in our moment equation. 

Below, I create the residuals for my data by arbitrarily choosing `\(\beta = \{0.1,0.1,0.1,0.1\}\)`. 


```r
params <- rep(0.1,4)
# residuals
(y - x %*% params)[1:5,]
```

```
## [1]   4.0380   4.0125  11.2680  -5.4215 -18.5440
```

The numbers[^3] represent residuals for the first 5 rows in the data.

[^3]: If you predicted the DV `mpg` based on the regression model - `\(0.1 + 0.1cyl_i + 0.1disp_i + 0.1wt_i\)` and then subtracted the predictions from the original DV, you would get these numbers

We have all the moving parts ready to create the moment conditions for our problem. Mathematically, what would these moments look like? Since we have 4 parameters ($\beta_0,\beta_1,\beta_2,\beta_3$) we will have 4 moment conditions. In the language of our definition above, these moment conditions look like this -


`$$E\left(\begin{array}
{rl}
(mpg_i - \beta_0 + \beta_1cyl_i + \beta_2disp_i + \beta_3wt_i)&\times 1 \\
(mpg_i - \beta_0 + \beta_1cyl_i + \beta_2disp_i + \beta_3wt_i)&\times cyl_i \\
(mpg_i - \beta_0 + \beta_1cyl_i + \beta_2disp_i + \beta_3wt_i)&\times disp_i \\
(mpg_i - \beta_0 + \beta_1cyl_i + \beta_2disp_i + \beta_3wt_i)&\times wt_i
\end{array}\right) = 0$$`

So, multiplying the residuals with the 4 IVs, one by one, and then taking the expectation would help us get our 4 moment conditions. I create the product of IVs and residuals below,


```r
(x * as.vector(y - x %*% params))[1:5,]
```

```
##          ones      cyl      disp        wt
## [1,]   4.0380   24.228   646.080  10.57956
## [2,]   4.0125   24.075   642.000  11.53594
## [3,]  11.2680   45.072  1216.944  26.14176
## [4,]  -5.4215  -32.529 -1398.747 -17.43012
## [5,] -18.5440 -148.352 -6675.840 -63.79136
```

The expectation of these 4 columns would represent the 4 moment conditions. But, how do we figure out the expectation of the 4 moments in practice? This is tantamount to answering how we can replace the population moments `\(E(\mathbf{x}e) = 0\)` with *sample moments*. We use the *analogy principle* to replace the population expectation operation `\(E\)` with its sample analogue - a mean operation `\(1/n \sum\)`. 

More concretely, using this principle we can rewrite the population moments `\(E(\mathbf{x}e) = 0\)` as sample moments `\(1/n \sum_i (\mathbf{x}_ie_i)\)`. Replacing `\(e_i\)` with `\(y-x^\top \beta\)` we can further re-write our sample moments as `\(1/n \sum_i \{\mathbf{x}_i\times  (y_i-\mathbf{x}_i^\top \beta)\}\)`.

If we find out the column means for the product matrix above, we will have obtained our sample moments. I perform this step below.


```r
colMeans(x * as.vector(y - x %*% params))
```

```
##         ones          cyl         disp           wt 
##    -4.022038   -53.555638 -3059.136434   -28.553806
```

These 4 numbers represent the sample moments corresponding to our choice of parameter `\(\beta = \{0.1,0.1,0.1,0.1\}\)`. If we change the beta values then the sample moments would also change.

Finally, we are in a position to define our MM estimator for linear regression. It is the *squared sum* of our sample moments. In matrix notation it can be written as - 

`$$[1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top][\{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}]$$`

Below, I take the sample moments and find out it's squared sum.


```r
s_mom <- colMeans(x * as.vector(y - x %*% params))
t(s_mom) %*% s_mom # MM estimator
```

```
##         [,1]
## [1,] 9362015
```

This number is the value of the estimator for our choice of beta `\(=\{0.1,0.1,0.1,0.1\}\)`. Our ultimate goal would be to chose beta values such that the squared sum above becomes 0. In other words, we need to *minimize* the number above, by choosing different values of beta. Mathematically, this is expressed as -

`$$\mathbf{\hat{\beta}} = \text{argmin}_{\beta} [ 1/n \sum_i \{\mathbf{x}_i\times e_i(\beta) \}^\top][\{1/n \sum_i \{\mathbf{x}_i\times e_i(\beta)\}]$$`

In theory, this step is equivalent to finding out the derivative of our estimator with respect to beta and solving for it by setting it equal to 0. For simple regression problems, it is possible to find out an exact analytical expression for the beta that solves the above equation. However, finding out derivatives analytically becomes impossible for more complicated problems. In such cases, numerical methods are used to find solutions to derivatives. 

So even though an analytical expression exists for our linear regression problem, I use numerical methods. Below, I use a built-in method in R called `optim`[^4] to find out the set of beta values that makes the value of our MM estimator 0.

[^4]: `optim` is a R function that helps to solve any general purpose optimization problem. The function takes as an argument starting values for parameters to be estimated and the function that needs to be optimized.


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

These numbers represent the values of beta that minimizes our estimator such that it becomes 0. I check if this is true below.


```r
optim(par = c(ones = 0, cyl = 0, disp = 0, wt = 0), 
                      fn = one_step,
                      method = "BFGS")$value
```

```
## [1] 2.014388e-07
```

This number is 0 for all intents and purposes. It's time to check if the beta values we found by minimizing our MM estimator is the same the beta values we found using OLS.


```r
ols_par
```

```
##     ones      cyl     disp       wt 
## 41.10768 -1.78494  0.00747 -3.63568
```

```r
round(mm_1step_par,5)
```

```
##     ones      cyl     disp       wt 
## 41.12529 -1.78776  0.00753 -3.63982
```

They are quite close. This shows that the MM estimator is another way to estimate the parameters of a regression model.
