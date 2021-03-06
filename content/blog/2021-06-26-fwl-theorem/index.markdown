---
title: 'The Frisch-Waugh-Lovell theorem'
subtitle: "The intriguing FWL theorem and its connection with a fixed effects estimator"
excerpt: "This blog post demonstrates the result of Frisch-Waugh-Lovell (FWL) theorem how it can be used to understand the equivalence of different fixed effects estimators used in panel data settings."
author: Pallav Routh
date: '2021-06-26'
slug: 
  - fwl-theorem
categories:
  - Regression
  - Residuals
  - Panel model
tags:
  - Econometrics
---



In this blog post, I demonstrate the main result of the Frisch-Waugh-Lovell (FWL) theorem how it can be used to understand the equivalence of different fixed effects estimators used in panel data settings. But, instead of using math definitions and derivations, I rely on simulations and practical examples. 

# What is FWL theorem?

The FWL theorem provides a very intriguing result that centers around the residuals obtained from linear regression models. Let's say you were attempting to estimate the relationship between a dependent variable (DV) `\(y\)` and independent variables (IV) `\(x_1,x_2,x_3,x_4\)` and `\(x_5\)` using the following linear model -

`$$y = \beta_0 + \beta_1x_1 + \beta_2x_2 + \beta_3x_3 + \beta_4x_4 + \beta_5x_5 + \epsilon_i$$`

But, you are only really interested in the effect of `\(x_1\)` on `\(y\)` (that is, `\(\beta_1\)`). And, you are not concerned about the effects of the remaining IVs. 

Conventionally you would use a full regression model[^1] that regresses `\(y\)` on all the IVs. This regression will give you the coefficient on `\(x_1\)` - `\(\beta_1\)` - along with the coefficients of the other IVs. 

[^1]: Linear regression models use least squares (LS) method to estimate coefficients. In matrix notation, LS estimator is given by `\((X^TX)^{-1}X^Ty\)`. Here `\(X\)` is the `\(i \times k\)` matrix of independent variables.

FWL theorem is a proof[^2] which shows that there is another way to estimate the same `\(\beta_1\)` using two sets of residuals from two different linear regressions. The first set of residual - lets call it `\(res1\)` - comes from a regression of `\(y\)` on all IVs baring `\(x_1\)`. The second set of residuals - lets call it `\(res2\)` - comes from another regression of `\(x_1\)` on all remaining IVs. If you were to then regress `\(res1\)` on `\(res2\)`, the coefficient on `\(res2\)` would be the same as the coefficient on `\(x_1\)`. 

[^2]: The initial proof was given by Frisch and Waugh in 1933, and later a generalization was developed by Lovell in 1963

Let's understand the result of the theorem with a simple simulated example. First, let's assume that our five IVs - `\(x_1,x_2,x_3,x_4\)` and `\(x_5\)` - come from different normal distributions with slightly different means and are mildly correlated with each other with a correlation coefficient of 0.15. Such correlations between IVs are quite frequent in data analysis. Below I simulate a sample data set consisting of 2000 observations of `\(x_1,x_2,x_3,x_4\)` and `\(x_5\)`.  



```r
means <- c(0.22,0.19,1.2,2.3,0.7)
(sigma <- matrix(0.15, nrow = 5, ncol = 5) + 0.85*diag(5))
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,] 1.00 0.15 0.15 0.15 0.15
## [2,] 0.15 1.00 0.15 0.15 0.15
## [3,] 0.15 0.15 1.00 0.15 0.15
## [4,] 0.15 0.15 0.15 1.00 0.15
## [5,] 0.15 0.15 0.15 0.15 1.00
```

```r
set.seed(123)
IV_mat <- MASS::mvrnorm(n = 2000, mu = means, Sigma = sigma)
cor(IV_mat) #the correlations among IVs
```

```
##           [,1]      [,2]      [,3]      [,4]      [,5]
## [1,] 1.0000000 0.1569706 0.1708500 0.1593886 0.1425942
## [2,] 0.1569706 1.0000000 0.1486081 0.0921195 0.1724402
## [3,] 0.1708500 0.1486081 1.0000000 0.1670487 0.1462504
## [4,] 0.1593886 0.0921195 0.1670487 1.0000000 0.1570052
## [5,] 0.1425942 0.1724402 0.1462504 0.1570052 1.0000000
```

