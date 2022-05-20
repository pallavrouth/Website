---
title: 'Advanced Pandas'
subtitle: "Dplyr style coding in Pandas"
excerpt: "Using Pandas to chain operations"
author: Pallav Routh
date: '2021-09-12'
slug: 
  - python-advanced
categories:
  - Python
tags:
  - Coding
---



In this blog post, I demonstrate how to replicate `dplyr` style data manipulation in `pandas`. A characteristic feature of `dplyr` is its ability to chain together multiple operations using the infamous `%>%` operator. I will show `dplyr` and `pandas` code side by side which will further highlight similarities and differences between the two packages. Hopefully, this helps users migrate from `dplyr` to `pandas`.   

## Loading pandas and dplyr


```python
import pandas as pd
import numpy as np
```


```r
suppressPackageStartupMessages(library(dplyr))
```


## Data

We are going to use the popular `cigarettes` dataset. Let's read in the data set.


```python
data = pd.read_csv('https://raw.githubusercontent.com/pallavrouth/AI-Bootcamp/main/Data/cigarettes.csv', index_col = 'Unnamed: 0')
```

Here is quick overview -


```r
glimpse(py$data)
```

```
## Rows: 96
## Columns: 9
## $ state      <chr> "AL", "AR", "AZ", "CA", "CO", "CT", "DE", "FL", "GA", "IA",…
## $ year       <dbl> 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985, 1985,…
## $ cpi        <dbl> 1.076, 1.076, 1.076, 1.076, 1.076, 1.076, 1.076, 1.076, 1.0…
## $ population <dbl> 3973000, 2327000, 3184000, 26444000, 3209000, 3201000, 6180…
## $ packs      <dbl> 116.4863, 128.5346, 104.5226, 100.3630, 112.9635, 109.2784,…
## $ income     <dbl> 46014968, 26210736, 43956936, 447102816, 49466672, 60063368…
## $ tax        <dbl> 32.50000, 37.00000, 31.00000, 26.00000, 31.00000, 42.00000,…
## $ price      <dbl> 102.18167, 101.47500, 108.57875, 107.83734, 94.26666, 128.0…
## $ taxs       <dbl> 33.34834, 37.00000, 36.17042, 32.10400, 31.00000, 51.48333,…
```

## Chaining in pandas

To demonstrate chaining in pandas, lets perform the following operation on the dataset : select 3 columns and filter rows corresponding to certain states. 

In R:


```r
py$data %>% 
  select(state,year,population) %>% 
  filter(state %in% c('AL','AZ','VT')) %>% 
  head(.,5)
```

```
##    state year population
## 1     AL 1985    3973000
## 3     AZ 1985    3184000
## 44    VT 1985     530000
## 49    AL 1995    4262731
## 51    AZ 1995    4306908
```

In python:


```python
(
  data
    .loc[:,['state','year','population']] # <-- select
    .loc[lambda d : d.state.isin(['AL','AZ','VT'])] # <-- filter
    .head()
)
```

```
##    state  year  population
## 1     AL  1985   3973000.0
## 3     AZ  1985   3184000.0
## 44    VT  1985    530000.0
## 49    AL  1995   4262731.0
## 51    AZ  1995   4306908.0
```

In python, you need to do two things to employ chaining : (1) use a parenthesis to enclose the set of operations and (2) use the '.' symbol at the end of each operation as a replacement for `%>%`. 

Now, lets look at some common set of operations in `dplyr`.

## Select, filter and sort

In R:


```r
py$data %>% 
  select(state,year,population) %>% 
  filter(state %in% c('AL','AZ','VT') & population > 10e3) %>% 
  arrange(population) %>% 
  head(.,5)
```

```
##    state year population
## 44    VT 1985     530000
## 92    VT 1995     582827
## 3     AZ 1985    3184000
## 1     AL 1985    3973000
## 49    AL 1995    4262731
```

In python:


```python
(data
  .loc[:,['state','year','population']] # <-- select
  .loc[lambda d : (d.state.isin(['AL','AZ','VT'])) & (d.population > 10e3)] 
  .sort_values(['population'])
  .head()) 
```

```
##    state  year  population
## 44    VT  1985    530000.0
## 92    VT  1995    582827.0
## 3     AZ  1985   3184000.0
## 1     AL  1985   3973000.0
## 49    AL  1995   4262731.0
```

For multiple selections, I prefer using `query` instead of `loc`.


```python
(data
  .loc[:,['state','year','population']] # <-- select
  .query('(population > 10e3) and (state in ["AL","AZ","VT"])') # <-- filters
  .sort_values(['population'])
  .head())
```

```
##    state  year  population
## 44    VT  1985    530000.0
## 92    VT  1995    582827.0
## 3     AZ  1985   3184000.0
## 1     AL  1985   3973000.0
## 49    AL  1995   4262731.0
```


## Mutate and group by

Let's group by and find maximum within each group.

In R: 


