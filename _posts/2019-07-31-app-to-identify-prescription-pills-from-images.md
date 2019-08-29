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
We knew there were two ways to approach this problem. One was to build our own solution by using image processing and optical character recognition (OCR) libraries. The other option was to use a text recognition service through [AWS Rekognition](https://aws.amazon.com/rekognition/), [Google Vision](https://cloud.google.com/vision/) or [Azure Computer Vision](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/).

#### *Using OpenCV and Tesseract*
Initially we worked with a model using OpenCV for text detection and Tesseract for text recognition. Here's how we would detect the text area in the image and create bounding boxes around the it:

```python
# grab the number of rows and columns from the scores volume, then
# initialize our set of bounding box rectangles and corresponding
# confidence scores
(numRows, numCols) = scores.shape[2:4]
rects = []
confidences = []

# loop over the number of rows
for y in range(0, numRows):
    # extract the scores (probabilities), followed by the geometrical
    # data used to derive potential bounding box coordinates that souround text
    scoresData = scores[0, 0, y]
    xData0 = geometry[0, 0, y]
    xData1 = geometry[0, 1, y]
    xData2 = geometry[0, 2, y]
    xData3 = geometry[0, 3, y]
    anglesData = geometry[0, 4, y]

    # loop over the number of columns
    for x in range(0, numCols):
	# if our score does not have sufficient probability, ignore it
	if scoresData[x] < 0.5:
	    continue

	# compute the offset factor as our resulting feature maps will
	# be 4x smaller than the input image
	(offsetX, offsetY) = (x * 4.0, y * 4.0)

	# extract the rotation angle for the prediction and then
	# compute the sin and cosine
	angle = anglesData[x]
	cos = np.cos(angle)
	sin = np.sin(angle)

	# use the geometry volume to derive the width and height of
	# the bounding box
	h = xData0[x] + xData2[x]
	w = xData1[x] + xData3[x]

	# compute both the starting and ending (x, y)-coordinates for
	# the text prediction bounding box
	endX = int(offsetX + (cos * xData1[x]) + (sin * xData2[x]))
	endY = int(offsetY - (sin * xData1[x]) + (cos * xData2[x]))
	startX = int(endX - w)
	startY = int(endY - h)

	# add the bounding box coordinates and probability score to
	# our respective lists
	rects.append((startX, startY, endX, endY))
	confidences.append(scoresData[x])

# apply non-maxima suppression to suppress weak, overlapping bounding
# boxes
boxes = non_max_suppression(np.array(rects), probs=confidences)

# loop over the bounding boxes
for (startX, startY, endX, endY) in boxes:
    # scale the bounding box coordinates based on the respective ratios
    startX = int(startX * rW)
    startY = int(startY * rH)
    endX = int(endX * rW)
    endY = int(endY * rH)

# draw the bounding box on the image
cv2.rectangle(orig, (startX, startY), (endX, endY), (0, 255, 0), 2)

# save the output image with bounding boxes
cv2.imwrite('/home/ec2-user/SageMaker/detected_images/DETECT_cap.10.jpg', orig)
```

This is the image of the pill with bounding boxes around the detected text:

```python
detected_img = imageio.imread('/home/ec2-user/SageMaker/detected_images/DETECT_cap.10.jpg')
plt.imshow(detected_img);
```
![Pill image with bounding boxes around text](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-bounding-box.png)

And to obtain the text within the bounding:
```python
config = ("-l eng --oem 1 --psm 7")
text = pytesseract.image_to_string(roi, config=config)
```
```python

# loop over the results
for ((startX, startY, endX, endY), text) in results:
    # display the text OCR'd by Tesseract
    print("OCR TEXT")
    print("========")
    print("{}\n".format(text))
```
![OCR text results](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/rxid-ocrtext.png)


The model's accuracy in detecting and reading text was very good, except when it came to images of pills with etched (see sample image below), which account to a good number of prescription pills in the market.
 <img src="https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/sample_pill_FAIL.jpg" alt="Pill image with etched imprint" width="60"> 

Even after applying several image filters we were not able to get OpenCV to detect the text. We could have spent more time applying other filters and tweaking OpenCV parameters but we were running out of time so we decided to opted to implement our other option.

#### *Implementing AWS Rekognition*

Working with AWS Rekognition was fairly simple and it provided mostly accurate results. Below is code showing our implementation:
```python
# ------------- Detecting text from original image ------------
with open(imageFile, 'rb') as image:
    print('detect started', imageFile)
    response = client.detect_text(Image={'Bytes': image.read()})
    print('detect completed', imageFile)

# ------------- Detected Text (List of Dictionaries) -------------
textDetections = response['TextDetections']

# ------------- Parsing Through Detected Text and 
# Making list of Unique Sets of Text Dectected -------------
text_found = []
for text in textDetections:
    if text['Confidence'] > con_fidence:
        text_found.append(text['DetectedText'])
text_set = list(set(text_found))
```
Along with the application of a few image filters Rekognition did not have much trouble in identifying text in pills with etched imprints. The ease of use along with accuracy made it easy for us to take as our final choice for the job.

## Conclusion
If time allowed we would have taken more time to make the OpenCV and Tesseract model work with etched pills. Also would have build a shape detection model that along with the pill imprint code would provide us with the pill shape to increase accuracy when querying the database.