Next, I'll assume that the true exact relationship between `\(y\)` and our IVs, is given by `\(y = 0.3x_1 - 0.7x_2 + 1.2x_3 + 0.5x_4 + 0.02x_5\)`. I introduce some randomness in this relationship by adding errors that come from a standard normal distribution. The errors make the relationship between `\(y\)` and our IVs inexact and more realistic.


```r
error <- rnorm(2000,0,1)
y <- 0.3*IV_mat[,1] - 0.7*IV_mat[,2] + 1.2*IV_mat[,3] + 
     0.5*IV_mat[,4] + 0.02*IV_mat[,5] + error

simulated_data <- data.frame(cbind(y,IV_mat))
names(simulated_data) <- c("y","x1","x2","x3","x4","x5")
head(simulated_data,5) #a look at our simulated data
```

```
##            y         x1         x2        x3       x4         x5
## 1  5.0908878  0.5501454  0.9407374 1.4215863 3.004324  0.2784711
## 2 -0.0298536  0.3612854  1.8514552 0.4598710 1.502037  1.0863918
## 3  4.0571596  0.3260666 -2.0433642 0.4727362 2.095463 -0.6495949
## 4  2.4999962 -0.8974912  0.2958454 2.2155288 1.728997  1.0676919
## 5  3.1743702 -1.3556435 -0.2469353 1.5504682 2.576308  1.7201213
```

Now lets visualize FWL theorem using this simulated data. Lets say we are only interested in the effect of `\(x_1\)` on `\(y\)`. Before applying the FWL theorem, lets estimate the full model - that is regress `\(y\)` on all IVs to get the coefficient on `\(x_1\)`. 


```r
coefficients(lm(y ~ x1 + x2 + x3 + x4 + x5, 
                data = simulated_data))['x1']
```

```
##        x1 
## 0.2864559
```

We can see from the result above that regression has done a decent job at estimating the true relationship between `\(x_1\)` and `\(y\)`. Next, lets try to use FWL theorem which proceeds in two steps. In the first step, I'll get residuals - `\(res1\)` from regressing `\(y\)` on all IVs except `\(x_1\)`.


```r
res1 <- residuals(lm(y ~ x2 + x3 + x4 + x5 , 
                     data = simulated_data)) 
```

In this next step, I'll get the residuals - `\(res2\)` - from regressing `\(x_1\)` - our IV of interest - on the remaining IVs. 


```r
res2 <- residuals(lm(x1 ~ x2 + x3 + x4 + x5, 
                     data = simulated_data))
```

Now, I will regress the residuals we obtained from the previous two steps and examine the coefficient on `\(res2\)`.


```r
coefficients(lm(res1 ~ res2))['res2']
```

```
##      res2 
## 0.2864559
```

The coefficient on `\(res2\)` is the same as `\(x_1\)`. This similarity is exactly the result of the FWL theorem.

# Why does FWL thorem work?

The intriguing result works because you can *partial out* or *net out* or *eliminate* the effects of certain IVs using residuals from a regression model. If you think about it, residuals in a linear regression represent the *left over* variation in the dependent variable after you have accounted for some of it using the IVs available to you. 

In our example, `\(y\)` is a random variable whose variation is explained by IVs `\(x_1\)` to `\(x_5\)` (along with some error). Now, if we were to exclude `\(x_1\)` from the regression of `\(y\)` on `\(x_2,x_3,x_4\)` and `\(x_5\)`, then the residuals from this regression will contain the *left over* variation of `\(y\)` due to `\(x_1\)` (and the error). Mathematically speaking, these residuals represent a *new* `\(y\)` where the variations due to `\(x_2,x_3,x_4\)` and `\(x_5\)` have been *eliminated* or *netted out*. 

Lets look at the correlation between `\(res1\)` and `\(x_2,x_3,x_4\)` and `\(x_5\)`. 


```r
cor(res1,IV_mat[,2:5])
```

```
##               [,1]          [,2]         [,3]         [,4]
## [1,] -3.737607e-20 -2.920042e-17 4.708105e-17 1.172559e-17
```

