---
title: Seeking to Interpret Customer Churn with Survival Analysis
subtitle: Finding how long customers are expected to remain and probable reasons why they may leave
image: https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/survival-analysis-plot.png
---

## What is Survival Analysis?
Survival analysis was first developed by actuaries and medical professionals to predict (as its name implies) how long individuals would survive. However, it has expanded into include many different applications.

Survival analysis it's also referred to as *reliability analysis* in engineering, or it can be referred to more generally as *time-to-event analysis*.

In the general sense, it can be thought of as *a way to model anything with a finite duration* - retention, churn, completion, etc. The culmination of this duration may have a "good" or "bad" (or "neutral") connotation, depending on the situation. Yet it is still most often called survival analysis and the following definitions are still commonly used:

* **birth**: the event that marks the beginning of the time period for observation
* **death**: the event of interest, which then marks the end of the observation period for an individual

## Customer Churn in a Telecomunications Company
Treselle Systems, a data consulting service, [analyzed customer churn data using logistic regression](http://www.treselle.com/blog/customer-churn-logistic-regression-with-r/). For simply modeling whether or not a customer left this can work, but if we want to model the actual tenure of a customer, survival analysis is more appropriate.

Here's are a look of the first five rows of the dataset:

![Telecom dataset first five rows](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/customer-churn-head.png)

The *"tenure"* feature represents the duration that a given customer has been with the company, and *"churn"* represents whether or not that customer left (i.e. the *event*, from a survival analysis perspective). So, any situation where churn is *"no"* means that a customer is still active, and so from a survival analysis perspective the observation is censored (we have their tenure up to now, but we don't know their true duration until event).

We will use this data by Treselle Systems to fit a survival model and answer questions such as:

* What features best model customer churn?
* What would we characterize as the "warning signs" that a customer may discontinue service?
* What actions would we recommend to this business to try to improve their customer retention?

### *Cleaning the data*
First we will need to make sure our dataset is fit for our analysis. We will need to do some get rid of columns that may not be useful, fix missing values, and make sure certain values are in the proper format.

```python
# We can drop 'customerID' column (not needed)
churn2 = churn_data.drop(columns = 'customerID')

# Last two columns should really be numerical
churn3 = churn2.copy()
churn3['MonthlyCharges'] = pd.to_numeric(churn3['MonthlyCharges'])
churn3['TotalCharges'] = pd.to_numeric(churn3['TotalCharges'], 
                                       errors='coerce')

# There are 11 nulls in TotalCharges -> to be replaced with the column mean
TotalCharges_mean = churn3['TotalCharges'].mean()
churn3['TotalCharges'].fillna(TotalCharges_mean, inplace=True)

# Lifelines requires numerical yes and no values.
churn4 = churn3.replace({'Yes':1, 'No':0})
```
Now we will use format categorical columns into numerical ones by using "dummy encoding." Dummy encoding turns our categorical columns into values of ones and zeros to convey the columns information. This process is needed since our algorithms can only process numerical data.

```python
numerical_columns = ['tenure','MonthlyCharges','TotalCharges','Churn']
churn5 = churn4.drop(columns=numerical_columns)
churn6 = pd.get_dummies(churn5)
churn7 = churn4[numerical_columns].join(churn6)
```

Here's a sample of what our dataset looks like now:

![Churn Dataset Encoding Features](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-onehot.png)

### *A Snapshot of the Company's Customer Retation*
This is a sample snapshot of only 80 customers out of more than the 7,000 that are in this dataset.

![Sample Customer Retation Plot](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-data-snapshot.png)

> Blue lines are current customers. Red lines are customers that have been lost.

### *Plotting A Survival Estimate Curve*
```python
time = churn7.tenure.values;
event = churn7.Churn.values;

kmf = lifelines.KaplanMeierFitter();
kmf.fit(time, event_observed=event);
kmf.plot();
plt.title('Probability of still having a customer after x months');
```
![Plot of Survival Estimate](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-survival-plot1.png)

This plot gives us an estimate of how long customers will remain. As expected we see here that the longer someone has been a customer, the higher the likelyhood of leaving.

### *Survival Regression Table*
```python
cph = lifelines.CoxPHFitter(penalizer=0.01)
cph.fit(churn7, 'tenure', event_col='Churn')
cph.print_summary()
```
![Table of Survival Regression](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-survival-regression1.png)

Note that after fitting the model, the following variables have coefficients significantly different from zero: TotalCharges, Partner, and PaperlessBilling. 

### *Plot of log(Hazard Ratio) for each variable*
This plot shows the features that may be significant:
```python
fig, ax = plt.subplots(figsize=(10,20));
cph.plot(ax=ax);
```
![Log Hazard Ratio Table](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-log-plot1.png)

### *Finding Best Predictors of Churn*
Let's plot predictions for several covariate groups to see if they look any different.
```python
cph.plot_covariate_groups('Partner', [0,1]);
cph.plot_covariate_groups('PaperlessBilling', [0,1]);
cph.plot_covariate_groups('MonthlyCharges', [10,20,30,60,80,100,110,120]);
```
![Plots of Best Predictors](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/chrun-predictor-plots1.png)

Definitely looks like higher monthly charges cause customers to leave.

Let's look at "TotalCharges" since customers that have been around longer should have accumulated higher amounts.

![Plot of Total Charges](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-predictor-total1.png)

"TotalCharges" and "tenure" are highly correlated, so we will remove this feature from our model and run these last few plots again.

### Regression Survival Table After Removing "TotalCharges"
Now "Partner" and "PaperlessBilling" are the best predictors. Although, I would say that "SeniorCitizen" and "Dependents" are not far either from being significant features.

![Changed Regression Survival Table Plot](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-survival-regression2.png)

### Changes in Best Predictors Plot
Now even these graphs are more distiguishable in what they reveal:
* In the previews plot the "Partner" feature, "no" partner was above the baseline but now it's reversed.
* There's more separation between the lines in the "PaperlessBilling" graph.

![Changed Best Predictors Plots](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/churn-predictor-plots2.png)

## Answers to Our Questions
#### What features best model customer churn?
* **Partner**: customers with partners seem to remain longer.
* **PaperlessBilling**: to some extent customer with paperless billing tend to leave sooner.
* **TotalCharges**: the higher the total charges, the longer the "tenure" (as expected).

#### What would we characterize as the "warning signs" that a customer may discontinue service?
I would recommend paying particular attention to customers that don't have a partner, those with children, and maybe senior citizens.

#### What actions would you recommend to this business to try to improve their customer retention?
* Targeting married couples with children (not senior) may be worth trying as a marketing/advertising strategy. *It may be that their brand strategy and marketing efforts should all be aligned to target families (couples with children).*
* Carrying out tests among groups that prefer paperless billing by sending various types of correspondance to each.