```r
py$data %>% 
  mutate(population_1000 = population / 1000) %>% 
  group_by(state) %>% 
    summarize(max(population_1000)) %>% 
  ungroup() %>% 
  head(.,5)
```

```
## # A tibble: 5 × 2
##   state `max(population_1000)`
##   <chr>                  <dbl>
## 1 AL                     4263.
## 2 AR                     2480.
## 3 AZ                     4307.
## 4 CA                    31494.
## 5 CO                     3738.
```

In python:


```python
(data[['state','year','population']] # <-- select
  .assign(population_1000 = lambda d : d.population / 1000) # <-- mutate
  .groupby('state') # <-- groupby
    .population_1000 # <-- apply summarize to this col
    .max()
  .head())
```

```
## state
## AL     4262.731
## AR     2480.121
## AZ     4306.908
## CA    31493.524
## CO     3738.061
## Name: population_1000, dtype: float64
```

Using `aggregate` instead


```python
(data[['state','year','population','packs']]
  .assign(population_1000 = lambda d: d.population / 1000)
  .groupby('state')
    .aggregate({'population_1000':'max'})
  .head())
```

```
##        population_1000
## state                 
## AL            4262.731
## AR            2480.121
## AZ            4306.908
## CA           31493.524
## CO            3738.061
```

## Group by and mutate

In R:


```r
py$data %>% 
  select(state,year,population,packs) %>% 
  mutate(population_1000 = population / 1000) %>% 
  group_by(state) %>% 
    mutate(mean_packs = mean(packs)) %>% 
  ungroup() %>% 
  head(.,5)
```

```
## # A tibble: 5 × 6
##   state  year population packs population_1000 mean_packs
##   <chr> <dbl>      <dbl> <dbl>           <dbl>      <dbl>
## 1 AL     1985    3973000  116.            3973      109. 
## 2 AR     1985    2327000  129.            2327      120. 
## 3 AZ     1985    3184000  105.            3184       88.2
## 4 CA     1985   26444000  100.           26444       78.6
## 5 CO     1985    3209000  113.            3209       97.8
```


In python:


```python
(data[['state','year','population','packs']]
  .assign(population_1000 = lambda d : d.population / 1000,
          mean_packs = lambda d : d.groupby('state')  
                                      .packs 
                                      .transform(lambda x : x.mean()))
  .head())
```

```
##   state  year  population       packs  population_1000  mean_packs
## 1    AL  1985   3973000.0  116.486282           3973.0  108.785858
## 2    AR  1985   2327000.0  128.534592           2327.0  119.788780
## 3    AZ  1985   3184000.0  104.522614           3184.0   88.238392
## 4    CA  1985  26444000.0  100.363037          26444.0   78.611172
## 5    CO  1985   3209000.0  112.963539           3209.0   97.773232
```

Alternatively,


```python
(data[['state','year','population','packs']]
  .assign(population_1000 = lambda d : d.population / 1000,
          mean_packs = lambda d : d.groupby('state')[['packs']]
                                      .transform(lambda x : x.mean()))
  .head())
```

```
##   state  year  population       packs  population_1000  mean_packs
## 1    AL  1985   3973000.0  116.486282           3973.0  108.785858
## 2    AR  1985   2327000.0  128.534592           2327.0  119.788780
## 3    AZ  1985   3184000.0  104.522614           3184.0   88.238392
## 4    CA  1985  26444000.0  100.363037          26444.0   78.611172
## 5    CO  1985   3209000.0  112.963539           3209.0   97.773232
```

## Conditional mutate - ifelse

In R:


```r
py$data %>% 
  select(state,year,population,packs) %>% 
  mutate(population_1000 = population / 1000,
         ifelsecol = ifelse(packs > 120,"> 120","< 120")) %>% 
  head(.,5)
```

```
##   state year population    packs population_1000 ifelsecol
## 1    AL 1985    3973000 116.4863            3973     < 120
## 2    AR 1985    2327000 128.5346            2327     > 120
## 3    AZ 1985    3184000 104.5226            3184     < 120
## 4    CA 1985   26444000 100.3630           26444     < 120
## 5    CO 1985    3209000 112.9635            3209     < 120
```


In python:


```python
(data[['state','year','population','packs']]
  .assign(population_1000 = lambda d : d.population / 1000,
          ifelsecol = lambda d : np.where(d.packs > 120, "> 120","< 120"))
  .head())
```

```
##   state  year  population       packs  population_1000 ifelsecol
## 1    AL  1985   3973000.0  116.486282           3973.0     < 120
## 2    AR  1985   2327000.0  128.534592           2327.0     > 120
## 3    AZ  1985   3184000.0  104.522614           3184.0     < 120
## 4    CA  1985  26444000.0  100.363037          26444.0     < 120
## 5    CO  1985   3209000.0  112.963539           3209.0     < 120
```


In my humble opinion, I think `dplyr` syntax is way more intuitive than `pandas`.