They are all 0. Now, lets check out the correlation between `\(res1\)` and `\(x_1\)`.


```r
cor(res1,IV_mat[,1])
```

```
## [1] 0.2581669
```

The correlation is intact! So, the effects of `\(x_2,x_3,x_4\)` have been eliminated from the residuals from first regression; with only the effects of `\(x_1\)` remaining. So, if you were to take `\(res1\)` and build a linear model by regressing on `\(x_1\)`, then the estimated coefficient on `\(x_1\)` would be the same as `\(\beta_1\)` in the full model (with all IVs). 

But, then, what's the point of the second regression? The reason for performing the second regression, is exactly the same as the first - to net out or eliminate the effects of all other IVs from `\(x_1\)`. In other words, the residuals from the regression of `\(x_1\)` on the remaining IVs, represent a 'new' `\(x_1\)` where the effects (or covariation) of the other IVs have been removed. Notice, when I was simulated our IVs previously, I simulated them in such a way that they had a correlation of 0.15 between them. In many practical settings, this is usually the case. You will rarely see IVs in a regression model that are completely independent of each other with 0 correlation. 

To make the above argument concrete imagine a regression where all IVs are independent (normal) with extremely small correlation among them. I simulate such a data set below -


```r
means <- c(0.22,0.19,1.2,2.3,0.7)

set.seed(123)
# sigma is just an identity matrix now
IV_mat <- MASS::mvrnorm(n = 2000, mu = means, Sigma = diag(5)) 
cor(IV_mat) #the correlations among IVs
```

```
##             [,1]          [,2]        [,3]          [,4]          [,5]
## [1,] 1.000000000  4.507323e-02  0.01832182  5.205300e-02  0.0057620051
## [2,] 0.045073230  1.000000e+00 -0.01207096 -6.918241e-05 -0.0009912489
## [3,] 0.018321817 -1.207096e-02  1.00000000  1.465362e-02 -0.0112260676
## [4,] 0.052053000 -6.918241e-05  0.01465362  1.000000e+00 -0.0130058347
## [5,] 0.005762005 -9.912489e-04 -0.01122607 -1.300583e-02  1.0000000000
```

As we can see the correlations between our IVs are almost zero. Then, I simulated our error and dependent variable `\(y\)` as before.


```r
error <- rnorm(2000,0,1)
y <- 0.3*IV_mat[,1] - 0.7*IV_mat[,2] + 1.2*IV_mat[,3] + 
     0.5*IV_mat[,4] + 0.02*IV_mat[,5] + error

simulated_data <- data.frame(cbind(y,IV_mat))
names(simulated_data) <- c("y","x1","x2","x3","x4","x5")
```

Now, lets compare the results of a full regression versus FWL regression but with one set of residual only.


```r
coefficients(lm(y ~ x1 + x2 + x3 + x4 + x5, 
                data = simulated_data))['x1']
```

```
##        x1 
## 0.3195986
```

```r
res1 <- residuals(lm(y ~ x2 + x3 + x4 + x5 , 
                     data = simulated_data))

coefficients(lm(res1 ~ x1, 
                data = simulated_data))['x1']
```

```
##        x1 
## 0.3179642
```

As you can see, they are extremely close. A minor difference persists because the correlation between the IVs in `IV_mat` is not completely zero. But, the above exercise shows why we need the second regression. 

In summary, FWL theorem works because it is able to sequentially net out or eliminate the effects of certain variables from (1) the DV and (2) the IV of interest by using two sets of residuals. The relationship between these sets of residuals mirror the original relationship between the DV and the IV of interest. 

# FWL and connection between FE estimators 

The FWL theorem has a strong connection with fixed models used in panel data settings. I will demonstrate this connection with an example. Consider the following panel data on wages earned for 532 workers over a period of 10 years. The variable `id` is an identifier for a particular person in the sample. And the variable `year` indicates a time point.


```r
data("LaborSupply") # from AER package in R
LaborSupply <-
  LaborSupply %>%
  filter(year < 1984)
head(LaborSupply,5)
```

