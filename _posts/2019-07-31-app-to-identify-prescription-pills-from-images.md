---
title: An App to Scan and Identify Your Prescription Pill
subtitle: Using SQL, Machine Learning and AWS to match picture of your pill
image: https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-sample-pill.JPG
---

A group of developers sought to build an app with a functionality that would let users scan their prescription pills by taking a picture with their device. After scannig the pill we would need to return a match for the scanned medication from a database we did not have in hand.

![RxId Landing Page](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-landing-page.png)

Two of us were given 16 days to give this a try. After looking around we found [Pillbox's API from the National Library of Medicine](https://github.com/carlos-gutier/carlos-gutier.github.io/tree/master/_posts) which for greater flexibility we turned into our own SQL database stored on [AWS RDS](https://aws.amazon.com/rds/) using PostgreSQL.

Putting getting the database and creating an API for the developers to work with was the major task in this project. Here's part of the code use used to query the SQL database and obtain results:
```python
#  _____ query and return SQL data ______________
def query_sql_data(parameter_list):
    im_print = parameter_list.get('imprint')
    pill_name = parameter_list.get('pill_name')
    sha_pe = parameter_list.get('shape')
    col_or = parameter_list.get('color')

    db_engine = db_connect()
    schema_name = 'rxid'
    table_name = 'rxid_meds_data'
    table_string = schema_name + '.' + table_name 
    query = """SELECT
                    author,
                    splimprint,
                    image_id,
                    spl_strength,
                    spl_ingredients,
                    splsize,
                    splcolor_text,
                    splshape_text,
                    product_code,
                    DEA_SCHEDULE_CODE,
                    setid,
                    spl_inactive_ing
                    FROM """ + table_string + """ 
                WHERE """
```

The API was...
