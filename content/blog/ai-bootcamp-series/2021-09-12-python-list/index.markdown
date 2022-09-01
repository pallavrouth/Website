---
title: 'Python data science workflow'
subtitle: "Steps for executing a typical predictive model in python."
excerpt: "This blogpost gives an overview of the typical steps one uses in a data science project."
author: Pallav Routh
date: '2021-09-13'
slug: 
  - python-advanced
categories:
  - Python
tags:
  - prediction
  - sklearn
  - pandas
  - tensorflow
---



In this blog post, I will outline the typical steps in a prediction task when using python. In general, a prediction task involves 5 steps - (1) data import, (2) data preprocessing, (3) splitting the dataset, (4) build model on train data, and fine tune (5) deploy model on test data and check for accuracy. In this post, I use a vanilla neural network but you can replace it will any other model. 

First lets import the required modules. 


```python
import tensorflow as tf # for NN
import numpy as np # for processing
import pandas as pd # for import
from sklearn.model_selection import train_test_split # for splitting data
from sklearn.compose import ColumnTransformer # for processing
from sklearn.preprocessing import StandardScaler, OrdinalEncoder # for processing
```

## Data Import

Let's import a sample data set. I have uploaded this data set to my github page. Below is a glimpse of the data -


```python
cars_df = pd.read_csv('https://raw.githubusercontent.com/dsnair/ISLR/master/data/csv/Carseats.csv')
cars_df.head()
```

```
##    Sales  CompPrice  Income  Advertising  ...  Age  Education Urban   US
## 0   9.50        138      73           11  ...   42         17   Yes  Yes
## 1  11.22        111      48           16  ...   65         10   Yes  Yes
## 2  10.06        113      35           10  ...   59         12   Yes  Yes
## 3   7.40        117     100            4  ...   55         14   Yes  Yes
## 4   4.15        141      64            3  ...   38         13   Yes   No
## 
## [5 rows x 11 columns]
```

Lets check the column names and types -


```python
cars_df.dtypes
```

```
## Sales          float64
## CompPrice        int64
## Income           int64
## Advertising      int64
## Population       int64
## Price            int64
## ShelveLoc       object
## Age              int64
## Education        int64
## Urban           object
## US              object
## dtype: object
```

```python
cars_df.columns
```

```
## Index(['Sales', 'CompPrice', 'Income', 'Advertising', 'Population', 'Price',
##        'ShelveLoc', 'Age', 'Education', 'Urban', 'US'],
##       dtype='object')
```

## Data splitting

Now lets split this data into training and testing. 


```python
train_data, test_data = train_test_split(cars_df, test_size=0.2, random_state=42)
```

## Data preprocessing

Now, let's preprocess the data. Note that this step is sometimes interchangeable with the previous one. 

Below I have created a small function that takes in a data and returns a list with columns that I intent to use as features and the column that I intend to use as target.


```python
def get_feats_and_labels(data, label):
  """ Take data and label as inputs, return features and labels separated """
  data_feats = data.drop(label, axis=1)
  data_label = data[label]
  return data_feats, data_label
```

I run this model by specifying that I intend to use `Sales` as my target and the remaining variables as features.


```python
train_feats, train_label = get_feats_and_labels(train_data, 'Sales')
```

I have two preprocessing tasks. One, I want to standardize all my continuous feature variables (a common prerequisite for neural network tasks) and two, I want to create dummies for my ordinal/categorical variables (a common preprocessing step for any machine learning algorithm). 

I use these two functions from `sklearn` to initialize the preprocessors. 


```python
scaler = StandardScaler() # normalization engine
encoder = OrdinalEncoder() # encoder engine
```

Before I apply the preprocessors lets make a list of all columns that are numerical or continuous and all columns that are categorical.


```python
train_feats.columns
```

```
## Index(['CompPrice', 'Income', 'Advertising', 'Population', 'Price',
##        'ShelveLoc', 'Age', 'Education', 'Urban', 'US'],
##       dtype='object')
```

