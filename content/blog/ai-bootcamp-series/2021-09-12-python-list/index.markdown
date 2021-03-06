---
title: 'Introduction to python lists'
subtitle: "Creating and manipulating lists in python"
excerpt: "Methods for python lists"
author: Pallav Routh
date: '2021-09-13'
slug: 
  - python-basics
categories:
  - Python
tags:
  - Coding
---



This blog post talks about basic operations you can do using one of python's most used objects called lists.

Python lists are very common due to its flexibility. It is typically used to store or assign a number or series of values. For example below, I create a list of prices of all the items in my study room. To create a list simply put all items inside `[]`. You can assign a name to it as well. Below, I create a list of common items I find in my study room. 


```python
laptop = 1200.00
monitor = 100.00
notebook = 3.00
chair = 30.00
table = 100.00
prices = [laptop,monitor,notebook,chair,table]
print(prices)
```

```
## [1200.0, 100.0, 3.0, 30.0, 100.0]
```

Alternatively, you could have also simply used the values itself.  


```python
prices = [1200,100,3,30,100]
print(prices)
```

```
## [1200, 100, 3, 30, 100]
```


Although it's not really common, a list can also contain a mix of Python types including strings, floats, booleans, etc. For example, lets make the list of `prices` above more meaningful by adding names (which are strings).


```python
laptop = 1200.00
monitor = 100.00
notebook = 3.00
chair = 30.00
table = 100.00
prices = ["laptop",laptop,"monitor",monitor,"notebook",notebook,"chair",chair,"table",table]
print(prices)
```

```
## ['laptop', 1200.0, 'monitor', 100.0, 'notebook', 3.0, 'chair', 30.0, 'table', 100.0]
```

Furthermore, we can actually create lists inside lists. For example, we can make our `prices` list more meaningful by creating nested lists such as the following -


```python
laptop = 1200.00
monitor = 100.00
notebook = 3.00
chair = 30.00
table = 100.00
prices = [["laptop",laptop],["monitor",monitor],["notebook",notebook],["chair",chair],["table",table]]
print(prices)
```

```
## [['laptop', 1200.0], ['monitor', 100.0], ['notebook', 3.0], ['chair', 30.0], ['table', 100.0]]
```

## Subsetting lists

Next, lets try to figure out how to extract individual or a collection of elements from the above list. This type of operation is commonly called 'subsetting'. Subsetting is extremely common in various real life applications because often times you need only certain elements from a list (and not the entire list itself). 

The general syntax is : `name of list[index of element]`. Index is simply the position of the item in the list. For example, lets try to extract the first element of `prices` :


```python
laptop = 1200.00
monitor = 100.00
notebook = 3.00
chair = 30.00
table = 100.00
prices = [["laptop",laptop],["monitor",monitor],["notebook",notebook],["chair",chair],["table",table]]
print(prices[0])
```

```
## ['laptop', 1200.0]
```

The index of 0 gives the first element. That's because in python indexing starts at 0,1,2... 

What if you wanted the last element of the list? One way is to count how many elements are there. And then subtract one. For example there are 5 elements in the list `prices`. So, the last element must have index as 5 - 1 = 4 (because remember the indexing starts at 0). 


```python
laptop = 1200.00
monitor = 100.00
notebook = 3.00
chair = 30.00
table = 100.00
prices = [["laptop",laptop],["monitor",monitor],["notebook",notebook],["chair",chair],["table",table]]
print(prices[4])
```

```
## ['table', 100.0]
```

But, there is another way to find the last element without having to manually count the number of elements - Python can also use indexing in the reverse direction. This is called 'negative indexing'. In this type of indexing, the last element has the index of -1. 


```python
laptop = 1200.00
monitor = 100.00
notebook = 3.00
chair = 30.00
table = 100.00
prices = [["laptop",laptop],["monitor",monitor],["notebook",notebook],["chair",chair],["table",table]]
print(prices[-1])
```

```
## ['table', 100.0]
```

So, using `prices[4]` returned the same result as `prices[-1]`.

## Subset and calculate

Remember I said that often times you need only certain elements from a list and not the entire list itself? Here is an example of that. For example what if you wanted to find out the total price of all electronics in our list of `prices`? I would have to first *subset out* the prices of electronics. Then, I can add them. 

