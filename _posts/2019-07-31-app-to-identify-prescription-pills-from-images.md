---
title: An App to Scan and Identify Your Prescription Pill
subtitle: Using SQL, Machine Learning and AWS to match picture of your pill
image: https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-sample-pill.JPG
---

A group of developers sought to build an app with a functionality that would let users scan their prescription pills by taking a picture with their device. After scannig the pill we would need to return a match for the scanned medication from a database we did not have in hand.

![RxId Landing Page](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-landing-page.png)

Two of us were given 16 days to give this a try. After looking around we found [Pillbox's API from the National Library of Medicine](https://github.com/carlos-gutier/carlos-gutier.github.io/tree/master/_posts) which for greater flexibility we turned into our own SQL database stored on [AWS RDS](https://aws.amazon.com/rds/) using PostgreSQL.

