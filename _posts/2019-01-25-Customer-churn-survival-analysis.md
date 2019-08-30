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

The *"tenure"* feature represents the duration that a given customer has been with the company, and *"churn"* represents whether or not that customer left (i.e. the *event*, from a survival analysis perspective). So, any situation where churn is *"no"* means that a customer is still active, and so from a survival analysis perspective the observation is censored (we have their tenure up to now, but we don't know their true duration until event).

Here we will use this data by Treselle Systems to fit a survival model and answer questions such as:

* What features best model customer churn?
* What would we characterize as the "warning signs" that a customer may discontinue service?
* What actions would we recommend to this business to try to improve their customer retention?



