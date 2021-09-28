---
layout: post
title:  "The Limitation of Using GBDT for Regression"
date:   2021-09-15 10:00:40
blurb: "When GBDT regression fail to predict accurately"
---

<!-- # The Limitation of Using GBDT for Regression -->

Tree models usually produce good predictions when the time series is smooth, but perform poor when the time series shows a tendency.

That's because tree models do not extrapolate. Take a single-tree as an example, the prediction made by a leaf node is the `mean / medium / min / max /...` values of all trained samples belong to this node, while for the integrated model of GBDT, a sample's prediction is the weighted average of the predictions made by the leaf nodes it falls on in each subtree, which leads to the problem that **the final prediction must be between the maximum and minimum values of the time series.** 

Assume the time series to be [1,2,3,4,5,6, ... . 1000], so the final prediction made by a tree or integrated decision tree will not be greater than 1000 in most cases, which means that once we reach an ever high sales in the future, like 2000 or 3000, the tree would fail to capture this trend in advance.
{:.note title="EXAMPLE"}


So what should we do if we still want to use tree models for extrapolating predictions?  

**One suggestion is to manually construct features that can imply future tendency**. For instance, we can fit a simple linear model first and use the predictions it made as a part of the inputs of the following tree model. However, the method suggested above has its downsides too. In business scenario, we have thousands of goods to predict, it is almost impossible to maintain a linear model for each of them. Besides, the predictions of linear models are easily affected by outliers, which may bring more uncertainty to the tree model. 

**Another solution is to remove the tendency from time series, which involves time series decomposition**. The `statsmodels` library provides an implementation of the naive, or classical, decomposition method in a function called `seasonal_decompose()`. 

~~~python
from statsmodels.tsa.seasonal import seasonal_decompose
series = [...]
result = seasonal_decompose(series, model='additive')
print(result.trend)
print(result.seasonal)
print(result.resid)
print(result.observed)
~~~

## Reference
<https://www.kaggle.com/c/web-traffic-time-series-forecasting/discussion/38352>