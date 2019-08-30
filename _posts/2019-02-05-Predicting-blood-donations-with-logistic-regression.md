---
title: Simple Predictions that Can Improve on Supply Chain
subtitle: Predicting blood donations with logistic regression
image: https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/clutch-preview.png
---

> **Good data-driven systems for tracking and predicting donations and supply needs can improve the entire supply chain, making sure that more patients get the blood transfusions they need.**

This [dataset](https://archive.ics.uci.edu/ml/datasets/Blood+Transfusion+Service+Center) is from a mobile blood donation vehicle in Taiwan. The Blood Transfusion Service Center drives to different universities and collects blood as part of a blood drive.

Our goal is to predict the last column, whether the donor made a donation in March 2007, using information about each donor's history. We'll measure success using recall score as the model evaluation metric.

This will be a simple example of what we can do with small size databases that have few columns; for example, small businesses that have data about their suppliers and their delieveries of certain product.

## Looking Into Our Dataset
```python
import pandas as pd

df = pd.read_csv('https://archive.ics.uci.edu/ml/machine-learning-databases/blood-transfusion/transfusion.data')

# Using more descriptive name for our columns
df = df.rename(columns={
    'Recency (months)': 'months_since_last_donation', 
    'Frequency (times)': 'number_of_donations', 
    'Monetary (c.c. blood)': 'total_volume_donated', 
    'Time (months)': 'months_since_first_donation', 
    'whether he/she donated blood in March 2007': 'made_donation_in_march_2007'
})
```
![Donor dataset first five rows](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-head.png)

## Begin with "Majority Class Baseline"
We want a baseline metric that we can use to compare our future prediction model against. The simplest prediction we can make in this case is select the category that has the most observation. 

*If we would guess that no one made donations in the month of March we would be 76% accurate* (not bad for just guessing, right?). Well, any model that we may build is can be considered useless if it cannot surpass this minimum "prediction" baseline.
```python
import numpy as np
from sklearn.metrics import accuracy_score, classification_report

y_val = df['made_donation_in_march_2007']
majority_class = y_val.mode()[0]
y_pred = np.full(shape=df.shape[0], fill_value=majority_class)

print(accuracy_score(y_val, y_pred))
```
![Majority baseline for model](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-baseline1.png)

```python
df.made_donation_in_march_2007.value_counts(normalize=True)
```
![Same majority baseline using value_counts method](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-baseline2.png)

According to model evaluation metrics with this baseline we achieved a *recall* of 1.0 for "0" ("did not donate" class ) and a recall of 0.0 for the "1" class ("did donate" class). Meaning we correctly retrieve all the cases of no donation but we fail to retrieve any cases of donations.

We can use Sci-kit learn's classification report method to display our prediction report metrics:
```python
import numpy as np
from sklearn.metrics import classification_report

y_pred = np.full(df.made_donation_in_march_2007.shape,
                df.made_donation_in_march_2007.mode()[0])


print(classification_report(df.made_donation_in_march_2007, y_pred))
```
## Check for Outliers or Collinearity with Pairplot
```python
import seaborn as sns
import matplotlib.pyplot as plt
sns.set(style="ticks", color_codes=True)
# y_vars and x_vars are lists of column names.
sns.pairplot(data=df, y_vars=['made_donation_in_march_2007'], x_vars=X.columns)
plt.show()
```
The only feature that seems to have values as possible outliers is "months_since_last_donation." It may be wise to normalize the values of our features with a scalar that accounts for outliers.
![Pairplot of dataset](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-pairplot.png)

## Split Data for Training and Testing
We will split the data into X_train, X_test, y_train, y_test, with random shuffle. (We will include 75% of the data for training our classification model, and hold out 25% of the data to test the predictions after our model has been trained.)

```python
from sklearn.model_selection import train_test_split

# Defining X (independent variables) and Y (dependent variable/target)
X = df.drop(columns='made_donation_in_march_2007')
y = df['made_donation_in_march_2007']

# Splitting data into train & test sets
# Shuffle parameter is True by default with sklearn's train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=0)

# Checking shape for each set
X_train.shape, y_train.shape, X_test.shape, y_test.shape
```
![Shape of train and test data sets](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-shape.png)

## Make a Pipeline
This will our pipeline to conveniently preprocess our data will include: 
* Normalizing our features with [Scikit-learn's RobustScalar](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.preprocessing)
* Feature selection with [SelectKBest(f_classif)](https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.f_classif.html)
* Classification with [LogisticRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)

```python
from sklearn.feature_selection import f_regression, SelectKBest
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import GridSearchCV
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import RobustScaler
from sklearn.preprocessing import StandardScaler

# Making a pipeline ('pipe')
pipe = make_pipeline(
    RobustScaler(), # RobustScaler is robust with outliers!
    SelectKBest(f_regression),
    LogisticRegression(solver='lbfgs'))
```

## Doing a Grid Search Cross-Validation
We will do GridSearchCV with our pipeline. Using 5 folds and recall score.

Parameters for our grid:
* SelectKBest: 
  - k:1, 2, 3, 4

* LogisticRegression
  - class_weight : None, 'balanced'
  - C : .0001, .001, .01, .1, 1.0, 10.0, 100.00, 1000.0, 10000.0
  
      'selectkbest__k': [1, 2, 3, 4],
    'logisticregression__class_weight': [None, 'balanced'],
    'logisticregression__C': [.0001, .001, .01, .1, 1.0, 10.0, 100.00, 1000.0, 10000.0]
}

```python
# Fit on the train set, with grid search cross-validation
gs = GridSearchCV(pipe, param_grid=param_grid, cv=5, 
                  scoring='recall', # using accuracy score to compare w/baseline
                  verbose=False)

gs.fit(X_train, y_train)
```
## Display best score and parameters
```python
validation_score = gs.best_score_ # <-- BEST SCORE!

# flipping validation score from negative to positive with neg sign on front
print('Cross-Validation Score:', validation_score) 
print()
print('Best estimator:', gs.best_estimator_)
```
![Model training score](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-trainscore1.png)

In other words, the best parameters for our logistic regression are:
* C (regularization strength): 0.0001
* Class weight: balanced
* SelectKBest: k=1 (the logistic regression performs best when it ignores all the other features and uses only the top one)

And so far according based on our trainning score (0.79), we are doing better than our "majority class baseline" (0.76). Moreover, this will need stand after testing our model with our test set.

### Testing Our Model
```python
test_score = gs.score(X_test, y_test)
print('Test Score:', test_score)
```
![Model test score](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-testscore1.png)

As we can see, our model did not do better than our baseline. We might do better just guessing instead of going with our model.

### Feature Engineering to Improve On Our Predictions
Many times the performance of our model does not necessarily depend on the algorithm we chose to work with, but on generating aditional useful features from the ones we already have.

Early on in our process we noticed that "months_since_last_donation" had some numbers that were high, meaning there were some donors that had not come for donations in a while. Let's create a new feature that gives more weight to those that have not donated for a long time by "squaring" the values in that column.

```python
df1 = df.copy()

# We add a new feature that squares "months_since_last_donation"
df1['lag_squared'] = df1.months_since_last_donation**2

# Defining X and Y
X1 = df1.drop(columns='made_donation_in_march_2007')
y1 = df1['made_donation_in_march_2007']

# Splitting data into train & test
# Shuffle parameter is True by default with sklearn's train_test_split
X_train1, X_test1, y_train1, y_test1 = train_test_split(
    X1, y1, test_size=0.25, random_state=0)
    
# Making a sencond pipeline using PCA
# PCA allows us to decide how many components to retain
from sklearn.decomposition import PCA
# Removing SelectKBest, since PCA allows us to decide how many components to retain.  
# It should achieves basically the same result.

param_grid2 = {
    'pca__n_components': [1,2,3,4,5],
    'logisticregression__class_weight': [None, 'balanced'],
    'logisticregression__C': [.0001, .001, .01, .1, 1.0, 10.0, 100.00, 1000.0, 10000.0]
}

pipe2 = make_pipeline(
    RobustScaler(),
    PCA(),
    LogisticRegression(solver='lbfgs'))

gs2 = GridSearchCV(pipe2, param_grid=param_grid2, cv=5, 
                  scoring='recall', 
                  verbose=1)

gs2.fit(X_train1, y_train1);
gs2.best_score_ 
```
![Training score for second model](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-trainscore2.png)

This is similar to our previous training score. It's a matter of making sure it does better than our baseline now, at at least below we can see that PCA included two parameters (compared to only one we feature being used by SelectKBest).

```python
gs3.best_params_
```
![Best model parameters with PCA](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-params2.png)

And here's our new test score:
```python
test_score = gs2.score(X_test1, y_test1)
print('Test Score:', test_score)
```
![Test score for second model](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/donor-data-testscore2.png)
