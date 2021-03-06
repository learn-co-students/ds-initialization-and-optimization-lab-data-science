
# Initialization and Optimization

For this lab on initialization and optimization, let's look at a slightly different type of neural network. This time, we will not perform a classification task as we've done before.  Instead, we'll look at a regression problem.

We can just as well use deep learning networks for regression as for a classification problem. However, note that getting regression to work with neural networks is a harder problem because the output is unbounded ($\hat y$ can technically range from $-\infty$ to $+\infty$, and the models are especially prone to **_exploding gradients_**. This issue makes a regression exercise the perfect learning case!

Run the cell below to import everything we'll need for this lab.


```python
import numpy as np
np.random.seed(0)
import pandas as pd
from keras.models import Sequential
from keras import initializers
from keras.layers import Dense
from keras.wrappers.scikit_learn import KerasRegressor
from sklearn.model_selection import cross_val_score
from sklearn.model_selection import KFold
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn import preprocessing
from keras import optimizers
from sklearn.model_selection import train_test_split
```

## 1. Loading the data

The data we'll be working with is data related to facebook posts published during the year of 2014 on the Facebook's page of a renowned cosmetics brand.  It includes 7 features known prior to post publication, and 12 features for evaluating the post impact. What we want to do is make a predictor for the number of "likes" for a post, taking into account the 7 features prior to posting.

First, let's import the data set and delete any rows with missing data.  

The dataset is contained with the file `dataset_Facebook.csv`. In the cell below, use pandas to read in the data from this file. Because of the way the data is structure, make sure you also set the `sep` parameter to `";"`, and the `header` parameter to `0`. 

Then, use the DataFrame's built-in `.dropna()` function to remove any rows with missing values. 


```python
# load dataset
data = None
```

Now, let's check the shape of our data to ensure that everything looks correct. 


```python
np.shape(data) #Expected Output: (495, 19)
```

And finally, let's inspect the `.head()` of the DataFrame to get a feel for what our dataset looks like. 

## 2. Initialization

### 2.1 Normalize the input data

A big part of Deep Learning is cleaning the data and getting into a shape usable by a neural network.  Let's get some additional practice with this.


Take a look at our input data. We'll use the 7 first columns as our predictors. We'll do the following two things:
- Normalize the continuous variables --> you can do this using `np.mean()` and `np.std()`
- make dummy variables of the categorical variables (you can do this by using `pd.get_dummies`)

We only count "Category" and "Type" as categorical variables. Note that you can argue that "Post month", "Post Weekday" and "Post Hour" can also be considered categories, but we'll just treat them as being continuous for now.

In the cell below, convert the data as needed by normalizing or converting to dummy variables, and then concatenate it all back into a single DataFrame once you've finished.  


```python
X0 = data["Page total likes"]
X1 = data["Type"]
X2 = data["Category"]
X3 = data["Post Month"]
X4 = data["Post Weekday"]
X5 = data["Post Hour"]
X6 = data["Paid"]

## standardize/categorize
X0= NoneNone
dummy_X1= None
dummy_X2= None
X3= None
X4= None
X5= None

# Add them all back into a single DataFrame
X = pd.concat([X0, dummy_X1, dummy_X2, X3, X4, X5, X6], axis=1)

# Store our labels in a separate variable
Y = data["like"]
```


```python
#Note: you get the same result for standardization if you use StandardScaler from sklearn.preprocessing

#from sklearn.preprocessing import StandardScaler
#sc = StandardScaler()
#X0 = sc.fit_transform(X0)
```

Our data is fairly small. Let's just split the data up in a training set and a validation set!

In the cell below:

* Split the data into training and testing sets by passing `X` and `Y` into `train_test_split`.  Set a `test_size` of `0.2`.


```python
X_train, X_val, Y_train, Y_val = None
```

Let's check the shape to make sure everything worked correctly.


```python
X_val.shape # Expected Output: (99, 12)
```


```python
X_train.shape # Expected Output: (396, 12)
```

## Building a Neural Network for Regression

Now, we'll build a neural network to predict the number of likes we think a post will receive.  

In the cell below, create a model with the following specifications:

* 1 Hidden Layer with 8 neurons.  In this layer, also set `input_dim` to `12`, and `activation` to `"relu"`.
* An output layer with 1 neuron.  For this neuron, set the activation to `linear`.  


```python
model = None
```

Now, we need to compile the model, with the following hyperparameters:

* `optimizer='sgd'`
* `loss='mse'`
* `metrics=['mse']`

Note that since our model is training for a regression task, not a classification task, we'll need to use a loss metric that corresponds with regression tasks--Mean Squared Error. 

Finally, let's train the model.  Call `model.fit()`. In addition to to the training data and labels, also set:

* `batch_size=32`
* `epochs=100`
* `verbose=1`
* `validation_data=(X_val, y_val)`

Did you see what happend? all the values for training and validation loss are "nan". There could be several reasons for that, but as we already mentioned there is likely a vanishing or exploding gradient problem.  This means that the values got so large or so small that they no longer fit in memory.   R

Recall that we normalized out inputs. But how about the outputs? Let's have a look.


```python
Y_train.head()
```

Yes, indeed. We didn't normalize them and we should, as they take pretty high values. Let
s rerun the model but make sure that the output is normalized as well!

### 2.2 Normalizing the output

In the cell below, we've included all the normalization code that we wrote up top, but this time, we've added a line to normalize the data in `Y`, as well. This should help alot!

Run the cell below to normalize our data and our labels.


```python
X0 = data["Page total likes"]
X1 = data["Type"]
X2 = data["Category"]
X3 = data["Post Month"]
X4 = data["Post Weekday"]
X5 = data["Post Hour"]
X6 = data["Paid"]

## standardize/categorize
X0= (X0-np.mean(X0))/(np.std(X0))
dummy_X1= pd.get_dummies(X1)
dummy_X2= pd.get_dummies(X2)
X3= (X3-np.mean(X3))/(np.std(X3))
X4= (X4-np.mean(X4))/(np.std(X4))
X5= (X5-np.mean(X5))/(np.std(X5))

X = pd.concat([X0, dummy_X1, dummy_X2, X3, X4, X5, X6], axis=1)

Y = (data["like"]-np.mean(data["like"]))/(np.std(data["like"]))
```

Now, let's split our data into appropriate training and testing sets again.  Split the data, just like we did before.  Use the same `test_size` as we did last time, too. 


```python
X_train, X_val, Y_train, Y_val = None
```

Now, let's reinitialize our model and build it from scratch again.  

**_NOTE:_**  If we don't reinitialize our model, our training would start with the weight values we ended with during the last training session.  In order to start fresh, we need to declare a new `Sequential()` object.  

Build the model with the exact same architecture and hyperparameters as we did above.


```python
model = None
```

Now, compile the model with the same parameters we used before.

And finally, fit the model using the same parameters we did before. 

The model did much, much better this time around!

Run the cell below to get the model's predictions for both the training and validation sets. 


```python
pred_train = model.predict(X_train).reshape(-1)
pred_val = model.predict(X_val).reshape(-1)
```

Let's look at the first 10 predictions from `pred_train`. Display those in the cell below.

Let's manually calculate the Mean Squared Error in the cell below.  

As a refresher, here's the formula for calculating Mean Squared Error:

<img src='mse_formula.gif'>

Use `pred_train` and `Y_train` to calculate our training MSE in the cell below.  

**_HINT:_** Use numpy to make short work of this!


```python
MSE_train = None
MSE_train 
```

Now, calculate the MSE for our validation set in the cell below.


```python
MSE_val = None
MSE_val 
```

### 2.3 Use weight initializers

Another way to increase the performance of our models is to initialize our weights in clever ways.  We'll explore some of those options below.  

#### 2.3.1  He initialization

Let's try and use a weight initializer.  We'll start with the **_He normalizer_**, which initializes the weight vector to have an average 0 and a variance of 2/n, with $n$ the number of features feeding into a layer.

In the cell below:

* Recreate the Neural Network that we created above.  This time, in the hidden layer, set the `kernel_initializer` to `"he_normal"`.
* Compile and fit the model with the same hyperparameters as we used before.  


```python
model = None
```

Great!

Run the cells below to get training and validation predictions are recalculate our MSE for each. 


```python
pred_train = model.predict(X_train).reshape(-1)
pred_val = model.predict(X_val).reshape(-1)

MSE_train = np.mean((pred_train-Y_train)**2)
MSE_val = np.mean((pred_val-Y_val)**2)
```


```python
print(MSE_train) 
print(MSE_val) 
```

The initializer does not really help us to decrease the MSE. We know that initializers can be particularly helpful in deeper networks, and our network isn't very deep. What if we use the `Lecun` initializer with a `tanh` activation?

#### 2.3.2  Lecun initialization

In the cell below, recreate the network again.  This time, set hidden layer's activation to `'tanh'`, and the `kernel_initializer` to `'lecun_normal'`.

Then, fit and compile the model as did before.  


```python
model = None
```

Now, run the cells below to get the predictions and calculate the MSE for training and validation again.  


```python
pred_train = model.predict(X_train).reshape(-1)
pred_val = model.predict(X_val).reshape(-1)

MSE_train = np.mean((pred_train-Y_train)**2)
MSE_val = np.mean((pred_val-Y_val)**2)
```


```python
print(MSE_train) 
print(MSE_val) 
```

## 3. Optimization

Another option we have is to play with the optimizers we choose for gradient descent during our back propagation step.  So far, we've only made use of basic `'sgd'`, or **_Stochastic Gradient Descent_**.  However, there are more advanced optimizers available to use will often converge to better minima, usually in a quicker fashion. 

In this lab, we'll try the two most popular methods: **_RMSprop_** and **_adam_**.

### 3.1 RMSprop

In the cell below, recreate the original network that we built in this lab--no kernel intialization parameter, and the activation set to `'relu'`. 

This time, when you compile the model, set the `optimizer` parameter to `"rmsprop"`.  No changes to the `fit()` call are needed--keep those parameters the same.  


```python
model = None
```

Now, run the cell below to get predictions and compute the MSE again.


```python
pred_train = model.predict(X_train).reshape(-1)
pred_val = model.predict(X_val).reshape(-1)

MSE_train = np.mean((pred_train-Y_train)**2)
MSE_val = np.mean((pred_val-Y_val)**2)
```


```python
print(MSE_train) 
print(MSE_val) 
```

### 3.2 Adam

Another popular optimizer is **_adam_**, which stands for `Adaptive Moment Estimation`. This is an optimzer that was created and open-sourced by a team at OpenAI, and is generally seen as the go-to choice for optimizers today. Adam combines the RMSprop algorithm with the concept of momentum, and is generally very effective at getting converging quickly.  

In the cell below, create the same network that we did above, but this time, set the optimizer parameter to `'adam'`.  Leave all other parameters the same. 


```python
model = None
```


```python
pred_train = model.predict(X_train).reshape(-1)
pred_val = model.predict(X_val).reshape(-1)

MSE_train = np.mean((pred_train-Y_train)**2)
MSE_val = np.mean((pred_val-Y_val)**2)
```


```python
print(MSE_train) 
print(MSE_val) 
```

### 3.3 Learning rate decay with momentum


The final item that we'll get practice with in this lab is implementing a **_Learning Rate Decay_** strategy, along with **_Momentum_**.  We'll accomplish this by creating a `SGD` object and setting learning rate, decay, and momentum parameters at initialization.  In this way, we can then pass in the `SGD` object we've initialized to our specificataions during the compile step, rather than just a string representing an off-the-shelf `'SGD'`  optimizer.  

In the cell below:

* Create a `SGD` optimizer, which can be found in the `optimizers` module.  
    * Set the `lr` parameter to  `0.03`.
    * Set the `decay` parameter to `0.0001`
    * Set the `momentum` parameter to `0.9`.
* Recreate the same network we used during the previous example.  
* Set the optimizer parameter during the compile step to the `sgd` object we created. 
* Fit the model with the same hyperparameters as we used before.  


```python
sgd = None
model = None
```

Finally, run the cell below to calcluate the MSE for our final version of this model and see how a learning rate decay strategy affected the model. 


```python
pred_train = model.predict(X_train).reshape(-1)
pred_val = model.predict(X_val).reshape(-1)

MSE_train = np.mean((pred_train-Y_train)**2)
MSE_val = np.mean((pred_val-Y_val)**2)
```


```python
print(MSE_train) 
print(MSE_val) 
```

## Further reading

https://machinelearningmastery.com/dropout-regularization-deep-learning-models-keras/

https://machinelearningmastery.com/grid-search-hyperparameters-deep-learning-models-python-keras/

https://machinelearningmastery.com/regression-tutorial-keras-deep-learning-library-python/

https://stackoverflow.com/questions/37232782/nan-loss-when-training-regression-network
