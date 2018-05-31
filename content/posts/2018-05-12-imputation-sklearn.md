---
title: "How does imputation work"
date: 2018-05-12T12:00:00-04:00
draft: false
tags: [ "python" , "imputation", "sklearn" ]
categories: [ "machine learning", "data science", "data processing"]
---

# How does imputation work?

Imputation is a pre-processing technique to handle missing data before fitting the data into model. One of the simplest implementation is to estimate the missing values using the mean/median or the most frequent value of the row/ column where the missing values are located. There are more advanced ways to impute data like regressions, multiple imputation etc. 

## Imputer in sklearn

The  `Imputer` function in the `sklearn` library is commonly used in various tutorials. `sklearn` use the simplest implementation with 3 different strategies: mean/ median/ most frequent value and it can be applied on either row or column.

Example: let's say we fit and transform 3x3 training data set: [[1, 2, 3], [np.nan, 5, 6], [7, 8, np.nan]]

```python
import numpy as np
from sklearn.preprocessing import Imputer

imp = Imputer(missing_values='NaN', strategy='mean', axis=0) # mean on col
imp2 = Imputer(missing_values='NaN', strategy='mean', axis=1) # mean on row

X_train = [[1, 2, 3], [np.nan, 5, 6], [7, 8, np.nan]]

print(imp.fit_transform(X_train))
print(imp2.fit_transform(X_train))

> [[ 1.   2.   3. ]
  [ 4.   5.   6. ] # nan replaced by (1+7)/2 = 4
  [ 7.   8.   4.5]] # nan replaced by (3+6)/2 = 4.5
> [[ 1.   2.   3. ]
  [ 5.5  5.   6. ] # nan replaced by (5+6)/2 = 5.5
  [ 7.   8.   7.5]] # nan replaced by (7+8)/2 = 7.5
```



We can see in the first imp, the missing value are replaced by 4 and 4.5, it is based on the mean of the same column where in imp2 it is based on row.

In practice, we would fit the imputation model based on training data, in order to predict on test set, we need to perform the exact imputation before fitting into model, hence we use the same my_imputer with the parameter fit by X_train and transform on X_test

```python
X_test = [[np.nan, 2, 4], [6, np.nan, 5], [np.nan, 8, 9]]
print(imp.transform(X_test)) 
print(imp2.transform(X_test))

> [[ 4.  2.  4.] # nan replaced by (1+7)/2 = 4 (from X_train)
  [ 6.  5.  5.] # nan replaced by (2+8)/2 = 5 (from X_train)
  [ 4.  8.  9.]] # nan replaced by (1+7)/2 = 4 (from X_train)
> [[ 3.   2.   4. ] # nan replaced by (2+4)/2 = 3 (from X_test)
  [ 6.   5.5  5. ] # nan replaced by (6+5)/2 = 5.5 (from X_test)
  [ 8.5  8.   9. ]] # nan replaced by (8+9)/2 = 8.5 (from X_test)

X_extreme = [[np.nan, 20, 40], [60, np.nan, 50], [np.nan, 8, 9]]
print(imp.transform(X_extreme)) 
print(imp2.transform(X_extreme)) 

> [[  4.  20.  40.] # nan replaced by (1+7)/2 = 4 (from X_train)
  [ 60.   5.  50.] # nan replaced by (2+8)/2 = 5 (from X_train)
  [  4.   8.   9.]] # nan replaced by (1+7)/2 = 4 (from X_train)
> [[ 30.   20.   40. ] # nan replaced by (20+40)/2 = 30 (from X_test)
  [ 60.   55.   50. ] # nan replaced by (60+50)/2 = 55 (from X_test)
  [  8.5   8.    9. ]] # nan replaced by (8+9)/2 = 8.5 (from X_test)
```



Interestingly, even we fit X_train on column mean, the mean is still computed from data in X_test only. That means the fit() step has no effects on row mean axis = 1.

*Note: fit_transform() is equalivalent to fit() and transform(). ref: [SO](https://datascience.stackexchange.com/questions/12321/difference-between-fit-and-fit-transform-in-scikit-learn-models)