We have 2 electronic items in our list `prices` : laptop and monitor. Laptop (being the first item) has an index of 0 whereas monitor (being the second item) has an index of 1 in `prices`. Once we extract them using their index, we can add them -


```python
prices =  [1200,100,3,30,100]
laptop_price = prices[0]
monitor_price = prices[1]
print(laptop_price + monitor_price)
```

```
## 1300
```


## Slicing a list

Slicing has the same philosophy as subsetting - to extract a collection of elements from a list rather than a single item. That is, in slicing we want to subset a range of values. The general syntax is `name of list[start index:end index]`. It is important to remember that when slicing using this syntax, the `end index` *is not* counted. 

In our `prices` list, if we wanted to slice the first to the third element, the `start index` should be 0 and the end index should be 3. Here item at index = 3 won't be counted. So, you are left with 0,1 and 2 - which are the first 3 elements.  


```python
prices =  [1200,100,3,30,100]
print(prices[0:3])
```

```
## [1200, 100, 3]
```

It is not mandatory to supply the `start index` or `end index`. If you leave it unspecified, python assumes you want to slice from the very beginning or to the very end of the list. For example, I can reproduce the same result above by doing this -


```python
prices =  [1200,100,3,30,100]
print(prices[:3])
```

```
## [1200, 100, 3]
```

## Changing a list

Sometimes you may need to alter elements in the list. This is pretty straightforward and another application of where subsetting is useful. For example, in our list of `prices` may be we would like to change 'table' to 'height adjustable table'. First, we find out the index of 'table' in `prices`. Since 'table' is the second last item in `prices`, I am going to use negative indexing. 'table' has an index of -2. Then, we can do this to alter the element 'table' -


```python
prices = ["laptop",laptop,"monitor",monitor,"notebook",notebook,"chair",chair,"table",table]
prices[-2] = "height adjustable table"
print(prices)
```

```
## ['laptop', 1200.0, 'monitor', 100.0, 'notebook', 3.0, 'chair', 30.0, 'height adjustable table', 100.0]
```

Another common reason for changing a list is when we want to add new elements to an existing list. There are a couple of ways of doing this. The first method is pretty straightforward. We first save the new elements (that are to be added) as its own list. Then, we can use the `+` operator to add these new elements to the existing list. For example, if we want to add a new price '500' to our existing list of `prices` we can do this -


```python
prices =  [1200,100,3,30,100]
new_price = [500]
print(prices + new_price)
```

```
## [1200, 100, 3, 30, 100, 500]
```

The other way is to use a *method* called `append()` designed specifically to add elements to lists. Append uses the following general syntax : `name of list.append(value to be added)`. Lets add '500' to `prices` using append -


```python
prices =  [1200,100,3,30,100]
prices.append(500)
print(prices)
```

```
## [1200, 100, 3, 30, 100, 500]
```

This method is particularly useful when using for loops to store elements in a list. I will demonstrate this later in a separate blog post on loops. Finally, you can also remove elements from your list. You can do this with the `del()` function. To delete the first element of `prices` we can do this -


```python
prices = ["laptop",laptop,"monitor",monitor,"notebook",notebook,"chair",chair,"table",table]
del(prices[0])
print(prices)
```

```
## [1200.0, 'monitor', 100.0, 'notebook', 3.0, 'chair', 30.0, 'table', 100.0]
```

## Other common operations on a list

1. Counting the number of repititions of an element

We can use the `count` method to count the number of times an entry appears in a list. Lets say we want to count the number of time '100' appears in `prices`, we can do this -


```python
prices =  [1200,100,3,30,100]
print(prices.count(100))
```

```
## 2
```

2. Sorting the elements in a list

We can use the `sort` method to sort the elements in a list. Lets use this to sort our list of `prices`.


```python
prices =  [1200,100,3,30,100]
prices.sort()
print(prices)
```

```
## [3, 30, 100, 100, 1200]
```

3. Finding the index of an element in the list

We don't need to hand calculate the position or index of an element in a list. We can do that easily using the `index` method. Lets find the index of 30 in `prices`


```python
prices =  [1200,100,3,30,100]
prices.index(30)
```

```
## 3
```

