---
layout: post
title:  "Scaling: preparing continuous data for neural network"
date:   2021-09-28 09:00:40
blurb: "The first step from GBDT to NN"
---

Scaling data is the process of increasing or decreasing the magnitude according to a fixed ratio. By scaling data, **we hope to change the size but not the shape of the data.**

It is a recommended yet not mandatory pre-processing step when working with deep learning neural networks. It helps handling disparities in units. During long processes it definitely helps reduce computational expenses. Sometimes it even determines whether the algorithm can work and how well it works.

This blog is going to talk about feature scaling as well as target scaling, it is more suitable for time series problem, but some of the idea may also work with visual and NLP.

### Table of Contents

* toc
{:toc}

### Why the scale of data matters

Normally speaking, the input data should be scaled when using algorithms involving distance calculation, because differences in the scales across input variables may increase the difficulty of the problem being modeled. 

**In neural network, scaling is even more important when the loss function contains a regular term or using gradient descent methods for optimization.**

For the former, in the linear model `y=wx+b`, any linear transformation of `x` (translation, deflation) can be "absorbed" by `w` and `b`. So in theory, this does not affect the fitting ability of the model. However, if the loss function contains a regular term, such as $$\lambda ∣∣W∣∣^2$$, where $$\lambda$$ is a hyperparameter that imposes penalties with a common ratio on the parameter of each feature $$w_i$$, scaling of data becomes important. Because for a one-dimensional feature $$x_i$$, the larger the scale is, the smaller the coefficient $$w_i$$ would be, thus its weight in the regular term would be smaller, which indicates a smaller penalty on $$w_i$$. As a result, the loss function will relatively ignore those features with larger scale and emphasize those with smaller scale, which is not reasonable. 
**From this perspective, feature scaling is to make the loss function look at each dimensional feature equally.**

For the latter, The gradient descent method updates its parameters as follows:

$$W(t+1)=W(t)-\eta\frac{dE(W)}{dW}$$,

where `E(W)` is the loss function. Convergence implies that a minimal value is obtained and the partial derivative is zero in each dimension. The convergence rate depends on the distance from the initial position of the parameter to the local minima as well as the size of the learning rate $\eta$. The same learning rate is shared by all parameter dimensions (algorithms that assign separate learning rates to each dimension are not considered for the moment). However the descent rate of dimensions are different. In order to converge in each dimension, the learning rate is likely to be set the smallest one of all dimensions at the current position.

### Data Scaling Methods

There are many [methods for data scaling](https://en.wikipedia.org/wiki/Feature_scaling), the majority of them can be divided into two groups, one is to simply divide by the length of the vector itself, the other is to subtract a statistic and then divide by another statistic.

Dividing by length can be seen as some kind of length-independent operation, e.g., word frequency features to remove the effect of article length. It is just like putting all samples to a unit ball. 

The second group, normalization, is more popular in practice. Two of the most widely adopted methods of this group are rescaling (also called min-max normalization) and standardization (z-score normalization).

#### Rescaling
A value is rescaled as follows:
```
y = (x - min) / (max - min)
```
`min` refers to the minimum value of the feature, and `max` refers to the maximum. The `min` in the dividend can be replaced by `mean` of the feature as well. It is realized in the scikit-learn object [MinMaxScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html).

Rescaling is preferred in the following situation:
- when the neural network requires its inputs to be bounded between 0 and 1.
- when the values can only take on a specific range, like in images.

#### Standarization
A standardized value is calculated as follows:
```
y = (x - mean) / standard_deviation
mean = sum(x) / count(x)
standard_deviation = sqrt( sum( (x - mean)^2 ) / count(x))
```
The standardized value reflects how far the point is to the mean value. You can visit the scikit-learn version [StandardScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html) for details.

#### Limitations of rescaling & standardization
As we can see from the formulas above, normalization methods have limitations too. Let's look into the two major steps of normalizing.

First, subtracting a statistic can be seen as choosing a new point (the minimum or the mean) as its origin. Therefore, all the observations would be translated to a new coordinate system. Such translation is beneficial and can be seen as some kind of bias-independent operation if the differences in bias among features have  negative impacts on the subsequent process; **however, if the original feature has a special meaning, translation may remove such characteristics.**

Second, dividing a statistic is similar to scaling the feature along the direction of the coordinate axis. Sometimes it can be regarded as a scale-independent operation. However, scaling uses either the span between the maximum and minimum values or the standard deviation (the average distance to the centroid) for scaling. **The former is sensitive to outliers, while the effect of outliers on the latter is related to the number of outliers and the size of the dataset**.

### Scaling the output

The intrinsic motivation of scaling the output is that, a target variable with a large spread of values, in turn, may result in large error gradient values causing weight values to change dramatically, making the learning process unstable.

Apart from that, scaling the output help interpret the model. When the target variable is scaled, the mean squared error (MSE) is automatically scaled. When looking at the mean absolute scaled error (MASE), we can say that,  MASE>1 indicates that the model performs worse than a constant (naive) prediction.

Another fact to should be taken into consideration is that the scale of the output variable should match the scale of the activation function on the output layer of the network. **It is generally better to choose an output activation function suited to the distribution of the targets than to force the data to conform to the output activation function.**

### Good practice

In the 2018 paper [Applying Deep Learning to Airbnb Search](http://export.arxiv.org/pdf/1810.09591), the authors adopted two methods for scaling. For those features whose distributions resemble a normal distribution，they transform them with standardization; while for those look closer to a power law distribution (long-tailed), they transform them as following:

$$log(\frac{1+feature_val}{1+median})$$

From the instance above, we can see that there is no practice suitable for all scenerios. Problems can be complex and it may not be clear how to best scale all the variable. Whether input variables require scaling and what is the best way of scaling depend on the specifics of the problem and of each variable.

### Reference
[How to use Data Scaling Improve Deep Learning Model Stability and Performance](https://machinelearningmastery.com/how-to-improve-neural-network-stability-and-modeling-performance-with-data-scaling/)

[Is it necessary to scale the target value?](https://stats.stackexchange.com/questions/111467/is-it-necessary-to-scale-the-target-value-in-addition-to-scaling-features-for-re/112152#112152)