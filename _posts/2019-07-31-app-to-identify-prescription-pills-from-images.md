---
title: An App to Scan and Identify Your Prescription Pill
subtitle: Using SQL, Machine Learning and AWS to match picture of your pill
image: https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-sample-pill.JPG
---

A group of developers sought to build an app with a functionality that would let users scan their prescription pills by taking a picture with their device. After scannig the pill we would need to return a match for the scanned medication from a database we did not have in hand.

---> Record DEMO Video of my own RxID and Place HERE <---

![RxId Landing Page](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-landing-page.png)

## Database & API

Two of us were given 16 days to give this a try. After looking around we found [Pillbox's API from the National Library of Medicine](https://github.com/carlos-gutier/carlos-gutier.github.io/tree/master/_posts) which for greater flexibility we turned into our own SQL database stored on [AWS RDS](https://aws.amazon.com/rds/) using PostgreSQL. The database contains information about prescription pills such as the producer, active and inactive ingredients, shape, color, and *pill imprint* which we would use later to identify the pills for our machine learning model.

You would think that building a Machine Learning model to identify prescription pills would be the hardest task of this project, but actually putting getting the database and creating an API for the developers to work with was the biggest challenge for us.

After putting togeter our SQL database we had to write code to query it. Here's part of the code used to obtain the description we needed for each pill identified. Users would have the the option of just taking the picture or providing the imprint, name, shape and color of the pill as a second option (which would return more exact results):
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

After quering our database we would post our results in a Flask application on [AWS EC2](https://aws.amazon.com/ec2/) as an endpoint of our work for the developers:

```python
# _____ imports _____________
from flask import Flask, request, render_template, jsonify, url_for
from joblib import load
from flask_cors import CORS
import pandas as pd
import json

# ______ Module imports _____
from rxid_util import parse_input
from rds_lib import db_connect, query_sql_data, query_from_rekog
from rekog import post_rekog, post_rekog_with_filter
from nnet import shape_detect
from ocr_site import allowed_image, add_to_s3, file_upload


""" create + config Flask app obj """
application = Flask(__name__)
CORS(application)


# _________ / HOME  _________________
@application.route("/")
def index():
    return render_template("upload.html")


# ________  /rekog/  route __________
@application.route('/rekog', methods=['GET', 'POST'])
def rekog():
    if request.method == 'POST':
        post_params = request.get_json(force=True)
        print('rekog started - params:', post_params)
        rekog_info = post_rekog(post_params)
        shape_info = shape_detect(post_params)
        print('rekog complete - found:', rekog_info)
        output_info = query_from_rekog(rekog_info)
        return jsonify(output_info)
    else:
        return jsonify("YOU just made a GET request to /rekog")
```

## Identifying Pills from Images
We knew there were two ways to approach this problem. One was to build our own solution by using optical character recognition (OCR) libraries. The other option was to use a text recognition service through [AWS Rekognition](https://aws.amazon.com/rekognition/), [Google Vision](https://cloud.google.com/vision/) or [Azure Computer Vision](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/).

### Using OpenCV and Tesseract
We first tried to create our own model.
