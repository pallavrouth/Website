---
title: 'Intro to python'
subtitle: "Python data types and basic operations in python"
excerpt: "Using python as a calculator and variable assignments and computation"
author: Pallav Routh
date: '2021-09-12'
slug: 
  - python-basics
categories:
  - Python
tags:
  - Coding
---



In this blog post I talk about two basic operations you can do with Python : (1) use python as a calculator and (2) variable assignment in python.

Before learning about these basic operations, it might be helpful to refresh your memory on the different types of objects in python. Here is a friendly short video on the most frequently used python types : 

https://campus.datacamp.com/courses/intro-to-python-for-data-science/chapter-1-python-basics?ex=6

## Using python as a calculator

Understanding how to use python as a calculator can be a useful starting point. Simple arithmetic operations are straightforward. We can use the `print()` function to see the output of these arithmetic operations. Below are some examples of using the addition, subtraction, multiplication, division, exponentiation and remainder operator on 2 arbitrary numbers. 


```python
# Addition and subtraction
print(5 + 5)
```

```
## 10
```

```python
print(5 - 5)
# Multiplication and division
```

```
## 0
```

```python
print(3 * 5)
```

```
## 15
```

```python
print(10 / 2)
# Exponentiation
```

```
## 5.0
```

```python
print(4 ** 2)
# Remainder (modulo)
```

```
## 16
```

```python
print(18 % 7)
```

```
## 4
```

## Variable assignment and computations

A common operation in python, is to assign some value to an object so it can be referenced later in some other operation. The syntax is simple : object name `=` object value. Lets do the following excercise to understand how this works. 

Create two objects called `savings` and `income`. Assign the value 100 and 500 to these objects. Then use the print function to see the result.


```python
savings = 100
income = 500
print(savings)
```

```
## 100
```

```python
print(income)
```

```
## 500
```

As I mentioned before, the point of assigning values to certain object names is that you can use it later in another operation. For example, below after I create `savings` and `income`, I can use them later to create another object called `expense` (which is income - savings).


```python
savings = 100
income = 500
expense = income - savings
print(expense)
```

```
## 400
```

Here is another example of this. Let's pretend, we invest our savings in a financial stock which has a 10\% interest rate per annum. Lets try to calculate how much our savings will grow in a years time. 


```r
savings = 100
interest_rate = 10/100
growth_in_savings = savings + savings * interest_rate
print(growth_in_savings)
```

```
## [1] 110
```

In one years time, our savings has grown to \$110. How much is it going to be after n years? For example if n = 2 years, we can use the following formula to calculate the growth in savings -


```python
savings = 100
interest_rate = 10/100
n_years = 2
growth_in_savings = savings + n_years * ( savings * interest_rate ) 
print(growth_in_savings)
```

```
## 120.0
```

We can assign names to ANY kind of python object. The rule is the same : object name `=` object value. Below are some more examples of assigning names to character and boolean value -


```python
interest_type = 'simple interest'
profitable = True
```

Tips : Using the `+` operator to paste together two strings can be very useful in building custom print messages. This is especially useful when you want to paste a string to a object. Below, I saved my name to an object called `my_name`. Then I used the `+` operator to print a sentence that uses `my_name`.


```python
my_name = "Pallav"
print("My name is " + my_name)
```

```
## My name is Pallav
```

Note that if you have assigned a numeric value to an object name, you first need to convert it to a string type. You can easily achieve this using the `str()` function. This is because pasting together strings and numbers is not allowed in Python. Here is an example -


```python
savings = 100
print('My savings are ' + str(savings))
```

```
## My savings are 100
```

Similar functions such as `int()`, `float()` and `bool()` will help you convert Python values into any type.
