---
title: A Deeper Look into Customer Churn with Survival Analysis
subtitle: Finding how long customers are expected to remain and probable reasons why they may leave
image: https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/survival-analysis-plot.png
---

Treselle Systems, a data consulting service, [analyzed customer churn data using logistic regression](http://www.treselle.com/blog/customer-churn-logistic-regression-with-r/). For simply modeling whether or not a customer left this can work, but if we want to model the actual tenure of a customer, survival analysis is more appropriate.

The "tenure" feature represents the duration that a given customer has been with the company, and "churn" represents whether or not that customer left (i.e. the "event", from a survival analysis perspective). So, any situation where churn is "no" means that a customer is still active, and so from a survival analysis perspective the observation is censored (we have their tenure up to now, but we don't know their true duration until event).

Here we will use this data by Treselle Systems to fit a survival model, and answer questions such as:

* What features best model customer churn?
* What would we characterize as the "warning signs" that a customer may discontinue service?
* What actions would we recommend to this business to try to improve their customer retention?