```
##   lnhr lnwg kids age disab id year
## 1 7.58 1.91    2  27     0  1 1979
## 2 7.75 1.89    2  28     0  1 1980
## 3 7.65 1.91    2  29     0  1 1981
## 4 7.47 1.89    2  30     0  1 1982
## 5 7.50 1.94    2  31     0  1 1983
```

Lets say we wish to model log wage (`lnhr`) as a linear function of log hours (`lnwg`), number of children (`kids`), age (`age`) and bad health (`disab`). Mathematically, we can can express this relationship as -

`$$lnwg_{it} = \beta_0 + \beta_1lnhr_{it} + \beta_2kids_{it} + \beta_3age_{it} + \beta_4disab_{it} + \mu_i + \epsilon_{it}$$`

where, `\(\mu_i\)` is an unobserved individual specific *time-invariant* effect[^3]. Now, when you are worried about the unobserved `\(\mu_i\)` being correlated with the other IVs, you would typically use a FE model[^4] to estimate the coefficients of the above model. FE models are either estimated using the least squares dummy variables (LSDV) approach or the within-transformation approach. The two approaches differ in the way it is formulated. 

[^3]: Since `\(\mu_i\)` is time-invariant, it is missing the index `\(t\)`. It represents an effect which remains constant over time for an individual. A good example would be gender of a person.

[^4]: FE models accounts for unobserved individual time-invariant effect and therefore helps address concerns of `\(\mu_i\)` biasing the coefficients on the other IVs.

If you use the LSDV model, you *estimate* the unobserved `\(\mu_i\)` by simply including dummy variables for every single individual in the data. So, the previously unobserved `\(\mu_i\)` is now observed and accounted for. Then, you would end up reformulating the above model as -

`$$lnwg_{it} = \beta_0 + \beta_1lnhr_{it} + \beta_2kids_{it} + \beta_3age_{it} + \beta_4disab_{it} + \sum_{i=1}^{n-1}D_i\mu_i + \epsilon_{it}$$`

where `\(\sum_{i=1}^{n-1}D_i\)`[^5] are the coefficients on the dummy variable for every single individual. Then, you can use linear regression to estimate the coefficients. Below I estimate the coefficients using the LSDV model.

[^5]: Only `\(n-1\)` dummies are included to avoid the dummy variable trap. Alternatively, you can eliminate the intercept `\(\beta_0\)` and include `\(n\)` dummies. 


```r
coefficients(lm(lnwg ~ lnhr + kids + age + disab + factor(id),
                 data = LaborSupply))[c('lnhr','kids','age','disab')]
```

```
##        lnhr        kids         age       disab 
## 0.074541480 0.033882585 0.004430521 0.023418050
```

```r
# using factor(ID) creates dummies for each ID on the fly
# I have omitted the coefficients from the dummy variable
```

On the other hand, if you were to estimate a within model, you perform a transformation to the data - you subtract the 'group' means from the data. By groups here, I mean groups of individuals. The transformation helps to completely eliminate the effect of `\(\mu_i\)`. So, you would end up reformulating the main model as -

`$$lnwg_{it} - \bar{lnwg_{i.}} = \beta_1(lnhr_{it} - \bar{lnhr}_{i.}) + ... + \beta_4(disab_{it}- \bar{disab}_{i.}) + \epsilon_{it}$$`

where `\(\bar{var}_{i.}\)` is the group based mean for a particular variable. By performing such a transformation, we can eliminate the `\(\mu_i\)` term entirely. After that, you can use linear regression to estimate the coefficients. Below I estimate the coefficients of our model using the within technique.


```r
coefficients(plm(lnwg ~ lnhr + kids + age + disab,
                 data = LaborSupply,
                 model = "within",
                 index = c("id","year"),
                 effect = "individual"))
```

```
##        lnhr        kids         age       disab 
## 0.074541480 0.033882585 0.004430521 0.023418050
```

As you can see, the two approaches produce the exact same result. Intuitively, they should because they either remove or account for the effect of `\(\mu_i\)` in the model. But, interestingly the mathematical procedures are completely different. One uses additional dummy variables while the other performs a transformation. Have you wondered why the two procedures give you identical result? In particular, why would including dummies variables be equivalent to using a mean transformation?