```python
num_feats = ['CompPrice','Income','Advertising','Population','Price','Age','Education']
cat_feats = ['ShelveLoc','Urban','US']
```

Now lets specify appropriate preprocessors to the appropriate columns using `ColumnTransformer()` function -


```python
final_pipe = ColumnTransformer([           
   ('num', scaler, num_feats),  
   ('cat', encoder, cat_feats)])
```

Now we can apply the preprocessors using the `fit_transform()` function -


```python
training_data_prepared = final_pipe.fit_transform(train_feats)
```

## Fit training model

Now that we have preprocessed our training data, we can fit our NN model. The fitting process usually involves fine tuning the model. Below, I initialize my base NN model with some reasonable hyper parameters. 


```python
input_shape = training_data_prepared.shape[1:]

model = tf.keras.Sequential([
    tf.keras.layers.Dense(128, activation = 'relu', input_shape = (10,)),
    tf.keras.layers.Dense(128, activation = 'relu'),
    tf.keras.layers.Dense(1, activation = 'relu')])
```

### Model fine tuning

Next step is to take the base model and fine tune the hyper parameters. Different machine learning models using different criteria to fine tune their hyper parameters. I use mean square error of my training predictions in this example.


```python
model.compile(loss = 'mean_squared_error', optimizer = 'adam')
model.summary()
```

```
## Model: "sequential"
## _________________________________________________________________
## Layer (type)                 Output Shape              Param #   
## =================================================================
## dense (Dense)                (None, 128)               1408      
## _________________________________________________________________
## dense_1 (Dense)              (None, 128)               16512     
## _________________________________________________________________
## dense_2 (Dense)              (None, 1)                 129       
## =================================================================
## Total params: 18,049
## Trainable params: 18,049
## Non-trainable params: 0
## _________________________________________________________________
```

We use the best hyper parameters to fit our final model. 


```python
model.fit(x = training_data_prepared,y = train_label,epochs = 10)
```

```
## Epoch 1/10
## 
 1/10 [==>...........................] - ETA: 3s - loss: 61.6900
10/10 [==============================] - 0s 796us/step - loss: 47.6068
## Epoch 2/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 30.8439
10/10 [==============================] - 0s 908us/step - loss: 27.1519
## Epoch 3/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 16.0433
10/10 [==============================] - 0s 929us/step - loss: 10.4883
## Epoch 4/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 7.8071
10/10 [==============================] - 0s 2ms/step - loss: 7.7415
## Epoch 5/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 8.6408
10/10 [==============================] - 0s 765us/step - loss: 6.2252
## Epoch 6/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 4.6321
10/10 [==============================] - 0s 905us/step - loss: 5.2828
## Epoch 7/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 4.0395
10/10 [==============================] - 0s 766us/step - loss: 4.8578
## Epoch 8/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 3.5016
10/10 [==============================] - 0s 845us/step - loss: 4.5123
## Epoch 9/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 4.8328
10/10 [==============================] - 0s 724us/step - loss: 4.3551
## Epoch 10/10
## 
 1/10 [==>...........................] - ETA: 0s - loss: 4.6914
10/10 [==============================] - 0s 956us/step - loss: 4.2242
## <keras.callbacks.History object at 0x7f811b714828>
```

## Model testing 

This is the final step which usually involves taking the best fit training model and using that model to predict using my test set. The test set resembles a data which we have not encountered yet. Therefore, the error in predictions summarizes how well the model will perform with a newer dataset.


```python
test_feats, test_label = get_feats_and_labels(test_data, 'Sales')
test_data_prepared = final_pipe.transform(test_feats)
```


```python
model.evaluate(test_data_prepared, test_label) 
```

```
## 
1/3 [=========>....................] - ETA: 0s - loss: 6.3869
3/3 [==============================] - 0s 983us/step - loss: 5.8923
## 5.89229154586792
```