The proof of FWL theorem provides the answer. To keep things simple, I will try to explain intuitively rather than using math.

Our first example demonstrates that in order to show the result of the theorem in practice, we first need to find the residuals when we regress `\(y\)` on all IVs baring `\(x_1\)`. Theoretically, FWL proof follows the same strategy - it derives an expression for the residuals. More formally, the mathematical derivation of FWL, involves formulating a so-called 'residual maker'. In matrix notation, the residual maker looks like this : `\(I - W(W^TW)^{-1}W^T\)` where `\(W\)` consists of IVs that are not important to the analysis. 

Applied to our first example, `\(W\)` would represents the matrix containing `\(x_2,x_3,x_4\)` and `\(x_5\)` and `\(I\)` would be an identity matrix[^6] of the same length as `\(y\)`. When this residual maker multiplied with the matrix containing `\(y\)`, it creates the residuals from the regression of `\(y\)` on `\(x_2,x_3,x_4\)` and `\(x_5\)`. 

[^6]: A identity matrix is a matrix where the diagonal positions are 1 and the rest are 0.

To show you that this is true, below I use the residual maker on the first example and show that its output is the same as you would get if you predicted `\(y\)` using OLS and subtracted it from the actual `\(y\)` (res1).


```r
# w consists of x2,x3,x4 and x5
w <- as.matrix(simulated_data[,c('x2','x3','x4','x5')])
# residual maker
d <- diag(length(simulated_data$y)) - w %*% solve((t(w)%*%w)) %*% t(w)
# multiply with y
y_res1 <- d %*% simulated_data$y %>% round(3)
# get residuals from regression
# of y on x2,x3,x4 and x5
betas <- solve( t(w) %*% w ) %*% t(w) %*% simulated_data$y
y_res2 <- (simulated_data$y - w %*% betas) %>% round(3)
# check if y_res1 is same as y_res2
all(y_res1 == y_res2)
```

```
## [1] TRUE
```

They are the same. 

Let's apply the above logic to our FE LSDV model example. Remember, in the LSDV model, we are interested in the coefficients of log hours, number of children, age and bad health. We are not interested in the coefficient of the dummies. So, `\(W\)` would consist of all the dummies for every individual in our data. Below I create `\(W\)` using the dummies and then create the residual maker. 


```r
# create dummy variables for every individual
w <- model.matrix(~as.factor(LaborSupply$id))
# create residual maker
d <- diag(length(LaborSupply$lnwg)) - w %*% solve((t(w)%*%w)) %*% t(w)
```

Next, I multiply this residual maker with our DV - log wage - which gives us our residuals `lmwg_res`. 


```r
# I round the values
lnwg_res <- round(d %*% LaborSupply$lnwg,3)
```

Recall, our interpretation of residuals of `\(y\)` in our first example. The residuals represent a *new* `\(y\)` with the variation of the IVs removed. Similarly, these residuals - `lnwg_res` - represent a new version of log wage with the effect of `\(\mu_i\)` eliminated or netted out. 

Now, let's turn out attention to the within version of FE models that use a transformation to get rid of `\(\mu_i\)`. Specifically, lets apply such a transformation to the DV - `lnwg`. The transformation would be equivalent to subtracting mean log wage of an individual from all the log wages of the same individual in the data. I carry out this transformation below and obtain the transformed log wage.


```r
LaborSupply_dm <-
  LaborSupply %>%
  group_by(id) %>%
   mutate(lnwg_tr = round(lnwg - mean(lnwg),3)) %>%
  ungroup()
```

Intuitively, `lnwg_tr` should be the same as `lnwg_res`. Why? Because the transformation and the residual maker, both remove or net out the effect of `\(\mu_i\)`. Below, I check if they are indeed identical.  


```r
all(LaborSupply_dm$lnwg_tr == lnwg_res)  
```

```
## [1] TRUE
```

Mind blown?

The mathematical operation of subtracting group means is exactly the same as using the residual maker! We have used FWL theorem to show that estimating the coefficients from LSDV model is mathematically equivalent to estimating the same coefficient using the within model. For those who are mathematically inclined the proof of this equivalence is outlined in. 
